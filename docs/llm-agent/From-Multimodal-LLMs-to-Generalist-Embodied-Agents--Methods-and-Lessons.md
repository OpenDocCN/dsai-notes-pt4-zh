<!--yml
category: 未分类
date: 2025-01-11 11:49:06
-->

# From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons

> 来源：[https://arxiv.org/html/2412.08442/](https://arxiv.org/html/2412.08442/)

Andrew Szot $\thanks{Core contributor}^{\ 1,2}$  Bogdan Mazoure^(∗1)  Omar Attia¹  Aleksei Timofeev¹
Harsh Agrawal¹  Devon Hjelm¹  Zhe Gan¹  Zsolt Kira²  Alexander Toshev¹
¹ Apple, ² Georgia Tech
a.szot@apple.com, toshev@apple.com Core contributor

###### Abstract

We examine the capability of Multimodal Large Language Models (MLLMs) to tackle diverse domains that extend beyond the traditional language and vision tasks these models are typically trained on. Specifically, our focus lies in areas such as Embodied AI, Games, UI Control, and Planning. To this end, we introduce a process of adapting an MLLM to a Generalist Embodied Agent (GEA). GEA is a single unified model capable of grounding itself across these varied domains through a multi-embodiment action tokenizer. GEA is trained with supervised learning on a large dataset of embodied experiences and with online RL in interactive simulators. We explore the data and algorithmic choices necessary to develop such a model. Our findings reveal the importance of training with cross-domain data and online RL for building generalist agents. The final GEA model achieves strong generalization performance to unseen tasks across diverse benchmarks compared to other generalist models and benchmark-specific approaches.

## 1 Introduction

![Refer to caption](img/b1ada9cee1182bf19d564915db76100c.png)

Figure 1: The Generalist Embodied Agent (GEA) is a multimodal LLM-based agent that can complete tasks from natural language instructions across a variety of domains and embodiments spanning manipulation, planning, game playing, and UI control. A pretrained MLLM is finetuned with supervised finetuning (SFT) on a large dataset of embodied experiences. The final GEA model is then finetuned with reinforcement learning (RL). GEA achieves competitive results in generalization to unseen settings.

Foundation Models have demonstrated broad capabilities across language and image understanding tasks [[5](https://arxiv.org/html/2412.08442v1#bib.bib5), [30](https://arxiv.org/html/2412.08442v1#bib.bib30), [66](https://arxiv.org/html/2412.08442v1#bib.bib66), [46](https://arxiv.org/html/2412.08442v1#bib.bib46), [16](https://arxiv.org/html/2412.08442v1#bib.bib16), [51](https://arxiv.org/html/2412.08442v1#bib.bib51), [45](https://arxiv.org/html/2412.08442v1#bib.bib45), [103](https://arxiv.org/html/2412.08442v1#bib.bib103), [95](https://arxiv.org/html/2412.08442v1#bib.bib95), [43](https://arxiv.org/html/2412.08442v1#bib.bib43), [42](https://arxiv.org/html/2412.08442v1#bib.bib42), [58](https://arxiv.org/html/2412.08442v1#bib.bib58), [33](https://arxiv.org/html/2412.08442v1#bib.bib33)]. In particular, Multimodal LLMs (MLLMs) – multimodal foundation models trained on vast amounts of textual and image data – excel at tasks that are natural to their text and image training modalities. As an extension of MLLMs, Vision-Language-Action models have been successfully applied in robotics and embodied AI [[3](https://arxiv.org/html/2412.08442v1#bib.bib3), [11](https://arxiv.org/html/2412.08442v1#bib.bib11), [18](https://arxiv.org/html/2412.08442v1#bib.bib18), [80](https://arxiv.org/html/2412.08442v1#bib.bib80)], as well as agents for the web  [[75](https://arxiv.org/html/2412.08442v1#bib.bib75), [102](https://arxiv.org/html/2412.08442v1#bib.bib102), [37](https://arxiv.org/html/2412.08442v1#bib.bib37), [27](https://arxiv.org/html/2412.08442v1#bib.bib27)] and user interface (UI) control [[68](https://arxiv.org/html/2412.08442v1#bib.bib68), [28](https://arxiv.org/html/2412.08442v1#bib.bib28), [89](https://arxiv.org/html/2412.08442v1#bib.bib89)].

These applications have demonstrated that MLLMs can be successfully applied to diverse domains for the purpose of controlling various embodiments like robots, playing games, and controlling devices via UIs. As many of these domains share similarities, it is natural to ask how a single agent can be trained to be generally proficient in all of these domains. This is a challenging problem as many of these tasks require physics and geometric reasoning, their embodiments are either static or share morphologies via a mobile manipulator, their applications require long-horizon planning, and many are partially observable and require reasoning over long sequences of observations. In addition, training with combined data across domains with these similarities may yield cross-domain benefits, where a single agent may outperform agents trained on individual domains.

In this work, we demonstrate an approach for adapting an MLLM into a single Generalist Embodied Agent (GEA) to solve a vast number of tasks across diverse domains spanning manipulation, navigation, video game playing, and UI control. To enable GEA to control diverse embodiments, we learn a unified learned tokenization mechanism across all continuous and discrete action spaces. As [Figure 1](https://arxiv.org/html/2412.08442v1#S1.F1 "In 1 Introduction ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") illustrates, we then employ supervised finetuning (SFT) [[87](https://arxiv.org/html/2412.08442v1#bib.bib87)] to adapt a pretrained MLLM to predict actions from trajectories of agents successfully completing tasks. This SFT dataset spans over 2.2 million trajectories from diverse collection methods like human labelers or learned policies. While this SFT process produces a capable agent, it suffers from an inherent lack of data, specifically data diversity, and rarely exhibits robustness to mistakes. We thus also train GEA with a second stage of online reinforcement learning (RL) training over a subset of the domains where the agent collects and learns from data in interactive simulators.

We demonstrate that GEA exhibits strong generalist capabilities. Specifically, it reaches state-of-the-art performance across many benchmarks against other generalist agents and even outperforms or closely matches bespoke specialist systems. For example, in the CALVIN manipulation benchmark [[60](https://arxiv.org/html/2412.08442v1#bib.bib60)], GEA reaches $90\%$ success rate while operating on unseen instructions and background, which is nearly $10\%$ higher than similar methods [[48](https://arxiv.org/html/2412.08442v1#bib.bib48)] and closely matches the performance of specialist systems [[35](https://arxiv.org/html/2412.08442v1#bib.bib35)]. In a Habitat mobile pick task [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)], GEA achieves $83\%$ success in unseen scenes, outperforming a policy trained with RL on the ground truth simulator state. Similarly, in Procgen video games [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)] GEA reaches $44\%$ of expert score, which is almost $20\%$ higher than prior specialist [[59](https://arxiv.org/html/2412.08442v1#bib.bib59)] models.

We also analyze the relationship between the generalist capabilities of GEA and its training data and base MLLM. We demonstrate that training on the combined data from a diverse set of domains for SFT provides a cross-domain performance boost over using only per-domain data. Finally, we explore the role of RL and online data collection for building generalist agents and empirically demonstrate the benefits of online RL over prior approaches of iterative SFT or offline RL.

As a further contribution to the community, we will release the code for training and evaluating GEA along with the GEA model itself. We will add the link to the code and model to this paper when they are ready for release.

## 2 Related Work

Prior works have explored building generalist agents by training policies on large multi-task datasets, illustrating the importance of scaling interactive data to create capable multi-task agents [[69](https://arxiv.org/html/2412.08442v1#bib.bib69), [20](https://arxiv.org/html/2412.08442v1#bib.bib20), [88](https://arxiv.org/html/2412.08442v1#bib.bib88)]. Additionally, prior works have studied new architectures for generalist agents [[86](https://arxiv.org/html/2412.08442v1#bib.bib86), [24](https://arxiv.org/html/2412.08442v1#bib.bib24)], while others focus on applying generalist agents to robotic contexts [[83](https://arxiv.org/html/2412.08442v1#bib.bib83), [10](https://arxiv.org/html/2412.08442v1#bib.bib10), [9](https://arxiv.org/html/2412.08442v1#bib.bib9)]. Some research also investigates generalist models in domain-specific benchmarks [[34](https://arxiv.org/html/2412.08442v1#bib.bib34), [82](https://arxiv.org/html/2412.08442v1#bib.bib82)] or in cross-embodiment scenarios [[65](https://arxiv.org/html/2412.08442v1#bib.bib65), [17](https://arxiv.org/html/2412.08442v1#bib.bib17)]. Like GEA, these approaches leverage extensive data, yet our work emphasizes the importance of adapting a pretrained MLLM via both finetuning and online RL. For example, differences between GEA and Gato [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)], are that GEA leverages RL, utilizes a pretrained MLLM, learns a multi-embodiment action tokenizer, and focuses on evaluating generalization to new task settings. As a result, GEA empirically outperforms Gato in many settings.

Like GEA, some prior work focuses on adapting MLLMs as agents. Works have proposed domain-specific pipelines for using the zero-shot capabilities of MLLMs for decision-making [[31](https://arxiv.org/html/2412.08442v1#bib.bib31), [3](https://arxiv.org/html/2412.08442v1#bib.bib3), [100](https://arxiv.org/html/2412.08442v1#bib.bib100), [50](https://arxiv.org/html/2412.08442v1#bib.bib50), [91](https://arxiv.org/html/2412.08442v1#bib.bib91), [85](https://arxiv.org/html/2412.08442v1#bib.bib85)], while our work focuses on finetuning MLLMs. Other works investigate schemes for finetuning MLLMs for decision-making and the benefits of doing so, but also in the context of specific domains [[48](https://arxiv.org/html/2412.08442v1#bib.bib48), [76](https://arxiv.org/html/2412.08442v1#bib.bib76), [81](https://arxiv.org/html/2412.08442v1#bib.bib81), [80](https://arxiv.org/html/2412.08442v1#bib.bib80)]. Prior work also finetunes MLLMs as generalist agents [[36](https://arxiv.org/html/2412.08442v1#bib.bib36), [11](https://arxiv.org/html/2412.08442v1#bib.bib11), [63](https://arxiv.org/html/2412.08442v1#bib.bib63)]. However, our work shows results across more diverse domains and studies the importance of supervised learning with multi-domain data and RL finetuning. Architecturally, GEA is different from OpenVLA [[36](https://arxiv.org/html/2412.08442v1#bib.bib36)] in that it uses a learned multi-embodiment action tokenizer as opposed to a uniform discretization, which prior work has demonstrated to perform better [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)]. Moreover, works have explored the value of finetuning MLLMs as UI agents [[4](https://arxiv.org/html/2412.08442v1#bib.bib4), [97](https://arxiv.org/html/2412.08442v1#bib.bib97), [23](https://arxiv.org/html/2412.08442v1#bib.bib23), [19](https://arxiv.org/html/2412.08442v1#bib.bib19), [62](https://arxiv.org/html/2412.08442v1#bib.bib62)]. While GEA explores adapting MLLMs as policies, there are also other ways to leverage MLLMs such as via reward models [[56](https://arxiv.org/html/2412.08442v1#bib.bib56), [55](https://arxiv.org/html/2412.08442v1#bib.bib55)], world models [[90](https://arxiv.org/html/2412.08442v1#bib.bib90)], or environment generation [[94](https://arxiv.org/html/2412.08442v1#bib.bib94)].

More broadly, prior work has demonstrated how LLMs can be used for agents that can reason and interact. Various works focus on training LLM agents through specific pipelines [[101](https://arxiv.org/html/2412.08442v1#bib.bib101)]. Similar to how GEA finetunes for decision-making capabilities not present in the base LLM, other works demonstrate the value of finetuning LLMs for reasoning and problem-solving capabilities [[92](https://arxiv.org/html/2412.08442v1#bib.bib92), [77](https://arxiv.org/html/2412.08442v1#bib.bib77)]. Prior works also demonstrate the value of finetuning LLMs with self-generated data [[54](https://arxiv.org/html/2412.08442v1#bib.bib54)], mistakes from the LLM [[74](https://arxiv.org/html/2412.08442v1#bib.bib74)], and with RL [[39](https://arxiv.org/html/2412.08442v1#bib.bib39)]. Likewise, GEA shows the importance of using such RL training, beyond just supervised learning, to create a capable agent.

![Refer to caption](img/677090694bab52ad017e2e36da313212.png)

Figure 2: GEA utilizes a pretrained MLLM together with a multi-embodiment action tokenizer to enable a generalist agent to operate across a wide range of domains, embodiments, and action spaces. GEA takes as input information about the embodiment and desired task with the embodiment prompt and instruction and the observation visuals (bottom). It produces a sequence of action tokens in the LLM vocabulary, which are decoded by the multi-embodiment action detokenizer into an action for the appropriate embodiment and action space.

## 3 Generalist Embodied Agent

### 3.1 Problem Settings

We focus on language-specified tasks with visual observations. Specifically, we consider the goal-specified Partially-Observable Markov Decision Processes (POMDPs) [[8](https://arxiv.org/html/2412.08442v1#bib.bib8)] with observation space $\mathcal{O}$, action space $\mathcal{A}$, goal space $\mathcal{G}$, and a reward model $R$. For brevity, we omit other elements of the MDP. In our settings, $\mathcal{G}$ is represented by a textual description of the task to solve. Observations consist of RGB images from the agent, which can either be from an agent camera in the case of embodied AI applications or screenshots in the case of video games or UI interactions.

We consider a diverse set of environment types, which we refer to as *domains* (see Table [1](https://arxiv.org/html/2412.08442v1#S4.T1 "Table 1 ‣ 4.2 Continuous Multi-Embodiment Tokenizer ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") for examples). These domains specify a diverse set of action spaces spanning various robotic control spaces, high-level primitives, and computer UI interfaces. Our goal is to learn one policy that operates over a number of environments, which we denote by $\mathcal{M}_{i}=(\mathcal{O}_{i},\mathcal{A}_{i},\mathcal{G}_{i},R_{i})$ for each environment $i\in\mathcal{E}$.

### 3.2 GEA Architecture

The Generalist Embodied Agent (GEA) performs tasks by producing actions that are executed in an environment conditioned on observations, past actions, and a task description. More formally, for timestep $t$ in environment $\mathcal{M}_{i}$, the GEA model takes as input an environment specific prompt $P_{i}$, followed by a task instruction $I\in\mathcal{G}_{i}$ and up to $c$ interleaved previous observations and actions $o_{t-c},a_{t-c},\dots,o_{t}$ from $\mathcal{O}_{i}$ and $\mathcal{A}_{i}$. From these inputs, the GEA model predicts action $a_{t}\in\mathcal{A}_{i}$ to execute in the environment. The prompt provides information about the environment and embodiment to control. The task instruction is a natural language description of the task the agent is to execute.

Multi-Embodiment Action Tokenizer. We study Generalist Embodied Agent (GEA) adapted from an MLLM. As MLLMs naturally consume only text and images and generate only text, we follow the findings of Szot et al. [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)] to modify the LLM vocabulary to represent actions. First, we represent all actions across $\{\mathcal{M}_{i}\}$, $i\in\mathcal{E}$ with two vocabularies: $V_{\textrm{disc}}$ for discrete actions and $V_{\textrm{cont}}$ for continuous actions, so that the final vocabulary is $V=V_{\textrm{disc}}\cup V_{\textrm{cont}}$. A discrete action is described by language, and then this language is tokenized into a sequence of text tokens representing this action. $V_{\textrm{disc}}$ is defined as all such text token sequences representing the discrete actions.

As continuous actions are not readily expressed as text, we use a learned action tokenizer that maps each continuous action into a sequence of new tokens, whose vocabulary we denote by $V_{\textrm{cont}}$. Details of how we train this tokenizer are in [Section 4.2](https://arxiv.org/html/2412.08442v1#S4.SS2 "4.2 Continuous Multi-Embodiment Tokenizer ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). We replace the $|V_{\textrm{cont}}|$ most infrequently used tokens in the original LLM vocabulary with $V_{\textrm{cont}}$.

## 4 Training

GEA starts from a base MLLM and first trains a continuous action tokenizer. As depicted in [Figure 3](https://arxiv.org/html/2412.08442v1#S4.F3 "In 4.3 Stage 1: Supervised-Instruction Finetuning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), the MLLM is adapted to GEA-Base by supervised finetuning on embodied experiences. Next, GEA-Base is adapted to the full GEA model through supervised and reinforcement learning.

### 4.1 Base MLLM

The primary consideration for selecting a base model beyond its inherent vision-language strength is its ability to scale to long contexts as embodied data consists of long trajectories of interleaved observations and actions. We thus build GEA off LLaVA-OneVision [[44](https://arxiv.org/html/2412.08442v1#bib.bib44)], a model specifically trained to handle sequences of images through interleaved image/text pairs and videos which extends to GEA operating over a history of observations.

### 4.2 Continuous Multi-Embodiment Tokenizer

To obtain a vocabulary $V_{\textrm{cont}}$ for continuous actions and a tokenizer/de-tokenizer for these actions, we follow Szot et al. [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)] and train a Residual VQ-VAE (RVQ) [[40](https://arxiv.org/html/2412.08442v1#bib.bib40)] model over action vectors. The RVQ model is a variational autoencoder that leverages a sequence of discrete embeddings to represent the data. Specifically, it encodes an action $a$ as a sequence of $M$ tokens $k_{1}(a),\ldots,k_{M}(a)$, where each token denotes a code from a learned vocabulary $V_{\textrm{cont}}^{m}$, for $m\in{1,\ldots,M}$. A key feature of RVQ is that the $m^{\textrm{th}}$ vocabulary is trained to encode the residual of the action after it has been encoded with vocabularies ${1,\ldots,m-1}$. This hierarchical encoding makes RVQ effective in precisely representing continuous actions with a minimal number of discrete tokens. The final continuous action vocabulary used by GEA is the union of the RVQ vocabularies $V_{\textrm{cont}}=\bigcup_{m}V_{\textrm{cont}}^{m}$.

The main distinction from Szot et al. [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)] is that we train a single tokenizer/de-tokenizer across all continuous action spaces. As shown in [Table 1](https://arxiv.org/html/2412.08442v1#S4.T1 "In 4.2 Continuous Multi-Embodiment Tokenizer ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), these spaces cover a variety of robotic control types, including end-effector, joint velocity, and joint position control. To facilitate training a unified RVQ, we pad all action vectors to the maximum action dimension. During inference, we decode the predicted action tokens and then truncate the output to match the dimensionality of the specific embodiment’s action space. We use 2 codebooks each with 512 tokens and a token vector dimension of 1024\. Additional details on action tokenization are in [Appendix A](https://arxiv.org/html/2412.08442v1#A1 "Appendix A Continuous Multi-Embodiment Tokenizer Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons").

 | Dataset Name | Domain | Action Type | Embodiment Type | # Trajs | Data Source |
| --- | --- | --- | --- | --- | --- |
| OpenX [[63](https://arxiv.org/html/2412.08442v1#bib.bib63)] | Static Manip | Cont. Various | 22 Various Robots | 1.2M | Various |
| Meta-World [[98](https://arxiv.org/html/2412.08442v1#bib.bib98)] | Static Manip. | Cont. EE+Gripper | Sawyer | 45k | Scripted |
| CALVIN [[60](https://arxiv.org/html/2412.08442v1#bib.bib60)] | Static Manip. | Cont. EE+Gripper | Franka Arm | 18k | Human |
| Maniskill [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)] | Static Manip. | Cont. Joint Velocity | ROKAE xMate3Pro | 5k | Motion Planner |
| Habitat Pick [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)] | Mobile Manip. | Cont. Joint Position + Base | Fetch | 50k | RL Expert |
| Habitat Place [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)] | Mobile Manip. | Cont. Joint Position+Base | Fetch | 50k | RL Expert |
| Habitat Nav [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)] | Navigation | Cont. Velocity | Fetch | 13k | Shortest Path |
| BabyAI [[14](https://arxiv.org/html/2412.08442v1#bib.bib14)] | Navigation | Discrete | Virtual | 50k | Shortest Path |
| LangR [[80](https://arxiv.org/html/2412.08442v1#bib.bib80)] | Planning | Discrete | Fetch | 150k | RL Expert |
| Procgen [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)] | Video Games | Discrete | Virtual | 320k | RL Expert |
| Atari [[7](https://arxiv.org/html/2412.08442v1#bib.bib7)] | Video Games | Discrete | Virtual | 286k | RL Expert |
| AndroidControl [[47](https://arxiv.org/html/2412.08442v1#bib.bib47)] | UI Control | Mixed | Virtual | 14k | Human | 

Table 1: Overview of the embodied datasets used for training GEA. The actions in each dataset can be either discrete or continuous with a specific control space for the continuous actions. The embodiment type describes the agent being controlled. Each trajectory in a dataset refers to a sequence of images and actions. The data source refers to the collection method for these trajectories.

### 4.3 Stage 1: Supervised-Instruction Finetuning

The first step of GEA is to use supervised-instruction finetuning (SFT) to adapt the base MLLM for embodied decision-making (left side of [Fig. 3](https://arxiv.org/html/2412.08442v1#S4.F3 "In 4.3 Stage 1: Supervised-Instruction Finetuning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")). We use a collection $\mathcal{D}=\bigcup_{i\in\mathcal{E}}\mathcal{D}_{i}$ of demonstration datasets from all environments $\mathcal{E}$. During this stage, we use a standard cross-entropy loss over actions in the case of interactive data or responses in the case of vision-language data. As is typically done in MLLM training, we maximize the negative log-likelihood of predicting these output tokens for each example:

|  | $\mathcal{L}_{\textrm{SFT}}(\mathcal{D})=-\!\!\!\!\!\!\!\!\!\!\!\!\sum_{(I,o_{t% -c:t},a_{t-c:t})\in\mathcal{D}}\!\!\!\!\!\!\!\!\!\!\!\!\!\!\!\!\log p(a_{t}&#124;P,% I,o_{t-c},a_{t-c},\dots,o_{t})$ |  | (1) |

We then train the base MLLM over all the above datasets with SFT to obtain the GEA-Base model.

Training Details. The entire GEA-Base model is initialized from the base MLLM. We train for 75k updates using AdamW [[53](https://arxiv.org/html/2412.08442v1#bib.bib53)] with cosine learning rate decay and linear learning rate warmup for the first $10\%$ of training steps; learning rate of $1e^{-5}$; global batch size of 256 and an observation context length of $c=3$. Training takes around 2 days on 8 nodes of 8xH100 GPUs (see [Appendix B](https://arxiv.org/html/2412.08442v1#A2 "Appendix B SFT Training Additional Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")).

![Refer to caption](img/0fe9d41ee21c7b02002550877fea6804.png)

Figure 3: GEA training stages. First, a MLLM is adapted to GEA-Base by finetuning the entire MLLM with SFT on interactive data. Next, GEA-Base is finetuned jointly with online RL (PPO) and SFT on the original data with LoRA.

### 4.4 Stage 2: Online Reinforcement Learning

While the previously described SFT training process produces a capable GEA-Base agent, it is only trained on a limited set of expert trajectories, which rarely demonstrate diverse behaviors like error recovery. Hence, we propose to utilize online RL for some of the environments. In a second stage of training, we continue to train GEA-Base with RL in addition to SFT to obtain the final GEA model (right side of [Fig. 3](https://arxiv.org/html/2412.08442v1#S4.F3 "In 4.3 Stage 1: Supervised-Instruction Finetuning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")). For online RL, we train the GEA-Base agent with PPO [[72](https://arxiv.org/html/2412.08442v1#bib.bib72)], whose optimization loss is denoted by $\mathcal{L}_{\textrm{PPO}}(\mathcal{M}_{i})$ for each environment in which we have a simulator $i\in\mathcal{E}_{\textrm{PPO}}\subset\mathcal{E}$. We combine this objective with the SFT objective from [Eq. 1](https://arxiv.org/html/2412.08442v1#S4.E1 "In 4.3 Stage 1: Supervised-Instruction Finetuning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") to obtain the final GEA objective:

|  | $\mathcal{L}_{\text{GEA}}=\sum_{i\in\mathcal{E}_{\textrm{PPO}}}\mathcal{L}_{% \textrm{PPO}}(\mathcal{M}_{i})+\lambda\sum_{i\in\mathcal{E}}\mathcal{L}_{% \textrm{SFT}}(\mathcal{D}_{i})$ |  | (2) |

where we weight the SFT loss by $\lambda=0.1$ to emphasize the RL loss. This RL+SFT training stage on the GEA-Base model produces the final model we refer to as GEA.

PPO Details. The GEA value function for RL is an MLP network which is initialized from scratch. It takes as input the MLLM final layer activations at the observation token step just before the action tokens and an average pooled representation of the visual embeddings from the MLLM vision encoder. The critic also optionally takes any privileged state information about the task since the critic is only used during training, and not during inference.

To stabilize RL training across numerous environments, we use PopArt [[84](https://arxiv.org/html/2412.08442v1#bib.bib84)] return normalization to account for the diverse reward distributions across these environments. Since the output space of the LLM can consist of many possible tokens, we use constrained decoding to force the autoregressive action sampling to be within the action space for the environment. For continuous action tasks, this amounts to constraining the output to the learned continuous action tokens. For the discrete control tasks this amounts to constraining the output to the valid language actions (for example, ”pick apple” or ”right”). To account for differences in the valid distribution of actions per environment, we normalize the entropy for PPO so a single entropy coefficient can apply to all tasks.

Training Details. Since GEA-Base already obtains some success, and RL introduces GPU memory overhead via environments simulated on the GPU, we use LoRA [[29](https://arxiv.org/html/2412.08442v1#bib.bib29)] to finetune the LLM while freezing all other components. All environments use a rollout length of 128, a learning rate of $3e^{-4}$, an entropy coefficient of $1e^{-4}$, and a value function learning loss of $1.5e^{-4}$. RL finetuning uses 8 nodes of 8xH100 GPUs with 4 parallel environments per GPU. Environments are partitioned per node into 3 of the GPUs running HabPick, 3 running Procgen, and 2 running LangR. The SFT per-device batch size is 2\. We train for 100M cumulative steps across all tasks, which takes around 1 day. Full RL training details are in [Appendix C](https://arxiv.org/html/2412.08442v1#A3 "Appendix C RL Training Additional Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons").

|  | GEA | Prior Work | # Tasks | Generalization Type |
| Manipulation |  |  |  |  |
| Meta-World | 94.7 | 84 MLLM+IL [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)]^S 87.0 [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)]^G | 45 | object positions |
| CALVIN (ABC →D) | 90.0 | 82.4 MLLM+IL [[48](https://arxiv.org/html/2412.08442v1#bib.bib48)]^S 92.2 IL+pointcloud[[35](https://arxiv.org/html/2412.08442v1#bib.bib35)] | 34* | instructions, background |
| Maniskill | 13.6 | 6.5 IL [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)]^S 47.8 IL+PPO [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)]^S | 5 | object positions |
| Habitat Pick | 82.5 | 29 IL [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)]^S 81.0 RL + sim state^S | 20 | house |
| Habitat Place | 93.5 | 95.5 RL + sim state^S | 10 | house |
| Video Games |  |  |  |  |
| Procgen | 44.0 | 25 [[59](https://arxiv.org/html/2412.08442v1#bib.bib59)]^S | 16 | background |
| Atari | 32.7 | 31 [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)]^G 85 Offline RL [[41](https://arxiv.org/html/2412.08442v1#bib.bib41)]^S | 44 | none |
| Navigation |  |  |  |  |
| Habitat Nav | 62.5 | 72 [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)]^S | 10 | house |
| BabyAI | 91.1 | 93.2 [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)]^G | 17* | instructions, grid state |
| UI Control |  |  |  |  |
| AndroidControl | 57.3 | 45 GPT-4o+SoM [[93](https://arxiv.org/html/2412.08442v1#bib.bib93)]^G | 35* | instructions |
| Planning |  |  |  |  |
| LangR | 50.0 | 51 MLLM+RL[[81](https://arxiv.org/html/2412.08442v1#bib.bib81)]^S | 10* | instructions, house |

Table 2: Zero-shot generalization of GEA to new tasks in terms of success rate % and in the video games tasks % of expert performance. We compare against prior works consisting of domain specialists (with superscript “S”) that are trained on only data from that benchmark and domain generalists (with superscript “G”) that are trained on data from several benchmarks. Bold indicates best, underline close second, and gray coloring that the method assumes access to additional input modalities like pointcloud or ground truth simulator state, meaning it is not a fair comparison to GEA. The “Prior Work” column also gives details about how the methods were trained (IL or RL) and if they assume additional input modalities. The “# Tasks” column gives a general indication of the number of distinct evaluation settings with a “*” indicating each task also has diverse language instructions.

## 5 Datasets and Environments

We use a diverse set of domains with associated environments and datasets (see [Table 1](https://arxiv.org/html/2412.08442v1#S4.T1 "In 4.2 Continuous Multi-Embodiment Tokenizer ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")). This section introduces these domains followed by an explanation of how we use them in stage 1 and 2 of our training procedure.

Static Manipulation. These datasets are of a fixed robot manipulator interacting with objects. Some of these datasets are simulated table top interactions with rigid-body objects such as Meta-World [[99](https://arxiv.org/html/2412.08442v1#bib.bib99)], CALVIN [[60](https://arxiv.org/html/2412.08442v1#bib.bib60)] and Maniskill [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)]. We also leverage a large dataset of interactions on real robot platforms [[63](https://arxiv.org/html/2412.08442v1#bib.bib63)]. These datasets span a variety of control spaces in end-effector and joint control. The camera is typically mounted in a static position so that the workspace and robot arm are fully visible.

Mobile Manipulation. We also investigate setups where the robot manipulator moves via a mobile base. We use the object rearrangement tasks from the Habitat platform [[71](https://arxiv.org/html/2412.08442v1#bib.bib71)] for datasets in these tasks. These datasets cover object picking and placing tasks where the robot starts up to 2 meters away from the object. The robot has to coordinate moving its base and arm to successfully pick up the object. These datasets involve first person egocentric cameras.

Navigation. We also use datasets of navigation in isolation. We use datasets of simulated robot navigation in Habitat. We also use navigation in grid-world environments from BabyAI [[14](https://arxiv.org/html/2412.08442v1#bib.bib14)]. Both datasets were collected with shortest path experts.

Video games. We use datasets from two standard benchmarks for decision making in video games, Procgen [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)] and Atari [[7](https://arxiv.org/html/2412.08442v1#bib.bib7)]. Both datasets were collected by RL agents which were separately trained to solve each individual game. We convert these tasks to be language-conditioned by providing the game name along with a short description of the game’s objective and rules. We only train on successful trajectories.

Planning. We use a dataset of successful episodes in the LangR dataset [[80](https://arxiv.org/html/2412.08442v1#bib.bib80)]. In this task the agent has to select between skill primitives that accomplish long horizon language-specified rearrangement tasks for a home robot.

UI Control. We use the AndroidControl [[47](https://arxiv.org/html/2412.08442v1#bib.bib47)] dataset of UI interaction in Android devices spanning 833 apps. The actions are combinations of tap actions specified by screen coordinates and text typing.

Vision language instruction data. To improve generalization of the model, we also include data used for training the original MLLM, which prior work has found is useful when finetuning MLLMs as control policies [[11](https://arxiv.org/html/2412.08442v1#bib.bib11)]. We used the following datasets of text and images without any actions: VQAv2 [[21](https://arxiv.org/html/2412.08442v1#bib.bib21)], OKVQA [[57](https://arxiv.org/html/2412.08442v1#bib.bib57)], A-OKVQA [[73](https://arxiv.org/html/2412.08442v1#bib.bib73)], GQA [[32](https://arxiv.org/html/2412.08442v1#bib.bib32)] and the LLaVA-Instruct-150k dataset [[52](https://arxiv.org/html/2412.08442v1#bib.bib52)].

Stage 1 Training: SFT Data. To obtain embodied data for Stage 1 Training (see Sec. [4.3](https://arxiv.org/html/2412.08442v1#S4.SS3 "4.3 Stage 1: Supervised-Instruction Finetuning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")), we collect a large dataset of language-conditioned behaviors from all of the above domains consisting of 2.2M trajectories. All trajectories are successful examples of language-conditioned behaviors with visual observations. The data is from diverse collection sources such as human demonstrations, RL-based policies, and motion planners. The dataset is diverse and spans thousands of distinct tasks and many embodiments (see [Appendix D](https://arxiv.org/html/2412.08442v1#A4 "Appendix D Dataset Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") for full details).

Stage 2 Training: RL Environments. For Stage 2 online RL (see [Sec. 4.4](https://arxiv.org/html/2412.08442v1#S4.SS4 "4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")) we use environments from the three domains of Habitat Pick [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)], Language Rearrengement (LangR) [[80](https://arxiv.org/html/2412.08442v1#bib.bib80)], and Procgen [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)]. Thus, we define $\mathcal{E}_{\textrm{PPO}}=\{\textrm{HabPick},\textrm{LangR},\textrm{ProcGen}\}$. Habitat Pick and LangR are simulated in the Habitat platform [[71](https://arxiv.org/html/2412.08442v1#bib.bib71)] and have reward functions for achieving and making progress towards the goal. In Procgen, we use all 16 games for RL and use the game specific reward functions (see [Appendix C](https://arxiv.org/html/2412.08442v1#A3 "Appendix C RL Training Additional Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") for full details).

## 6 Empirical Evaluation

We empirically demonstrate the ability of GEA as a generalist agent that can generalize to new instructions and settings across diverse embodiments and domains. We assess the role of the RL training in achieving this goal. In ablations, we study the impact of scaling data between multiple domains, compare RL to other forms of policy collected data, the impact of the the base MLLM.

### 6.1 GEA Generalization Capabilities

In this section, we evaluate the generalization capabilities of the final GEA model. We use the associated benchmarks from the datasets in [Table 1](https://arxiv.org/html/2412.08442v1#S4.T1 "In 4.2 Continuous Multi-Embodiment Tokenizer ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), which span manipulation, navigation, video games, UI control, and planning. These benchmarks evaluate agents in new settings not present in the training data, such as new object positions, scenes, visual backgrounds, tasks, or instructions. All benchmarks specify evaluation instructions in natural language and require the agent to operate from visual observations.

We report the “online” performance of GEA, meaning we evaluate its performance in an interactive simulator. The only exception is AndroidControl, where we instead check the correspondence with a ground truth test trajectory [[68](https://arxiv.org/html/2412.08442v1#bib.bib68)]. Each benchmark also evaluates agents over many distinct tasks. For example, in the Procgen video game benchmark, we report the average Procgen performance over all 16 Procgen games each of which is an entirely different video game. Full evaluation details are in [Appendix E](https://arxiv.org/html/2412.08442v1#A5 "Appendix E Evaluation Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons").

 |  | Habitat Pick | Procgen | LangR | All Other |
| --- | --- | --- | --- | --- |
| GEA | 82.5 | 44.0 | 50.0 | 70.5 |
| GEA-Base | 60.5 | 36.1 | 15.5 | 69.5 | 

Table 3: Effect of stage 2 RL training on GEA-Base (7b model). Tasks included in RL training increase their generalization performance. Performance on the remaining tasks slightly increases thanks to continued SFT training.

We seek to comprehensively frame the empirical performance of GEA relative to prior work on our evaluated benchmarks. First, in all benchmarks, we use only images as observations without using any privileged information such as the simulation state or additional observations such as 3D point clouds. Second, we evaluate our single GEA model across all environments, which is referred to as a generalist. Some of the approaches we compare against are trained on data from a single environment, and we refer to those as specialists. In other comparative approaches, the split between train or test data is unclear. For example, Gato [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] is a generalist model like GEA, yet evaluates and trains on some less diverse tasks with their own training datasets, which are not released. [Section F.1](https://arxiv.org/html/2412.08442v1#A6.SS1 "F.1 Additional Baseline Details ‣ Appendix F Further Experimental Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") discusses these connections in detail.

[Table 2](https://arxiv.org/html/2412.08442v1#S4.T2 "In 4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") summarizes the comparative evaluation. GEA excels at manipulation tasks, either exceeding or matching the performance of specialist models. For example, in Meta-World GEA greatly outperforms both specialist and generalist models trained for this task with a $7\%$ absolute increase relative to the best-performing baseline. In CALVIN, GEA outperforms a variety of recent specialist models. GEA also performs closely to the specialist 3D Diffuser Actor method [[35](https://arxiv.org/html/2412.08442v1#bib.bib35)], which uses a manipulation-specific action representation of end-effector key points and uses a depth camera to represent the scene as a 3D feature cloud. GEA only uses the third PoV RGB camera and does not use an action or observation space specific to table-top manipulation. GEA outperforms the baseline in Habitat Pick and closely matches it in Habitat Place despite these baselines being trained with the ground truth simulator state. In Maniskill, the difficult, often occluded, camera view results in low overall success rates, yet GEA outperforms other results that only use IL. However, GEA is outperformed by methods that use RL in this benchmark.

In video game benchmarks, GEA outperforms the specialist model baseline in Procgen. In Atari, GEA outperforms the generalist Gato model [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)]. However, in Atari, GEA is outperformed by Multi-Game DT [[41](https://arxiv.org/html/2412.08442v1#bib.bib41)], which uses offline RL from suboptimal demonstrations. In Atari, GEA does not learn with offline or online RL.

In the BabyAI navigation benchmark, GEA is close in performance to Gato [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] despite GEA using RGB renderings of the top-down view rather than any underlying state information and 100x fewer demonstrations for this environment. In Habitat Nav, GEA underperforms an RL-trained expert. This gap could be due to the context of GEA only consisting of the previous three observations, which could limit its ability in partially observable settings. In UI control, another discrete action task, GEA outperforms GPT-4o [[64](https://arxiv.org/html/2412.08442v1#bib.bib64)] with set of marks [[93](https://arxiv.org/html/2412.08442v1#bib.bib93)] generated by a UI detection model. This demonstrates that GEA benefits from being specialized for interactive decision-making even against a powerful LLM and specialized perception system. Finally, in the discrete control benchmark of the LangR planning task, GEA closely matches the performance of a specialist baseline, which trains on only this task with RL.

 |  | Habitat Pick | CALVIN | Procgen | Android Control | BabyAI |
| GEA-Base | 57.0 | 48.0 | 24.5 | 50.5 | 84.7 |
| Domain Specific | 54.5 | 35.5 | 23.7 | 48.9 | 82.1 |
| Only LLM | 9.5 | 0.0 | 7.6 | 26.4 | 49.4 |
| Only VisEncoder | 34.5 | 13.0 | 24.5 | 28.3 | 70.6 |
| None | 9.0 | 0.0 | 7.4 | 14.1 | 44.4 | 

Table 4: We present results using LLaVA-OneVision-500m as the MLLM base (row 1) as well as training with only domain specific data (row 2). We also present results for a transformer of the same architecture but only the LLM subnet is initialized (row 3), or the visual encoder (row 4), or neither (row 5).

Comparative Benefit of RL to SFT. Training with RL was important to achieving these strong results. [Table 3](https://arxiv.org/html/2412.08442v1#S6.T3 "In 6.1 GEA Generalization Capabilities ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") compares the performance of GEA-Base, which is only trained with SFT on expert demonstrations, and GEA, which is additionally trained with a second stage of RL. The results show that the success rate of GEA greatly increases on Habitat Pick, Procgen, and LangR, which are the environments where RL was used. Furthermore, the average success rate across all the other tasks is not influenced by RL due to the continued joint SFT training.

### 6.2 GEA Training and Model Analysis

In this section, we explore the relationship between the generalist capabilities of GEA and its training data. We analyze the role of embodied SFT data for adapting the MLLM for interaction and the importance of online data. We also evaluate the importance of the base MLLM. For all results in this section, we train all models with 32% of the original embodied SFT data and 40k updates to ease the computational burden of the analysis. We also evaluate on the tasks from [Table 2](https://arxiv.org/html/2412.08442v1#S4.T2 "In 4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") with a reduced number of evaluation episodes with around 200 episodes per benchmark. [Section F.2](https://arxiv.org/html/2412.08442v1#A6.SS2 "F.2 Ablation Analysis Setting ‣ Appendix F Further Experimental Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") describes this analysis evaluation and dataset in detail.

Impact of Multi-Domain Data. We evaluate training a generalist model on data from all diverse domains, versus training a model only on data from the particular target domain. Specifically, we train the smaller LLaVA-OneVision-500m model on either data from a single domain (“Domain Specific”) or data across all domains (“GEA-Base ”). Comparing these options in [Table 4](https://arxiv.org/html/2412.08442v1#S6.T4 "In 6.1 GEA Generalization Capabilities ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") shows that across all the benchmarks, training with all the data is beneficial. However, the gain is smaller in some of the domains like Android Control and Procgen, likely due to less overlap with the other training domains as opposed to the wealth of manipulation data we train with. [Section G.4](https://arxiv.org/html/2412.08442v1#A7.SS4 "G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") contains a more granular investigation into the pairwise transferability between each dataset and also supports this conclusion that GEA benefits from multi-domain data.

![Refer to caption](img/e29fdab2d3d8efc3269423e39a4ed900.png)

Figure 4: Online learning in Habitat Pick. MLLM methods finetune LLaVA-OV while other methods finetune GEA-Base.

Impact of Policy-Collected Data. Next, we explore the role of learning from data sources beyond SFT on expert demonstrations. While GEA-Base is a capable embodied policy, it is only trained on successful demonstrations. These demonstrations rarely exhibit recovery behaviors or robustness to non-expert behaviors. Unlike typical MLLM applications such as visual question answering, in interactive tasks, agents trained with expert data can suffer from the problem of “covariate shift” where small agent errors cause the observation distribution to shift from the expert’s and for errors to compound  [[70](https://arxiv.org/html/2412.08442v1#bib.bib70)].

We analyze how GEA-Base can be trained with additional data to improve performance in the Habitat Pick task and compare the following alternatives. First, GEA-Base Success SFT collects 10k successful examples in the environment with the GEA-Base policy and then trains on these successes with supervised learning. GEA-Base Offline RL collects 10k trajectories consisting of both successes and failures, both labeled with the dense Habitat Pick reward, and then trains on these with the IQL offline-RL algorithm [[38](https://arxiv.org/html/2412.08442v1#bib.bib38)]. *GEA Online RL* finetunes GEA-Base with PPO, which leverages online interactions with the simulator like the GEA Stage-2 training (but omits the joint SFT loss). We again use the smaller base LLaVA-OneVision-500m model for these experiments. [Figure 4](https://arxiv.org/html/2412.08442v1#S6.F4 "In 6.2 GEA Training and Model Analysis ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") shows the results of these variations.

A main takeaway message is the strong impact of online RL on top of a finetuned MLLM, which increases the success of GEA-Base from $57\%$ to $83\%$, despite the latter being trained on 50k successful Habitat Pick demonstrations. Online RL outperforms both Success SFT and IQL offline RL, highlighting the need for online interactions. It is worth noting that applying success SFT and offline RL on top of GEA-Base decreases the model’s performance, which could be due to the lack of diverse data.

These results further show that online RL is beneficial when applied on top of a finetuned model with domain data. Online RL alone is unable to bring the performance of the base MLLM to GEA-Base without the stage 1 SFT. [Section G.2](https://arxiv.org/html/2412.08442v1#A7.SS2 "G.2 Habitat Pick RL Finetuning ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") explores this further and shows that GEA-Base is also far more sample efficient with RL than the base MLLM. This analysis demonstrates it is important for the GEA model to use RL in combination with SFT.

![Refer to caption](img/9e12197e6b99ad96e4c3e11dd1dc5a1b.png)

Figure 5: Analyzing the impact of training GEA with different base MLLMs with different parameter counts.

Impact of pretrained MLLM. We assess the importance of the pretrained MLLM in the GEA architecture. To do so, we present results using a base model that has an architecture identical to the pretrained MLLM. However, instead of initializing the full model with LLaVA-OneVision we initialize only the LLM or the visual encoder with the corresponding subnet weights.

The results in [Table 4](https://arxiv.org/html/2412.08442v1#S6.T4 "In 6.1 GEA Generalization Capabilities ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") show that the full MLLM has a substantial impact on performance. Although this isn’t surprising, it is important to note that the visual encoder initialization seems to have a stronger impact on the final performance compared to the LLM. We conjecture that this is because the benchmarks require visual generalization, and training the LLaVA-OneVision SigLIP visual encoder from scratch with only the embodied data is challenging.

Further, we analyze the impact of the base MLLM size on the GEA-Base. We use two classes of backbone models, LLaVA-OneVision [[44](https://arxiv.org/html/2412.08442v1#bib.bib44)] and MM1.5 [[58](https://arxiv.org/html/2412.08442v1#bib.bib58)], and use two sizes for each of them ($0.5B$ and $7B$ for the former, and $1.2B$ and $6.4B$ for the latter). As shown in [Figure 5](https://arxiv.org/html/2412.08442v1#S6.F5 "In 6.2 GEA Training and Model Analysis ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), in our agentic applications increasing model capacity leads to stronger performance, both within and also across backbone model class. Interestingly, the class of models has little effect as we see very similar performance across multiple backbone models. The three different $7B$ models have been trained on different web-scale data. Nevertheless, their difference has a negligible impact on GEA.

Additionally, in [Section G.3](https://arxiv.org/html/2412.08442v1#A7.SS3 "G.3 Model Variance Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), we show that GEA training and evaluation is robust to the random seed.

## 7 Conclusion

In this work, we studied how finetuning pretrained MLLMs with large-scale embodied experience via expert trajectories and online RL unlocks their ability to act as Generalist Embodied Agent. To interface with diverse embodiments, GEA uses a learned action tokenizer. We illustrate the importance of RL finetuning for GEA, which results in competitive results across a variety of domains spanning manipulation, video games, navigation, UI control, and planning.

While GEA demonstrates impressive abilities across a wide diversity of tasks, it is still not at the level of a foundation model for decision-making like similar models for language and vision [[1](https://arxiv.org/html/2412.08442v1#bib.bib1)]. GEA cannot control arbitrary embodiments and operate in arbitrary environments zero-shot. Furthermore, the performance of GEA in some domains, such as Maniskill, Atari, and AndroidControl, is far from perfect. Extending RL to these environments could be a solution. Future work can continue to scale GEA to more tasks to extend its generalist capabilities.

## References

*   Achiam et al. [2023] Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, et al. Gpt-4 technical report. *arXiv preprint arXiv:2303.08774*, 2023.
*   Agarwal et al. [2020] Rishabh Agarwal, Dale Schuurmans, and Mohammad Norouzi. An optimistic perspective on offline reinforcement learning. In *International conference on machine learning*, pages 104–114\. PMLR, 2020.
*   Ahn et al. [2022] Michael Ahn, Anthony Brohan, Noah Brown, Yevgen Chebotar, Omar Cortes, Byron David, Chelsea Finn, Chuyuan Fu, Keerthana Gopalakrishnan, Karol Hausman, et al. Do as i can, not as i say: Grounding language in robotic affordances. *arXiv preprint arXiv:2204.01691*, 2022.
*   Bai et al. [2024] Hao Bai, Yifei Zhou, Mert Cemri, Jiayi Pan, Alane Suhr, Sergey Levine, and Aviral Kumar. Digirl: Training in-the-wild device-control agents with autonomous reinforcement learning. *arXiv preprint arXiv:2406.11896*, 2024.
*   Bai et al. [2023] Jinze Bai, Shuai Bai, Shusheng Yang, Shijie Wang, Sinan Tan, Peng Wang, Junyang Lin, Chang Zhou, and Jingren Zhou. Qwen-vl: A frontier large vision-language model with versatile abilities. *arXiv preprint arXiv:2308.12966*, 2023.
*   Bellemare et al. [2013] M. G. Bellemare, Y. Naddaf, J. Veness, and M. Bowling. The arcade learning environment: An evaluation platform for general agents. *Journal of Artificial Intelligence Research*, 47:253–279, 2013.
*   Bellemare et al. [2013] Marc G Bellemare, Yavar Naddaf, Joel Veness, and Michael Bowling. The arcade learning environment: An evaluation platform for general agents. *Journal of Artificial Intelligence Research*, 47:253–279, 2013.
*   Bellman [1957] Richard Bellman. A markovian decision process. *Indiana Univ. Math. J.*, 6:679–684, 1957.
*   Bousmalis et al. [2023] Konstantinos Bousmalis, Giulia Vezzani, Dushyant Rao, Coline Devin, Alex X Lee, Maria Bauza, Todor Davchev, Yuxiang Zhou, Agrim Gupta, Akhil Raju, et al. Robocat: A self-improving foundation agent for robotic manipulation. *arXiv preprint arXiv:2306.11706*, 2023.
*   Brohan et al. [2022] Anthony Brohan, Noah Brown, Justice Carbajal, Yevgen Chebotar, Joseph Dabis, Chelsea Finn, Keerthana Gopalakrishnan, Karol Hausman, Alex Herzog, Jasmine Hsu, et al. Rt-1: Robotics transformer for real-world control at scale. *arXiv preprint arXiv:2212.06817*, 2022.
*   Brohan et al. [2023] Anthony Brohan, Noah Brown, Justice Carbajal, Yevgen Chebotar, Xi Chen, Krzysztof Choromanski, Tianli Ding, Danny Driess, Avinava Dubey, Chelsea Finn, et al. Rt-2: Vision-language-action models transfer web knowledge to robotic control. *arXiv preprint arXiv:2307.15818*, 2023.
*   Castro et al. [2018] Pablo Samuel Castro, Subhodeep Moitra, Carles Gelada, Saurabh Kumar, and Marc G. Bellemare. Dopamine: A Research Framework for Deep Reinforcement Learning. 2018.
*   Chen et al. [2021] Lili Chen, Kevin Lu, Aravind Rajeswaran, Kimin Lee, Aditya Grover, Misha Laskin, Pieter Abbeel, Aravind Srinivas, and Igor Mordatch. Decision transformer: Reinforcement learning via sequence modeling. *Advances in neural information processing systems*, 34:15084–15097, 2021.
*   Chevalier-Boisvert et al. [2019] Maxime Chevalier-Boisvert, Dzmitry Bahdanau, Salem Lahlou, Lucas Willems, Chitwan Saharia, Thien Huu Nguyen, and Yoshua Bengio. BabyAI: First steps towards grounded language learning with a human in the loop. In *ICLR*, 2019.
*   Cobbe et al. [2020] Karl Cobbe, Chris Hesse, Jacob Hilton, and John Schulman. Leveraging procedural generation to benchmark reinforcement learning. In *International conference on machine learning*, pages 2048–2056\. PMLR, 2020.
*   Dai et al. [2023] Wenliang Dai, Junnan Li, Dongxu Li, Anthony Meng Huat Tiong, Junqi Zhao, Weisheng Wang, Boyang Li, Pascale Fung, and Steven Hoi. Instructblip: Towards general-purpose vision-language models with instruction tuning, 2023.
*   Doshi et al. [2024] Ria Doshi, Homer Walke, Oier Mees, Sudeep Dasari, and Sergey Levine. Scaling cross-embodied learning: One policy for manipulation, navigation, locomotion and aviation. *arXiv preprint arXiv:2408.11812*, 2024.
*   Driess et al. [2023] Danny Driess, Fei Xia, Mehdi SM Sajjadi, Corey Lynch, Aakanksha Chowdhery, Brian Ichter, Ayzaan Wahid, Jonathan Tompson, Quan Vuong, Tianhe Yu, et al. PaLM-E: An embodied multimodal language model. *arXiv preprint arXiv:2303.03378*, 2023.
*   Furuta et al. [2023] Hiroki Furuta, Kuang-Huei Lee, Ofir Nachum, Yutaka Matsuo, Aleksandra Faust, Shixiang Shane Gu, and Izzeddin Gur. Multimodal web navigation with instruction-finetuned foundation models. *arXiv preprint arXiv:2305.11854*, 2023.
*   Gallouédec et al. [2024] Quentin Gallouédec, Edward Beeching, Clément Romac, and Emmanuel Dellandréa. Jack of all trades, master of some, a multi-purpose transformer agent. *arXiv preprint arXiv:2402.09844*, 2024.
*   Goyal et al. [2017] Yash Goyal, Tejas Khot, Douglas Summers-Stay, Dhruv Batra, and Devi Parikh. Making the v in vqa matter: Elevating the role of image understanding in visual question answering. In *Proceedings of the IEEE conference on computer vision and pattern recognition*, pages 6904–6913, 2017.
*   Gu et al. [2023] Jiayuan Gu, Fanbo Xiang, Xuanlin Li, Zhan Ling, Xiqiang Liu, Tongzhou Mu, Yihe Tang, Stone Tao, Xinyue Wei, Yunchao Yao, et al. Maniskill2: A unified benchmark for generalizable manipulation skills. *arXiv preprint arXiv:2302.04659*, 2023.
*   Gur et al. [2023] Izzeddin Gur, Hiroki Furuta, Austin Huang, Mustafa Safdari, Yutaka Matsuo, Douglas Eck, and Aleksandra Faust. A real-world webagent with planning, long context understanding, and program synthesis. *arXiv preprint arXiv:2307.12856*, 2023.
*   Haldar et al. [2024] Siddhant Haldar, Zhuoran Peng, and Lerrel Pinto. Baku: An efficient transformer for multi-task policy learning. *arXiv preprint arXiv:2406.07539*, 2024.
*   Hansen et al. [2023] Nicklas Hansen, Hao Su, and Xiaolong Wang. Td-mpc2: Scalable, robust world models for continuous control. *arXiv preprint arXiv:2310.16828*, 2023.
*   Harish et al. [2025] Abhinav Narayan Harish, Larry Heck, Josiah P Hanna, Zsolt Kira, and Andrew Szot. Reinforcement learning via auxiliary task distillation. In *European Conference on Computer Vision*, pages 214–230. Springer, 2025.
*   He et al. [2024] Hongliang He, Wenlin Yao, Kaixin Ma, Wenhao Yu, Yong Dai, Hongming Zhang, Zhenzhong Lan, and Dong Yu. Webvoyager: Building an end-to-end web agent with large multimodal models. *arXiv preprint arXiv:2401.13919*, 2024.
*   Hong et al. [2024] Wenyi Hong, Weihan Wang, Qingsong Lv, Jiazheng Xu, Wenmeng Yu, Junhui Ji, Yan Wang, Zihan Wang, Yuxiao Dong, Ming Ding, et al. Cogagent: A visual language model for gui agents. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 14281–14290, 2024.
*   Hu et al. [2021] Edward J Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, and Weizhu Chen. Lora: Low-rank adaptation of large language models. *arXiv preprint arXiv:2106.09685*, 2021.
*   Huang et al. [2023] Shaohan Huang, Li Dong, Wenhui Wang, Yaru Hao, Saksham Singhal, Shuming Ma, Tengchao Lv, Lei Cui, Owais Khan Mohammed, Barun Patra, Qiang Liu, Kriti Aggarwal, Zewen Chi, Johan Bjorck, Vishrav Chaudhary, Subhojit Som, Xia Song, and Furu Wei. Language is not all you need: Aligning perception with language models, 2023.
*   Huang et al. [2022] Wenlong Huang, Pieter Abbeel, Deepak Pathak, and Igor Mordatch. Language models as zero-shot planners: Extracting actionable knowledge for embodied agents. In *International conference on machine learning*, pages 9118–9147\. PMLR, 2022.
*   Hudson and Manning [2019] Drew A. Hudson and Christopher D. Manning. Gqa: a new dataset for compositional question answering over real-world images. In *CVPR*, 2019.
*   IDEFICS [2023] IDEFICS. Introducing idefics: An open reproduction of state-of-the-art visual language model. [https://huggingface.co/blog/idefics](https://huggingface.co/blog/idefics), 2023.
*   Jiang et al. [2022] Yunfan Jiang, Agrim Gupta, Zichen Zhang, Guanzhi Wang, Yongqiang Dou, Yanjun Chen, Li Fei-Fei, Anima Anandkumar, Yuke Zhu, and Linxi Fan. Vima: General robot manipulation with multimodal prompts. *arXiv preprint arXiv:2210.03094*, 2022.
*   Ke et al. [2024] Tsung-Wei Ke, Nikolaos Gkanatsios, and Katerina Fragkiadaki. 3d diffuser actor: Policy diffusion with 3d scene representations. *arXiv preprint arXiv:2402.10885*, 2024.
*   Kim et al. [2024] Moo Jin Kim, Karl Pertsch, Siddharth Karamcheti, Ted Xiao, Ashwin Balakrishna, Suraj Nair, Rafael Rafailov, Ethan Foster, Grace Lam, Pannag Sanketi, et al. Openvla: An open-source vision-language-action model. *arXiv preprint arXiv:2406.09246*, 2024.
*   Koh et al. [2024] Jing Yu Koh, Robert Lo, Lawrence Jang, Vikram Duvvur, Ming Chong Lim, Po-Yu Huang, Graham Neubig, Shuyan Zhou, Ruslan Salakhutdinov, and Daniel Fried. Visualwebarena: Evaluating multimodal agents on realistic visual web tasks. *arXiv preprint arXiv:2401.13649*, 2024.
*   Kostrikov et al. [2021] Ilya Kostrikov, Ashvin Nair, and Sergey Levine. Offline reinforcement learning with implicit q-learning. *arXiv preprint arXiv:2110.06169*, 2021.
*   Kumar et al. [2024] Aviral Kumar, Vincent Zhuang, Rishabh Agarwal, Yi Su, John D Co-Reyes, Avi Singh, Kate Baumli, Shariq Iqbal, Colton Bishop, Rebecca Roelofs, et al. Training language models to self-correct via reinforcement learning. *arXiv preprint arXiv:2409.12917*, 2024.
*   Lee et al. [2022a] Doyup Lee, Chiheon Kim, Saehoon Kim, Minsu Cho, and Wook-Shin Han. Autoregressive image generation using residual quantization. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 11523–11532, 2022a.
*   Lee et al. [2022b] Kuang-Huei Lee, Ofir Nachum, Mengjiao Yang, Lisa Lee, Daniel Freeman, Winnie Xu, Sergio Guadarrama, Ian Fischer, Eric Jang, Henryk Michalewski, et al. Multi-game decision transformers. *arXiv preprint arXiv:2205.15241*, 2022b.
*   Li et al. [2023a] Bo Li, Yuanhan Zhang, Liangyu Chen, Jinghao Wang, Fanyi Pu, Jingkang Yang, Chunyuan Li, and Ziwei Liu. Mimic-it: Multi-modal in-context instruction tuning. *arXiv preprint arXiv:2306.05425*, 2023a.
*   Li et al. [2023b] Bo Li, Yuanhan Zhang, Liangyu Chen, Jinghao Wang, Jingkang Yang, and Ziwei Liu. Otter: A multi-modal model with in-context instruction tuning. *arXiv preprint arXiv:2305.03726*, 2023b.
*   Li et al. [2024a] Bo Li, Yuanhan Zhang, Dong Guo, Renrui Zhang, Feng Li, Hao Zhang, Kaichen Zhang, Yanwei Li, Ziwei Liu, and Chunyuan Li. Llava-onevision: Easy visual task transfer. *arXiv preprint arXiv:2408.03326*, 2024a.
*   Li et al. [2023c] Chunyuan Li, Zhe Gan, Zhengyuan Yang, Jianwei Yang, Linjie Li, Lijuan Wang, and Jianfeng Gao. Multimodal foundation models: From specialists to general-purpose assistants. *arXiv preprint arXiv:2309.10020*, 2023c.
*   Li et al. [2023d] Junnan Li, Dongxu Li, Silvio Savarese, and Steven Hoi. Blip-2: Bootstrapping language-image pre-training with frozen image encoders and large language models, 2023d.
*   Li et al. [2024b] Wei Li, William Bishop, Alice Li, Chris Rawles, Folawiyo Campbell-Ajala, Divya Tyamagundlu, and Oriana Riva. On the effects of data scale on computer control agents. *arXiv preprint arXiv:2406.03679*, 2024b.
*   Li et al. [2023e] Xinghang Li, Minghuan Liu, Hanbo Zhang, Cunjun Yu, Jie Xu, Hongtao Wu, Chilam Cheang, Ya Jing, Weinan Zhang, Huaping Liu, et al. Vision-language foundation models as effective robot imitators. *arXiv preprint arXiv:2311.01378*, 2023e.
*   Li et al. [2024c] Zhangheng Li, Keen You, Haotian Zhang, Di Feng, Harsh Agrawal, Xiujun Li, Mohana Prasad Sathya Moorthy, Jeff Nichols, Yinfei Yang, and Zhe Gan. Ferret-ui 2: Mastering universal user interface understanding across platforms. *arXiv preprint arXiv:2410.18967*, 2024c.
*   Liang et al. [2023] Jacky Liang, Wenlong Huang, Fei Xia, Peng Xu, Karol Hausman, Brian Ichter, Pete Florence, and Andy Zeng. Code as policies: Language model programs for embodied control. In *2023 IEEE International Conference on Robotics and Automation (ICRA)*, pages 9493–9500\. IEEE, 2023.
*   Liu et al. [2023] Haotian Liu, Chunyuan Li, Qingyang Wu, and Yong Jae Lee. Visual instruction tuning, 2023.
*   Liu et al. [2024] Haotian Liu, Chunyuan Li, Yuheng Li, and Yong Jae Lee. Improved baselines with visual instruction tuning. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 26296–26306, 2024.
*   Loshchilov and Hutter [2017] Ilya Loshchilov and Frank Hutter. Decoupled weight decay regularization. *arXiv preprint arXiv:1711.05101*, 2017.
*   Luo et al. [2024] Liangchen Luo, Yinxiao Liu, Rosanne Liu, Samrat Phatale, Harsh Lara, Yunxuan Li, Lei Shu, Yun Zhu, Lei Meng, Jiao Sun, et al. Improve mathematical reasoning in language models by automated process supervision. *arXiv preprint arXiv:2406.06592*, 2024.
*   Ma et al. [2023] Yecheng Jason Ma, William Liang, Guanzhi Wang, De-An Huang, Osbert Bastani, Dinesh Jayaraman, Yuke Zhu, Linxi Fan, and Anima Anandkumar. Eureka: Human-level reward design via coding large language models. *arXiv preprint arXiv:2310.12931*, 2023.
*   Ma et al. [2024] Yecheng Jason Ma, William Liang, Hung-Ju Wang, Sam Wang, Yuke Zhu, Linxi Fan, Osbert Bastani, and Dinesh Jayaraman. Dreureka: Language model guided sim-to-real transfer. *arXiv preprint arXiv:2406.01967*, 2024.
*   Marino et al. [2019] Kenneth Marino, Mohammad Rastegari, Ali Farhadi, and Roozbeh Mottaghi. Ok-vqa: A visual question answering benchmark requiring external knowledge. In *Proceedings of the IEEE/cvf conference on computer vision and pattern recognition*, pages 3195–3204, 2019.
*   McKinzie et al. [2024] Brandon McKinzie, Zhe Gan, Jean-Philippe Fauconnier, Sam Dodge, Bowen Zhang, Philipp Dufter, Dhruti Shah, Xianzhi Du, Futang Peng, Floris Weers, et al. Mm1: Methods, analysis & insights from multimodal llm pre-training. *arXiv preprint arXiv:2403.09611*, 2024.
*   Mediratta et al. [2023] Ishita Mediratta, Qingfei You, Minqi Jiang, and Roberta Raileanu. The generalization gap in offline reinforcement learning. *arXiv preprint arXiv:2312.05742*, 2023.
*   Mees et al. [2022] Oier Mees, Lukas Hermann, Erick Rosete-Beas, and Wolfram Burgard. Calvin: A benchmark for language-conditioned policy learning for long-horizon robot manipulation tasks. *IEEE Robotics and Automation Letters*, 7(3):7327–7334, 2022.
*   Mnih et al. [2013] Volodymyr Mnih, Koray Kavukcuoglu, David Silver, Alex Graves, Ioannis Antonoglou, Daan Wierstra, and Martin Riedmiller. Playing atari with deep reinforcement learning. *arXiv preprint arXiv:1312.5602*, 2013.
*   Nakano et al. [2021] Reiichiro Nakano, Jacob Hilton, Suchir Balaji, Jeff Wu, Long Ouyang, Christina Kim, Christopher Hesse, Shantanu Jain, Vineet Kosaraju, William Saunders, et al. Webgpt: Browser-assisted question-answering with human feedback. *arXiv preprint arXiv:2112.09332*, 2021.
*   O’Neill et al. [2023] Abby O’Neill, Abdul Rehman, Abhinav Gupta, Abhiram Maddukuri, Abhishek Gupta, Abhishek Padalkar, Abraham Lee, Acorn Pooley, Agrim Gupta, Ajay Mandlekar, et al. Open x-embodiment: Robotic learning datasets and rt-x models. *arXiv preprint arXiv:2310.08864*, 2023.
*   OpenAI [2024] OpenAI. Gpt-4o system card. *arXiv preprint arXiv:2410.21276*, 2024.
*   Patel and Song [2024] Austin Patel and Shuran Song. Get-zero: Graph embodiment transformer for zero-shot embodiment generalization. *arXiv preprint arXiv:2407.15002*, 2024.
*   Peng et al. [2023] Zhiliang Peng, Wenhui Wang, Li Dong, Yaru Hao, Shaohan Huang, Shuming Ma, and Furu Wei. Kosmos-2: Grounding multimodal large language models to the world. *arXiv preprint arXiv:2306.14824*, 2023.
*   Rajeswaran et al. [2017] Aravind Rajeswaran, Vikash Kumar, Abhishek Gupta, Giulia Vezzani, John Schulman, Emanuel Todorov, and Sergey Levine. Learning complex dexterous manipulation with deep reinforcement learning and demonstrations. *arXiv preprint arXiv:1709.10087*, 2017.
*   Rawles et al. [2024] Christopher Rawles, Alice Li, Daniel Rodriguez, Oriana Riva, and Timothy Lillicrap. Androidinthewild: A large-scale dataset for android device control. *Advances in Neural Information Processing Systems*, 36, 2024.
*   Reed et al. [2022] Scott Reed, Konrad Zolna, Emilio Parisotto, Sergio Gomez Colmenarejo, Alexander Novikov, Gabriel Barth-Maron, Mai Gimenez, Yury Sulsky, Jackie Kay, Jost Tobias Springenberg, et al. A generalist agent. *arXiv preprint arXiv:2205.06175*, 2022.
*   Ross et al. [2011] Stéphane Ross, Geoffrey Gordon, and Drew Bagnell. A reduction of imitation learning and structured prediction to no-regret online learning. In *Proceedings of the fourteenth international conference on artificial intelligence and statistics*, pages 627–635\. JMLR Workshop and Conference Proceedings, 2011.
*   Savva et al. [2019] Manolis Savva, Abhishek Kadian, Oleksandr Maksymets, Yili Zhao, Erik Wijmans, Bhavana Jain, Julian Straub, Jia Liu, Vladlen Koltun, Jitendra Malik, Devi Parikh, and Dhruv Batra. Habitat: A Platform for Embodied AI Research. *ICCV*, 2019.
*   Schulman et al. [2017] John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, and Oleg Klimov. Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*, 2017.
*   Schwenk et al. [2022] Dustin Schwenk, Apoorv Khandelwal, Christopher Clark, Kenneth Marino, and Roozbeh Mottaghi. A-okvqa: A benchmark for visual question answering using world knowledge. In *European conference on computer vision*, pages 146–162. Springer, 2022.
*   Setlur et al. [2024] Amrith Setlur, Saurabh Garg, Xinyang Geng, Naman Garg, Virginia Smith, and Aviral Kumar. Rl on incorrect synthetic data scales the efficiency of llm math reasoning by eight-fold. *arXiv preprint arXiv:2406.14532*, 2024.
*   Shaw et al. [2023] Peter Shaw, Mandar Joshi, James Cohan, Jonathan Berant, Panupong Pasupat, Hexiang Hu, Urvashi Khandelwal, Kenton Lee, and Kristina N Toutanova. From pixels to ui actions: Learning to follow instructions via graphical user interfaces. *NeurIPS*, 2023.
*   Shi et al. [2023] Ruizhe Shi, Yuyao Liu, Yanjie Ze, Simon S Du, and Huazhe Xu. Unleashing the power of pre-trained language models for offline reinforcement learning. *arXiv preprint arXiv:2310.20587*, 2023.
*   Singh et al. [2023] Avi Singh, John D Co-Reyes, Rishabh Agarwal, Ankesh Anand, Piyush Patil, Xavier Garcia, Peter J Liu, James Harrison, Jaehoon Lee, Kelvin Xu, et al. Beyond human data: Scaling self-training for problem-solving with language models. *arXiv preprint arXiv:2312.06585*, 2023.
*   Szot et al. [2021] Andrew Szot, Alexander Clegg, Eric Undersander, Erik Wijmans, Yili Zhao, John Turner, Noah Maestre, Mustafa Mukadam, Devendra Singh Chaplot, Oleksandr Maksymets, et al. Habitat 2.0: Training home assistants to rearrange their habitat. *Advances in Neural Information Processing Systems*, 34, 2021.
*   Szot et al. [2022] Andrew Szot, Karmesh Yadav, Alex Clegg, Vincent-Pierre Berges, Aaron Gokaslan, Angel Chang, Manolis Savva, Zsolt Kira, and Dhruv Batra. Habitat rearrangement challenge 2022. [https://aihabitat.org/challenge/2022_rearrange](https://aihabitat.org/challenge/2022_rearrange), 2022.
*   Szot et al. [2023] Andrew Szot, Max Schwarzer, Harsh Agrawal, Bogdan Mazoure, Walter Talbott, Katherine Metcalf, Natalie Mackraz, Devon Hjelm, and Alexander Toshev. Large language models as generalizable policies for embodied tasks. *arXiv preprint arXiv:2310.17722*, 2023.
*   Szot et al. [2024] Andrew Szot, Bogdan Mazoure, Harsh Agrawal, Devon Hjelm, Zsolt Kira, and Alexander Toshev. Grounding multimodal large language models in actions. *arXiv preprint arXiv:2406.07904*, 2024.
*   Team et al. [2023] Adaptive Agent Team, Jakob Bauer, Kate Baumli, Satinder Baveja, Feryal Behbahani, Avishkar Bhoopchand, Nathalie Bradley-Schmieg, Michael Chang, Natalie Clay, Adrian Collister, et al. Human-timescale adaptation in an open-ended task space. *arXiv preprint arXiv:2301.07608*, 2023.
*   Team et al. [2024] Octo Model Team, Dibya Ghosh, Homer Walke, Karl Pertsch, Kevin Black, Oier Mees, Sudeep Dasari, Joey Hejna, Tobias Kreiman, Charles Xu, et al. Octo: An open-source generalist robot policy. *arXiv preprint arXiv:2405.12213*, 2024.
*   Van Hasselt et al. [2016] Hado P Van Hasselt, Arthur Guez, Matteo Hessel, Volodymyr Mnih, and David Silver. Learning values across many orders of magnitude. *Advances in neural information processing systems*, 29, 2016.
*   Wang et al. [2023] Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, and Anima Anandkumar. Voyager: An open-ended embodied agent with large language models. *arXiv preprint arXiv:2305.16291*, 2023.
*   Wang et al. [2024] Lirui Wang, Xinlei Chen, Jialiang Zhao, and Kaiming He. Scaling proprioceptive-visual learning with heterogeneous pre-trained transformers. *arXiv preprint arXiv:2409.20537*, 2024.
*   Wei et al. [2021] Jason Wei, Maarten Bosma, Vincent Y Zhao, Kelvin Guu, Adams Wei Yu, Brian Lester, Nan Du, Andrew M Dai, and Quoc V Le. Finetuned language models are zero-shot learners. *arXiv preprint arXiv:2109.01652*, 2021.
*   Wei et al. [2023] Yao Wei, Yanchao Sun, Ruijie Zheng, Sai Vemprala, Rogerio Bonatti, Shuhang Chen, Ratnesh Madaan, Zhongjie Ba, Ashish Kapoor, and Shuang Ma. Is imitation all you need? generalized decision-making with dual-phase training. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*, pages 16221–16231, 2023.
*   Wen et al. [2023] Hao Wen, Yuanchun Li, Guohong Liu, Shanhui Zhao, Tao Yu, Toby Jia-Jun Li, Shiqi Jiang, Yunhao Liu, Yaqin Zhang, and Yunxin Liu. Empowering llm to use smartphone for intelligent task automation. *arXiv e-prints*, pages arXiv–2308, 2023.
*   Wu et al. [2023a] Hongtao Wu, Ya Jing, Chilam Cheang, Guangzeng Chen, Jiafeng Xu, Xinghang Li, Minghuan Liu, Hang Li, and Tao Kong. Unleashing large-scale video generative pre-training for visual robot manipulation. *arXiv preprint arXiv:2312.13139*, 2023a.
*   Wu et al. [2023b] Jimmy Wu, Rika Antonova, Adam Kan, Marion Lepert, Andy Zeng, Shuran Song, Jeannette Bohg, Szymon Rusinkiewicz, and Thomas Funkhouser. Tidybot: Personalized robot assistance with large language models. *Autonomous Robots*, 47(8):1087–1102, 2023b.
*   Xie et al. [2024] Yuxi Xie, Anirudh Goyal, Wenyue Zheng, Min-Yen Kan, Timothy P Lillicrap, Kenji Kawaguchi, and Michael Shieh. Monte carlo tree search boosts reasoning via iterative preference learning. *arXiv preprint arXiv:2405.00451*, 2024.
*   Yang et al. [2023] Jianwei Yang, Hao Zhang, Feng Li, Xueyan Zou, Chunyuan Li, and Jianfeng Gao. Set-of-mark prompting unleashes extraordinary visual grounding in gpt-4v. *arXiv preprint arXiv:2310.11441*, 2023.
*   Yang et al. [2024] Yue Yang, Fan-Yun Sun, Luca Weihs, Eli VanderBilt, Alvaro Herrasti, Winson Han, Jiajun Wu, Nick Haber, Ranjay Krishna, Lingjie Liu, et al. Holodeck: Language guided generation of 3d embodied ai environments. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 16227–16237, 2024.
*   Ye et al. [2023] Qinghao Ye, Haiyang Xu, Guohai Xu, Jiabo Ye, Ming Yan, Yiyang Zhou, Junyang Wang, Anwen Hu, Pengcheng Shi, Yaya Shi, et al. mplug-owl: Modularization empowers large language models with multimodality. *arXiv preprint arXiv:2304.14178*, 2023.
*   Yenamandra et al. [2023] Sriram Yenamandra, Arun Ramachandran, Karmesh Yadav, Austin Wang, Mukul Khanna, Theophile Gervet, Tsung-Yen Yang, Vidhi Jain, Alexander William Clegg, John Turner, et al. Homerobot: Open-vocabulary mobile manipulation. *arXiv preprint arXiv:2306.11565*, 2023.
*   You et al. [2025] Keen You, Haotian Zhang, Eldon Schoop, Floris Weers, Amanda Swearngin, Jeffrey Nichols, Yinfei Yang, and Zhe Gan. Ferret-ui: Grounded mobile ui understanding with multimodal llms. In *European Conference on Computer Vision*, pages 240–255. Springer, 2025.
*   Yu et al. [2019a] Tianhe Yu, Deirdre Quillen, Zhanpeng He, R. Julian, Karol Hausman, Chelsea Finn, and S. Levine. Meta-world: A benchmark and evaluation for multi-task and meta reinforcement learning. In *CoRL*, 2019a.
*   Yu et al. [2019b] Tianhe Yu, Deirdre Quillen, Zhanpeng He, Ryan Julian, Karol Hausman, Chelsea Finn, and Sergey Levine. Meta-world: A benchmark and evaluation for multi-task and meta reinforcement learning. In *Conference on Robot Learning (CoRL)*, 2019b.
*   Zeng et al. [2022] Andy Zeng, Maria Attarian, Brian Ichter, Krzysztof Choromanski, Adrian Wong, Stefan Welker, Federico Tombari, Aveek Purohit, Michael Ryoo, Vikas Sindhwani, et al. Socratic models: Composing zero-shot multimodal reasoning with language. *arXiv preprint arXiv:2204.00598*, 2022.
*   Zhang et al. [2024] Jianguo Zhang, Tian Lan, Rithesh Murthy, Zhiwei Liu, Weiran Yao, Juntao Tan, Thai Hoang, Liangwei Yang, Yihao Feng, Zuxin Liu, et al. Agentohana: Design unified data and training pipeline for effective agent learning. *arXiv preprint arXiv:2402.15506*, 2024.
*   Zheng et al. [2024] Boyuan Zheng, Boyu Gou, Jihyung Kil, Huan Sun, and Yu Su. Gpt-4v (ision) is a generalist web agent, if grounded. *arXiv preprint arXiv:2401.01614*, 2024.
*   Zhu et al. [2023] Deyao Zhu, Jun Chen, Xiaoqian Shen, Xiang Li, and Mohamed Elhoseiny. Minigpt-4: Enhancing vision-language understanding with advanced large language models. *arXiv preprint arXiv:2304.10592*, 2023.

## Appendix A Continuous Multi-Embodiment Tokenizer Details

As stated in [Section 4.2](https://arxiv.org/html/2412.08442v1#S4.SS2 "4.2 Continuous Multi-Embodiment Tokenizer ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), we train a Residual VQ-VAE (RVQ) model to convert continuous actions from diverse robot embodiments into a shared discrete token space. In our experiments, we use 2 codebooks, each with 512 tokens, and the token vector dimension is 1024\. Actions are encoded into these latent codebooks via a 4-layer MLP with 4096 hidden dimensions per layer. The decoder uses the same MLP architecture but now inputs the 1024-dimension latent and outputs the padded action. Refer to Lee et al. [[40](https://arxiv.org/html/2412.08442v1#bib.bib40)] for further details about the RVQ method. We map these 512 tokens for both of the 2 codebooks to tokens $30000-30512$ of the LLM vocabulary. Since these tokens are non-English tokens for all the LLMs we consider and all our tasks use English instructions, we use this same token range for all MLLM experiments. All actions are padded to be 21 dimensions during tokenization. During detokenization, the 21 dimension continuous vector is truncated from the right to fit the expected action dimension for the environment.

The RVQ tokenizer is trained with MSE loss using all datasets with continuous actions from the overall set of datasets used to train GEA described in [Appendix D](https://arxiv.org/html/2412.08442v1#A4 "Appendix D Dataset Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). The commitment loss from the RVQ is weighted by $1.0$ relative to the MSE loss. The tokenizer is trained with a per-GPU batch size of 256 across 8 H100 GPUs. The model is then trained for 15k updates using the AdamW optimizer [[53](https://arxiv.org/html/2412.08442v1#bib.bib53)] with a cosine learning rate decaying to 0 at the end of training and a linear learning rate warmup schedule for the first $10\%$ of training. At this number of updates, the validation MSE loss on unseen actions had largely plateaued at $\approx 0.0038$ averaged across all datasets and $0.002-0.007$ depending on the dataset. This trained RVQ tokenizer was then used to train and evaluate all models with continuous actions.

## Appendix B SFT Training Additional Details

To limit the number of tokens per frame to be processed by the LLM, we use the “video” encoding strategy from LLaVA-OneVision. This does not use the AnyRes technique and also applies bilinear interpolation to reduce the number of tokens per image. This results in 196 tokens for each image after being processed by the visual encoder and downsampled. For the training sequence format described in [Section 3.2](https://arxiv.org/html/2412.08442v1#S3.SS2 "3.2 GEA Architecture ‣ 3 Generalist Embodied Agent ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), we use the following prompt format where per-domain strings are manually defined and substituted into each bracketed statement.

[⬇](data:text/plain;base64,VXNlcjoKQWdlbnQ6IDxhZ2VudCBkZXNjcmlwdGlvbj4uCkFjdGlvbnM6ClNpbXVsYXRvcjogPHNpbXVsYXRpb24gcGxhdGZvcm0+CkNhbWVyYTogPGNhbWVyYSBkZXRhaWxzPgpJbnN0cnVjdGlvbjogPHRhc2sgaW5zdHJ1Y3Rpb24+CgpBZ2VudDo=)User:Agent:  <agent  description>.Actions:Simulator:  <simulation  platform>Camera:  <camera  details>Instruction:  <task  instruction>Agent:

When forming training batches, we randomly sample trajectories, and then randomly sample a span of the appropriate context length from that trajectory, and then use the sampled span of observation and actions as an element of the data batch. In the case of the interactive data, the model is only trained to predict the action tokens; the loss for the prompt, instruction, and image tokens is masked out.

For the details about the datasets used in this training process, see [Appendix D](https://arxiv.org/html/2412.08442v1#A4 "Appendix D Dataset Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons").

## Appendix C RL Training Additional Details

In this section, we list additional RL training details omitted from [Section 4.4](https://arxiv.org/html/2412.08442v1#S4.SS4 "4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). The value function is a 4-layer MLP with a hidden dimension of 2048\. The value function takes as input a single vector from the mean-pooled visual tokens from the visual encoder, the final activation of the LLM for the observation, and any task-specific state information that is available. For LoRA finetuning, we use a value of $r=128,\alpha=32$, and a dropout value of $0.1$. We use GAE return estimation with $\tau=0.95$ and a discount factor of $\gamma=0.999$.

We run RL with the Habitat Pick, LangR, and Procgen environments. Each GPU runs a different benchmark instance. Both Habitat Pick and Procgen are allocated to run on $50\%$ more GPUs than LangR because we found that LangR learns much faster with RL compared to the other tasks. Per benchmark variation, such as different episodes in Habitat Pick and different games in Procgen, are equally divided between the GPUs assigned to that benchmark. On each GPU, a parallelized set of 6 environments are running for batched environment experience collection. We update with 2 PPO epochs, 6 minibatches per epoch, and a clip parameter of $0.2$. We also use the AdamW optimizer for RL training but without any learning rate schedule. We now detail the RL considerations specific to each environment.

In Habitat Pick, we use the same environment details as from prior works using the same task [[81](https://arxiv.org/html/2412.08442v1#bib.bib81), [26](https://arxiv.org/html/2412.08442v1#bib.bib26), [79](https://arxiv.org/html/2412.08442v1#bib.bib79)]. The task requires the agent to pick up a target object specified by name. Specifically, the agent starts within 2 meters of the target object and faces towards the receptacle the target object is on but with random noise $\mathcal{N}(0,1.57)$ applied to the direction of facing directly at the receptacle. The task ends in failure if the agent excessively collides with the scene, drops the object, or picks up the wrong object. The maximum number of steps per episode is 300 steps. We use the same reward as the RL-trained pick skill from Szot et al. [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)] where a dense reward is provided for moving the end-effector closer to the object, a positive bonus for picking up the object, and then negative penalties for collisions. We use the 50k training episodes from Szot et al. [[79](https://arxiv.org/html/2412.08442v1#bib.bib79)]. Note that these training episodes used for RL training are distinct from the episodes used for testing, which are in unseen house layouts.

For the LangR environment, we use the RL environment details from Szot et al. [[80](https://arxiv.org/html/2412.08442v1#bib.bib80)]. The reward function for this environment involves a sparse reward for completing the task, subgoal rewards for completing individual parts of the task, and a slack penalty to encourage completing the task faster. This environment has a maximum of 32 steps per task. In LangR, memory is important because the agent must explore to find certain objects. We therefore increased the number of visual observations in the context to 16 for the LangR RL training. We achieve this by increasing the visual encoder bilinear interpolation factor to produce only 32 visual tokens per image observation. Implementing such environment-specific policy tweaks is simpler in the RL training phase because each GPU worker runs a distinct environment.

For Procgen, we use the RL environment details from Cobbe et al. [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)]. Importantly, we train over all 16 of the Procgen games at once using RL. We use the standard per-game reward functions.

## Appendix D Dataset Details

MetaWorld: We use the Metaworld simulator [[99](https://arxiv.org/html/2412.08442v1#bib.bib99)] with the pre-defined MT-45 split, which consists of 45 training tabletop manipulation tasks using a Sawyer XYZ arm. Each of the 45 tasks has a language instruction associated to it, e.g. ”Open the drawer”, which we use in all experiments. As inputs to the generalist agent, we use RGB observations from the corner3 camera resized to GEA resolution. We use no proprioceptive information.

The dataset consists of 500 trajectories from each of the 45 train tasks. To construct the train, validation and test datasets, we rollout the scripted expert policy 10 times for each task’s starting state until the first success, to obtain sufficient data diversity. We then assign some of the starting states from the 45 training tasks to the validation set and the remaining trajectories to the training dataset.

Atari: We use the DQN replay dataset [[2](https://arxiv.org/html/2412.08442v1#bib.bib2)], which consists of 50 million transitions collected while training the DQN [[61](https://arxiv.org/html/2412.08442v1#bib.bib61)] algorithm on each of the 44 environments separately. Based on the findings of Multi-Game Decision Transformer [[41](https://arxiv.org/html/2412.08442v1#bib.bib41)], we construct the SFT training dataset to be the top-$10\%$ of the DQN replay data by trajectory returns, over 5 random seeds and 50 splits. We do not use $100\%$ of the data since it is suboptimal to include low-return trajectories in the SFT dataset. However, if one would like to run offline RL on the data, they should include all of the available data.

BabyAI: This dataset consists of 50k trajectories split equally between each of the BabyAI tasks from Chevalier-Boisvert et al. [[14](https://arxiv.org/html/2412.08442v1#bib.bib14)]. These trajectories are collected by the shortest path expert provided in the code release for Chevalier-Boisvert et al. [[14](https://arxiv.org/html/2412.08442v1#bib.bib14)]. The observations are top-down RGB renderings of the image at $336\times 336$ resolution.

Procgen: We use the training datasets across all Procgen games provided from Mediratta et al. [[59](https://arxiv.org/html/2412.08442v1#bib.bib59)]. This dataset consists of 10M observation-action transitions between all the games collected by a PPO expert policy that was individually trained on each game. These transitions amount to around 320k trajectories.

CALVIN: We use the standard CALVIN $ABC\rightarrow D$ dataset provided by Mees et al. [[60](https://arxiv.org/html/2412.08442v1#bib.bib60)], which are language-labeled task instructions collected by human teleoperation of the robot. This dataset consists of around 18k demonstrations. The observations are $200\times 200$ RGB renderings from the robot head camera. Note that unlike prior work [[48](https://arxiv.org/html/2412.08442v1#bib.bib48)], we do not use the gripper camera or any robot proprioception from this dataset. The actions in the dataset are 6D end-effector control for the relative orientation and position and the gripper state.

LangR: For this work, we collect 150k demonstrations for each of the 150k unique training episodes defined by Szot et al. [[80](https://arxiv.org/html/2412.08442v1#bib.bib80)]. We collect this data by utilizing the RL-trained policy from [[80](https://arxiv.org/html/2412.08442v1#bib.bib80)]. This policy achieves high performance on the training set of instructions (around $98\%$ success rate), and we collect 1 successful demonstration per training episode. The observations are $336\times 336$ RGB images from the robot head camera. The actions are between 70 high-level skills that include picking up objects by name, navigating to receptacles, placing on receptacles by name, and opening and closing receptacles by name.

Habitat Pick/Place: We collect 50k demonstrations for each of these tasks via an expert policy trained with RL. We use the same setup as from the skill training in Szot et al. [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)] but operate from an object class to pickup rather than the original geometric goal task specification. This expert policy was trained with the ground truth simulator state, consisting of the relative position of the target object to the robot’s end-effector. The expert is trained for 100M steps until convergence. The performance of both experts on the unseen episodes are displayed in [Table 2](https://arxiv.org/html/2412.08442v1#S4.T2 "In 4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") as baselines.

For the online learning experiments from [Section 6.2](https://arxiv.org/html/2412.08442v1#S6.SS2 "6.2 GEA Training and Model Analysis ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), we construct the training dataset as follows. We use the GEA-Base checkpoint to collect 10,000 trajectories with action sampling to allow for suboptimal trajectories to be added to the data. The rollouts policy has a success rate of about $50\%$, meaning that half of the trajectories can be used for SFT from successful trajectories, and all of them can be used for offline RL. When training the SFT policy, we restrict the loss to be optimized only over successful trajectories, as SFT cannot learn from suboptimal data. In addition to observations and actions, we also log the reward values which are required for offline RL.

Habitat Nav: We collect 13k shortest path demonstrations of an agent navigating to receptacles by name in the house. The navigation is performed by an oracle shortest path agent.

Maniskill: These datasets are from the released imitation learning datasets from Gu et al. [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)]. They are generated via a motion planning algorithm. We only utilize the RGB 3rd-person RGB camera. We generate data for the “StackCube”, “PegInsertionSide”, “PlugCharger”, “PushCube”, and “PickCube” tasks. The dataset from Gu et al. [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)] has 1k demonstrations for each of these tasks.

Android Control: This is the dataset from Li et al. [[47](https://arxiv.org/html/2412.08442v1#bib.bib47)] consisting of 14k human demonstrations of using apps to accomplish UI control tasks. Each trajectory has a unique language instruction. The data spans 833 apps. The original images are $1080\times 1920$, corresponding to the phone screen size. Since we do not use any AnyRes techniques in the MLLM visual encoder, we resize the images to be square before inputting them into the MLLM visual encoder. Using AnyRes with image crops to account for these image aspect ratios could improve the performance of GEA. Actions are generally represented as the name of the action type (like “tap”, “scroll”, or “input text”) followed by the argument for that action where applicable. For tap action arguments, we discretize the original $1080\times 1920$ screen into $50\times 50$ patches. A tap action argument is represented as two integers representing the horizontal and vertical patch coordinates where the tap occurred on the screen. All these actions are represented as textual tokens and are separate from the continuous action tokens.

OpenX: The Open X-Embodiment dataset consists of numerous individual datasets spanning different robots. We include the following individual datasets from OpenX: Austin Buds dataset, Austin Sailor dataset, Austin Sirius dataset, Berkeley Cable Routing, CMU Stretch, DLR EDAN Shared Control, Fractal, IAMLab CMU Pickup Insert, Jaco Play, Kuka, UCSD Kitchen Dataset, UTAustin Mutex, Dobbe, FMB, RoboSet and Spoc.

Vision language instruction data: We use the datasets described in [Section 5](https://arxiv.org/html/2412.08442v1#S5 "5 Datasets and Environments ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons").

When sampling batches for any SFT training, we weight each dataset defined in [Table 1](https://arxiv.org/html/2412.08442v1#S4.T1 "In 4.2 Continuous Multi-Embodiment Tokenizer ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") equally, except OpenX is upsampled 2x, Procgen 2x, and all the VQA data is upsampled 3x.

## Appendix E Evaluation Details

Overall, we use the standard evaluation settings per environment as defined by the prior work. We detail the evaluation settings for each environment below:

MetaWorld: We evaluate the generalization performance of our agent on Metaworld unseen starting states over 5 episodes each, totaling to 450 evaluation trajectories in total. We use the simulator’s notion of success to define the success rate.

Atari: We evaluate the in-distribution performance of our model on 44 Atari games, by using the same setup as the DQN replay dataset, which in turn relies on the Dopamine [[12](https://arxiv.org/html/2412.08442v1#bib.bib12)] framework. Specifically, we use the {GameID}NoFramskip-v4 version of the ALE simulator [[6](https://arxiv.org/html/2412.08442v1#bib.bib6)]. We conduct 10 rollouts over all 44 Atari games and average their respective human-normalized scores. The human-normalized scores follow the same protocol as [[41](https://arxiv.org/html/2412.08442v1#bib.bib41), [69](https://arxiv.org/html/2412.08442v1#bib.bib69)]:

|  | $\text{score}_{\text{normalized}}(s)=\frac{&#124;s-\text{score}_{\text{random}}&#124;}{% \text{score}_{\text{human}}-\text{score}_{\text{random}}}$ |  |

Through our experiments, we have found that GEA-Base performs better according to the human-normalized average score than GEA (40.3 vs 32.71). The result is not surprising, as we do not perform any Atari online RL finetuning on the GEA-Base model, and hence, performance can degrade at the expense of much better performance on Habitat Pick and Procgen. To solve this issue, one should co-train on all tasks that support fast online simulation. However, since Atari is structurally similar to Procgen, and Procgen supports procedural level generation to test generalization, we choose not to perform online RL finetuning on Atari.

BabyAI: We evaluate each task from Chevalier-Boisvert et al. [[14](https://arxiv.org/html/2412.08442v1#bib.bib14)] and evaluate over 100 random episodes for each of the 17 tasks, resulting in 1700 total episodes for the numbers reported in [Table 2](https://arxiv.org/html/2412.08442v1#S4.T2 "In 4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). Each episode has a different environment state and new language instruction.

Procgen: We follow the test evaluation setting Mediratta et al. [[59](https://arxiv.org/html/2412.08442v1#bib.bib59)] which uses the “easy” mode setting of the game. We evaluate 50 episodes for each of the 16 games. Like Atari, the performance is evaluated in Procgen via the min-max normalized per-game scores from a random policy and the a PPO expert policy using the reported scores from Cobbe et al. [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)].

CALVIN: We use the $ABC\rightarrow D$ evaluation setting. This means that during our evaluation, the table background and the language instructions are unseen. We report results over the full 1k evaluation episodes defined by Mees et al. [[60](https://arxiv.org/html/2412.08442v1#bib.bib60)].

LangR: We follow the standard evaluation settings from Szot et al. [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)]. Like the original work, our numbers are reported over the 9 unseen language instruction splits. We evaluate in 100 episodes for each of the 9 evaluation splits. Each test episode is also in an unseen house layout.

Habitat Pick/Place/Nav: We follow the evaluation split and settings from Szot et al. [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)] and evaluate the agent for 500 episodes in unseen house layouts. This version of the task where the agent has to operate from a language instruction of which object to pick is a harder version of the original geometric goal task and has been employed by prior works [[26](https://arxiv.org/html/2412.08442v1#bib.bib26), [81](https://arxiv.org/html/2412.08442v1#bib.bib81)].

Maniskill: We use the standard evaluation settings from Gu et al. [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)]. Aligning with the generated dataset for Maniskill, we evaluate in the “StackCube”, “PegInsertionSide”, “PlugCharger”, “PushCube”, and “PickCube” tasks. Each of the 5 tasks are evaluated for 100 episodes each.

Android Control: We evaluate on the full 1,540 test episodes from Li et al. [[47](https://arxiv.org/html/2412.08442v1#bib.bib47)]. We measure the per-step success rate, meaning the percent of the time the agent predicts the right action based on the ground truth test episode. This is distinct from the episode level success rate which requires the agent to predict every action in the trajectory correctly. We report the success rate under the more challenging setting of following high-level instructions only, without per-step low-level instructions. We consider an action as predicted correctly if it is within 5 of the horizontal and vertical patches defined in the dataset generation from [Appendix D](https://arxiv.org/html/2412.08442v1#A4 "Appendix D Dataset Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). Any entered text is considered correct if the correct text is contained in the entered text or the entered text is contained in the correct text.

![Refer to caption](img/8219c98373746ea0d5e981667ae7dcd9.png)

(a) Validation Loss (GEA-Base-500m).

 |  | HabPick | Procgen | CALVIN | BabyAI | Meta-World | Android Control |
| --- | --- | --- | --- | --- | --- | --- |
| Seed 0 | 46.5 | 29.0 | 44.5 | 82.6 | 82.7 | 48.4 |
| Seed 1 | 49.0 | 25.4 | 42.5 | 80.0 | 86.7 | 50.1 |
| Seed 2 | 53.0 | 27.3 | 44.5 | 82.4 | 88.4 | 49.2 |
| Combined | 49.5 $\pm$ 3.3 | 27.2 $\pm$ 1.8 | 43.8 $\pm$ 1.2 | 81.7 $\pm$ 1.5 | 85.9 $\pm$ 3.0 | 49.2 $\pm$ 0.8 | 

(b) Evaluation success rates (GEA-Base-500m).

Figure 6: Variance in training jobs and evaluation for GEA. We train three different random seeds for GEA-Base-500m. The left shows the validation loss during SFT is similar between each random seed. The right shows the resulting online evaluation for these three random seeds across six of the benchmarks, along with the combined averages and standard deviation per benchmark. While the standard deviation is low, it is still a couple of percent on some benchmarks despite the validation loss being very similar.

## Appendix F Further Experimental Details

### F.1 Additional Baseline Details

For each benchmark, to the best of our knowledge, we report the method from prior work with the highest performance. In this section, we add more details about these baselines from prior works in [Table 2](https://arxiv.org/html/2412.08442v1#S4.T2 "In 4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") and any differences in the evaluation settings and method assumptions from GEA. We source methods from prior work that train a single policy over all tasks in a benchmark. We consider methods that only train on tasks from a single benchmark as “specialist” and methods that train across multiple benchmarks as “generalist”. Note that this means we do not compare against methods that train policies on individual tasks from the benchmark. For example, in Procgen, we do not compare to the performance of Cobbe et al. [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)] since a separate policy is trained for each of the 16 tasks. Instead, we only compare to methods that train a single policy across multiple tasks.

Meta-World: Many prior works report performance on the Meta-World benchmark [[24](https://arxiv.org/html/2412.08442v1#bib.bib24)], but fewer works train and evaluate over the full set of 45 tasks. Gato [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] trains over all the Meta-World tasks and reports an average performance of $87.0\%$ success rate. ¹¹1We reference the Gato numbers from Table 8 of the TMLR paper version: https://openreview.net/pdf?id=1ikK0kHjvj Unlike GEA, Gato also takes as input the proprioceptive state in Meta-World. Reed et al. [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] also does not clarify if the Meta-World evaluation is performed over unseen state configurations or using the same states seen during training. GEA is evaluated on unseen state configurations. The GEA Meta-World numbers are reported in the same setting as Szot et al. [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)] with the same inputs.

CALVIN: To the best of our knowledge, there are no generalist agents that report the performance on CALVIN in addition to other benchmarks. RoboFlamingo [[48](https://arxiv.org/html/2412.08442v1#bib.bib48)] also adapts an MLLM for control by finetuning it with supervised learning. We include this specialist agent since it also leverages finetuning MLLMs, to demonstrate the gains from scaling to a generalist model with GEA. Compared to GEA, RoboFlamingo uses the gripper camera, image augmentations during training and a longer context length. 3D Diffuser Actor [[35](https://arxiv.org/html/2412.08442v1#bib.bib35)] is the state-of-the-art specialist system for CALVIN $ABC\rightarrow D$ setting and narrowly outperforms GEA. However, as mentioned in the main text, 3D Diffuser Actor also assumes input to 3D pointclouds features. This method uses the head and gripper RGBD cameras to extra pointclouds of the scene. These pointclouds are then converted into a 3D feature cloud using a pretrained CLIP model. This method also simplifies the problem by compressing longer sequences of actions into end-effector keyposes.

Maniskill: We compare to the numbers using RGBD in Table 2 and 3 of Gu et al. [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)]. Unlike these numbers, GEA uses only RGB inputs and no depth images. These results train a single policy per-task which is technically narrower than our definition of “Specialist Agent” which requires training one policy on all tasks from the benchmark. However, we still include these baselines to situate our Maniskill results. GEA outperforms doing imitation learning with the exact same demonstrations as from Table 2 of Gu et al. [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)]. The superior results of $47.8\%$ success are from Table 3 of Gu et al. [[22](https://arxiv.org/html/2412.08442v1#bib.bib22)], which train with DAPG [[67](https://arxiv.org/html/2412.08442v1#bib.bib67)] and PPO. GEA does not train with RL in this task. Hansen et al. [[25](https://arxiv.org/html/2412.08442v1#bib.bib25)] reports higher success rates in Maniskill2 tasks, but uses the ground truth state information instead of visual observations and trains a single policy per individual Maniskill2.

Habitat Pick: Other methods that report performance on the version of the Habitat Pick task that requires picking from the object name typically achieve low success rates [[26](https://arxiv.org/html/2412.08442v1#bib.bib26), [81](https://arxiv.org/html/2412.08442v1#bib.bib81), [96](https://arxiv.org/html/2412.08442v1#bib.bib96)]. We thus also compare to the expert policy that was used to generate the Habitat Pick training dataset as described in [Appendix D](https://arxiv.org/html/2412.08442v1#A4 "Appendix D Dataset Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). Note that this expert policy was not trained in the evaluation scenes and thus also must generalize to unseen scenes. This expert policy has performance on par with pick skills trained in Habitat that operates from the much stronger assumption of a geometric goal input [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)]. The specialist numbers from [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)] also finetune a MLLM with imitation learning and are evaluated in the exact same setting as GEA.

Habitat Place: We follow the same evaluation criteria as Habitat Pick and compare to the expert policy trained with RL that was used to generate the expert demonstrations as described in [Appendix D](https://arxiv.org/html/2412.08442v1#A4 "Appendix D Dataset Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons").

Procgen: We compare against the BC test numbers from Figure 2 of Mediratta et al. [[59](https://arxiv.org/html/2412.08442v1#bib.bib59)], which use the same datasets as our setup. While Gato [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] also reports numbers in Procgen, we are not able to compare to these numbers because Gato reports performance relative to unknown score of the data collection policy. To the best of our knowledge the score of the data collection policy is not released. Thus, it is unclear how the Gato Procgen performance is normalized according to the standard Procgen normalization scores [[15](https://arxiv.org/html/2412.08442v1#bib.bib15)] rendering a direct comparison impossible.

Atari: For a generalist system, we compare with Gato [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] which as Table 8 shows of Reed et al. [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] shows, achieves $30.9$ normalized score. Multi-Game Decision Transformers [[41](https://arxiv.org/html/2412.08442v1#bib.bib41)] achieves $85$ normalized score in the same scoring setting. This method was trained with offline RL based on conditioning the training on the reward-to-go [[13](https://arxiv.org/html/2412.08442v1#bib.bib13)].

Habitat Nav: We compare against the success rate of the navigation policy from Szot et al. [[78](https://arxiv.org/html/2412.08442v1#bib.bib78)]. This policy is trained with RL on the same training set of episodes and evaluated on the same testing set of episodes as GEA. However, this policy operates on the geometric goal specification of the receptacle rather than the receptacle name. Additionally, this policy also takes an egomotion sensor as input, whereas GEA does not, simplifying the problem.

BabyAI: We compare against Gato [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] which as Table 8 in Reed et al. [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] shows, achieves $93.2$ normalized score. While Reed et al. [[69](https://arxiv.org/html/2412.08442v1#bib.bib69)] does not clarify this detail, presumably the normalization is with respect to a perfect expert policy, so the normalized score is equal to the success rate. Gato trains with far more data than GEA with 4.61M episodes.

AndroidControl: We compare against using a Set-of-Mark prompting with GPT-4o. The Set-of-Mark was implemented using the Ferret-UI model [[49](https://arxiv.org/html/2412.08442v1#bib.bib49)] for UI element detection to generate the marks with GPT-4o to determine the action. GEA and this baseline are evaluated under the same success criteria. We do not compare to the numbers from the original AndroidControl paper [[47](https://arxiv.org/html/2412.08442v1#bib.bib47)] since it uses different evaluation criteria from ours described in [Appendix E](https://arxiv.org/html/2412.08442v1#A5 "Appendix E Evaluation Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") and the code for the evaluation in Li et al. [[47](https://arxiv.org/html/2412.08442v1#bib.bib47)] is not publically released.

LangR: We compare against the state-of-the-art numbers from Szot et al. [[81](https://arxiv.org/html/2412.08442v1#bib.bib81)]. This method was trained with RL over the same set of training episodes as GEA.

### F.2 Ablation Analysis Setting

For the analysis experiments, we used a reduced subset of the total dataset. Specifically, we use the Meta-World, CALVIN, Habitat Pick, BabyAI, Procgen, and AndroidControl datasets. Datasets from the other domains are excluded.

When training methods in the analysis setting, we keep all the same settings as from the main GEA experiments with the hyperparameters described in [Appendix B](https://arxiv.org/html/2412.08442v1#A2 "Appendix B SFT Training Additional Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). However, we reduce the number of updates to 40k and use a global batch size of 256 across 2 nodes of 8 H100 GPUs each. All results in the analysis section are reported in the LLaVA-OneVision-500m setting unless specified otherwise.

## Appendix G Further Results

### G.1 Benchmark Per-Task Success Rate

We breakdown the performance of the GEA model reported in [Table 2](https://arxiv.org/html/2412.08442v1#S4.T2 "In 4.4 Stage 2: Online Reinforcement Learning ‣ 4 Training ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") per individual task for the benchmarks of Meta-World ([Table 5](https://arxiv.org/html/2412.08442v1#A7.T5 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")), CALVIN ([Table 6](https://arxiv.org/html/2412.08442v1#A7.T6 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")), Procgen ([Table 7](https://arxiv.org/html/2412.08442v1#A7.T7 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")), Maniskill ([Table 8](https://arxiv.org/html/2412.08442v1#A7.T8 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")), LangR ([Table 9](https://arxiv.org/html/2412.08442v1#A7.T9 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")), and BabyAI ([Table 10](https://arxiv.org/html/2412.08442v1#A7.T10 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")).

### G.2 Habitat Pick RL Finetuning

Complimenting the results demonstrating the value of online learning from [Figure 4](https://arxiv.org/html/2412.08442v1#S6.F4 "In 6.2 GEA Training and Model Analysis ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), in this section, we compare the RL sample efficiency of GEA-Base versus the base MLLM. [Figure 7](https://arxiv.org/html/2412.08442v1#A7.F7 "In G.2 Habitat Pick RL Finetuning ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") shows that doing RL from the GEA-Base model is far more sample efficient and converges to much higher performance than doing RL on the MLLM model. The GEA-Base model is trained with SFT on demonstrations from the Habitat Pick task, so it is expected that its performance will start higher. However, the MLLM model is never able to make up for the performance gap, even with continued RL finetuning.

![Refer to caption](img/605fd7fc96f317eb9d657e27c121ad67.png)

Figure 7: Success rate on the Habitat Pick task of GEA-Base and the base MLLM when finetuning with online RL. Displayed are success rates on the training dataset used in the RL process.

### G.3 Model Variance Analysis

In this section, we analyze the sensitivity of training GEA-Base-500m with different random seeds. Specifically, we vary the random seed used to initialize the model, all aspects of the algorithm, and dataset sampling. We then train the GEA-Base-500m model in the analysis setting from [Section F.2](https://arxiv.org/html/2412.08442v1#A6.SS2 "F.2 Ablation Analysis Setting ‣ Appendix F Further Experimental Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). The results in [Figure 6](https://arxiv.org/html/2412.08442v1#A5.F6 "In Appendix E Evaluation Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") demonstrate that while the training and validation curves are very similar. The online evaluation performance of GEA-Base-500m does have some variance per random seed within a couple of percentage points.

### G.4 Domain Transfer Analysis

In [Figure 8](https://arxiv.org/html/2412.08442v1#A7.F8 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") we study how performance transfers between domains by training the GEA-Base-500m model on every possible pair of datasets from BabyAI, CALVIN, Habitat Pick, Android Control and Procgen. We train the model in the same analysis setting as from [Section F.2](https://arxiv.org/html/2412.08442v1#A6.SS2 "F.2 Ablation Analysis Setting ‣ Appendix F Further Experimental Details ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"). We compare the performance of the dataset pair to only training on one of the datasets (the same as the “Domain Specific” results from [Table 4](https://arxiv.org/html/2412.08442v1#S6.T4 "In 6.1 GEA Generalization Capabilities ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons")). The results in [Figure 8](https://arxiv.org/html/2412.08442v1#A7.F8 "In G.4 Domain Transfer Analysis ‣ Appendix G Further Results ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons"), with more blue colors for negative transfer and more red colors for positive transfer, show that some domains such as Procgen and CALVIN enormously benefit from data in other domains. Android Control slightly benefits from Procgen, another discrete control task. On the other hand, Habitat Pick and BabyAI have negative transfer from a variety of tasks. Despite this negative transfer, [Table 4](https://arxiv.org/html/2412.08442v1#S6.T4 "In 6.1 GEA Generalization Capabilities ‣ 6 Empirical Evaluation ‣ From Multimodal LLMs to Generalist Embodied Agents: Methods and Lessons") still demonstrates that training on all the data across all these domains improves the performance.

![Refer to caption](img/035477b237cae7a9f44806df7065a10f.png)

Figure 8: Each square represents the success rate of GEA-Base-500m trained with the datasets from the two domains indicated by the column and row name and evaluated on the domain indicated by the column. Success rates are scaled by training on only data from that domain, meaning each column is normalized relative to the diagonal. A more blue color means negative transfer and a more red color means positive transfer.

|  | GEA |
| --- | --- |
| assembly | 86.0 |
| basketball | 100.0 |
| button-press-topdown | 100.0 |
| button-press-topdown-wall | 100.0 |
| button-press | 100.0 |
| button-press-wall | 92.0 |
| coffee-button | 100.0 |
| coffee-pull | 100.0 |
| coffee-push | 100.0 |
| dial-turn | 100.0 |
| disassemble | 88.0 |
| door-close | 100.0 |
| door-open | 100.0 |
| drawer-close | 100.0 |
| drawer-open | 100.0 |
| faucet-open | 100.0 |
| faucet-close | 100.0 |
| hammer | 100.0 |
| handle-press-side | 100.0 |
| handle-press | 100.0 |
| handle-pull-side | 80.0 |
| handle-pull | 100.0 |
| lever-pull | 80.0 |
| peg-insert-side | 100.0 |
| peg-unplug-side | 98.0 |
| pick-out-of-hole | 60.0 |
| pick-place | 92.0 |
| pick-place-wall | 82.0 |
| plate-slide | 100.0 |
| plate-slide-side | 96.0 |
| plate-slide-back | 100.0 |
| plate-slide-back-side | 100.0 |
| push-back | 90.0 |
| push | 100.0 |
| push-wall | 100.0 |
| reach | 60.0 |
| reach-wall | 94.0 |
| shelf-place | 98.0 |
| soccer | 76.0 |
| stick-push | 100.0 |
| stick-pull | 98.0 |
| sweep-into | 90.0 |
| sweep | 100.0 |
| window-open | 100.0 |
| window-close | 100.0 |

Table 5: Meta-World per-task success rate breakdown.

|  | GEA |
| --- | --- |
| turn off led | 87.0 |
| move slider left | 100.0 |
| rotate red block right | 69.0 |
| open drawer | 100.0 |
| rotate red block left | 95.5 |
| push pink block right | 100.0 |
| push blue block right | 65.2 |
| push red block left | 85.7 |
| push pink block left | 87.1 |
| push red block right | 31.0 |
| push blue block left | 88.6 |
| push into drawer | 86.7 |
| rotate pink block left | 100.0 |
| turn on lightbulb | 97.6 |
| rotate pink block right | 93.5 |
| rotate blue block right | 78.6 |
| turn off lightbulb | 97.1 |
| lift blue block table | 100.0 |
| close drawer | 100.0 |
| rotate blue block left | 95.8 |
| move slider right | 95.4 |
| turn on led | 96.4 |
| lift blue block slider | 63.3 |
| lift pink block table | 100.0 |
| lift red block slider | 73.1 |
| lift red block table | 83.3 |
| lift pink block slider | 96.0 |

Table 6: CALVIN per-task success rate breakdown. Note the tasks are not equally represented in the evaluation episodes.

|  | GEA |
| --- | --- |
| bigfish | 43.1 |
| bossfight | 54.2 |
| caveflyer | 27.7 |
| chaser | 54.0 |
| coinrun | 66.0 |
| dodgeball | 3.4 |
| fruitbot | 84.8 |
| heist | 32.0 |
| leaper | 26.0 |
| maze | 58.0 |
| miner | 27.0 |
| climber | 46.2 |
| ninja | 62.0 |
| plunder | 4.5 |
| jumper | 50.0 |

Table 7: Procgen per-game score breakdown.

|  | GEA |
| --- | --- |
| StackCube | 4.0 |
| PegInsertionSide | 0.0 |
| PlugCharger | 0.0 |
| PushCube | 52.0 |
| PickCube | 12.0 |

Table 8: Maniskill per-task score breakdown.

|  | GEA |
| --- | --- |
| rephrasing | 84.0 |
| referring expressions | 16.0 |
| spatial relationships | 0.0 |
| context | 34.0 |
| irrelevant text | 86.0 |
| multiple rearrangements | 82.0 |
| novel objects | 96.0 |
| multiple objects | 0.0 |
| conditional instructions | 52.0 |

Table 9: LangR per-task score breakdown.

|  | GEA |
| --- | --- |
| GoToRedBallGrey | 88.0 |
| GoToRedBall | 97.0 |
| GoToRedBallNoDists | 100.0 |
| GoToObj | 100.0 |
| GoToObjS4 | 100.0 |
| GoToLocalS8N7 | 93.0 |
| GoToRedBlueBall | 92.0 |
| GoToDoor | 100.0 |
| Open | 76.0 |
| OpenRedDoor | 100.0 |
| OpenDoorColor | 100.0 |
| OpenDoorLoc | 80.0 |
| OpenTwoDoors | 100.0 |
| OpenRedBlueDoors | 100.0 |
| Pickup | 41.0 |
| PickupLoc | 88.0 |
| PickupDist | 94.0 |

Table 10: BabyAI per-task score breakdown.

|  | GEA |
| --- | --- |
| Alien | 6.1 |
| Amidar | 0.0 |
| Assault | 86.5 |
| Asterix | 2.3 |
| Atlantis | 0.0 |
| BankHeist | 8.9 |
| BattleZone | 39.2 |
| BeamRider | 3.1 |
| Boxing | 0.0 |
| Breakout | 0.0 |
| Centipede | 50.0 |
| ChopperCommand | 30.2 |
| CrazyClimber | 0.0 |
| DemonAttack | 20.8 |
| DoubleDunk | 300.0 |
| Enduro | 0.0 |
| FishingDerby | 0.0 |
| Freeway | 81.1 |
| Frostbite | 2.7 |
| Gopher | 0.0 |
| Gravitar | 0.0 |
| Hero | 0.0 |
| IceHockey | 0.0 |
| Jamesbond | 0.0 |
| Kangaroo | 38.5 |
| Krull | 373.0 |
| KungFuMaster | 0.0 |
| MsPacman | 27.9 |
| NameThisGame | 0.0 |
| Phoenix | 0.0 |
| Pong | 0.0 |
| Qbert | 0.6 |
| Riverraid | 0.0 |
| RoadRunner | 33.0 |
| Robotank | 307.2 |
| Seaquest | 0.3 |
| SpaceInvaders | 52.7 |
| StarGunner | 1.4 |
| TimePilot | 0.0 |
| UpNDown | 0.0 |
| VideoPinball | 0.0 |
| WizardOfWor | 0.0 |
| YarsRevenge | 0.0 |
| Zaxxon | 0.0 |

Table 11: Atari success rate breakdown.