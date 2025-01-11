<!--yml
category: 未分类
date: 2025-01-11 12:35:27
-->

# Large Language Model Sentinel: LLM Agent for Adversarial Purification

> 来源：[https://arxiv.org/html/2405.20770/](https://arxiv.org/html/2405.20770/)

Guang Lin^(1, 2), Qibin Zhao^(2, 1,) Corresponding Author

###### Abstract

Over the past two years, the use of large language models (LLMs) has advanced rapidly. While these LLMs offer considerable convenience, they also raise security concerns, as LLMs are vulnerable to adversarial attacks by some well-designed textual perturbations. In this paper, we introduce a novel defense technique named Large LAnguage MOdel Sentinel (LLAMOS), which is designed to enhance the adversarial robustness of LLMs by purifying the adversarial textual examples before feeding them into the target LLM. Our method comprises two main components: a) Agent instruction, which can simulate a new agent for adversarial defense, altering minimal characters to maintain the original meaning of the sentence while defending against attacks; b) Defense guidance, which provides strategies for modifying clean or adversarial examples to ensure effective defense and accurate outputs from the target LLMs. Remarkably, the defense agent demonstrates robust defensive capabilities even without learning from adversarial examples. Additionally, we conduct an intriguing adversarial experiment where we develop two agents, one for defense and one for attack, and engage them in mutual confrontation. During the adversarial interactions, neither agent completely beat the other. Extensive experiments on both open-source and closed-source LLMs demonstrate that our method effectively defends against adversarial attacks, thereby enhancing adversarial robustness.

## Introduction

![Refer to caption](img/07e38b1226dceb875410f07e0a2dd2a6.png)

Figure 1: Illustration of adversarial attacks and adversarial purification on large language models. (a) When the clean example $x$ is input into the target LLM, a correct label will be output. However, after the attack, the adversarial example $x^{\prime}$ will make the target LLM outputs an incorrect label. We use LLAMOS (b) to purify the adversarial example before inputting it into the target LLM, ensuring that the purified example $\hat{x}$ can be predicted as the correct label.

