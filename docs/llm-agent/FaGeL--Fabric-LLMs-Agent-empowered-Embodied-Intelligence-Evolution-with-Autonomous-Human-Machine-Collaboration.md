<!--yml
category: 未分类
date: 2025-01-11 11:42:41
-->

# FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration

> 来源：[https://arxiv.org/html/2412.20297/](https://arxiv.org/html/2412.20297/)

\pdfcolInitStack

tcb@breakable

Jia Liu, Min Chen Jia Liu is with the School of Computer Science and Technology, Huazhong University of Science and Technology, Wuhan, China.Min Chen is with the School of Computer Science and Engineering, South China University of Technology, Guangzhou 510640, China, and also with Pazhou Laboratory, Guangzhou 510640, China (e-mail: minchen@ieee.org).

###### Abstract

Recent advancements in Large Language Models (LLMs) have enhanced the reasoning capabilities of embodied agents, driving progress toward AGI-powered robotics. While LLMs have been applied to tasks like semantic reasoning and task generalization, their potential in open physical space exploration remains underexplored. This paper introduces FaGeL (Fabric aGent empowered by embodied intelligence with LLMs), an embodied agent integrating smart fabric technology for seamless, non-intrusive human-agent interaction. FaGeL autonomously generates tasks using multimodal data from wearable and ambient sensors, refining its behavior based on implicit human feedback in generated text, without explicit ratings or preferences. We also introduce a token-level saliency map to visualize LLM fine-tuning, enhancing the interpretability of token-level alignment. The system leverages dual feedback mechanisms to improve token-level alignment and addresses challenges in non-intrusive human-machine interaction and cognition evolution. Our contributions include FaGeL’s development, the DualCUT algorithm for AI alignment, and experimental validation in cooperative tasks, demonstrating FaGeL’s ability to adapt and evolve autonomously through implicit feedback. In the future, we plan to explore FaGeL’s scalability in dynamic environments and its integration with other AI systems to develop AGI agents that adapt seamlessly to diverse human needs.

###### Index Terms:

Large Language Models (LLMs), Embodied Agent, Fabric Computing, Non-Disturbance Feedback, DualCUT Algorithm.![Refer to caption](img/1aaeef29c4532e738b8e4907aa28bf84.png)

Figure 1: The Fabric Agent Empowered by Embodied Intelligence with LLM (FaGeL) integrates (1) a sensing module empowered by wearable intelligence, equipped with a natural language describer; (2) an inference module composed of task mining, AI alignment, and embodied action decomposition; (3) an interaction module consisting of task execution and user feedback perception; (4) an embodied intelligence evolution module based on token-level AI alignment.

## I Introduction

Recent advancements in Large Language Models (LLMs) have provided powerful reasoning capabilities for embodied agents, enabling dynamic interaction with their environments [[1](https://arxiv.org/html/2412.20297v1#bib.bib1)][[2](https://arxiv.org/html/2412.20297v1#bib.bib2)], bringing new hope for the realization of AGI-empowered robotics [[3](https://arxiv.org/html/2412.20297v1#bib.bib3)].

Numerous representative works leveraging LLM technology to realize robotics-centric physical embodied entities have continuously emerged in recent years. For instance, multimodal Large Language Models (MLLMs), trained on large-scale internet data, can be integrated into end-to-end robotic control systems to achieve semantic reasoning and task generalization abilities [[4](https://arxiv.org/html/2412.20297v1#bib.bib4)][[5](https://arxiv.org/html/2412.20297v1#bib.bib5)][[6](https://arxiv.org/html/2412.20297v1#bib.bib6)]. Additionally, LLMs have been successfully applied to robot control in zero-shot settings, especially for solving challenging planning tasks [[7](https://arxiv.org/html/2412.20297v1#bib.bib7)][[8](https://arxiv.org/html/2412.20297v1#bib.bib8)]. However, these pioneering explorations have not fully realized the potential of LLMs. Most studies on embodied agents primarily focus on understanding and executing tasks within a specified task space in the physical world. As a result, there has been a significant underutilization of LLM technology in task learning and generalization in open physical environments.

Compared to robotics-centric embodied intelligence, there have been some attempts to use LLMs as the control center for autonomous embodied agents in simulated environments. These agents exhibit behaviors such as active collaboration with teammates in games [[9](https://arxiv.org/html/2412.20297v1#bib.bib9)], improving task completion over time [[10](https://arxiv.org/html/2412.20297v1#bib.bib10)], and generating social behaviors in interactive sandbox environments [[2](https://arxiv.org/html/2412.20297v1#bib.bib2)]. However, the evolution of such virtual autonomous agents typically relies on the large amounts of low-level data provided by virtual environments, which creates a significant gap between virtual tasks and real-world tasks. As a result, applying them to the physical world still requires further exploration and experimentation.

Therefore, existing work has limitations in realizing the future vision of AGI, i.e., the ability of embodied agents to understand complex intentions, decompose tasks, and, particularly, autonomously explore natural physical environments and achieve intelligent evolution in open spaces [[3](https://arxiv.org/html/2412.20297v1#bib.bib3)][[11](https://arxiv.org/html/2412.20297v1#bib.bib11)]. To meet higher demands for the future evolution of embodied intelligence, building intelligent agents in the physical world that can understand the environment from perceptual data, autonomously explore, and iteratively optimize based on human feedback is a long-term and unresolved challenge [[12](https://arxiv.org/html/2412.20297v1#bib.bib12)], which mainly faces the following challenges:

*   •

    Non-intrusive Human-Machine Interaction: From a hardware perspective, this requires embodied agents to perceive user states, environmental parameters, and embodied interactions without significantly disturbing the user’s daily life. By enabling natural interaction with the system, the burden on users regarding excessive operational demands is reduced, thereby improving the quality of the user’s experience and interaction efficiency.

*   •

    Implicit-Feedback-based AI Alignment: In terms of algorithms, traditional AI Alignment feedback mechanisms rely on explicit ratings or preference values, such as Reinforcement Learning from Human Feedback (RLHF) [[13](https://arxiv.org/html/2412.20297v1#bib.bib13)], Direct Preference Optimization (DPO) [[14](https://arxiv.org/html/2412.20297v1#bib.bib14)], Rank Responses to align language models with Human Feedback (RRHF) [[15](https://arxiv.org/html/2412.20297v1#bib.bib15)], etc. These explicit forms of feedback are labor-intensive, especially during human-computer interactions, where frequent evaluations disrupt the smooth flow of interaction. Therefore, utilizing implicit feedback, such as contextual states or user activity data, to achieve AI alignment is a challenging task.

*   •

    Explainable AI Training: Considering system reliability, we hope for the internal workings of AI to be presented in an observable manner. To ensure trustworthiness and transparency, the AI’s decision-making processes should be interpretable, allowing developers and users to understand how conclusions are reached. This enhances debugging and improves user trust by making the system’s actions more predictable and accountable. Additionally, explainability aids in the continuous improvement of AI models by providing actionable feedback based on the observed behaviors and outcomes.

To address these challenges, this paper introduces the interdisciplinary technology of fabric computing, which combines material science with AI frontiers [[16](https://arxiv.org/html/2412.20297v1#bib.bib16)]. Smart fabric technology endows traditional textiles with intelligent properties, providing embodied agents with new potentials, especially for long-term and non-intrusive coexistence with humans. Based on smart fabric technology, we can build an embodied agent capable of human high comfortableness guaranteeing, autonomously exploring, human-centric, and naturally interacting, as well as aligning agents’ values with human values.

Smart fabric technology integrates multifunctional sensors (e.g., sound, light, force, heat, magnetic) into textile items (e.g., clothing, sofas, carpets), enabling agents to interact seamlessly with humans, monitor behavior and environmental changes in real-time, and thus improve perception, adaptability, and learning capabilities [[17](https://arxiv.org/html/2412.20297v1#bib.bib17)][[18](https://arxiv.org/html/2412.20297v1#bib.bib18)]. By utilizing large-scale multimodal data in the real world for direct interaction and feedback mechanisms, this approach helps dynamically optimize the agent’s behavior, ensuring its values remain aligned with human needs and intentions.

Incorporating with the smart fabric technology, this paper proposes an embodied agent named FaGeL (Fabric aGent empowered by embodied intelligence with LLMs). FaGeL can explore the user’s need space, autonomously generate collaborative tasks, and adjust its values by capturing subtle behaviors in daily life, without requiring explicit guidance, as shown in Fig. [1](https://arxiv.org/html/2412.20297v1#S0.F1 "Figure 1 ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration"). It utilizes smart fabric to obtain multimodal data such as body temperature, heart rate, and respiration, which can be embedded into physical entities like sofas, clothing, and carpets to minimize disruption to the user’s life. FaGeL has the following features: (1) exploring human need spaces; (2) determining its positioning and values from the perspective of human-machine collaboration; (3) autonomously generating tasks and evolving by human interaction feedback.

The contributions of this paper are listed as follows:

1.  1.

    We construct FaGeL, an embodied agent empowered by smart fabric and LLMs, which can continuously collect large-scale wearable and ambient multimodal data. FaGeL leverages autonomous task exploration and human-machine interaction to achieve value alignment and evolution, iteratively adjusting model outputs.

2.  2.

    An experimental platform was built to validate FaGeL’s ability to align its value system with human values using implicit feedback (generated textual summaries, rather than explicit scalar rewards or preferences).

3.  3.

    To the best of our knowledge, we are the first to introduce a token-level saliency map during large language model training, which visualizes the internal mechanisms of LLM fine-tuning.

4.  4.

    We observe that both positive and negative bidirectional textual feedback significantly affect token-level alignment. Based on the observation, we propose the DualCUT algorithm, which is an extension of the Contrastive Unlikelihood Training (CUT) alignment model [[19](https://arxiv.org/html/2412.20297v1#bib.bib19)]. The DualCUT algorithm improves the identification of positive tokens and enhances the recognition of negative tokens, thereby improving alignment efficiency.

5.  5.

    To validate the effectiveness of the FaGeL evolution strategy in cooperative tasks, we apply the FaGeL evolution algorithm in the open-source Overcooked-AI environment, allowing the agent to evolve through observation during collaboration with the user. Based on the current SOTA model ProAgent [[9](https://arxiv.org/html/2412.20297v1#bib.bib9)], FaGeL achieved an 11.3% ¹¹1The ProAgent algorithm surpasses its previous state-of-the-art (SOTA) algorithm by 0.51% in the Cramped Room Scenario, which is our test environment. scoring performance improvement in 10 games, relying solely on observation (without human guidance).

The remainder of this paper is structured as follows. Section II reviews related work, focusing on advancements in embodied intelligence, fabric-integrated technologies, and AI alignment. Section III introduces the DualCUT algorithm for intelligence evolution. Section IV describes the experimental setups. Section V presents a comprehensive performance evaluation, highlighting the effectiveness of the evolution module of FaGeL. Finally, Section VI concludes the paper and outlines potential directions for future work.

## II Ralated Work

### II-A Fabric-Integrated Embodied Intelligence

Intelligent fabric technology offers new possibilities for innovative agent design by integrating embodied intelligence, particularly for achieving long-term, comfortable, and non-intrusive coexistence with humans. This technology responds to external stimuli, such as light, temperature, electric fields, and magnetic fields, thereby altering the fabric’s appearance to function as an intelligent medium [[20](https://arxiv.org/html/2412.20297v1#bib.bib20)][[21](https://arxiv.org/html/2412.20297v1#bib.bib21)][[22](https://arxiv.org/html/2412.20297v1#bib.bib22)].

Multifunctional fibers are designed to enable fabrics with responsive capabilities, such as dynamically changing color in response to variations in light and temperature [[23](https://arxiv.org/html/2412.20297v1#bib.bib23)]. For example, TiO2-x and dyes were used to coat cotton textile fibers, enabling reversible color changes under different light conditions [[24](https://arxiv.org/html/2412.20297v1#bib.bib24)]. Reverse Thermal Responsive Fibers (VTF) can undergo color changes in response to temperature shifts, effectively detecting alterations in body temperature [[25](https://arxiv.org/html/2412.20297v1#bib.bib25)]. Additionally, fiber colors can change due to electric fields and pressures [[26](https://arxiv.org/html/2412.20297v1#bib.bib26)], and smart color-changing fabrics have been developed using conductive substrates that alter color with applied electric current [[27](https://arxiv.org/html/2412.20297v1#bib.bib27)].

These interactive modes allow fabrics to engage dynamically with their physical environments, enhancing human-device interactions through natural expressions [[28](https://arxiv.org/html/2412.20297v1#bib.bib28)]. Alterations in fabric color, for instance, can represent environmental changes or convey specific information, providing detailed and safe methods for communication and interaction [[29](https://arxiv.org/html/2412.20297v1#bib.bib29)][[30](https://arxiv.org/html/2412.20297v1#bib.bib30)].

However, the explorations of the full potential of intelligent fabrics in sensing and interaction are still in their infancy. Fabrics integrated into wearable and ambient intelligence systems can offer more comfortable and expansive multimodal avenues for human-computer interaction.

### II-B AI Alignment with Human Feedback

By continuously refining the model’s tasks based on users’ feedback, we can foster an ongoing cycle of evolutionary intelligence in real-world, embodied interactions. Ensuring that the model’s output consistently aligns with human values and preferences—particularly within human-agent interaction scenarios—is central to this process. This iterative refinement is known as AI Alignment [[31](https://arxiv.org/html/2412.20297v1#bib.bib31)].

Conventional approaches often rely on direct user evaluations, including scalar ratings [[32](https://arxiv.org/html/2412.20297v1#bib.bib32)] or comparative judgments, to train reward models through methods like Reinforcement Learning from Human Feedback (RLHF) [[13](https://arxiv.org/html/2412.20297v1#bib.bib13)] and Direct Preference Optimization (DPO) [[14](https://arxiv.org/html/2412.20297v1#bib.bib14)].

However, constantly requesting explicit ratings can be intrusive and burdensome, motivating the exploration of indirect user signals—such as actions or verbal assessments—and textual summaries of user feedback.

![Refer to caption](img/39e17395a08f5647bae2eaf12e090cb2.png)

Figure 2: Architecture and Implementation of the Intelligence Evolution Module:(a) Functional components of FaGeL evolution algorithm; (b) The comparison of ProAgent algorithm and FaGeL evolution algorithm; (c) The advantages illustration of FaGeL; (d) Demonstration of the FaGeL evolution algorithm outperforming ProAgent using the Overcooked-AI platform.

## III FaGeL’s Intelligence Evolution

The evolution module refines the tasks explored by the FaGeL system based on user feedback—collected from various environmental inputs and unified into textual form—to better serve users in subsequent interactions.

To achieve non-intrusive user feedback data collection, we can obtain the user’s positive or negative opinions on the current machine interaction behavior from various forms of feedback, such as user actions or given verbal evaluations, as indirect feedback for guiding alignment. This indirect feedback is better summarized in textual form, such as: “I really liked the personalized tips, like adjusting the room temperature, taking short naps, and switching to lighter workouts—they made me feel more comfortable and well-rested. But staying off screens before bed and using a heart rate monitoring app felt a bit impractical and invasive. A more flexible approach would work better for me.” . Compared to scalar or comparative feedback, textual feedback can contain more guiding information and is expected to perform AI alignment in a fine-grained manner.

Therefore, this chapter will detail the specific methods of human-AI alignment using both positive and negative textual feedback.

### III-A Problem Transformation

Suppose there is a triplet ¡description, machine task, human feedback¿. The description refers to the summary of all current environmental state information, which serves as the input to the large language model (LLM), represented as $x=[x_{1},\ldots,x_{M}]$; the machine task refers to the model’s output based on the state information, denoted as $y=[y_{1},\ldots,y_{N}]$; human feedback refers to the textual description of the feedback provided by the user after the machine task is executed under the current description, denoted as $j=[j_{1},\ldots,j_{Q}]$. These are token sequences of lengths M, N, and Q, respectively. Human feedback contains a detailed analysis of the strengths and weaknesses of the output, which is either directly provided by the user or drafted by the LLM based on the analysis of the current state. Our goal is to use the information provided by $j$ to adjust $y$ to meet the user’s expectations, i.e., to achieve alignment between the ’instruction-response’ pair, which can be expressed as $x\rightarrow y$.

### III-B Potential Solution

To utilize linguistic feedback for AI alignment, the Contrastive Unlikelihood Training (CUT) [[19](https://arxiv.org/html/2412.20297v1#bib.bib19)] provides an interesting approach. CUT uses contrastive learning and fine-tuning to enable LLMs to modify their output errors based on human negative judgments.

CUT finds that, under two different input conditions (with judgment and without judgment), tokens whose generation probabilities change dramatically in the same response are mostly those that are erroneous (compared to other suitable tokens). Therefore, CUT constructs three types of alignment data: Align-P, Align-N, and Misalign, which form two distinct contrastive pairs: ¡Align-N, Misalign¿ and ¡Align-P, Align-N¿, to identify inappropriate tokens. Maximum Likelihood Estimation (MLE) is used to handle the correct content, while Unlikelihood Training (UT) [[33](https://arxiv.org/html/2412.20297v1#bib.bib33)] is applied to deal with inappropriate tokens.

For these three categories of data: Align-P represents when the content generated by the LLM is correct and has received positive feedback; Align-N represents when errors occurred during generation, and judgment provided a detailed description of the errors; Misalign represents when the output contains errors but the feedback is positive. This is a set of data where human feedback and LLM responses are mismatched.

Regarding the two contrastive pairs, in the ¡Align-N, Misalign¿ contrast, during the next token prediction, $p(y_{t}|y_{<t},x,j^{-})>>p(y_{t}|y_{<t},x,j^{+})$. Therefore, inappropriate tokens can be detected using the following criterion:

|  | $U=\{t&#124;p(y_{t}&#124;y_{<t},x,j^{-})-\lambda*p(y_{t}&#124;y_{<t},x,j^{+})>0\}$ |  | (1) |

Subsequently, a dynamic weighting mechanism is used to adjust the weight of the erroneous token, and the following loss function is constructed:

|  | $\displaystyle L_{1}=-\frac{1}{N}\left(\sum_{t\notin U}\log p(y_{t}\mid\mathbf{% y}_{<t},\mathbf{x})\right)$ |  | (2) |
|  | $\displaystyle+\sum_{t\in U}\alpha p(y_{t}\mid\mathbf{y}_{<t},\mathbf{x},% \mathbf{j}^{-})^{\gamma}\log\left(1-p(y_{t}\mid\mathbf{y}_{<t},\mathbf{x})\right)$ |  |

Where $\alpha$ and $\gamma$ are hyperparameters used to control the dynamic weighting term, penalizing the token according to the degree of its error.

In the ¡Align-P, Align-N¿ contrast, the following MLE loss function is used:

|  | $\displaystyle L_{2}=-\frac{\mathbb{1}(x\rightarrow\mathbf{y})}{N}\sum_{t}\log p% (y_{t}\mid\mathbf{y}_{<t},\mathbf{x})$ |  | (3) |
|  | $\displaystyle-\frac{\left(1-\mathbb{1}(x\rightarrow\mathbf{y})\right)}{N}\sum_% {t}\log p(y_{t}\mid\mathbf{y}_{<t},\mathbf{j},\mathbf{x})$ |  |

Where $\mathbb{1}$ is an indicator function that returns 1 if the alignment condition is satisfied. Finally, the total loss is:

|  | $L_{CUT}=L_{1}+L_{2}$ |  | (4) |

### III-C AI Alignment based on DualCUT

The CUT algorithm primarily locates erroneous tokens in the output based on judgments for those tokens and suppresses their generation probabilities using the loss function to achieve alignment. However, feedback from human-machine interactions is not always negative. We find that positive feedback not only helps in locating erroneous tokens with higher precision but also can be used to locate satisfactory token sequences in the output. By using dynamic weighting, we can increase the generation probability of satisfactory tokens. In the experimental section [V-B](https://arxiv.org/html/2412.20297v1#S5.SS2 "V-B Token-Level Saliency Maps within the FaGeL-evo Algorithm ‣ V Performance Evaluation of FaGeL ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration"), we can visually observe the effects of both positive and negative feedback in locating positive and negative tokens.

Therefore, by considering both positive and negative feedback, we propose the DualCUT algorithm to correct the generation probabilities for correct and incorrect tokens, achieving AI alignment.

Specifically, we found that when LLM performs the next token prediction, for those erroneous tokens, both $p(y_{t}|y_{<t},x,j^{-})>>p(y_{t}|y_{<t},x,j^{+})$ and $p(y_{t}|y_{<t},x,j^{-})>>p(y_{t}|y_{<t},x)$ hold. Compared to CUT, the existence of the second comparison term enables further filtering. Additionally, we found some false positives coming from low-probability tokens, which can be further filtered out. Thus, we can detect incorrect tokens using the following criterion:

|  | $\displaystyle U_{-}=\{t&#124;p(y_{t}&#124;y_{<t},x,j^{-})-\lambda_{1}*p(y_{t}&#124;y_{<t},x,j% ^{+})>0\}$ |  | (5) |
|  | $\displaystyle\cap\{t&#124;p(y_{t}&#124;y_{<t},x,j^{-})-\lambda_{2}*p(y_{t}&#124;y_{<t},x)>0\}$ |  |
|  | $\displaystyle\cap\{t&#124;p(y_{t}&#124;y_{<t},x,j^{-})>\sigma_{1}\}$ |  |

Where $\lambda_{1}$ and $\lambda_{2}$ are hyperparameters used to measure the precision of erroneous tokens, and $\sigma_{1}$ is a small value close to 0 to exclude the influence of very low-probability tokens.

Similarly, we identify correct tokens using the following criterion:

|  | $\displaystyle U_{+}=\{t&#124;p(y_{t}&#124;y_{<t},x,j^{+})-\lambda_{3}*p(y_{t}&#124;y_{<t},x,j% ^{-})>0\}$ |  | (6) |
|  | $\displaystyle\cap\{t&#124;p(y_{t}&#124;y_{<t},x,j^{+})-\lambda_{4}*p(y_{t}&#124;y_{<t},x)>0\}$ |  |
|  | $\displaystyle\cap\{t&#124;p(y_{t}&#124;y_{<t},x,j^{+})>\sigma_{2}\}$ |  |

Subsequently, we use the sigmoid function to construct dynamic weighting terms so that the model’s regulation strength is strongly correlated with the significance of token recognition:

|  | $\displaystyle\text{scale}_{-}=\frac{\alpha}{1+e^{-(p(y_{t}&#124;y_{<t},x,j^{-})/p(y% _{t}&#124;y_{<t},x)-\lambda_{2})}}$ |  | (7) |

|  | $\displaystyle\text{scale}_{+}=\frac{\beta}{1+e^{-(p(y_{t}&#124;y_{<t},x,j^{+})/p(y_% {t}&#124;y_{<t},x)-\lambda_{4})}}+1$ |  | (8) |

The “+1” term in Equation [8](https://arxiv.org/html/2412.20297v1#S3.E8 "In III-C AI Alignment based on DualCUT ‣ III FaGeL’s Intelligence Evolution ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration") ensures that the reward strength exceeds that of other tokens not belonging to $U_{+}$ or $U_{-}$. $\alpha$ and $\beta$ are hyperparameters. Therefore, the total loss is:

|  | $\displaystyle L=-\frac{1}{N}\Bigg{(}$ | $\displaystyle\sum_{t\in U_{-}}\text{scale}_{-}*\log(1-p(y_{t}&#124;y_{<t},x))$ |  | (9) |
|  |  | $\displaystyle+\sum_{t\in U_{+}}\text{scale}_{+}*\log p(y_{t}&#124;y_{<t},x)$ |  |
|  |  | $\displaystyle+\sum_{t\notin U_{-}\cup U_{+}}\log p(y_{t}&#124;y_{<t},x)\Bigg{)}$ |  |

![Refer to caption](img/8c99f791b2f2b8097282c926d9a6d9dd.png)

Figure 3: Task space exploration across various daily life scenarios visualized using t-distributed Stochastic Neighbor Embedding (t-SNE). (a) The left shows a semantic visualization of 1000 collaborative tasks generated by the FaGeL’s task mining algorithm. The right provides specific descriptions of three tasks located near each other in the task space, highlighting that semantic similarity is represented by spatial proximity. (b) Each circle represents a task output, with its text encoded by GPT-4 and reduced to a 2D plane using the t-SNE algorithm into a “semantic point” (s-point). This visualization also indicates that tasks generated by multiple agents within the same living environment exhibit certain semantic clustering characteristics.

## IV Experimental Setups

This section will describe the experimental platform built to evaluate autonomous collaborative task mining and intelligence evolution. We first introduce the hardware environment used by FaGeL, the dataset we collected, and the algorithm used to initialize user preference alignment. Then, we present the open-source virtual human-machine collaboration environment Overcooked-AI [[34](https://arxiv.org/html/2412.20297v1#bib.bib34)]. Experiments were conducted on this platform to quantitatively analyze the system’s performance in intelligence evolution, serving as a test for the generalization performance of the system when applied to real or virtual-world scenarios.

### IV-A The Hardware Design for FaGeL

The hardware testing environment in this paper follows the setup described in Wearable 2.0 [[35](https://arxiv.org/html/2412.20297v1#bib.bib35)], where a fabric-based wearable device was constructed to collect user behavior and various physiological indicators. The FaGeL system is capable of real-time collection of physiological data, such as heart rate, blood pressure, and blood oxygen saturation. The perception module in the FaGeL system analyzes this data in real time and generates easy-to-understand health status descriptions through a natural language generator, helping users stay informed about their health condition in a timely manner.

### IV-B DataSet Collection for FaGeL Task Mining

To further validate the system’s task mining capabilities, we have constructed a dataset for Task Mining to meet users’ personalized preference requirements.

We present 1000 task samples output by multi-agent workflows across 120 different scenarios and visualize them using the t-SNE (t-distributed Stochastic Neighbor Embedding) algorithm, as shown in Figure [3](https://arxiv.org/html/2412.20297v1#S3.F3 "Figure 3 ‣ III-C AI Alignment based on DualCUT ‣ III FaGeL’s Intelligence Evolution ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration"). t-SNE is a dimensionality reduction algorithm for high-dimensional data visualization, suitable for representing high-dimensional data in two or three-dimensional space, allowing for an intuitive observation of the data’s distribution and structure. It can be observed that tasks from different living environments exhibit clustering characteristics, and the extracted semantic information shows similarity.

### IV-C User Preference Initialization of FaGeL

To ensure that the task mining results align with the user’s specific preferences, we collect feedback from user ratings and construct preference pairs based on these rating results. We apply the DPO algorithm to fine-tune the Llama3-8b-instruct model [[36](https://arxiv.org/html/2412.20297v1#bib.bib36)]. This process ensures that the task mining results output by the FaGeL system are in accordance with the actual preferences exhibited by users in the survey, thereby enhancing the personalization and practicality of the task mining outcomes.

### IV-D Testing FaGeL on Overcooked-AI, an Open Source Platform

To quantitatively validate FaGeL’s capability in the intelligence revolution algorithm, we used Overcooked-AI [[34](https://arxiv.org/html/2412.20297v1#bib.bib34)], a virtual human-machine collaboration environment, as an additional testbed.

Overcooked-AI is a benchmark environment for fully cooperative human-AI task performance, based on the Overcooked game. The goal of the game is to deliver soups as fast as possible. Each soup requires placing up to three ingredients in a pot, waiting for the soup to cook, and then having an agent pick up the soup and deliver it. The agents must split up tasks on the fly and coordinate effectively in order to achieve high rewards. In this environment, two agents work together by placing ingredients, cooking, and delivering soups in a coordinated manner, emphasizing dynamic task allocation and effective collaboration.

#### IV-D1 Setups on Overcooked-AI

This work mainly focuses on whether the agent can autonomously generate and adjust task generation strategies by observing the environment and user feedback during gameplay. In this study, Player 1 is controlled by the evolution algorithm, while the AI partner acts as Player 2\. During the game, the agent collaborates with the AI partner, and based on the evaluation of the current state, reflects on whether there were reasoning errors in the previous time slice. This reflection generates analysis annotations, which are then optimized using the DualCUT algorithm [III](https://arxiv.org/html/2412.20297v1#S3 "III FaGeL’s Intelligence Evolution ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration") to achieve intelligent evolution throughout the game process.

In this study, we choose ProAgent [[9](https://arxiv.org/html/2412.20297v1#bib.bib9)] as the baseline model. The ProAgent model, a leading example of zero-shot collaborative agents utilizing large language models, has achieved state-of-the-art results on the Overcooked-AI platform. We use the llama3-8b-Instruct [[36](https://arxiv.org/html/2412.20297v1#bib.bib36)] model as the base LLM.

The “Cramped Room” layout is selected for the experiment, as shown in Figure [2](https://arxiv.org/html/2412.20297v1#S2.F2 "Figure 2 ‣ II-B AI Alignment with Human Feedback ‣ II Ralated Work ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration")(d), where the compact room layout increases the likelihood of collisions. In a 400-time-step game, the agent and its AI partner collaborate to place ingredients into a pot according to the recipe, wait for the ingredients to cook, and then serve the dish using a plate to score 20 points.

#### IV-D2 Interaction and Reflection

Compared to ProAgent, the FaGeL-evolution algorithm introduces three additional components: the reflector, annotator, and alignment operator, as shown in Figure [2](https://arxiv.org/html/2412.20297v1#S2.F2 "Figure 2 ‣ II-B AI Alignment with Human Feedback ‣ II Ralated Work ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration")(b). The agent’s reasoning process is placed in a queue of length $N$, which is recorded in the status pool. By comparing the previous state to the present state, the differences between the two contiguous states reveal whether there are any reasoning errors. The annotator is used to provide annotations for erroneous reasoning processes.

## V Performance Evaluation of FaGeL

### V-A Performance Evolution on the Overcooked-AI Platform

This section presents both qualitative and quantitative analysis results of applying the FaGeL-evolution algorithm on the open-source platform Overcooked-AI.

We use the Greedy algorithm provided by the Overcooked-AI platform [[34](https://arxiv.org/html/2412.20297v1#bib.bib34)] as the AI partner. Through observation and reflection , the agent evolves its strategy using the DualCUT algorithm described in Section [III](https://arxiv.org/html/2412.20297v1#S3 "III FaGeL’s Intelligence Evolution ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration") every 1000 timesteps, the alignment model is updated once via DualCUT.

![Refer to caption](img/05b04997e321502c74359c73d28ac68e.png)

Figure 4: Average timesteps per completion with an AI partner as observation time increases. A ‘completion’ represents delivering a fully prepared dish to the customer. The figure shows that within a single game session (lasting 400 timesteps), increasing observation time enables the agent to collaborate more effectively with the AI partner, resulting in shorter completion times.

Figure [4](https://arxiv.org/html/2412.20297v1#S5.F4 "Figure 4 ‣ V-A Performance Evolution on the Overcooked-AI Platform ‣ V Performance Evaluation of FaGeL ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration") demonstrates the evolution of the agent’s performance over time, measured as the average number of timesteps required to complete each task. As observation time increases, the agent gradually reduces the average completion time, indicating improved efficiency in coordination with the AI partner. This trend highlights that increased observation contributes to better decision-making capabilities, thereby enhancing the overall performance in the collaborative task. The graph also compares the FaGeL-evolution algorithm with other baseline approaches, clearly showing its effectiveness in reducing completion time.

![Refer to caption](img/86145481c48fd1ad516c246883da1a40.png)

Figure 5: Real-time scores during one episode in the Cramped Room scenario. The plot illustrates the real-time score accumulation for the ProAgent baseline and the FaGeL-evolution variants (with different numbers of rounds of evolution) during one complete episode.

![Refer to caption](img/a755e7378dd5277cc43518f8967dd17b.png)

Figure 6: An example of the token-level saliency map. a) This section shows the input sample. b) This section presents an example of token-level saliency mapping in the next-word prediction task using the DualCUT algorithm. The figure illustrates how user feedback (positive or negative) influences the model’s learning process. Specifically, for Task 1 (receiving negative feedback), the output of the relevant tokens is suppressed by increasing the loss weight, which is then adjusted through contrastive probability. For Task 3 (receiving positive feedback), the probability of relevant token outputs is increased to align with user preferences. This visualization demonstrates how fine-tuning the model by enhancing the tokens associated with positive feedback and suppressing those with negative feedback effectively aligns token saliency with user preferences, ultimately achieving preference alignment.

![Refer to caption](img/e836c73ada933833db386c46a6b75dac.png)

Figure 7: Comparison of token-level significance between our algorithm (DualCUT) and traditional response-level supervised learning approaches that use pairwise data. Unlike response-level supervision, DualCUT leverages positive and negative semantic information from judgments to guide token selection effectively during the Markov decision process in LLM decoding. This enables more granular token selection by incorporating both positive and negative labels within a single dataset, achieving finer control over the response generation at a token level.

Figure [5](https://arxiv.org/html/2412.20297v1#S5.F5 "Figure 5 ‣ V-A Performance Evolution on the Overcooked-AI Platform ‣ V Performance Evaluation of FaGeL ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration") presents the real-time scoring performance of the agent in a single episode, highlighting the differences between the ProAgent baseline and the FaGeL-evolution variants. The figure shows that the FaGeL-evolution models with increased rounds of evolution perform better in accumulating scores, demonstrating improved learning and adaptability. The inset zooms in on the performance near the maximum score region, where the advantage of additional evolution rounds becomes even more evident. This result further confirms the benefit of continuous evolution in achieving higher collaboration efficiency and faster learning rates.

TABLE I: A case study for the entire autonomous collaboration and embodied intelligence evolution pipeline of FaGeL.

| Fabric Wearable Intelligence: | Multimodal data acquisition using fabric wearable devices, including heart rate, blood pressure, blood oxygen, body temperature, sleeping posture, etc. |
| Natural Language Descriptor for FaGeL: | Monday, Early Morning, in Bedroom; The user showed signs of restlessness, frequently tossing and turning throughout the night. Sleep was fragmented, with only 6 hours achieved, below the recommended 7-9 hours for adults. Heart rate fluctuated between 70 and 95 bpm, aligning with restlessness. Breathing averaged 18 breaths per minute, with increases during movement. Blood oxygen levels stayed around 96-98%, while blood pressure averaged 130/85 mmHg. |
|  | The user primarily slept on their side, shifting to a prone position during periods of tossing. |
|  | The bedroom was quiet with minimal external disturbances, but the room temperature was slightly higher than the optimal sleep range. The device on the nightstand indicated increased body temperature and heart rate. |
| Autonomous and Collaborative Task Mining: | • Life Assistant: Adjust the smart home thermostat to lower the room temperature for optimal sleep and a comfortable environment. • Health Consultant: Advise on power naps for recovery from poor sleep. Suggest 20-30 minute naps in a restful environment to avoid disrupting nighttime sleep. |
|  | • Psychological Counselor: Using a heart rate monitoring app to track heart rate and recommend deep-breathing exercises for relaxation. • Nutritionist: Recommend the user to avoid screens and strenuous activities before bedtime to promote better sleep quality in the future. |
|  | • Educational Consultant: Recommend consistent bedtime routine, screen time reduction, and relaxation techniques to improve sleep quality and minimize restlessness. |
|  | • Fitness Coach: Modify the user’s workout to a lighter, energizing activity like a morning walk or gentle yoga to combat fatigue from fragmented sleep. |
| Embodied Task Execution: | 1\. set_temperature [bedroom, 20] 2\. dim_light [bedroom, 40]
3\. play_audio [bedroom_speakers, ‘‘Consider a 20-30 minute power nap to recover from poor sleep.’’]
4\. set_reminder [user_phone, ‘‘Establish a consistent bedtime routine and reduce screen exposure.’’]
5\. set_alarm [bedroom_speakers, ‘‘06:30’’, ‘‘Time for a light walk! Let’s start the day refreshed.’’]
and more actions… |
| Positive Feedback: | I appreciate the personalized suggestions, particularly the adjustment of the room temperature for optimal sleep and comfort, as it significantly helped me sleep better. I also found the recommendation
for a 20-30 minute power nap highly beneficial for restoring energy without affecting nighttime sleep. Additionally, the suggestion to modify my workout to lighter activities like a morning walk or gentle yoga was exactly what I needed to combat fatigue after a poor night’s sleep. These recommendations were practical and directly contributed to improving my overall comfort and restfulness. |
| Negative Feedback: | Some suggestions, however, did not align well with my preferences. Avoiding screens before bedtime felt impractical, as I often use videos to unwind. Similarly, using a heart rate monitoring app to recommend deep-breathing exercises seemed intrusive and made me feel more stressed rather than relaxed. I would prefer a less rigid approach that adapts to my personal habits and comfort. |
| Saliency Maps | ![[Uncaptioned image]](img/2b9a8ed2f5ab2524cd6644a34f0d58c9.png)  |
| New Outputs of Task Mining: | • Life Assistant: Adjust thermostat to lower room temperature for optimal sleep and comfort. • Health Consultant: Advise on power naps. Suggest 20-30 minute naps in a restful environment.
• Psychological Counselor: Suggest non-intrusive relaxation techniques, such as calming music or guided imagery, tailored to the user’s preferences to promote relaxation before sleep.
• Nutritionist: Recommend using blue light filters on devices and engaging with relaxing content before bedtime to support better sleep quality, acknowledging the user’s habit of unwinding with videos.
• Educational Consultant: Encourage establishing a consistent bedtime routine and incorporating relaxation techniques aligned with the user’s preferences to improve sleep quality.
• Fitness Coach: Modify the user’s workout to lighter activities like a morning walk or gentle yoga to combat fatigue from fragmented sleep. |

### V-B Token-Level Saliency Maps within the FaGeL-evo Algorithm

In this section, the token-level saliency map is used to visualize the finetuning details of the DualCUT algorithm(see section [III](https://arxiv.org/html/2412.20297v1#S3 "III FaGeL’s Intelligence Evolution ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration")), which offers a transparent view of how user feedback influences the model’s decision-making process at a token level. By visualizing the significance of individual tokens during the learning and prediction phases, the saliency map provides insight into the inner workings of the model, allowing for better understanding and control.

As shown in Fig. [6](https://arxiv.org/html/2412.20297v1#S5.F6 "Figure 6 ‣ V-A Performance Evolution on the Overcooked-AI Platform ‣ V Performance Evaluation of FaGeL ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration")(a), we take instructions and positive and negative feedback as input. Then, DualCUT will generate a saliency map, which is visualized in Fig. [6](https://arxiv.org/html/2412.20297v1#S5.F6 "Figure 6 ‣ V-A Performance Evolution on the Overcooked-AI Platform ‣ V Performance Evaluation of FaGeL ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration")(b). The token-level saliency maps highlight a key advantage of DualCUT: the ability to fine-tune model outputs by directly manipulating the token selection process based on both positive and negative feedback. This contrasts with traditional response-level supervised learning, which typically operates at a coarser granularity by adjusting the model’s output at the response level, often without considering the individual contributions of specific tokens to the overall prediction. By incorporating semantic information from both positive and negative judgments, DualCUT improves the model’s capacity to align its outputs with user preferences in a more targeted manner.

Token-level control enhances feedback precision and enables dynamic learning by directly adjusting token probabilities. In contrast, response-level supervision, which focuses on pairwise data, overlooks subtle token-level contributions and thus provides weaker AI alignment. As shown in Figure [7](https://arxiv.org/html/2412.20297v1#S5.F7 "Figure 7 ‣ V-A Performance Evolution on the Overcooked-AI Platform ‣ V Performance Evaluation of FaGeL ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration"), DualCUT leverages both positive and negative feedback at the token level, outperforming traditional response-level methods.

Our method integrates positive and negative feedback into a unified framework, enabling DualCUT to achieve precise alignment. Token-level saliency maps show that DualCUT enhances output control, delivering more accurate and personalized responses for improved performance in AI alignment.

### V-C A Case Study on FaGeL

To demonstrate FaGeL’s functionality in autonomous collaboration and embodied intelligence evolution, we present a case study on sleep disorder, as shown in Table [I](https://arxiv.org/html/2412.20297v1#S5.T1 "TABLE I ‣ V-A Performance Evolution on the Overcooked-AI Platform ‣ V Performance Evaluation of FaGeL ‣ FaGeL: Fabric LLMs Agent empowered Embodied Intelligence Evolution with Autonomous Human-Machine Collaboration").

In this scenario, the user experienced restlessness and poor sleep quality. The FaGeL system collected physiological and environmental data via fabric-integrated wearable devices, including heart rate, blood pressure, blood oxygen, body temperature, and sleeping posture. Using this data, the system generated a natural language descriptor of the user’s sleep status and environment, noting, for example, only 6 hours of fragmented sleep, heart rate fluctuations between 70 and 95 bpm, and a room temperature slightly above the optimal range.

Based on the descriptor and user history, FaGeL’s task mining module generated targeted tasks such as adjusting room temperature, advising on power naps, and modifying the workout plan. After executing these tasks, the user provided feedback, expressing satisfaction with some suggestions and dissatisfaction with others. The system utilized this feedback to update the task mining model, producing new tasks better aligned with the user’s preferences—for instance, suggesting blue light filters and relaxing content instead of avoiding screens entirely.

This case study illustrates how FaGeL dynamically aligns and evolves with user needs through its sensing, inference, interaction, and evolution modules. By continuously adapting to user feedback, FaGeL offers more personalized and effective suggestions, enhancing the user’s quality of life.

## VI Conclusion

In this paper, we introduced FaGeL, an embodied agent empowered by LLMs and smart fabric technology, which enables non-intrusive human-agent interaction and autonomous task generation. By leveraging multimodal sensor data from wearable and ambient sources, FaGeL continuously evolves its value system through indirect human feedback, enhancing its ability to align with human needs and intentions. Additionally, we introduced a token-level saliency map that visualizes the internal mechanisms of LLM fine-tuning, enhancing the transparency and interpretability of the agent’s decision-making process. Integrating the DualCUT algorithm further improves the agent’s performance by refining token-level feedback, contributing to more effective task learning and decision-making.

The experimental results in cooperative environments, such as the Overcooked-AI platform, demonstrate the practical effectiveness of FaGeL’s autonomous task generation and evolution capabilities. Our approach shows that it is possible to reduce the reliance on explicit feedback while still achieving substantial improvements in agent performance. This offers promising implications for the development of autonomous embodied agents capable of long-term coexistence and collaboration with humans in real-world settings.

Looking ahead, further work can explore the scalability of FaGeL’s framework in more complex, dynamic environments and its potential for integration with other AI systems. The evolution of embodied intelligence, driven by advanced feedback mechanisms and continuous learning, will be a key factor in realizing AGI-powered agents that can seamlessly adapt to diverse human needs and contexts.

## References

*   [1] J. A. OpenAI, A. Steven *et al.*, “Gpt-4 technical report. arxiv. 2023\. doi: 10.48550,” *arXiv preprint arXiv.2303.08774*.
*   [2] J. S. Park, J. O’Brien, C. J. Cai, M. R. Morris, P. Liang, and M. S. Bernstein, “Generative agents: Interactive simulacra of human behavior,” in *Proceedings of the 36th annual acm symposium on user interface software and technology*, 2023, pp. 1–22.
*   [3] Y. Liu, W. Chen, Y. Bai, J. Luo, X. Song, K. Jiang, Z. Li, G. Zhao, J. Lin, G. Li *et al.*, “Aligning cyber space with physical world: A comprehensive survey on embodied ai,” *arXiv preprint arXiv:2407.06886*, 2024.
*   [4] A. Brohan, N. Brown, J. Carbajal, Y. Chebotar, X. Chen, K. Choromanski, T. Ding, D. Driess, A. Dubey, C. Finn *et al.*, “Rt-2: Vision-language-action models transfer web knowledge to robotic control,” *arXiv preprint arXiv:2307.15818*, 2023.
*   [5] A. Brohan, N. Brown, J. Carbajal, Y. Chebotar, J. Dabis, C. Finn, K. Gopalakrishnan, K. Hausman, A. Herzog, J. Hsu *et al.*, “Rt-1: Robotics transformer for real-world control at scale,” *arXiv preprint arXiv:2212.06817*, 2022.
*   [6] S. Belkhale, T. Ding, T. Xiao, P. Sermanet, Q. Vuong, J. Tompson, Y. Chebotar, D. Dwibedi, and D. Sadigh, “Rt-h: Action hierarchies using language,” *arXiv preprint arXiv:2403.01823*, 2024.
*   [7] S. H. Vemprala, R. Bonatti, A. Bucker, and A. Kapoor, “Chatgpt for robotics: Design principles and model abilities,” *IEEE Access*, 2024.
*   [8] M. Ahn, A. Brohan, N. Brown, Y. Chebotar, O. Cortes, B. David, C. Finn, C. Fu, K. Gopalakrishnan, K. Hausman *et al.*, “Do as i can, not as i say: Grounding language in robotic affordances,” *arXiv preprint arXiv:2204.01691*, 2022.
*   [9] C. Zhang, K. Yang, S. Hu, Z. Wang, G. Li, Y. Sun, C. Zhang, Z. Zhang, A. Liu, S.-C. Zhu *et al.*, “Proagent: Building proactive cooperative ai with large language models,” *CoRR*, 2023.
*   [10] Z. Wang, S. Cai, A. Liu, Y. Jin, J. Hou, B. Zhang, H. Lin, Z. He, Z. Zheng, Y. Yang *et al.*, “Jarvis-1: Open-world multi-task agents with memory-augmented multimodal language models,” *arXiv preprint arXiv:2311.05997*, 2023.
*   [11] N. Fei, Z. Lu, Y. Gao, G. Yang, Y. Huo, J. Wen, H. Lu, R. Song, X. Gao, T. Xiang *et al.*, “Towards artificial general intelligence via a multimodal foundation model,” *Nature Communications*, vol. 13, no. 1, p. 3094, 2022.
*   [12] Z. Jia, M. Wang, B. Tong, S.-C. Zhu, and Z. Zheng, “Langsuite: Planning, controlling and interacting with large language models in embodied text environments,” *arXiv preprint arXiv:2406.16294*, 2024.
*   [13] L. Ouyang, J. Wu, X. Jiang, D. Almeida, C. Wainwright, P. Mishkin, C. Zhang, S. Agarwal, K. Slama, A. Ray *et al.*, “Training language models to follow instructions with human feedback,” *Advances in neural information processing systems*, vol. 35, pp. 27 730–27 744, 2022.
*   [14] R. Rafailov, A. Sharma, E. Mitchell, C. D. Manning, S. Ermon, and C. Finn, “Direct preference optimization: Your language model is secretly a reward model,” *Advances in Neural Information Processing Systems*, vol. 36, 2024.
*   [15] H. Yuan, Z. Yuan, C. Tan, W. Wang, S. Huang, and F. Huang, “Rrhf: Rank responses to align language models with human feedback,” *Advances in Neural Information Processing Systems*, vol. 36, 2024.
*   [16] M. Chen, J. Liu, P. Li, H. Gharavi, Y. Hao, J. Ouyang, J. Hu, L. Hu, C. Hou, I. Humar *et al.*, “Fabric computing: concepts, opportunities and challenges,” *The Innovation*, 2022.
*   [17] K. M. Herbert, S. Schrettl, S. J. Rowan, and C. Weder, “50th anniversary perspective: Solid-state multistimuli, multiresponsive polymeric materials,” *Macromolecules*, vol. 50, no. 22, pp. 8845–8870, 2017.
*   [18] J. Hu, H. Meng, G. Li, and S. I. Ibekwe, “A review of stimuli-responsive polymers for smart textile applications,” *Smart Materials and Structures*, vol. 21, no. 5, p. 053001, 2012.
*   [19] W. Xu, D. Cai, Z. Zhang, W. Lam, and S. Shi, “Reasons to reject? aligning language models with judgments,” *arXiv preprint arXiv:2312.14591*, 2023.
*   [20] M. Ghahremani, M. Latifi, and M. Babaei, “Simulation of conductivity made by inkjet-printed silver tracks in e-textiles with different weave patterns,” *Journal of Industrial Textiles*, vol. 47, no. 2, pp. 173–196, 2017.
*   [21] S. Mondal, “Phase change materials for smart textiles–an overview,” *Applied thermal engineering*, vol. 28, no. 11-12, pp. 1536–1550, 2008.
*   [22] Y. Fang, G. Chen, M. Bick, and J. Chen, “Smart textiles for personalized thermoregulation,” *Chemical Society Reviews*, vol. 50, no. 17, pp. 9357–9374, 2021.
*   [23] D. K. Macharia, S. Ahmed, B. Zhu, Z. Liu, Z. Wang, J. I. Mwasiagi, Z. Chen, and M. Zhu, “Uv/nir-light-triggered rapid and reversible color switching for rewritable smart fabrics,” *ACS applied materials & interfaces*, vol. 11, no. 14, pp. 13 370–13 379, 2019.
*   [24] Y. Zhang, Z. Hu, H. Xiang, G. Zhai, and M. Zhu, “Fabrication of visual textile temperature indicators based on reversible thermochromic fibers,” *Dyes and Pigments*, vol. 162, pp. 705–711, 2019.
*   [25] G. Huang, L. Liu, R. Wang, J. Zhang, X. Sun, and H. Peng, “Smart color-changing textile with high contrast based on a single-sided conductive fabric,” *Journal of Materials Chemistry C*, vol. 4, no. 32, pp. 7589–7594, 2016.
*   [26] C. Moretti, X. Tao, L. Koehl, and V. Koncar, “Electrochromic textile displays for personal communication,” in *Smart textiles and their applications*.   Elsevier, 2016, pp. 539–568.
*   [27] Y. Wang, W. Niu, C.-Y. Lo, Y. Zhao, X. He, G. Zhang, S. Wu, B. Ju, and S. Zhang, “Interactively full-color changeable electronic fiber sensor with high stretchability and rapid response,” *Advanced Functional Materials*, vol. 30, no. 19, p. 2000356, 2020.
*   [28] M. Chen, J. Ouyang, A. Jian, J. Liu, P. Li, Y. Hao, Y. Gong, J. Hu, J. Zhou, R. Wang *et al.*, “Imperceptible, designable, and scalable braided electronic cord,” *Nature Communications*, vol. 13, no. 1, p. 7097, 2022.
*   [29] A. Ferri, M. R. Plutino, and G. Rosace, “Recent trends in smart textiles: Wearable sensors and drug release systems,” in *AIP conference proceedings*, vol. 2145, no. 1.   AIP Publishing, 2019.
*   [30] S. Ray, J. Park, and S. Bhunia, “Wearables, implants, and internet of things: the technology needs in the evolving landscape,” *IEEE Transactions on Multi-Scale Computing Systems*, vol. 2, no. 2, pp. 123–128, 2016.
*   [31] J. Ji, T. Qiu, B. Chen, B. Zhang, H. Lou, K. Wang, Y. Duan, Z. He, J. Zhou, Z. Zhang *et al.*, “Ai alignment: A comprehensive survey,” *arXiv preprint arXiv:2310.19852*, 2023.
*   [32] Z. Wu, Y. Hu, W. Shi, N. Dziri, A. Suhr, P. Ammanabrolu, N. A. Smith, M. Ostendorf, and H. Hajishirzi, “Fine-grained human feedback gives better rewards for language model training,” *Advances in Neural Information Processing Systems*, vol. 36, pp. 59 008–59 033, 2023.
*   [33] S. Welleck, I. Kulikov, S. Roller, E. Dinan, K. Cho, and J. Weston, “Neural text generation with unlikelihood training,” *arXiv preprint arXiv:1908.04319*, 2019.
*   [34] M. Carroll, R. Shah, M. K. Ho, T. Griffiths, S. Seshia, P. Abbeel, and A. Dragan, “On the utility of learning about humans for human-ai coordination,” *Advances in neural information processing systems*, vol. 32, 2019.
*   [35] M. Chen, Y. Ma, Y. Li, D. Wu, Y. Zhang, and C. H. Youn, “Wearable 2.0: Enabling human-cloud integration in next generation healthcare systems,” *IEEE Communications Magazine*, vol. 55, no. 1, pp. 54–61, 2017.
*   [36] AI@Meta, “Llama 3 model card,” 2024.

| ![[Uncaptioned image]](img/62a03d314b05c313f1f12340f55daee5.png) | Jia Liu is currently a Ph.D candidate at Embedded and Pervasive Computing (EPIC) Laboratory in the School of Computer Science and Technology, Huazhong University of Science and Technology (HUST), China. She received her bachelor’s degree in Computer Science and Technology from University of Electronic Science and Technology of China (UESTC) in 2020, China. Her research interests include internet of things and fabric computing. |

| ![[Uncaptioned image]](img/481c29b4468bac0e536de705ec9ba557.png) | Min Chen (Fellow, IEEE) is currently a Full Professor with the School of Computer Science and Engineering, South China University of Technology. He is also the Director of the Embedded and Pervasive Computing (EPIC) Laboratory, Huazhong University of Science and Technology (HUST). His Google Scholar Citations reached more than 48,500 with an H-index of 101\. His top paper was cited more than 5,000 times. He was a recipient of the IEEE Communications Society Fred W. Ellersick Prize in 2017, the IEEE Jack Neubauer Memorial Award in 2019, and the IEEE ComSoc APB Oustanding Paper Award in 2022\. He is the founding Chair of the IEEE Computer Society Special Technical Communities on Big Data. He was selected as a Highly Cited Researcher from 2018 to 2024\. He is a fellow of IEEE. |