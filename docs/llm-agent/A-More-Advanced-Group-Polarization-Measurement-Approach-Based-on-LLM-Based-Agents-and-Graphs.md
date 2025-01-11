<!--yml
category: 未分类
date: 2025-01-11 11:55:30
-->

# A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs

> 来源：[https://arxiv.org/html/2411.12196/](https://arxiv.org/html/2411.12196/)

¹¹institutetext: Wuhan University, Wuhan, Hubei 430072, ChinaZixin Liu^∗ \orcidlink0009-0003-2353-7847   Ji Zhang \orcidlink0009-0000-1759-5184   Yiran Ding  \orcidlink0009-0005-8624-8545
Wuhan University
Wuhan The first two authors contribute equally.    Hubei Province    P.R.China. 430072
jizhang@whu.edu.cn, yrding@whu.edu.cn, liuzixin@whu.edu.cn

###### Abstract

Group polarization is an important research direction in social media content analysis, attracting many researchers to explore this field. Therefore, how to effectively measure group polarization has become a critical topic. Measuring group polarization on social media presents several challenges that have not yet been addressed by existing solutions. First, social media group polarization measurement involves processing vast amounts of text, which poses a significant challenge for information extraction. Second, social media texts often contain hard-to-understand content, including sarcasm, memes, and internet slang. Additionally, group polarization research focuses on holistic analysis, while texts is typically fragmented. To address these challenges, we designed a solution based on a multi-agent system and used a graph-structured Community Sentiment Network (CSN) to represent polarization states. Furthermore, we developed a metric called Community Opposition Index (COI) based on the CSN to quantify polarization. Finally, we tested our multi-agent system through a zero-shot stance detection task and achieved outstanding results. In summary, the proposed approach has significant value in terms of usability, accuracy, and interpretability.

###### Keywords:

Multi-Agent System Group Polarization LLM-Based Agent Social Media

## 1 Introduction

With the development of internet technology, social media has gained widespread popularity. GLOBAL DIGITAL REPORT [[11](https://arxiv.org/html/2411.12196v2#bib.bib11)] indicate that platforms such as Facebook, YouTube, and TikTok boast billions of users worldwide. Social media has become a key avenue for the public to express opinions and engage in discussions. Its anonymity and convenience enable users to freely express their true views, thereby shaping social media public opinion. As a result, research in this field has also flourished.

In these studies, research from the perspective of group polarization holds a significant position. The concept of group polarization was first introduced by Stoner [[37](https://arxiv.org/html/2411.12196v2#bib.bib37)], who observed that group decisions tend to be more extreme compared to individual decisions [[16](https://arxiv.org/html/2411.12196v2#bib.bib16)]. In the internet era, group polarization is broadly defined as the divergence of public opinions or stances. Building on this definition, researchers have conducted extensive and comprehensive studies on various issues related to group polarization. One of the fundamental research problems in the field of social media group polarization is its measurement. Early measurement methods based on statistical approaches [[5](https://arxiv.org/html/2411.12196v2#bib.bib5), [13](https://arxiv.org/html/2411.12196v2#bib.bib13), [15](https://arxiv.org/html/2411.12196v2#bib.bib15), [20](https://arxiv.org/html/2411.12196v2#bib.bib20), [38](https://arxiv.org/html/2411.12196v2#bib.bib38)] suffered from issues such as overly simplistic for the complexity of social media dynamics. Current mainstream methods, such as text clustering or sentiment classification [[4](https://arxiv.org/html/2411.12196v2#bib.bib4), [22](https://arxiv.org/html/2411.12196v2#bib.bib22), [35](https://arxiv.org/html/2411.12196v2#bib.bib35), [39](https://arxiv.org/html/2411.12196v2#bib.bib39)], struggle to balance efficiency and accuracy. While some researchers have made significant progress by focusing on the relationships between different viewpoints [[7](https://arxiv.org/html/2411.12196v2#bib.bib7), [17](https://arxiv.org/html/2411.12196v2#bib.bib17), [21](https://arxiv.org/html/2411.12196v2#bib.bib21), [26](https://arxiv.org/html/2411.12196v2#bib.bib26), [32](https://arxiv.org/html/2411.12196v2#bib.bib32)], these studies still fall short in understanding the deeper nuances of opinion stances and their evolution.

To address the existing issues in measuring group polarization and improve efficiency, accuracy, and interpretability, we propose a new group polarization measurement approach based on LLM-based agents and graphs. This approach draws inspiration from the stance detection task in natural language processing (NLP) and the earlier "Sentiment Thermometer" method. We use a Community Sentiment Network (CSN) represented by a graph structure to model the polarization state, where LLM-based agents are employed to construct the network. Additionally, we design polarization measurement metrics based on CSN. To validate the effectiveness of our approach, we tested the module responsible for constructing CSN on zero-shot stance detection tasks, and the results demonstrated its superiority in capturing the nuances of group polarization.

In summary, our contributions are as follows: (1) We propose a temporal Community Sentiment Network (CSN) to represent the polarization state over time. (2) We introduce LLM-based Agents for stance detection into group polarization measurement, significantly enhancing both efficiency and accuracy. (3) We propose a more robust metric based on the CSN, Community Opposition Index (COI).

## 2 Related Work

### 2.1 Opinion Polarization Measurement

As research on group polarization deepens, extensive exploration has also been conducted on the measurement of group polarization, leading to the development of a relatively comprehensive system of measurement approaches. Existing research [[5](https://arxiv.org/html/2411.12196v2#bib.bib5), [20](https://arxiv.org/html/2411.12196v2#bib.bib20)] suggests that the current mainstream group polarization measurement schemes can be primarily divided into three categories: volume-based, sentiment-based, and network-based. We will discuss the characteristics of these three measurement schemes and their shortcomings when applied to the task of measuring group polarization on social media.

Volume-based approaches primarily rely on statistical methods and were widely applied in the early research of group polarization. Early researchers collected data through surveys and experiments and used statistical analysis to obtain relevant polarization results. In current trend of exploring group polarization via social media, volume-based schemes focus more on various data metrics and employ statistical methods in research. Notable examples include Gaurav et al. [[13](https://arxiv.org/html/2411.12196v2#bib.bib13)]’s political polarization study based on the moving average aggregate probability method, Tumasjan et al. [[38](https://arxiv.org/html/2411.12196v2#bib.bib38)]’s analysis of political polarization using the LIWC tool, and Hart et al. [[15](https://arxiv.org/html/2411.12196v2#bib.bib15)]’s use of multidimensional statistical analysis to analyze polarization during COVID-19.

However, existing studies suggest that volume-based schemes have limitations in terms of capturing information and analyzing large datasets. They fail to accurately understand the opinions and sentiments conveyed in the text and generally rely on broad statistical metrics (e.g. word frequency, likes, bookmarks, etc) to gather limited information. The lack of rapid information capture and in-depth understanding makes these techniques less effective for tracking and real-time analysis of group polarization, and they also fall short in terms of precision in measuring polarization.

Compared to volume-based approaches, sentiment-based approaches place greater emphasis on the meaning and emotions conveyed in the text. Typically, sentiment-based approaches are grounded in natural language processing (NLP) and analyze social media text from the perspectives of opinions and emotions. These methods generally follow two main strategies. The first involves clustering texts based on the similarity of sentiments, such as the IOM-NN method proposed by Belcastro [[4](https://arxiv.org/html/2411.12196v2#bib.bib4)] for accurately detecting emotional information in political polarization. The second strategy leverages deep learning for direct sentiment classification, exemplified by Tyagi et al. [[39](https://arxiv.org/html/2411.12196v2#bib.bib39)]’s research on polarization driven by climate change, and the explorations by Ribeiro et al. [[35](https://arxiv.org/html/2411.12196v2#bib.bib35)] and Jiang et al. [[22](https://arxiv.org/html/2411.12196v2#bib.bib22)] on the relationship between misinformation and polarization.

Unfortunately, both strategies face notable challenges in practice. For text clustering, current clustering algorithms are relatively coarse and simplistic, making it difficult to distinguish between disruptive information (such as advertisements or neutral statements) and significant content. Moreover, they do not account for the specific relationships between subgroups or their contribution to polarization, resulting in outcomes that lack precision and interpretability. In sentiment classification, current methods often rely on binary classifications, failing to capture the intensity of emotions. This oversimplified method negatively impacts both the interpretability and accuracy of polarization measurement.

It is also worth noting that in political polarization research, a method called the "Sentiment Thermometer" has been widely adopted [[7](https://arxiv.org/html/2411.12196v2#bib.bib7), [17](https://arxiv.org/html/2411.12196v2#bib.bib17), [18](https://arxiv.org/html/2411.12196v2#bib.bib18), [26](https://arxiv.org/html/2411.12196v2#bib.bib26), [41](https://arxiv.org/html/2411.12196v2#bib.bib41)]. This approach uses surveys to gather voters’ emotional scores toward various political parties, thus enhancing the precision and interpretability of the analysis. However, this method is costly, limited by small sample sizes, and not well-suited for measuring polarization in the context of social media.

Network-based approaches represent an approach that evaluates group polarization by considering social positions and relationships among groups, focusing on emotional direction and the stance between subgroups to better measure polarization levels. Traditional social network analysis in group polarization studies explores peripheral connections around core opinions to assess subgroup positions and emotions [[8](https://arxiv.org/html/2411.12196v2#bib.bib8), [10](https://arxiv.org/html/2411.12196v2#bib.bib10), [12](https://arxiv.org/html/2411.12196v2#bib.bib12), [14](https://arxiv.org/html/2411.12196v2#bib.bib14), [33](https://arxiv.org/html/2411.12196v2#bib.bib33), [40](https://arxiv.org/html/2411.12196v2#bib.bib40)]. Some researchers have further developed this by dynamically simulating the process of group polarization to explore its evolutionary pathways [[32](https://arxiv.org/html/2411.12196v2#bib.bib32), [36](https://arxiv.org/html/2411.12196v2#bib.bib36)]. With the advancement of Graph Neural Networks (GNNs), network-based approaches have been enhanced, as demonstrated by valuable explorations from researchers such as Xiao et al. [[42](https://arxiv.org/html/2411.12196v2#bib.bib42)], Zhang et al. [[46](https://arxiv.org/html/2411.12196v2#bib.bib46)], and Jiang et al. [[21](https://arxiv.org/html/2411.12196v2#bib.bib21)], who utilized sentiment networks and GNNs in their studies. While these studies have achieved promising results in improving the scientific rigor and interpretability of group polarization research, they generally lack detailed analysis of the textual content. Moreover, social division within purely social networks does not necessarily result from group polarization, raising questions about the accuracy of some conclusions drawn from these methods.

### 2.2 LLM-Based Agents

With the introduction of OpenAI’s GPT series of large language models (LLMs) [[9](https://arxiv.org/html/2411.12196v2#bib.bib9)], numerous research fields have incorporated or examined GPT’s capabilities. In the realm of group polarization research, some researchers have also explored its applications. For instance, Lu et al. [[31](https://arxiv.org/html/2411.12196v2#bib.bib31)] used agent-based simulations to model group polarization dynamics, while Zhang et al. [[44](https://arxiv.org/html/2411.12196v2#bib.bib44)] employed LLM-Based Agents to detect stances within polarized groups. These studies have yielded promising results, demonstrating the feasibility and value of applying large language models in this field.

## 3 Method

As we mentioned in Section 1, to achieve an accurate measurement of group polarization, our proposed method consists of three parts, specifically:

1.  (i)

    A Community Sentiment Network (CSN) used to represent subgroups, the emotions between and within subgroups.

2.  (ii)

    A efficient multi-agent system for CSN construction.

3.  (iii)

    A group polarization metric based on the CSN, Community Opposition Index (COI).

The set of opinion texts will be used to identify subgroups and conduct sentiment analysis through the multi-agent system, forming a CSN. The current polarization measurement result for the time slice can then be calculated using the COI.

### 3.1 Community Sentiment Network

The Community Sentiment Network (CSN, see Fig. [1](https://arxiv.org/html/2411.12196v2#S3.F1 "Figure 1 ‣ 3.1 Community Sentiment Network ‣ 3 Method ‣ A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs")) is an extension of the "Sentiment Thermometer" method. The traditional "Sentiment Thermometer" could only be applied to two subgroups [[18](https://arxiv.org/html/2411.12196v2#bib.bib18)]. CSN extends the "Sentiment Thermometer" to a directed cyclic graph that involves emotions between multiple subgroups (see Figure 1). Let $G=(V,E)$ be a graph, where $V$ is the vertex set containing subgroups and $E$ is the set of edges representing the sentimental relationships between subgroups. Each edge $e\in E$ is defined as $(u,v,s)$ where $u,v\in V$ and $s$ is the sentiment score. It should be noted that nodes $v$ and $u$ are allowed to be the same node, meaning self-loops are permitted. The score $s$ can be either positive or negative, reflecting the positive or negative nature of the sentiment.

Unlike traditional social networks, CSN uses sentiment rather than interactions as the basis for constructing connections. Also, CSN clearly illustrates the various subgroups with different stances within the target time period and reveals the emotional states between subgroups as well as the internal cohesion of each subgroup. Therefore, compared to clustering results or social networks, CSN significantly highlights the contributions of different subgroups to polarization, providing greater interpretability of the polarization state.

![Refer to caption](img/c0143291bce915d5e098cbc5ff8bd9ad.png)

Figure 1: An example of a CSN generated by Graphviz. This CSN was generated based on comments during the Russia-Ukraine conflict.

### 3.2 A Multi-Agent System For CSN Construction

The construction of the CSN involves multiple issues, such as subgroup identification, stance detection of opinion information, and sentiment recognition. However, existing research in the field of group polarization is insufficient to provide effective solutions to these issues. Therefore, inspired by the advancements in stance detection tasks [[42](https://arxiv.org/html/2411.12196v2#bib.bib42)], we designed a multi-agent system based on LLM-based agents (see Fig. [2](https://arxiv.org/html/2411.12196v2#S3.F2 "Figure 2 ‣ 3.2 A Multi-Agent System For CSN Construction ‣ 3 Method ‣ A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs")).

![Refer to caption](img/0dff3d41723dd56cbdd25e53f3574fd0.png)

Figure 2: The structure of multi-agent system for CSN construction, containing Backround Mining Stage, Semantic Analysis Stage and Polarization Assessment Stage.

In summary, our multi-agent system is composed of three stages: the Background Mining Stage, the Semantic Analysis Stage, and the Polarization Assessment Stage. The Background Mining Stage consists of Subgroup Exploration Expert and Domain Specialist, the Semantic Analysis Stage includes Social Media Veteran, Linguistic Expert, and Sentiment Analysis Expert. The Polarization Assessment Stage comprises Polarization Assessor. The agents collaborate through a series of interactions to provide accurate subgroup division results and reliable sentiment scores (see Alg.  [1](https://arxiv.org/html/2411.12196v2#alg1 "Algorithm 1 ‣ 3.2 A Multi-Agent System For CSN Construction ‣ 3 Method ‣ A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs")). We will provide a detailed explanation of each stage and the functions of the respective agents in the following paragraphs.

Algorithm 1 Triplets Construction Algorithm

1:Input: Comments data within a specified time range2:Output: Set of triplets as analysis results3:procedure AnalyzeComments($comments$)4:     $bg\leftarrow$ DomainSpecialist($comments$) $\triangleright$ bg for background5:     $sg\leftarrow\emptyset$ $\triangleright$ sg for subgroups6:     $uncertainComments\leftarrow[]$7:     SubgroupExplorationExpert($bg$)8:     for each $comment$ in $comments$ do9:         $grp\leftarrow$ SubgroupExplorationExpert($comment$)10:         if not $grp$ then11:              $uncertainComments.add(comment)$12:              if length($uncertainComments$) reaches threshold then13:                  $grp\leftarrow$ HumanExpertHelp($uncertainComments$)14:                  $sg.add(grp)$15:                  $uncertainComments\leftarrow[]$16:              end if17:         else18:              $sg.add(grp)$19:         end if20:     end for21:     Initialize Experts:22:     SocialMediaVeteran($sg,bg$)23:     LinguisticExpert($bg$)24:     SentimentAnalysisExpert($sg$)25:     PolarizationAssessment($sg,bg$)26:     $triplets\leftarrow\emptyset$27:     for each $comment$ in $comments$ do28:         $out1\leftarrow$ SocialMediaVeteran($comment$)29:         $out2\leftarrow$ LinguisticExpert($comment$)30:         $out3\leftarrow$ SentimentAnalysisExpert($comment,out1,out2$)31:         $out4\leftarrow$ PolarizationAssessment($comment,out3$)32:         if $out4.group$ then33:              $triplet\leftarrow(out4.group,out4.score,out4.targetGroup)$34:         else35:              $triplet\leftarrow(\text{null},out4.score,out4.targetGroup)$36:         end if37:         $triplets.add(triplet)$38:     end for39:     return $triplets$40:end procedure

#### 3.2.1 Background Mining Stage

For the construction of the CSN, the first issue we need to address is how the multi-agent system understands the overall event within the context of the event. To solve this, we propose the Background Mining Stage, which uses textual information to understand the event and identify potential subgroups. Its functionality can be described as follows:

Input: All the comment texts related to the target topic (sampled if necessary).

Output: A description of the event’s background, all potential subgroups present in the topic, and detailed descriptions of each subgroup.

##### Domain Specialist

Domain Specialist is primarily responsible for extracting the event background described in the comment texts. Their main tasks include exploring the core event of the topic and key stakeholders. The Domain Specialist develops a comprehensive description of the event’s timeline and related parties, providing this background information to the Subgroup Exploration Expert and other subsequent stages of the process.

##### Subgroup Exploration Expert

The task of the Subgroup Exploration Expert is to use the background information provided by the Domain Specialist along with the source texts to identify potential subgroups involved in the event and summarize the possible speaking patterns of each subgroup’s members. This requires the expert to explore the organizations, stances, religions, and other social identities referenced in the texts and form subgroups based on the similarity of their expressions. It is important to note that if there is a significant amount of unclassifiable content, the expert is permitted to consult human experts for clarification.

#### 3.2.2 Semantic Analysis Stage

Building on the background information, Semantic Analysis Stage needs to address the primary challenge of accurately interpreting the emotions conveyed in the texts, especially when slang, homophones, sarcasm, and other nuanced expressions are present. To achieve better results on this complex task, we design the system with the specific attributes of social media in mind. Its functionality can be described as follows:

Input: Comment texts under the target topic and results from Background Mining Stage.

Output: Sentiment analysis result.

##### Social Media Veteran

The Social Media Veteran is one of the key agents responsible for semantic understanding of social media content. Its main role is to explore the patterns and characteristics of language expression on social media platforms. The agent needs to interpret the actual meanings of hashtags, internet slang, memes, and other unique forms of expression commonly used on social media. After this analysis, the Social Media Veteran passes the comprehended information to the Sentiment Analysis Expert for further processing.

##### Linguistic Expert

The Linguistic Expert is another key agent responsible for the semantic understanding of social media content. Unlike the Social Media Veteran, the Linguistic Expert focuses on analyzing the text from a linguistic perspective, examining aspects such as grammatical structure, rhetorical devices, word choice, and tense. The analysis results are also passed to the Sentiment Analysis Expert. It is important to note that the Linguistic Expert’s analysis is not conducted in isolation; the background information provided by the Domain Expert supports and informs the linguistic analysis.

##### Sentiment Analysis Expert

The Sentiment Analysis Expert is the agent responsible for synthesizing various inputs to determine the final sentiment and its direction. It combines the emotional language present in the text with the semantic analysis results provided by other agents, such as the Social Media Veteran and the Linguistic Expert, to derive the sentiment of the text. Additionally, it utilizes the subgroup information provided by the subgroup exploration expert to identify the potential target of the sentiment. The results of this sentiment analysis will serve as the output of the sentiment analysis stage and will be passed to the next stage.

#### 3.2.3 Polarization Assessment Stage

The primary task of the Polarization Assessment Stage is to utilize the information from the Background Mining Stage and the Semantic Analysis Stage to generate CSN in the form of triplets. Its functionality can be described as follows:

Input: All information from the Background Mining Stage (including potential subgroups and event background), each text from the target topic, and their sentiment analysis results.

Output: Sentiment expressed in the form of triplets for each comment: (personal stance, sentiment score, target subgroup).

##### Polarization Assessor

The Polarization Assessor is the core agent of the Polarization Assessment Stage. It is responsible for analyzing the author’s stance, sentiment score, and the target group of the sentiment for each comment, based on the information passed from the other stages. The Polarization Assessor integrates this information into triplets. With the multi-dimensional and in-depth analysis provided by the other stages, the Polarization Assessor can make precise judgments and give credible sentiment score.

##### CSN Construction

To construct the final Community Sentiment Network (CSN) based on the sentiment triplets from the existing comments, we designed the relevant algorithm (see Alg. [2](https://arxiv.org/html/2411.12196v2#alg2 "Algorithm 2 ‣ CSN Construction ‣ 3.2.3 Polarization Assessment Stage ‣ 3.2 A Multi-Agent System For CSN Construction ‣ 3 Method ‣ A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs"), Table [1](https://arxiv.org/html/2411.12196v2#S3.T1 "Table 1 ‣ CSN Construction ‣ 3.2.3 Polarization Assessment Stage ‣ 3.2 A Multi-Agent System For CSN Construction ‣ 3 Method ‣ A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs") explains some variables). We use an adjacency matrix, $adjMatrix$, to represent the CSN, where $adjMatrix[i][j]$ represents the sentiment score of subgroup $i$ towards subgroup $j$. We first use all the triplets to build an initial network and then merge the nodes that belong to the same subgroup. During the merging process, we use the number of likes on the comments as a weighting factor, applying a weighted calculation to the sentiment scores between subgroups involved in the sentiment. This results in a total sentiment score between the subgroups. Since not all triplets have a personal stance, we complete them by approximating the occurrence frequency of all known personal stances as probabilities and use these probabilities to fill in the incomplete triplets. These operations results in the final CSN.

Table 1: The explanation of some variables in Alg. [2](https://arxiv.org/html/2411.12196v2#alg2 "Algorithm 2 ‣ CSN Construction ‣ 3.2.3 Polarization Assessment Stage ‣ 3.2 A Multi-Agent System For CSN Construction ‣ 3 Method ‣ A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs").

| Variable | Explanation |
| --- | --- |
| $weightSum$ | triplets’ weight |
| $countMatrix[i][j]$ | number of triplets whose person stance is $i$ and target subgroup is $j$ |
| $incompleteTriplets$ | variable for storing incomplete triplets |
| $commentCount$ | number of comments in every subgroup |

Algorithm 2 Construction of Community Sentiment Network

1:Initialize:2:Initialize 10x10 matrices $adjMatrix$, $weightSum$ and $countMatrix$ to zero3:Initialize 1x10 vector $commentCount$ to zero4:$incompleteTriplets\leftarrow[]$5:Process Complete Triplets:6:for each $triplet$ in $triplets$ do7:     if $triplet.group$ is not null then8:         $src\leftarrow triplet.group$9:         $tgt\leftarrow triplet.targetGroup$10:         $weightedScore\leftarrow triplet.score\times\max(triplet.likes,1)$11:         $adjMatrix[src][tgt]\leftarrow adjMatrix[src][tgt]+weightedScore$12:         $weightSum[src][tgt]\leftarrow weightSum[src][tgt]+\max(triplet.likes,1)$13:         $countMatrix[src][tgt]\leftarrow countMatrix[src][tgt]+1$14:         $commentCount[src]\leftarrow commentCount[src]+1$15:     else16:         $incompleteTriplets.add(triplet)$17:     end if18:end for19:Process Incomplete Triplets:20:for each $triplet$ in $incompleteTriplets$ do21:     $tgt\leftarrow triplet.targetGroup$22:     $probabilities\leftarrow[]$23:     for $i\leftarrow 1$ to $10$ do24:         $probabilities[i]\leftarrow countMatrix[i][tgt]/\sum_{j=1}^{10}countMatrix[j][tgt]$25:     end for26:     $src\leftarrow$ sample a group based on $probabilities$27:     $weightedScore\leftarrow triplet.score\times\max(triplet.likes,1)$28:     $adjMatrix[src][tgt]\leftarrow adjMatrix[src][tgt]+weightedScore$29:     $weightSum[src][tgt]\leftarrow weightSum[src][tgt]+\max(triplet.likes,1)$30:     $countMatrix[src][tgt]\leftarrow countMatrix[src][tgt]+1$31:     $commentCount[src]\leftarrow commentCount[src]+1$32:end for33:Compute Averages:34:for $i\leftarrow 1$ to $10$ do35:     for $j\leftarrow 1$ to $10$ do36:         if $weightSum[i][j]>0$ then37:              $adjMatrix[i][j]\leftarrow adjMatrix[i][j]/weightSum[i][j]$38:         end if39:     end for40:end for41:Output: Draw the group affect network using $adjMatrix$

### 3.3 Community Opposition Index (COI)

We have designed a dedicated group polarization metric for CSN to derive a comparable and interpretable polarization index from its complex graph structure. In previous research on sentiment-based polarization measurement, a widely accepted viewpoint is that the stronger the internal cohesion within subgroups and the greater the hostility between subgroups, the higher the level of polarization [[18](https://arxiv.org/html/2411.12196v2#bib.bib18), [19](https://arxiv.org/html/2411.12196v2#bib.bib19), [26](https://arxiv.org/html/2411.12196v2#bib.bib26), [41](https://arxiv.org/html/2411.12196v2#bib.bib41), [43](https://arxiv.org/html/2411.12196v2#bib.bib43)]. The "Sentiment Thermometer" was developed based on this perspective, and its approach of calculating sentiment temperature differences has gained widespread recognition and practical use [[7](https://arxiv.org/html/2411.12196v2#bib.bib7), [17](https://arxiv.org/html/2411.12196v2#bib.bib17), [18](https://arxiv.org/html/2411.12196v2#bib.bib18), [26](https://arxiv.org/html/2411.12196v2#bib.bib26), [41](https://arxiv.org/html/2411.12196v2#bib.bib41)]. However, as we mentioned earlier, this method is only applicable when there are exactly two subgroups involved in group polarization. Therefore, we extended this calculation method to the multi-group domain and proposed the Community Opposition Index (COI).

Firstly, we calculate the sentiment score of a subgroup towards the other subgroups:

|  | $(-e_{ij})\cdot 1_{e_{ij}\leq 0}$ |  | (1) |

Here, $e_{ij}$ represents the sentiment score of subgroup $i$ towards subgroup $j$. $1_{e_{ij}\leq 0}$ means that we consider friendly subgroups as not contributing to the overall group polarization.

Subsequently, we sum the sentiment scores of subgroup $i$ towards all other related subgroups and take into account the internal cohesion within each subgroup.Therefore, we get the polarization score of subgroup $i$. Here, $t_{i}$ represents the internal cohesion of the subgroup $i$.

|  | $t_{i}\cdot\sum_{j}{(-e_{ij})\cdot 1_{e_{ij}\leq 0}}$ |  | (2) |

Finally, we weight the overall sentiment score of each subgroup according to its size and calculate the final polarization score.

|  | $\sum_{i}({\frac{n_{i}}{N}\cdot t_{i}\cdot\sum_{j}{(-e_{ij})\cdot 1_{e_{ij}\leq 0% }}})$ |  | (3) |

Here $N$ represents the total number of comments on the target topic and $n_{i}$ represents the number of comments of subgroup $i$ within this topic.

It is important to emphasize that, since our metric is a relative indicator, it can avoid interference in the polarization measurement results caused by differences in the number of comments. Additionally, this metric, by focusing on both the internal and external sentiments of subgroups, offers better interpretability.

## 4 Zero-Shot Experiments

Our experiments will be conducted on the multi-agent system. Since there is no established benchmark in the field of group polarization research, we have chosen to test the system using stance detection tasks, which share a similar nature. We describe the specific setup of our experiments as follows.

### 4.1 Datasets

Based on existing work in the field of stance detection [[3](https://arxiv.org/html/2411.12196v2#bib.bib3), [27](https://arxiv.org/html/2411.12196v2#bib.bib27), [28](https://arxiv.org/html/2411.12196v2#bib.bib28)], we will conduct our experiments on the following three datasets:

SEM16 [[34](https://arxiv.org/html/2411.12196v2#bib.bib34)]. This dataset includes six different targets selected from various domains, namely Donald Trump (DT), Hillary Clinton (HC), Feminist Movement (FM), Legalization of Abortion (LA), Atheism (A), and Climate Change is a Real Concern (CC). It includes three types of stances: Favor, Against, and None.

P-Stance [[30](https://arxiv.org/html/2411.12196v2#bib.bib30)]. This dataset includes six different targets selected from political domains, namely Donald Trump (Trump), Joe Biden (Biden), Bernie Sanders (Sanders). It includes three types of stances: Favor and Against.

VAST [[1](https://arxiv.org/html/2411.12196v2#bib.bib1)]. This dataset includes large number of varying targets, and it includes three types of stances: Pro, Con and Neutral.

The statistics of our utilized datasets are shown in Table [2](https://arxiv.org/html/2411.12196v2#S4.T2 "Table 2 ‣ 4.1 Datasets ‣ 4 Zero-Shot Experiments ‣ A More Advanced Group Polarization Measurement Approach Based on LLM-Based Agents and Graphs"). Since our model’s use case is almost zero-shot, we will utilize these three datasets to conduct zero-shot stance detection. We will strictly adhere to the licensing requirements of the respective datasets.

To better evaluate the model’s performance, we selected appropriate metrics based on existing literature [[2](https://arxiv.org/html/2411.12196v2#bib.bib2), [25](https://arxiv.org/html/2411.12196v2#bib.bib25), [29](https://arxiv.org/html/2411.12196v2#bib.bib29)]. For the SEM16 and P-Stance datasets, we chose $F_{avg}$, which represents the average of F1 scores for Favor and Against. For the VAST dataset, we opted for Macro-F1 as the evaluation metric.

Table 2: Statistics of our utilized datasets.

| Dataset | Target | Pro | Con | Neutral |
| SEM16 | DT | 148 (20.9%) | 299 (42.3%) | 260 (36.8%) |
| HC | 163 (16.6%) | 565 (57.4%) | 256 (26.0%) |
| FM | 268 (28.2%) | 511 (53.8%) | 170 (17.9%) |
| LA | 167 (17.9%) | 544 (58.3%) | 222 (23.8%) |
| A | 124 (16.9%) | 464 (63.3%) | 145 (19.8%) |
|  | CC | 335 (59.4%) | 26 (4.6%) | 203 (36.0%) |
| P-Stance | Biden | 3217 (44.1%) | 4079 (55.9%) | - |
| Sanders | 3551 (56.1%) | 2774 (43.9%) | - |
| Trump | 3663 (46.1%) | 4290 (53.9%) | - |
| VAST | - | 6952 (37.5%) | 7297 (39.3%) | 4296 (23.2%) |

### 4.2 Model Adjustment

Since the primary purpose of our designed multi-agent system is to construct the CSN, we need to adjust the model for the experiments. We removed the Subgroup Exploration Expert and eliminated the subgroup exploration process. Instead, we input texts with predefined target groups into the remaining five agents and obtained the output from the Polarization Assessor. The adjusted model’s output only includes the sentiment score and target group, making it suitable for performing stance detection tasks.

Regarding the specific details of the model configuration, we use multiple GPT-3.5 Turbo models provided by OpenAI to serve as agents in the Background Mining Stage and Semantic Analysis Stage, while GPT-4 is employed as the Polarization Assessor. This selection was primarily based on a balance between cost and the desired final performance.

### 4.3 Comparison Methods

We compare our method with various methods in zero-shot stance detection. This includes adversarial learning method: TOAD [[2](https://arxiv.org/html/2411.12196v2#bib.bib2)], contrastive learning methods: PT-HCL [[28](https://arxiv.org/html/2411.12196v2#bib.bib28)], Bert-based techniques: TGANet [[1](https://arxiv.org/html/2411.12196v2#bib.bib1)] and Bert-GCN [[29](https://arxiv.org/html/2411.12196v2#bib.bib29)], LLM-based techniques: GPT-3.5 Turbo [[44](https://arxiv.org/html/2411.12196v2#bib.bib44)], GPT-3.5 Turbo+Chainof-thought (COT) [[45](https://arxiv.org/html/2411.12196v2#bib.bib45)] and COLA [[25](https://arxiv.org/html/2411.12196v2#bib.bib25)].

### 4.4 Zero-Shot Stance Detection Results

In Table 3, we present the performance of our method on the zero-shot stance detection task, along with a comparison to other baselines. The results demonstrate that our method exhibits excellent performance in this task, with a performance improvement of 8.4% over the current best result on the VAST dataset. Although our method did not achieve state-of-the-art (SOTA) results across all metrics, its ability to come close to or surpass current SOTA algorithms indicates its significant value when applied to group polarization research.

Table 3: Comparison of our method and baselines in zero-shot stance detection task, all values are percentages. Bold refer to the best performance. * denotes our method improves the best baseline at p < 0.05 with paired t-test.

| Model | SEM16 | P-Stance | VAST |
| --- | --- | --- | --- |
| DT | HC | FM | LA | A | CC | Trump | Biden | Sanders | All |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| TOAD | 49.5 | 51.2 | 54.1 | 46.2 | 46.1 | 30.9 | 53.0 | 68.4 | 62.9 | 41.0 |
| TGA Net | 40.7 | 49.3 | 46.6 | 45.2 | 52.7 | 36.6 | - | - | - | 65.7 |
| BERT-GCN | 42.3 | 50.0 | 44.3 | 44.2 | 53.6 | 35.5 | - | - | - | 68.6 |
| PT-HCL | 50.1 | 54.5 | 54.6 | 50.9 | 56.5 | 38.9 | - | - | - | 71.6 |
| GPT-3.5 | 62.5 | 68.7 | 44.7 | 51.5 | 9.1 | 31.1 | 62.9 | 80.0 | 71.5 | 62.3 |
| GPT-3.5+COT | 63.3 | 70.9 | 47.7 | 53.4 | 13.3 | 34.0 | 63.9 | 81.2 | 73.2 | 68.9 |
| COLA | 68.5 | 81.7 | 63,4 | 71.0 | 70.8 | 65.5 | 86.6 | 84.0 | 79.7 | 73.0 |
| Ours | 74.4* | 81.9 | 70.3* | 75.8* | 76.9* | 70.7 | 87.9 | 83.2 | 86.2* | 81.4* |

## 5 Conclusion

In this paper, we discussed the shortcomings of current group polarization measurement approaches and proposed our multi-agent and graph-based measurement scheme. Our solution innovatively introduces a large language model-based multi-agent system into the measurement of group polarization and utilizes the Community Sentiment Network (CSN) to represent the polarization state. Additionally, we provided a metric (Community Opposition Index) for calculating polarization levels using the CSN, allowing the polarization state to be quantified. Finally, we tested our multi-agent system through a zero-shot stance detection task, and the results demonstrated its usability and value.

## References

*   [1] Allaway, E., McKeown, K.: Zero-shot stance detection: A dataset and model using generalized topic representations. arXiv preprint arXiv:2010.03640 (2020)
*   [2] Allaway, E., Srikanth, M., McKeown, K.: Adversarial learning for zero-shot stance detection on social media. arXiv preprint arXiv:2105.06603 (2021).
*   [3] Augenstein, I., Rocktäschel, T., Vlachos, A., Bontcheva, K.: Stance detection with bidirectional conditional encoding. arXiv preprint arXiv:1606.05464 (2016)
*   [4] Belcastro, L., Cantini, R., Marozzo, F., Talia, D., Trunfio, P.: Learning political polarization on social media using neural networks. IEEE Access 8, 47177–47187 (2020)
*   [5] Bilal, M., Gani, A., Marjani, M., Malik, N.: Predicting Elections: Social Media Data and Techniques. In: 2019 International Conference on Engineering and Emerging Technologies (ICEET), pp. 1-6\. Springer, 2019\. \doi10.1109/CEET1.2019.8711854
*   [6] Bomsdorf, E., Otto, C.: A new approach to the measurement of polarization for grouped data. AStA Advances in Statistical Analysis 91, 181–196 (2007)
*   [7] Boxell, L., Conway, J., Druckman, J.N., Gentzkow, M.: Affective polarization did not increase during the coronavirus pandemic. National Bureau of Economic Research, (2020)
*   [8] Bravo, R.B., Del Valle, M.E., Gavidia, À.R.: A multilayered analysis of polarization and leaderships in the Catalan Parliamentarians’ Twitter Network. In: 2015 Fifteenth International Conference on Advances in ICT for Emerging Regions (ICTer), pp. 200–206\. IEEE (2015). \doi10.1109/ICTER.2015.7377689
*   [9] Brown, T.B., Mann, B., Ryder, N., Subbiah, M., Kaplan, J., Dhariwal, P., Neelakantan, A., Shyam, P., Sastry, G., Askell, A., Agarwal, S., Herbert-Voss, A., Krueger, G., Henighan, T., Child, R., Ramesh, A., Ziegler, D.M., Wu, J., Winter, C., Hesse, C., Chen, M., Sigler, E., Litwin, M., Gray, S., Chess, B., Clark, J., Berner, C., McCandlish, S., Radford, A., Sutskever, I., Amodei, D.: Language models are few-shot learners. arXiv preprint arXiv:2005.14165 (2020)
*   [10] Conover, M., Ratkiewicz, J., Francisco, M., Gonçalves, B., Menczer, F., Flammini, A.: Political polarization on Twitter. In: Proceedings of the International AAAI Conference on Web and Social Media, vol. 5, no. 1, pp. 89–96 (2011). \doi10.1609/icwsm.v5i1.14126
*   [11] DIGITAL 2024: GLOBAL OVERVIEW REPORT, [https://datareportal.com/global-digital-overview](https://datareportal.com/global-digital-overview), last accessed 2024/09/14
*   [12] Garcia, D., Abisheva, A., Schweighofer, S., Serdült, U., Schweitzer, F.: Ideological and temporal components of network polarization in online political participatory media. Policy & Internet 7(1), 46–79 (2015)
*   [13] Gaurav, M., Srivastava, A., Kumar, A., Miller, S.: Leveraging candidate popularity on Twitter to predict election outcome. In: Proceedings of the 7th Workshop on Social Network Mining and Analysis, article no. 7, pp. 1-8\. Association for Computing Machinery, New York (2013). \doi10.1145/2501025.2501038
*   [14] Guerra, P., Meira Jr, W., Cardie, C., Kleinberg, R.: A measure of polarization on social media networks based on community boundaries. In: Proceedings of the International AAAI Conference on Web and Social Media, vol. 7, no. 1, pp. 215–224 (2013). \doi10.1609/icwsm.v7i1.14421
*   [15] Hart, P.S., Chinn, S., Soroka, S.: Politicization and polarization in COVID-19 news coverage. Science Communication 42(5), 679–697 (2020)
*   [16] Isenberg, D.J.: Group polarization: A critical review and meta-analysis. Journal of Personality and Social Psychology 50(6), 1141–1151 (1986)
*   [17] Iyengar, S., Lelkes, Y., Levendusky, M., Malhotra, N., Westwood, S.J.: The origins and consequences of affective polarization in the United States. Annual Review of Political Science 22(1), 129–146 (2019)
*   [18] Iyengar, S., Sood, G., Lelkes, Y.: Affect, not ideology: A social identity perspective on polarization. Public Opinion Quarterly 76(3), 405–431 (2012)
*   [19] Iyengar, S., Westwood, S.J.: Fear and loathing across party lines: New evidence on group polarization. American Journal of Political Science 59(3), 690–707 (2015)
*   [20] Jaidka, K., Ahmed, S., Skoric, M., Hilbert, M.: Predicting elections from social media: a three-country, three-method comparative study. Asian Journal of Communication 29(3), 252–-273 (2018)
*   [21] Jiang, J., Ren, X., Ferrara, E.: Retweet-BERT: Political leaning detection using language features and information diffusion on social networks. In: Proceedings of the International AAAI Conference on Web and Social Media, vol. 17, pp. 459–469 (2023). \doi10.1609/icwsm.v17i1.22160
*   [22] Jiang, S., Wilson, C.: Linguistic signals under misinformation and fact-checking: Evidence from user comments on social media. Proceedings of the ACM on Human-Computer Interaction 2(CSCW), 1–23 (2018).
*   [23] Knox, R.E., Inkster, J.A.: Postdecision dissonance at post time. Journal of Personality and Social Psychology 8(4, Pt. 1), 319 (1968)
*   [24] Kogan, N., Wallach, M.A.: Risky-shift phenomenon in small decision-making groups: A test of the information-exchange hypothesis. Journal of Experimental Social Psychology 3(1), 75–84 (1967)
*   [25] Lan, X., Gao, C., Jin, D., et al.: Stance detection with collaborative role-infused LLM-based agents. arXiv preprint arXiv:2310.10467 (2023)
*   [26] Lelkes, Y., Westwood, S.J.: The limits of partisan prejudice. The Journal of Politics 79(2), 485–501 (2017)
*   [27] Li, A., Liang, B., Zhao, J., Zhang, B., Yang, M., Xu, R.: Stance detection on social media with background knowledge. In: Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing, pp. 15703–15717 (2023). \doi10.18653/v1/2023.emnlp-main.972
*   [28] Liang, B., Chen, Z., Gui, L., He, Y., Yang, M., Xu, R.: Zero-shot stance detection via contrastive learning. In: Proceedings of the ACM Web Conference 2022, pp. 2738–2747 (2022). \doi10.1145/3485447.3511994
*   [29] Liu, R., Lin, Z., Tan, Y., et al.: Enhancing zero-shot and few-shot stance detection with commonsense knowledge graph. In: Findings of the Association for Computational Linguistics: ACL-IJCNLP 2021, pp. 3152–3157 (2021). \doi10.18653/v1/2021.findings-acl.278
*   [30] Li, Y., Sosea, T., Sawant, A., Nair, A.J., Inkpen, D., Caragea, C.: P-stance: A large dataset for stance detection in political domain. In: Findings of the Association for Computational Linguistics: ACL-IJCNLP 2021, pp. 2355–2365 (2021). \doi10.18653/v1/2021.findings-acl.208
*   [31] Lu, H.C., Lee, H.W.: Agents of Discord: Modeling the Impact of Political Bots on Opinion Polarization in Social Networks. Social Science Computer Review (2024). \doi10.1177/08944393241270382
*   [32] Maia, H.P., Ferreira, S.C., Martins, M.L.: Controversy-seeking fuels rumor-telling activity in polarized opinion networks. Chaos, Solitons & Fractals 169, 113287 (2023)
*   [33] Medaglia, R., Zhu, D.: Public deliberation on government-managed social media: A study on Weibo users in China. Government Information Quarterly 34(3), 533–544 (2017)
*   [34] Mohammad, S., Kiritchenko, S., Sobhani, P., Zhu, X., Cherry, C.: SemEval-2016 Task 6: Detecting stance in tweets. In: Proceedings of the 10th International Workshop on Semantic Evaluation (SemEval-2016), pp. 31–41 (2016). \doi10.18653/v1/S16-1003
*   [35] Ribeiro, M.H., Calais, P.H., Almeida, V.A., Meira Jr, W.: "Everything I disagree with is #FakeNews": Correlating political polarization and spread of misinformation. arXiv preprint arXiv:1706.05924 (2017).
*   [36] Santos, F.P., Lelkes, Y., Levin, S.A.: Link recommendation algorithms and dynamics of polarization in online social networks. Proceedings of the National Academy of Sciences 118(50), e2102141118 (2021). \doi10.1073/pnas.2102141118
*   [37] Stoner, J.A.: A comparison of individual and group decisions involving risk (Doctoral dissertation, Massachusetts Institute of Technology) (1961)
*   [38] Tumasjan, A., Sprenger, T., Sandner, P., Welpe, I.: Predicting elections with Twitter: What 140 characters reveal about political sentiment. In: Proceedings of the International AAAI Conference on Web and Social Media, vol. 4, no. 1, pp. 178–185 (2010). \doi10.1609/icwsm.v4i1.14009
*   [39] Tyagi, A., Uyheng, J., Carley, K.M.: Affective Polarization in Online Climate Change Discourse on Twitter. In: 2020 IEEE/ACM International Conference on Advances in Social Networks Analysis and Mining (ASONAM), pp. 443–447\. IEEE, The Hague (2020). \doi10.1109/ASONAM49781.2020.9381419
*   [40] Vicario, M.D., Gaito, S., Quattrociocchi, W., Zignani, M., Zollo, F.: News Consumption during the Italian Referendum: A Cross-Platform Analysis on Facebook and Twitter. In: 2017 IEEE International Conference on Data Science and Advanced Analytics (DSAA), pp. 648–657\. IEEE, Tokyo (2017). \doi10.1109/DSAA.2017.33
*   [41] Wakefield, R.L., Wakefield, K.: The antecedents and consequences of intergroup affective polarization on social media. Information Systems Journal 33(3), 640–668 (2023)
*   [42] Xiao, Z., Song, W., Xu, H., Ren, Z., Sun, Y.: TIMME: Twitter ideology-detection via multi-task multi-relational embedding. In: Proceedings of the 26th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining, pp. 2258–2268 (2020). \doi10.1145/3394486.3403275
*   [43] Yarchi, M., Baden, C., Kligler-Vilenchik, N.: Political polarization on the digital sphere: A cross-platform, over-time analysis of interactional, positional, and affective polarization on social media. Political Communication 38(1-2), 98–139 (2021)
*   [44] Zhang, B., Ding, D., Jing, L.: How would stance detection techniques evolve after the launch of ChatGPT? arXiv preprint arXiv:2212.14548 (2022)
*   [45] Zhang, B., Fu, X., Ding, D., Huang, H., Li, Y., Jing, L.: Investigating chain-of-thought with ChatGPT for stance detection on social media. arXiv preprint arXiv:2304.03087 (2023)
*   [46] Zhang, C., Song, D., Huang, C., Swami, A., Chawla, N.V.: Heterogeneous graph neural network. In: Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining, pp. 793–803 (2019). \doi10.1145/3292500.3330961