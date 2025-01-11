<!--yml
category: 未分类
date: 2025-01-11 12:05:51
-->

# Enhancing LLM Agents for Code Generation with Possibility and Pass-rate Prioritized Experience Replay

> 来源：[https://arxiv.org/html/2410.12236/](https://arxiv.org/html/2410.12236/)

Yuyang Chen^($\dagger$1,2), Kaiyan Zhao^($\dagger$1), Yiming Wang³, Ming Yang³, Jian Zhang¹, Xiaoguang Niu¹

###### Abstract

Nowadays transformer-based Large Language Models (LLM) for code generation tasks usually apply sampling and filtering pipelines. Due to the sparse reward problem in code generation tasks caused by one-token incorrectness, transformer-based models will sample redundant programs till they find a correct one, leading to low efficiency. To overcome the challenge, we incorporate Experience Replay (ER) in the fine-tuning phase, where codes and programs produced are stored and will be replayed to give the LLM agent a chance to learn from past experiences. Based on the spirit of ER, we introduce a novel approach called BTP pipeline which consists of three phases: beam search sampling, testing phase, and prioritized experience replay phase. The approach makes use of failed programs collected by code models and replays programs with high Possibility and Pass-rate Prioritized value (P2Value) from the replay buffer to improve efficiency. P2Value comprehensively considers the possibility of transformers’ output and pass rate and can make use of the redundant resources caused by the problem that most programs collected by LLMs fail to pass any tests. We empirically apply our approach in several LLMs, demonstrating that it enhances their performance in code generation tasks and surpasses existing baselines.

## 1 Introduction

In recent years, there has been significant progress in the development of Large Language Models (LLMs) like Transformer (Vaswani et al. [2017](https://arxiv.org/html/2410.12236v1#bib.bib27)) and Llama (Touvron et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib26)) across various domains. A particular trend has emerged in leveraging LLMs for automatic code generation tasks. Models such as WizardCode (Luo et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib18)) and StarCode (Li et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib14)) have been developed to address these tasks. To evaluate the effectiveness and performance of LLMs in code generation, various benchmarks have been established. For instance, APPS (Hendrycks et al. [2021](https://arxiv.org/html/2410.12236v1#bib.bib11)) is widely used in evaluations of code models, and Code-Contests (Li et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib15)) has been established as a standard for competition-level coding tasks. Among all models, transformers have demonstrated significant success in benchmarking tasks such as code translation, code completion, and challenging problem-solving (Svyatkovskiy et al. [2020](https://arxiv.org/html/2410.12236v1#bib.bib25)). Some transformer-based pipelines have even achieved remarkable results on difficult tasks (Zhang et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib34)).

However, traditional transformer-based pipelines, which consist of sampling and filtering phases, have obvious shortcomings due to their structure. A salient problem in code generation tasks is the significant waste of redundant resources caused by low efficiency. Specifically, when tasks are provided as input, the code agent samples a large number of programs from pre-trained transformer-based LLMs and passes them through public test sets, where they are tested and filtered based on their pass rates. Low efficiency arises in cases where most programs fail to pass the tests due to even a single incorrect token (Zhang et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib34)). As a result, models must sample many incorrect programs to find a precisely accurate one, leading to the wastage of redundant resources, including unsuccessful programs that are not reused.

However, programs that fail some tests do not necessarily lack value. On the contrary, most pre-trained LLMs are well-trained on large corpora, which means that the programs they generate are almost accurate but may fail due to minor errors. Thus, it would save time and improve efficiency if we could reduce the waste of these valuable resources.

To leverage the value hidden in these redundant resources and increase efficiency, we introduce Experience Replay (ER), a buffer that stores programs sampled by LLMs along with each program’s P2Value (possibility and pass rate value), which we consider as its value. P2Value comprehensively considers both the likelihood of a transformer’s output and the pass rate. On one hand, a program with a higher pass rate in public test sets demonstrates good performance in a particular task; on the other hand, a program with a higher likelihood of output is considered to have higher value according to the results calculated by pre-trained transformers. Based on ER, we introduce a novel approach called the BTP pipeline, which consists of three phases: beam search sampling, testing, and prioritized experience replay. The core algorithm, PPER (P2Value-Prioritized Experience Replay), utilizes beam search to sample and store programs in ER, and then replays programs in ER based on a probability dependent on their P2Value.

We empirically demonstrate that our pipeline improves the performance of LLMs in code generation tasks, outperforming the original models regardless of whether the training data is self-generated or generated by models of higher quality. More specifically, our contributions are as follows:

*   •

    First, we propose a novel algorithm, BTP pipeline, consisting of beam search sampling phase testing phase and experience replay phase to fine-tune LLMs.

*   •

    We empirically demonstrate that our algorithm performs well not only in scenarios where better LLMs generate data to fine-tune normal LLMs, but also in scenarios where LLMs sample programs to fine-tune themselves. That is to say, our BTP pipeline is generic.

*   •

    At last, we introduce a novel buffer called experience replay buffer which is efficient in fine-tuning. In the future, we can combine it with other algorithms to find more efficient way to fine-tune LLMs.

## 2 Related Work

Considering the association of ER and LLMs in our pipeline, here we will introduce them respectively.

LLMs for code generation: Our work is closely related to LLMs for code generation. In recent years, high-performing LLMs such as GPT-4 (OpenAI [2023](https://arxiv.org/html/2410.12236v1#bib.bib19)), Llama (Touvron et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib26)), PaLM (Chowdhery et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib6)), and Chinchilla (Hoffmann et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib12)) have emerged in different areas. Particularly, our work is based on transformers (Vaswani et al. [2017](https://arxiv.org/html/2410.12236v1#bib.bib27)) for code generation tasks (Roziere et al. [2020](https://arxiv.org/html/2410.12236v1#bib.bib22)). Code models such as BERT (Devlin et al. [2019](https://arxiv.org/html/2410.12236v1#bib.bib7); Feng et al. [2020](https://arxiv.org/html/2410.12236v1#bib.bib9); Guo et al. [2020](https://arxiv.org/html/2410.12236v1#bib.bib10)), T5 (Raffel et al. [2020](https://arxiv.org/html/2410.12236v1#bib.bib21)), GPT-2 (Radford et al. [2019](https://arxiv.org/html/2410.12236v1#bib.bib20)), Codex (Chen et al. [2021a](https://arxiv.org/html/2410.12236v1#bib.bib4)), CodeT5 (Wang et al. [2021](https://arxiv.org/html/2410.12236v1#bib.bib29)), StarCode (Li et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib14)), and WizardCoder (Luo et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib18)) have become backbones for code understanding and generation. Meanwhile, to evaluate the performance of code models, different benchmarks have been created, such as APPS (Hendrycks et al. [2021](https://arxiv.org/html/2410.12236v1#bib.bib11)), CODE-CONTESTS (Li et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib15)), OpenOrca (Lian et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib16)), and HumanEval (Chen et al. [2021b](https://arxiv.org/html/2410.12236v1#bib.bib5)). Additionally, (Roziere et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib23)) constructs training datasets for unsupervised code translation tasks, and (Ellis et al. [2019](https://arxiv.org/html/2410.12236v1#bib.bib8)) constructs test cases from different specific areas to train an RL agent. Moreover, many new methods have been proposed in code generation. For example, (Chen et al. [2021b](https://arxiv.org/html/2410.12236v1#bib.bib5)) fine-tunes powerful pre-trained LLMs to index knowledge and refine performance in code completion. (Austin et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib2)) summarizes that LLMs can be applied to code generation tasks, and (Wei et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib30)) introduces Chain-of-Thought (CoT) prompting to encourage LLMs to think step by step and reduce error rates.

RL for Code Generation: (Bunel et al. [2018](https://arxiv.org/html/2410.12236v1#bib.bib3)) claims that code generation tasks can be broken down into a series of decision-making problems, which are similar to problem definitions in RL. This implies that RL algorithms can be applied to transformers, effectively leveraging their sequential decision-making capabilities. For instance, (Zhang et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib34)) combine Monte Carlo Tree Search (MCTS) with transformers in code generation tasks. In this approach, MCTS is used to explore potential sequences of code by simulating different code paths, evaluating them based on a reward function that measures code correctness and efficiency. This enables the model to select the most promising code sequences during inference(Yang et al. [2024a](https://arxiv.org/html/2410.12236v1#bib.bib32)). Similarly, (Le et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib13)) optimize the correctness of generated programs by framing it as a reward maximization problem, a common objective in RL(Yang et al. [2024b](https://arxiv.org/html/2410.12236v1#bib.bib33)). They employ policy gradient methods, where the transformer model’s policy is iteratively improved by sampling code sequences, estimating the reward (e.g., program correctness)(Wang et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib28)), and updating the model to increase the likelihood of generating correct code sequences in future iterations. These approaches demonstrate how RL’s exploration-exploitation(Yang et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib31)) mechanisms can be integrated with transformers to enhance the performance of code generation models by not just predicting the next token but optimizing over entire sequences based on cumulative rewards.

Experience Replay: (Lin [1992](https://arxiv.org/html/2410.12236v1#bib.bib17)) first introduced the concept of experience replay, suggesting that an agent can store its experiences in a buffer and later sample from this buffer to break the temporal correlation of consecutive observations, thus stabilizing the learning process. By replaying these experiences multiple times, the agent can improve sample efficiency, as it can learn from past experiences that may have been missed during the initial training phase.

Hindsight Experience Replay (HER) (Andrychowicz et al. [2017](https://arxiv.org/html/2410.12236v1#bib.bib1)) expanded on this by addressing the sparse reward problem in goal-conditioned reinforcement learning (RL). In HER, after a failed episode, the transitions leading up to the failure are stored in the replay buffer. During training, these transitions are re-labeled with different goals than originally intended, particularly goals that were actually achieved in the failed episode. By assigning new rewards corresponding to these new goals, HER enables the agent to learn from episodes where the original goal was not achieved, effectively turning failures into learning opportunities.

Meanwhile, Prioritized Experience Replay (PER) (Schaul et al. [2016](https://arxiv.org/html/2410.12236v1#bib.bib24)) introduced the idea that not all transitions are equally valuable for learning. PER modifies the experience replay mechanism by sampling transitions with a probability proportional to their temporal-difference (TD) error, which indicates how surprising or unexpected the transition is. High TD-error transitions, where the agent’s prediction was significantly different from the actual outcome, are more informative and thus are replayed more frequently. This approach ensures that the agent focuses on learning from the most informative experiences, speeding up the convergence of the learning process by reducing the time spent on less useful transitions.

## 3 Methodology

![Refer to caption](img/f04ae86dc49c9284bf9b21c08af76e4d.png)

Figure 1: The pass rate of GPT2-Wizard using BTP in different $\alpha$

In this section, we first briefly present the framework of the BTP pipeline, which is composed of beam search sampling, testing, and PPER. Then, we illustrate the details of the proposed framework. Figure 1 provides a complete process of the BTP pipeline.

### 3.1 BTP pipeline

As shown in Figure 2, most previous works adopt a traditional framework where the transformer model utilizes a sampling and filtering pipeline. In the sampling phase, LLM simply finds best code sequences in every time step according to

|  | $\displaystyle P(Y&#124;X)$ | $\displaystyle=P(y_{1}&#124;X)P(y_{2}&#124;X,y_{1})\dots$ |  | (1) |
|  |  | $\displaystyle\quad P(y_{T}&#124;X,y_{1},y_{2},\dots,y_{T-1})$ |  |

where X is the input, $y_{i}$ is the token generated in time step i, $P(y_{i}|X,y_{1},y_{2},\dots,y_{i-1})$ is the conditional probability of $y_{i}$ given X, $y_{1}$,…,$y_{i-1}$.

Particularly, in code generation tasks, X is a code task in the formulation of prompt.Traditional LLM simply uses greedy algorithm and maximizes $P(y_{i}|X,y_{1},y_{2},\dots,y_{i-1})$ for every time step i till gets a complete code sequence $y_{1}y_{2}\dots y_{T}$

In the filtering phase, the code sequence is sent to public test sets. However, it may fail due to some token mistakes. One main reason is that the greedy algorithm ignores the values hidden in other sequences with lower probabilities. Therefore, we introduce beam search sampling in our pipeline.

### 3.2 Beam search sampling phase

As shown in Figure 4, at every time step, the code model finds the top-k most probable candidate sequences and keeps them in a container called ”beams.” At each step i, all candidate sequences explore all possible tokens from the vocabulary, generating different new sequences. Then, the LLM selects the top-k sequences based on the combined probability and adds them to the new beam for the next time step.

|  | $P(y_{1}y_{2}\dots y_{i})=P(y_{1}y_{2}\dots y_{i-1})P(y_{i})$ |  | (2) |

Repeat the process till the model finds a complete program $y_{1}y_{2}\dots y_{T}$where T is the end time step. Store the top-k programs $t_{1}t_{2}\dots t_{k}$ in the experience replay buffer along with their possibilities $P(t_{1})P(t_{2})\dots P(t_{k})$, the code task X and its corresponding test sets S in the following tuple form:

|  | $T=(X,S,t_{i},P(t_{i}))$ |  | (3) |

### 3.3 Testing phase

In the testing phase, the model will sequentially take out every tuple from T and test $t_{i}$ in every test case of test set S, and compute the pass rate $pass\_rate_{i}$:

|  | $\text{pass\_rate}_{i}=\sum_{forS_{k}\in S}\mathbf{1}(\text{if }t_{i}\text{ % pass }S_{k})$ |  | (4) |

The pass rate, together with the combined probability, will be stored in the experience replay buffer (ER).

|  | $ER_{i}=(X,S,t_{i},P(t_{i}),pass\_rate_{i})$ |  | (5) |

### 3.4 PPER phase

In the Possibility and Pass-rate Prioritized Experience Replay (PPER) phase, the code model will be fine-tuned using the method we call PPER. Specifically, the programs stored in the replay buffer will be sampled with probabilities that are associated with their possibility and pass rate. The sampled programs will then be used to construct a minibatch, which will be used to fine-tune the code model.

#### P2Value

In our PPER method, the most important factor is to establish a standard for defining the priorities of every program in the ER. While it is challenging to determine an accurate measurement standard for priority, a reasonable alternative is to consider the P2Value, which combines the output probability of the transformer and the pass rate on test sets. Particularly for any tuple $ER_{i}=(X,S,t_{i},P(t_{i}),pass\_rate_{i})$ sampled from ER, P2value is calculated as followed:

|  | $\text{P2Value}=\alpha\cdot P(t_{i})+(1-\alpha)\cdot pass\_rate_{i}$ |  | (6) |

Where $\alpha$ is a parameter that determines the weights of possibility and pass rate.The closer it gets to 1, the more important the possibility becomes. Correspondingly, sampled programs that the original code model prefers will have more influence in the fine-tuning process. Conversely, programs that pass the most test sets but are not as highly preferred by the original code model will carry more weight in the fine-tuning process.

The reason we consider such a formula is that programs with higher pass rates are more suitable and valuable for particular code tasks. However, due to the possibility of low pass rates, which can even approach zero, we consider applying possibility to value a program. It is evident that a program preferred by the pre-trained LLM holds higher value in the LLM’s corpus. And how to balance their weights is the reason why we set parameter $\alpha$

#### Random Proprotization sampling

It is straightforward to uniformly sample programs from the ER or to fine-tune the LLM using the entire ER. However, it is more efficient to give programs with higher value a greater chance of being selected. Therefore, we introduce a random sampling method to ensure that every program stored in the ER is sampled in a strictly monotonic manner with respect to its priority. This method increases the likelihood of sampling programs with higher priority, while still maintaining a fixed non-zero probability of sampling the program with the lowest priority, ensuring that every trajectory in the ER is utilized.

Specifically, we define the sampling probability of a transition i in Equation 5.

|  | $P(i)=\frac{p_{i}^{\alpha}}{\sum_{k}p_{k}^{\alpha}}$ |  | (7) |

where pi is the priority of program $t_{i}$. The index $\alpha$ determines the level of prioritization, with $\alpha$ = 0 We consider two ways to define pi. In the first case, we directly define $p_{i}$ as P2Value. It intuitively depicts the relationship between sampling possibility and priority.

However, this method is sensitive to points that deviate significantly from the average value. For instance, trajectories with much higher P2Value will be sampled too frequently.

To solve this problem, we introduce the second definition

|  | $p_{i}=\frac{1}{\text{rank}(i)}$ |  | (8) |

where rank(i) represents the rank of the program’s priority among all trajectories. This method has several advantages compared with the previous approach. Firstly, it follows a power law distribution, meaning that most data are concentrated around the centroid, while a small proportion is distributed around the very large and very small values. Moreover, it is more robust and less sensitive to points that deviate significantly from the average value. For instance, P(i) of a trajectory with the lowest rank will not vary significantly even if its P2Value decreases substantially.

It is noteworthy that the possibility of the code model’s output is non-zero, which means that P2Value is also non-zero, so there is no need for a constant to prevent zero probability.

Algorithm 1 BTP Pipeline

1:$T$: Code model; $beam$: a buffer that stores programs in the beam search sampling phase; $k$: size of beam; $X_{\text{set}}$: task sets with test sets; $ER$: experience replay buffer; $batch$: a minibatch that stockpiles programs used to fine-tune $T$; $n$: size of minibatch2:Beam Search Sampling Phase3:for each $(X,S)$ in $X_{\text{set}}$ do4:     $beam\leftarrow\text{Beam\_search}(X,k)$5:     for each $(t,P)$ in $beam$ do6:         Store $(X,S,t,P)$ in $ER$7:     end for8:end for9:Test Phase10:for each $(X,S,t,P)$ in $ER$ do11:     Test $t$ in $S$ and get $pass\_rate$12:     Replace $(X,S,t,P)$ with $(X,S,t,P,pass\_rate)$13:end for14:PPER Phase15:for $i\leftarrow 1$ to $n$ do16:     Sample $(X,S,t,P,pass\_rate)$ from $ER$ with probability according to P2Value17:     Store $(X,S,t)$ in $batch$18:end for19:Fine-tune $T$ with $batch$

## 4 Experiments

 |  | Pass Rate (%) | Accuracy rate(%) |
| --- | --- | --- |
|  | APPS Intro. | APPS Inter. | APPS comp. | APPS mixed | APPS Intro. | APPS Inter. | APPS comp. | APPS mixed |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| APPS GPT-2 |  |  |  |  |  |  |  |  |
| GPT-2 | 12.37 | 10.67 | 4.33 | 7.30 | 5.30 | 3.30 | 1.32 | 2.72 |
| GPT-2-GPT4 | 50.79 | 43.27 | 37.68 | 40.91 | 25.84 | 19.87 | 15.34 | 20.21 |
| GPT-2-GPT3.5 | 41.68 | 37.62 | 28.59 | 32.25 | 19.42 | 16.25 | 12.21 | 15.63 |
| GPT-2-Llama | 35.82 | 31.14 | 24.37 | 27.60 | 14.35 | 11.40 | 8.27 | 11.69 |
| GPT-2-Wizard | 33.76 | 29.08 | 22.31 | 25.54 | 12.29 | 9.34 | 6.21 | 9.63 |
| APPS GPT-Neo |  |  |  |  |  |  |  |  |
| GPT-Neo | 14.32 | 9.80 | 6.39 | 5.73 | 6.70 | 2.00 | 2.10 | 3.31 |
| GPT-Neo-GPT4 | 51.23 | 46.39 | 38.89 | 42.12 | 26.88 | 23.34 | 17.12 | 20.54 |
| GPT-Neo-GPT3.5 | 39.23 | 34.39 | 26.89 | 30.12 | 14.88 | 11.34 | 5.12 | 8.54 |
| GPT-Neo-Llama | 38.24 | 33.40 | 25.90 | 29.13 | 13.89 | 10.35 | 4.13 | 7.55 |
| GPT-Neo-Wizard | 35.83 | 30.99 | 23.49 | 26.72 | 11.48 | 8.54 | 2.32 | 5.74 | 

Table 1: Result of ”Better models help fine-tune normal models” experiment. On the top and bottom of the table, we show the performance of GPT-2 and GPT-Neo, and how they perform after they are fine-tuned by programs sampled by better models including GPT-4-turbo, GPT-3.5-turbo, CodeLlama- 34B, WizardCoder-34B

In this section, we empirically measure the effectiveness of our BTP pipeline. We conduct experiments sequentially to verify the following conjectures.

 |  | Pass Rate (%) | Accuracy rate(%) |
| --- | --- | --- |
|  | APPS Intro. | APPS Inter. | APPS comp. | APPS mixed | APPS Intro. | APPS Inter. | APPS comp. | APPS mixed |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-2 | 12.37 | 10.67 | 4.33 | 7.30 | 5.30 | 3.30 | 1.32 | 2.72 |
| GPT-2-2 | 15.36 | 11.23 | 5.11 | 8.12 | 4.25 | 2.78 | 2.25 | 3.92 |
| GPT-Neo | 14.32 | 9.80 | 6.39 | 5.73 | 6.70 | 2.00 | 2.10 | 3.31 |
| GPT-Neo-Neo | 14.02 | 9.23 | 7.25 | 5.39 | 6.32 | 2.13 | 2.92 | 3.72 |
| WizardCoder | 45.24 | 41.56 | 37.46 | 40.24 | 20.13 | 17.92 | 14.92 | 16.29 |
| WizardCoder-Wizard | 45.25 | 40.23 | 35.36 | 41.25 | 22.25 | 15.31 | 13.92 | 17.55 | 

Table 2: Result of ”Models help fine-tune themselves” experiment. we show the performance of GPT-2, GPT-Neo, WizardCoder and how they perform after they are fine-tuned by programs sampled by themselves

1: Our BTP pipeline helps code models generate better programs in the scenario where programs sampled from a better model are used to fine-tune a standard model.

2: Our BTP pipeline helps code models generate better programs in the scenario where programs sampled from the code model itself are used to fine-tune the model.

3: The best code model fine-tuned by our BTP pipeline is competitive compared to baseline methods.

4: Is there a better way to maximize the effectiveness of our BTP pipeline? (e.g., mixing sampled programs in the ER)

### 4.1 Experiment Settings

#### Datasets

In recent years, a variety of open-source programming datasets have emerged, providing a robust foundation for evaluating code models. To ensure the robustness and generalizability of our proposed BTP pipeline, we applied it to fine-tune several state-of-the-art code models and evaluated them on a diverse set of popular benchmark datasets, including CodeContests from AlphaCode (Li et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib15)), APPS (Hendrycks et al. [2021](https://arxiv.org/html/2410.12236v1#bib.bib11)), and HumanEval (Chen et al. [2021b](https://arxiv.org/html/2410.12236v1#bib.bib5)). For HumanEval, which comprises 164 programming problems complete with function signatures, docstrings, bodies, and unit tests, we utilized all unit tests for a given problem as the test set, while the remaining descriptions were used as code generation tasks. For CodeContests, we similarly treated the problem descriptions as code generation tasks and combined all public and private test cases into a unified test set. In the case of APPS, where public and private test cases are not differentiated, we aggregated all test cases to form a comprehensive test set for each code generation task.

#### Models

We categorized the models used in our experiments into two distinct groups. The first group comprises models that undergo fine-tuning, including GPT-2 and GPT-Neo. The second group consists of models employed for generating code samples. Within this latter category, we explored two scenarios: in the first, we utilized advanced code models such as GPT-4-turbo (OpenAI [2023](https://arxiv.org/html/2410.12236v1#bib.bib19)), GPT-3.5-turbo, CodeLlama-34B (Roziere et al. [2022](https://arxiv.org/html/2410.12236v1#bib.bib23)), and WizardCoder-34B (Luo et al. [2023](https://arxiv.org/html/2410.12236v1#bib.bib18)); in the second, we utilized code models that were identical to those used for fine-tuning.

 |  | Pass Rate (%) | Accuracy Rate (%) |
| --- | --- | --- |
|  | APPS Intro. | APPS Inter. | APPS Comp. | APPS Mixed | APPS Intro. | APPS Inter. | APPS Comp. | APPS Mixed |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-Neo-GPT4 | 51.23 | 46.39 | 38.89 | 42.12 | 26.88 | 23.34 | 17.12 | 20.54 |
| WizardCoder | 45.24 | 41.56 | 37.46 | 40.24 | 29.73 | 25.36 | 21.25 | 23.34 |
| GPT-4-turbo | 84.24 | 80.36 | 78.46 | 81.25 | 65.25 | 60.45 | 53.59 | 59.27 |
| GPT-3.5-turbo | 55.75 | 52.26 | 51.37 | 52.64 | 45.47 | 37.41 | 30.29 | 38.25 |
| CodeLlama | 53.45 | 51.26 | 49.24 | 52.27 | 28.35 | 25.92 | 23.25 | 26.78 | 

Table 3: Comparison of fine-tuned code models and baseline models on the APPS dataset across different difficulty levels.

 |  | Pass Rate (%) | Accuracy Rate (%) |
| --- | --- | --- |
|  | APPS Mixed | CodeContests | HumanEval | APPS Mixed | CodeContests | HumanEval |
| --- | --- | --- | --- | --- | --- | --- |
| GPT-2 GPT4 | 43.27 | 35.38 | 29.25 | 5.39 | 4.25 | 4.11 |
| CodeContests | 36.32 | 46.93 | 38.25 | 4.82 | 6.10 | 5.25 |
| HumanEval | 31.92 | 25.21 | 47.93 | 2.62 | 4.25 | 8.25 |
| Mixed | 40.25 | 42.98 | 41.52 | 5.21 | 5.87 | 6.23 |
| GPT-Neo GPT4 | 45.22 | 38.18 | 30.92 | 6.48 | 5.25 | 5.59 |
| CodeContests | 39.11 | 48.52 | 39.26 | 5.29 | 7.20 | 6.23 |
| HumanEval | 33.74 | 27.93 | 50.92 | 4.92 | 6.24 | 10.29 |
| Mixed | 42.85 | 45.21 | 43.58 | 9.24 | 8.53 | 8.39 | 

Table 4: Performance of GPT-4-turbo fine-tuned with BTP pipeline on different datasets compared with baseline models.

#### Hyperparameter Optimization

We conducted a series of experiments to determine the most effective hyperparameters for our models. Initially, we investigated whether beam search sampling outperforms simple sampling in terms of effectiveness. The results confirmed the superiority of beam search sampling, prompting further experiments to determine the optimal value for the beam search parameter $k$. Balancing effectiveness and resource consumption, we selected $k=3$ for our primary experiments, sampling the top-3 programs based on their probabilities. Detailed results and analysis are provided in Appendix A.

Additionally, we explored the impact of the hyperparameter $\alpha$ during the PPER phase. However, our findings revealed that the optimal $\alpha$ value varies across different models and datasets. To address this, we conducted targeted experiments across various datasets to identify the best-performing $\alpha$ for each scenario. The outcomes of these experiments, along with a comprehensive analysis, are presented in Appendix B and Appendix C.

### 4.2 Fine-tuning Code Models with the BTP Pipeline

In this section, we systematically address the four key questions posed earlier by dividing our experiments into four distinct parts.

#### Leveraging Advanced Models to Enhance Baseline Models

To investigate our first hypothesis, denoted as C1, we conducted an experiment to test whether the performance of baseline models can be significantly improved by leveraging advanced models within our proposed BTP (Better Transformer Programming) pipeline. Specifically, we employed the APPS dataset, which is structured into three levels of difficulty: introductory, intermediate, and competition-level tasks. These tasks were designed to assess the models’ capabilities across a range of programming challenges.

As described in Table [1](https://arxiv.org/html/2410.12236v1#S4.T1 "Table 1 ‣ 4 Experiments ‣ Enhancing LLM Agents for Code Generation with Possibility and Pass-rate Prioritized Experience Replay"), we consolidated all three sections of the APPS dataset into a single, comprehensive dataset referred to as APPS mixed. This combined dataset was used to train and evaluate the models, ensuring that they were exposed to a diverse array of task difficulties, thereby providing a robust assessment of their generalization capabilities.

For this experiment, we selected four state-of-the-art transformer-based models: GPT-4-turbo, GPT-3.5-turbo, CodeLlama-34B, and WizardCoder-34B. These models were tasked with generating sample programs, which were subsequently used to fine-tune two baseline models: GPT-2 and GPT-Neo. The fine-tuning process involved using the sample programs generated by each advanced model to create eight fine-tuned variants of the baseline models, named as follows:

We utilized four advanced transformer models to generate sample programs, which were then used to fine-tune two baseline models. Specifically, the GPT-4-turbo, GPT-3.5-turbo, CodeLlama-34B, and WizardCoder-34B models were employed to generate samples that were subsequently used to fine-tune GPT-2 and GPT-Neo. This process resulted in eight fine-tuned models: GPT-2 fine-tuned with samples from GPT-4-turbo, GPT-3.5-turbo, CodeLlama-34B, and WizardCoder-34B, respectively named GPT-2-GPT4, GPT-2-GPT3.5, GPT-2-Llama, and GPT-2-Wizard; similarly, GPT-Neo was fine-tuned with samples from these four models, resulting in GPT-Neo-GPT4, GPT-Neo-GPT3.5, GPT-Neo-Llama, and GPT-Neo-Wizard.

After the fine-tuning process, we evaluated each of these models on the three distinct sections of the APPS dataset as well as on the combined APPS mixed dataset. The objective was to assess the extent to which the fine-tuned models could improve their performance on code generation tasks of varying complexity.

The results, presented in Table [2](https://arxiv.org/html/2410.12236v1#S4.T2 "Table 2 ‣ 4 Experiments ‣ Enhancing LLM Agents for Code Generation with Possibility and Pass-rate Prioritized Experience Replay"), reveal a substantial improvement in the performance of the fine-tuned models compared to their original, unmodified versions. This improvement is observed consistently across all sections of the APPS dataset, which underscores the effectiveness of our BTP pipeline. By incorporating advanced models for program sampling, we significantly enhance the capabilities of baseline transformer models like GPT-2 and GPT-Neo.

These findings strongly suggest that even relatively simple models can achieve notable performance gains when they are exposed to more advanced models during the training process, thus enabling them to perform better on complex code generation tasks.

#### Self-Improvement Through Model Self-Fine-Tuning

In our second experiment, we aimed to validate our second hypothesis, C2, which proposes that models can improve their own performance through a self-sampling approach within the BTP pipeline. For this experiment, we once again utilized the APPS dataset, divided into introductory, intermediate, and competition-level tasks, to evaluate the models comprehensively across different levels of difficulty.

As with our first experiment, we combined the three parts of the APPS dataset into a single, unified dataset termed APPS mixed. This ensured that each model was trained and evaluated on a diverse set of tasks, providing a rigorous test of the self-fine-tuning approach.

We selected three models for this experiment: GPT-2, GPT-Neo, and WizardCoder-34B. Each of these models was used to sample programs from the APPS mixed dataset, and the sampled programs were subsequently employed to fine-tune the same models that generated them. This process effectively creates a feedback loop, allowing the models to refine their own capabilities using their generated outputs.

The resulting self-fine-tuned models were named as follows:

GPT-2, GPT-Neo, and WizardCoder-34B were each fine-tuned using programs that they sampled themselves. This self-fine-tuning process resulted in the following models: GPT-2-2, which is GPT-2 fine-tuned with its own sampled programs; GPT-Neo-Neo, which is GPT-Neo fine-tuned with its own sampled programs; and WizardCoder-Wizard, which is WizardCoder-34B fine-tuned with its own sampled programs.

We then evaluated these self-fine-tuned models on the different sections of the APPS dataset, as well as on the APPS mixed dataset, to determine the effectiveness of this self-sampling and fine-tuning approach.

The results, shown in Table 1, indicate that while the performance improvements are modest, there is a consistent positive trend across most tasks. This suggests that the BTP pipeline can indeed enhance model performance even when models are fine-tuning themselves using their generated programs. This experiment supports the notion that models can incrementally improve their capabilities through self-guided learning, highlighting the potential of self-improvement mechanisms in transformer-based models.

#### Comparative Analysis of the Best BTP-Generated Model and Baseline Models

Among all the fine-tuned code models generated through the BTP pipeline, GPT-Neo-GPT4 demonstrated the best overall performance, as shown in Table [3](https://arxiv.org/html/2410.12236v1#S4.T3 "Table 3 ‣ Models ‣ 4.1 Experiment Settings ‣ 4 Experiments ‣ Enhancing LLM Agents for Code Generation with Possibility and Pass-rate Prioritized Experience Replay"). To further assess the effectiveness of our approach, we conducted a comparative analysis between GPT-Neo-GPT4 and other baseline models, as presented in Table [4](https://arxiv.org/html/2410.12236v1#S4.T4 "Table 4 ‣ Models ‣ 4.1 Experiment Settings ‣ 4 Experiments ‣ Enhancing LLM Agents for Code Generation with Possibility and Pass-rate Prioritized Experience Replay"). Although GPT-Neo-GPT4 does not outperform the most advanced baseline models, it exhibits significant improvements over its original performance and narrows the performance gap with baseline models.

This comparative analysis underscores the capability of the BTP pipeline to enhance the performance of baseline models, bringing them closer to the state-of-the-art models, albeit with some remaining performance differences.

#### Optimizing the BTP Pipeline for Maximum Effectiveness

To address our fourth question (Q4), we explored different strategies to maximize the effectiveness of the BTP pipeline. Specifically, we conducted experiments where we sampled programs using GPT-4-turbo from four different datasets: APPS-only, CodeContests-only, HumanEval-only, and a mixture of these datasets. We then fine-tuned GPT-2 and GPT-Neo using the sampled programs within the BTP pipeline and subsequently tested these fine-tuned models on the three datasets.

The results, as depicted in Table [4](https://arxiv.org/html/2410.12236v1#S4.T4 "Table 4 ‣ Models ‣ 4.1 Experiment Settings ‣ 4 Experiments ‣ Enhancing LLM Agents for Code Generation with Possibility and Pass-rate Prioritized Experience Replay"), indicate that when mixed datasets are used for sampling programs in the BTP pipeline, the resulting fine-tuned models show improved performance across a broader range of tasks compared to models fine-tuned on single, non-mixed datasets. However, the performance of these models on a specific dataset may not reach the level of models that were fine-tuned exclusively on that dataset.

These findings suggest that varying the datasets used for program sampling is a viable strategy to enhance the overall effectiveness of the BTP pipeline. By incorporating a diverse range of tasks during the fine-tuning process, models can achieve better generalization and performance across different code generation tasks.

## 5 Conclusion

In code generation tasks, large language models (LLMs) often need to sample a large number of programs to find a completely correct one, as even a single incorrect token can lead to failure in testing. Consequently, many sampled programs are wasted.

To utilize these resources and improve efficiency, in this work, we propose a novel algorithm called the BTP pipeline, which combines beam search sampling with prioritized experience replay to fine-tune LLMs. We empirically applied our algorithm to fine-tune several LLMs and found that they showed improvement compared to previous models. We also demonstrate that our algorithm is effective not only in scenarios where programs sampled by a better code model are used to enhance a standard code model, but also in scenarios where a code model enhances itself using programs it has sampled.

Beyond improving LLM performance in code generation tasks, we believe our BTP pipeline can be beneficial for enhancing general LLMs, particularly in cases where results sampled from LLMs are difficult to pass tests. A key limitation of this work is its reliance on code tasks and corresponding test cases. Tasks with few test cases typically result in pass rates close to zero, which can hinder the effectiveness of our algorithm. In future work, we plan to explore similar test sets and expand the available test sets to address this limitation.

## References

*   Andrychowicz et al. (2017) Andrychowicz, M.; et al. 2017. Hindsight Experience Replay. In *Proceedings of the 31st International Conference on Neural Information Processing Systems (NeurIPS)*, 5048–5058\. NeurIPS.
*   Austin et al. (2022) Austin, J.; et al. 2022. Program synthesis with large language models. In *Proceedings of the 2022 International Conference on Learning Representations (ICLR)*. ICLR.
*   Bunel et al. (2018) Bunel, R.; et al. 2018. Leveraging grammar and reinforcement learning for neural program synthesis. In *Proceedings of the 2018 International Conference on Learning Representations (ICLR)*. ICLR.
*   Chen et al. (2021a) Chen, M.; et al. 2021a. Codex: Evaluating large language models trained on code. In *Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 6730–6736\. Association for Computational Linguistics.
*   Chen et al. (2021b) Chen, M.; et al. 2021b. Evaluating Large Language Models Trained on Code. In *Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 6730–6736\. Association for Computational Linguistics.
*   Chowdhery et al. (2022) Chowdhery, A.; et al. 2022. PaLM: Scaling Language Modeling with Pathways. In *Advances in Neural Information Processing Systems (NeurIPS)*. NeurIPS.
*   Devlin et al. (2019) Devlin, J.; et al. 2019. BERT: Pre-training of deep bidirectional transformers for language understanding. In *Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics (NAACL)*, 4171–4186\. Association for Computational Linguistics.
*   Ellis et al. (2019) Ellis, K.; et al. 2019. Synthesizing program input grammars. In *Proceedings of the 2019 International Conference on Learning Representations (ICLR)*. ICLR.
*   Feng et al. (2020) Feng, Z.; et al. 2020. CodeBERT: A pre-trained model for programming and natural languages. In *Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 1536–1547\. Association for Computational Linguistics.
*   Guo et al. (2020) Guo, D.; et al. 2020. GraphCodeBERT: Pre-training code representations with data flow. In *Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 1548–1559\. Association for Computational Linguistics.
*   Hendrycks et al. (2021) Hendrycks, D.; et al. 2021. Measuring Coding Challenge Competence with APPS. In *Proceedings of the 35th Conference on Neural Information Processing Systems (NeurIPS)*. NeurIPS.
*   Hoffmann et al. (2022) Hoffmann, J.; et al. 2022. Chinchilla: Training compute-optimal large language models. In *Advances in Neural Information Processing Systems*. NeurIPS.
*   Le et al. (2022) Le, H.; et al. 2022. Reinforcement learning with augmented data for program synthesis. In *Proceedings of the 2022 International Conference on Learning Representations (ICLR)*. ICLR.
*   Li et al. (2023) Li, R.; et al. 2023. StarCoder: May the Source Be With You! In *Proceedings of the 2023 International Conference on Learning Representations (ICLR)*. ICLR.
*   Li et al. (2022) Li, Y.; et al. 2022. Competition-Level Code Generation with AlphaCode. https://www.deepmind.com/publications/alphacode.
*   Lian et al. (2023) Lian, Z. X.; et al. 2023. OpenOrca: An open dataset of GPT augmented FLAN reasoning traces. https://github.com/OpenOrca.
*   Lin (1992) Lin, L.-J. 1992. Self-improving reactive agents based on reinforcement learning, planning and teaching. *Machine learning*, 8(3-4): 293–321.
*   Luo et al. (2023) Luo, Z.; et al. 2023. WizardCoder: Empowering Code Large Language Models with Evol-Instruct. In *Proceedings of the 2023 International Conference on Learning Representations*. ICLR.
*   OpenAI (2023) OpenAI. 2023. GPT-4 technical report. https://openai.com/research/gpt-4.
*   Radford et al. (2019) Radford, A.; et al. 2019. Language models are unsupervised multitask learners. https://openai.com/blog/better-language-models.
*   Raffel et al. (2020) Raffel, C.; et al. 2020. Exploring the limits of transfer learning with a unified text-to-text transformer. In *Proceedings of the 2020 International Conference on Learning Representations (ICLR)*. ICLR.
*   Roziere et al. (2020) Roziere, B.; et al. 2020. Transformers for program synthesis. In *Proceedings of the 2020 Conference on Neural Information Processing Systems (NeurIPS)*. NeurIPS.
*   Roziere et al. (2022) Roziere, B.; et al. 2022. Leveraging automatically generated unit tests for unsupervised code translation. In *Proceedings of the 2022 International Conference on Learning Representations (ICLR)*. ICLR.
*   Schaul et al. (2016) Schaul, T.; Quan, J.; Antonoglou, I.; and Silver, D. 2016. Prioritized Experience Replay. In *Proceedings of the 4th International Conference on Learning Representations (ICLR)*. ICLR.
*   Svyatkovskiy et al. (2020) Svyatkovskiy, A.; et al. 2020. Code Completion with Transformers. In *Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining*, 1803–1813\. ACM.
*   Touvron et al. (2023) Touvron, H.; et al. 2023. Llama: Open and efficient foundation language models. https://arxiv.org/abs/2302.13971.
*   Vaswani et al. (2017) Vaswani, A.; Shazeer, N.; Parmar, N.; Uszkoreit, J.; Jones, L.; Gomez, A. N.; Kaiser, L.; and Polosukhin, I. 2017. Attention is all you need. In *Advances in Neural Information Processing Systems*, 5998–6008\. NeurIPS.
*   Wang et al. (2023) Wang, Y.; Yang, M.; Dong, R.; Sun, B.; Liu, F.; and U, L. H. 2023. Efficient Potential-based Exploration in Reinforcement Learning using Inverse Dynamic Bisimulation Metric. In Oh, A.; Naumann, T.; Globerson, A.; Saenko, K.; Hardt, M.; and Levine, S., eds., *Advances in Neural Information Processing Systems*, volume 36, 38786–38797\. Curran Associates, Inc.
*   Wang et al. (2021) Wang, Y.; et al. 2021. CodeT5: Identifier-aware unified pre-trained encoder-decoder models for code understanding and generation. In *Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 8697–8708\. Association for Computational Linguistics.
*   Wei et al. (2022) Wei, J.; et al. 2022. Chain-of-thought (CoT) prompting. In *Advances in Neural Information Processing Systems (NeurIPS)*. NeurIPS.
*   Yang et al. (2023) Yang, M.; Dong, R.; Wang, Y.; Liu, F.; Du, Y.; Zhou, M.; and U, L. H. 2023. TieComm: Learning a Hierarchical Communication Topology Based on Tie Theory. In *Database Systems for Advanced Applications*, 604–613\. Cham: Springer Nature Switzerland. ISBN 978-3-031-30637-2.
*   Yang et al. (2024a) Yang, M.; Wang, Y.; Yu, Y.; Zhou, M.; and U, L. H. 2024a. MixLight: Mixed-Agent Cooperative Reinforcement Learning for Traffic Light Control. *IEEE Transactions on Industrial Informatics*, 20(2): 2653–2661.
*   Yang et al. (2024b) Yang, M.; Zhao, K.; Wang, Y.; Dong, R.; Du, Y.; Liu, F.; Zhou, M.; and U, L. H. 2024b. Team-wise effective communication in multi-agent reinforcement learning. *Autonomous Agents and Multi-Agent Systems*, 38(2): 36.
*   Zhang et al. (2023) Zhang, K.; et al. 2023. Planning with Large Language Models for Code Generation. In *Proceedings of the 2023 International Conference on Learning Representations (ICLR)*. ICLR.