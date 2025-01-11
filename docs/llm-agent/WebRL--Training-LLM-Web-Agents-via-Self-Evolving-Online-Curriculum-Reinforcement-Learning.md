<!--yml
category: 未分类
date: 2025-01-11 12:00:23
-->

# WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning

> 来源：[https://arxiv.org/html/2411.02337/](https://arxiv.org/html/2411.02337/)

Zehan Qi^(1∗), Xiao Liu^(12∗), Iat Long Iong¹, Hanyu Lai¹, Xueqiao Sun¹, Wenyi Zhao², Yu Yang²

Xinyue Yang², Jiadai Sun², Shuntian Yao², Tianjie Zhang², Wei Xu¹, Jie Tang¹, Yuxiao Dong¹

¹Tsinghua University  ²Zhipu AI 

###### Abstract

Large language models (LLMs) have shown remarkable potential as autonomous agents, particularly in web-based tasks. However, existing LLM web agents heavily rely on expensive proprietary LLM APIs, while open LLMs lack the necessary decision-making capabilities. This paper introduces WebRL, a self-evolving online curriculum reinforcement learning framework designed to train high-performance web agents using open LLMs. WebRL addresses three key challenges in building LLM web agents, including the scarcity of training tasks, sparse feedback signals, and policy distribution drift in online learning. Specifically, WebRL incorporates 1) a self-evolving curriculum that generates new tasks from unsuccessful attempts, 2) a robust outcome-supervised reward model (ORM), and 3) adaptive reinforcement learning strategies to ensure consistent improvements. We apply WebRL to transform open Llama-3.1 and GLM-4 models into proficient web agents. On WebArena-Lite, WebRL improves the success rate of Llama-3.1-8B from 4.8% to 42.4%, and from 6.1% to 43% for GLM-4-9B. These open models significantly surpass the performance of GPT-4-Turbo (17.6%) and GPT-4o (13.9%) and outperform previous state-of-the-art web agents trained on open LLMs (AutoWebGLM, 18.2%). Our findings demonstrate WebRL’s effectiveness in bridging the gap between open and proprietary LLM-based web agents, paving the way for more accessible and powerful autonomous web interaction systems. The code, model, and data are made publicly available at [https://github.com/THUDM/WebRL](https://github.com/THUDM/WebRL).

¹¹footnotetext: Equal contribution. Emails: qzh23@mails.tsinghua.edu.cn, shawliu9@gmail.com^†^†footnotetext: Work done when ZQ interned at Zhipu AI.![Refer to caption](img/3d5b0940211fe23a09b5836fe30a8dbf.png)

((a))

![Refer to caption](img/abea0bd71fa06eea64894be6a837b171.png)

((b))

Figure 1: (a) Compared with all proprietary and open-sourced LLMs, GLM-4-9B with WebRL achieves the best results. (b) The performance of GLM-4-9B on WebArena-Lite (Zhou et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib56); Liu et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib22)), trained using WebRL, shows significant improvement over other baselines across all five evaluated websites.

## 1 Introduction

Large language models (LLMs) have exhibited not only superior comprehension of human language, commonsense reasoning, and knowledge acquisition, but also significant potential in complex planning and logical reasoning, indicating their promising trajectory towards serving as autonomous LLM agents (Wang et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib35); Liu et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib21)). A diverse array of applications for LLM agents has proliferated, encompassing domains such as code generation (Jimenez et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib19)), database manipulation (Zhou et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib57); Gu et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib12)), and graphical user interface (GUI) interaction (Rawles et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib31); Yang et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib41); Xie et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib38)). Among these, web agents powered by LLMs (Deng et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib6); Zheng et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib54); Lai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib20); Pan et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib25)) have garnered particular attention due to their extensive application prospects and unique potential for fostering authentic autonomous intelligence within the digital ecosystem.

Notwithstanding these advancements, existing LLM web agents, regardless of their performance metrics or architectural paradigms, remain under-developed. High-performing LLM web agents predominantly rely on meticulously crafted prompts in conjunction with proprietary LLM APIs (e.g., OpenAI GPT-4) for web page comprehension and manipulation, which is both expensive and time-intensive. Conversely, open-source LLMs exhibit notable deficiencies in their capability to function as proficient web agents, primarily due to the scarcity of decision-centric data in both pre-training and post-training periods. Despite recent endeavors (Lai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib20); Pan et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib25)) to train web agents on open LLMs via imitation learning, these approaches insufficiently leverage the inherently online nature of web interactions and fail to yield consistent, continual improvements.

Challenges. In this work, we propose to train high-performance web agents based on open LLMs within online environments, specifically utilizing WebArena (Zhou et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib56)). Our investigation has identified several critical challenges inherent to this task: 1) Insufficiency of training tasks: In contrast to offline datasets (Deng et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib6); Rawles et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib31)) that facilitate agent training and evaluation on human-annotated oracle trajectories, online benchmarks such as WebArena typically provide only a limited test set for evaluation purposes. This dearth of predefined training tasks significantly impedes the effective training of agents within these environments. 2) Sparsity and cost of feedback signals: The assessment of success for arbitrary web browsing tasks is difficult in the absence of task-specific evaluation functions. Moreover, unlike tasks in certain GUI datasets (e.g., AITW (Rawles et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib31)) and WebShop (Yao et al., [2022](https://arxiv.org/html/2411.02337v2#bib.bib42))), those in WebArena are typically of long horizons, with oracle solutions averaging about 10 steps. This characteristic introduces substantial sparsity in the available signals during online exploration. 3) Policy distribution drift in online learning: The absence of a predefined training set necessitates online exploration, inevitably leading to distribution drift in the agent’s policy. This phenomenon is likely to induce catastrophic forgetting and performance degradation over time.

The WebRL Framework. In response to these challenges, we introduce WebRL, a self-evolving online curriculum reinforcement learning framework designed for training LLM web agents. To the best of our knowledge, this represents the first systematic framework enabling effective reinforcement learning for LLM web agents from initialization in online web environments. Through the application of WebRL, we have successfully transformed a Llama-3.1-8B model into a proficient LLM web agent, elevating its success rate (SR) on WebArena-Lite (Zhou et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib56); Liu et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib22)) from an initial 4.8% to 42.4% across a diverse set of five websites. Furthermore, when applied to Llama-3.1-70B, we achieve a remarkable 49.1% SR, significantly surpassing the performance of the most advanced proprietary LLM API (GPT-4-Turbo, 17.6% SR) and the previous state-of-the-art web agents trained on open-source LLMs (AutoWebGLM (Lai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib20)), 18.2% SR).

![Refer to caption](img/8bb252e1907266079c530edf1bba77ac.png)

Figure 2: Overview of WebRL. WebRL is a self-evolving online curriculum reinforcement learning framework for LLM-based web agents, yielding consistent continual improvements throughout the iterative self-evolution.

The substantial performance gains from WebRL can be attributed to several key architectural designs. To address the scarcity of web agent training tasks, we have devised a self-evolving online curriculum that harnesses the trial-and-error process inherent in exploration. This curriculum is underpinned by a robust outcome-supervised reward model (ORM) that we have newly developed. In each training phase, novel tasks are autonomously generated from unsuccessful attempts in the preceding phase, facilitating a progressive learning trajectory. To mitigate the policy distribution shift induced by curriculum-based reinforcement learning, we incorporate a KL-divergence term between the reference and actor policies into our learning algorithm, thereby constraining policy updates and promoting stability. We implement an experience replay buffer augmented with a novel actor confidence filtering strategy to ensure the fidelity of replayed experiences and prevent over-fitting to previously acquired knowledge. The experimental results confirm the effectiveness of WebRL. In particular, the agent demonstrates improved performance when selecting past experiences of moderate difficulty—neither too simple nor too challenging relative to the agent’s current capabilities. Additionally, the use of a larger KL divergence constraint in the policy update process results in better performance when incorporating past experience.

In summary, our work makes the following significant contributions to the field:

*   •

    We introduce WebRL, a novel self-evolving online curriculum RL framework for training LLM-based web agents. For the first time, it implements the infrastructure for RL in the WebArena environment, together with a strong ORM, to drive open LLMs to become capable web agents.

*   •

    WebRL advances the RL for LLM agent training by addressing key challenges including the scarcity of training tasks, sparsity of feedback signals, and distribution drift in online learning. The self-evolving curriculum and adaptive learning strategies allow the consistent continual improvement of LLM web agents during iteration.

*   •

    We demonstrate WebRL’s substantial performance improvements over existing methodologies such as AWR and DigiRL, achieving state-of-the-art results on the WebArena-Lite benchmark. It surpasses the best proprietary LLM API and previously trained web agent on open LLMs by over 160% relatively.

## 2 WebRL: Self-Evolving Online Curriculum RL

