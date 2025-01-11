<!--yml
category: 未分类
date: 2025-01-11 12:34:19
-->

# Hello Again! LLM-powered Personalized Agent for Long-term Dialogue

> 来源：[https://arxiv.org/html/2406.05925/](https://arxiv.org/html/2406.05925/)

Hao Li¹
18th.leolee@gmail.com
&Chenghao Yang^(2∗)
yangchenghao@mail.ustc.edu.cn
&An Zhang³
anzhang@u.nus.edu
&Yang Deng³
ydeng@nus.edu.sg
&Xiang Wang²
xiangwang1223@gmail.com
&Tat-Seng Chua³
dcscts@nus.edu.sg
&¹University of Electronic Science and Technology of China
²University of Science and Technology of China
³National University of Singapore These authors contribute equally to this work.An Zhang is the corresponding author.

###### Abstract

Open-domain dialogue systems have seen remarkable advancements with the development of large language models (LLMs). Nonetheless, most existing dialogue systems predominantly focus on brief single-session interactions, neglecting the real-world demands for long-term companionship and personalized interactions with chatbots. Crucial to addressing this real-world need are event summary and persona management, which enable reasoning for appropriate long-term dialogue responses. Recent progress in the human-like cognitive and reasoning capabilities of LLMs suggests that LLM-based agents could significantly enhance automated perception, decision-making, and problem-solving. In response to this potential, we introduce a model-agnostic framework, the Long-term Dialogue Agent (LD-Agent), which incorporates three independently tunable modules dedicated to event perception, persona extraction, and response generation. For the event memory module, long and short-term memory banks are employed to separately focus on historical and ongoing sessions, while a topic-based retrieval mechanism is introduced to enhance the accuracy of memory retrieval. Furthermore, the persona module conducts dynamic persona modeling for both users and agents. The integration of retrieved memories and extracted personas is subsequently fed into the generator to induce appropriate responses. The effectiveness, generality, and cross-domain capabilities of LD-Agent are empirically demonstrated across various illustrative benchmarks, models, and tasks. The code is released at [https://github.com/leolee99/LD-Agent](https://github.com/leolee99/LD-Agent).

## 1 Introduction

Open-domain dialogue systems aim to establish long-term, personalized interactions with users via human-like chatbots [[1](https://arxiv.org/html/2406.05925v1#bib.bib1), [2](https://arxiv.org/html/2406.05925v1#bib.bib2), [3](https://arxiv.org/html/2406.05925v1#bib.bib3)]. Unlike most existing studies [[4](https://arxiv.org/html/2406.05925v1#bib.bib4), [5](https://arxiv.org/html/2406.05925v1#bib.bib5), [6](https://arxiv.org/html/2406.05925v1#bib.bib6)] that are limited to brief, single-session interactions spanning 2-15 turns, real-life scenarios often necessitate a chatbot’s capability for long-term companionship and familiarity [[1](https://arxiv.org/html/2406.05925v1#bib.bib1), [2](https://arxiv.org/html/2406.05925v1#bib.bib2), [7](https://arxiv.org/html/2406.05925v1#bib.bib7)]. Achieving this requires the chatbot not only to understand and remember extensive dialogue histories but also to faithfully reflect and consistently update both the user’s and its personalized characteristics [[1](https://arxiv.org/html/2406.05925v1#bib.bib1), [7](https://arxiv.org/html/2406.05925v1#bib.bib7), [8](https://arxiv.org/html/2406.05925v1#bib.bib8)].

Motivated by real-life demands, the core challenge of open-domain dialogue systems is to simultaneously maintain long-term event memory and preserve persona consistency [[9](https://arxiv.org/html/2406.05925v1#bib.bib9), [10](https://arxiv.org/html/2406.05925v1#bib.bib10), [11](https://arxiv.org/html/2406.05925v1#bib.bib11), [3](https://arxiv.org/html/2406.05925v1#bib.bib3)]. Existing research often addresses these aspects separately—focusing either on event memory or persona extraction—thereby hindering long-term consistency. Current strategies for event memory typically involve constructing a memory bank that stores historical event summaries, complemented by retrieval-augmented approaches to access relevant information for response generation [[12](https://arxiv.org/html/2406.05925v1#bib.bib12), [13](https://arxiv.org/html/2406.05925v1#bib.bib13)]. Studies on persona-based dialogue rang from unidirectional user modeling [[14](https://arxiv.org/html/2406.05925v1#bib.bib14)] to bidirectional agent-user modeling [[15](https://arxiv.org/html/2406.05925v1#bib.bib15), [16](https://arxiv.org/html/2406.05925v1#bib.bib16), [3](https://arxiv.org/html/2406.05925v1#bib.bib3)], enhancing personalized chat abilities by leveraging profile information. Worse still, the aforementioned methods are highly dependent on specific model architectures, making them challenging to adapt to other models. Additionally, These dialogue models largely lack zero-shot generalization capabilities, essential for effective deployment across various real-world domains [[2](https://arxiv.org/html/2406.05925v1#bib.bib2), [3](https://arxiv.org/html/2406.05925v1#bib.bib3)]. We conjecture that an optimal long-term dialogue framework should be model-agnostic, deployable in various real-world domains, and capable of autonomously integrating comprehensive data from both event memories and personas, as illustrated in Figure [1](https://arxiv.org/html/2406.05925v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). However, developing such a model-agnostic, cross-domain, and autonomous framework remains unexplored and challenging.

![Refer to caption](img/c72a42e7110d57607bfefc4c157326a4.png)

Figure 1: The illustration of how event memory and personas guide long-term dialogue. The event summary and personas are extracted from a conversation that occurred one week ago. In today’s interaction, the event memory prompts the girl to inquire about the swimming lesson they scheduled last week. The personas, indicating that she is careful and professional in swimming, guide her to offer detailed and professional advice.

Benefiting from the excellent human-like cognitive and reasoning abilities of large language models (LLM), there is an increasing trend [[17](https://arxiv.org/html/2406.05925v1#bib.bib17), [18](https://arxiv.org/html/2406.05925v1#bib.bib18), [19](https://arxiv.org/html/2406.05925v1#bib.bib19), [20](https://arxiv.org/html/2406.05925v1#bib.bib20), [21](https://arxiv.org/html/2406.05925v1#bib.bib21)] to employ LLMs as the cores of agent-based simulation systems to automate the process of perception, decision-making, and problem-solving. While recent studies have developed LLM-powered agents in various fields, such as economics [[22](https://arxiv.org/html/2406.05925v1#bib.bib22)], politics [[23](https://arxiv.org/html/2406.05925v1#bib.bib23)], sociology [[24](https://arxiv.org/html/2406.05925v1#bib.bib24)], and recommendation [[21](https://arxiv.org/html/2406.05925v1#bib.bib21)], its application in open-domain dialogue remains unexplored. To effectively support long-term open-domain dialogue, an LLM-powered dialogue agent framework should exhibit broad generality, cross-domain adaptability, and the ability to dynamically refine information across dimensions like events, user personalities, and agent personalities.

In this paper, we propose LD-Agent—a model-agnostic Long-term Dialogue Agent framework consisting of three principal components: an event memory perception module, a persona extraction module, and response generation module (see the framework of LD-Agent in Figure [2](https://arxiv.org/html/2406.05925v1#S2.F2 "Figure 2 ‣ 2.2 Event Perception ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue")). The event memory perception module is designed to enhance coherence across sessions by separately maintaining long-term and short-term memory banks. The long-term memory bank stores vector representations of high-level event summaries from previous sessions, refined through a tunable event summary module. The short-term memory bank maintains contextual information for ongoing conversations. The persona extraction module, designed to facilitate personalized interactions, incorporates a disentangled, tunable mechanism for accurate user-agent modeling. Extracted personas are continuously updated and stored in a long-term persona bank. These personas, along with relevant memories, are then integrated into the response generation module, guiding the generation of appropriate responses, as depicted in Figure [1](https://arxiv.org/html/2406.05925v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

We conduct comprehensive experiments on two illustrative long-term multi-session daily dialogue datasets, MSC [[1](https://arxiv.org/html/2406.05925v1#bib.bib1)] and Conversation Chronicles (CC) [[7](https://arxiv.org/html/2406.05925v1#bib.bib7)], to evaluate the effectiveness, generality, and cross-domain capabilities of the proposed framework. In terms of effectiveness, LD-Agent achieves state-of-the-art performance on both benchmarks, significantly outperforming existing methods [[2](https://arxiv.org/html/2406.05925v1#bib.bib2), [25](https://arxiv.org/html/2406.05925v1#bib.bib25), [26](https://arxiv.org/html/2406.05925v1#bib.bib26)]. To assess generality, we examine the framework from both model and task perspectives. From the model perspective, LD-Agent is evaluated across a range of both online and offline models, including LLMs [[25](https://arxiv.org/html/2406.05925v1#bib.bib25)] and non-LLMs [[26](https://arxiv.org/html/2406.05925v1#bib.bib26)]. From the task perspective, we extend our evaluation to multiparty dialogue tasks [[27](https://arxiv.org/html/2406.05925v1#bib.bib27)], where LD-Agent also demonstrates substantial improvements, showcasing its adaptability across different models and tasks. Regarding the method’s cross-domain capabilities, we design two cross-domain settings: tuning the model on the MSC dataset and testing it on the CC dataset, and vice versa. In both scenarios, LD-Agent shows competitive performance, nearly matching the results of in-domain training.

Our contributions can be summarized as follows:

*   •

    We develop LD-Agent, a general long-term dialogue agent framework, considering both historical events and personas. The event memory module ensures dialogue coherence across sessions, while the persona module ensures character consistency.

*   •

    We introduce a disentangled, tunable approach for long-term dialogue to ensure the accuracy of each module. The highly modular framework enables it to adapt to various dialogue tasks through module re-training.

*   •

    We confirm the superiority of our proposed framework through rigorous experiments across multiple challenging benchmarks, diverse illustrative models, and various tasks. Extensive insightful ablation studies further highlight its effectiveness and generalization.

## 2 Method

In this section, we introduce the detailed description of LD-Agent with the framework shown in Figure [2](https://arxiv.org/html/2406.05925v1#S2.F2 "Figure 2 ‣ 2.2 Event Perception ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). We first introduce the task definition of long-term dialogue in Section. [2.1](https://arxiv.org/html/2406.05925v1#S2.SS1 "2.1 Task Definition ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). Consequently, we separately introduce the mechanism of event perception (Section. [2.2](https://arxiv.org/html/2406.05925v1#S2.SS2 "2.2 Event Perception ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue")), dynamic personas extraction (Section. [2.3](https://arxiv.org/html/2406.05925v1#S2.SS3 "2.3 Dynamic Personas Extraction ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue")), and response generation (Section. [2.4](https://arxiv.org/html/2406.05925v1#S2.SS4 "2.4 Response Generation ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue")).

### 2.1 Task Definition

The goal of the long-term multi-session dialogue task is to generate an appropriate response $r$, by utilizing the context of the current session $C$, along with selected information extracted from historical session $H$. In this task, the current conversation session $C$ is defined as $\{u_{1},u_{2},\dots,u_{d_{c}-1},u_{d_{c}}\}$, where each $u_{i}$ represents $i$-th utterance, and $d_{c}$ represents $d_{c}$ turns of the current session. Each historical session within $H$ in $N$ historical sessions is denoted as $H^{i}$, containing $\{h^{i}_{1},h^{i}_{2},\dots,h^{i}_{d_{i}}\}$, where $d_{i}$ is the number of utterances of the $i$-th conversational session. Distinct from single-session dialogue models, a long-term multi-session dialogue system integrates both current and long-term historical conversational cues to generate contextually appropriate responses.

### 2.2 Event Perception

The event memory module is designed to perceive historical events to generate coherent responses across interval time. As shown in Figure [2](https://arxiv.org/html/2406.05925v1#S2.F2 "Figure 2 ‣ 2.2 Event Perception ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"), this event memory module is segmented into two major sub-modules that focus separately on long-term and short-term memory.

![Refer to caption](img/6c63b3e0a2d0ccb9e3f97652e3402cb5.png)

Figure 2: The Framework of LD-Agent. The event module stores historical memories from past sessions in long-term memory and current context in short-term memory. The persona module dynamically extracts and updates personas for both users and agents from ongoing utterances, storing them in a persona bank for each character. The response module then synthesizes this data to generate informed and appropriate responses.

#### 2.2.1 Long-term Memory

##### Memory Storage.

The long-term memory module aims to extract and encode events from past sessions. Specifically, this involves recording the occurrence times $t$ and brief summaries $o$ into representations that are stored in a low-cost memory bank $M_{L}=\{\phi(t_{j},o_{j})\mid j\in\{1,2,\dots,l\}\}$. Here, $\phi(\cdot)$ indicates the text encoder (e.g., MiniLM [[28](https://arxiv.org/html/2406.05925v1#bib.bib28)]), and $l$ specifies the length of the memory bank. The encoded representations are then efficiently retrieved through an embedding-based mechanism, which enhances the accessibility of the stored memory.

##### Event Summary.

Different from previous agent approaches [[20](https://arxiv.org/html/2406.05925v1#bib.bib20), [21](https://arxiv.org/html/2406.05925v1#bib.bib21), [29](https://arxiv.org/html/2406.05925v1#bib.bib29)] that entirely rely on LLM’s zero-shot ability to excavate and summarize events, we apply instruction tuning [[30](https://arxiv.org/html/2406.05925v1#bib.bib30)] to the event summary module, which can directly improve the event summary quality. Specifically, we rebuild the DialogSum dataset [[31](https://arxiv.org/html/2406.05925v1#bib.bib31)], a large-scale dialogue summarization dataset, into the following format: (1) an introduction to the task background, (2) the related conversations that need to be understood, and (3) detailed summarization requests. These three parts serve as input prompts (see Appendix. [D.1](https://arxiv.org/html/2406.05925v1#A4.SS1 "D.1 Prompt of Event Summary ‣ Appendix D Prompt ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue") for more details), combined with the original summaries from DialogSum as answers, and are jointly used to fine-tune the event summary module, thereby directly improving the quality of event summarization.

##### Memory Retrieval.

To improve retrieval accuracy, we employ a retrieval mechanism that comprehensively considers semantic relevance, topic overlap, and time decay. Optimizing the retrieval accuracy of agent memory is challenging due to the difficulty in obtaining accurate memory retrieval data. Most existing methods [[20](https://arxiv.org/html/2406.05925v1#bib.bib20), [21](https://arxiv.org/html/2406.05925v1#bib.bib21)] use event summaries as keys and context as queries, calculating the query-key semantic relevance score $s_{\text{sem}}$ to find relevant memories, which inevitably results in significant errors. To enhance retrieval reliability, we extract nouns from corresponding conversations with the summaries to construct a topic library $V$ and calculate topic overlap score $s_{\text{top}}$ by:

|  | $s_{\text{top}}=\frac{1}{2}(\frac{\left&#124;V_{q}\cap V_{k}\right&#124;}{V_{q}}+\frac{% \left&#124;V_{q}\cap V_{k}\right&#124;}{V_{k}}),$ |  | (1) |

where $V_{q}$, $V_{k}$ denote the topic noun set of query and key. Additionally, we apply a time decay coefficient $\lambda_{t}=e^{{-t}/{\tau}}$ to reweight the overall retrieval score $s_{r}$, signified as:

|  | $s_{\text{overall}}=\lambda_{t}(s_{\text{sem}}+s_{\text{top}}).$ |  | (2) |

To avoid retrieving inappropriate memory due to no suitable memories existing, we implement a semantic threshold $\gamma$. Only memories with semantic score $s_{\text{sem}}$ greater than $\gamma$ could be retrieved. If no appropriate memories are retrieved, “No relevant memory” will be returned. Eventually, the process of retrieving relevant memory $m$ can be denoted as:

|  | $m=\psi(M_{L},\gamma).$ |  | (3) |

#### 2.2.2 Short-term Memory

The short-term memory module actively manages a dynamic dialogue cache $M_{S}=\{(t_{i},u_{i})|i=\{1,2,3,\dots,r_{c}\}\}$ with timestamps to preserve the detailed context of the current session. Upon receiving a new utterance $u^{\prime}$, the module first evaluates the time interval between the current time $t^{\prime}$ and the last recorded time $t_{r_{c}}$ in the cache. If this interval exceeds a threshold $\beta$, the module triggers the long-term memory module to summarize the cached dialogue entries, creating new event records for storage in the long-term memory bank. Simultaneously, the short-term memory cache is cleared, and the new dialogue record $(t^{\prime},u^{\prime})$ is added to the cache. The mathematical expression of this process is given by:

|  | $\displaystyle M^{\prime}_{L}$ | $\displaystyle=M_{L}\cup\{(\phi(t_{r_{c}},A(M_{S}))\},$ |  | (4) |
|  | $\displaystyle M_{S}$ | $\displaystyle=\{(t^{\prime},u^{\prime})\}.$ |  |

where $M^{\prime}_{L}$ denotes the updated long-term memory bank, $o=A(\cdot)$ represents the event summary function, which process the accumulated dialogue in $M_{S}$.

### 2.3 Dynamic Personas Extraction

The persona module is pivotal in maintaining long-term persona consistency for both participants in a dialogue system. Drawing inspiration from prior work [[3](https://arxiv.org/html/2406.05925v1#bib.bib3)], we adopt a bidirectional user-agent modeling approach, utilizing a tunable persona extractor to manage long-term persona bank $P_{u}$ and $P_{a}$ for the user and agent, respectively. Specifically, we develop an open-domain, utterance-based persona extraction dataset derived from MSC [[1](https://arxiv.org/html/2406.05925v1#bib.bib1)]. We enhance the persona extractor with LoRA-based instruction tuning, which allows for the dynamic extraction of personality traits during conversations. These traits are subsequently stored in the corresponding character’s persona bank. For utterances devoid of personality traits, the module outputs “No Trait”. Additionally, we employ a tuning-free strategy that harnesses the zero-shot capabilities of LLM models to directly extract personas based on prompts. To further improve the agent’s ability to excavate user personas without training, we adjust our reasoning strategy from direct reasoning to a Chain-of-Thought reasoning [[32](https://arxiv.org/html/2406.05925v1#bib.bib32)] (see Appendix. [D.2](https://arxiv.org/html/2406.05925v1#A4.SS2 "D.2 Prompt of Persona Extraction ‣ Appendix D Prompt ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue")).

### 2.4 Response Generation

Upon receiving a new user utterance $u^{\prime}$, the agent integrates various inputs: retrieved relevant memories $m$, short-term context $M_{S}$, and the personas $P_{u}$ and $P_{a}$ for the user and agent, respectively. These combined inputs are fed into a response generator to deduce an appropriate response $r$, as formulated in Eq. [5](https://arxiv.org/html/2406.05925v1#S2.E5 "In 2.4 Response Generation ‣ 2 Method ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

|  | $r=G(u^{\prime},m,M_{S},P_{u},P_{a}).$ |  | (5) |

To enhance the agent’s ability for coherent and contextually appropriate responses, we develop a long-term, multi-session dialogue dataset, featuring dynamic retrieval memories, context, and personas sourced from the MSC and CC datasets for generator tuning. Specifically, for each sample, covering five sessions, we dynamically simulate the entire progression of the conversation. As each new utterance is introduced, the previously tuned modules for event summarization, persona extraction, and topic-aware memory retrieval are utilized to collect the necessary context, retrieved memories, and both user and agent personas related to the utterance. This comprehensive data is then integrated into a response generation prompt (see Appendix. [D.3](https://arxiv.org/html/2406.05925v1#A4.SS3 "D.3 Prompt of Response Generation ‣ Appendix D Prompt ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue")). The original responses from the MSC and CC datasets are used as ground truth sentences.

## 3 Experiments

We aim to answer the following research questions:

*   •

    RQ1: How does LD-Agent perform in long-term dialogue tasks?

*   •

    RQ2: How is the generality and practicality of LD-Agent?

### 3.1 Evaluation Settings

In this subsection, we briefly introduce the experimental dataset, evaluation metrics, and baseline models in our study. Detailed evaluation settings are elaborated in Appendix. [C](https://arxiv.org/html/2406.05925v1#A3 "Appendix C Detailed Evaluation Settings ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

##### Datasets.

Extensive experiments are conducted on two illustrative multi-session datasets, MSC [[1](https://arxiv.org/html/2406.05925v1#bib.bib1)] and CC [[7](https://arxiv.org/html/2406.05925v1#bib.bib7)], each comprising 5 sessions with approximately 50 conversational turns per sample, to investigate the effectiveness of LD-Agent on long-term dialogue scenarios. The experiments cover model independence assessment, module ablation, persona extractor analysis, and cross-domain evaluation.

Additionally, to evaluate the transferability of the LD-Agent, we apply our method to the Ubuntu IRC benchmark [[27](https://arxiv.org/html/2406.05925v1#bib.bib27)], a dataset known for its multiparty interaction tasks.

##### Metrics.

Our evaluation combines both automatic and human assessments to thoroughly investigate the effectiveness of LD-Agent. For automatic evaluation, we use three widely used standard metrics: BLEU-N (BL-N) [[33](https://arxiv.org/html/2406.05925v1#bib.bib33)], ROUGE-L (R-L) [[34](https://arxiv.org/html/2406.05925v1#bib.bib34)], and METEOR (MET) [[35](https://arxiv.org/html/2406.05925v1#bib.bib35)] to measure the quality of response generation. Additionally, accuracy (ACC) is employed to evaluate the classification performance of the persona extractor. In human evaluation, we measure topic coherence across sessions, interaction fluency, and user engagement using the metrics of coherence, fluency, and engagingness, respectively.

##### Baselines.

To demonstrate the effectiveness and model independence of LD-Agent, we deploy LD-Agent on multiple platforms and models. Specifically, the LLM-based models (online model: ChatGPT; offline model: ChatGLM [[25](https://arxiv.org/html/2406.05925v1#bib.bib25)]) and traditional language models (BlenderBot [[26](https://arxiv.org/html/2406.05925v1#bib.bib26)], and BART [[36](https://arxiv.org/html/2406.05925v1#bib.bib36)]) are employed as our baselines. In our experiments, The notation “Model[LDA]” denotes models that incorporate the LD-Agent framework, while “Model” refers to the original baseline models without LD-Agent. Additionally, we also utilize HAHT [[2](https://arxiv.org/html/2406.05925v1#bib.bib2)], the previous state-of-the-art model in long-term dialogue task, as a contrast. See the above baselines stand and their role in rich literature in Appendix. [A](https://arxiv.org/html/2406.05925v1#A1 "Appendix A Related Work ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

Table 1: Experimental results of the automatic evaluation for response generation on MSC and CC.

 |  |  | Session 2 | Session 3 | Session 4 | Session 5 |
|  | Model | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L |
| MSC |
|  | ChatGLM | 5.44 | 1.49 | 16.76 | 5.18 | 1.55 | 15.51 | 5.63 | 1.33 | 16.35 | 5.92 | 1.45 | 16.63 |
|  | ChatGLM[LDA] | 5.74 | 1.73 | 17.21 | 6.05 | 1.73 | 16.97 | 6.09 | 1.59 | 16.76 | 6.60 | 1.94 | 17.18 |
|  | ChatGPT | 5.22 | 1.45 | 16.04 | 5.18 | 1.55 | 15.51 | 4.64 | 1.32 | 15.19 | 5.38 | 1.58 | 15.48 |
| Zero-shot | ChatGPT[LDA] | 8.67 | 4.63 | 19.86 | 7.92 | 3.55 | 18.54 | 7.08 | 2.97 | 17.90 | 7.37 | 3.03 | 17.86 |
|  | HAHT | 5.06 | 1.68 | 16.82 | 4.96 | 1.50 | 16.48 | 4.75 | 1.45 | 15.82 | 4.99 | 1.51 | 16.24 |
|  | BlenderBot | 5.71 | 1.62 | 16.15 | 8.10 | 2.50 | 18.23 | 7.55 | 1.96 | 17.45 | 8.02 | 2.36 | 17.65 |
|  | BlenderBot[LDA] | 8.45 | 3.27 | 19.07 | 8.68 | 3.06 | 18.87 | 8.16 | 2.77 | 18.06 | 8.31 | 2.69 | 18.19 |
|  | ChatGLM | 5.48 | 1.59 | 17.65 | 6.12 | 1.78 | 17.91 | 6.14 | 1.63 | 17.78 | 6.16 | 1.69 | 17.65 |
| Tuning | ChatGLM[LDA] | 10.70 | 5.63 | 23.31 | 10.03 | 5.12 | 21.55 | 9.07 | 4.06 | 20.19 | 8.96 | 4.01 | 19.94 |
| CC |
| Zero-shot | ChatGLM | 8.94 | 4.44 | 21.54 | 8.34 | 4.03 | 21.00 | 8.28 | 3.82 | 20.67 | 8.12 | 3.81 | 20.54 |
| ChatGLM[LDA] | 9.53 | 4.82 | 22.76 | 9.22 | 4.43 | 22.18 | 9.15 | 4.48 | 22.18 | 8.99 | 4.43 | 22.10 |
| ChatGPT | 10.57 | 5.50 | 22.10 | 10.58 | 5.59 | 22.04 | 10.61 | 5.58 | 21.92 | 10.17 | 5.22 | 21.45 |
| ChatGPT[LDA] | 15.89 | 11.01 | 26.96 | 12.92 | 8.27 | 24.31 | 12.20 | 7.35 | 23.69 | 11.54 | 6.74 | 22.87 |
| Tuning | BlenderBot | 8.99 | 4.86 | 21.58 | 9.44 | 5.19 | 22.13 | 9.46 | 5.21 | 22.08 | 8.99 | 4.75 | 21.73 |
| BlenderBot[LDA] | 14.47 | 10.16 | 27.91 | 15.66 | 11.33 | 29.10 | 15.13 | 10.80 | 28.38 | 14.08 | 9.72 | 27.37 |
| ChatGLM | 15.89 | 9.90 | 30.59 | 15.97 | 10.06 | 30.27 | 16.10 | 10.31 | 30.54 | 15.10 | 9.34 | 29.43 |
| ChatGLM[LDA] | 25.69 | 19.53 | 39.67 | 25.93 | 19.72 | 39.15 | 25.82 | 19.40 | 39.05 | 24.26 | 18.16 | 37.61 | 

### 3.2 Results of Multi-Session Dialogue

We adopt two multi-session dialogue dataset to quantitatively evaluate our method in long-term dialogue scenarios. The first session is used to initialize conversation and the subsequent four sessions are used to evaluate the performance of long-term dialogue. In these experiments, our framework is applied to both zero-shot models, including ChatGLM and ChatGPT, and to tuned models such as BlenderBot and ChatGLM. The results are shown in Table [1](https://arxiv.org/html/2406.05925v1#S3.T1 "Table 1 ‣ Baselines. ‣ 3.1 Evaluation Settings ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

##### Impressive performance on long-term dialogue tasks.

On both datasets, all models employing LD-Agent consistently achieve significant improvements across all sessions and metrics, showcasing the powerful ability of LD-Agent on supporting long-term dialogue. Most notably, comparing with previous state-of-the-art model HAHT, BlenderBot employing LD-Agent, which has similar parameter scale to HAHT, outperforms it with a large performance gap of 3.39%, 3.72%, 3.41%, and 3.32% on BLEU-2 ranging from session 2 to 5\. This further highlighting the effectiveness of LD-Agent on long-term dialogue tasks.

##### Remarkable generality of LD-Agent.

The generality of LD-Agent are proved from two aspects: data transferability and model transferability. The consistently improvements brought by LD-Agent on both benchmarks demonstrate the generality of our framework on various long-term dialogue scenarios. In parallel, we observe that LD-Agent also plays positive roles in the zero-shot setting, employing to the online model of ChatGPT and the offline model of ChatGLM. In the tuning setting, LD-Agent achieves significant enhancements on both LLM of ChatGLM and traditional model of BlenderBot, fully proving the remarkable model transferability of LD-Agent. These results comprehensive demonstrate the generality of LD-Agent.

### 3.3 Ablation Studies

To further analyze the effectiveness of each components, we conduct ablation studies for memory module and personas module. We adopt ChatGLM as our backbone, which is tuned solely using the context of the current session, referred to here as “Baseline”. Afterward, we separately add “Event Memory”, “Agent personas”, and “User personas” modules for additional tuning on top of the baseline. The results are presented in Table [2](https://arxiv.org/html/2406.05925v1#S3.T2 "Table 2 ‣ 3.3 Ablation Studies ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

The results clearly demonstrate that all modules positively influence long-term dialogue capabilities, with the event memory module contributing the most significant improvements. It is worth noting that although all modules experience a performance decline as the number of sessions increased, the addition of the event memory module results in more stable performance compared to the use of user or agent personas. This highlights the critical role of event memory in maintaining coherence across multiple sessions.

Table 2: Ablation study results of LD-Agent on MSC. The experiments are conducted on tuned ChatGLM. Baseline denotes the model tuned with context of current session. “+ module name” indicates the model tuned solely with context and corresponding module. “Full” indicates the model tuned with all modules.

 |  |  | Session 2 | Session 3 | Session 4 | Session 5 |
| --- | --- | --- | --- | --- | --- |
|  | Model | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  | Baseline | 5.48 | 1.59 | 17.65 | 6.12 | 1.78 | 17.91 | 6.14 | 1.63 | 17.78 | 6.16 | 1.69 | 17.65 |
|  | + Mem | 7.57 | 2.49 | 19.50 | 7.70 | 2.48 | 19.46 | 7.53 | 2.31 | 19.26 | 7.56 | 2.33 | 19.03 |
|  | + Persona [user] | 7.54 | 2.57 | 19.68 | 7.51 | 2.38 | 19.39 | 7.30 | 2.09 | 18.80 | 7.08 | 2.27 | 18.79 |
|  | + Persona [agent] | 7.00 | 2.27 | 18.70 | 7.23 | 2.33 | 18.75 | 7.32 | 2.18 | 18.47 | 7.13 | 2.36 | 18.48 |
|  | Full | 10.70 | 5.63 | 23.31 | 10.03 | 5.12 | 21.55 | 8.96 | 4.01 | 19.94 | 9.07 | 4.06 | 20.19 | 

Table 3: The effect of different extractors on persona extraction and response generation on MSC.

 |  |  | Extraction | Generation |
| --- | --- | --- | --- |
|  | Extractor | BL-2 | BL-3 | R-L | ACC | BL-2 | BL-3 | R-L |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  | CoT | 5.05 | 2.69 | 25.54 | 61.6 | 5.82 | 1.69 | 16.95 |
|  | Tuning | 8.31 | 5.65 | 43.70 | 77.8 | 6.12 | 1.75 | 17.03 | 

### 3.4 Persona Extraction Analysis

To explore the effect of different persona extractor, including zero-shot ChatGLM with Chain-of-Thought [[32](https://arxiv.org/html/2406.05925v1#bib.bib32)] and ChatGLM tuned on the persona extraction dataset collected from MSC training set, we carry out comparison experiments on two perspectives: Persona Extraction Accuracy and Impact to Response Generation. The results are shown in Table [3](https://arxiv.org/html/2406.05925v1#S3.T3 "Table 3 ‣ 3.3 Ablation Studies ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

##### Extraction Accuracy.

We evaluate the extraction accuracy on the persona extraction dataset collected from MSC testing set, through BLEU-2/3, R-L, and ACC. ACC is to assess the classification accuracy of dividing utterance into “with personas” or “without personas”. The results of extraction in Table [3](https://arxiv.org/html/2406.05925v1#S3.T3 "Table 3 ‣ 3.3 Ablation Studies ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue") show that the extractor after tuning performs better than CoT ChatGLM on all metrics. The higher BLEU and R-L indicates the tuned extractor performs better capability to extract personas from an utterance, while higher ACC indicates a stronger capability to distinguish whether personas are contained in an utterance.

##### Impact to Response Generation.

In addition, to explore the effect of different persona extractor to the final response generation, we conduct experiments on MSC by comparing the results of zero-shot ChatGLM[LDA] with personas extracted by CoT and tuned extractor, respectively. The Generation results in Table [3](https://arxiv.org/html/2406.05925v1#S3.T3 "Table 3 ‣ 3.3 Ablation Studies ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue") indicate the tuned extractor performs better in most sessions. As the number of sessions increases, the gap is also constantly expanding, demonstrating tuned extractor is more suitable for long-term dialogue.

### 3.5 Human Evaluation

To further explore the performance of LD-Agent in real-life conversation, we adopt human evaluation to separately evaluate the ability of memory recall and response generation with the results on Figure [3](https://arxiv.org/html/2406.05925v1#S3.F3 "Figure 3 ‣ Retrieval Mechanism Analysis. ‣ 3.5 Human Evaluation ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue").

##### Retrieval Mechanism Analysis.

Retrieval mechanism plays a crucial role for event memory accurately utilized in long-term dialogue. To evaluate the superiority of topic-based retrieval approach than direct semantic retrieval commonly used in previous methods, we conduct an event memory human evaluation. In the beginning, we initialize a conversation using first four sessions and store event memories for each session into long-term memory bank. In the last session, we let evaluators select relevant memories from long-term memory bank for each utterance as the ground truths. Consequently, we separately utilize direct semantic retrieval and topic-based retrieval to search relevant memories for each utterance, and calculate the accuracy and recall based on human annotations. The results are shown in Figure [3a](https://arxiv.org/html/2406.05925v1#S3.F3.sf1 "In Figure 3 ‣ Retrieval Mechanism Analysis. ‣ 3.5 Human Evaluation ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). The topic-based retrieval outperforms direct semantic retrieval with significant gap on both ACC and Recall, proving that our retrieval method accurately retrieves relevant memories.

![Refer to caption](img/6c44bd9864bbd1d0abefe722c5a4f8ba.png)

(a) Comparison of Different Retrieval Mechanism

![Refer to caption](img/012d80a66a6dc814b2a559efb2a4c412.png)

(b) Comparison of Response Generation

Figure 3: The results of human evaluation on retrieval mechanism and response generation.

##### Response Generation Analysis.

To further validate the superiority of LD-Agent in long-term open-domain dialogue tasks, we organize multiple multi-session human-bot conversations on ChatGLM with LD-Agent and w/o LD-Agent. We first initialize a predefined dialogue as the first session for all chatbots. Subsequently, we employ some human evaluators to chat with each chatbot with a time interval from first session. The interactions are evaluated on three aspects: coherence, fluency and engagingness. The results in Figure [3b](https://arxiv.org/html/2406.05925v1#S3.F3.sf2 "In Figure 3 ‣ Retrieval Mechanism Analysis. ‣ 3.5 Human Evaluation ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue") demonstrate the advantages of LD-Agent in long-term real-life dialogue scenarios.

Table 4: The results of cross-domain evaluation on MSC and CC. “Zero-shot” indicates the ChatGLM without tuning. “CC-tuning” indicates the ChatGLM tuned on CC dataset. “MSC-tuning” indicates the ChatGLM tuned on MSC dataset.

 |  | Session 2 | Session 3 | Session 4 | Session 5 |
| Model | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L | BL-2 | BL-3 | R-L |
| MSC |
| Zero-shot | 5.44 | 1.49 | 16.76 | 5.59 | 1.49 | 16.47 | 5.63 | 1.33 | 16.35 | 5.92 | 1.45 | 16.63 |
| Zero-shot[LDA] | 5.74 | 1.73 | 17.21 | 6.05 | 1.73 | 16.97 | 6.09 | 1.59 | 16.76 | 6.60 | 1.94 | 17.18 |
| CC-tuning | 5.81 | 1.74 | 18.79 | 6.08 | 1.83 | 18.58 | 5.96 | 1.74 | 18.31 | 5.95 | 1.68 | 18.23 |
| CC-tuning[LDA] | 7.86 | 3.63 | 21.00 | 7.46 | 3.16 | 20.00 | 7.15 | 2.87 | 19.53 | 7.12 | 2.64 | 19.30 |
| MSC-tuning | 5.48 | 1.59 | 17.65 | 6.12 | 1.78 | 17.91 | 6.14 | 1.63 | 17.78 | 6.16 | 1.69 | 17.65 |
| MSC-tuning[LDA] | 10.70 | 5.63 | 23.31 | 10.03 | 5.12 | 21.55 | 9.07 | 4.06 | 20.19 | 8.96 | 4.01 | 19.94 |
| CC |
| Zero-shot | 9.53 | 4.82 | 22.76 | 9.22 | 4.43 | 22.18 | 9.15 | 4.48 | 22.18 | 8.99 | 4.43 | 22.10 |
| Zero-shot[LDA] | 8.94 | 4.44 | 21.54 | 8.34 | 4.03 | 21.00 | 8.28 | 3.82 | 20.67 | 8.12 | 3.81 | 20.54 |
| MSC-tuning | 8.37 | 3.88 | 22.93 | 8.49 | 3.99 | 22.96 | 7.97 | 3.75 | 22.15 | 7.60 | 3.70 | 21.87 |
| MSC-tuning[LDA] | 21.71 | 15.42 | 34.97 | 20.87 | 14.74 | 34.01 | 19.57 | 13.51 | 32.72 | 18.59 | 12.80 | 31.68 |
| CC-tuning | 15.89 | 9.90 | 30.59 | 15.97 | 10.06 | 30.27 | 16.10 | 10.31 | 30.54 | 15.10 | 9.34 | 29.43 |
| CC-tuning[LDA] | 25.69 | 19.53 | 39.67 | 25.93 | 19.72 | 39.15 | 25.82 | 19.40 | 39.05 | 24.26 | 18.16 | 37.61 | 

### 3.6 Generality Analysis

We further explore the generality of LD-Agent from two perspectives: cross-domain and cross-task capability.

##### Cross-domain Results

The cross-domain capability is important for open-domain dialogue task. Poor cross-domain capability, easily occuring on models tuned with specific datasets, largely limits its practicality in real-world environments. To explore the practicality of our tuned model in real-world, we conducts two cross-evaluation experiments on MSC and CC, two long-term dialogue datasets with significant domain gap due to the difference of collecting approaches, including manually annotating and LLM generation. Specifically, we first tune ChatGLM on CC, and test it on MSC. Then we tune ChatGLM on MSC, and test it on CC. The results are reported in Table [4](https://arxiv.org/html/2406.05925v1#S3.T4 "Table 4 ‣ Response Generation Analysis. ‣ 3.5 Human Evaluation ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). We can observe that the models tuned on one dataset still performs well on the other dataset, only with a slight performance decrease than the models tuned on the same dataset. Besides, cross-domain tuned model always outperforms zero-shot model with a large performance gap. These experiments fully demonstrate the powerful cross-domain capability and strong practical potential of our method.

##### Cross-task Results

The other capability worth exploring is the transferability of LD-Agent to different dialogue tasks. We explore the effectiveness of our method on multiparty dialogue, a task requires playing multiple roles simultaneously. We conduct our experiments on Ubuntu IRC dataset [[27](https://arxiv.org/html/2406.05925v1#bib.bib27)], a commonly used multiparty dialogue dataset. where our backbone adopts BART [[36](https://arxiv.org/html/2406.05925v1#bib.bib36)]. We compare our method with some previous multiparty dialogue methods, including GPT-2 [[37](https://arxiv.org/html/2406.05925v1#bib.bib37)], GSN [[27](https://arxiv.org/html/2406.05925v1#bib.bib27)], HeterMPC[BART] [[38](https://arxiv.org/html/2406.05925v1#bib.bib38)], and BART tuned without prompt. The results are reported at Table [5](https://arxiv.org/html/2406.05925v1#S3.T5 "Table 5 ‣ Cross-task Results ‣ 3.6 Generality Analysis ‣ 3 Experiments ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). It can be seen that BART tuned with LD-Agent obtained the state-of-the-art performance in most metrics, outperforming previous multiparty dialogue approach HeterMPC[BART], which also employs BART as backbone. This well proves the powerful task tranferability of LD-Agent.

Table 5: Multi-party performance on the Ubuntu IRC benchmark

 | Model | BL-1 | BL-2 | BL-3 | BL-4 | MET | R-L |
| --- | --- | --- | --- | --- | --- | --- |
| GPT-2 [[37](https://arxiv.org/html/2406.05925v1#bib.bib37)] | 10.37 | 3.60 | 1.66 | 0.93 | 4.01 | 9.53 |
| GSN [[27](https://arxiv.org/html/2406.05925v1#bib.bib27)] | 10.23 | 3.57 | 1.70 | 0.97 | 4.10 | 9.91 |
| HeterMPC[BART] [[38](https://arxiv.org/html/2406.05925v1#bib.bib38)] | 12.26 | 4.80 | 2.42 | 1.49 | 4.94 | 11.20 |
| BART [[36](https://arxiv.org/html/2406.05925v1#bib.bib36)] | 11.25 | 4.02 | 1.78 | 0.95 | 4.46 | 9.90 |
| BART[LDA] | 14.40 | 4.92 | 2.07 | 1.00 | 5.30 | 12.28 | 

## 4 Conclusion

In this work, we delved into the long-term open-domain dialogue agent to satisfy real-life chatbot demands of long-term companionship and personalized interactions. We introduced a model-agnostic long-term dialogue agent framework, LD-Agent, which comprehensively considers both historical events and user-agent personas to support coherent and consistent conversation. Our framework was capably decomposed into three learnable modules, significantly improving the adaptability and transferability. We conducted extensive experiments, well demonstrated the strong capability of LD-Agent to handle long-term dialogue tasks. The practicality of LD-Agent was also demonstrated through extensive experiments across multiple benchmarks, diverse models, and various tasks. Limited by the length of existing long-term dialogue datasets, the capability of LD-Agent in longer conversation scenarios is valuable to be explored.

## References

*   Xu et al. [2022a] Jing Xu, Arthur Szlam, and Jason Weston. Beyond goldfish memory: Long-term open-domain conversation. In *ACL*, pages 5180–5197, 2022a.
*   Zhang et al. [2022] Tong Zhang, Yong Liu, Boyang Li, Zhiwei Zeng, Pengwei Wang, Yuan You, Chunyan Miao, and Lizhen Cui. History-aware hierarchical transformer for multi-session open-domain dialogue system. In *EMNLP Findings*, pages 3395–3407, 2022.
*   Xu et al. [2022b] Xinchao Xu, Zhibin Gou, Wenquan Wu, Zheng-Yu Niu, Hua Wu, Haifeng Wang, and Shihang Wang. Long time no see! open-domain conversation with long-term persona memory. In *ACL Findings*, pages 2639–2650, 2022b.
*   Li et al. [2017] Yanran Li, Hui Su, Xiaoyu Shen, Wenjie Li, Ziqiang Cao, and Shuzi Niu. Dailydialog: A manually labelled multi-turn dialogue dataset. In *IJCNLP*, pages 986–995, 2017.
*   Zhang et al. [2018] Saizheng Zhang, Emily Dinan, Jack Urbanek, Arthur Szlam, Douwe Kiela, and Jason Weston. Personalizing dialogue agents: I have a dog, do you have pets too? In *ACL*, pages 2204–2213, 2018.
*   Rashkin et al. [2019] Hannah Rashkin, Eric Michael Smith, Margaret Li, and Y-Lan Boureau. Towards empathetic open-domain conversation models: A new benchmark and dataset. In *ACL*, pages 5370–5381, 2019.
*   Jang et al. [2023] Jihyoung Jang, Minseong Boo, and Hyounghun Kim. Conversation chronicles: Towards diverse temporal and relational dynamics in multi-session conversations. In *EMNLP*, pages 13584–13606, 2023.
*   Zhang et al. [2023a] Qiang Zhang, Jason Naradowsky, and Yusuke Miyao. Mind the gap between conversations for improved long-term dialogue generation. In *EMNLP Findings*, pages 10735–10762, 2023a.
*   Gu et al. [2019] Jia-Chen Gu, Zhen-Hua Ling, Xiaodan Zhu, and Quan Liu. Dually interactive matching network for personalized response selection in retrieval-based chatbots. In *EMNLP-IJCNLP*, pages 1845–1854, 2019.
*   Cao et al. [2022] Yu Cao, Wei Bi, Meng Fang, Shuming Shi, and Dacheng Tao. A model-agnostic data manipulation method for persona-based dialogue generation. In *ACL*, pages 7984–8002, 2022.
*   Zhao et al. [2023] Kang Zhao, Wei Liu, Jian Luan, Minglei Gao, Li Qian, Hanlin Teng, and Bin Wang. Unimc: A unified framework for long-term memory conversation via relevance representation learning. *CoRR*, abs/2306.10543, 2023.
*   Chen et al. [2019] Xiuyi Chen, Jiaming Xu, and Bo Xu. A working memory model for task-oriented dialog response generation. In *ACL*, pages 2687–2693, 2019.
*   Zhang et al. [2019] Zheng Zhang, Minlie Huang, Zhongzhou Zhao, Feng Ji, Haiqing Chen, and Xiaoyan Zhu. Memory-augmented dialogue management for task-oriented dialogue systems. *ACM Trans. Inf. Syst.*, 37(3):34:1–34:30, 2019.
*   Chen et al. [2023a] Liang Chen, Hongru Wang, Yang Deng, Wai-Chung Kwan, Zezhong Wang, and Kam-Fai Wong. Towards robust personalized dialogue generation via order-insensitive representation regularization. In *ACL Findings*, pages 7337–7345, 2023a.
*   Wu et al. [2020] Bowen Wu, Mengyuan Li, Zongsheng Wang, Yifu Chen, Derek F. Wong, Qihang Feng, Junhong Huang, and Baoxun Wang. Guiding variational response generator to exploit persona. In *ACL*, pages 53–65, 2020.
*   Liu et al. [2020] Qian Liu, Yihong Chen, Bei Chen, Jian-Guang Lou, Zixuan Chen, Bin Zhou, and Dongmei Zhang. You impress me: Dialogue generation via mutual persona perception. In *ACL*, pages 1417–1427, 2020.
*   Deng et al. [2023] Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samual Stevens, Boshi Wang, Huan Sun, and Yu Su. Mind2web: Towards a generalist agent for the web. In *NeurIPS*, 2023.
*   Wang et al. [2023a] Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, and Anima Anandkumar. Voyager: An open-ended embodied agent with large language models. *CoRR*, abs/2305.16291, 2023a.
*   Qian et al. [2023] Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, and Maosong Sun. Communicative agents for software development. *CoRR*, abs/2307.07924, 2023.
*   Park et al. [2023] Joon Sung Park, Joseph C. O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. Generative agents: Interactive simulacra of human behavior. In *UIST*, pages 2:1–2:22, 2023.
*   Zhang et al. [2023b] An Zhang, Leheng Sheng, Yuxin Chen, Hao Li, Yang Deng, Xiang Wang, and Tat-Seng Chua. On generative agents in recommendation. *CoRR*, abs/2310.10108, 2023b.
*   Cheng and Chin [2024] Junyan Cheng and Peter Chin. Sociodojo: Building lifelong analytical agents with real-world text and time series. In *ICLR*, 2024.
*   Hua et al. [2023] Wenyue Hua, Lizhou Fan, Lingyao Li, Kai Mei, Jianchao Ji, Yingqiang Ge, Libby Hemphill, and Yongfeng Zhang. War and peace (waragent): Large language model-based multi-agent simulation of world wars. *CoRR*, 2023.
*   Xu et al. [2024] Ruoxi Xu, Yingfei Sun, Mengjie Ren, Shiguang Guo, Ruotong Pan, Hongyu Lin, Le Sun, and Xianpei Han. AI for social science and social science of AI: A survey. *CoRR*, abs/2401.11839, 2024.
*   Zeng et al. [2023] Aohan Zeng, Xiao Liu, Zhengxiao Du, Zihan Wang, Hanyu Lai, Ming Ding, Zhuoyi Yang, Yifan Xu, Wendi Zheng, Xiao Xia, Weng Lam Tam, Zixuan Ma, Yufei Xue, Jidong Zhai, Wenguang Chen, Zhiyuan Liu, Peng Zhang, Yuxiao Dong, and Jie Tang. GLM-130B: an open bilingual pre-trained model. In *ICLR*, 2023.
*   Roller et al. [2021] Stephen Roller, Emily Dinan, Naman Goyal, Da Ju, Mary Williamson, Yinhan Liu, Jing Xu, Myle Ott, Eric Michael Smith, Y-Lan Boureau, and Jason Weston. Recipes for building an open-domain chatbot. In *EACL*, pages 300–325, 2021.
*   Hu et al. [2019] Wenpeng Hu, Zhangming Chan, Bing Liu, Dongyan Zhao, Jinwen Ma, and Rui Yan. GSN: A graph-structured network for multi-party dialogues. In *IJCAI*, pages 5010–5016, 2019.
*   Wang et al. [2020] Wenhui Wang, Furu Wei, Li Dong, Hangbo Bao, Nan Yang, and Ming Zhou. Minilm: Deep self-attention distillation for task-agnostic compression of pre-trained transformers. In *NeurIPS*, 2020.
*   Zhong et al. [2024] Wanjun Zhong, Lianghong Guo, Qiqi Gao, He Ye, and Yanlin Wang. Memorybank: Enhancing large language models with long-term memory. In *AAAI*, pages 19724–19731, 2024.
*   Wei et al. [2022a] Jason Wei, Maarten Bosma, Vincent Y. Zhao, Kelvin Guu, Adams Wei Yu, Brian Lester, Nan Du, Andrew M. Dai, and Quoc V. Le. Finetuned language models are zero-shot learners. In *ICLR*, 2022a.
*   Chen et al. [2021] Yulong Chen, Yang Liu, Liang Chen, and Yue Zhang. Dialogsum: A real-life scenario dialogue summarization dataset. In *ACL-IJCNLP Findings*, volume ACL/IJCNLP 2021, pages 5062–5074, 2021.
*   Wei et al. [2022b] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed H. Chi, Quoc V. Le, and Denny Zhou. Chain-of-thought prompting elicits reasoning in large language models. In *NeurIPS*, 2022b.
*   Papineni et al. [2002] Kishore Papineni, Salim Roukos, Todd Ward, and Wei-Jing Zhu. Bleu: a method for automatic evaluation of machine translation. In *ACL*, pages 311–318, 2002.
*   Lin [2004] Chin-Yew Lin. Rouge: A package for automatic evaluation of summaries. In *ACL*, 2004.
*   Banerjee and Lavie [2005] Satanjeev Banerjee and Alon Lavie. METEOR: an automatic metric for MT evaluation with improved correlation with human judgments. In *ACL Workshop*, pages 65–72, 2005.
*   Lewis et al. [2020] Mike Lewis, Yinhan Liu, Naman Goyal, Marjan Ghazvininejad, Abdelrahman Mohamed, Omer Levy, Veselin Stoyanov, and Luke Zettlemoyer. BART: denoising sequence-to-sequence pre-training for natural language generation, translation, and comprehension. In *ACL*, pages 7871–7880, 2020.
*   [37] Alec Radford, Karthik Narasimhan, Tim Salimans, and Ilya Sutskever. Improving language understanding by generative pre-training.
*   Gu et al. [2022] Jia-Chen Gu, Chao-Hong Tan, Chongyang Tao, Zhen-Hua Ling, Huang Hu, Xiubo Geng, and Daxin Jiang. Hetermpc: A heterogeneous graph neural network for response generation in multi-party conversations. In *ACL*, pages 5086–5097, 2022.
*   Wang et al. [2023b] Lanrui Wang, Jiangnan Li, Chenxu Yang, Zheng Lin, and Weiping Wang. Enhancing empathetic and emotion support dialogue generation with prophetic commonsense inference. *CoRR*, abs/2311.15316, 2023b.
*   Wang et al. [2024] Hongru Wang, Wenyu Huang, Yang Deng, Rui Wang, Zezhong Wang, Yufei Wang, Fei Mi, Jeff Z. Pan, and Kam-Fai Wong. Unims-rag: A unified multi-source retrieval-augmented generation for personalized dialogue systems. *CoRR*, abs/2401.13256, 2024.
*   Chen et al. [2023b] Siyuan Chen, Mengyue Wu, Kenny Q. Zhu, Kunyao Lan, Zhiling Zhang, and Lyuchun Cui. Llm-empowered chatbots for psychiatrist and patient simulation: Application and evaluation. *CoRR*, abs/2305.13614, 2023b.
*   Chen et al. [2023c] Yirong Chen, Xiaofen Xing, Jingkai Lin, Huimin Zheng, Zhenyu Wang, Qi Liu, and Xiangmin Xu. Soulchat: Improving llms’ empathy, listening, and comfort abilities through fine-tuning with multi-turn empathy conversations. In *EMNLP Findings*, pages 1170–1183, 2023c.
*   Deng et al. [2022] Yang Deng, Yaliang Li, Wenxuan Zhang, Bolin Ding, and Wai Lam. Toward personalized answer generation in e-commerce via multi-perspective preference modeling. *ACM Trans. Inf. Syst.*, 40(4):87:1–87:28, 2022.
*   Dillion et al. [2023] Danica Dillion, Niket Tandon, Yuling Gu, and Kurt Gray. Can ai language models replace human participants? *Trends in Cognitive Sciences*, 2023.
*   Shaikh et al. [2023] Omar Shaikh, Valentino Chai, Michele J. Gelfand, Diyi Yang, and Michael S. Bernstein. Rehearsal: Simulating conflict to teach conflict resolution. *CoRR*, abs/2309.12309, 2023.
*   Gao et al. [2023] Chen Gao, Xiaochong Lan, Zhihong Lu, Jinzhu Mao, Jinghua Piao, Huandong Wang, Depeng Jin, and Yong Li. S${}^{\mbox{3}}$: Social-network simulation system with large language model-empowered agents. *CoRR*, abs/2307.14984, 2023.
*   Huang et al. [2023] Xu Huang, Jianxun Lian, Yuxuan Lei, Jing Yao, Defu Lian, and Xing Xie. Recommender AI agent: Integrating large language models for interactive recommendations. *CoRR*, abs/2308.16505, 2023.
*   Shao et al. [2023] Yunfan Shao, Linyang Li, Junqi Dai, and Xipeng Qiu. Character-llm: A trainable agent for role-playing. In *EMNLP*, pages 13153–13187, 2023.
*   Zhou et al. [2023] Jinfeng Zhou, Zhuang Chen, Dazhen Wan, Bosi Wen, Yi Song, Jifan Yu, Yongkang Huang, Libiao Peng, Jiaming Yang, Xiyao Xiao, Sahand Sabour, Xiaohan Zhang, Wenjing Hou, Yijia Zhang, Yuxiao Dong, Jie Tang, and Minlie Huang. Characterglm: Customizing chinese conversational AI characters with large language models. *CoRR*, abs/2311.16832, 2023.
*   Wang et al. [2023c] Zekun Moore Wang, Zhongyuan Peng, Haoran Que, Jiaheng Liu, Wangchunshu Zhou, Yuhan Wu, Hongcheng Guo, Ruitong Gan, Zehao Ni, Man Zhang, Zhaoxiang Zhang, Wanli Ouyang, Ke Xu, Wenhu Chen, Jie Fu, and Junran Peng. Rolellm: Benchmarking, eliciting, and enhancing role-playing abilities of large language models. *CoRR*, abs/2310.00746, 2023c.
*   Kingma and Ba [2015] Diederik P. Kingma and Jimmy Ba. Adam: A method for stochastic optimization. In Yoshua Bengio and Yann LeCun, editors, *ICLR*, 2015.

## Appendix

In this Appendix, we discuss the following topics: (1): We elaborate on some related work about long-term open-domain dialogue and LLM-based autonomous agent in Appendix. [A](https://arxiv.org/html/2406.05925v1#A1 "Appendix A Related Work ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). (2): We visualize responses of original ChatGLM and LD-Agent to further demonstrate the ability of LD-Agent in long-term dialogue (see Appendix. [B](https://arxiv.org/html/2406.05925v1#A2 "Appendix B Response Visualization ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue")). (3): More detailed experimental settings are introduced in Appendix. [C](https://arxiv.org/html/2406.05925v1#A3 "Appendix C Detailed Evaluation Settings ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). (4): In the Appendix. [D](https://arxiv.org/html/2406.05925v1#A4 "Appendix D Prompt ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"), the prompts utilized in LD-Agent is illustrated.

## Appendix A Related Work

### A.1 Long-term Open-domain Dialogue

Open-domain dialogue aims to develop a human-like chatbot that can emulate human conversation, facilitating free-flowing dialogue on a wide range of topics. However, the dialogue’s extent in earlier studies is often limited by conversation length, focusing primarily on brief conversations of about 2-15 turns within a single session [[4](https://arxiv.org/html/2406.05925v1#bib.bib4), [5](https://arxiv.org/html/2406.05925v1#bib.bib5), [6](https://arxiv.org/html/2406.05925v1#bib.bib6)]. To support more realistic and extended conversations, a series of studies have explored the role of both external [[39](https://arxiv.org/html/2406.05925v1#bib.bib39), [40](https://arxiv.org/html/2406.05925v1#bib.bib40)] and internal knowledge [[2](https://arxiv.org/html/2406.05925v1#bib.bib2), [3](https://arxiv.org/html/2406.05925v1#bib.bib3)] on maintaining the feasibility of long-term dialogue. Commonly referenced external knowledge, such as commonsense [[40](https://arxiv.org/html/2406.05925v1#bib.bib40)], medical [[41](https://arxiv.org/html/2406.05925v1#bib.bib41)], and psychological [[42](https://arxiv.org/html/2406.05925v1#bib.bib42)] knowledge, serves as supplementary guidance for the reasoning process, ensuring logical coherence in extended contexts. In parallel, internal knowledge captured dynamically during long conversations generally contains historical events [[1](https://arxiv.org/html/2406.05925v1#bib.bib1), [8](https://arxiv.org/html/2406.05925v1#bib.bib8), [2](https://arxiv.org/html/2406.05925v1#bib.bib2), [7](https://arxiv.org/html/2406.05925v1#bib.bib7)] and personas [[9](https://arxiv.org/html/2406.05925v1#bib.bib9), [3](https://arxiv.org/html/2406.05925v1#bib.bib3), [10](https://arxiv.org/html/2406.05925v1#bib.bib10), [43](https://arxiv.org/html/2406.05925v1#bib.bib43)]. Historical events are typically summarized and stored into a memory bank to maintain dialogue coherence across sessions, while interlocutors’ personas are maintained via a dynamic persona memory bank, which ensures character consistency in long-term conversations. In this study, we focus on the internal knowledge to integrate dynamically updated historical events and personas to conduct long-term personalized conversations.

### A.2 LLM-based Autonomous Agents

AI Agent conception is geared towards autonomous environmental perception, decision-making, and problem-solving capabilities. With the large language models (LLMs) underlining their impressive generalization potential, leading to their widespread adoption as substitutes for human operators in various research fields [[17](https://arxiv.org/html/2406.05925v1#bib.bib17), [19](https://arxiv.org/html/2406.05925v1#bib.bib19), [44](https://arxiv.org/html/2406.05925v1#bib.bib44), [21](https://arxiv.org/html/2406.05925v1#bib.bib21)]. Generally, these agents can be categorized into task-oriented agents [[17](https://arxiv.org/html/2406.05925v1#bib.bib17), [18](https://arxiv.org/html/2406.05925v1#bib.bib18), [19](https://arxiv.org/html/2406.05925v1#bib.bib19), [29](https://arxiv.org/html/2406.05925v1#bib.bib29)] and simulation-oriented agents [[44](https://arxiv.org/html/2406.05925v1#bib.bib44), [45](https://arxiv.org/html/2406.05925v1#bib.bib45), [46](https://arxiv.org/html/2406.05925v1#bib.bib46), [21](https://arxiv.org/html/2406.05925v1#bib.bib21), [47](https://arxiv.org/html/2406.05925v1#bib.bib47)]. Task-oriented agents are designed to accurately perform and achieve predefined tasks, as seen in applications for web assistance [[17](https://arxiv.org/html/2406.05925v1#bib.bib17)], game-playing [[18](https://arxiv.org/html/2406.05925v1#bib.bib18)], and software development [[19](https://arxiv.org/html/2406.05925v1#bib.bib19)]. On the other hand, simulation-oriented agents are devised to emulate human emotive and cognitive behaviors, having played roles in psychological studies [[44](https://arxiv.org/html/2406.05925v1#bib.bib44)], social networking platforms [[46](https://arxiv.org/html/2406.05925v1#bib.bib46)], conflict resolution scenarios [[45](https://arxiv.org/html/2406.05925v1#bib.bib45)], and recommendation systems [[21](https://arxiv.org/html/2406.05925v1#bib.bib21), [47](https://arxiv.org/html/2406.05925v1#bib.bib47)]. In addition, recent developments have seen the advent of individual-level agents that are utilized to simulate specific character behaviors, enhancing the realism and personalization of user-agent interactions [[48](https://arxiv.org/html/2406.05925v1#bib.bib48), [49](https://arxiv.org/html/2406.05925v1#bib.bib49), [50](https://arxiv.org/html/2406.05925v1#bib.bib50)]. This paper falls into the simulation-oriented agent to build a human-like open-domain dialogue agent with memory retrieved and character analysis modules.

## Appendix B Response Visualization

To further analyze the ability of LD-Agent in long-term dialogue, we illustrate an example in Figure [4](https://arxiv.org/html/2406.05925v1#A2.F4 "Figure 4 ‣ Appendix B Response Visualization ‣ Hello Again! LLM-powered Personalized Agent for Long-term Dialogue"). It can be seen that the response generated by LD-Agent successfully captures the information about “General Nathan Bedford Forrest” they talked about in the history session.

![Refer to caption](img/5261bea4dd1918b41a76c97117a1c5b7.png)

Figure 4: Example of separately chatting with original ChatGLM and ChatGLM with LD-Agent. A more relevant response to history conversation is generated.

## Appendix C Detailed Evaluation Settings

In this section, we introduce the detailed experimental dataset, evaluation metrics, baseline models, and our implementation details.

### C.1 Datasets

##### Multi-session Dataset.

Our experiments are conducted on two illustrative multi-session datasets: MSC [[1](https://arxiv.org/html/2406.05925v1#bib.bib1)] and CC [[7](https://arxiv.org/html/2406.05925v1#bib.bib7)]. Both datasets feature 5 sessions, with approximately 50 conversational turns per sample. MSC extends the PersonaChat dataset [[5](https://arxiv.org/html/2406.05925v1#bib.bib5)], utilizing PersonaChat for the initial session and employing human-human crowd workers to simulate the dialogues in subsequent sessions. The time intervals between sessions can span several days, and the dataset includes records of the participants’ personas. We follow the split of [[2](https://arxiv.org/html/2406.05925v1#bib.bib2)] with 4,000 conversations for training, 500 conversations for validation, and 501 conversations for testing. CC is complied by ChatGPT, which guides interactions according to a predefined event graph and participant relationships, with time intervals between sessions extending over several years. We employ the same data scale as MSC, with 4,000 conversations for training, 500 conversations for validation, and 501 conversations for testing.

##### Multi-party Dataset.

To explore the transferability of LD-Agent on other dialogue tasks. We apply our method to the Ubuntu IRC benchmark [[27](https://arxiv.org/html/2406.05925v1#bib.bib27)], a dataset of multiparty tasks. We follow the split of previous works [[27](https://arxiv.org/html/2406.05925v1#bib.bib27), [38](https://arxiv.org/html/2406.05925v1#bib.bib38)] with 311,725 dialogues for training, 5,000 dialogues for validation, and 5,000 dialogues for testing.

### C.2 Metrics

##### Automatic Evaluation Metrics.

BLEU-N [[33](https://arxiv.org/html/2406.05925v1#bib.bib33)] (BL-N) and ROUGE-L [[34](https://arxiv.org/html/2406.05925v1#bib.bib34)] (R-L) metrics are commonly used automatic evaluation metrics in dialogue generation tasks. BLEU-N measures N-gram overlaps between the generated text and the reference text, while ROUGE-L focuses on sequential coherence. We employ the METEOR (MET) [[35](https://arxiv.org/html/2406.05925v1#bib.bib35)] metric in multi-party tasks as a complement to the BLEU metric, enhancing it with synonym calculation capabilities. In addition, accuracy (ACC) is calculated to measure the classification accuracy of different persona extractors.

##### Human Evaluation Metrics.

In human evaluation, we evaluate LD-Agent on three aspects: coherence, fluency, and engagingness. Coherence measures the chatbot’s capabilities to maintain the coherence of topic and logic across sessions. Fluency reflects the natural and fluent degree of interactions, making the interaction similar to human-human interactions. Engagingness measures a user’s interest in interacting with the target chatbot.

### C.3 Baselines

To validate the effectiveness of our method on various baselines, we employ LD-Agent on both online and offline models, tuned and zero-shot models, LLMs, and non-LLMs.

*   •

    HAHT [[2](https://arxiv.org/html/2406.05925v1#bib.bib2)]: This is the state-of-the-art model crafted for multi-session, open-domain dialogue. It encodes all historical information and utilizes an attention mechanism to capture the relevant information to an ongoing conversation.

*   •

    BlenderBot [[26](https://arxiv.org/html/2406.05925v1#bib.bib26)]: This is a commonly used large-scale open-domain dialogue model pre-trained on online social discussion data.

*   •

    ChatGLM3 [[25](https://arxiv.org/html/2406.05925v1#bib.bib25)]: This is an offline large language model 6B parameters. The model is pre-trained on 1T corpus, performing remarkable zero-shot reasoning capabilities.

*   •

    ChatGPT: This is an online large language model based on the GPT architecture with excellent human-like cognitive and reasoning abilities. In this paper, we use the API service with the model of “gpt-3.5-turbo-1106”.

*   •

    BART [[36](https://arxiv.org/html/2406.05925v1#bib.bib36)]: This is a denoising autoencoder with transformer architecture, trained to reconstruct original text from corrupting text.

### C.4 Implementation Details.

For the event summarizer, persona extractor, and response generator modules, we employ the LoRA mechanism across all configurations. All training and evaluation operated on a single NVIDIA A100 GPU. For the ChatGLM3-6B, it is optimized by an Adam [[51](https://arxiv.org/html/2406.05925v1#bib.bib51)] optimizer with the learning rate of 5e-5\. We configure this model with a batch size of 4 and train it over 3 epochs. For BlenderBot, the initial learning rate is set to 2e-5, with the batch size and the number of training epochs set at 4 and 5, respectively.

## Appendix D Prompt

In this section, we separately provide the illustrations of the prompts used in the Event Module, Persona Module, and Response Module.

### D.1 Prompt of Event Summary

<svg class="ltx_picture" height="107.83" id="A4.SS1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,107.83) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 91.06)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">Prompt 1:  Event Summary Prompt</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">$\mathcal{SYS\ PROMPT}$: You are good at extracting events and summarizing them in brief sentences. You will be shown a conversation between $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$ and $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ name\}$.    $\mathcal{USER\ PROMPT}$: Conversation: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{context\}$. Based on the Conversation, please summarize the main points of the conversation with brief sentences in English, within 20 words. SUMMARY:</foreignobject></g></g></svg>

### D.2 Prompt of Persona Extraction

<svg class="ltx_picture" height="192.39" id="A4.SS2.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,192.39) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 175.61)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">Prompt 2:  Persona Extraction Prompt</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="146.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">$\mathcal{SYS\ PROMPT}$: You excel at extracting user personal traits from their words, a renowned local communication expert.    $\mathcal{USER\ PROMPT}$: If no traits can be extracted in the sentence, you should reply NO$\_$TRAIT. Given you some format examples of traits extraction, such as: 1\. No, I have no longer serve in the millitary, I had served up the full term that I signed up for, and now work outside of the millitary. Extracted Traits: I now work elsewhere. I used to be in the military. 2\. That must a been some kind of endeavor. Its great that people are aware of issues that arise in their homes, otherwise it can be very problematic in the future. NO$\_$TRAIT Please extract the personal traits who said this sentence (no more than 20 words):$\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{sentence\}$</foreignobject></g></g></svg>

### D.3 Prompt of Response Generation

<svg class="ltx_picture" height="142.58" id="A4.SS3.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,142.58) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 125.8)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">Prompt 3:  Base Response Generation Prompt</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="96.86" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">$\mathcal{SYS\ PROMPT}$: As a communication expert with outstanding communication habits, you embody the role of $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ name\}$ throughout the following dialogues.    $\mathcal{USER\ PROMPT}$: <CONTEXT> Drawing from your recent conversation with $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{context\}$ Now, please role-play as $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ name\}$ to continue the dialogue between $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ name\}$ and $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$. $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$ just said: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{input\}$ Please respond to $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$’s statement using the following format (maximum 30 words, must be in English): RESPONSE:</foreignobject></g></g></svg><svg class="ltx_picture" height="192.39" id="A4.SS3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,192.39) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 175.61)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">Prompt 4:  Agent Response Generation Prompt</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="146.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">$\mathcal{SYS\ PROMPT}$: As a communication expert with outstanding communication habits, you embody the role of $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ name\}$ throughout the following dialogues. Here are some of your distinctive personal traits: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ traits\}$.    $\mathcal{USER\ PROMPT}$: <CONTEXT> Drawing from your recent conversation with $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{context\}$ <MEMORY> The memories linked to the ongoing conversation are: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{memories\}$ <USER  TRAITS> In recent conversations with this user, you’ve noticed that $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$ possesses the following personal traits: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ traits\}$ Now, please role-play as $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ name\}$ to continue the dialogue between $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% agent\ name\}$ and $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$. $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$ just said: $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{input\}$ Please respond to $\color[rgb]{.75,.5,.25}\definecolor[named]{pgfstrokecolor}{rgb}{.75,.5,.25}\{% user\ name\}$’s statement using the following format (maximum 30 words, must be in English): RESPONSE:</foreignobject></g></g></svg>