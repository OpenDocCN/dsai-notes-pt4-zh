<!--yml
category: 未分类
date: 2025-01-11 12:31:26
-->

# AgentReview: Exploring Peer Review Dynamics with LLM Agents

> 来源：[https://arxiv.org/html/2406.12708/](https://arxiv.org/html/2406.12708/)

Yiqiao Jin^(1∗), Qinlin Zhao^(2∗), Yiyang Wang¹, Hao Chen³, Kaijie Zhu⁴, Yijia Xiao⁵, Jindong Wang⁶
¹Georgia Institute of Technology, ²University of Science and Technology of China,
³Carnegie Mellon University, ⁴University of California, Santa Barbara,
⁵University of California, Los Angeles, ⁶William & Mary
¹{yjin328,ywang3420}@gatech.edu
²ac99@mail.ustc.edu.cn
³haoc3@andrew.cmu.edu
⁴kaijiezhu@ucsb.edu
⁵yijia.xiao@cs.ucla.edu
⁶jwang80@wm.edu

[https://agentreview.github.io/](https://agentreview.github.io/)

###### Abstract

Peer review is fundamental to the integrity and advancement of scientific publication. Traditional methods of peer review analyses often rely on exploration and statistics of existing peer review data, which do not adequately address the multivariate nature of the process, account for the latent variables, and are further constrained by privacy concerns due to the sensitive nature of the data. We introduce AgentReview, the first large language model (LLM) based peer review simulation framework, which effectively disentangles the impacts of multiple latent factors and addresses the privacy issue. Our study reveals significant insights, including a notable 37.1% variation in paper decisions due to reviewers’ biases, supported by sociological theories such as the social influence theory, altruism fatigue, and authority bias. We believe that this study could offer valuable insights to improve the design of peer review mechanisms. Our code is available at [https://github.com/Ahren09/AgentReview](https://github.com/Ahren09/AgentReview)

^†^†^∗ Both authors contributed equally.

### 1 Introduction

Peer review is a cornerstone for academic publishing, ensuring that accepted manuscripts meet the novelty, accuracy, and significance standards. Despite its importance, peer reviews often face several challenges, such as biases [[1](https://arxiv.org/html/2406.12708v2#bib.bib1)], variable review quality [[1](https://arxiv.org/html/2406.12708v2#bib.bib1)], unclear reviewer motives [[2](https://arxiv.org/html/2406.12708v2#bib.bib2)], and imperfect review mechanism [[3](https://arxiv.org/html/2406.12708v2#bib.bib3)], exacerbated by the ever-growing number of submissions. The rise of open science and preprint platforms has further complicated these systems, which may disclose author identities under double-blind policies [[4](https://arxiv.org/html/2406.12708v2#bib.bib4)].

Efforts to mitigate these problems have focused on enhancing fairness [[2](https://arxiv.org/html/2406.12708v2#bib.bib2)], reducing biases among novice reviewers [[1](https://arxiv.org/html/2406.12708v2#bib.bib1)], calibrating noisy peer review ratings [[5](https://arxiv.org/html/2406.12708v2#bib.bib5)], and refining mechanisms for paper assignment and reviewer expertise matching [[6](https://arxiv.org/html/2406.12708v2#bib.bib6), [7](https://arxiv.org/html/2406.12708v2#bib.bib7)]. However, several challenges persist in systematically exploring factors influencing peer review outcomes: 1) *Multivariate Nature.* The peer review process is affected by a variety of factors, ranging from reviewer expertise, area chair involvement, to the review mechanism design. This complexity makes it difficult to isolate specific factors that impact the review quality and outcomes; 2) *Latent Variables.* Factors such as reviewer biases and intentions are difficult to measure but have significant effects on the review process, often leading to less predictable outcomes; 3) *Privacy Concerns.* Peer review data are inherently sensitive and carry the risk of revealing reviewer identities. Investigation of such data not only poses ethical concerns but also deters future reviewer participation.

![Refer to caption](img/2046f74620914415c2a525b689b3c2fe.png)

Figure 1: AgentReview is an open and flexible framework designed to realistically simulate the peer review process. It enables controlled experiments to *disentangle* multiple variables in peer review, allowing for an in-depth examination of their effects on review outcomes. Our findings align with established sociological theories.

This Work. We introduce AgentReview, the first framework that integrates large language models (LLMs) [[8](https://arxiv.org/html/2406.12708v2#bib.bib8), [9](https://arxiv.org/html/2406.12708v2#bib.bib9)] with agent-based modeling [[10](https://arxiv.org/html/2406.12708v2#bib.bib10)] to simulate the peer review process (Sec. [2](https://arxiv.org/html/2406.12708v2#S2 "2 The AgentReview Framework ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). As shown in Figure [1](https://arxiv.org/html/2406.12708v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), AgentReview is built upon the capabilities of LLMs to perform realistic simulations of societal environments [[11](https://arxiv.org/html/2406.12708v2#bib.bib11), [12](https://arxiv.org/html/2406.12708v2#bib.bib12), [13](https://arxiv.org/html/2406.12708v2#bib.bib13), [14](https://arxiv.org/html/2406.12708v2#bib.bib14), [15](https://arxiv.org/html/2406.12708v2#bib.bib15), [16](https://arxiv.org/html/2406.12708v2#bib.bib16)] and provide high-quality feedback on academic literature comparable to or exceeds human levels [[17](https://arxiv.org/html/2406.12708v2#bib.bib17), [18](https://arxiv.org/html/2406.12708v2#bib.bib18), [19](https://arxiv.org/html/2406.12708v2#bib.bib19), [20](https://arxiv.org/html/2406.12708v2#bib.bib20), [21](https://arxiv.org/html/2406.12708v2#bib.bib21)].

AgentReview is open and flexible, designed to capture the *multivariate nature* of the peer review process. It features a range of customizable variables, such as characteristics of reviewers, authors, area chairs (ACs), as well as the reviewing mechanisms (Sec. [2.1](https://arxiv.org/html/2406.12708v2#S2.SS1 "2.1 Framework Overview ‣ 2 The AgentReview Framework ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). This adaptability allows for the systematic exploration and *disentanglement* of the distinct roles and influences of the various parties involved in the peer review process. Moreover, AgentReview supports the exploration of alternative reviewer characteristics and more complex review processes. By simulating peer review activities with over 53,800 generated peer review documents, including over 10,000 reviews, on over 500 submissions across four years of ICLR, AgentReview achieves statistically significant insights without needing real-world reviewer data, thereby maintaining reviewer *privacy*. AgentReview also supports the extension to alternative reviewer characteristics and more complicated reviewing processes. We conduct both content-level and numerical analyses after running large-scale simulations of the peer review process.

Key findings. Our findings are as follows, which could inspire future design of peer review systems:

*   •

    Social Influence [[22](https://arxiv.org/html/2406.12708v2#bib.bib22)]. Reviewers often adjust their ratings after rebuttals to align with their peers, driven by the pressure to conform to the perceived majority opinion. This conformity results in a 27.2% decrease in the standard deviation of ratings (Sec. [3.1.1](https://arxiv.org/html/2406.12708v2#S3.SS1.SSS1 "3.1.1 Overview ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"));

*   •

    Altruism Fatigue and Peer Effects  [[23](https://arxiv.org/html/2406.12708v2#bib.bib23)]. Even *one* under-committed reviewer can lead to a pronounced decline of commitment (18.7%) among all reviewers (Sec. [3.1.2](https://arxiv.org/html/2406.12708v2#S3.SS1.SSS2 "3.1.2 Reviewer Commitment ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"));

*   •

    Groupthink and Echo Chamber Effects [[24](https://arxiv.org/html/2406.12708v2#bib.bib24), [25](https://arxiv.org/html/2406.12708v2#bib.bib25)]. Biased reviewers tend to amplify each other’s negative opinions through interactions (Sec. [3.1.3](https://arxiv.org/html/2406.12708v2#S3.SS1.SSS3 "3.1.3 Reviewer Intention ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). This can lead to a 0.17 drop in ratings among biased reviewers and cause a *spillover effect*, influencing the judgments of unbiased reviewers and leading to a 0.25 decrease in ratings;

*   •

    Authority Bias and Halo Effects [[26](https://arxiv.org/html/2406.12708v2#bib.bib26)]. Reviewers tend to perceive manuscripts from renowned authors as more accurate. When all reviewers know the author identities for only 10% of the papers, decisions can change by a significant 27.7% (Sec. [3.3](https://arxiv.org/html/2406.12708v2#S3.SS3 "3.3 Impacts of Author Anonymity ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"));

*   •

    Anchoring Bias [[27](https://arxiv.org/html/2406.12708v2#bib.bib27)]. The rebuttal phase, despite its role in addressing reviewers’ concerns, exerts a less significant effect on final outcomes than other factors. This is potentially due to anchoring bias in which reviewers rely heavily on initial impressions of the submission.

Contributions. Our contributions are three-fold:

*   •

    *Versatile framework.* AgentReview is the first framework to employ LLM agents to simulate the entire peer review process;

*   •

    *Comprehensive Dataset.* We curated a large-scale dataset through our simulation, encompassing more than 53,800 generated reviews, rebuttals, updated reviews, meta-reviews, and final decisions, which can support future research on analyzing the academic peer review process;

*   •

    *Novel Insights.* Our study uncovers several significant findings that align with sociological theories to support future research;

![Refer to caption](img/0ab971509fb5ed155ed79c003d995726.png)

Figure 2: Our paper review pipeline consists of 5 phases. Solid black arrows $\rightarrow$ represent authorship connections, while blue dashed arrow $\rightarrow$ indicate visibility relations.

### 2 The AgentReview Framework

#### 2.1 Framework Overview

AgentReview is designed as an extensible testbed to study the impact of various stakeholders and mechanism designs on peer review results. It follows procedures of popular Natural Language Processing (NLP) and Machine Learning (ML) conferences, where reviewers provide initial paper reviews, update their reviews based on authors’ feedback, and area chairs (ACs) organize discussions among reviewers and make final decisions.¹¹1Some conferences or journals may follow slightly different review processes. AgentReview integrates three roles—reviewers, authors, and ACs—all powered by LLM agents.

Reviewers play a pivotal role in peer review. We identify three key dimensions that determine the quality of their reviews. 1) *Commitment* refers to the reviewer’s dedication and sense of responsibility in engaging with the manuscript. This involves a proactive and careful approach to provide thorough and constructive feedback on submissions. 2) *Intention* describes the motivation behind the reviews, focusing on whether the reviewer aims to genuinely help authors improve their papers or is influenced by biases or conflict of interests. 3) *Knowledgeability* measures the reviewer’s expertise in the manuscript’s subject area. Understanding the effects of each individual aspect is crucial for improving the peer review process.

To explore these dimensionalities, we assign reviewers into specific categories: knowledgeable versus unknowledgeable reviewers for *knowledgeability*, responsible versus irresponsible for *commitment*, and benign versus malicious for *intention*. These categorizations are set by prompts and fed into our system as fixed characteristics. For example, knowledgeable reviewers are described as reviewers that are adept at identifying the significance of the research and pinpointing any technical issues that require attention. In contrast, unknowledgeable reviewers lack expertise and may overlook critical flaws or misinterpret the contributions. Reviewer descriptions and prompts are detailed in Appendix Figure [10](https://arxiv.org/html/2406.12708v2#A1.F10 "Figure 10 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents").

Authors submit papers to the conference and provide rebuttals to the initial reviews during the Reviewer-AC discussion period (Phase 2 in Figure [1](https://arxiv.org/html/2406.12708v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). Although double-blind review policies are typically in place, authors may still opt to release preprints or publicize their works on social media, potentially revealing their identities. We consider two scenarios: 1) reviewers are aware of the authors’ identities due to the public release of their works, and 2) author identities remain unknown to the reviewers. This allows us to explore the implications of anonymity on the review process.

Area Chairs (ACs) have multiple duties, ranging from facilitating reviewer discussions, synthesizing feedback into meta-reviews, and making final decisions. ACs ensure the integrity of the review outcomes by maintaining constructive dialogues, integrating diverse viewpoints, and assessing papers for quality, originality, and relevance. Our work identifies three styles of ACs based on their involvement strategies, each influencing the review process differently: 1) *authoritarian* ACs dominate the decision-making, prioritizing their own evaluations over the collective input from reviewers; 2) *conformist* ACs rely heavily on other reviewers’ evaluations, minimizing the influence of their own expertise; 3) *inclusive* ACs consider all available discussion and feedback, including reviews, author rebuttals, and reviewer comments, along with their expertise, to make well-rounded final decisions.

#### 2.2 Review Process Design

AgentReview uses a structured, 5-phase pipeline (Figure [1](https://arxiv.org/html/2406.12708v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")) to simulate the peer review process.

I. Reviewer Assessment. In this phase, three reviewers critically evaluate the manuscript. To simulate an unbiased review process, each reviewer has access only to the manuscript and their own assessment, preventing any cross-influence among reviewers. Following [[18](https://arxiv.org/html/2406.12708v2#bib.bib18)], we ask LLM agents to generate reviews that comprise four sections, including *significance and novelty*, *potential reasons for acceptance*, *potential reasons for rejection*, and *suggestions for improvement*. This format is aligned with the conventional review structures of major ML/NLP conferences. Unless specified otherwise, each reviewer provides a numerical rating from 1 to 10 for each paper.

II. Author-Reviewer Discussion. Authors respond to each review with a rebuttal document to address misunderstandings, justify their methodologies, and acknowledge valid critiques.

III. Reviewer-AC Discussion. The AC initiates a discussion among the reviewers, asking them to reconsider their initial ratings, and provide an updated review after considering the rebuttals.

IV. Meta-Review Compilation. The AC integrates insights from Phase I-III discussions, their own observations, and numeric ratings into a meta-review. This document provides a synthesized assessment of the manuscript’s strengths and weaknesses that guides the final decision.

V. Paper Decision. In the final phase, the AC reviews all meta-reviews for their assigned papers to make an informed decision regarding their acceptance or rejection. We adopt a fixed acceptance rate of 32%, reflecting the actual average acceptance rate for ICLR $2020\sim 2023$. Therefore, each AC is tasked with making decisions for a batch of 10 papers and accepts $3\sim 4$ papers in the batch.

#### 2.3 Data Selection

The paper data for AgentReview is sourced from real conference submissions to ensure that our simulated reviews closely mirror real scenarios. We adhere to four criteria for data selection: 1) The conference must have international impact with a large number of authors and a wide audience, and the academic achievements discussed should have significant real-world impacts; 2) the papers must be publicly available; 3) the quality of the papers must reflect real-world distribution, including both accepted and rejected papers; 4) the papers must span a broad time range to cover a variety of topics and mitigate the effects of evolving reviewer preferences over time.

We select ICLR due to its status as a leading publication venue in computer science and its transparency in making both accepted and rejected submissions available. We retrieve papers spanning four years (2020$\sim$2023) using OpenReview API²²2[https://github.com/openreview/openreview-py](https://github.com/openreview/openreview-py). Papers are categorized into oral (top 5%), spotlight (top 25%), poster, and rejection. We then employ a stratified sampling technique to select papers from each category, resulting in a diverse dataset with 350 rejected papers, 125 posters, 29 spotlights, and 19 orals. This approach ensures the inclusion of papers with varying quality, closely mirroring real-world conferences. Finally, we extract the title, abstract, figure and table captions, and the main text that serve as the inputs for the LLM agents.

#### 2.4 Baseline Setting

Real peer review process inherently entails substantial uncertainty due to variations in reviewers’ expertise, commitment, and intentions, often leading to seemingly inconsistent numeric ratings. For example, NeurIPS experiments found significant differences in reviewers’ ratings when different sets of reviewers evaluated the same submissions [[28](https://arxiv.org/html/2406.12708v2#bib.bib28), [2](https://arxiv.org/html/2406.12708v2#bib.bib2)]. Directly comparing numeric ratings of our experimental outcomes with actual ratings can be inappropriate and fail to *disentangle* the latent variables.

To address this, we establish a *baseline* setting with no specific characteristics of LLM agents (referred to as ‘*baseline*’ in Table [1](https://arxiv.org/html/2406.12708v2#S3.T1 "Table 1 ‣ 3.1.1 Overview ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). This allows us to measure the impact of changes in one variable against a consistent reference. Across all settings, we generate 10,460 reviews and rebuttals, 23,535 reviewer-AC discussions, 9,414 meta-reviews, and 9,414 paper decisions. Detailed statistics for the dataset are in Appendix Table [4](https://arxiv.org/html/2406.12708v2#A1.T4 "Table 4 ‣ A.1 Review Categorization ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), and the experimental cost is in Appendix [A.2](https://arxiv.org/html/2406.12708v2#A1.SS2 "A.2 Experimental Costs ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")).

### 3 Results

#### 3.1 The Role of Reviewers

![Refer to caption](img/a7f661d4c64a9d0d69ee0f23cfa99171.png)![Refer to caption](img/f35b9d52dd9dec4d31696bcd0350844e.png)

Figure 3: Distribution of initial and final scores with respect to varying number of irresponsible ![Refer to caption](img/dc03d46ebee6f6ef2e9e78c5843fefeb.png) (left) & malicious ![Refer to caption](img/8902eccb27842c92cb3ed18bc899c965.png) (right) reviewers.

To study the effect of commitment on the peer review outcomes, we start with replacing a *normal* reviewer with either a responsible or an irresponsible reviewer, then gradually increase the number of reviews. The settings we consider as well as the initial & final ratings are in Table [1](https://arxiv.org/html/2406.12708v2#S3.T1 "Table 1 ‣ 3.1.1 Overview ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), and the rating distribution is in Figure [9](https://arxiv.org/html/2406.12708v2#A1.F9 "Figure 9 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"). Agent-based reviewers in our environment demonstrate classic phenomena in sociology, such as social influence, echo chamber, and halo effects.

##### 3.1.1 Overview

|  | Initial (Phase I) | Final (Phase III) |
| Setting | Avg. | Std. | Avg. | Std. |
| ![[Uncaptioned image]](img/1f2e638e85880e5cb699145f5272c6f9.png)  *baseline* | 5.053 | 0.224 | 5.110 | 0.163 |
| ![[Uncaptioned image]](img/1c6b2f01c50475c915d3b0eb2f88cba6.png) responsible | 4.991 | 0.276 | 5.032 | 0.150 |
| ![[Uncaptioned image]](img/35ec100d50b3fa63055fea033918aba9.png) irresponsible | 4.750 | 0.645 | 4.815 | 0.434 |
| ![[Uncaptioned image]](img/6ce892ecb30cc9589e07f1adb7e6bc96.png) benign | 4.990 | 0.281 | 5.098 | 0.211 |
| ![[Uncaptioned image]](img/dbee2350912b4af6acf07ea4068ac86c.png) malicious | 4.421 | 1.181 | 4.368 | 1.014 |
| ![[Uncaptioned image]](img/c23c9aeaa9a09739f0836b7847c78d09.png) knowledgeable | 5.004 | 0.260 | 5.052 | 0.152 |
| ![[Uncaptioned image]](img/363485acc92aaca4b5ec35680c2a2410.png) unknowledgeable | 4.849 | 0.479 | 4.987 | 0.220 |

Table 1: Summary of results. We report the reviewer scores before & after Reviewer-Author Discussion (Phase III in Figure [2](https://arxiv.org/html/2406.12708v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). ‘Initial’ & ‘Final’ indicate the reviewer ratings in Phase I & III, respectively.

Social Influence Theory [[29](https://arxiv.org/html/2406.12708v2#bib.bib29)] suggests that individuals in a group tend to revise their beliefs towards a common viewpoint. A similar tendency towards convergence is also observed among the reviewers. Across all settings, the standard deviation of reviewer ratings (Table [1](https://arxiv.org/html/2406.12708v2#S3.T1 "Table 1 ‣ 3.1.1 Overview ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")) significant declines after the Reviewer-AC discussion, revealing a trend towards *conformity*. This is particularly evident when a highly knowledgeable or responsible reviewer dominates the discussion.

Overall, responsible, knowledgeable, and benign (well-intentioned) reviewers generally give higher scores than less committed or biased (malicious) reviewers. Although initial review ratings can be low, the final ratings in most settings significantly improve following discussions, highlighting the importance of reviewer-author interactions on addressing reviewers’ concerns. In Sec. [3.4](https://arxiv.org/html/2406.12708v2#S3.SS4 "3.4 Effects of Peer Review Mechanisms ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), we further explore whether these interactions and subsequent paper improvements influence the final decisions.

##### 3.1.2 Reviewer Commitment

Altruism Fatigue & Peer Effect [[23](https://arxiv.org/html/2406.12708v2#bib.bib23)] Paper review is typically unpaid and time-consuming [[30](https://arxiv.org/html/2406.12708v2#bib.bib30)], requiring substantial time investment beyond reviewers’ regular professional duties. This demanding nature, coupled with *altruism fatigue*—where reviewers feel their voluntary efforts are unrecognized—often results in reduced commitment and superficial assessments.

The presence of just one irresponsible reviewer can lead to a pronounced decline in overall reviewer commitment compared with the *baseline*. Although the initial review length is similar between the two settings (*baseline* and *irresponsible*), averaging 432.4 and 429.2 words, the average word count experienced a significant 18.7% drop, from 195.5 to 159.0 words, after reviewers interact during the reviewer-AC discussion. This *peer effect* illustrates how one reviewer’s subpar performance can lower the standards and efforts of others, leading to more cursory review post-rebuttal. The reduction in overall engagement during critical review discussions underscores the negative impact of insufficient reviewer commitment, which can permit the publication of potentially flawed research, misleading subsequent studies and eroding trust in the academic review process.

| Var. | Setting | Jacc. | $\kappa$ | %Agree |
| --- | --- | --- | --- | --- |
| ![[Uncaptioned image]](img/c6588f426225d44562004c404a0a6eb9.png) | ![[Uncaptioned image]](img/2b4241453928674fea9284b52d50d53f.png) responsible | 0.372 | 0.349 | 72.85 |
| ![[Uncaptioned image]](img/8ea67be989fd47e898ac7b7d0b03c82c.png) irresponsible | 0.314 | 0.257 | 69.02 |
| ![[Uncaptioned image]](img/c4523d57555fa3723dfd5880c2d93166.png) benign | 0.632 | 0.679 | 86.62 |
| ![[Uncaptioned image]](img/73169b8827ff9db42fd563a3c320f904.png) malicious | 0.230 | 0.111 | 62.91 |
| ![[Uncaptioned image]](img/ca08d3de7982eab4ec3286b0f6e348eb.png) knowledgeable | 0.297 | 0.230 | 67.88 |
| ![[Uncaptioned image]](img/0f0ff677e479718b9b82f30af33c2545.png) unknowledgeable | 0.325 | 0.276 | 69.79 |
| ![[Uncaptioned image]](img/a627b5fb6fe5a1d6f4778720a90e4822.png) | ![[Uncaptioned image]](img/91a330c644d95dd212aea2b9e61abd40.png) conformist | 0.535 | 0.569 | 82.03 |
| ![[Uncaptioned image]](img/bef28eca00ac15673e419ed21c639c67.png) authoritarian | 0.319 | 0.266 | 69.41 |
| ![[Uncaptioned image]](img/8b8a369efa426121e8ad0f1fa0f25ff3.png) inclusive | 0.542 | 0.578 | 82.41 |
| ![[Uncaptioned image]](img/459b1913497d2d0b1339ac1bebd31c23.png) | ![[Uncaptioned image]](img/3794d454bf018b846cbb834d0bf4b562.png) no rebuttals | 0.622 | 0.668 | 86.14 |
| ![[Uncaptioned image]](img/109a3f0cb6831f5ef136823160a539ff.png) no numeric rating | 0.200 | 0.052 | 60.40 |

Table 2: Comparison of final decisions in various settings relative to the *baseline* experiment in terms of Jaccard Index (Jacc.), Cohen’s Kappa Coefficient ($\kappa$), and Percentage Agreement (%Agree). Jacc. indicate the set of papers accepted by both the investigated setting and the baseline. The highest and second highest values are highlighted in bold and underlined, respectively.

Groupthink [[24](https://arxiv.org/html/2406.12708v2#bib.bib24)] occurs when a group of reviewers, driven by a desire for harmony or conformity, reach a consensus without critical reasoning or evaluation of a manuscript. It can be especially detrimental when the group includes irresponsible or malicious reviewers. To examine such effects, we substitute $1\sim 3$ normal reviewers with irresponsible reviewers and analyze the changes in ratings before & after reviewer-AC discussion.

Table [3](https://arxiv.org/html/2406.12708v2#S3.T3 "Table 3 ‣ 3.1.3 Reviewer Intention ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") highlights a noticeable decline in review ratings under the influence of irresponsible reviewers. Replacing 2 normal reviewers with irresponsible ones results in a significant drop of 0.25 from 5.256 to 5.005 in the average reviewer rating after Reviewer-AC Discussion (Phase III). In contrast, in the *baseline* scenario, the final ratings improve by an average 0.06 post-rebuttal, as reviewers more proactively scrutinize the author feedback and have their concerns addressed. Interestingly, the scores among irresponsible reviewers exhibit a slight increase, suggesting a tendency to conform to the assessments of normal reviewers.

![Refer to caption](img/c5e4d10962c4a03796afe2a8c08cbb24.png)

Figure 4: Distribution of reasons for acceptance and rejections.

##### 3.1.3 Reviewer Intention

Conflict Theory [[31](https://arxiv.org/html/2406.12708v2#bib.bib31)] states that societal interactions are often driven by conflict rather than consensus. In the context of peer review, where the acceptance of papers is competitive, reviewers may perceive other high-quality submissions as threats to their own work due to conflict of interests. This competitive behavior can lead to low ratings for competing papers, particularly for concurrent works with highly similar ideas, as reviewers aim to protect their own standing in the field. Empirically, the reviewer ratings in Figure [9](https://arxiv.org/html/2406.12708v2#A1.F9 "Figure 9 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") show a significant shift to a bimodal distribution, primarily centered around $[4.0,4.25]$, when just one malicious reviewer is involved. This forms a stark contrast to the unimodal distribution between $[5.0,5.25]$ observed in the *baseline* condition.

Echo Chamber Effects [[25](https://arxiv.org/html/2406.12708v2#bib.bib25)] occur when a group of reviewers sharing similar biases amplify their opinions, leaning towards a collective decision without critically evaluating merits of the work. As illustrated in Figure [3](https://arxiv.org/html/2406.12708v2#S3.F3 "Figure 3 ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), increasing the number of malicious reviewers from 0 to 3 results in a consistent drop in the average rating from 5.11 to 3.35, suggesting that the presence of malicious reviewers significantly impacts the overall evaluation. Meanwhile, as malicious reviewers predominate, the average rating among these biased reviewers (Table [5](https://arxiv.org/html/2406.12708v2#A1.T5 "Table 5 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")) experiences a greater drop post-rebuttal, indicating that the inclusion of more biased reviewers not only amplifies the paper’s issues but also solidifies their strong negative opinions about the work. This process not only reinforces pre-existing biases and reduces critical scrutiny, but also has a spillover effect that adversely impacts evaluations from unbiased reviewers. The presence of 1 and 2 *malicious* reviewers corresponds to a decline by 0.14 and 0.10, respective, among the normal reviewers.

Content-level Analysis We categorize the reasons for acceptance and rejection as shown in Figure [4](https://arxiv.org/html/2406.12708v2#S3.F4 "Figure 4 ‣ 3.1.2 Reviewer Commitment ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") with additional details provided in Appendix [A.1](https://arxiv.org/html/2406.12708v2#A1.SS1 "A.1 Review Categorization ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"). While reasons for accepting the papers are consistent across all settings, the reasons for rejection differ significantly in distribution. Irresponsible reviews tend to be shallow, cursory, and notably 22.2% shorter, whereas malicious reviews disproportionally criticize the *lack of novelty* in the work (Figure [4](https://arxiv.org/html/2406.12708v2#S3.F4 "Figure 4 ‣ 3.1.2 Reviewer Commitment ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")d), a common but vague reason for rejection. Specifically, mentions of *lack of novelty* by *malicious* reviewers account for 10.4% of feedback, marking a 182.9% increase compared to just 3.69% by *benign* reviewers. They also highlight more *presentation* issues which, although important for clarity, do not pertain to the theoretical soundness of the research. On the other hand, benign reviewers tend to focus more on discussions about *scalability and practicality* issues, providing suggestions to help enhance papers’ comprehensiveness.

 | ![[Uncaptioned image]](img/839cd41e1a0b85f054ac6d41f37a77af.png) normal reviewers | ![[Uncaptioned image]](img/0aae60770b05dc81ece85e3b7f4fa0e9.png) irresponsible reviewers |
| --- | --- |
| # | Initial | Final | $+/-$ | # | Initial | Final | $+/-$ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 3 | 5.053 $\pm$ 0.623 | 5.110 $\pm$ 0.555 | $+$0.06 | 0 | / | / | / |
| 2 | 5.056 $\pm$ 0.633 | 5.015 $\pm$ 0.546 | $-$0.04 | 1 | 4.139 $\pm$ 1.121 | 4.416 $\pm$ 0.925 | $+$0.27 |
| 1 | 5.256 $\pm$ 0.896 | 5.005 $\pm$ 0.630 | $-$0.25 | 2 | 4.548 $\pm$ 0.925 | 4.543 $\pm$ 0.872 | $-$0.01 |
| 0 | / | / | / | 3 | 4.591 $\pm$ 0.912 | 4.677 $\pm$ 0.745 | $+$0.09 | 

Table 3: Average reviewer ratings when varying numbers of ![[Uncaptioned image]](img/839cd41e1a0b85f054ac6d41f37a77af.png) *normal* reviewers are replaced by ![[Uncaptioned image]](img/f7a465c680c30630255fc96bfd095e9e.png) irresponsible reviewers. ‘#’ represents the number of reviewers of each type. ‘Initial’ & ‘Final’ refer to the average ratings in Phase I & III. The left and right side of the table shows average ratings from ![[Uncaptioned image]](img/839cd41e1a0b85f054ac6d41f37a77af.png) normal reviewers and ![[Uncaptioned image]](img/e533aadc5a80033bcbcbec4277dee1eb.png) irresponsible reviewers, respectively. $+/-$ indicates the change in average ratings after rebuttals.

##### 3.1.4 Reviewer Knowledgeability

Knowledgeability poses two challenges. Firstly, despite extended efforts at matching expertise, review assignments are often imperfect or random [[6](https://arxiv.org/html/2406.12708v2#bib.bib6), [32](https://arxiv.org/html/2406.12708v2#bib.bib32)]. Secondly, the recent surge in submissions to computer science conferences has necessitated an expansion of the reviewer pools, raising concerns about the adequacy of reviewers’ expertise to conduct proper and effective evaluations. As shown in Figure [4](https://arxiv.org/html/2406.12708v2#S3.F4 "Figure 4 ‣ 3.1.2 Reviewer Commitment ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), less knowledgeable reviewers are 24% more likely to mention *insufficient discussion of limitations*, whereas expert reviewers not only address these basic aspects but also provide 6.8 % more critiques on experimental validation, resulting in more concrete and beneficial feedback for improving the paper.

#### 3.2 Involvements of Area Chairs

We quantify the alignment between reviews and meta-reviews using BERTScore [[33](https://arxiv.org/html/2406.12708v2#bib.bib33)] and sentence embedding similarity [[34](https://arxiv.org/html/2406.12708v2#bib.bib34)] in Table [2](https://arxiv.org/html/2406.12708v2#S3.T2 "Table 2 ‣ 3.1.2 Reviewer Commitment ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), and measure the agreement of final decisions between *baseline* and each setting in Figure [5](https://arxiv.org/html/2406.12708v2#S3.F5 "Figure 5 ‣ 3.4 Effects of Peer Review Mechanisms ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"). Inclusive ACs align most closely with the *baseline* for final decisions, demonstrating their effectiveness in integrating diverse viewpoints and maintaining the integrity of the review process through a balanced consideration of reviews and their own expertise. In contrast, authoritarian ACs manifest significantly lower correlation with the *baseline*, with a Cohen’s Kappa of only 0.266 and an agreement rate of 69.8%. This suggests that their decisions may be skewed by personal biases, leading to acceptance of lower quality papers or the rejection of high-quality papers that do not align with their viewpoints, thereby compromising the integrity and fairness of the peer review process. Conformist ACs, while showing a high semantic overlap with reviewers’ evaluations as evidenced in Figure [5](https://arxiv.org/html/2406.12708v2#S3.F5 "Figure 5 ‣ 3.4 Effects of Peer Review Mechanisms ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), might lack independent judgment. This dependency could perpetuate existing biases or errors in initial reviews, underscoring a critical flaw in overly deferential approaches.

#### 3.3 Impacts of Author Anonymity

Recent conferences have increasingly permitted the release of preprints, potentially impacting paper acceptance [[35](https://arxiv.org/html/2406.12708v2#bib.bib35)]. Although reviewers are instructed not to proactively seek information about author identities, concerns persist that reviews may still be biased by author reputation.

Authority bias is the tendency to attribute greater accuracy and credibility to the opinions of authority figures. This bias is closely related to the Halo Effects, a cognitive bias where the positive perception of an individual in one area, such as their previous groundbreaking research, influences judgments about their current work. Reviewers influenced by authority bias are more likely to give favorable reviews to well-known and respected scientists.

To analyze the impact of author identities on review outcomes, we vary the number of reviewers aware of the authors’ identities ($k$), ranging from 1 to 3, and adjusted the proportion of papers with known author identities ($r$) from 10% to 30%. Specifically, the reviewers were informed that the authors of certain papers were renowned and highly accomplished in the field. We categorized papers into two types: higher quality and lower quality, based on their ground-truth acceptance decisions.

For lower-quality papers, awareness of the authors’ renowned identities among 1, 2, or 3 reviewers resulted in Jaccard indices of 0.364, 0.154, and 0.008, respectively, in terms of paper acceptance (Figure [6](https://arxiv.org/html/2406.12708v2#S3.F6 "Figure 6 ‣ 3.4 Effects of Peer Review Mechanisms ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). The most extreme case has a negative Cohen’s Kappa $\kappa$ (Figure [8](https://arxiv.org/html/2406.12708v2#A1.F8 "Figure 8 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")), indicating a substantial deviation in paper decisions. When high-quality papers had known author identities, much less significant changes were observed in accepted papers. Notably, changes in paper decisions are more influenced by the number of reviewers aware of the author identities than by the percentage of papers with known author identities.

#### 3.4 Effects of Peer Review Mechanisms

We investigate two variations to peer review mechanisms. 1) *no rebuttal*—excluding the Reviewer-Author Discussion (Phase II) and the Reviewer-AC Discussion (Phase III); 2) *no numeric rating*—removing the requirement to assign overall numeric ratings (Phase I & III), thus making the AC’s decision solely dependent on the content of the reviews.

![Refer to caption](img/81ce9db80314244d528251e3bc4ab6c0.png)

Figure 5: Similarities between reviews and meta-reviews w/ various intervention strategies from AC. Left: BERTScore, right: sentence embedding similarity.

Effects of Rebuttals. Eliminating the rebuttal phase, which requires substantial time commitments from both reviewers and authors, has a surprisingly minimal impact on the final paper decisions, aligning closely with the *baseline* scenario.

One explanation for this minimal impact is the *anchoring bias*, where the initial impression formed during the first submission (the “anchor”) predominantly influences reviewers’ judgments. Even though authors may make substantial improvements during the rebuttal phase that address reviewers’ concerns (Sec. [3.1.1](https://arxiv.org/html/2406.12708v2#S3.SS1.SSS1 "3.1.1 Overview ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")), these changes may fail to alter their initial judgments. Another plausible reason is that all submissions improve in quality during the rebuttal phase. Thus, the relative position (ranking of quality) of each paper among all submissions experiences little change.

Effects of Overall Ratings. Numeric ratings from reviewers may serve as a shortcut in the final decision-making process for paper acceptance. When these ratings are omitted, the decision-making landscape changes significantly, leading to potentially divergent decisions. The comparison of outcomes with respect to *baseline* reveals only a minimal overlap, with a Jaccard index of 0.20 in terms of accepted papers (Table [2](https://arxiv.org/html/2406.12708v2#S3.T2 "Table 2 ‣ 3.1.2 Reviewer Commitment ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")).

![Refer to caption](img/6cf938ca155cf79fed1c2008f8bc1065.png)

Figure 6: Comparison of final decisions with respect to *baseline* when the author identity is known for varying ratios of papers, relative to the *baseline*. Smaller Jaccard indices suggest lower correlation with the baseline.

### 4 Related Work

Analysis of Peer Review Systems. Peer review serves as the backbone of academic research, ensuring the integrity and quality of published work [[36](https://arxiv.org/html/2406.12708v2#bib.bib36)]. Several studies have scrutinized various challenges within peer review, such as bias [[1](https://arxiv.org/html/2406.12708v2#bib.bib1), [37](https://arxiv.org/html/2406.12708v2#bib.bib37), [38](https://arxiv.org/html/2406.12708v2#bib.bib38)], conflict of interests [[39](https://arxiv.org/html/2406.12708v2#bib.bib39)], and the broader issues of review quality and fairness [[1](https://arxiv.org/html/2406.12708v2#bib.bib1), [39](https://arxiv.org/html/2406.12708v2#bib.bib39), [40](https://arxiv.org/html/2406.12708v2#bib.bib40)]. Research has also delved into the operational aspects, such as reviewer assignments [[41](https://arxiv.org/html/2406.12708v2#bib.bib41), [32](https://arxiv.org/html/2406.12708v2#bib.bib32), [42](https://arxiv.org/html/2406.12708v2#bib.bib42)] and author rebuttals [[43](https://arxiv.org/html/2406.12708v2#bib.bib43)], identifying areas for improvement in transparency, fairness, and accountability [[2](https://arxiv.org/html/2406.12708v2#bib.bib2)]. These studies primarily focus on analyzing existing real-world review data and outcomes. However, due to the complexity and inherent variability of peer review, isolating the effects of specific factors on review outcomes remains a significant challenge.

LLMs as Agents. Large Language Models (LLMs) such as GPT-4 [[8](https://arxiv.org/html/2406.12708v2#bib.bib8)], Claude 3 [[44](https://arxiv.org/html/2406.12708v2#bib.bib44)], and Gemini [[45](https://arxiv.org/html/2406.12708v2#bib.bib45)] have not only demonstrated sophisticated language understanding [[46](https://arxiv.org/html/2406.12708v2#bib.bib46), [47](https://arxiv.org/html/2406.12708v2#bib.bib47), [48](https://arxiv.org/html/2406.12708v2#bib.bib48)], reasoning [[49](https://arxiv.org/html/2406.12708v2#bib.bib49), [50](https://arxiv.org/html/2406.12708v2#bib.bib50), [51](https://arxiv.org/html/2406.12708v2#bib.bib51)] and generation skills [[52](https://arxiv.org/html/2406.12708v2#bib.bib52), [53](https://arxiv.org/html/2406.12708v2#bib.bib53), [54](https://arxiv.org/html/2406.12708v2#bib.bib54), [55](https://arxiv.org/html/2406.12708v2#bib.bib55), [56](https://arxiv.org/html/2406.12708v2#bib.bib56), [57](https://arxiv.org/html/2406.12708v2#bib.bib57)], but also exhibit planning, collaboration, and competitive behaviors [[14](https://arxiv.org/html/2406.12708v2#bib.bib14), [58](https://arxiv.org/html/2406.12708v2#bib.bib58), [59](https://arxiv.org/html/2406.12708v2#bib.bib59)]. These capabilities have facilitated their adoption as autonomous agents that interact with each other to complete tasks [[60](https://arxiv.org/html/2406.12708v2#bib.bib60), [61](https://arxiv.org/html/2406.12708v2#bib.bib61)]. Our study aligns with recent works in agent-based modeling (ABM) [[12](https://arxiv.org/html/2406.12708v2#bib.bib12), [60](https://arxiv.org/html/2406.12708v2#bib.bib60), [61](https://arxiv.org/html/2406.12708v2#bib.bib61), [62](https://arxiv.org/html/2406.12708v2#bib.bib62), [63](https://arxiv.org/html/2406.12708v2#bib.bib63)] that leverage the capabilities of LLM agents to simulate realistic environments for scientific research.

### 5 Conclusion

We presented AgentReview, the first LLM-based framework for simulating the peer review process. AgentReview addresses key challenges by disentangling intertwined factors that impact review outcomes while preserving reviewer privacy. Our work lays a solid foundation for more equitable and transparent review mechanism designs in academic publishing. Future works could investigate how intricate interactions between different variables collectively affect review outcomes.

### Limitation

Our work has the following limitations. First, AgentReview is unable to dynamically incorporate or adjust experimental results in response to reviewer comments during Reviewer-Author Discussion (Phase II in Figure [2](https://arxiv.org/html/2406.12708v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")), as LLMs lack the capability to generate new empirical data. Secondly, our analysis mainly isolates and examines individual variables of the peer review process, such as reviewer commitment or knowledgeability. Real-world peer reviews, however, involve multiple interacting dimensions. Finally, we did not directly compare the simulation outcomes with actual peer review results. As described in Sec [2.4](https://arxiv.org/html/2406.12708v2#S2.SS4 "2.4 Baseline Setting ‣ 2 The AgentReview Framework ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"), establishing a consistent baseline for such comparisons is challenging due to the wide variability in human reviewer characteristics, such as commitment, intention, and knowledgeability, which can vary across papers, topics, and time periods. The inherent variability and arbitrariness in human peer reviews [[28](https://arxiv.org/html/2406.12708v2#bib.bib28)] add complexity to direct comparisons between simulated and real outcomes.

### Ethical Consideration

Further Investigation into Peer Review data. The sensitivity and scarcity of real-world review data complicate comprehensive studies of peer reviews due to ethical and confidentiality constraints. Our AgentReview framework generates simulated data to study various peer review dynamics, effectively overcoming related challenges.

Peer Review Integrity. As discussed, the integrity of the peer review process is underpinned by the commitment, intention, and knowledgeability of reviewers. *Knowledgeability* ensures that reviewers can accurately assess the novelty, significance, and technical soundness of submissions. Good *intention* are essential for maintaining the objectivity and fairness of reviews, thereby supporting the credibility and integrity of academic publications. A high level of *commitment* from reviewers ensures comprehensive and considerate evaluations of submission, which is important for a fair and rigorous evaluation process. However, paper review is usually an unpaid and time-consuming task. Such demanding nature can lead the reviewers to conduct cursory or superficial evaluations.

Caution about Use of LLMs. Our AgentReview mirrors real-world academic review practices to ensure the authenticity and relevance of our findings. While AgentReview uses LLMs to generate paper reviews, there are ethical concerns regarding their use in actual peer review processes [[64](https://arxiv.org/html/2406.12708v2#bib.bib64)]. Recent machine learning conferences have shown an increase in reviews suspected to be AI-generated [[65](https://arxiv.org/html/2406.12708v2#bib.bib65)]. Although LLM-generated reviews can provide valuable feedback, we strongly advise against their use as replacements for human reviewers in real-world peer review processes. As LLMs are still imperfect, human oversight is crucial for ensuring fair and valuable assessments of manuscripts and for maintaining the integrity and quality of peer reviews.

### References

*   [1] Ivan Stelmakh, Nihar B Shah, Aarti Singh, and Hal Daumé III. Prior and prejudice: The novice reviewers’ bias against resubmissions in conference peer review. HCI, 5(CSCW1):1–17, 2021.
*   [2] Jiayao Zhang, Hongming Zhang, Zhun Deng, and Dan Roth. Investigating fairness disparities in peer review: A language model enhanced approach. arXiv:2211.06398, 2022.
*   [3] Charles W Fox, Jennifer Meyer, and Emilie Aimé. Double-blind peer review affects reviewer ratings and editor decisions at an ecology journal. Functional Ecology, 37(5):1144–1157, 2023.
*   [4] Mengyi Sun, Jainabou Barry Danfa, and Misha Teplitskiy. Does double-blind peer review reduce bias? evidence from a top computer science conference. Journal of the Association for Information Science and Technology, 73(6):811–819, 2022.
*   [5] Yuxuan Lu and Yuqing Kong. Calibrating “cheap signals” in peer review without a prior. NeurIPS, 36, 2024.
*   [6] Yixuan Xu, Steven Jecmen, Zimeng Song, and Fei Fang. A one-size-fits-all approach to improving randomness in paper assignment. NeurIPS, 36, 2024.
*   [7] Ying Liu, Kaiqi Yang, Yue Liu, and Michael GB Drew. The shackles of peer review: Unveiling the flaws in the ivory tower. arXiv:2310.05966, 2023.
*   [8] OpenAI. Gpt-4 technical report. Arxiv Preprint, arXiv:2303.08774, 2023.
*   [9] Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, et al. Llama 2: Open foundation and fine-tuned chat models. arXiv:2307.09288, 2023.
*   [10] Significant-Gravitas. Autogpt. [https://github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT), 2023.
*   [11] Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. Autogen: Enabling next-gen llm applications via multi-agent conversation framework. arXiv:2308.08155, 2023.
*   [12] Yuxiang Wu, Zhengyao Jiang, Akbir Khan, Yao Fu, Laura Ruis, Edward Grefenstette, and Tim Rocktäschel. Chatarena: Multi-agent language game environments for large language models. GitHub repository, 2023.
*   [13] Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chi-Min Chan, Heyang Yu, Yaxi Lu, Yi-Hsin Hung, Chen Qian, et al. Agentverse: Facilitating multi-agent collaboration and exploring emergent behaviors. In ICLR, 2023.
*   [14] Qinlin Zhao, Jindong Wang, Yixuan Zhang, Yiqiao Jin, Kaijie Zhu, Hao Chen, and Xing Xie. Competeai: Understanding the competition behaviors in large language model-based agents. In ICML, 2024.
*   [15] Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S Bernstein. Generative agents: Interactive simulacra of human behavior. In UIST, pages 1–22, 2023.
*   [16] Ruosen Li, Teerth Patel, and Xinya Du. Prd: Peer rank and discussion improve large language model based evaluations. arXiv preprint arXiv:2307.02762, 2023.
*   [17] Liang Niu, Nian Xue, and Christina Pöpper. Unveiling the sentinels: Assessing ai performance in cybersecurity peer review. arXiv:2309.05457, 2023.
*   [18] Weixin Liang, Yuhui Zhang, Hancheng Cao, Binglu Wang, Daisy Ding, Xinyu Yang, Kailas Vodrahalli, Siyu He, Daniel Smith, Yian Yin, et al. Can large language models provide useful feedback on research papers? a large-scale empirical analysis. arXiv:2310.01783, 2023.
*   [19] Mahsa Shamsabadi and Jennifer D’Souza. A fair and free prompt-based research assistant. arXiv:2405.14601, 2024.
*   [20] Miao Li, Jey Han Lau, and Eduard Hovy. Exploring multi-document information consolidation for scientific sentiment summarization. arXiv:2402.18005, 2024.
*   [21] Mike D’Arcy, Tom Hope, Larry Birnbaum, and Doug Downey. Marg: Multi-agent review generation for scientific papers. arXiv:2401.04259, 2024.
*   [22] John C Turner. Social influence. Thomson Brooks/Cole Publishing Co, 1991.
*   [23] Joshua D Angrist. The perils of peer effects. Labour Economics, 30:98–108, 2014.
*   [24] Irving L Janis. Groupthink. IEEE Engineering Management Review, 36(1):36, 2008.
*   [25] Matteo Cinelli, Gianmarco De Francisci Morales, Alessandro Galeazzi, Walter Quattrociocchi, and Michele Starnini. The echo chamber effect on social media. PNAS, 118(9):e2023301118, 2021.
*   [26] Richard E Nisbett and Timothy D Wilson. The halo effect: Evidence for unconscious alteration of judgments. Journal of personality and social psychology, 35(4):250, 1977.
*   [27] Mahsan Nourani, Chiradeep Roy, Jeremy E Block, Donald R Honeycutt, Tahrima Rahman, Eric Ragan, and Vibhav Gogate. Anchoring bias affects mental model formation and user reliance in explainable ai systems. In IUI, pages 340–350, 2021.
*   [28] Corinna Cortes and Neil D Lawrence. Inconsistency in conference peer review: revisiting the 2014 neurips experiment. arXiv:2109.09774, 2021.
*   [29] Robert B Cialdini and Noah J Goldstein. Social influence: Compliance and conformity. Annu. Rev. Psychol., 55:591–621, 2004.
*   [30] Guangyao Zhang, Furong Shang, Weixi Xie, Yuhan Guo, Chunlin Jiang, and Xianwen Wang. Do conspicuous manuscripts experience shorter time in the duration of peer review? arXiv:2112.09360, 2021.
*   [31] Otomar J Bartos and Paul Wehr. Using conflict theory. Cambridge University Press, 2002.
*   [32] Martin Saveski, Steven Jecmen, Nihar Shah, and Johan Ugander. Counterfactual evaluation of peer-review assignment policies. NeurIPS, 36, 2024.
*   [33] Tianyi Zhang, Varsha Kishore, Felix Wu, Kilian Q Weinberger, and Yoav Artzi. Bertscore: Evaluating text generation with bert. In ICLR, 2020.
*   [34] Nils Reimers and Iryna Gurevych. Sentence-bert: Sentence embeddings using siamese bert-networks. In EMNLP, pages 3982–3992, 2019.
*   [35] Yanai Elazar, Jiayao Zhang, David Wadden, Bo Zhang, and Noah A Smith. Estimating the causal effect of early arxiving on paper acceptance. In CLeaR, pages 913–933\. PMLR, 2024.
*   [36] Yichi Zhang, Fang-Yi Yu, Grant Schoenebeck, and David Kempe. A system-level analysis of conference peer review. In EC, pages 1041–1080, 2022.
*   [37] Alexander Ugarov. Peer prediction for peer review: designing a marketplace for ideas. arXiv:2303.16855, 2023.
*   [38] Jeroen PH Verharen. Chatgpt identifies gender disparities in scientific peer review. Elife, 12:RP90230, 2023.
*   [39] Leslie D McIntosh and Cynthia Hudson Vitale. Safeguarding scientific integrity: Examining conflicts of interest in the peer review process. arXiv:2308.04297, 2023.
*   [40] Dimity Stephen. Distinguishing articles in questionable and non-questionable journals using quantitative indicators associated with quality. arXiv:2405.06308, 2024.
*   [41] Jelena Jovanovic and Ebrahim Bagheri. Reviewer assignment problem: A scoping review. arXiv:2305.07887, 2023.
*   [42] Kayvan Kousha and Mike Thelwall. Artificial intelligence to support publishing and peer review: A summary and review. Learned Publishing, 37(1):4–12, 2024.
*   [43] Junjie Huang, Win-bin Huang, Yi Bu, Qi Cao, Huawei Shen, and Xueqi Cheng. What makes a successful rebuttal in computer science conferences?: A perspective on social interaction. Journal of Informetrics, 17(3):101427, 2023.
*   [44] Anthropic. Introducing the next generation of claude, 2024.
*   [45] Gemini Team, Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, et al. Gemini: a family of highly capable multimodal models. arXiv:2312.11805, 2023.
*   [46] Kaijie Zhu, Jindong Wang, Qinlin Zhao, Ruochen Xu, and Xing Xie. Dynamic evaluation of large language models by meta probing agents. In ICML, 2024.
*   [47] Yiqiao Jin, Mohit Chandra, Gaurav Verma, Yibo Hu, Munmun De Choudhury, and Srijan Kumar. Better to ask in english: Cross-lingual evaluation of large language models for healthcare queries. In Web Conference, 2024.
*   [48] Yiqiao Jin, Minje Choi, Gaurav Verma, Jindong Wang, and Srijan Kumar. Mm-soc: Benchmarking multimodal large language models in social media platforms. In ACL, 2024.
*   [49] Weiqi Wang and Yangqiu Song. Mars: Benchmarking the metaphysical reasoning abilities of language models with a multi-task evaluation dataset, 2024.
*   [50] Tianyu Liu, Tianqi Chen, Wangjie Zheng, Xiao Luo, and Hongyu Zhao. scelmo: Embeddings from language models are good learners for single-cell data analysis. bioRxiv, pages 2023–12, 2023.
*   [51] Haoran Wang and Kai Shu. Backdoor activation attack: Attack large language models using activation steering for safety-alignment. arXiv:2311.09433, 2023.
*   [52] Yijia Xiao, Yiqiao Jin, Yushi Bai, Yue Wu, Xianjun Yang, Xiao Luo, Wenchao Yu, Xujiang Zhao, Yanchi Liu, Haifeng Chen, et al. Large language models can be good privacy protection learners. In EMNLP, 2024.
*   [53] Jinghan Zhang, Xiting Wang, Yiqiao Jin, Changyu Chen, Xinhao Zhang, and Kunpeng Liu. Prototypical reward network for data-efficient rlhf. In ACL, 2024.
*   [54] Wenyue Hua, Kaijie Zhu, Lingyao Li, Lizhou Fan, Shuhang Lin, Mingyu Jin, Haochen Xue, Zelong Li, JinDong Wang, and Yongfeng Zhang. Disentangling logic: The role of context in large language model reasoning capabilities. arXiv preprint arXiv:2406.02787, 2024.
*   [55] Kaijie Zhu, Jindong Wang, Qinlin Zhao, Ruochen Xu, and Xing Xie. Dyval 2: Dynamic evaluation of large language models by meta probing agents. arXiv:2402.14865, 2024.
*   [56] Lizhou Fan, Wenyue Hua, Xiang Li, Kaijie Zhu, Mingyu Jin, Lingyao Li, Haoyang Ling, Jinkui Chi, Jindong Wang, Xin Ma, et al. Nphardeval4v: A dynamic reasoning benchmark of multimodal large language models. arXiv:2403.01777, 2024.
*   [57] Changyu Chen, Xiting Wang, Yiqiao Jin, Victor Ye Dong, Li Dong, Jie Cao, Yi Liu, and Rui Yan. Semi-offline reinforcement learning for optimized text generation. In ICML, 2023.
*   [58] Yushi Bai, Jiahao Ying, Yixin Cao, Xin Lv, Yuze He, Xiaozhi Wang, Jifan Yu, Kaisheng Zeng, Yijia Xiao, Haozhe Lyu, et al. Benchmarking foundation models with language-model-as-an-examiner. arXiv:2306.04181, 2023.
*   [59] Chengxing Xie, Canyu Chen, Feiran Jia, Ziyu Ye, Kai Shu, Adel Bibi, Ziniu Hu, Philip Torr, Bernard Ghanem, and Guohao Li. Can large language model agents simulate human trust behaviors? arXiv:2402.04559, 2024.
*   [60] Da Yin, Faeze Brahman, Abhilasha Ravichander, Khyathi Chandu, Kai-Wei Chang, Yejin Choi, and Bill Yuchen Lin. Lumos: Learning Agents with Unified Data, Modular Design, and Open-Source LLMs. arXiv:2311.05657, 2023.
*   [61] Zhaoyi Li, Kelin Yu, Shuo Cheng, and Danfei Xu. League++: Empowering continual robot learning through guided skill acquisition with large language models. In ICLR 2024 Workshop on Large Language Model (LLM) Agents, 2024.
*   [62] Xuechen Li, Tianyi Zhang, Yann Dubois, Rohan Taori, Ishaan Gulrajani, Carlos Guestrin, Percy Liang, and Tatsunori B Hashimoto. Alpacaeval: An automatic evaluator of instruction-following models, 2023.
*   [63] Chi-Min Chan, Weize Chen, Yusheng Su, Jianxuan Yu, Wei Xue, Shanghang Zhang, Jie Fu, and Zhiyuan Liu. Chateval: Towards better llm-based evaluators through multi-agent debate. In ICLR, 2023.
*   [64] Ji-Ung Lee, Haritz Puerto, Betty van Aken, Yuki Arase, Jessica Zosa Forde, Leon Derczynski, Andreas Rücklé, Iryna Gurevych, Roy Schwartz, Emma Strubell, et al. Surveying (dis) parities and concerns of compute hungry nlp research. arXiv:2306.16900, 2023.
*   [65] Weixin Liang, Zachary Izzo, Yaohui Zhang, Haley Lepp, Hancheng Cao, Xuandong Zhao, Lingjiao Chen, Haotian Ye, Sheng Liu, Zhi Huang, et al. Monitoring ai-modified content at scale: A case study on the impact of chatgpt on ai conference peer reviews. arXiv:2403.07183, 2024.
*   [66] Xu Ma, Yuqian Zhou, Huan Wang, Can Qin, Bin Sun, Chang Liu, and Yun Fu. Image as set of points. In ICLR, 2022.

## Appendix

\parttoc

### Appendix A Experimental Details

#### A.1 Review Categorization

In our experiment, we utilize GPT-4 to summarize and categorize the reasons for paper acceptance and rejection, as illustrated in Figure [4](https://arxiv.org/html/2406.12708v2#S3.F4 "Figure 4 ‣ 3.1.2 Reviewer Commitment ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"). Specifically, we analyze each line from the ‘reasons for acceptance’ and ‘reasons for rejection’ fields in the generated reviews. GPT-4 is tasked with automatically classifying each listed reason. If an entry does not align with predefined categories, the model establish a new category. Ultimately, we identify five distinct reasons for acceptance and seven reasons for rejection.

|  | #Words | #Characters |
| --- | --- | --- |
| Review | 438.2 $\pm$ 72.0 | 3067.4 $\pm$ 510.1 |
| Rebuttal | 370.6 $\pm$ 49.9 | 2584.8 $\pm$ 376.5 |
| Updated Review | 189.7 $\pm$ 46.6 | 1304.0 $\pm$ 320.8 |
| Meta-review | 256.9 $\pm$ 64.8 | 1849.9 $\pm$ 454.5 |

Table 4: Statistics of our dataset.

![Refer to caption](img/c1abaaa4afc93417fccebd8d35f7b901.png)

Figure 7: Distribution of initial and final ratings when varying numbers of reviewers are aware of the authors’ prestigious identity.

#### A.2 Experimental Costs

To ensure consistent evaluation results, we use the gpt-4-1106-preview version of the GPT-4 model throughout our experiments. The model is selected for its superior language understanding and generation capabilities, essential for simulating an authentic peer review process. To enhance reproducibility and minimize API usage, we establish a *baseline* settings (Sec. [2.4](https://arxiv.org/html/2406.12708v2#S2.SS4 "2.4 Baseline Setting ‣ 2 The AgentReview Framework ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")), where no specific personalities of the role are detailed (‘*baseline*’ in Table [1](https://arxiv.org/html/2406.12708v2#S3.T1 "Table 1 ‣ 3.1.1 Overview ‣ 3.1 The Role of Reviewers ‣ 3 Results ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents")). This setting allows us to measure the impact of changes in individual variables against a consistent standard. For subsequent experiments, we adopt reviews and rebuttals (Phase I-II) from this *baseline* when applicable. For example, when we investigate the effects of substituting a normal reviewer with an irresponsible person, we only generate the reviews for that specific reviewer while adopting existing reviews from the *baseline* setting. This approach minimizes the variability caused by different experimental runs and significantly reduces the API cost compared with rerunning the entire review pipeline each time. The total cost of API usage across all tests is approximately $2780.

#### A.3 Model Selection

Additionally, we have also explored the feasibility of alternative models, such as gpt-35-turbo and Gemini. These models were initially considered to assess the cost-effectiveness and performance diversity. However, these models either encounter issues related to content filtering limitations, resulting in the omission of critical feedback, or generate superficial evaluations and exhibited a bias towards overly generous scoring. Therefore, despite the higher operational costs, we choose despite the higher operational costs, due to its more consistent and realistic output in peer review simulations due to its more consistent and realistic output in peer review simulations.

#### A.4 Additional Results and Statistics

*   •

    Table [4](https://arxiv.org/html/2406.12708v2#A1.T4 "Table 4 ‣ A.1 Review Categorization ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") is the statistics of our dataset, including the word and character counts of the generated reviews, rebuttals, updated reviews, and meta-reviews.

*   •

    Table [5](https://arxiv.org/html/2406.12708v2#A1.T5 "Table 5 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") is the average reviewer ratings when varying number of *normal* reviewers are replaced by *malicious* reviewers.

*   •

    Table [6](https://arxiv.org/html/2406.12708v2#A1.T6 "Table 6 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"),[7](https://arxiv.org/html/2406.12708v2#A1.T7 "Table 7 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"),[8](https://arxiv.org/html/2406.12708v2#A1.T8 "Table 8 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents"),[9](https://arxiv.org/html/2406.12708v2#A1.T9 "Table 9 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") present the generated review, rebuttal, and meta-review for the paper *Image as Set of Points* [[66](https://arxiv.org/html/2406.12708v2#bib.bib66)] in our simulation. LLM-generate review shows a high degree of overlap with actual reviews in Table [10](https://arxiv.org/html/2406.12708v2#A1.T10 "Table 10 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents").

*   •

    Table [9](https://arxiv.org/html/2406.12708v2#A1.F9 "Figure 9 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") shows the prompts used in AgentReview and the characteristics of each type of roles.

*   •

    Figure [7](https://arxiv.org/html/2406.12708v2#A1.F7 "Figure 7 ‣ A.1 Review Categorization ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") is the distribution of initial and final ratings as $0\sim 3$ reviewers become aware of the authors’ prestigious identity. It shows that the average reviewer ratings consistently increase with more reviewers knowing the author identities. Meanwhile, reviewer ratings consistently increase after rebuttals.

*   •

    Figure [8](https://arxiv.org/html/2406.12708v2#A1.F8 "Figure 8 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") is the Cohen’s Kappa coefficient ($\kappa$) when the author identity is known for varying ratios of papers, relative to the *baseline*. Different lines represent different numbers of reviewers that are aware of the authors’ identities.

*   •

    Figure [9](https://arxiv.org/html/2406.12708v2#A1.F9 "Figure 9 ‣ A.4 Additional Results and Statistics ‣ Appendix A Experimental Details ‣ Appendix ‣ AgentReview: Exploring Peer Review Dynamics with LLM Agents") is the final rating distribution when we vary one reviewer in the experiment, including their commitment, intention, or knowledgeability. Reviewers powered by LLMs assign highly consistent numeric ratings to most submissions, with the majority of the scores in $[5,5.25]$. Notable exceptions occur under the *irresponsible* and *malicious* settings, where the ratings exhibit a bimodal distribution with peaks at $[5,5.25]$ and $[4.25,4.5]$.

| ![[Uncaptioned image]](img/839cd41e1a0b85f054ac6d41f37a77af.png) normal reviewers | ![[Uncaptioned image]](img/79fbd71f60b0f148b28127eb71cce509.png) malicious reviewers |
| --- | --- |
| # | Initial | Final | $+/-$ | # | Initial | Final | $+/-$ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 3 | 5.053 $\pm$ 0.623 | 5.110 $\pm$ 0.555 | $+$0.06 | 0 | / | / | / |
| 2 | 5.066 $\pm$ 0.608 | 4.925 $\pm$ 0.552 | $-$0.14 | 1 | 3.130 $\pm$ 0.838 | 3.254 $\pm$ 0.882 | $+$0.12 |
| 1 | 5.210 $\pm$ 0.849 | 5.111 $\pm$ 0.790 | $-$0.10 | 2 | 3.503 $\pm$ 1.024 | 3.485 $\pm$ 0.994 | $-$0.02 |
| 0 | / | / | / | 3 | 3.529 $\pm$ 1.030 | 3.357 $\pm$ 0.970 | $-$0.17 |

Table 5: Reviewer ratings when varying number of ![[Uncaptioned image]](img/839cd41e1a0b85f054ac6d41f37a77af.png) *normal* reviewers are replaced by ![[Uncaptioned image]](img/ffbd4f911feec67383fc29b509a2ad9e.png) *malicious* reviewers.

![Refer to caption](img/7159f944cacfa0016c92b28eeee530d8.png)

Figure 8: Comparison of final decisions with respect to *baseline* when the author identity is known for varying ratios of papers, relative to the *baseline*. A smaller Cohen’s Kappa coefficient suggests a lower correlation with the baseline.

![Refer to caption](img/d8a4c3238aeddfb04f90035789f3333c.png)

Figure 9: Final rating distribution when we vary one reviewer in the experiment, including their commitment, intention, and knowledgeability.

| Phase | Content |
| Review | Overall rating: 5 Significance and novelty: The work puts forth a novel image representation paradigm by conceptualizing images as unorganized sets of points and using a simplified clustering algorithm for feature extraction. This approach steps away from traditional ConvNets and Vision Transformers, which rely on convolution and attention mechanisms, respectively. The proposed method, characterized as Context Clusters (CoCs), has demonstrated comparable or even slightly better performance on several benchmarks to existing architectures, offering a fresh perspective in the domain of visual representation. Reasons for acceptance: • The paper introduces a novel paradigm for visual representation that does not rely on conventional convolution or attention mechanisms. • The method achieves comparable or better results on various benchmarks relative to established architectures such as ConvNets and ViTs, suggesting its potential efficacy. • Authors provide interpretability by visualizing the clustering process, which is beneficial for understanding the model’s decisions. • The paper includes a link to the code repository, which facilitates reproducibility and further research by the community. Reasons for rejection: • Novelty and potential impact: – The work, while novel, may not offer a significantly different or superior approach to existing methods given that the improvements over state-of-the-art are not substantial or broadly demonstrated across multiple benchmarks. – The rationale behind the conceptual shift from organized to unorganized points is not convincingly tied to specific advantages in the fundamental understanding or performance gain across a wide range of tasks. • Technical soundness and completeness: – The ablation study suggests that the method’s components contribute to performance, but it is not clear if all components are necessary or if there might be a more efficient configuration. – The paper lacks in-depth comparison with other recent methods in terms of computational efficiency, memory usage, and scalability to very large datasets. • Clarity and organization of the paper: – Some parts of the methodology, especially regarding the clustering operation and region partition, could benefit from clearer explanation or visualization to aid in comprehension. – The paper would benefit from a deeper discussion on the limitations and potential failures of the proposed method, as well as comparisons to the failure modes of other methods. • Reproducibility and empirical validation: – While code is provided, the paper lacks details about hyperparameter tuning, data preprocessing, and other setup specifications that might affect reproducibility. – The empirical validation is not broad enough, largely focusing on performance metrics with less emphasis on robustness or behaviour on out-of-distribution data. |

Table 6: Example of a simulated review for the paper *Image as Set of Points* [[66](https://arxiv.org/html/2406.12708v2#bib.bib66)]. The highlighted parts in red are overlaps that match the real review.

| Phase | Content |
| Review | Suggestions for improvement: • Conduct more extensive experiments across diverse datasets, including out-of-distribution or adversarial settings, to test the robustness and generalizability of the proposed method. • Provide a more comprehensive ablation study of individual components to deduce the necessity and efficiency of each part of the Context Clusters framework. • Improve the clarity and depth of the methodology section, possibly including more diagrams or pseudocode, to aid the community in understanding and building upon the work. • Discuss potential limitations and failure modes of the context cluster approach, providing insight into the scenarios where this method may not perform optimally in comparison to other methods. |

Table 7: Example of a simulated review (continued) for the paper *Image as Set of Points* [[66](https://arxiv.org/html/2406.12708v2#bib.bib66)].

| Phase | Content |
| Rebuttal | We appreciate the constructive feedback and the reviewer’s recognition of the novelty in our conceptual shift from organized pixels to unorganized point sets and our method’s potential efficacy. Regarding the novelty and potential impact, we agree that the performance improvements over the state-of-the-art are not substantial across all benchmarks. Our primary objective is to offer a new paradigm rather than focusing solely on performance metrics. We believe that the shift to unorganized points can lead to more efficient learning in certain scenarios due to the reduction of inductive biases, which we will investigate further. We will expand on the specific advantages of this conceptual shift, exploring applications where the lack of structure in data is inherent (e.g., point clouds, non-uniform datasets). Concerning the technical soundness and completeness, we will extend our ablation studies to better understand the contribution of each component to the overall performance. We will explore alternative configurations to enhance efficiency without compromising accuracy. We will also conduct a comprehensive comparison with recent methods in terms of computational efficiency, memory usage, scalability, and adaptability to large datasets to position our methodology within the current landscape more accurately. In terms of clarity and organization, we acknowledge that the methodology section could be more accessible. We will add further explanations, visual aids, and pseudocode for the clustering operation and region partition strategy, aiming to foster better comprehension among readers. Regarding reproducibility and empirical validation, we commit to providing detailed information on hyperparameter tuning, data preprocessing, and setup specifications to ensure reproducibility. We acknowledge the importance of robustness and behavior on out-of-distribution data and will incorporate experiments in these settings to demonstrate the method’s generalizability and limitations. For the suggestions for improvement, we will: • Enhance our experimental section to cover diverse datasets, including those with out-of-distribution and adversarial examples, to assess the robustness of our method more thoroughly. • Execute a more comprehensive ablation study and refine the explanation of the individual components within our framework. • Improve methodological clarity with additional diagrams, pseudocode, and an expanded discussion on the rationale behind design choices. • Include a dedicated section on potential limitations and failure modes, comparing scenarios where our method may be less optimal than existing methods, and outlining strategies for future improvements. We hope that these improvements will address the concerns and enhance the value of our contribution to the community. |

Table 8: Simulated rebuttal for the paper *Image as Set of Points* [[66](https://arxiv.org/html/2406.12708v2#bib.bib66)].

| Phase | Content |
| Meta review | Score: 5 Summary: This paper introduces a novel approach to visual representation by conceptualizing images as unorganized sets of points. The authors propose Context Clusters (CoCs), an architecture that uses a simplified clustering algorithm for feature extraction without relying on traditional convolution or attention mechanisms. The authors have presented empirical evidence suggesting that CoCs can achieve performance comparable to or slightly better than existing ConvNets and Vision Transformers on several benchmarks. The method aims to provide a new perspective on image representation and is convolution- and attention-free, potentially offering insights into feature extraction for different domains and interpretability through visualization of the clustering process. In the rebuttal phase, the authors responded to concerns regarding the novelty and potential impact, technical soundness, and completeness. They addressed the need for a clear rationale behind the shift from organized pixels to unorganized points and agreed to expand on specific advantages and applications where this could lead to more efficient learning. The authors acknowledged that in-depth comparison with other recent methods in terms of computational efficiency, memory usage, scalability, and adaptability to large datasets was missing and committed to conducting comprehensive comparisons to position CoCs within the current research landscape accurately. The authors also responded to concerns about the method’s clarity and organization, reproducibility, empirical validation, and the general lack of a thorough comparison with state-of-the-art methods. They promised to enhance the clarity of the methodology section by including further explanations, visual aids, and pseudocode. Moreover, the authors committed to providing detailed information on hyperparameter tuning, data preprocessing, and setup specifications, as well as incorporating experiments on out-of-distribution data to demonstrate the method’s robustness and generalizability. The reviewers have acknowledged the authors’ response and appreciated the commitment to addressing the concerns. However, since there’s no updated manuscript at this stage that reflects these proposed changes, the reviewers maintained their initial ratings. Should the authors implement the improvements they outlined in their rebuttal, it is anticipated that the value of their contribution to the community could be notably enhanced. However, the current version of the paper lacks sufficient empirical validation and theoretical analysis to warrant a higher score. |

Table 9: Simulated metareview for the paper *Image as Set of Points* [[66](https://arxiv.org/html/2406.12708v2#bib.bib66)].

| Human Review |
| Summary Of The Paper: This paper proposes a new view of images that considers each image as a set of points (the pixels) and uses a clustering algorithm to extract the features from it. The goal is to investigate the way to utilize this new form of visual representations and evaluate the performance that could be achieved. To this end, the paper introduces a novel backbone network that includes the proposed Context Clusters and evaluates this model on several vision tasks as well as a point cloud data application. Strengths And Weaknesses: Strengths: • To the best of the reviewer’s knowledge, the topic of considering an image as a set of points and extracting features from it for vision tasks is original and very interesting. • The proposed method that uses the clustering algorithm as the basic build block is novel and of significance to the community. • The evaluation plan of the paper is comprehensive. It provides experiments on standard vision tasks like image classification and object detection/segmentation and applications for point cloud inputs like object classification. • The evaluation results show that the method provides improvements on various tasks over the CNN and ViT baselines (though not outperforming the state-of-the-art approach). Weaknesses: • By using the region partition mechanism, the set of points is no longer unorganized but becomes structured based on their locality. Additional experiments are required to clarify the role of the region partition. • Before applying the context clusters operation, the region partition operation, which is similar to the shifting windows in Swin, is introduced to reduce the computational cost. The authors seem to imply that the region partition trades off performance for speed. However, the locality introduced by the region partition could also bring useful inductive bias for the encoder. Therefore, additional experiments are required to answer the following questions: – If the region partition operation is removed in the clustering process, could the model achieve similar or better performance? What would the clustering map be like in this case? – It would be nice to introduce Swin as one baseline to investigate this problem. Clarity, Quality, Novelty And Reproducibility: The paper is well-written and easy to follow. The authors also provide additional explanations of some model designs in the appendix which are much appreciated. Both the topic and the proposed method are original. The general architecture is reproducible based on the model description, but additional hyper-parameters are required to reproduce the experimental results. Summary Of The Review: This paper introduces a new form of image representation that considers each image as a set of points and proposes a clustering-based architecture for feature extraction. Both the idea of “image as set of points” and the proposed architecture are interesting and novel. The experiment result also shows that the method achieves comparable performance to ConvNets and ViTs. A small concern is that the role of the region partition mechanism is unclear since good performance could actually be attributed to this design. |

Table 10: A real human-authored review for the paper *Image as Set of Points* [[66](https://arxiv.org/html/2406.12708v2#bib.bib66)]. The sections highlighted in red indicate the overlaps with the simulated review.

![Refer to caption](img/5067caef2263d1c54187f3f66fbf6d25.png)

Figure 10: Characteristics and prompts in AgentReview.