We present a self-evolving online curriculum learning framework designed for training web agents, targeting the WebArena (Zhou et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib56)) environment. In this system, as illustrated in Figure [2](https://arxiv.org/html/2411.02337v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), the agent continuously interacts with its environment to collect real-time trajectory data. This interaction is guided by the self-evolving curriculum learning strategy that dynamically generates tasks, effectively mitigating the insufficiency of training tasks. Furthermore, the tasks generated by the self-evolving curriculum learning strategy are tailored to the agent’s current proficiency, thereby increasing the likelihood of receiving positive feedback and alleviating the challenge of sparse feedback signals. Additionally, we train an outcome-supervised reward model (ORM) to evaluate task success. We introduce a KL-constrained policy update algorithm that prevents severe policy shifts during curriculum learning. A replay buffer is also utilized to retain prior knowledge and mitigate the risks of catastrophic forgetting. These techniques enable the agent to improve incrementally, progressively handling more complex tasks. The overall training process can be found in Algorithm [1](https://arxiv.org/html/2411.02337v2#alg1 "Algorithm 1 ‣ B.3 Details of ORM ‣ Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

Problem Formulation. We model the process of completing the web task as a finite-horizon Markov Decision Process (MDP), denoted by $(S,A,R,\mathcal{T})$. Given a user instruction $I$, the agent is required to complete the corresponding task. The state $s$ is defined as the HTML content of the current web page along with the history of previous actions. The agent receives a reward of 1 upon successful task completion, and 0 otherwise. In the finite-horizon setting, the trajectory ends either when the task is accomplished or when the maximum number of interactions $T$ is exceeded. To explain our method clearly, we introduce the following notation. The policy $\pi(\cdot|s_{t},I)$ represents the distribution over actions given the state $s_{t}$ and the instruction $I$. The value function $V(s_{h},I)=\mathbb{E}_{\pi}\left[\sum_{t=h}^{T}r(s_{t},a_{t},I)\right]$ represents the expected cumulative reward from the state $s_{h}$ under policy $\pi$. The action-value function $Q(s_{t},a_{t},I)$ is the expected cumulative reward for taking action $a_{t}$ on state $s_{t}$ and following policy $\pi$ thereafter: $Q(s_{t},a_{t},I)=r(s_{t},a_{t})+V(s_{t+1},I)$.

ORM Training. In the curriculum learning process, we need to determine whether the corresponding instruction is completed based on the trajectory generated by the agent. Due to the lack of feedback from the environment, we train an LLM as the outcome-supervised reward model $\mathcal{M}_{\text{ORM}}$ to achieve this task success evaluation. $\mathcal{M}_{\text{ORM}}$ is utilized to assess whether the agent’s rollout trajectory accomplishes a given task, providing a binary reward signal (0 for failure and 1 for success).

Similar to the approach in (Zhang et al., [2024e](https://arxiv.org/html/2411.02337v2#bib.bib50)), we configure $\mathcal{M}_{\text{ORM}}$ to output “YES” or “NO” to indicate whether a trajectory successfully completes a task, leveraging the learned knowledge from the language head of $\mathcal{M}_{\text{ORM}}$. Given the limited context window of LLMs and the typically long length of HTML documents, we adopt a strategy akin to  (Pan et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib25)), keeping the HTML of only the final state to the input. In addition, the historical actions of agents, which provide information about previous steps of trajectories are also included. Thus, the input to the model consists of several components: the instruction $I$, historical actions, and HTML of the final state. We wrap these components into the prompt asking the model to determine whether the trajectory successfully completes the task described by instruction $I$. To obtain the outcome, we compare the probabilities of generating “YES” and “NO” from $\mathcal{M}_{\text{ORM}}$. If the probability of generating “YES” is higher than that of generating “NO”, the task is considered successful, and the reward is set to 1. Otherwise, the reward is set to 0.

### 2.1 Self-evolving New Instruction for Curriculum Learning

A typical challenge in training LLM web agents within WebArena is the scarcity of training tasks, resonating with the situation of developing real-world web agents. Although the recent work (Liu et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib22)) has curated a trajectory fine-tuning set for WebArena, it only contains around 1k instructions with oracle trajectories, far from enough for training strong LLM web agents. To address this limitation and drive continuous improvement, we employ a self-evolving curriculum learning strategy. This method generates new training instructions at each phase. As the phase progresses, the generated instructions become increasingly complex, allowing the agent’s capabilities to improve gradually. We implement a two-step process of generation and filtering, to produce tasks that are incrementally more challenging, while still being suitable for the agent’s current capability. During the generation step, we use the in-breadth evolving approach (Xu et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib39)) to create new instructions. We select instructions the model failed to complete in previous interaction phases as seeds for generating new instructions. Detailed prompts are provided in the Appendix [§ D](https://arxiv.org/html/2411.02337v2#A4 "Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). To ensure that the generated instructions are both feasible in the target environment and aligned with the desired difficulty level, we first filter them using the trained critic. Specifically, we use the critic to evaluate each new instruction by considering its initial state. We select instructions with critic scores between 0.05 and 0.75, ensuring that only tasks meeting our difficulty criteria are retained. We manually review generated tasks and identify tasks that cannot be completed in WebArena. Based on these findings, we develop a prompt (Figure [22](https://arxiv.org/html/2411.02337v2#A4.F22 "Figure 22 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")) and use GPT-4o to exclude infeasible tasks in WebArena automatically. The resulting set of instructions is used for interaction and training in this phase.

### 2.2 Reinforcement Learning for LLMs in Online Web Environments

In each phase of curriculum learning, the model progressively encounters and learns a new set of tasks. Considering this setting, a major challenge here is to avoid excessive policy distribution drift during each learning phase, which could lead to the catastrophic forgetting of previously acquired knowledge. Traditional approaches typically mitigate the issue by mixing data from different phases. However, in web agent tasks, intermediate steps do not receive direct process rewards, with only weak signals from the outcome of the final state. Consequently, even if an intermediate step is executed correctly, an error in later steps can easily lead to the final failure, resulting in misjudgment of the intermediate step and making it difficult to be reused. As a result, in this work, we primarily seek algorithmic improvements to address policy distribution drift more directly.

A potential solution comes from ideas in reinforcement learning with human feedback (RLHF) (Ouyang et al., [2022](https://arxiv.org/html/2411.02337v2#bib.bib24)), where the Kullback-Leibler (KL) divergence between two policies is constrained to mitigate policy distribution drift. By adapting this to our curriculum learning setup, we aim to ensure that the policy in the current phase does not deviate too much from the policy in the previous phase, while still optimizing performance on new tasks. Let the policy from the previous phase be denoted as $\pi_{\text{ref}}$, and the current policy being optimized as $\pi_{\theta}$. The instruction distribution for the current phase is represented as $\rho(I)$. The objective for optimizing $\pi_{\theta}$ in the current phase can then be written as follows:

|  | $\max_{\pi_{\theta}}\mathbb{E}_{I\sim\rho(I),a_{t}\sim\pi_{\theta}(\cdot&#124;s_{t})% }\left[\sum_{t=0}^{T}\left(r(s_{t},a_{t},I)+\beta\log\pi_{\text{ref}}(a_{t}&#124;s_% {t},I)\right)+\beta\mathcal{H}(\pi_{\theta})\right]$ |  | (1) |

where $\beta$ is a coefficient controlling the strength of the KL divergence constraint and $\mathcal{H}(\pi_{\theta})$ represents the entropy of the current policy.

Following the work of (Rafailov et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib29)), we can interpret the objective of eq. [1](https://arxiv.org/html/2411.02337v2#S2.E1 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") as a maximum entropy reinforcement learning problem. The optimal policy $\pi^{*}$ for this problem can be expressed as:

|  | $\pi^{*}(a_{t}&#124;s_{t},I)=e^{\left(Q^{*}(s_{t},a_{t},I)-V^{*}(s_{t},I)\right)/\beta}$ |  | (2) |

where $V^{*}(s_{t},I)$ is the optimal value function, representing the expected cumulative reward under the optimal policy $\pi^{*}$. $Q^{*}(s_{t},a_{t},I)$ is the optimal action-value function. The relationship between $Q^{*}$ and $V^{*}$ is given by:

|  | $Q^{*}(s_{t},a_{t},I)=\begin{cases}r(s_{t},a_{t},I)+\beta\log\pi_{\text{ref}}(a% _{t}&#124;s_{t},I)+V^{*}(s_{t+1},I),&\text{if }s_{t+1}\text{ is not terminal}\\ r(s_{t},a_{t},I)+\beta\log\pi_{\text{ref}}(a_{t}&#124;s_{t},I),&\text{if }s_{t+1}% \text{ is terminal}\end{cases}$ |  | (3) |

Based on eq. [2](https://arxiv.org/html/2411.02337v2#S2.E2 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") and eq. [3](https://arxiv.org/html/2411.02337v2#S2.E3 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), we can derive:

|  | $\beta\log\frac{\pi^{*}(a_{t}&#124;s_{t},I)}{\pi_{\text{ref}}(a_{t}&#124;s_{t},I)}=r(s_{t% },a_{t},I)+V^{*}(s_{t+1},I)-V^{*}(s_{t},I)=A^{*}(s_{t},a_{t},I)$ |  | (4) |

Here, $A^{*}(s_{t},a_{t},I)$ indicates the advantage of taking action $a_{t}$ in state $s_{t}$ compared to the average reward expected in that state. Based on the condition, we can formulate the loss function of policy $\pi_{\theta}$ as:

|  | $\mathcal{L}(\pi_{\theta})=\mathbb{E}_{\nu}\left[\left(\beta\log\frac{\pi_{% \theta}(a&#124;s,I)}{\pi_{\text{ref}}(a&#124;s,I)}-A^{*}(s,a,I)\right)^{2}\right]$ |  | (5) |

where $\nu(s)$ represents the distribution of experience in this phase. Note that our algorithm operates in the off-policy manner. A comprehensive derivation, thorough analysis, and a detailed comparison with other RL algorithms can be found in Appendix [A.1](https://arxiv.org/html/2411.02337v2#A1.SS1 "A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

What Does The Update Do? To gain a mechanistic understanding of the loss function, we analyze the gradient of the loss function, $\mathcal{L}(\pi_{\theta})$. The gradient with respect to the parameters $\theta$ can be expressed as:

|  | $\nabla_{\theta}\mathcal{L}(\pi_{\theta})=-2\beta\mathbb{E}_{\nu}\Big{[}% \underbrace{\big{(}A^{*}(s,a,I)\vphantom{\frac{\pi_{\theta}(a&#124;s,I)}{\pi_{\text% {ref}}(a&#124;s,I}}}_{\smash{\text{update direction}}}-\underbrace{\beta\log\frac{% \pi_{\theta}(a&#124;s,I)}{\pi_{\text{ref}}(a&#124;s,I)}}_{\smash{\text{KL divergence % constraint}}}\big{)}\underbrace{\nabla_{\theta}\log\pi_{\theta}(a&#124;s,I)% \vphantom{\frac{\pi_{\theta}(a&#124;s,I)}{\pi_{\text{ref}}(a&#124;s,I}}}_{\smash{\text{% sft loss}}}\Big{]}$ |  | (6) |

The gradient demonstrates the following attributions:

*   $\bullet$

    When the advantage $A^{*}(s,a,I)>0$, action $a$ is valuable, so its probability should increase. If $\pi_{\theta}$ is lower than $\pi_{\text{ref}}$, this increase will be amplified, especially as the gap between them grows. If $\pi_{\theta}$ is already higher than $\pi_{\text{ref}}$, the increase will be moderated to avoid excessive deviation.

*   $\bullet$

    When $A^{*}(s,a,I)<0$, the action is suboptimal, so its probability should decrease. If $\pi_{\theta}$ is lower than $\pi_{\text{ref}}$, the KL divergence constraint will limit how much it can be reduced to avoid a large divergence. If $\pi_{\theta}$ is higher than $\pi_{\text{ref}}$, a larger decrease will be allowed.

*   $\bullet$

    The parameter $\beta$ controls the strength of the KL divergence constraint. Adjusting $\beta$ can help fine-tune this constraint. For instance, increasing $\beta$ can prevent unnecessary boosts in action probabilities when $\pi_{\text{ref}}$ already assigns a high probability to an action.

Training a Reliable Advantage Estimator. A reliable advantage estimator is essential for effective policy updates. We train a value network $V(s_{t},I)$ and use Generalized Advantage Estimation (GAE) (Schulman et al., [2015](https://arxiv.org/html/2411.02337v2#bib.bib32)) to compute the advantage. In our setting, we only receive a binary reward (0 or 1) at the final step, with no intermediate rewards (i.e., intermediate rewards are effectively zero). Following recent approaches (Farebrother et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib8)), we train the value network using a cross-entropy objective. The loss function for the value network $V$ is defined as:

|  | $\mathcal{L}(V)=-\mathbb{E}_{\nu}\Big{[}r(s_{T},a_{T},I)\log V(s,a,I)+(1-r(s_{T% },a_{T},I))\log(1-V(s,a,I))\Big{]}$ |  | (7) |

In line with (Bai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib2)), we focus solely on the next-step and final-step advantage estimators, since there is no intermediate reward.

|  | $A(s_{t},a_{t},I)=\lambda\big{(}r(s_{t},a_{t},I)+V(s_{t+1},I)-V(s_{t},I)\big{)}% +(1-\lambda)\big{(}r(s_{T},a_{T},I)-V(s_{t},I)\big{)}$ |  | (8) |

where $\lambda$ is a balancing factor that controls the trade-off between bias and variance in advantage estimation. We set $\lambda$ as 0.5 in our work. We approximate the true advantage function $A^{*}$ using the estimated advantage function $A$, which is derived from the value network $V$. The feasibility of this approximation is demonstrated in Appendix [A.2](https://arxiv.org/html/2411.02337v2#A1.SS2 "A.2 Proof ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), where we show that using $A$ to update the policy can lead to policy improvement.

Experience Replay Buffer with Actor Confidence Filtering. In addition to controlling the policy distribution drift at the algorithmic level through KL divergence, we also implement an adaptive replay buffer to alleviate knowledge forgetting at the data level. Specifically, we only store those successful trajectories (which can be sparse) from each phase in the replay buffer. During phase $i$, we use the actor from the last phase to compute the perplexity of all actions in the buffer. Actions with a perplexity within the range of 1/0.95 to 1/0.5, along with their corresponding states, are added to the training data for the current phase. This filtering process excludes both over-familiar data and data that remains too challenging for the actor. Additionally, by storing only successful trajectories, we avoid the challenge of accurately estimating intermediate states for incorrect trajectories from previous phases.

## 3 Experiments

### 3.1 Environments and Baselines

Environments. The effectiveness of WebRL and baseline methods is evaluated using the WebArena environment (Zhou et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib56)). WebArena is particularly well-suited to our needs, as it provides a highly interactive platform that supports online learning. Additionally, WebArena encompasses a variety of websites, including OpenStreetMap (Map), Reddit, GitLab, online store content management system (CMS), and OneStopShop (OSS), making it an ideal benchmark for comprehensively assessing model performance on web tasks. In the original WebArena environment, a total of 812 instructions are provided. Considering the cost of testing, we use 165 test cases from WebArena-Lite (Liu et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib22)) for evaluation.

Baselines. We compare WebRL with proprietary LLMs utilizing prompting techniques, as well as open-sourced LLMs trained with alternative methods. For proprietary models, we select GPT-4-Turbo-2024-0409 (GPT-4-Turbo) (Achiam et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib1)) and [GPT-4o](https://openai.com/index/hello-gpt-4o/). In addition to AWM (Wang et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib36)) and WebPilot (Zhang et al., [2024f](https://arxiv.org/html/2411.02337v2#bib.bib51)), we also use the results of models under the simple prompt as baselines. Details of the simple prompt can be seen in Appendix [§ D](https://arxiv.org/html/2411.02337v2#A4 "Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). For the open-source models, in addition to using these models with the simple prompt as baselines, we also train Llama3.1 (Dubey et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib7)) and GLM-4-9B (GLM et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib11)) using various approaches as baselines. Specifically, we employ imitation learning, also referred to as supervised fine-tuning (SFT), to train these models. The training data is derived from publicly available human-labeled demonstrations, sourced from the WebArena-Lite. In addition, we also explore several reinforcement learning methods for comparison, including Filtered Behavior Cloning (Filtered BC) (Pan et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib25)), advantage-weighted regression (AWR) (Peng et al., [2019](https://arxiv.org/html/2411.02337v2#bib.bib27)) and DigiRL (Bai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib2)). For WebRL and the reinforcement learning-based baselines, we utilize the SFT-trained model as the initial model for the actor. The critic is similarly based on the SFT-trained model, with the addition of a randomly initialized value head. The training details of WebRL and baselines can be found in Appendix [§ B](https://arxiv.org/html/2411.02337v2#A2 "Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

Table 1: Task success rate (SR) of WebRL and other comparison methods, evaluated on WebArena-Lite (Zhou et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib56); Liu et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib22)), a human-verified subset of WebArena (* denotes results on full WebArena taken from literature reporting). The best and second-best models are highlighted.

| Models | #Params | Reddit | Gitlab | CMS | Map | OSS | Avg. SR |
| Proprietary LLMs |
| GPT-4-Turbo | N/A | 10.5 | 16.7 | 14.3 | 36.7 | 13.3 | 17.6 |
| GPT-4o | N/A | 10.5 | 10.0 | 20.0 | 20.0 | 11.1 | 13.9 |
| AWM + GPT-4-0613^∗ (Wang et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib36)) | N/A | 50.9 | 31.8 | 29.1 | 43.3 | 30.8 | 35.5 |
| WebPilot + GPT-4o^∗ (Zhang et al., [2024f](https://arxiv.org/html/2411.02337v2#bib.bib51)) | N/A | 65.1 | 39.4 | 24.7 | 33.9 | 36.9 | 37.2 |
| Open-sourced LLMs |
| AutoWebGLM (Lai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib20)) | 6B | 9.4 | 15.0 | 28.6 | 24.8 | 17.1 | 18.2 |
| GLM-4-Chat (GLM et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib11)) | 9B | 5.3 | 10.0 | 6.7 | 3.3 | 6.7 | 6.1 |
| GLM-4 + SFT (BC) | 9B | 47.4 | 13.3 | 31.4 | 23.3 | 13.3 | 22.4 |
| GLM-4 + Filtered BC | 9B | 52.6 | 10.0 | 31.4 | 26.7 | 20.0 | 24.8 |
| GLM-4 + AWR (Peng et al., [2019](https://arxiv.org/html/2411.02337v2#bib.bib27)) | 9B | 52.6 | 16.7 | 34.3 | 30.0 | 22.2 | 27.9 |
| GLM-4 + DigiRL (Bai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib2)) | 9B | 63.2 | 30.0 | 34.3 | 26.7 | 26.7 | 31.5 |
| GLM-4 + WebRL (ours) | 9B | 57.9 | 50.0 | 48.6 | 36.7 | 37.8 | 43.0 |
| Llama3.1-Instruct (Dubey et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib7)) | 8B | 0.0 | 3.3 | 2.9 | 3.3 | 11.1 | 4.8 |
| Llama3.1 + SFT (BC) | 8B | 36.8 | 6.7 | 20.0 | 33.3 | 17.8 | 20.6 |
| Llama3.1 + Filtered BC | 8B | 52.6 | 20.0 | 31.4 | 23.3 | 8.9 | 23.0 |
| Llama3.1 + AWR (Peng et al., [2019](https://arxiv.org/html/2411.02337v2#bib.bib27)) | 8B | 57.9 | 26.7 | 31.4 | 26.7 | 17.8 | 28.5 |
| Llama3.1 + DigiRL (Bai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib2)) | 8B | 57.9 | 26.7 | 37.1 | 33.3 | 17.8 | 30.3 |
| Llama3.1 + WebRL (ours) | 8B | 63.2 | 46.7 | 54.3 | 36.7 | 31.1 | 42.4 |
| Llama3.1-Instruct (Dubey et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib7)) | 70B | 10.5 | 16.7 | 17.1 | 20.0 | 4.4 | 12.7 |
| Llama3.1 + SFT (BC) | 70B | 52.6 | 20.0 | 20.0 | 26.7 | 13.3 | 23.0 |
| Llama3.1 + WebRL (ours) | 70B | 78.9 | 50.0 | 54.3 | 40.0 | 44.4 | 49.1 |

ORM. WebArena-Lite (Liu et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib22)) provides training samples along with a corresponding reward function. We further enhance this set of data by introducing task rewrites, as well as modifying certain data variables, such as place names and product names. We also make adjustments to the associated reward function. $\mathcal{M}_{\text{ORM}}$ is trained using rollouts of WebRL and part of baseline methods on this set of tasks, with evaluation results determined by the reward function. More details can be found in Appendix [§ B](https://arxiv.org/html/2411.02337v2#A2 "Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

### 3.2 Main Results

Our main results, presented in Table [1](https://arxiv.org/html/2411.02337v2#S3.T1 "Table 1 ‣ 3.1 Environments and Baselines ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), show that Llama3.1-8B trained using WebRL achieves an average accuracy of 42.4%, surpassing all baselines, including prompting and training alternatives. Notably, WebRL excels in specific tasks such as Gitlab (46.7%) and CMS (54.3%), demonstrating its ability to address complex web tasks effectively. Reinforcement learning-based approaches outperform those based on imitation learning, including SFT and Filtered BC, which tend to over-repeat certain actions. For instance, in the table analysis task of CMS, SFT-trained models often over-optimize the “Scroll Down” action, which occurs with high frequency. This over-optimize can cause the model to become trapped in local loops, thereby hindering its ability to achieve the overall task objective effectively. In contrast, reinforcement learning mitigates this by using a critic to estimate the value of each step, optimizing for long-term cumulative rewards, hence enabling more effective handling of complex, multi-step tasks. Furthermore, WebRL consistently outperforms DigiRL. A significant limitation of DigiRL is that it conducts policy updates on a predefined, fixed set of tasks, which may not align with the model’s current skill level. Some of these tasks are particularly challenging for the model to learn due to the sparse reward situations. This misalignment can cause the model to converge to suboptimal solutions and restrict its capacity for exploration and skill advancement. WebRL addresses this limitation by employing self-evolving curriculum learning, adjusting the task complexity based on the model’s current abilities. This strategy promotes wider exploration and supports continuous improvement. A similar phenomenon is also observed in the case of the GLM-4-9B, providing evidence that the benefits of WebRL extend across different model architectures, validating its robustness and adaptability.

### 3.3 Scaling Effect of WebRL

We further validate the effectiveness of WebRL on larger-scale models by training Llama3.1-70B using WebRL. The specific results are presented in Table [1](https://arxiv.org/html/2411.02337v2#S3.T1 "Table 1 ‣ 3.1 Environments and Baselines ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). After training with WebRL, Llama3.1-70B achieves an overall accuracy of 49.1%, reflecting a 26.1% improvement over the accuracy achieved with SFT. This indicates that WebRL is scalable and can be effectively applied to larger-scale models. Furthermore, when comparing the performance improvement from Llama3.1-8B to Llama3.1-70B achieved through SFT, WebRL demonstrates even greater performance gains as the model scale increases.

### 3.4 Distribution Analysis of Error Types

We compare the performance of Llama 3.1-8B trained with WebRL against baseline methods across different error types: “Fail to Recover”, “Get Stuck Midway”, “Stop at Wrong Page”, and “Fail to Make Reasonable Attempt”, as shown in Figure [3](https://arxiv.org/html/2411.02337v2#S3.F3 "Figure 3 ‣ 3.4 Distribution Analysis of Error Types ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). WebRL demonstrates significant advantages in reducing the “Get Stuck Midway” error, especially compared to SFT and Filtered BC. The “Get Stuck Midway” error typically arises when the model gets trapped in a loop, repeatedly executing the same action without making progress. Reinforcement learning helps mitigate this issue by optimizing each action while considering its overall impact on the task, enabling the model to make more effective decisions. Additionally, the model trained with WebRL demonstrates enhanced robustness in handling the “Fail to Recover” error. Through curriculum learning, the model gradually learns how to adapt its actions when encountering failures. For example, when the search query “Pharmacy near CMU within a 20-minute walking distance” does not yield the desired results, the model learns to modify the query to “Pharmacy near CMU” and attempts the search again, rather than repeating ineffective actions. In addition, WebRL exhibits the lowest error rate on both “Stop at Wrong Page” and “Fail to Make Reasonable Attempt” errors, indicating the model trained with WebRL has a more profound comprehension of the relationship between tasks and web pages. It can better identify the correct page needed to complete a specific task, reducing the chances of mistakenly stopping on the wrong page or navigating to an incorrect page.

![Refer to caption](img/7dfcd806d34ec4d283a4b8eddaedc6dd.png)

Figure 3: Distribution analysis of error types for WebRL and baseline methods.

### 3.5 Performance on Tasks with Varying Step Requirements

We evaluate the performance of Llama3.1-8B, trained using WebRL and baseline methods, on tasks with varying step requirements. To determine the required step count for each task, we exclude tasks that no model completes and use the trajectory with the fewest steps as the required step count for each remaining task. The results are shown in Figure [5](https://arxiv.org/html/2411.02337v2#S3.F5 "Figure 5 ‣ 3.5 Performance on Tasks with Varying Step Requirements ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). It can be seen that the performance of models trained with SFT and Filtered BC shows a noticeable decline as the task length increases. This is likely because these models optimize individual steps without considering the cumulative impact, making them less effective on long-horizon tasks. DigiRL-trained model improves performance on medium-length tasks but struggles with longer tasks (more than 10 steps). This limitation may stem from DigiRL’s online learning on a fixed set of tasks. Even when the model executes intermediate steps correctly, it doesn’t receive positive rewards if errors occur in later steps, making it harder for the model to learn how to complete tasks that require many steps effectively. In contrast, WebRL overcomes this issue with curriculum learning, progressively increasing task difficulty. This approach enhances the model’s ability to handle long sequences, leading to significant performance improvements on tasks requiring long-term planning compared to other methods.

![Refer to caption](img/a0aab74ce4c1c90e428f9ee4ca70b1e8.png)

Figure 4: Accuracy of WebRL and baselines for tasks requiring different steps.

![Refer to caption](img/dd738171fadcadc7827e7cd0e6773303.png)

Figure 5: Ablation study of WebRL on replay buffer, KL-constrained policy update and curriculum strategy.

### 3.6 Performance on Tasks with Varying Complexity

![Refer to caption](img/c67908a0f757c878f65fdb92bf6c0949.png)

Figure 6: Accuracy of WebRL and baselines for tasks with different complexity.

We further analyze the performance of WebRL and baselines across instructions of varying complexity, as shown in Figure [6](https://arxiv.org/html/2411.02337v2#S3.F6 "Figure 6 ‣ 3.6 Performance on Tasks with Varying Complexity ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). Instruction complexity is measured by the number of requirements in the task. For example, the instruction “What are the top-3 best-selling products in Jan 2023” has two requirements: identifying the top-3 products and specifying the timeframe, giving it a complexity level of 2. Our results show that WebRL performs well across different complexity levels, particularly excelling in more complex instructions. In contrast, while DigiRL uses online learning, it struggles with higher complexity due to its focus on a predefined set of tasks that do not align with the model’s capabilities, limiting its adaptability. This highlights the effectiveness of our self-evolving curriculum learning strategy, which progressively increases task complexity based on the model’s capacity, enabling better performance on challenging tasks.

### 3.7 Ablation Study

We conduct an ablation study to evaluate the impact of the replay buffer, KL-constrained policy update algorithm, and the curriculum learning strategy on WebRL. To assess their contributions, we compare WebRL with four alternative models: (1) WebRL w/o replay buffer, where training uses only the current interaction trajectory, (2) WebRL w/o KL, where the policy is updated using REINFORCE with value function baseline (the gradient is $\mathbb{E}_{\nu}[(A(s,a,I)\nabla_{\theta}\log\pi_{\theta}(a|s,I)]$) but retains the replay buffer, (3) WebRL w/o KL & replay buffer, which uses neither a replay buffer nor the KL-constrained policy update algorithm, and (4) WebRL w/o CL, which ablates the curriculum learning approach, utilizing only the instructions generated in the first phase.

The results, shown in Figure [5](https://arxiv.org/html/2411.02337v2#S3.F5 "Figure 5 ‣ 3.5 Performance on Tasks with Varying Step Requirements ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), demonstrate that all the components used by WebRL are essential. (1) The role of the replay buffer. The results reveal that when the replay buffer is removed, both WebRL w/o replay buffer and WebRL w/o KL & replay buffer experience worsening performance over time. This decline occurs because the models lose access to earlier experiences and focus only on recent data, leading to knowledge degradation. (2) The role of the KL-constrained policy update algorithm. Comparing WebRL and WebRL w/o KL, WebRL consistently performs better, due to the incorporation of KL-constrained policy update algorithm. When the replay buffer is not used, the KL-constrained policy update algorithm degrades more slowly than REINFORCE with a value function baseline because it better retains past knowledge by controlling KL divergence. In contrast, REINFORCE with a value function baseline quickly overfits the current phase’s data and consistently underperforms its initial value. Overall, the KL-constrained policy update algorithm is more effective at balancing the retention of past knowledge with the learning of new information. (3) The role of the self-evolving curriculum learning strategy. When comparing WebRL to WebRL w/o CL, both exhibit an overall upward trend due to online learning. However, WebRL w/o CL progresses more slowly and reaches a lower performance ceiling because it operates within a fixed task framework, whereas WebRL generates new tasks that adapt to its evolving capabilities. This highlights the effectiveness of our self-evolving curriculum learning approach. Additional ablation studies to further investigate the effects of curriculum learning are presented in Figure [14](https://arxiv.org/html/2411.02337v2#A3.F14 "Figure 14 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

Table 2: The impact of perplexity in replay buffer filtering of WebRL.

| $\mathbf{[1,\infty]}$ | $\mathbf{[1,\frac{1}{0.95}]}$ | $\mathbf{[\frac{1}{0.95},\!\frac{1}{0.5}]}$ | $\mathbf{[\frac{1}{0.5},\!\infty]}$ |
| 29.1 | 27.9 | 31.5 | 23.0 |

The influence of perplexity. We analyze the impact of using perplexity to select data from the replay buffer for training. Various perplexity thresholds are tested in the first learning phase, and the results are summarized in Table [2](https://arxiv.org/html/2411.02337v2#S3.T2 "Table 2 ‣ 3.7 Ablation Study ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). It can be observed that training on data with very low perplexity (range [1, 1/0.95]) leads to performance deterioration. This suggests that repeatedly learning overly familiar data harms the model. Similarly, training exclusively on data with high perplexity (above 1/0.5) also degrades performance, likely due to the model struggling with unfamiliar data, causing a significant shift in policy distribution and hindering generalization. Optimal performance is achieved when training on data with a perplexity range of [1/0.95, 1/0.5], indicating that a balance between simple and complex data enhances model performance by focusing on moderately difficult examples.

![Refer to caption](img/205fa5e7abdc47011acfc5f06ac9139d.png)

Figure 7: The impact of $\beta$ of KL-constrained policy update algorithm on the model’s performance.

The impact of $\beta$. We investigate the effect of $\beta$ on performance with and without the replay buffer, as shown in Figure [7](https://arxiv.org/html/2411.02337v2#S3.F7 "Figure 7 ‣ 3.7 Ablation Study ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). The study is conducted on curriculum learning in one phase. First, when $\beta$ is too small (e.g., $\beta$ = 0.01), model performance deteriorates, regardless of whether a replay buffer is used. This decline occurs because a small $\beta$ imposes a weak control over the KL divergence, causing the model to overfit the current data. Second, without the replay buffer, performance initially improves as $\beta$ increases but then declines when $\beta$ becomes too large, indicating that large $\beta$ (e.g., $\beta\geq 1$) will overly restrict KL divergence, limiting the model’s ability to update its policy effectively. In contrast, with the replay buffer, performance stays high even at larger $\beta$ values. The historical experiences stored in the replay buffer facilitate more frequent and diverse parameter updates, supporting a stable improvement process, even as the $\beta$ value increases.

Table 3: Evaluation on output-supervised methods (baselines adopted from (Pan et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib25))). Our ORM, without accessing proprietary GPT-4, performs the best among all.

|  | Our ORM (8B) | GPT-4 | Captioner + GPT-4 | GPT-4V |
| --- | --- | --- | --- | --- |
| Test Dataset (%) | 80.8 | 71.9 | 72.6 | 71.2 |
| Rollout (%) | 79.4 | 71.2 | 73.3 | 70.5 |

### 3.8 Evaluation of ORM

In the WebRL framework, continuous improvement depends significantly on the effectiveness of the ORM, which plays a crucial role in evaluating interaction trajectories to guide the agent’s learning process. To assess ORM’s effectiveness, we compare its performance with several baseline models, including GPT-4-Turbo using identical inputs of our ORM, Captioner + GPT-4-Turbo, and GPT-4V, both using the same prompts with Pan et al. ([2024](https://arxiv.org/html/2411.02337v2#bib.bib25)). We evaluate ORM and the baselines on two datasets: the WebArena-Lite test set and 100 sampled rollouts which are manually labeled. For the WebArena-Lite test data, we use its reward function outputs as labels. The results, shown in Table [3](https://arxiv.org/html/2411.02337v2#S3.T3 "Table 3 ‣ 3.7 Ablation Study ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), indicate that while the baseline models consistently achieve an accuracy slightly above 70%, our ORM surpasses them with an accuracy of approximately 80%.

### 3.9 Case Study

![Refer to caption](img/a0ffbff92431c7a9a0694f8698de4d5e.png)

Figure 8: Examples of instructions generated in different phases under self-evolving curriculum learning.

Figure [8](https://arxiv.org/html/2411.02337v2#S3.F8 "Figure 8 ‣ 3.9 Case Study ‣ 3 Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") presents some instructions generated by the self-evolving curriculum learning strategy across different phases. Although these instructions are grouped by phase, the instructions shown in phase $i+1$ are not necessarily generated using the instructions from phase $i$ as seeds. As the phase increases, two types of augmentation occur for previously incomplete instructions. In one case, new instructions with similar task requirements or lower difficulty levels are generated. These new instructions help the model to successfully complete previously unfinishable tasks by allowing it to practice and build competence with the easier or related tasks first. For example, in phase 2, the instruction improves upon the phase 1 instruction by offering a clearer task description and explicitly requiring results in “yearly interval”. This enhancement enables the model to complete the task successfully by removing ambiguity. Additionally, changing the description of the instruction can improve agent exploration. With the positive feedback from the clarified phase 2 instruction, the model can better understand and accurately perform the original phase 1 task as well. In another case, tasks with increased complexity and diversity are generated. This task complication facilitates a continuous improvement in the model’s capabilities by challenging its performance boundaries. Therefore, the process of instruction generation exhibits such a pattern: for tasks that the model is unable to perform, analogous tasks are created to provide incremental steps that facilitate learning how to accomplish that type of task. Furthermore, tasks that remain challenging for the model are also generated and undergo the aforementioned iterative process. Through this cycle of challenge and refinement, the model’s capabilities gradually expand, enabling it to handle increasingly complex tasks over time.

## 4 Related Works

Adopting LLMs as Agent. As LLM capabilities advance, their applications extend beyond text generation (Zheng et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib55); Zhao et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib53)) and complex reasoning (Zelikman et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib43); Zhang et al., [2024d](https://arxiv.org/html/2411.02337v2#bib.bib49)), and increasingly involve acting as agents for device control. Current research in this area falls into two main categories: training-free and training-based approaches. Training-free methods enhance pre-existing LLMs through prompt engineering (Yan et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib40); He et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib16); Zhang et al., [2024c](https://arxiv.org/html/2411.02337v2#bib.bib48)) and constructing complex systems (Liu et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib23); Yang et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib41); Wang et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib35); Wu et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib37); Iong et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib18); Zhang et al., [2024a](https://arxiv.org/html/2411.02337v2#bib.bib46)). However, their performance is constrained by the limitations of the underlying LLMs, and the absence of fine-tuning restricts further improvement (Chen et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib5); Zeng et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib44); Xie et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib38)). Training-based approaches, primarily relying on imitation learning, require extensive expert demonstrations (Zhang & Zhang, [2023](https://arxiv.org/html/2411.02337v2#bib.bib52); Gur et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib13); Deng et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib6); Hong et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib17); Rawles et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib31); Zhang et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib47)), which are costly to obtain. Although some methods use powerful LLMs like GPT-4 to generate demonstrations (Chen et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib5)), their accuracy remains insufficient for complex tasks. These methods often maximize the likelihood of individual actions without adequately considering long-term effects, limiting generalization (Ghosh et al., [2021](https://arxiv.org/html/2411.02337v2#bib.bib10); Bai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib2)). To mitigate this, some studies use sampling-based methods to estimate long-term effects (Lai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib20); Putta et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib28)), while others, like ours, leverage reinforcement learning (Carta et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib3); Bai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib2); Pan et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib25); Tan et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib34); Zhai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib45)). However, most existing methods rely on static task sets, which hinder the agent’s continuous improvement as its capabilities evolve. To overcome this, we propose a dynamic task generation framework that adjusts task complexity based on the agent’s progress, alongside a policy-update algorithm for ongoing performance enhancement.

Reinforcement Learning for LLMs. Reinforcement learning (RL) has gained traction in training large language models (LLMs), with applications ranging from preference optimization (Ouyang et al., [2022](https://arxiv.org/html/2411.02337v2#bib.bib24); Casper et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib4)) to complex reasoning (Hao et al., [2023](https://arxiv.org/html/2411.02337v2#bib.bib15); Pang et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib26)). A growing area of interest involves using RL for device control tasks, which require multi-step interactions where the model selects appropriate actions based on the device state. This sequential decision-making aligns well with RL techniques. Existing research has explored RL-trained LLM agents for complex device control, primarily using online learning methods. For instance, AgentQ (Putta et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib28)) uses DPO (Rafailov et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib30)) for policy updates based on interaction data, while other methods (Bai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib2); Zhou et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib58); Zhai et al., [2024](https://arxiv.org/html/2411.02337v2#bib.bib45)) utilize actor-critic architectures, which we also adopt. However, in web tasks, feedback is often limited to binary success or failure after multiple interaction rounds. This can penalize correct intermediate actions due to later mistakes, complicating the reuse of previous data. Moreover, current research tends to focus on a fixed set of tasks for comparison with imitation learning, limiting the potential for continuous improvement through trial and error. To address this, we propose an autonomous curriculum learning mechanism that dynamically generates tasks based on the agent’s evolving skills, fostering ongoing progress. Additionally, we introduce a KL-constrained policy update algorithm and a specialized replay buffer to reuse valuable historical data and prevent knowledge forgetting during iterative curriculum updates.

## 5 Conclusion

In this work, we introduce WebRL, a novel self-evolving online curriculum reinforcement learning framework for training LLM-based web agents. By addressing key challenges including the scarcity of training tasks, feedback signal sparsity, and policy distribution drift, WebRL enables continual and consistent improvement in agent performance within online environments like WebArena. Our approach demonstrates substantial performance gains, significantly surpassing existing state-of-the-art web agents and proprietary LLM APIs. These results highlight the effectiveness of WebRL in advancing the capabilities of open-source LLMs for web-based tasks.

Acknowledgments. We would like to thank Zhipu AI for sponsoring the computation resources and annotation cost used in this work.

## References

*   Achiam et al. (2023) Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, et al. Gpt-4 technical report. *arXiv preprint arXiv:2303.08774*, 2023.
*   Bai et al. (2024) Hao Bai, Yifei Zhou, Mert Cemri, Jiayi Pan, Alane Suhr, Sergey Levine, and Aviral Kumar. Digirl: Training in-the-wild device-control agents with autonomous reinforcement learning. *arXiv preprint arXiv:2406.11896*, 2024.
*   Carta et al. (2023) Thomas Carta, Clément Romac, Thomas Wolf, Sylvain Lamprier, Olivier Sigaud, and Pierre-Yves Oudeyer. Grounding large language models in interactive environments with online reinforcement learning. In *International Conference on Machine Learning*, pp.  3676–3713\. PMLR, 2023.
*   Casper et al. (2024) Stephen Casper, Xander Davies, Claudia Shi, Thomas Krendl Gilbert, Jérémy Scheurer, Javier Rando, Rachel Freedman, Tomasz Korbak, David Lindner, Pedro Freire, et al. Open problems and fundamental limitations of reinforcement learning from human feedback. *Transactions on Machine Learning Research*, 2024.
*   Chen et al. (2023) Baian Chen, Chang Shu, Ehsan Shareghi, Nigel Collier, Karthik Narasimhan, and Shunyu Yao. Fireact: Toward language agent fine-tuning. *arXiv preprint arXiv:2310.05915*, 2023.
*   Deng et al. (2024) Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Sam Stevens, Boshi Wang, Huan Sun, and Yu Su. Mind2web: Towards a generalist agent for the web. *Advances in Neural Information Processing Systems*, 36, 2024.
*   Dubey et al. (2024) Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan, et al. The llama 3 herd of models. *arXiv preprint arXiv:2407.21783*, 2024.
*   Farebrother et al. (2024) Jesse Farebrother, Jordi Orbay, Quan Vuong, Adrien Ali Taïga, Yevgen Chebotar, Ted Xiao, Alex Irpan, Sergey Levine, Pablo Samuel Castro, Aleksandra Faust, et al. Stop regressing: Training value functions via classification for scalable deep rl. *arXiv preprint arXiv:2403.03950*, 2024.
*   Fujimoto & Gu (2021) Scott Fujimoto and Shixiang Shane Gu. A minimalist approach to offline reinforcement learning. *Advances in neural information processing systems*, 34:20132–20145, 2021.
*   Ghosh et al. (2021) Dibya Ghosh, Jad Rahme, Aviral Kumar, Amy Zhang, Ryan P Adams, and Sergey Levine. Why generalization in rl is difficult: Epistemic pomdps and implicit partial observability. *Advances in neural information processing systems*, 34:25502–25515, 2021.
*   GLM et al. (2024) Team GLM, Aohan Zeng, Bin Xu, Bowen Wang, Chenhui Zhang, Da Yin, Diego Rojas, Guanyu Feng, Hanlin Zhao, Hanyu Lai, et al. Chatglm: A family of large language models from glm-130b to glm-4 all tools. *arXiv preprint arXiv:2406.12793*, 2024.
*   Gu et al. (2024) Yu Gu, Yiheng Shu, Hao Yu, Xiao Liu, Yuxiao Dong, Jie Tang, Jayanth Srinivasa, Hugo Latapie, and Yu Su. Middleware for llms: Tools are instrumental for language agents in complex environments. *arXiv preprint arXiv:2402.14672*, 2024.
*   Gur et al. (2024) Izzeddin Gur, Hiroki Furuta, Austin V Huang, Mustafa Safdari, Yutaka Matsuo, Douglas Eck, and Aleksandra Faust. A real-world webagent with planning, long context understanding, and program synthesis. In *The Twelfth International Conference on Learning Representations*, 2024.
*   Haarnoja et al. (2018) Tuomas Haarnoja, Aurick Zhou, Kristian Hartikainen, George Tucker, Sehoon Ha, Jie Tan, Vikash Kumar, Henry Zhu, Abhishek Gupta, Pieter Abbeel, et al. Soft actor-critic algorithms and applications. *arXiv preprint arXiv:1812.05905*, 2018.
*   Hao et al. (2023) Shibo Hao, Yi Gu, Haodi Ma, Joshua Hong, Zhen Wang, Daisy Wang, and Zhiting Hu. Reasoning with language model is planning with world model. In *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing*, pp.  8154–8173, 2023.
*   He et al. (2024) Hongliang He, Wenlin Yao, Kaixin Ma, Wenhao Yu, Yong Dai, Hongming Zhang, Zhenzhong Lan, and Dong Yu. Webvoyager: Building an end-to-end web agent with large multimodal models. *arXiv preprint arXiv:2401.13919*, 2024.
*   Hong et al. (2024) Wenyi Hong, Weihan Wang, Qingsong Lv, Jiazheng Xu, Wenmeng Yu, Junhui Ji, Yan Wang, Zihan Wang, Yuxiao Dong, Ming Ding, et al. Cogagent: A visual language model for gui agents. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pp.  14281–14290, 2024.
*   Iong et al. (2024) Iat Long Iong, Xiao Liu, Yuxuan Chen, Hanyu Lai, Shuntian Yao, Pengbo Shen, Hao Yu, Yuxiao Dong, and Jie Tang. Openwebagent: An open toolkit to enable web agents on large language models. In *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 3: System Demonstrations)*, pp.  72–81, 2024.
*   Jimenez et al. (2024) Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik R Narasimhan. SWE-bench: Can language models resolve real-world github issues? In *The Twelfth International Conference on Learning Representations*, 2024. URL [https://openreview.net/forum?id=VTF8yNQM66](https://openreview.net/forum?id=VTF8yNQM66).
*   Lai et al. (2024) Hanyu Lai, Xiao Liu, Iat Long Iong, Shuntian Yao, Yuxuan Chen, Pengbo Shen, Hao Yu, Hanchen Zhang, Xiaohan Zhang, Yuxiao Dong, and Jie Tang. Autowebglm: A large language model-based web navigating agent. In *Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*, KDD ’24, pp.  5295–5306, 2024.
*   Liu et al. (2024a) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, et al. Agentbench: Evaluating llms as agents. In *The Twelfth International Conference on Learning Representations*, 2024a.
*   Liu et al. (2024b) Xiao Liu, Tianjie Zhang, Yu Gu, Iat Long Iong, Yifan Xu, Xixuan Song, Shudan Zhang, Hanyu Lai, Xinyi Liu, Hanlin Zhao, et al. Visualagentbench: Towards large multimodal models as visual foundation agents. *arXiv preprint arXiv:2408.06327*, 2024b.
*   Liu et al. (2023) Zhiwei Liu, Weiran Yao, Jianguo Zhang, Le Xue, Shelby Heinecke, Rithesh Murthy, Yihao Feng, Zeyuan Chen, Juan Carlos Niebles, Devansh Arpit, et al. Bolaa: Benchmarking and orchestrating llm-augmented autonomous agents. *arXiv preprint arXiv:2308.05960*, 2023.
*   Ouyang et al. (2022) Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, et al. Training language models to follow instructions with human feedback. *Advances in neural information processing systems*, 35:27730–27744, 2022.
*   Pan et al. (2024) Jiayi Pan, Yichi Zhang, Nicholas Tomlin, Yifei Zhou, Sergey Levine, and Alane Suhr. Autonomous evaluation and refinement of digital agents. In *First Conference on Language Modeling*, 2024.
*   Pang et al. (2024) Richard Yuanzhe Pang, Weizhe Yuan, Kyunghyun Cho, He He, Sainbayar Sukhbaatar, and Jason Weston. Iterative reasoning preference optimization. *arXiv preprint arXiv:2404.19733*, 2024.
*   Peng et al. (2019) Xue Bin Peng, Aviral Kumar, Grace Zhang, and Sergey Levine. Advantage-weighted regression: Simple and scalable off-policy reinforcement learning. *arXiv preprint arXiv:1910.00177*, 2019.
*   Putta et al. (2024) Pranav Putta, Edmund Mills, Naman Garg, Sumeet Motwani, Chelsea Finn, Divyansh Garg, and Rafael Rafailov. Agent q: Advanced reasoning and learning for autonomous ai agents. *arXiv preprint arXiv:2408.07199*, 2024.
*   Rafailov et al. (2024a) Rafael Rafailov, Joey Hejna, Ryan Park, and Chelsea Finn. From $r$ to $q^{*}$: Your language model is secretly a q-function. *arXiv preprint arXiv:2404.12358*, 2024a.
*   Rafailov et al. (2024b) Rafael Rafailov, Archit Sharma, Eric Mitchell, Christopher D Manning, Stefano Ermon, and Chelsea Finn. Direct preference optimization: Your language model is secretly a reward model. *Advances in Neural Information Processing Systems*, 36, 2024b.
*   Rawles et al. (2024) Christopher Rawles, Alice Li, Daniel Rodriguez, Oriana Riva, and Timothy Lillicrap. Androidinthewild: A large-scale dataset for android device control. *Advances in Neural Information Processing Systems*, 36, 2024.
*   Schulman et al. (2015) John Schulman, Philipp Moritz, Sergey Levine, Michael Jordan, and Pieter Abbeel. High-dimensional continuous control using generalized advantage estimation. *arXiv preprint arXiv:1506.02438*, 2015.
*   Schulman et al. (2017) John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, and Oleg Klimov. Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*, 2017.
*   Tan et al. (2024) Weihao Tan, Wentao Zhang, Shanqi Liu, Longtao Zheng, Xinrun Wang, and Bo An. True knowledge comes from practice: Aligning large language models with embodied environments via reinforcement learning. In *The Twelfth International Conference on Learning Representations*, 2024.
*   Wang et al. (2024a) Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, and Anima Anandkumar. Voyager: An open-ended embodied agent with large language models. *Transactions on Machine Learning Research*, 2024a.
*   Wang et al. (2024b) Zora Zhiruo Wang, Jiayuan Mao, Daniel Fried, and Graham Neubig. Agent workflow memory. *arXiv preprint arXiv:2409.07429*, 2024b.
*   Wu et al. (2024) Zhiyong Wu, Chengcheng Han, Zichen Ding, Zhenmin Weng, Zhoumianze Liu, Shunyu Yao, Tao Yu, and Lingpeng Kong. Os-copilot: Towards generalist computer agents with self-improvement. *arXiv preprint arXiv:2402.07456*, 2024.
*   Xie et al. (2024) Tianbao Xie, Danyang Zhang, Jixuan Chen, Xiaochuan Li, Siheng Zhao, Ruisheng Cao, Toh Jing Hua, Zhoujun Cheng, Dongchan Shin, Fangyu Lei, et al. Osworld: Benchmarking multimodal agents for open-ended tasks in real computer environments. *arXiv preprint arXiv:2404.07972*, 2024.
*   Xu et al. (2023) Can Xu, Qingfeng Sun, Kai Zheng, Xiubo Geng, Pu Zhao, Jiazhan Feng, Chongyang Tao, and Daxin Jiang. Wizardlm: Empowering large language models to follow complex instructions. *arXiv preprint arXiv:2304.12244*, 2023.
*   Yan et al. (2023) An Yan, Zhengyuan Yang, Wanrong Zhu, Kevin Lin, Linjie Li, Jianfeng Wang, Jianwei Yang, Yiwu Zhong, Julian McAuley, Jianfeng Gao, et al. Gpt-4v in wonderland: Large multimodal models for zero-shot smartphone gui navigation. *arXiv preprint arXiv:2311.07562*, 2023.
*   Yang et al. (2023) Zhao Yang, Jiaxuan Liu, Yucheng Han, Xin Chen, Zebiao Huang, Bin Fu, and Gang Yu. Appagent: Multimodal agents as smartphone users. *arXiv preprint arXiv:2312.13771*, 2023.
*   Yao et al. (2022) Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. Webshop: Towards scalable real-world web interaction with grounded language agents. *Advances in Neural Information Processing Systems*, 35:20744–20757, 2022.
*   Zelikman et al. (2024) Eric Zelikman, Georges Harik, Yijia Shao, Varuna Jayasiri, Nick Haber, and Noah D Goodman. Quiet-star: Language models can teach themselves to think before speaking. *arXiv preprint arXiv:2403.09629*, 2024.
*   Zeng et al. (2023) Aohan Zeng, Mingdao Liu, Rui Lu, Bowen Wang, Xiao Liu, Yuxiao Dong, and Jie Tang. Agenttuning: Enabling generalized agent abilities for llms. *arXiv preprint arXiv:2310.12823*, 2023.
*   Zhai et al. (2024) Yuexiang Zhai, Hao Bai, Zipeng Lin, Jiayi Pan, Shengbang Tong, Yifei Zhou, Alane Suhr, Saining Xie, Yann LeCun, Yi Ma, et al. Fine-tuning large vision-language models as decision-making agents via reinforcement learning. *arXiv preprint arXiv:2405.10292*, 2024.
*   Zhang et al. (2024a) Chaoyun Zhang, Liqun Li, Shilin He, Xu Zhang, Bo Qiao, Si Qin, Minghua Ma, Yu Kang, Qingwei Lin, Saravan Rajmohan, et al. Ufo: A ui-focused agent for windows os interaction. *arXiv preprint arXiv:2402.07939*, 2024a.
*   Zhang et al. (2024b) Jianguo Zhang, Tian Lan, Rithesh Murthy, Zhiwei Liu, Weiran Yao, Juntao Tan, Thai Hoang, Liangwei Yang, Yihao Feng, Zuxin Liu, et al. Agentohana: Design unified data and training pipeline for effective agent learning. *arXiv preprint arXiv:2402.15506*, 2024b.
*   Zhang et al. (2024c) Jiwen Zhang, Jihao Wu, Yihua Teng, Minghui Liao, Nuo Xu, Xiao Xiao, Zhongyu Wei, and Duyu Tang. Android in the zoo: Chain-of-action-thought for gui agents. *arXiv preprint arXiv:2403.02713*, 2024c.
*   Zhang et al. (2024d) Kechi Zhang, Jia Li, Ge Li, Xianjie Shi, and Zhi Jin. Codeagent: Enhancing code generation with tool-integrated agent systems for real-world repo-level coding challenges. *arXiv preprint arXiv:2401.07339*, 2024d.
*   Zhang et al. (2024e) Lunjun Zhang, Arian Hosseini, Hritik Bansal, Mehran Kazemi, Aviral Kumar, and Rishabh Agarwal. Generative verifiers: Reward modeling as next-token prediction. *arXiv preprint arXiv:2408.15240*, 2024e.
*   Zhang et al. (2024f) Yao Zhang, Zijian Ma, Yunpu Ma, Zhen Han, Yu Wu, and Volker Tresp. Webpilot: A versatile and autonomous multi-agent system for web task execution with strategic exploration. *arXiv preprint arXiv:2408.15978*, 2024f.
*   Zhang & Zhang (2023) Zhuosheng Zhang and Aston Zhang. You only look at screens: Multimodal chain-of-action agents. *arXiv preprint arXiv:2309.11436*, 2023.
*   Zhao et al. (2023) Wayne Xin Zhao, Kun Zhou, Junyi Li, Tianyi Tang, Xiaolei Wang, Yupeng Hou, Yingqian Min, Beichen Zhang, Junjie Zhang, Zican Dong, et al. A survey of large language models. *arXiv preprint arXiv:2303.18223*, 2023.
*   Zheng et al. (2024) Boyuan Zheng, Boyu Gou, Jihyung Kil, Huan Sun, and Yu Su. Gpt-4v (ision) is a generalist web agent, if grounded. In *Forty-first International Conference on Machine Learning*, 2024.
*   Zheng et al. (2023) Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric Xing, et al. Judging llm-as-a-judge with mt-bench and chatbot arena. *Advances in Neural Information Processing Systems*, 36:46595–46623, 2023.
*   Zhou et al. (2024a) Shuyan Zhou, Frank F Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, et al. Webarena: A realistic web environment for building autonomous agents. In *The Twelfth International Conference on Learning Representations*, 2024a.
*   Zhou et al. (2023) Xuanhe Zhou, Guoliang Li, and Zhiyuan Liu. Llm as dba. *arXiv preprint arXiv:2308.05481*, 2023.
*   Zhou et al. (2024b) Yifei Zhou, Andrea Zanette, Jiayi Pan, Sergey Levine, and Aviral Kumar. Archer: Training language model agents via hierarchical multi-turn rl. In *Forty-first International Conference on Machine Learning*, 2024b.

## Appendix A Details of Policy Update Algorithm in WebRL

### A.1 Derivation

By substituting eq. [3](https://arxiv.org/html/2411.02337v2#S2.E3 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") into eq. [2](https://arxiv.org/html/2411.02337v2#S2.E2 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), we can obtain:

|  | $\beta\log\frac{\pi^{*}(a_{t}&#124;s_{t},I)}{\pi_{\text{ref}}(a_{t}&#124;s_{t},I)}=r(s_{t% },a_{t},I)+V^{*}(s_{t+1},I)-V^{*}(s_{t})=A^{*}(s_{t},a_{t},I)$ |  | (9) |

|  | $\pi^{*}(a_{t}&#124;s_{t},I)=\pi_{\text{ref}}(a_{t}&#124;s_{t},I)\exp\left(\frac{1}{\beta% }A^{*}(s_{t},a_{t},I)\right)$ |  | (10) |

$\pi^{*}$ is the optimal solution to our target (eq. [1](https://arxiv.org/html/2411.02337v2#S2.E1 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")). If $\pi_{\theta}$ is represented by a function approximator, we aim to make it as close as possible to $\pi^{*}$.

|  | $\begin{split}&\operatorname*{arg\,min}\limits_{\theta}\mathbb{E}_{\nu}\left[% \left(\log\pi_{\theta}(a&#124;s,I)-\log\pi^{*}(a&#124;s,I)\right)^{2}\right]\\ =&\operatorname*{arg\,min}\limits_{\theta}\mathbb{E}_{\nu}\left[\left(\log\pi_% {\theta}(a&#124;s,I)-\log\pi_{\text{ref}}(a&#124;s,I)-\frac{1}{\beta}A^{*}(s,a,I)\right)% ^{2}\right]\\ =&\operatorname*{arg\,min}\limits_{\theta}\mathbb{E}_{\nu}\left[\left(\beta% \log\frac{\pi_{\theta}(a&#124;s,I)}{\pi_{\text{ref}}(a&#124;s,I)}-A^{*}(s,a,I)\right)^{2% }\right]\end{split}$ |  | (11) |

Hence, the loss function can be defined as:

|  | $\mathcal{L}(\pi_{\theta})=\mathbb{E}_{\nu}\left[\left(\beta\log\frac{\pi_{% \theta}(a&#124;s,I)}{\pi_{\text{ref}}(a&#124;s,I)}-A^{*}(s,a,I)\right)^{2}\right]$ |  | (12) |

where $\nu$ represents the distribution of experience used for training. For any given state-action pair, $\pi_{\theta}$ is expected to match the target policy $\pi^{*}(a|s,I)$. There is no restriction on the data distribution $\nu$, indicating the algorithm can function effectively in an off-policy setting. Since $A*$ is unknown, we estimate $A^{*}(s,a,I)$ using eq. [7](https://arxiv.org/html/2411.02337v2#S2.E7 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") and eq. [8](https://arxiv.org/html/2411.02337v2#S2.E8 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") to train the policy $\pi_{\theta}$. The feasibility of this approach is demonstrated in Appendix [A.2](https://arxiv.org/html/2411.02337v2#A1.SS2 "A.2 Proof ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

Further Analysis. Although $\nu$ can follow any distribution, achieving stable policy improvement often requires some control over $\nu$. The primary goal is to enhance the probability of actions that successfully complete the task and to address the deficiencies of the current policy $\pi_{\theta}$. Therefore, $\nu$ typically consists of data sampled from the current policy being trained and prior successful experiences.

Why not use KL divergence to measure the distance between $\pi^{*}$ and $\pi_{\theta}$. For eq. [11](https://arxiv.org/html/2411.02337v2#A1.E11 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), many studies use KL divergence to measure the distance between two policies. When KL divergence is used, the optimization goal of $\theta$ is:

|  | $\begin{split}&\operatorname*{arg\,min}\limits_{\theta}\mathbb{E}_{s\sim d(s)}% \left[D_{\text{KL}}\left(\pi^{*}(\cdot&#124;s,I)&#124;&#124;\pi_{\theta}(\cdot&#124;s,I)\right)% \right]\\ =&\operatorname*{arg\,max}\limits_{\theta}\mathbb{E}_{s\sim d(s)}\mathbb{E}_{a% \sim\pi^{*}(a&#124;s,I)}\left[\log\pi_{\theta}(a&#124;s,I)\right]\\ =&\operatorname*{arg\,max}\limits_{\theta}\mathbb{E}_{s\sim d(s)}\int_{a}\pi_{% \text{ref}}(a&#124;s,I)\exp(\frac{1}{\beta}A^{*}(s,a,I)\log\pi_{\theta}(a&#124;s,I)\text% {d}a\\ =&\operatorname*{arg\,max}\limits_{\theta}\mathbb{E}_{s\sim d(s)}\mathbb{E}_{a% \sim\pi_{\text{ref}}(a&#124;s,I)}\left[\log\pi_{\theta}(a&#124;s,I)\exp(\frac{1}{\beta}A% ^{*}(s,a,I))\right]\end{split}$ |  | (13) |

where $d(s)$ is a distribution of state. The choice of using eq. [11](https://arxiv.org/html/2411.02337v2#A1.E11 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") instead of eq. [13](https://arxiv.org/html/2411.02337v2#A1.E13 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") as the optimization target can be explained as follows: (1) Eq. [13](https://arxiv.org/html/2411.02337v2#A1.E13 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") imposes stronger restrictions on the training data. Specifically, it requires the training data to conform to the distribution of $\pi_{\text{ref}}$.

![Refer to caption](img/7db89439f5e995ba9e26a7d6e5a537f3.png)

Figure 9: Comparison of using eq. [11](https://arxiv.org/html/2411.02337v2#A1.E11 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") and eq. [13](https://arxiv.org/html/2411.02337v2#A1.E13 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") as target.

When $\pi_{\text{ref}}$ fails to capture previous successful experiences, it becomes difficult to sample these experiences from $\pi_{\text{ref}}$, which in turn, can lead to further forgetting of those experiences. (2) Both eq. [11](https://arxiv.org/html/2411.02337v2#A1.E11 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") and eq. [13](https://arxiv.org/html/2411.02337v2#A1.E13 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") aim to approximate the distance between two distributions. The use of Mean Squared Error as a metric to measure the distance between policy distributions is also employed in Fujimoto & Gu ([2021](https://arxiv.org/html/2411.02337v2#bib.bib9)). (3) Eq. [13](https://arxiv.org/html/2411.02337v2#A1.E13 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") is unable to decrease the probability of actions with negative advantage. When correct action is hard to sample from $\pi_{\text{ref}}$, eq. [13](https://arxiv.org/html/2411.02337v2#A1.E13 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") may instead increase the probability of actions with negative advantage, compounding the challenge of sampling correct actions. We also conduct experiments to further validate the superiority of eq. [11](https://arxiv.org/html/2411.02337v2#A1.E11 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), with the results presented in Figure [9](https://arxiv.org/html/2411.02337v2#A1.F9 "Figure 9 ‣ A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). In these experiments, eq. [13](https://arxiv.org/html/2411.02337v2#A1.E13 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") is trained exclusively on data sampled from $\pi_{\text{ref}}$, while eq. [11](https://arxiv.org/html/2411.02337v2#A1.E11 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") utilize both replay buffer data and sampled data.

When discount factor $\gamma$ is used. When considering the discount factor, eq. [9](https://arxiv.org/html/2411.02337v2#A1.E9 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") is modified as follows:

|  | $\beta\log\frac{\pi^{*}(a_{t}&#124;s_{t},I)}{\pi_{\text{ref}}(a_{t}&#124;s_{t},I)}=r(s_{t% },a_{t},I)+\gamma V^{*}(s_{t+1},I)-V^{*}(s_{t})=A^{*}(s_{t},a_{t},I)$ |  | (14) |

This will not affect the subsequent derivation of the policy loss function (eq. [12](https://arxiv.org/html/2411.02337v2#A1.E12 "In A.1 Derivation ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")). The loss function for the critic remains unchanged. However, the equation to calculate the advantage is modified as follows:

|  | $A(s_{t},\!a_{t},\!I)\!=\!\lambda\big{(}r(s_{t},\!a_{t},\!I)+\gamma V(s_{t+1},% \!I)\!-\!V(s_{t},\!I)\big{)}\!+\!(1\!-\!\lambda)\big{(}\gamma^{T-t}r(s_{T},\!a% _{T},\!I)\!-\!V(s_{t},\!I)\big{)}$ |  | (15) |

In our experiment, we use the discount factor and set its value to 0.9.

We use eq. [7](https://arxiv.org/html/2411.02337v2#S2.E7 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") and eq. [8](https://arxiv.org/html/2411.02337v2#S2.E8 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") to estimate $A^{*}(s,a,I)$. The training of critic (eq. [7](https://arxiv.org/html/2411.02337v2#S2.E7 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")) also operates in an off-policy manner.

### A.2 Proof

To prove the feasibility of replacing $A^{*}$ with an advantage estimator $A$ based on a value network $V$, which is trained on data collected throughout the training process, we aim to demonstrate that using $V$ to train the policy can lead to policy improvement.

The current policy is denoted by $\pi_{\text{old}}$. The training dataset $D$ is composed of trajectories sampled using $\pi_{\text{old}}$ as well as historical experiences stored in a replay buffer. The behavioral policy associated with the dataset $D$ is referred to as $\pi_{\mu}$. The corresponding value function and action-value function under $\pi_{\mu}$ are denoted as $V^{\pi_{\mu}}$ and $Q^{\pi_{\mu}}$, respectively. Below, we demonstrate that minimizing the objective $\arg\min_{\theta}\mathbb{E}_{D}[(\log_{\pi_{\theta}}(a_{t}|s_{t})-\exp(\frac{Q% ^{\pi_{\mu}}(s_{t},a_{t})-V^{\pi_{\mu}}(s_{t})}{\beta}))^{2}]$ can lead to policy improvement. For brevity, we omit $I$ in the following proof.

Let the new policy $\pi_{\text{new}}(a_{t}|s_{t})$ be defined as:

|  | $\pi_{\text{new}}(a_{t}&#124;s_{t})=\exp\left(\frac{Q^{\pi_{\mu}}(s_{t},a_{t})-V^{% \pi_{\mu}}(s_{t})}{\beta}\right)$ |  | (16) |

It can be shown that: $D_{\text{KL}}(\pi_{\text{new}}||\pi_{\text{new}})\leq D_{\text{KL}}(\pi_{\mu}|% |\pi_{\text{new}})$. From this, we derive the following inequality:

|  | $\begin{split}&\mathbb{E}_{a_{t}\sim\pi_{\text{new}}}\!\left[\log\pi_{\text{new% }}(a_{t}&#124;s_{t})\!-\!\frac{Q^{\pi_{\mu}}(s_{t},a_{t})}{\beta}\!+\!\frac{V^{\pi_% {\mu}}(s_{t})}{\beta}\right]\\ \leq&\ \mathbb{E}_{a_{t}\sim\pi_{\mu}}\!\left[\log\pi_{\mu}(a_{t}&#124;s_{t})\!-\!% \frac{Q^{\pi_{\mu}}(s_{t},a_{t})}{\beta}\!+\!\frac{V^{\pi_{\mu}}(s_{t})}{\beta% }\right]\end{split}$ |  | (17) |

Since $V^{\pi_{\mu}}(s_{t})$ depends only on the state $s_{t}$ and not on the actions, it can be factored out of the expectation. We can rewrite the inequality as:

|  | $\mathbb{E}_{a_{t}\sim\pi_{\text{new}}}\left[Q^{\pi_{\mu}}(s_{t},a_{t})-\beta% \log\pi_{\text{new}}(a_{t}&#124;s_{t})\right]\geq V^{\pi_{\mu}}(s_{t})$ |  | (18) |

where $V^{\pi_{\mu}}(s_{t})=\mathbb{E}_{a\sim\pi_{\mu}(a_{t}|s_{t})}[Q^{\pi_{\mu}}(s_% {t},a_{t})-\beta\log\pi_{\mu}(a_{t}|s_{t})]$, which is satisfied in maximum entropy reinforcement learning setting (Haarnoja et al., [2018](https://arxiv.org/html/2411.02337v2#bib.bib14)).

Consider the soft Bellman equation:

|  | $\begin{split}Q^{\pi_{\mu}}(s_{t},a_{t})&=r(s_{t}&#124;a_{t})+\beta\log\pi_{\text{% ref}}(a_{t}&#124;s_{t})+V^{\pi_{\mu}}(s_{t+1})\\ &\leq r(s_{t}&#124;a_{t})+\beta\log\pi_{\text{ref}}(a_{t}&#124;s_{t})+\mathbb{E}_{a_{t+1% }\sim\pi_{\text{new}}}[Q^{\pi_{\mu}}(s_{t+1},a_{t+1})-\beta\log\pi_{\text{new}% }(a_{t+1}&#124;s_{t+1})]\\ &\cdots\\ &\leq Q^{\pi_{\text{new}}}(s_{t},a_{t})\end{split}$ |  | (19) |

where we iteratively expand $Q^{\pi_{\mu}}$ by applying the soft Bellman equation and the bound provided in Eq [18](https://arxiv.org/html/2411.02337v2#A1.E18 "In A.2 Proof ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). Since $\pi_{\text{new}}$ is an improved policy over $\pi_{\mu}$ (i.e., $Q^{\pi_{\text{new}}(s_{t},a_{t})}\geq Q^{\pi_{\mu}}(s_{t},a_{t})$), and $\pi_{\theta}$ is trained to closely match $\pi_{\text{new}}$, it follows that $\pi_{\theta}$ will also be an improved policy over $\pi_{\mu}$ provided the approximation is accurate. Formally, we have:

|  | $V^{\pi_{\theta}}(s)\approx V^{\pi_{\text{new}}(s)}\geq V^{\pi_{\mu}}(s)$ |  | (20) |

Since we only incorporate the correct trajectory from the historical data, we can assert that policy $\pi_{\mu}$ will not perform worse than the old policy $\pi_{\text{old}}$. Therefore, we have the relationship:

|  | $V^{\pi_{\theta}}(s)\approx V^{\pi_{\text{new}}}(s)\geq V^{\pi_{\mu}}(s)\geq V^% {\pi_{\text{old}}}(s)$ |  | (21) |

The policy improvement process is well-established. By iteratively performing policy evaluation and policy improvement, the policy $\pi_{\theta}$ can be made to converge toward the optimal policy $\pi^{*}$. If the data in the replay buffer is not utilized, the same theoretical process holds. However, in this case, $V^{\pi_{\mu}}$ and $Q^{\pi_{\mu}}$ are replaced by $V^{\pi_{\text{old}}}$ and $Q^{\pi_{\text{old}}}$.

![Refer to caption](img/cbe759a821fc22bcc9ec8639a6f8d7c4.png)

Figure 10: Comparison of using $V^{\pi_{\mu}}$ and $V^{\pi_{\text{old}}}$.

We conduct an experiment to evaluate the effectiveness of using $V^{\pi_{\mu}}$ instead of $V^{\pi_{\text{old}}}$ in WebRL training. In this experiment, the policy $\pi_{\theta}$ is trained using data from two sources: rollout trajectories during the current phase and past experiences stored in the replay buffer. For the critic, we compare two training settings: training with data from both rollout trajectories and experiences in the replay buffer versus using only rollout trajectories. The results, as presented in Figure [10](https://arxiv.org/html/2411.02337v2#A1.F10 "Figure 10 ‣ A.2 Proof ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), reveal that the model performs better when the critic is trained with data from both the replay buffer and the rollout trajectories. This observation underscores the significance of integrating experiences from prior successful trajectories into the critic’s training process, which can lead to more effective policy improvement.

### A.3 Comparison

The comparison result between the policy update algorithm of WebRL and other algorithms is shown in Table [4](https://arxiv.org/html/2411.02337v2#A1.T4 "Table 4 ‣ A.3 Comparison ‣ Appendix A Details of Policy Update Algorithm in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

Table 4: Comparison between WebRL and different algorithms based on various criteria.

| Algorithm | Off-policy | Pair-wise Data | KL-constrained Target | Reduce Error Prob |
| WebRL | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  | ![[Uncaptioned image]](img/21ffa47607f381d12569a5e3ee1dea90.png)  | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  |
| DPO | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  |
| PPO | ![[Uncaptioned image]](img/21ffa47607f381d12569a5e3ee1dea90.png)  | ![[Uncaptioned image]](img/21ffa47607f381d12569a5e3ee1dea90.png)  | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  |
| AWR | ![[Uncaptioned image]](img/241b6e4bf050bf5329788566d7e82a99.png)  | ![[Uncaptioned image]](img/21ffa47607f381d12569a5e3ee1dea90.png)  | ![[Uncaptioned image]](img/21ffa47607f381d12569a5e3ee1dea90.png)  | ![[Uncaptioned image]](img/21ffa47607f381d12569a5e3ee1dea90.png)  |

Comparison with DPO (Rafailov et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib30)). DPO also solves for the optimal value of eq. [1](https://arxiv.org/html/2411.02337v2#S2.E1 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"), obtaining the relationship between the optimal policy $\pi^{*}$ and the reward $r$. Rather than explicitly solving for the reward $r$, DPO employs contrastive learning by constructing paired data to build the learning target of $\pi_{\theta}$. This approach allows DPO to bypass the need to estimate $r$. In contrast, our approach explicitly fits the value function $V$ to guide the optimization of $\pi_{\theta}$, making it converge towards the optimal policy $\pi^{*}$. However, DPO requires pair-wise data, which is challenging to obtain in web tasks. This is primarily because implementing state backtracking on web pages is difficult, making it hard to collect outcomes of different actions on the same page state.

Comparison with PPO (Schulman et al., [2017](https://arxiv.org/html/2411.02337v2#bib.bib33)). Eq.[1](https://arxiv.org/html/2411.02337v2#S2.E1 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") is also the target of RLHF. Prior work often employs PPO to optimize this objective. However, as an on-policy algorithm, PPO has low sampling efficiency and requires large amounts of data for stable improvement. This limitation makes it unsuitable for environments such as WebArena and real-world websites, where interaction is expensive and inefficient. Instead, our approach operates in an off-policy manner.

Comparison with AWR (Peng et al., [2019](https://arxiv.org/html/2411.02337v2#bib.bib27)). AWR is one of the widely used offline RL algorithms. The loss function of AWR is $\operatorname*{arg\,max}_{\theta}\mathbb{E}_{\nu}\left[\log\pi_{\theta}(a|s,I)% \cdot\exp(A(s,a,I)/\beta\right]$. Compared to our method, AWR has a different optimization goal. Our approach explicitly minimizes the KL divergence between the reference policy $\pi_{\text{ref}}$ and the current policy $\pi_{\theta}$, while AWR focuses on constraining the KL divergence between the current policy and the behavioral policy derived from training data $\nu$ (instead of the reference policy $\pi_{\text{ref}}$). Additionally, AWR does not directly reduce the probability of incorrect actions. In web tasks, sampled data often include unsuccessful traces. Ignoring such data wastes valuable information, while using it risks increasing the likelihood of incorrect actions. In contrast, our method effectively reduces the probability of incorrect actions.

## Appendix B Training Details

### B.1 Details of WebRL

The agent observation consists of three components:

*   $\bullet$

    User instruction: The instruction provided by the user.

*   $\bullet$

    Action history: A record of the actions the agent has previously taken.

*   $\bullet$

    Webpage HTML: HTML content of the current webpage.

We process the HTML, simplifying its structure and assigning distinct element IDs to all clickable elements. This facilitates the model’s ability to identify and indicate which specific element requires manipulation.

The agent actions are mostly similar to those defined in WebArena-Lite (Liu et al., [2024b](https://arxiv.org/html/2411.02337v2#bib.bib22)), with some additional actions.

*   $\bullet$

    Click: Clicks an element with a specific ID.

*   $\bullet$

    Hover: Hovers over an element with a specific ID.

*   $\bullet$

    Type: Types a message into an input box with a specific ID.

*   $\bullet$

    Search: Types a message into an input box with a specific ID and presses Enter to initiate a search.

*   $\bullet$

    Press: Emulates a specific keyboard key combination.

*   $\bullet$

    Scroll: Scrolls the page up or down.

*   $\bullet$

    Select_dropdown_option: Selects an option from a dropdown menu with a specific ID.

*   $\bullet$

    New_tab: Opens a new tab in the current browser.

*   $\bullet$

    Tab_focus: Switches focus to a browser tab at a specified index.

*   $\bullet$

    Close_tab: Closes the current tab.

*   $\bullet$

    Goto: Navigates to a specific URL.

*   $\bullet$

    Go_back: Returns to the previous page.

*   $\bullet$

    Go_forward: Moves to the next page if available.

*   $\bullet$

    Exit: Terminates the operation, returns the response, and exits.

To provide more detailed information about the action, we include comments labeled with “# Element:” in the action, which describe the operated element. Similarly, we include comments labeled “# Note:”, which quote relevant information from the current webpage that supports completing the instruction. An example of the agent’s specific input and output is shown in Figure [11](https://arxiv.org/html/2411.02337v2#A2.F11 "Figure 11 ‣ B.2 Details of Baselines ‣ Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). The input is the observation, while the output specifies the action to be performed on the webpage. The “element” argument identifies the target element for the action. More examples can be found in Appendix [E](https://arxiv.org/html/2411.02337v2#A5 "Appendix E Qualitative Examples ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

The detailed training process of WebRL is shown in Algorithm [1](https://arxiv.org/html/2411.02337v2#alg1 "Algorithm 1 ‣ B.3 Details of ORM ‣ Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). First, we perform supervised fine-tuning using the WebArena-Lite training dataset. We then initialize the replay buffer and failure set by running the SFT-trained model on instructions corresponding to the WebArena-Lite training dataset. Subsequently, in each phase of the self-evolving curriculum reinforcement learning process, 500 new instructions that meet the filtering criteria are selected from those generated by GPT-4o. Both newly generated interaction data on these instructions and historical data with perplexity between 1/0.95 and 1/0.5 from the replay buffer are used to train the actor and critic. The amount of historical data used is limited to twice the size of the interaction data. The hyperparameters employed in WebRL are presented in Table [5](https://arxiv.org/html/2411.02337v2#A2.T5 "Table 5 ‣ B.3 Details of ORM ‣ Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

### B.2 Details of Baselines

For RL-based baselines (except DigiRL), the interaction data from WebRL’s first phase is used, while DigiRL is trained using the first-phase instructions in an online learning setup. Hence, except for DigiRL, the other RL baselines fall under offline reinforcement learning.

We reproduce the same framework used in DigiRL within the WebArena environment. Specifically, we use the same components, including the AWR method, the instruction-level and step-level value functions, and the replay buffer described in DigiRL. To apply DigiRL in WebArena-Lite, the main modifications we make are adjusting the data format to align with WebRL and tweaking certain hyperparameters. DigiRL also conducts 8 rounds of interaction and training. The hyperparameters employed in those baselines are presented in Table [5](https://arxiv.org/html/2411.02337v2#A2.T5 "Table 5 ‣ B.3 Details of ORM ‣ Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning").

![Refer to caption](img/c2f92469d5d959fb7fac35153240c85e.png)

Figure 11: The input and output format of WebRL and baselines, where the input is composed of task instruction (in green), action history (in blue), and HTML of the current webpage (in orange). The output (in red) is the action taken on the current webpage.

### B.3 Details of ORM

WebArena-Lite provides 1,186 training samples each comprising an instruction, a trajectory, and a reward function. To train the $\mathcal{M}_{\text{ORM}}$, we first enhance the WebArena-Lite training dataset by rewriting the instructions. Then, we collect rollouts from all baseline methods on this augmented dataset. These rollouts, combined with the original WebArena-Lite training samples, form the complete training dataset for the $\mathcal{M}_{\text{ORM}}$, with a total of 12,200 samples. We use the associated reward function to label these trajectories, identifying whether they successfully complete the task. These trajectories are subsequently used to train the ORM, with the specific hyperparameters listed in Table [6](https://arxiv.org/html/2411.02337v2#A2.T6 "Table 6 ‣ B.3 Details of ORM ‣ Appendix B Training Details ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). The prompt for $\mathcal{M}_{\text{ORM}}$ is shown in Figure [17](https://arxiv.org/html/2411.02337v2#A4.F17 "Figure 17 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). $\mathcal{M}_{\text{ORM}}$ is required to produce either “YES” or “NO” as its output. To determine the evaluation result, we compare the probabilities assigned to “YES” and “NO” and select the one with the higher probability. We need to train $\mathcal{M}_{\text{ORM}}$ because new tasks that do not have predefined reward functions will be generated in the subsequent self-evolving curriculum reinforcement learning process. We rely on the trained $\mathcal{M}_{\text{ORM}}$ to judge whether each task is successfully completed.

Algorithm 1 WebRL

Input: WebArena-Lite training set $D_{0}$ and corresponding instruction set $I_{0}$, Base model $\mathcal{M}_{\text{base}}$, Replay buffer $\mathcal{B}$, Failure set $\mathcal{F}$, Reward model $\mathcal{M}_{\text{ORM}}$, Number of phases $N$, Number of instructions per phase $K$

Part 1\. SFT Training

1:  $\mathcal{M}_{\text{SFT}}$ $\longleftarrow$ perform supervised fine-tuning on model $\mathcal{M}_{\text{base}}$ with dataset $D_{0}$2:  $D_{\text{rollout}}$ $\longleftarrow$ rollout trajectories of $\mathcal{M}_{\text{SFT}}$ on $I_{0}$3:  $D_{\text{success}}$, $D_{\text{fail}}$ $\longleftarrow$ evaluate $D_{\text{rollout}}$ using WebArena-Lite reward function4:  $\mathcal{B}$ $\longleftarrow$ $D_{\text{success}}$ # Initialize replay buffer with successful trajectories5:  $\mathcal{F}$ $\longleftarrow$ $D_{\text{fail}}$ # Initialize failure set with instructions of failing trajectoriesPart 2\. Self-evolving Curriculum RL6:  $\pi_{1}$ $\longleftarrow$ $\mathcal{M}_{\text{SFT}}$ # Initialize actor/policy7:  $V_{1}$ $\longleftarrow$ $\mathcal{M}_{\text{SFT}}$ with a randomly initialized value head # Initialize critic8:  for $n$ in 1…$N$ do9:     $I_{n}$ $\longleftarrow$ {}10:     while size($I_{n}$) $\leq$ $K$ do11:        $I_{\text{generation}}$ $\longleftarrow$ instructions generated by GPT-4o with instructions from $\mathcal{F}$ as examples # instruction generation process12:        $I_{\text{filter}}$ $\longleftarrow$ filter($I_{\text{generation}}$) # instruction filtering process13:        $I_{n}$ $\longleftarrow$ $I_{n}\cup I_{\text{filter}}$14:     end while15:     $D_{\text{rollout}}$ $\longleftarrow$ rollout trajectories of $\pi_{n}$ on $I_{n}$16:     $D_{\text{success}}$, $D_{\text{fail}}$ $\longleftarrow$ evaluate $D_{\text{rollout}}$ with $\mathcal{M}_{\text{ORM}}$ # use $\mathcal{M}_{\text{ORM}}$ to label the rollout trajectories17:     $D_{\text{experience}}$ $\longleftarrow$ experiences from $\mathcal{B}$ with perplexity computed by $\pi_{n}$ between $\frac{1}{0.95}$ and $\frac{1}{0.5}$18:     $\pi_{n+1}$,​ $V_{n+1}$ $\longleftarrow$ train($\pi_{n}$, $V_{n}$, $D_{\text{rollout}}\cup D_{\text{experience}}$) # use loss functions from eq. [4](https://arxiv.org/html/2411.02337v2#S2.E4 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") and eq. [7](https://arxiv.org/html/2411.02337v2#S2.E7 "In 2.2 Reinforcement Learning for LLMs in Online Web Environments ‣ 2 WebRL: Self-Evolving Online Curriculum RL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")19:     $\mathcal{B}$ $\longleftarrow$ $\mathcal{B}\cup D_{\text{success}}$20:     $\mathcal{F}$ $\longleftarrow$ $\mathcal{F}\cup D_{\text{fail}}$21:  end for

Table 5: The hyperparameters we employ in WebRL and baselines.

| Method | Hyperparameter | Value |
| SFT | learning rate | 1e-5 |
| lr scheduler type | cosine |
| warmup ratio | 0.1 |
| batch size | 128 |
| training epoch | 1 |
| cutoff length | 16384 |
| Filtered BC | learning rate | 1e-6 |
| lr scheduler type | constant |
| batch size | 128 |
| training epoch | 1 |
| cutoff length | 16384 |
| filtering threshold | 70th percentile |
| AWR | actor learning rate | 1e-6 |
| actor lr scheduler type | constant |
| critic learning rate | 1e-6 |
| critic lr scheduler type | constant |
| batch size | 128 |
| discount factor | 0.9 |
| actor training epoch | 1 |
| critic training epoch | 1 |
| DigiRL | actor learning rate | 1e-6 |
| actor lr scheduler type | constant |
| critic learning rate | 1e-6 |
| critic lr scheduler type | constant |
| instruction value function lr | 1e-6 |
| instruction value function lr scheduler type | constant |
| batch size | 128 |
| discount factor | 0.9 |
| actor training epoch | 1 |
| critic training epoch | 1 |
| instruction value function epoch | 1 |
| rollout temperature | 1 |
| replay buffer size | 100000 |
| WebRL | actor learning rate | 1e-6 |
| actor lr scheduler type | constant |
| critic learning rate | 1e-6 |
| critic lr scheduler type | constant |
| batch size | 128 |
| discount factor | 0.9 |
| actor training epoch | 1 |
| critic training epoch | 1 |
| rollout temperature | 1 |

Table 6: The hyperparameters we employ to train the ORM.

| Hyperparameter | Value |
| --- | --- |
| learning rate | 5e-6 |
| lr scheduler type | cosine |
| warmup ratio | 0.1 |
| batch size | 128 |
| training epoch | 4 |
| cutoff length | 16384 |

## Appendix C Other Quantitative Experiments

Figure [12](https://arxiv.org/html/2411.02337v2#A3.F12 "Figure 12 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") illustrates the performance variation curves of Llama3.1-8B trained with WebRL on each website. It can be seen that in all the sites except for Map, there is a clear upward trend. However, in the case of Map, there is an initial upward trend followed by a decline. We hypothesize that the final decline is caused by a significant increase in OSS and CMS implementation, which creates a trade-off. This trade-off leads to a performance drop in Map and a slight decline in GitLab.

Figure [14](https://arxiv.org/html/2411.02337v2#A3.F14 "Figure 14 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") illustrates the error bars for WebRL across each phase. We conduct four sets of experiments, each utilizing a distinct random seed. The results from these four sets of experiments are presented in Figure [14](https://arxiv.org/html/2411.02337v2#A3.F14 "Figure 14 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). Despite some fluctuations in accuracy, WebRL consistently demonstrates an overall upward trend. Furthermore, its accuracy consistently surpasses all baseline methods after completing the final phase of training.

Figure [14](https://arxiv.org/html/2411.02337v2#A3.F14 "Figure 14 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") presents the success rate of WebRL and DigiRL, both with and without the application of our self-evolving curriculum learning. The results indicate that incorporating curriculum learning significantly enhances the performance of DigiRL. However, its performance still remains below that of WebRL. Conversely, when WebRL is implemented without curriculum learning, its performance experiences a notable decline, yet it still slightly surpasses that of DigiRL.

Figure [16](https://arxiv.org/html/2411.02337v2#A3.F16 "Figure 16 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") illustrates the error bars for both the baselines, DigiRL w/ CL, and WebRL. Notably, the performance of WebRL and DigiRL with curriculum learning (CL) shows a higher standard deviation, which can be attributed to the inherent randomness in task generation. Despite this variability, WebRL consistently outperforms all baseline methods, demonstrating its effectiveness.

Figure [16](https://arxiv.org/html/2411.02337v2#A3.F16 "Figure 16 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") compares two settings of the replay buffer: one containing only past successful trajectories and the other including all past trajectories, both successful and failed. The results show that including failed trajectories leads to slower and more volatile performance improvement. This issue arises because the advantage of an action in historical trajectories is estimated based on the trajectory’s final reward. Including failed historical trajectories introduces a potential problem: even if certain actions in a failed trajectory are correct, they are likely to receive a negative advantage when the trajectory’s final reward is zero. This reduces the likelihood of selecting these correct actions, thereby hindering effective policy improvement and leading to degraded performance.

Table 7: Performance of different prompts for task generation.

| Prompt 1 | Prompt 2 | Prompt 3 |
| 42.4 | 40.6 | 43.6 |

We employ a variety of prompts to assess the stability of the curriculum learning strategy in WebRL. Specifically, we utilize three distinct prompts: Prompt 1 (in Figure [18](https://arxiv.org/html/2411.02337v2#A4.F18 "Figure 18 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")), Prompt 2 (in Figure [19](https://arxiv.org/html/2411.02337v2#A4.F19 "Figure 19 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")), and Prompt 3 (in Figure [20](https://arxiv.org/html/2411.02337v2#A4.F20 "Figure 20 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning")). The training results for these prompts, measured after eight phases of training, are presented in Table [7](https://arxiv.org/html/2411.02337v2#A3.T7 "Table 7 ‣ Appendix C Other Quantitative Experiments ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). The results demonstrate that when using a variety of prompts to generate tasks, WebRL consistently delivers strong performance, highlighting the stability of our method.

![Refer to caption](img/ccb0ada101ffaeea4118c55aa25552b4.png)

Figure 12: Performance variation curves of Llama3.1-8B on each website under WebRL training

![Refer to caption](img/2868ea55fab9555e55de28c001312756.png)

Figure 13: Error bars of WebRL for each phase.

![Refer to caption](img/aab8c0cb6acc3ee4e25c8d89d936d279.png)

Figure 14: Comparison of WebRL and DigiRL with and without curriculum learning (CL).

![Refer to caption](img/912f0c712aa5781fbd440d1aedec8cc4.png)

Figure 15: Error bars of baselines, DigiRL w/ CL, and WebRL.

![Refer to caption](img/26561b23cf8ae60e20c40370e8686e17.png)

Figure 16: Comparison of WebRL and WebRL with failed trajectories in the replay buffer.

## Appendix D Prompts Employed in WebRL

The prompt used for $\mathcal{M}_{\text{ORM}}$ is shown in Figure [17](https://arxiv.org/html/2411.02337v2#A4.F17 "Figure 17 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). We require the model to output “YES” or “NO” to determine whether a certain trajectory successfully completes the corresponding task. Considering the limited context size, we only input the HTML content of the last state.

![Refer to caption](img/a98ee29bd6df278f91eed9fa1e42009f.png)

Figure 17: Prompts for $\mathcal{M}_{\text{ORM}}$ to assess the completion of Instructions.

The prompt for generating new instructions is presented in Figure [18](https://arxiv.org/html/2411.02337v2#A4.F18 "Figure 18 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). We use tasks that the model fails to complete successfully in previous phases as seeds for generating new tasks of similar difficulty but with greater variety. The prompts shown in Figure [19](https://arxiv.org/html/2411.02337v2#A4.F19 "Figure 19 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") and Figure [20](https://arxiv.org/html/2411.02337v2#A4.F20 "Figure 20 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning") represent alternative versions of the prompts used for task generation.

![Refer to caption](img/3ac3c23c29ad43a99bf4cd90c3be330c.png)

Figure 18: Prompts for instruction generation.

![Refer to caption](img/550e02fb87f1de17beb0417c9905623a.png)

Figure 19: Prompts for instruction generation (version 2).

![Refer to caption](img/488a59bc9e8033ddbc2a44b05ba0c77a.png)

Figure 20: Prompts for instruction generation (version 3).

The simple prompt we use to test models including GPT-4-Turbo, GPT-4o, Llama3.1-8B-Instruct, and Llama3.1-70B-Instruct is shown in Figure [21](https://arxiv.org/html/2411.02337v2#A4.F21 "Figure 21 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). In this prompt, we define the feasible actions and provide illustrative examples. Additionally, we outline a set of requirements in the “REMEMBER” part to guide the model’s behavior.

![Refer to caption](img/0d25509281a327c12ad458069c051294.png)

Figure 21: The simple prompt employed in baselines.

The prompt we use to filter the instructions generated by GPT-4o is presented in Figure [22](https://arxiv.org/html/2411.02337v2#A4.F22 "Figure 22 ‣ Appendix D Prompts Employed in WebRL ‣ WebRL: Training LLM Web Agents via Self-Evolving Online Curriculum Reinforcement Learning"). In this prompt, we provide a rule defining the feasibility of instructions across the five websites in WebArena.

![Refer to caption](img/7bae87f94a0d3615a78214ceaf11172e.png)

Figure 22: The prompt used to filter instructions.

## Appendix E Qualitative Examples

We list one example of WebRL on each of the five sites in WebArena.

![Refer to caption](img/948dcbcce8a8cab790b712afafcd4e29.png)

Figure 23: CMS Example.

![Refer to caption](img/f598eabaa46ed29bfdee828a8fecc493.png)

Figure 24: Gitlab Example.

![Refer to caption](img/bcb787d5b93aa6a192ae4455e1accbd0.png)

Figure 25: MAP Example.

![Refer to caption](img/f0ac971e6308e1e411e11ed212fd936c.png)

Figure 26: Reddit Example.

![Refer to caption](img/eec772db225ecb6f62d214a74be6f168.png)

Figure 27: OSS Example.