Large language models (LLMs) have garnered significant attention due to their impressive performance across a wide range of natural language tasks (Minaee et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib34)). The pre-trained LLMs, such as Meta’s LLAMA (Touvron et al. [2023a](https://arxiv.org/html/2405.20770v3#bib.bib52), [b](https://arxiv.org/html/2405.20770v3#bib.bib53)) and OpenAI’s ChatGPT (OpenAI [2022](https://arxiv.org/html/2405.20770v3#bib.bib37); Achiam et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib1)), have become essential foundations for AI applications in various sectors such as healthcare, education, and visual tasks (Kasneci et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib19); Thirunavukarasu et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib51); OpenAI [2023](https://arxiv.org/html/2405.20770v3#bib.bib38); Köpf et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib20); Romera-Paredes et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib42)). Despite their widespread use and convenience, concerns about the security of these models are increasing. Specifically, LLMs have been shown to be vulnerable to adversarial textual examples (Wang et al. [2023a](https://arxiv.org/html/2405.20770v3#bib.bib55); Xu et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib63)), which involve subtle modifications to textual content that maintain the same meaning for humans but completely change the prediction results to LLMs, often with severe consequences.

To achieve robust defense against adversarial attacks on LLMs, a prevalent strategy is fine-tuning the LLMs with adversarial examples to enhance model alignment (Shen et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib43); Wang et al. [2023c](https://arxiv.org/html/2405.20770v3#bib.bib59)). LLM-based adversarial fine-tuning (AFT) can be implemented either through in-context learning (Dong et al. [2022](https://arxiv.org/html/2405.20770v3#bib.bib12); Xiang et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib62)) or by optimizing the parameters of pre-trained LLMs using adversarial examples (Dettmers et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib11); Li et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib24)). However, LLM-based AFT methods necessitate additional computational resources and training time. Achieving robust and reliable LLMs typically requires significant costs (Hu et al. [2022](https://arxiv.org/html/2405.20770v3#bib.bib18)), which is prohibitive for ordinary users. Additionally, due to the discrete nature of textual information, these adversarial examples can have substitutes in any token of the sentence, with each having a large candidate list (Li, Song, and Qiu [2023](https://arxiv.org/html/2405.20770v3#bib.bib22)). This leads to a combinatorial explosion, making the application of AFT methods challenging or resulting in poor generalization when trained on a limited dataset of adversarial examples. Consequently, developing an efficient and user-friendly robust LLM system remains a huge challenge and an urgent issue that continues to be addressed.

In this paper, focusing on adversarial textual attacks targeting LLM-based classification tasks, we propose a novel defense technique named Large LAnguage MOdel Sentinel (LLAMOS), which utilizes the LLM as a defense agent for adversarial purification, as illustrated in Figure 1a. To streamline our explanation, we condense certain details in Figure 1b, with comprehensive instructions provided in the Methods Section. Specifically, LLAMOS comprises two components: Agent instruction, which can simulate a new agent for adversarial defense, altering minimal characters to maintain the original meaning of the sentence, and Defense guidance, which provides strategies for modifying clean or adversarial examples to ensure effective defense and accurate outputs from the target LLMs. LLAMOS serves as a pre-processing method aiming to eliminate harmful information from potentially attacked textual inputs before feeding them into the target LLM for classification. In contrast to the AFT method, the LLM-based AP method functions as an additional module capable of defending against adversarial attacks without necessitating fine-tuning of the target LLM.

We comprehensively evaluate the performance of our method on GLUE datasets, conducting experiments with both representative open-source and closed-source LLMs, LLAMA-2 (Touvron et al. [2023b](https://arxiv.org/html/2405.20770v3#bib.bib53)) and GPT-3.5 (OpenAI [2022](https://arxiv.org/html/2405.20770v3#bib.bib37)). The experimental results demonstrate that the LLM-based AP method effectively defends against adversarial attacks. Specifically, our proposed method achieves a maximum reduction in the attack success rate (ASR) by up to 45.59% and 37.86% with LLAMA-2 and GPT-3.5, respectively. Additionally, we observe that the initial defense agent fails to achieve the expected results under some obvious attacks. Therefore, we employ the in-context learning (Dong et al. [2022](https://arxiv.org/html/2405.20770v3#bib.bib12)) to further optimize the defense agent, significantly enhancing the defense capabilities almost without adding any additional costs. Finally, we conduct an intriguing online adversarial experiment, creating an adversarial system using two LLM-based agents (one for defense and one for attack) along with a target LLM for classification. During the adversarial interaction, the defense agent and attack agent continuously counter each other, resembling the adversarial training process in traditional image tasks (Goodfellow, Shlens, and Szegedy [2015](https://arxiv.org/html/2405.20770v3#bib.bib16)).

Our contributions are summarized as follows.

*   •

    We propose a novel defense technique named LLAMOS, which aims to purify the adversarial textual examples before feeding them into the target LLM. To the best of our knowledge, we are the first to employ an LLM agent to enhance the adversarial robustness of LLMs.

*   •

    The defense agent in LLAMOS is a plug-and-play module, serving as a pre-processing step. Notably, it operates without retraining of target LLM, rendering it efficient and user-friendly.

*   •

    We conduct extensive experiments to empirically demonstrate that the proposed method can effectively defend against adversarial attacks.

## Preliminary

This section briefly reviews the adversarial attacks and evaluations of adversarial robustness on LLMs.

### Adversarial Attcks on LLMs

Given a target LLM $f_{t}$ with task instruction, input $x$, and correct output $y$, the adversarial attacks aim to find the adversarial examples $x^{\prime}$ that can fool the target LLM $f_{t}$ on classification tasks. The adversarial examples $x^{\prime}$ can be obtained by the LLM itself $f_{atk}$ with different system prompt (Xu et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib63)),

|  | $x^{\prime}=f_{atk}(x)=x+\delta,\quad f_{t}(x^{\prime})=y^{\prime}\neq y,$ |  |

where $\delta$ represents textual perturbations from a series of candidate sets for modifications, which are made at the character level, word level, or sentence level. In a specific instance, the system prompt of $f_{t}$ can be: “Analyze the tone of this statement and respond with either ‘positive’ or ‘negative’.” and the system prompt of corresponding $f_{atk}$ can be: “Your task is to generate a new sentence that keeps the same semantic meaning as the original one but be classified as a different label.” There are more details in Appendix B.

### Evaluations of LLMs Robustness

To evaluate the effectiveness of the defense method, we follow the setting from Wang et al. ([2021](https://arxiv.org/html/2405.20770v3#bib.bib57)); Xu et al. ([2024](https://arxiv.org/html/2405.20770v3#bib.bib63)), using the attack success rate (ASR) and traditional robust accuracy (RA) on the adversarial examples as measures of the robustness of the defense method.

|  | $\text{ASR}=\frac{\sum_{(x,y)\in D}\mathds{1}\left[f_{t}(x^{\prime})\neq y% \right]\cdot\mathds{1}\left[f_{t}(x)=y\right]}{\sum_{(x,y)\in D}\mathds{1}% \left[f_{t}(x)=y\right]},$ |  | (1) |

|  | $\text{RA}=\frac{1}{N}\sum_{(x^{\prime},y)\in D^{\prime}}\mathds{1}\left[f_{t}(% x^{\prime})=y\right],$ |  | (2) |

where $\mathds{1}[\cdot]\in\{0,1\}$ is an indicator function. $D$ is the original test dataset and $D^{\prime}$ is the adversarial example dataset, and $N$ is the number of examples. The lower the ASR, the higher the RA, indicating greater model robustness.

## Methods

We propose a novel defense technique for large language model-based adversarial purification (LLAMOS), which purifies adversarial examples by the LLM-based defense agent before feeding examples into the target LLM. Firstly, we outline the overall pipeline of LLAMOS. Subsequently, we further augment the defense agent using in-context learning. Finally, we present the design of the adversarial system, incorporating the defense agent, attack agent, and target LLM.

### Overview of LLAMOS

To defend against adversarial textual attacks targeting LLM-based classification tasks, we propose Large LAnguage MOdel Sentinel (LLAMOS) that employs the LLM as a defense agent for adversarial purification. LLAMOS comprises two components: Agent instruction and Defense guidance. Next, we introduce the overall pipeline in sequential order.

In this paper, we utilize the existing LLMs denoted by target LLMs $f_{t}$ as classifiers. Given an adversarial example $x^{\prime}$, the target LLM $f_{t}$ outputs the incorrect label $y^{\prime}=f_{t}(x^{\prime})$, while after the purification, the purified example $\hat{x}=g_{stl}(x^{\prime})$ can be predicted as the correct label $y=f_{t}(g_{stl}(x^{\prime}))$. To achieve this, we design prompts for generating a defense agent $g_{stl}$ as described in the following.

{mdframed}

# Defense Agent Instruction
To begin, let me provide a brief overview of the input text: [Input Description]. The classification task for these sentences is [Task Description]. However, be aware that these sentences might be susceptible to adversarial attacks, which could lead to an incorrect label. Note that not all sentences will be affected by the attacks. Your task is to generate a new sentence that replaces the original one, which must satisfy the following conditions: [Defense Goal].
# Defense Guidance
You can complete the task using the following guidance: [Defense Guidance].
Input: [Input]. Now, let’s start the defense process and only output the generated sentence.

Input Description. The format of Input $x_{in}\in\{D_{i},i=1...6\}$ varies significantly across different datasets $D_{i}$, necessitating tailored input formats $x_{in}$ to correspond with the specific structure and content of each dataset, details in Table 10\. For instance, the SST-2 dataset (Socher et al. [2013](https://arxiv.org/html/2405.20770v3#bib.bib46)) typically consists of a single sentence per data point. On the other hand, the MNLI dataset (Williams, Nangia, and Bowman [2018](https://arxiv.org/html/2405.20770v3#bib.bib61)) is structured to include pairs of sentences labeled as premise and hypothesis.

Task Description. Similar to the input descriptions, the tasks associated with each dataset $D_{i}$ are distinct. As illustrated earlier, SST-2 focuses on determining the sentiment of a given sentence, making it a straightforward classification challenge. Conversely, MNLI presents a more complex task of natural language inference, where the relationship between a pair of sentences must be discerned and classified correctly. Detailed input and task descriptions are provided in Table 9¹¹1Tables 9 to 12 are in the Appendix..

Defense Goal. Based on traditional adversarial purification methods (Shi, Holtz, and Mishne [2021](https://arxiv.org/html/2405.20770v3#bib.bib44); Srinivasan et al. [2021](https://arxiv.org/html/2405.20770v3#bib.bib47); Lin et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib26), [c](https://arxiv.org/html/2405.20770v3#bib.bib27)), we design the goal for LLAMOS as follows: “Keeping the semantic meaning of the new sentence the same as the original one; For natural examples, the new sentence should remain unchanged. For adversarial examples, modify the sentence so that it is classified as the correct label, effectively reversing the adversarial effect.”

Defense Guidance. The defense guidance offers specific instructions to the defense agent on how to modify the input text to ensure effective defense and accurate outputs from the target LLM. In designing our guidance, we considered attacks at various levels (Xu et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib63)), including character, word, and sentence levels, which are presented in [Table 1](https://arxiv.org/html/2405.20770v3#Sx3.T1 "In Overview of LLAMOS ‣ Methods ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"). These guidances are not rigidly fixed; they can be fine-tuned according to specific tasks.

Table 1: The defense guidances for defense agent.

|       Index | [Defense Guidance] |
| --- | --- |
|       1 | Modify as few characters as possible. |
|       2 | Correct any clear spelling errors. |
|       3 | Eliminate redundant symbols. |
|       4 | If necessary, feel free to replace, delete, add words, or adjust the word order. |
|       5 | Improve structure for better readability. |
|       6 | Ensure sentence is coherent and logical. |

After the defense agent generates the new sentence, the purified example $\hat{x}=g_{stl}(x^{\prime})$ is input into the target LLM $f_{t}$ for classification.

### Enhencing the Defense with In-Context Learning

In the initial defense agent, the defense guidance relies on common sense, which may result in poor performance against some special attacks, even when the attacker adds obvious characters. To address this limitation, we introduce ICL (Dong et al. [2022](https://arxiv.org/html/2405.20770v3#bib.bib12)) to further optimize the defense agent by,

{mdframed}

# In-Context Learning
The new sentence still contains a lot of harmful content caused by adversarial attacks, such as [Specific Guidance]. Please consider these contents and output a new sentence for me.
Input: [Input]. Now, let’s start the defense process and only output the generated sentence.

The specific guidance is designed to assist the defense agent in better understanding an attack and generating a new sentence capable of effectively defending against the attack. These guidelines can be fine-tuned to address specific attacks and can be incorporated into the defense agent as needed. Through in-context learning, the defense agent can be continuously optimized, significantly enhancing its performance almost without adding any additional costs.

### Adversarial System with Multiple LLMs

In this section, we devise an adversarial system involving multiple LLMs. Given that our method introduces a defense agent against attackers, a natural idea is to then create an attack agent to counter the defender. The attack agent is tasked with generating adversarial examples from purified examples to deceive the target LLM once more. To accomplish this, we design prompts for generating an attack agent $g_{atk}$ by,

{mdframed}

# Attack Agent Instruction
To begin, let me provide a brief overview of the input text: [Input Description]. The classification task for these sentences is [Task Description]. Your task is to generate a new sentence that replaces the original one, which must satisfy the following conditions: [Attack Instruction].
# Attack Guidance
For example, the original sentence [Purified Example] is classified as [Correct Label]. You should generate a new sentence which is classified as [Incorrect Label].
Input: [Input]. Now, let’s start the attack process and only output the generated sentence.

The prompt structure of the attack agent and the defense agent is basically the same, although there are some differences in details. The input description of the attack agent includes the correct label $y$, and the input format is $(x_{in},y)\in D$. The attack instruction is “1\. The new sentence should be classified as the opposite of the ‘correct label’. 2\. Change at most two letters in the sentence.” Finally, we provide a specific example to help the attack agent better understand the attack task.

Then, we combine the defense agent and attack agent to form an adversarial system, as illustrated in [Figure 2](https://arxiv.org/html/2405.20770v3#Sx3.F2 "In Adversarial System with Multiple LLMs ‣ Methods ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"). In the adversarial system, the purified examples can be attacked again by the attack agent, and likewise, the adversarial examples can also be purified by the defense agent. They continuously counter each other, much like adversarial training (Goodfellow, Shlens, and Szegedy [2015](https://arxiv.org/html/2405.20770v3#bib.bib16)).

![Refer to caption](img/adade25df048cbae85bf78c045d86cdd.png)

Figure 2: Adversarial system with multiple LLMs, including the defense agent, attack agent, and target LLM.

Table 2: The attack success rate (ASR) defense against PromptAttack-EN and PromptAttack-FS-EN on the GLUE dataset with GPT-3.5\. The lower the ASR, the greater the model robustness.

|       Attacks | LLAMOS | SST-2 | RTE | QQP | QNLI | MNLI-mm | MNLI-m | Avg. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|       PA-EN | $\times$ | 56.00 | 34.30 | 37.03 | 40.39 | 43.51 | 44.00 | 42.25 |
| ✓ | 23.77 | 8.91 | 16.11 | 11.69 | 5.65 | 17.66 | 13.22 |
|       PA-FS-EN | $\times$ | 75.23 | 36.12 | 39.61 | 49.00 | 44.10 | 45.97 | 48.81 |
| ✓ | 48.94 | 9.58 | 16.49 | 14.33 | 7.73 | 19.05 | 19.42 |

Table 3: The attack success rate (ASR) defense against PA-EN and PA-FS-EN on the SST-2 dataset with LLAMA-2.

|        Defenses | PA-EN | PA-FS-EN |
| --- | --- | --- |
|        Vanilla | 66.77 | 48.39 |
|        LLAMOS | 21.18 | 37.82 |

Table 4: Standard accuracy and average robust accuracy on LLAMA-2 defense against three types of PromptAttack: Character-level, Word-level, and Sentence-level attacks.

|       FS | Standard | w/o LLAMOS | Character | Word | Sentence |
| --- | --- | --- | --- | --- | --- |
|       $\times$ | 92.18 | 47.93 | 83.59 | 85.03 | 84.62 |
|       ✓ | 92.18 | 30.42 | 85.16 | 79.13 | 68.96 |

## Related Work

Adversarial Attack. Deep neural networks (DNNs) are vulnerable to adversarial examples (Szegedy et al. [2014](https://arxiv.org/html/2405.20770v3#bib.bib49)), which are generated by adding small, human-imperceptible perturbations to natural examples, but completely change the prediction results to DNNs (Goodfellow, Shlens, and Szegedy [2015](https://arxiv.org/html/2405.20770v3#bib.bib16); Lin et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib26)). With the rapidly increasing applications of LLMs (OpenAI [2023](https://arxiv.org/html/2405.20770v3#bib.bib38); Köpf et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib20); Romera-Paredes et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib42)), security concerns have emerged as a critical area of research (Gehman et al. [2020](https://arxiv.org/html/2405.20770v3#bib.bib13); Bender et al. [2021](https://arxiv.org/html/2405.20770v3#bib.bib3); Mckenna et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib33); Manakul, Liusie, and Gales [2023](https://arxiv.org/html/2405.20770v3#bib.bib32); Liu et al. [2023b](https://arxiv.org/html/2405.20770v3#bib.bib29); Zhu et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib68); Li, Song, and Qiu [2023](https://arxiv.org/html/2405.20770v3#bib.bib22); Qi et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib40); Yao et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib67)), with researchers increasingly focusing on adversarial attacks targeting LLMs. In a similar setup to DNNs, for LLMs, attackers manipulate a small amount of text to change the output of the target LLM while maintaining the semantic information for humans (Wang et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib56); Xu et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib63)). Presently, addressing the security issues surrounding LLMs is of paramount importance and requires urgent attention.

Adversarial Defense. There are two main defense techniques on traditional DNNs, including adversarial training (AT) (Goodfellow, Shlens, and Szegedy [2015](https://arxiv.org/html/2405.20770v3#bib.bib16)) and adversarial purification (AP) (Shi, Holtz, and Mishne [2021](https://arxiv.org/html/2405.20770v3#bib.bib44); Srinivasan et al. [2021](https://arxiv.org/html/2405.20770v3#bib.bib47)). Unlike traditional DNNs, retraining LLMs is nearly impossible due to cost issues (Li, Song, and Qiu [2023](https://arxiv.org/html/2405.20770v3#bib.bib22)). Therefore, most methods enhance the robustness of LLMs through adversarial fine-tuning (AFT) (Xiang et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib62); Li et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib24); Bianchi et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib5); Deng et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib10); Qi et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib40)). While AFT can effectively defend against attacks, it remains susceptible to unseen attacks whose adversarial examples that the LLMs have not previously learned (Li, Song, and Qiu [2023](https://arxiv.org/html/2405.20770v3#bib.bib22)). Additionally, even with fine-tuning, training the LLMs will still consume a significant cost (Hu et al. [2022](https://arxiv.org/html/2405.20770v3#bib.bib18); Dettmers et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib11)). Adversarial purification (AP) aims to purify adversarial examples before feeding them into the target model, which has emerged as a promising defense method (Shi, Holtz, and Mishne [2021](https://arxiv.org/html/2405.20770v3#bib.bib44); Srinivasan et al. [2021](https://arxiv.org/html/2405.20770v3#bib.bib47); Lin et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib26)). Compared with the AT or AFT method, the AP method utilizes an additional model that can defend against unseen attacks without retraining the target model (Lin et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib26); Li, Song, and Qiu [2023](https://arxiv.org/html/2405.20770v3#bib.bib22)). In some traditional computer vision and natural language processing tasks, researchers have started using LLMs as purifiers for adversarial purification (Singh and Subramanyam [2024](https://arxiv.org/html/2405.20770v3#bib.bib45); Li et al. [2024a](https://arxiv.org/html/2405.20770v3#bib.bib23); Moraffah et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib35)), but the security issues of LLMs themselves have not been deeply considered. Therefore, we propose a novel LLM defense technique named LLAMOS to purify the adversarial textual examples before feeding them into the target LLM, aiming to improve the robustness.

Large Language Model Agent. The LLM agent is a new research direction that has emerged in recent years (Ha, Florence, and Song [2023](https://arxiv.org/html/2405.20770v3#bib.bib17); Mu et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib36); M. Bran et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib31)). This novel type of agent is capable of interacting with humans in natural language, leading to a significant increase in applications across fields such as chatbots, natural sciences, robotics, and workflows (Boiko, MacKnight, and Gomes [2023](https://arxiv.org/html/2405.20770v3#bib.bib6); Yang et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib64); Lin et al. [2024a](https://arxiv.org/html/2405.20770v3#bib.bib25); Wang et al. [2023b](https://arxiv.org/html/2405.20770v3#bib.bib58); Liu et al. [2023a](https://arxiv.org/html/2405.20770v3#bib.bib28)). Furthermore, LLMs have demonstrated promising zero-shot/few-shot planning and reasoning capabilities across various configurations (Sumers et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib48)), covering specific environments and reasoning tasks (Yao et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib66); Gong et al. [2023](https://arxiv.org/html/2405.20770v3#bib.bib15); Yao et al. [2024a](https://arxiv.org/html/2405.20770v3#bib.bib65)). In this paper, we introduce a new variant of the LLM agent designed specifically to purify adversarial textual examples generated by attacks.

## Experiments

In this section, we conduct extensive experiments on GLUE datasets to evaluate the effectiveness of the proposed method (LLAMOS). Specifically, our method significantly reduces the attack success rate (ASR) by up to 37.86% with GPT-3.5 and 45.59% with LLAMA-2, respectively.

### Experimental Setup

Datasets. The experiments are conducted on six tasks in GLUE datasets (Wang et al. [2018](https://arxiv.org/html/2405.20770v3#bib.bib54)), including SST-2, RTE, QQP, QNLI, MNLI-mm, MNLI-m (Socher et al. [2013](https://arxiv.org/html/2405.20770v3#bib.bib46); Dagan, Glickman, and Magnini [2005](https://arxiv.org/html/2405.20770v3#bib.bib9); Bar-Haim et al. [2006](https://arxiv.org/html/2405.20770v3#bib.bib2); Giampiccolo et al. [2007](https://arxiv.org/html/2405.20770v3#bib.bib14); Bos and Markert [2005](https://arxiv.org/html/2405.20770v3#bib.bib7); Bentivogli et al. [2009](https://arxiv.org/html/2405.20770v3#bib.bib4); Wang, Hamza, and Florian [2017](https://arxiv.org/html/2405.20770v3#bib.bib60); Rajpurkar et al. [2016](https://arxiv.org/html/2405.20770v3#bib.bib41); Williams, Nangia, and Bowman [2018](https://arxiv.org/html/2405.20770v3#bib.bib61)). The detailed descriptions are provided in Appendix A.

Adversarial Attacks. We evaluate our method against PromptAttack (Xu et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib63)), which is a powerful attack that combines nine different types of attacks, as illustrated in Table 11\. Furthermore, Xu et al. ([2024](https://arxiv.org/html/2405.20770v3#bib.bib63)) introduce the few-shot (FS) strategy (Logan IV et al. [2021](https://arxiv.org/html/2405.20770v3#bib.bib30)) and ensemble (EN) strategy (Croce and Hein [2020](https://arxiv.org/html/2405.20770v3#bib.bib8)) to boost the attack power of PromptAttack, details in Appendix B.

Evaluation Metrics. We evaluate the performance of defense methods using two metrics: attack success rate (ASR) and robust accuracy (RA). These metrics are derived from testing on adversarial examples, where a lower ASR or a higher RA indicates greater model robustness.

Training Details. The experiments in this paper are conducted using GPT-3.5 (OpenAI [2023](https://arxiv.org/html/2405.20770v3#bib.bib38)) with ‘GPT-3.5-Turbo-0613’ version and LLAMA-2 (Touvron et al. [2023b](https://arxiv.org/html/2405.20770v3#bib.bib53)) with ‘LLAMA-2-7b’ version. For GPT-3.5, we purchase OpenAI’s API service²²2https://openai.com/api/ and conduct testing experiments with the ‘openai’ package in Python. For LLAMA-2, we deploy it locally on NVIDIA RTX A6000 and utilize the available checkpoint published by MetaAI from HuggingFace³³3https://huggingface.co/meta-llama/.

Table 5: Standard accuracy and robust accuracy defense against PromptAttack with GPT-3.5\. In the first three lines, we present the average robust accuracy across different attacks to more clearly highlight the improvements of LLAMOS. The remaining lines show the robust accuracy of LLAMOS to defend against various attacks.

|         Methods | FS | SST-2 | RTE | QQP | QNLI | MNLI-mm | MNLI-m | Avg. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|         Standard | - | 97.66 | 80.47 | 75.78 | 66.41 | 66.41 | 71.87 | 76.43 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|         without | $\times$ | 42.97 | 52.93 | 47.66 | 39.65 | 37.50 | 40.23 | 43.49 |
|         LLAMOS | ✓ | 24.22 | 51.37 | 45.70 | 33.79 | 37.11 | 38.87 | 38.51 |
|         LLAMOS | $\times$ | 82.52 | 77.17 | 67.93 | 61.95 | 67.44 | 63.15 | 70.03 |
| ✓ | 72.38 | 76.99 | 66.72 | 60.68 | 65.27 | 62.62 | 67.44 |
|         C1 | $\times$ | 96.09 | 81.25 | 72.66 | 63.28 | 69.53 | 68.75 | 75.26 |
| ✓ | 96.88 | 81.25 | 66.41 | 64.84 | 68.75 | 66.41 | 74.09 |
|         C2 | $\times$ | 75.78 | 74.22 | 67.19 | 64.84 | 68.75 | 61.72 | 68.75 |
| ✓ | 83.59 | 77.34 | 63.28 | 60.16 | 67.97 | 58.59 | 68.49 |
|         C3 | $\times$ | 89.06 | 82.81 | 71.48 | 62.50 | 74.22 | 63.28 | 73.89 |
| ✓ | 66.41 | 81.25 | 73.44 | 64.84 | 70.31 | 69.53 | 70.96 |
|         W1 | $\times$ | 85.01 | 79.66 | 70.50 | 61.25 | 68.67 | 65.79 | 71.81 |
| ✓ | 80.18 | 77.93 | 69.92 | 59.19 | 64.63 | 63.40 | 69.21 |
|         W2 | $\times$ | 81.37 | 77.81 | 67.86 | 63.42 | 67.83 | 64.87 | 70.53 |
| ✓ | 80.04 | 75.77 | 66.15 | 61.18 | 64.28 | 63.31 | 68.46 |
|         W3 | $\times$ | 75.79 | 75.12 | 69.57 | 63.76 | 64.85 | 60.91 | 68.33 |
| ✓ | 64.48 | 75.00 | 68.17 | 61.69 | 63.00 | 60.51 | 65.47 |
|         S1 | $\times$ | 74.45 | 75.48 | 63.86 | 58.65 | 62.66 | 59.18 | 65.71 |
| ✓ | 71.18 | 72.76 | 64.01 | 56.89 | 61.79 | 58.18 | 64.14 |
|         S2 | $\times$ | 85.83 | 74.83 | 64.68 | 59.90 | 65.33 | 62.70 | 68.88 |
| ✓ | 58.77 | 76.53 | 64.52 | 59.01 | 63.39 | 61.08 | 63.88 |
|         S3 | $\times$ | 79.31 | 73.30 | 63.57 | 59.96 | 65.10 | 61.12 | 67.06 |
| ✓ | 49.86 | 75.06 | 64.59 | 58.31 | 63.25 | 62.61 | 62.28 |

### Results

Evaluation of LLAMOS Performance on Attack Success Rate (ASR). We evaluate the ASR of the LLAMOS against PromptAttack-EN and PromptAttack-FS-EN on the GLUE datasets with GPT-3.5 (OpenAI [2023](https://arxiv.org/html/2405.20770v3#bib.bib38)). As shown in [Table 2](https://arxiv.org/html/2405.20770v3#Sx3.T2 "In Adversarial System with Multiple LLMs ‣ Methods ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"), our method significantly reduces the ASR of both PromptAttack-EN and PromptAttack-FS-EN across all tasks. Specifically, our method achieves an average ASR reduction of 29.33% and 29.39%, respectively. These results demonstrate that LLAMOS is effective in defending against adversarial textual attacks. Additionally, we also evaluate the performance of LLAMOS on the SST-2 dataset with LLAMA-2 (Touvron et al. [2023b](https://arxiv.org/html/2405.20770v3#bib.bib53)), as shown in [Table 3](https://arxiv.org/html/2405.20770v3#Sx3.T3 "In Adversarial System with Multiple LLMs ‣ Methods ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"). The results are similar to the previous experiments. The ASR of PromptAttack-EN and PromptAttack-FS-EN is significantly reduced by 45.59% and 10.57%, respectively.

Evaluation of LLAMOS Performance on Robust Accuracy (RA). We evaluate the RA of LLAMOS on the SST-2 dataset with LLAMA-2 against three types of PromptAttack: character, word, and sentence attacks. In [Table 4](https://arxiv.org/html/2405.20770v3#Sx3.T4 "In Adversarial System with Multiple LLMs ‣ Methods ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"), the accuracies in the first two columns represent the standard accuracy and robust accuracy without defense, while the last three columns represent the robust accuracy with LLAMOS. Under strong attacks, the classification accuracy of the target LLM decreased from 92.18% to 30.42%. LLAMOS can effectively defend against adversarial textual attacks, significantly improving the robust accuracy. Specifically, the lowest robust accuracy reaches 86.96%. Additionally, we conduct comprehensive experiments across nine types of attacks and six tasks with GPT-3.5, as shown in [Table 5](https://arxiv.org/html/2405.20770v3#Sx5.T5 "In Experimental Setup ‣ Experiments ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"). LLAMOS can effectively defend against character-level attacks, achieving results on C1 and C3 that closely match the standard accuracy.

Table 6: Comparison on robust accuracy against C3 attack with in-context learning.

|        ICL | C3 | C3 with few-shot |
| --- | --- | --- |
|        $\times$ | 89.06 | 66.41 |
|        ✓ | 97.66 | 92.19 |

Table 7: Robust accuracy defense against the attack agent under different iterations with GPT-3.5.

|       Iterations | Round 1 | Round 2 | Round 3 | Round 4 | Round 5 |
| --- | --- | --- | --- | --- | --- |
|       Defense | 96.09 | 84.38 | 83.59 | 91.41 | 90.63 |
|       Attack | 56.25 | 43.53 | 45.93 | 50.34 | 28.13 |

Table 8: Comparison between clean examples, adversarial examples, and purified examples. When incorrectly classified, the modified area is highlighted with a yellow background.

|        Method | [Input] | Prediction |
|        Target LLM | Clean (Original): a fast , funny , highly enjoyable movie . | Positive |
|        After Attack | Adversarial: a fast , funny , highly enjoyable movie . :( | Negative |
|        LLAMOS | Purified: a fast , funny , highly enjoyable movie . :( | Negative |
|        LLAMOS+ICL | Purified: a fast , funny , highly enjoyable movie .. | Positive |
| The purified and adversarial examples generated by the agents in the adversarial system during iterations. |
|        Round 1 | Purified: You don’t have to know music to appreciate the film’s easygoing blend of tragedy and romance. | Positive |
| Adversarial: Youdo n’thave to know music to appreciate the film’s easygoing … | Negative |
|        Round 2 | Purified: You don’t have to know music to appreciate the film’s easygoing … | Positive |
| Adversarial: You don’t have to know music todepreciatethe film’s easygoing … | Negative |
|        Round 3+ | Purified: You don’t have to know music to appreciate the film’s easygoing … | Positive |
| Adversarial: You don’t have to know music todepreciatethe film’s easygoing … | Negative |

Evaluation of LLAMOS Performance with In-Context Learning (ICL). The C3-based attack (Xu et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib63)) is a very obvious attack that adds up to two extraneous characters to the end of the sentence, as shown in [Table 8](https://arxiv.org/html/2405.20770v3#Sx5.T8 "In Results ‣ Experiments ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"). However, our method only achieves robust accuracies of 89.06% for the C3 attack and 66.41% for the C3-FS attack, respectively. To further improve the robustness, we introduce ICL to enhance the performance of the defense agent. As shown in [Table 6](https://arxiv.org/html/2405.20770v3#Sx5.T6 "In Results ‣ Experiments ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"), the defense agent with ICL significantly improves the robust accuracy against the C3 attack, achieving a robust accuracies of 97.66% for the C3 attack and 92.19% for the C3-FS attack.

Analysis of Adversarial System. We conduct experiments with an adversarial system and evaluate the robust accuracy defense against adversarial examples generated by the attack agent over multiple iterations. As shown in [Table 7](https://arxiv.org/html/2405.20770v3#Sx5.T7 "In Results ‣ Experiments ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"), the defense agent initially achieves a robust accuracy of 96.09% in the first round of confrontation. However, after the purified examples are re-attacked by the attack agent, the robust accuracy decreases to 56.25%. The defense agent then purifies these adversarial examples again, leading to an increase in robust accuracy, but it will decrease once more by subsequent attacks. This continual fluctuation in robust accuracy is a common phenomenon in adversarial training (Goodfellow, Shlens, and Szegedy [2015](https://arxiv.org/html/2405.20770v3#bib.bib16)). Upon reviewing the generated texts, we observe that after several rounds of confrontation, both the defense agent and attack agent may generate the same sentences as previous ones, resulting in a potential infinite loop, as shown in [Table 8](https://arxiv.org/html/2405.20770v3#Sx5.T8 "In Results ‣ Experiments ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"). This is an interesting phenomenon that requires further investigation, particularly strategies to disrupt such loops.

### Discussion

The Advantages of LLAMOS. As emphasized by the experimental results presented in the Results Section and Table 12, LLAMOS significantly enhances performance across various tasks and attacks with LLAMA-2 and GPT-3.5\. Additionally, the defense agent in LLAMOS is a plug-and-play module, serving as a pre-processing step. Through in-context learning (ICL), the defense agent can be continuously optimized to defend against emerging attacks. This invisibly resolves a major challenge in adversarial robustness: Due to the significant differences between different attacks, the model trained on specific attacks often fails to generalize to other unseen attacks (Poursaeed et al. [2021](https://arxiv.org/html/2405.20770v3#bib.bib39); Laidlaw, Singla, and Feizi [2021](https://arxiv.org/html/2405.20770v3#bib.bib21); Tack et al. [2022](https://arxiv.org/html/2405.20770v3#bib.bib50)). The model necessitates to be continuously fine-tuning to adapt to emerging attacks. However, fine-tuning parameters requires substantial costs (Hu et al. [2022](https://arxiv.org/html/2405.20770v3#bib.bib18); Dettmers et al. [2024](https://arxiv.org/html/2405.20770v3#bib.bib11)), and the emergence of new attack techniques continually makes it impractical to train the model to defend against emerging attacks (Laidlaw, Singla, and Feizi [2021](https://arxiv.org/html/2405.20770v3#bib.bib21); Lin et al. [2024b](https://arxiv.org/html/2405.20770v3#bib.bib26)). In contrast, our method can effectively enhance the robustness through ICL without adjusting the parameters of the LLMs, which is undoubtedly a significant advantage.

The Challenges in LLM-based Defense. The defense agent is tasked with purifying adversarial examples, but it is difficult to distinguish between natural examples and adversarial examples in some cases. As shown in Table 12.4, the attacker altered the original meaning by inserting ‘not’, rendering the adversarial example indistinguishable from a natural example, resulting in the defense agent failing to generate the correct sentence. Although we hope that the defense agent can observe the sentences like humans, it presents a huge challenge at present. Unlike the attacker or humans, the defense agent lacks access to the original label of the input sentence. Furthermore, although the defense agent can effectively defend against adversarial attacks, it cannot prevent subsequent attacks, as illustrated in [Table 7](https://arxiv.org/html/2405.20770v3#Sx5.T7 "In Results ‣ Experiments ‣ Large Language Model Sentinel: LLM Agent for Adversarial Purification"). For instance, the malicious LLMs can embed specific system prompts to influence the output, which is unbeknownst to users; they can add ‘:)’ to each input sentence for prediction rather than predicting the original sentence. In this case, the defense agent also fails to defend. This issue is also an important problem in traditional adversarial training (Goodfellow, Shlens, and Szegedy [2015](https://arxiv.org/html/2405.20770v3#bib.bib16)), and no method has completely resolved this issue. Nonetheless, as previously discussed, LLMs offer advantages not available to traditional DNNs, and we have naturally solved one challenge in adversarial robustness, which is that the model can adapt to new attacks. Hence, future advancements may resolve adversarial issues of attack and defense within LLM frameworks, representing a challenging but promising research direction.

## Conclusion

In this paper, we propose LLAMOS, a novel LLM-based defense technique designed to purify adversarial examples before feeding them into the target LLM. The defense agent within LLAMOS operates as a plug-and-play module that functions effectively as a pre-processing step without requiring retraining of the target LLM. We conduct extensive experiments across various tasks and attacks with LLAMA-2 and GPT-3.5\. The results demonstrate that LLAMOS can effectively defend against adversarial attacks. Furthermore, we discuss certain existing shortcomings and challenges, which we aim to address in future research. Ultimately, we discuss the Limitations and Impact Statements in the Appendix D.

## References

*   Achiam et al. (2023) Achiam, J.; Adler, S.; Agarwal, S.; Ahmad, L.; Akkaya, I.; Aleman, F. L.; Almeida, D.; Altenschmidt, J.; Altman, S.; Anadkat, S.; et al. 2023. Gpt-4 technical report. *arXiv preprint arXiv:2303.08774*.
*   Bar-Haim et al. (2006) Bar-Haim, R.; Dagan, I.; Dolan, B.; Ferro, L.; Giampiccolo, D.; Magnini, B.; and Szpektor, I. 2006. The Second PASCAL Recognising Textual Entailment Challenge.
*   Bender et al. (2021) Bender, E. M.; Gebru, T.; McMillan-Major, A.; and Shmitchell, S. 2021. On the dangers of stochastic parrots: Can language models be too big? In *Proceedings of the 2021 ACM conference on fairness, accountability, and transparency*, 610–623.
*   Bentivogli et al. (2009) Bentivogli, L.; Clark, P.; Dagan, I.; and Giampiccolo, D. 2009. The Fifth PASCAL Recognizing Textual Entailment Challenge. *TAC*, 7(8): 1.
*   Bianchi et al. (2023) Bianchi, F.; Suzgun, M.; Attanasio, G.; Röttger, P.; Jurafsky, D.; Hashimoto, T.; and Zou, J. 2023. Safety-tuned llamas: Lessons from improving the safety of large language models that follow instructions. *arXiv preprint arXiv:2309.07875*.
*   Boiko, MacKnight, and Gomes (2023) Boiko, D. A.; MacKnight, R.; and Gomes, G. 2023. Emergent autonomous scientific research capabilities of large language models. *arXiv preprint arXiv:2304.05332*.
*   Bos and Markert (2005) Bos, J.; and Markert, K. 2005. Recognising textual entailment with logical inference. In *Proceedings of Human Language Technology Conference and Conference on Empirical Methods in Natural Language Processing*, 628–635.
*   Croce and Hein (2020) Croce, F.; and Hein, M. 2020. Reliable evaluation of adversarial robustness with an ensemble of diverse parameter-free attacks. In *International conference on machine learning*, 2206–2216\. PMLR.
*   Dagan, Glickman, and Magnini (2005) Dagan, I.; Glickman, O.; and Magnini, B. 2005. The pascal recognising textual entailment challenge. In *Machine learning challenges workshop*, 177–190\. Springer.
*   Deng et al. (2024) Deng, Y.; Zhang, W.; Pan, S. J.; and Bing, L. 2024. Multilingual Jailbreak Challenges in Large Language Models. In *The Twelfth International Conference on Learning Representations*.
*   Dettmers et al. (2024) Dettmers, T.; Pagnoni, A.; Holtzman, A.; and Zettlemoyer, L. 2024. Qlora: Efficient finetuning of quantized llms. *Advances in Neural Information Processing Systems*, 36.
*   Dong et al. (2022) Dong, Q.; Li, L.; Dai, D.; Zheng, C.; Wu, Z.; Chang, B.; Sun, X.; Xu, J.; and Sui, Z. 2022. A survey on in-context learning. *arXiv preprint arXiv:2301.00234*.
*   Gehman et al. (2020) Gehman, S.; Gururangan, S.; Sap, M.; Choi, Y.; and Smith, N. A. 2020. RealToxicityPrompts: Evaluating Neural Toxic Degeneration in Language Models. *Findings of the Association for Computational Linguistics: EMNLP 2020*.
*   Giampiccolo et al. (2007) Giampiccolo, D.; Magnini, B.; Dagan, I.; and Dolan, W. B. 2007. The third pascal recognizing textual entailment challenge. In *Proceedings of the ACL-PASCAL workshop on textual entailment and paraphrasing*, 1–9.
*   Gong et al. (2023) Gong, R.; Huang, Q.; Ma, X.; Vo, H.; Durante, Z.; Noda, Y.; Zheng, Z.; Zhu, S.-C.; Terzopoulos, D.; Fei-Fei, L.; et al. 2023. Mindagent: Emergent gaming interaction. *arXiv preprint arXiv:2309.09971*.
*   Goodfellow, Shlens, and Szegedy (2015) Goodfellow, I. J.; Shlens, J.; and Szegedy, C. 2015. Explaining and harnessing adversarial examples. *International Conference on Learning Representations*.
*   Ha, Florence, and Song (2023) Ha, H.; Florence, P.; and Song, S. 2023. Scaling up and distilling down: Language-guided robot skill acquisition. In *Conference on Robot Learning*, 3766–3777\. PMLR.
*   Hu et al. (2022) Hu, E. J.; yelong shen; Wallis, P.; Allen-Zhu, Z.; Li, Y.; Wang, S.; Wang, L.; and Chen, W. 2022. LoRA: Low-Rank Adaptation of Large Language Models. In *International Conference on Learning Representations*.
*   Kasneci et al. (2023) Kasneci, E.; Seßler, K.; Küchemann, S.; Bannert, M.; Dementieva, D.; Fischer, F.; Gasser, U.; Groh, G.; Günnemann, S.; Hüllermeier, E.; et al. 2023. ChatGPT for good? On opportunities and challenges of large language models for education. *Learning and individual differences*, 103: 102274.
*   Köpf et al. (2024) Köpf, A.; Kilcher, Y.; von Rütte, D.; Anagnostidis, S.; Tam, Z. R.; Stevens, K.; Barhoum, A.; Nguyen, D.; Stanley, O.; Nagyfi, R.; et al. 2024. Openassistant conversations-democratizing large language model alignment. *Advances in Neural Information Processing Systems*, 36.
*   Laidlaw, Singla, and Feizi (2021) Laidlaw, C.; Singla, S.; and Feizi, S. 2021. Perceptual Adversarial Robustness: Defense Against Unseen Threat Models. In *International Conference on Learning Representations (ICLR)*.
*   Li, Song, and Qiu (2023) Li, L.; Song, D.; and Qiu, X. 2023. Text Adversarial Purification as Defense against Adversarial Attacks. In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 338–350.
*   Li et al. (2024a) Li, T.; Liu, Q.; Pang, T.; Du, C.; Guo, Q.; Liu, Y.; and Lin, M. 2024a. Purifying Large Language Models by Ensembling a Small Language Model. *arXiv preprint arXiv:2402.14845*.
*   Li et al. (2024b) Li, Y.; Li, T.; Chen, K.; Zhang, J.; Liu, S.; Wang, W.; Zhang, T.; and Liu, Y. 2024b. BadEdit: Backdooring Large Language Models by Model Editing. In *The Twelfth International Conference on Learning Representations*.
*   Lin et al. (2024a) Lin, B. Y.; Fu, Y.; Yang, K.; Brahman, F.; Huang, S.; Bhagavatula, C.; Ammanabrolu, P.; Choi, Y.; and Ren, X. 2024a. Swiftsage: A generative agent with fast and slow thinking for complex interactive tasks. *Advances in Neural Information Processing Systems*, 36.
*   Lin et al. (2024b) Lin, G.; Li, C.; Zhang, J.; Tanaka, T.; and Zhao, Q. 2024b. Adversarial Training on Purification (AToP): Advancing Both Robustness and Generalization. In *The Twelfth International Conference on Learning Representations*.
*   Lin et al. (2024c) Lin, G.; Tao, Z.; Zhang, J.; Tanaka, T.; and Zhao, Q. 2024c. Robust Diffusion Models for Adversarial Purification. *arXiv preprint arXiv:2403.16067*.
*   Liu et al. (2023a) Liu, X.; Yu, H.; Zhang, H.; Xu, Y.; Lei, X.; Lai, H.; Gu, Y.; Ding, H.; Men, K.; Yang, K.; et al. 2023a. Agentbench: Evaluating llms as agents. *arXiv preprint arXiv:2308.03688*.
*   Liu et al. (2023b) Liu, Y.; Deng, G.; Xu, Z.; Li, Y.; Zheng, Y.; Zhang, Y.; Zhao, L.; Zhang, T.; and Liu, Y. 2023b. Jailbreaking chatgpt via prompt engineering: An empirical study. *arXiv preprint arXiv:2305.13860*.
*   Logan IV et al. (2021) Logan IV, R. L.; Balažević, I.; Wallace, E.; Petroni, F.; Singh, S.; and Riedel, S. 2021. Cutting down on prompts and parameters: Simple few-shot learning with language models. *arXiv preprint arXiv:2106.13353*.
*   M. Bran et al. (2024) M. Bran, A.; Cox, S.; Schilter, O.; Baldassari, C.; White, A. D.; and Schwaller, P. 2024. Augmenting large language models with chemistry tools. *Nature Machine Intelligence*, 1–11.
*   Manakul, Liusie, and Gales (2023) Manakul, P.; Liusie, A.; and Gales, M. 2023. SelfCheckGPT: Zero-Resource Black-Box Hallucination Detection for Generative Large Language Models. In *The 2023 Conference on Empirical Methods in Natural Language Processing*.
*   Mckenna et al. (2023) Mckenna, N.; Li, T.; Cheng, L.; Hosseini, M.; Johnson, M.; and Steedman, M. 2023. Sources of Hallucination by Large Language Models on Inference Tasks. In *Findings of the Association for Computational Linguistics: EMNLP 2023*, 2758–2774.
*   Minaee et al. (2024) Minaee, S.; Mikolov, T.; Nikzad, N.; Chenaghlu, M.; Socher, R.; Amatriain, X.; and Gao, J. 2024. Large language models: A survey. *arXiv preprint arXiv:2402.06196*.
*   Moraffah et al. (2024) Moraffah, R.; Khandelwal, S.; Bhattacharjee, A.; and Liu, H. 2024. Adversarial Text Purification: A Large Language Model Approach for Defense. In *Pacific-Asia Conference on Knowledge Discovery and Data Mining*, 65–77\. Springer.
*   Mu et al. (2024) Mu, Y.; Zhang, Q.; Hu, M.; Wang, W.; Ding, M.; Jin, J.; Wang, B.; Dai, J.; Qiao, Y.; and Luo, P. 2024. Embodiedgpt: Vision-language pre-training via embodied chain of thought. *Advances in Neural Information Processing Systems*, 36.
*   OpenAI (2022) OpenAI. 2022. Introducing chatgpt.
*   OpenAI (2023) OpenAI. 2023. GPT-4V(ision) System Card.
*   Poursaeed et al. (2021) Poursaeed, O.; Jiang, T.; Yang, H.; Belongie, S.; and Lim, S.-N. 2021. Robustness and generalization via generative adversarial training. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*, 15711–15720.
*   Qi et al. (2024) Qi, X.; Zeng, Y.; Xie, T.; Chen, P.-Y.; Jia, R.; Mittal, P.; and Henderson, P. 2024. Fine-tuning Aligned Language Models Compromises Safety, Even When Users Do Not Intend To! In *The Twelfth International Conference on Learning Representations*.
*   Rajpurkar et al. (2016) Rajpurkar, P.; Zhang, J.; Lopyrev, K.; and Liang, P. 2016. Squad: 100,000+ questions for machine comprehension of text. *arXiv preprint arXiv:1606.05250*.
*   Romera-Paredes et al. (2024) Romera-Paredes, B.; Barekatain, M.; Novikov, A.; Balog, M.; Kumar, M. P.; Dupont, E.; Ruiz, F. J.; Ellenberg, J. S.; Wang, P.; Fawzi, O.; et al. 2024. Mathematical discoveries from program search with large language models. *Nature*, 625(7995): 468–475.
*   Shen et al. (2023) Shen, T.; Jin, R.; Huang, Y.; Liu, C.; Dong, W.; Guo, Z.; Wu, X.; Liu, Y.; and Xiong, D. 2023. Large language model alignment: A survey. *arXiv preprint arXiv:2309.15025*.
*   Shi, Holtz, and Mishne (2021) Shi, C.; Holtz, C.; and Mishne, G. 2021. Online adversarial purification based on self-supervision. *International Conference on Learning Representations*.
*   Singh and Subramanyam (2024) Singh, H.; and Subramanyam, A. 2024. Language Guided Adversarial Purification. In *ICASSP 2024-2024 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP)*, 7685–7689\. IEEE.
*   Socher et al. (2013) Socher, R.; Perelygin, A.; Wu, J.; Chuang, J.; Manning, C. D.; Ng, A. Y.; and Potts, C. 2013. Recursive deep models for semantic compositionality over a sentiment treebank. In *Proceedings of the 2013 conference on empirical methods in natural language processing*, 1631–1642.
*   Srinivasan et al. (2021) Srinivasan, V.; Rohrer, C.; Marban, A.; Müller, K.-R.; Samek, W.; and Nakajima, S. 2021. Robustifying models against adversarial attacks by langevin dynamics. *Neural Networks*, 137: 1–17.
*   Sumers et al. (2023) Sumers, T. R.; Yao, S.; Narasimhan, K.; and Griffiths, T. L. 2023. Cognitive architectures for language agents. *arXiv preprint arXiv:2309.02427*.
*   Szegedy et al. (2014) Szegedy, C.; Zaremba, W.; Sutskever, I.; Bruna, J.; Erhan, D.; Goodfellow, I.; and Fergus, R. 2014. Intriguing properties of neural networks. *International Conference on Learning Representations*.
*   Tack et al. (2022) Tack, J.; Yu, S.; Jeong, J.; Kim, M.; Hwang, S. J.; and Shin, J. 2022. Consistency regularization for adversarial robustness. In *Proceedings of the AAAI Conference on Artificial Intelligence*, 8414–8422.
*   Thirunavukarasu et al. (2023) Thirunavukarasu, A. J.; Ting, D. S. J.; Elangovan, K.; Gutierrez, L.; Tan, T. F.; and Ting, D. S. W. 2023. Large language models in medicine. *Nature medicine*, 29(8): 1930–1940.
*   Touvron et al. (2023a) Touvron, H.; Lavril, T.; Izacard, G.; Martinet, X.; Lachaux, M.-A.; Lacroix, T.; Rozière, B.; Goyal, N.; Hambro, E.; Azhar, F.; et al. 2023a. Llama: Open and efficient foundation language models. *arXiv preprint arXiv:2302.13971*.
*   Touvron et al. (2023b) Touvron, H.; Martin, L.; Stone, K.; Albert, P.; Almahairi, A.; Babaei, Y.; Bashlykov, N.; Batra, S.; Bhargava, P.; Bhosale, S.; et al. 2023b. Llama 2: Open foundation and fine-tuned chat models. *arXiv preprint arXiv:2307.09288*.
*   Wang et al. (2018) Wang, A.; Singh, A.; Michael, J.; Hill, F.; Levy, O.; and Bowman, S. R. 2018. GLUE: A multi-task benchmark and analysis platform for natural language understanding. *arXiv preprint arXiv:1804.07461*.
*   Wang et al. (2023a) Wang, B.; Chen, W.; Pei, H.; Xie, C.; Kang, M.; Zhang, C.; Xu, C.; Xiong, Z.; Dutta, R.; Schaeffer, R.; et al. 2023a. DecodingTrust: A Comprehensive Assessment of Trustworthiness in GPT Models. In *Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track*.
*   Wang et al. (2024) Wang, B.; Chen, W.; Pei, H.; Xie, C.; Kang, M.; Zhang, C.; Xu, C.; Xiong, Z.; Dutta, R.; Schaeffer, R.; et al. 2024. DecodingTrust: A Comprehensive Assessment of Trustworthiness in GPT Models. *Advances in Neural Information Processing Systems*, 36.
*   Wang et al. (2021) Wang, B.; Xu, C.; Wang, S.; Gan, Z.; Cheng, Y.; Gao, J.; Awadallah, A. H.; and Li, B. 2021. Adversarial GLUE: A Multi-Task Benchmark for Robustness Evaluation of Language Models. In *Thirty-fifth Conference on Neural Information Processing Systems Datasets and Benchmarks Track (Round 2)*.
*   Wang et al. (2023b) Wang, S.; Liu, C.; Zheng, Z.; Qi, S.; Chen, S.; Yang, Q.; Zhao, A.; Wang, C.; Song, S.; and Huang, G. 2023b. Avalon’s Game of Thoughts: Battle Against Deception through Recursive Contemplation. *arXiv preprint arXiv:2310.01320*.
*   Wang et al. (2023c) Wang, Y.; Zhong, W.; Li, L.; Mi, F.; Zeng, X.; Huang, W.; Shang, L.; Jiang, X.; and Liu, Q. 2023c. Aligning large language models with human: A survey. *arXiv preprint arXiv:2307.12966*.
*   Wang, Hamza, and Florian (2017) Wang, Z.; Hamza, W.; and Florian, R. 2017. Bilateral multi-perspective matching for natural language sentences. *arXiv preprint arXiv:1702.03814*.
*   Williams, Nangia, and Bowman (2018) Williams, A.; Nangia, N.; and Bowman, S. R. 2018. The multi-genre nli corpus.
*   Xiang et al. (2024) Xiang, Z.; Jiang, F.; Xiong, Z.; Ramasubramanian, B.; Poovendran, R.; and Li, B. 2024. BadChain: Backdoor Chain-of-Thought Prompting for Large Language Models. In *The Twelfth International Conference on Learning Representations*.
*   Xu et al. (2024) Xu, X.; Kong, K.; Liu, N.; Cui, L.; Wang, D.; Zhang, J.; and Kankanhalli, M. 2024. An LLM can Fool Itself: A Prompt-Based Adversarial Attack. In *The Twelfth International Conference on Learning Representations*.
*   Yang et al. (2023) Yang, Z.; Li, L.; Wang, J.; Lin, K.; Azarnasab, E.; Ahmed, F.; Liu, Z.; Liu, C.; Zeng, M.; and Wang, L. 2023. Mm-react: Prompting chatgpt for multimodal reasoning and action. *arXiv preprint arXiv:2303.11381*.
*   Yao et al. (2024a) Yao, S.; Yu, D.; Zhao, J.; Shafran, I.; Griffiths, T.; Cao, Y.; and Narasimhan, K. 2024a. Tree of thoughts: Deliberate problem solving with large language models. *Advances in Neural Information Processing Systems*, 36.
*   Yao et al. (2023) Yao, S.; Zhao, J.; Yu, D.; Du, N.; Shafran, I.; Narasimhan, K. R.; and Cao, Y. 2023. ReAct: Synergizing Reasoning and Acting in Language Models. In *The Eleventh International Conference on Learning Representations*.
*   Yao et al. (2024b) Yao, Y.; Duan, J.; Xu, K.; Cai, Y.; Sun, Z.; and Zhang, Y. 2024b. A survey on large language model (llm) security and privacy: The good, the bad, and the ugly. *High-Confidence Computing*, 100211.
*   Zhu et al. (2023) Zhu, K.; Wang, J.; Zhou, J.; Wang, Z.; Chen, H.; Wang, Y.; Yang, L.; Ye, W.; Gong, N. Z.; Zhang, Y.; et al. 2023. Promptbench: Towards evaluating the robustness of large language models on adversarial prompts. *arXiv preprint arXiv:2306.04528*.

See pages - of [figures/appendix.pdf](figures/appendix.pdf)