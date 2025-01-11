<!--yml
category: 未分类
date: 2025-01-11 12:02:43
-->

# MiniFed : Integrating LLM-based Agentic-Workflow for Simulating FOMC Meeting

> 来源：[https://arxiv.org/html/2410.18012/](https://arxiv.org/html/2410.18012/)

Sungil Seok^($1$), Shuide Wen^($1$), Qiyuan Yang^($2$), Juan Feng^($3*$), Wenming Yang^($1*$) ^($1$) Shenzhen International Graduate School, Tsinghua University, Shenzhen, China ^($2$) SKEMA Business School, Nanjing Audit University, Nanjing, China ^($3$) School of Economics and Management, Tsinghua University, Shenzhen, China shi-cy22@mails.tsinghua.edu.cn, yangqiyuan@stu.nau.edu.cn,
{wenshuide, yang.wenming}@sz.tsinghua.edu.cn, fengjuan@sem.tsinghua.edu.cn

###### Abstract

The Federal Funds rate in the United States plays a significant role in both domestic and international financial markets. However, research has predominantly focused on the effects of adjustments to the Federal Funds rate rather than on the decision-making process itself. Recent advancements in large language models (LLMs) offer a potential method for reconstructing the original FOMC meetings, which are responsible for setting the Federal Funds rate. In this paper, we propose a five-stage FOMC meeting simulation framework, MiniFed, which employs LLM agents to simulate real-world FOMC meeting members and optimize the FOMC structure. This framework effectively revitalizes the FOMC meeting process and facilitates projections of the Federal Funds rate. Experimental results demonstrate that our proposed MiniFed framework achieves both high accuracy in Federal Funds rate projections and behavioral alignment with the agents’ real-world counterparts. Given that few studies have focused on employing LLM agents to simulate large-scale real-world conferences, our work can serve as a benchmark for future developments.

###### Index Terms:

Large Language Models, Multi-Agent System, Real World Simulation

## I Introduction

Despite a slight decline in its share until 2021, the US dollar continues to reign as the most crucial currency in foreign exchange reserves (FXR), reinforcing its pivotal role in global finance[[1](https://arxiv.org/html/2410.18012v2#bib.bib1)]. This prominence has generated substantial interest in the adjustments of US interest rates. While extensive research exists on the effects of these adjustments on foreign economies[[2](https://arxiv.org/html/2410.18012v2#bib.bib2)], international capital flows[[3](https://arxiv.org/html/2410.18012v2#bib.bib3)], stock markets[[4](https://arxiv.org/html/2410.18012v2#bib.bib4)], and even the cryptocurrency market[[5](https://arxiv.org/html/2410.18012v2#bib.bib5)], the process by which US monetary policies are formulated remains less examined.

The intricacies of the policy-making process play a significant role in the scarcity of research in this domain. The Federal Open Market Committee (FOMC) is tasked with setting the Federal Funds rate, which directly impacts US interest rates. While the FOMC discloses all pertinent materials and the complete discussion process for determining the target interest rate range five years post-meeting, effectively and accurately reconstructing this entire process continues to pose a significant challenge.

However, the breakthroughs in AI technologies, especially in the area of generative models, make addressing this challenge feasible. Generative models, particularly Large Language Models (LLMs) such as ChatGPT¹¹1https://openai.com/index/chatgpt/, have proven to be exceptionally effective as agents capable of producing realistic simulations of human behavior. These models adapt dynamically to their changing experiences and environment under specific agent architectures[[6](https://arxiv.org/html/2410.18012v2#bib.bib6)]. A growing body of research explores the potential of LLM-based multi-agent systems for real-world scenario applications, including simulations of the US Supreme Court[[7](https://arxiv.org/html/2410.18012v2#bib.bib7)], classroom environments[[8](https://arxiv.org/html/2410.18012v2#bib.bib8)], everyday life[[6](https://arxiv.org/html/2410.18012v2#bib.bib6)], and market research participant role-playing[[9](https://arxiv.org/html/2410.18012v2#bib.bib9)]. This extensive research underscores the capabilities of LLM-based multi-agent systems and has motivated us to develop and utilize our own architecture to accurately reconstruct the decision-making process for US interest rate adjustments.

In this paper, we introduce MiniFed, our innovatively designed LLM-based multi-agent system that leverages the capabilities of the newly developed ChatGPT-4o mini model in conjunction with our custom FOMC simulation architecture. This system is designed to accurately replicate each FOMC meeting from 2018\. Experimental results indicate that our architecture excels in both predictive accuracy, alignment and content comprehension and generation.

The paper is structured as follows: we will begin with a review of related works, followed by a detailed introduction of our MiniFed architecture, the experimental setup, the results of our experiments and further reflections and discussions.

In summary, the paper makes the following contributions:

*   •

    We reconstructed the FOMC meeting participants by modeling their socio-demographic and personality attributes to align with their real-world counterparts, thereby demonstrating the accuracy of our agent pre-definition method.

*   •

    We proposed our five-stage MiniFed architecture, which captures the most important aspects of the FOMC meeting, effectively facilitating communication among the participant agents and eliciting the final monetary policies.

*   •

    Given that few prior studies have focused on reconstructing large-scale real-world conferences, our work can serve as a benchmark for subsequent research.

![Refer to caption](img/31d22310dc52243e46f8b17935620c6c.png)

Figure 1: MiniFed Pipeline

## II Related Works

In this section, we review prior literature on the reflection and application of FOMC materials, credible proxies for human behavior, and LLM-based multi-agent frameworks for real-world applications. We demonstrate that current LLM models are capable of simulating complex interactive human behaviors, such as those observed in FOMC meetings.

### II-A FOMC Related Researches

The Federal Open Market Committee (FOMC) plays a crucial role in shaping the United States’ monetary policy to promote economic stability and growth. Its primary duties include setting interest rates, conducting open market operations to regulate the money supply, and assessing economic conditions to achieve goals like controlling inflation and maximizing employment. Its importance lies in its ability to influence national and global financial markets, impact borrowing costs, and steer the economy toward sustainable growth through informed policy decisions.

The FOMC disseminates a comprehensive array of materials throughout its meeting process, including transcripts (detailed records of meeting proceedings), tealbooks (economic analyses and descriptions of policy alternatives), agendas (lists of topics to be addressed at each meeting), minutes (summaries of issues discussed), and beige books (information on current economic conditions by district). Although the materials provided by the FOMC are detailed and exhaustive, research on the FOMC remains in its early stages.

Existing literature primarily focuses on empirical analyses of the effects of FOMC monetary policy announcements on global asset prices[[10](https://arxiv.org/html/2410.18012v2#bib.bib10)] and U.S. financial market responses[[11](https://arxiv.org/html/2410.18012v2#bib.bib11)][[12](https://arxiv.org/html/2410.18012v2#bib.bib12)]. Additionally, some researchers argue that informed traders in the financial market are able to trade effectively before the FOMC releases its monetary policy announcements. Consequently, studying financial market turbulence prior to the publication of the FOMC’s target rate is also considered valuable[[13](https://arxiv.org/html/2410.18012v2#bib.bib13)][[14](https://arxiv.org/html/2410.18012v2#bib.bib14)].

Recently, as deep learning technologies have begun to emerge, many researchers have sought to employ deep learning–enhanced methods to analyze contextual sentiments in documents released by the FOMC. Tsang et al.[[15](https://arxiv.org/html/2410.18012v2#bib.bib15)] developed a deep learning model based on a self-attention mechanism to measure disagreement among members in each FOMC meeting, utilizing records of dissents on FOMC votes and meeting transcripts from 1976 to 2017\. Additionally, with the growing popularity of fine-tuning methods for large language models (LLMs), Gössi et al.[[16](https://arxiv.org/html/2410.18012v2#bib.bib16)] fine-tuned the pre-trained FinBERT model to analyze the sentiments present in FOMC Minutes materials. However, overall research on deep learning–based approaches to FOMC materials remains in its infancy.

### II-B LLM-Based Multi-Agent Frameworks

As the field of artificial intelligence rapidly expands, researchers are exploring multi-agent systems in which multiple AI entities collaborate to achieve common objectives and address complex real-world challenges[[17](https://arxiv.org/html/2410.18012v2#bib.bib17)][[18](https://arxiv.org/html/2410.18012v2#bib.bib18)]. One notable example is CAMEL[[19](https://arxiv.org/html/2410.18012v2#bib.bib19)], which introduces a communicative multi-agent framework designed to reduce the need for human user input prompts and provides a scalable approach for examining the cooperative behaviors and capabilities of multi-agent systems.

Another intriguing area in this field involves utilizing large language model (LLM) agents to simulate real-world scenarios. LLMs, particularly GPT-4[[20](https://arxiv.org/html/2410.18012v2#bib.bib20)], have demonstrated near-human-level performance across a variety of challenging tasks, including mathematics[[21](https://arxiv.org/html/2410.18012v2#bib.bib21)], coding[[22](https://arxiv.org/html/2410.18012v2#bib.bib22)], vision[[23](https://arxiv.org/html/2410.18012v2#bib.bib23)], medicine[[24](https://arxiv.org/html/2410.18012v2#bib.bib24)], law[[25](https://arxiv.org/html/2410.18012v2#bib.bib25)], psychology[[26](https://arxiv.org/html/2410.18012v2#bib.bib26)], and more. Consequently, developing methods to enable LLMs to behave like real humans within simulated environments and observing their interactions and performance represents a compelling research direction.

One notable study involves simulating human community life and monitoring interactions, behaviors, and dialogues in a game environment by leveraging generative agents and a novel agent architecture[[6](https://arxiv.org/html/2410.18012v2#bib.bib6)]. Additionally, other studies have focused on simulating various real-world scenarios by deploying LLM agents, such as school environments[[8](https://arxiv.org/html/2410.18012v2#bib.bib8)] and travel trajectories[[27](https://arxiv.org/html/2410.18012v2#bib.bib27)].

Specifically, simulating human roles within specific scenarios is important because it provides a novel method for observing detailed decision-making and thought processes that are difficult to study in the real world. These studies typically require more specialized information to define or pretrain the agents, enabling them to more accurately emulate real roles under specific circumstances. One such work is Blind Judgment[[7](https://arxiv.org/html/2410.18012v2#bib.bib7)], which employs LLM-based agents to simulate the roles of judges on the Roberts IV court. These agents are pretrained with real information and tasked with voting on real-world cases. However, due to the limited capabilities of LLMs three years ago, the results were not as satisfying as those of current research. Another important work involves allowing LLM agents to participate in market research[[9](https://arxiv.org/html/2410.18012v2#bib.bib9)] and observing their reasoning processes. The experimental results show similarities between LLM agent participants and human participants, demonstrating the possibility of LLM agents substituting humans in market surveys.

Other research primarily focuses on designing LLM agent frameworks, including the development of frameworks for practical applications[[18](https://arxiv.org/html/2410.18012v2#bib.bib18)] and the optimization of LLM agent frameworks[[28](https://arxiv.org/html/2410.18012v2#bib.bib28)]. We will leverage insights from these studies in our forthcoming research.

## III MiniFed: A multi LLM agent-based approach for simulating FOMC meeting

In this section, we introduce MiniFed, our novel agentic-workflow-based framework for simulating FOMC meetings. MiniFed encompasses agent-based real-world role simulations and our proposed architectural design. We will demonstrate the efficacy and superiority of our agentic collaboration framework.

### III-A Agent-based Real-World Role Simulation

According to the meeting transcripts provided by the Federal Reserve website, each FOMC meeting includes approximately 100 individuals. These participants comprise members and alternate members of the FOMC, presidents of Federal Reserve banks, secretaries and general counsels, economists, members of the board of governors, and other relevant personnel.

However, only a limited number of FOMC meeting members participate in the discussion and voting process for the federal funds rate. Previous studies[[28](https://arxiv.org/html/2410.18012v2#bib.bib28)] have also demonstrated that there is no linear relationship between the size of the agent team and the final accuracy, suggesting that increasing the team size may actually reduce overall efficacy. Additionally, recreating the original meeting scenario, which involves hours of discussion, is currently unfeasible using LLMs. To effectively capture the characteristics of FOMC meetings and obtain as much useful information as possible, we select the seven most important and representative roles among the meeting participants: the chairman and vice chairman of the FOMC, one regional Federal Reserve Bank president, one member of the board of governors, one economist, and one legal expert.

Since we have already selected the representative members who will participate in our MiniFed meeting, accurately attributing their personal characteristics and ensuring that the agents behave like their real-world counterparts is a vital factor affecting our experimental results. Previous research has primarily focused on socio-demographic[[29](https://arxiv.org/html/2410.18012v2#bib.bib29)] and personality[[30](https://arxiv.org/html/2410.18012v2#bib.bib30)][[31](https://arxiv.org/html/2410.18012v2#bib.bib31)][[32](https://arxiv.org/html/2410.18012v2#bib.bib32)] approaches to enhance LLM agents’ ability to mimic real humans. Additionally, Ji et al.[[29](https://arxiv.org/html/2410.18012v2#bib.bib29)] demonstrated that it is possible to reconstruct an LLM agent’s personality using only simple prompts with appropriate socio-demographic settings. Therefore, we follow their methodology to reconstruct our agents’ socio-demographic profiles and personalities based on accurate information.

To be more specific, we manually assign each agent socio-demographic and personality information before commencing the meeting. Socio-demographic information includes name, gender, past work, and educational experience, which can be directly obtained from publicly available online sources. In contrast, personality information is relatively abstract and challenging to acquire. To address this, we leverage the capabilities of large language models (LLMs) to generate personas, stances, and attitudes based on their past speeches and disclosed information, such as transcripts from previous FOMC meetings. We will initiate our experiments using these predefined agents.

### III-B MiniFed: Toward A Simple But Efficient Meeting Simulation

Real-world meetings are often prolix and verbose, making them less engaging for audiences. The original FOMC meeting transcripts include every sentence spoken by FOMC members, even incorporating gossip and jokes. Additionally, each FOMC meeting is meticulously planned and organized with a clear structure, addressing around ten issues according to the agendas on the FOMC website. Simply initiating the meeting with basic prompts may result in repetitive and meaningless conversations and outcomes. We have carefully evaluated the capacities of LLM agents[[33](https://arxiv.org/html/2410.18012v2#bib.bib33)][[34](https://arxiv.org/html/2410.18012v2#bib.bib34)][[35](https://arxiv.org/html/2410.18012v2#bib.bib35)] and determined that reconstructing a full, multi-hour meeting with numerous issues is challenging given the current capabilities of LLMs. Therefore, to best leverage the core characteristics of FOMC meetings and the capacities of LLM agents, we have designed our MiniFed meeting framework, which consists of five stages. This framework encompasses all essential components of the original FOMC meetings while reducing the overall dialogue volume.

The MiniFed meeting framework is outlined as follows:

1.  1.

    Monetary Policy Proposal Generalization. According to the transcripts of each FOMC meeting disclosed on the FOMC website, the target federal funds rate is selected from three alternatives provided by economists at the Federal Reserve Bank, rather than being directly discussed by FOMC members. To emulate this process, we create an economist agent pre-trained using the Beige Book, which is publicly accessible on the website before each FOMC meeting, and TealBook A, an internal resource containing more comprehensive and detailed economic data and analysis provided by Federal Reserve Bank economists. We deliberately exclude TealBook B, which includes the actual monetary policy alternatives, to prevent the economist agent from accessing the answers prematurely. Before each MiniFed meeting, the economist agent is tasked with generating three monetary policy alternatives for the discussion and voting process during the meeting.

2.  2.

    Personal Idea Generalization. In this process, each participant agent in our MiniFed meeting who has the authority to vote on the final monetary policy will be required to read the Beige Book and TealBook A. Based on their predefined socio-demographic backgrounds and personalities, they will generate their own thoughts and ideas regarding the target federal funds rate. These ideas will remain confidential and will not be shared with other agents.

3.  3.

    First Round Discussion: Idea Presentation. Once each agent has thoroughly studied the provided materials and developed their own perspectives on the target federal funds rate based on their individual information and the current economic conditions, they are invited to present their viewpoints and explain the reasoning behind their positions. The order of presentations is randomized, allowing all participants to hear each other’s presentations without engaging in further comments or discussions.

4.  4.

    Second Round Discussion: Debate and Reflection. After each agent presents their monetary policy perspectives in the first round, they engage in a second round of discussion to debate and reflect on both their own and others’ viewpoints. Each agent listens to the presentations of the other agents and is required to consider all the information they have heard before making their subsequent remarks. Each agent is allotted three speaking opportunities, resulting in a total of fifteen speech rounds. The order of speeches during the discussion is randomly determined.

5.  5.

    Legislative Suggestions and Final Vote. Before voting on one of the three monetary policy alternatives generated by the economist agent, the legal expert provides concise reviews of each proposed alternative from legal perspectives, including regulatory and compliance considerations. After the legal expert has evaluated all three alternatives, all participant agents—excluding the legal expert and the economist—are asked to vote for their preferred monetary policy.

![Refer to caption](img/92a25d97e4f504c11d0803ffe59aa447.png)

Figure 2: MiniFed Meeting Framework

## IV Experiments

### IV-A Dataset

We obtained all necessary data from the Federal Reserve’s open-access database on their official website²²2https://www.federalreserve.gov/monetarypolicy/fomc_historical.htm to predefine our agents. Although the Federal Reserve has disclosed all FOMC meeting-related materials from 1936 to 2018, spanning the five years prior to 2018, our research focuses exclusively on the year 2018\. We primarily utilize the Beige Book and TealBook A to pre-train and predefine the agents participating in our MiniFed meeting, and we organize the meeting framework by referencing the Agenda and Transcripts materials described in the previous section.

### IV-B Experiment Setup

We organized eight meetings for the year 2018, aligning them with their real-world counterparts. To mitigate hallucinations caused by the limitations of current large language models (LLMs)[[7](https://arxiv.org/html/2410.18012v2#bib.bib7)], we initialized each agent prior to the commencement of each meeting. Each experiment lasted approximately 45 minutes and encompassed all procedures, including agent generation, predefining configurations, and our five-stage MiniFed meeting framework. The cumulative cost of the 8 experiments was approximately 1 billion tokens, powered by the ChatGPT’s GPT-4o mini model. All experiments were conducted using our independently developed platform, which directly interfaces with the ChatGPT’s API to utilize the base LLMs and construct the MiniFed framework.

### IV-C Evaluation Metrics

Although many previous studies have utilized LLM-based multi-agent frameworks to simulate real-world scenarios[[6](https://arxiv.org/html/2410.18012v2#bib.bib6)][[7](https://arxiv.org/html/2410.18012v2#bib.bib7)][[8](https://arxiv.org/html/2410.18012v2#bib.bib8)], research specifically focused on conference or meeting simulation remains in its infancy. Furthermore, each meeting involves a substantial amount of natural language, making it challenging to directly apply numerical metrics to evaluate the quality of our simulated meetings. To address this issue, we assess our simulated MiniFed meetings from two perspectives: alignment and accuracy, in order to evaluate the efficacy and quality of our MiniFed framework.

1.  1.

    Alignment: We compare each agent’s voting result with that of their real-world counterpart to assess whether the virtual agents, equipped with our predefined socio-demographic and personality profiles, vote in the same direction as their real-world equivalents. Specifically, the current federal funds rate typically has three possible directions: increasing the fund rate, maintaining the current fund rate, or decreasing the fund rate. If an agent votes in the same direction as its real-world counterpart, we categorize its vote as “aligned”; otherwise, it is considered “not aligned”. We calculate the Alignment Rate (AR) for each agent, where a higher AR signifies better performance of our framework. The formula for AR is defined as follows:

    |  | $\textit{AR}_{t}=\frac{1}{n}\sum_{i=1}^{n}Alig_{i,t}$ |  |

    where $i$ represents the meeting instances, $t$ represents the agent numbers and $Alig_{i}$ is equal to 1 if the agent $t$’s vote result aligns with its real-world counterpart, and 0 otherwise.

2.  2.

    Accuracy: Since our MiniFed framework can be used to predict the potential federal funds rate, we examine the gap between the adjusted funds rates of real FOMC meetings and those of our MiniFed meetings. The difference between the real-world results and the virtual outcomes serves as a guideline for improving our framework. We measure the Mean Squared Error (MSE) as an indicator for each agent, and the formula can be represented as:

    |  | $\textit{MSE}=\frac{1}{n}\sum_{i=1}^{n}\left(R_{\text{real},i}-R_{\text{% simulated},i}\right)^{2}$ |  |

    where $R_{real}$ and $R_{simulated}$ represent the Federal Funds rate decided by the real FOMC meeting and our MiniFed framework, respectively. Here, $i$ denotes the meeting instances.

### IV-D Main Results

We present our experimental results in the following tables. Table 1 presents the alignment rate for each agent. It is important to note that in 2018, J. Yellen served as the FOMC chairman only in January, after which Powell assumed the position. Consequently, she participated in voting only once during the year.

TABLE I: Agent Alignment Analysis

| Agent | Alignment Rate |
| J. Yellen | 0 |
| J. Powell | 85.7% |
| W. Dudley | 87.5% |
| L. Brainard | 50% |
| R.Bostic | 37.5% |
| L. Mester | 75% |

Table 4 provides a detailed overview of the eight experiments, including each agent’s initial stance on the current monetary policy during the first-round discussion, their final vote after the second-round discussion, and the actual vote decisions of their real-world counterparts. The symbols $\uparrow$ and $\rightarrow$ denote support for the alternatives of “increasing the Fed Funds rate” and “maintaining the current Fed Funds rate,” respectively.

TABLE II: Meeting Result Analysis

| Date | MiniFed Funds Rate | FOMC Funds Rate | Gap |
| Jan. 2018 | 1.25% $\rightarrow$ 1.5% | 1.25% $\rightarrow$ 1.25% | 0.25% |
| Mar. 2018 | 1.25% $\rightarrow$ 1.5% | 1.25% $\rightarrow$ 1.5% | 0 |
| May 2018 | 1.5% $\rightarrow$ 1.5% | 1.5% $\rightarrow$ 1.5% | 0 |
| June 2018 | 1.5% $\rightarrow$ 1.75% | 1.5% $\rightarrow$ 1.75% | 0 |
| July 2018 | 1.75% $\rightarrow$ 1.75% | 1.75% $\rightarrow$ 1.75% | 0 |
| Sep. 2018 | 1.75% $\rightarrow$ 1.75% | 1.75% $\rightarrow$ 2.0% | -0.25% |
| Nov. 2018 | 2.0% $\rightarrow$ 2.0% | 2.0% $\rightarrow$ 2.0% | 0 |
| Dec. 2018 | 2.0% $\rightarrow$ 2.25% | 2.0% $\rightarrow$ 2.25% | 0 |

TABLE III: Monetary Policy Comparasion

 | Date | Fed Funds Rate | MiniFed Economist’s Alt. | FOMC Alt. |
| Jan. 2018 | 1.25% | A: 1.5% $\uparrow$ | A: 1.25% $\rightarrow$ |
| B: 1.25% $\rightarrow$ | B: 1.25% $\rightarrow$ |
| C: 1.0% $\downarrow$ | C: 1.5% $\uparrow$ |
| Mar. 2018 | 1.25% | A: 1.5% $\uparrow$ | A: 1.25% $\rightarrow$ |
| B: 1.25% $\rightarrow$ | B: 1.5% $\uparrow$ |
| C: 1.0% $\downarrow$ | C: 1.5% $\uparrow$ |
| May 2018 | 1.5% | A: 1.75% $\uparrow$ | A: 1.5% $\rightarrow$ |
| B: 1.5% $\rightarrow$ | B: 1.5% $\rightarrow$ |
| C: 1.25% $\downarrow$ | C: 1.75% $\uparrow$ |
| June 2018 | 1.5% | A: 1.75% $\uparrow$ | A: 1.5% $\rightarrow$ |
| B: 1.5% $\rightarrow$ | B: 1.75% $\uparrow$ |
| C: 1.25% $\downarrow$ | C: 1.75% $\uparrow$ |
| July 2018 | 1.75% | A: 2.0% $\uparrow$ | A: 1.75% $\rightarrow$ |
| B: 1.5% $\downarrow$ | B: 1.75% $\rightarrow$ |
| C: 1.75% $\rightarrow$ | C: 2.0% $\uparrow$ |
| Sep. 2018 | 1.75% | A: 2.0% $\uparrow$ | A: 1.75% $\rightarrow$ |
| B: 1.75% $\rightarrow$ | B: 2.0% $\uparrow$ |
| C: 1.5% $\downarrow$ | C: 2.0% $\uparrow$ |
| Nov. 2018 | 2.0% | A: 2.25% $\uparrow$ | A: 2.0% $\rightarrow$ |
| B: 2.0% $\rightarrow$ | B: 2.0% $\rightarrow$ |
| C: 1.75% $\downarrow$ | C: 2.25% $\uparrow$ |
| Dec. 2018 | 2.0% | A: 2.25% $\uparrow$ | A: 2.0% $\rightarrow$ |
| B: 2.0% $\rightarrow$ | B: 2.25% $\uparrow$ |
| C: 1.75% $\downarrow$ | C: 2.25% $\uparrow$ | 

Table 2 presents the gap between the federal funds rate decisions from real-world FOMC meetings and those generated by our MiniFed framework across eight experiments. We use annotations such as “1.5% $\rightarrow$ 1.5%” to indicate that the vote result was to “maintain the Federal Funds rate at 1.5%”, and apply similar annotations for other actions.

TABLE IV: Experiments Summary

 | Date | Participant Agent | Initial Idea | Final Idea | Real Idea |
| Jan. 2018 | J. Yellen | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| W. Dudley | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| L. Brainard | $\uparrow$ | $\uparrow$ | $\rightarrow$ |
| R. Bostic | $\rightarrow$ | $\uparrow$ | $\rightarrow$ |
| L. Mester | $\uparrow$ | $\uparrow$ | $\rightarrow$ |
| Mar. 2018 | J. Powell | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| W. Dudley | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| L. Brainard | $\uparrow$ | $\rightarrow$ | $\uparrow$ |
| R. Bostic | $\uparrow$ | $\rightarrow$ | $\uparrow$ |
| L. Mester | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| May 2018 | J. Powell | $\rightarrow$ | $\rightarrow$ | $\rightarrow$ |
| W. Dudley | $\rightarrow$ | $\rightarrow$ | $\rightarrow$ |
| L. Brainard | $\rightarrow$ | $\rightarrow$ | $\rightarrow$ |
| R. Bostic | $\rightarrow$ | $\uparrow$ | $\rightarrow$ |
| L. Mester | $\rightarrow$ | $\rightarrow$ | $\rightarrow$ |
| June 2018 | J. Powell | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| W. Dudley | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| L. Brainard | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| R. Bostic | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| L. Mester | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| July 2018 | J. Powell | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| W. Dudley | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| L. Brainard | $\uparrow$ | $\uparrow$ | $\rightarrow$ |
| R. Bostic | $\uparrow$ | $\uparrow$ | $\rightarrow$ |
| L. Mester | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| Sep. 2018 | J. Powell | $\uparrow$ | $\rightarrow$ | $\uparrow$ |
| W. Dudley | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| L. Brainard | $\uparrow$ | $\rightarrow$ | $\uparrow$ |
| R. Bostic | $\uparrow$ | $\rightarrow$ | $\uparrow$ |
| L. Mester | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| Nov. 2018 | J. Powell | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| W. Dudley | $\rightarrow$ | $\rightarrow$ | $\rightarrow$ |
| L. Brainard | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| R. Bostic | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| L. Mester | $\uparrow$ | $\rightarrow$ | $\rightarrow$ |
| Dec. 2018 | J. Powell | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| W. Dudley | $\uparrow$ | $\rightarrow$ | $\uparrow$ |
| L. Brainard | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| R. Bostic | $\uparrow$ | $\uparrow$ | $\uparrow$ |
| L. Mester | $\uparrow$ | $\rightarrow$ | $\uparrow$ | 

Table 3 presents a comparison of all alternatives generated by our MiniFed economist agent with the real monetary policy alternatives provided by Federal Reserve economists. The notation “A: 2.25% $\uparrow$ ” signifies “Alternative A: increase the Federal Funds rate to 2.25%.” It is important to note that the Federal Funds rate in real-world alternatives typically includes a target range; however, in our experiments, we only consider the lower boundary. Additionally, Federal Reserve economists usually propose two alternatives for adjusting the Federal Funds rate within the same range but with different paces and methods, as illustrated in Table 3.

Alignment. We compared the Alignment Rate (AR) of each agent as shown in Table 1\. The AR ranged from 37.5% to 87.5%, excluding J. Yellen, who only voted once. We observed that there are no specific distribution patterns of AR among agents. However, agents in higher positions (chairman and vice chairman of the FOMC) generally exhibited higher ARs compared to their real-world counterparts.

Accuracy. We report the accuracy of the Federal Funds rate projections generated by our MiniFed framework in comparison with real-world results. As shown in Table 2, six meetings produced identical outcomes to their real-world counterparts. The maximum discrepancies in the Federal Funds rate were limited to approximately 0.25%, resulting in an overall Mean Squared Error (MSE) of 0.0156%. Additionally, our framework achieved a 75% alignment rate with real-world results in terms of Federal Funds rate projections.

## V Dicsussion

In this section, we will discuss several important issues and potential risks that occurred in our experiments and related research, as well as the methods we devised to address them.

### V-A LLM Agents’ Hallucination and Recognition

Before each meeting begins, we provide our agents with materials that typically include a large amount of textual data, such as descriptions of economic conditions, as well as non-textual data like graphs, tables, and economic models. Understanding large-scale multimodal data and storing it within their limited memory presents a significant challenge for current LLMs. This may lead to hallucination issues for each agent, as mentioned in [[6](https://arxiv.org/html/2410.18012v2#bib.bib6)] and [[7](https://arxiv.org/html/2410.18012v2#bib.bib7)]. To assess whether the agents can truly understand and remember what they have learned—including both text and non-text data—we use the following prompt to examine their comprehension abilities:

Prompt: Based on your understanding of the provided materials, what is the current economic condition in the region of {a randomly selected Federal Reserve district}?

Then we compare the answers generated by the agents with the corresponding content in the original materials. If hallucination issues occur, we have the agents review the materials again and reconsider their answers until they can produce the correct responses.

### V-B Alternative Approaches to the Meeting Procedure

As we mentioned before, FOMC meetings are usually complicated and prolonged, making it difficult for current LLM agents to effectively emulate them. One example is the voting procedure. In real FOMC meetings, participants typically go through hours of discussions and debates and conclude with a unanimous decision, leading all participants to vote for the same monetary policy alternative, as shown in Table 4\. However, it’s difficult to replicate this process with the capacity of current LLMs. Therefore, we simplified the voting procedure by allowing each agent to vote for one of the three alternatives they regard as the most appropriate.

### V-C Limitations

Although we aim to reconstruct the original FOMC meeting scenario with the highest degree of realism, there remain some barriers that are difficult to overcome. Hallucinations still occur randomly during our experimental process, despite our efforts to control their occurrence as described in subsection A. Additionally, it is challenging to balance accuracy and the number of agents, since increasing the number of agents participating in the meeting may lead to higher memory costs and decreased overall performance. Therefore, we have made the expedient decision to limit the number of meeting participants to seven. Moreover, using OpenAI’s API is costly—the total experiments cost us hundreds of dollars—so we have endeavored to control the length of our experiments while maintaining optimal performance. We will address these questions in our future research. For ease of use, we have conducted our experiments in Chinese, consistent with the demo, and have manually translated content such as prompts for this paper. We believe that the superior language capabilities of LLMs ensure that language will not pose any issues.

## VI Conclusion

In this paper, we introduced MiniFed, our novel multi-LLM agent meeting framework designed to simulate eight FOMC meetings in 2018\. We began by defining the agents’ characteristics from socio-demographic and persona perspectives, followed by the implementation of our proposed five-stage MiniFed meeting framework. The experimental results demonstrate the efficacy and accuracy of our framework, surpassing outcomes generated by random guessing. This work establishes a benchmark in the simulation of large-scale real-world conferences using LLM agentic workflows, with results applicable to both prediction and reconstruction tasks. Future work may focus on addressing several challenges identified in our experiments and developing more realistic meeting scenarios without limiting the number of participants.

## References

*   [1] S. Arslanalp, B. Eichengreen, and C. Simpson-Bell, “The stealth erosion of dollar dominance and the rise of nontraditional reserve currencies,” *Journal of International Economics*, vol. 138, p. 103656, 2022.
*   [2] M. Iacoviello and G. Navarro, “Foreign effects of higher us interest rates,” *Journal of International Money and Finance*, vol. 95, pp. 232–250, 2019.
*   [3] F. E. Warnock and V. C. Warnock, “International capital flows and us interest rates,” *Journal of International Money and Finance*, vol. 28, no. 6, pp. 903–919, 2009.
*   [4] W. Huang, A. V. Mollick, and K. H. Nguyen, “Us stock markets and the role of real interest rates,” *The Quarterly Review of Economics and Finance*, vol. 59, pp. 231–242, 2016.
*   [5] S. Karau, “Monetary policy and bitcoin,” *Journal of International Money and Finance*, vol. 137, p. 102880, 2023.
*   [6] J. S. Park, J. O’Brien, C. J. Cai, M. R. Morris, P. Liang, and M. S. Bernstein, “Generative agents: Interactive simulacra of human behavior,” in *Proceedings of the 36th annual acm symposium on user interface software and technology*, 2023, pp. 1–22.
*   [7] S. Hamilton, “Blind judgement: Agent-based supreme court modelling with gpt,” *arXiv preprint arXiv:2301.05327*, 2023.
*   [8] Z. Zhang, D. Zhang-Li, J. Yu, L. Gong, J. Zhou, Z. Liu, L. Hou, and J. Li, “Simulating classroom education with llm-empowered agents,” *arXiv preprint arXiv:2406.19226*, 2024.
*   [9] P. Li, N. Castelo, Z. Katona, and M. Sarvary, “Frontiers: Determining the validity of large language models for automated perceptual analysis,” *Marketing Science*, vol. 43, no. 2, pp. 254–266, 2024.
*   [10] J. Hausman and J. Wongswan, “Global asset prices and fomc announcements,” *Journal of International Money and Finance*, vol. 30, no. 3, pp. 547–571, 2011.
*   [11] A. Cieslak, A. Morse, and A. Vissing-Jorgensen, “Stock returns over the fomc cycle,” *The Journal of Finance*, vol. 74, no. 5, pp. 2201–2248, 2019.
*   [12] C. Rosa, “The financial market effect of fomc minutes,” *Economic Policy Review*, vol. 19, no. 2, 2013.
*   [13] B. Du, S. Fung, and R. Loveland, “The informational role of options markets: Evidence from fomc announcements,” *Journal of Banking & Finance*, vol. 92, pp. 237–256, 2018.
*   [14] D. O. Lucca and E. Moench, “The pre-fomc announcement drift,” *The Journal of finance*, vol. 70, no. 1, pp. 329–371, 2015.
*   [15] K. P. Tsang and Z. Yang, “Agree to disagree: Measuring hidden dissents in fomc meetings,” *arXiv preprint arXiv:2308.10131*, 2023.
*   [16] S. Gössi, Z. Chen, W. Kim, B. Bermeitinger, and S. Handschuh, “Finbert-fomc: Fine-tuned finbert model with sentiment focus method for enhancing sentiment analysis of fomc minutes,” in *Proceedings of the Fourth ACM International Conference on AI in Finance*, 2023, pp. 357–364.
*   [17] D. Dohan, W. Xu, A. Lewkowycz, J. Austin, D. Bieber, R. G. Lopes, Y. Wu, H. Michalewski, R. A. Saurous, J. Sohl-Dickstein *et al.*, “Language model cascades,” *arXiv preprint arXiv:2207.10342*, 2022.
*   [18] Y. Talebirad and A. Nadiri, “Multi-agent collaboration: Harnessing the power of intelligent llm agents,” *arXiv preprint arXiv:2306.03314*, 2023.
*   [19] G. Li, H. Hammoud, H. Itani, D. Khizbullin, and B. Ghanem, “Camel: Communicative agents for” mind” exploration of large language model society,” *Advances in Neural Information Processing Systems*, vol. 36, pp. 51 991–52 008, 2023.
*   [20] S. Bubeck, V. Chandrasekaran, R. Eldan, J. Gehrke, E. Horvitz, E. Kamar, P. Lee, Y. T. Lee, Y. Li, S. Lundberg *et al.*, “Sparks of artificial general intelligence: Early experiments with gpt-4,” *arXiv preprint arXiv:2303.12712*, 2023.
*   [21] R. Zhang, D. Jiang, Y. Zhang, H. Lin, Z. Guo, P. Qiu, A. Zhou, P. Lu, K.-W. Chang, P. Gao *et al.*, “Mathverse: Does your multi-modal llm truly see the diagrams in visual math problems?” *arXiv preprint arXiv:2403.14624*, 2024.
*   [22] D. Nam, A. Macvean, V. Hellendoorn, B. Vasilescu, and B. Myers, “Using an llm to help with code understanding,” in *Proceedings of the IEEE/ACM 46th International Conference on Software Engineering*, 2024, pp. 1–13.
*   [23] W. Wang, Z. Chen, X. Chen, J. Wu, X. Zhu, G. Zeng, P. Luo, T. Lu, J. Zhou, Y. Qiao *et al.*, “Visionllm: Large language model is also an open-ended decoder for vision-centric tasks,” *Advances in Neural Information Processing Systems*, vol. 36, 2024.
*   [24] A. J. Thirunavukarasu, D. S. J. Ting, K. Elangovan, L. Gutierrez, T. F. Tan, and D. S. W. Ting, “Large language models in medicine,” *Nature medicine*, vol. 29, no. 8, pp. 1930–1940, 2023.
*   [25] J. Cui, Z. Li, Y. Yan, B. Chen, and L. Yuan, “Chatlaw: Open-source legal large language model with integrated external knowledge bases,” *arXiv preprint arXiv:2306.16092*, 2023.
*   [26] Q. Yang, Z. Wang, H. Chen, S. Wang, Y. Pu, X. Gao, W. Huang, S. Song, and G. Huang, “Llm agents for psychology: A study on gamified assessments,” *arXiv preprint arXiv:2402.12326*, 2024.
*   [27] X. Li, F. Huang, J. Lv, Z. Xiao, G. Li, and Y. Yue, “Be more real: Travel diary generation using llm agents and individual profiles,” *arXiv preprint arXiv:2407.18932*, 2024.
*   [28] Z. Liu, Y. Zhang, P. Li, Y. Liu, and D. Yang, “Dynamic llm-agent network: An llm-agent collaboration framework with agent team optimization,” *arXiv preprint arXiv:2310.02170*, 2023.
*   [29] Y. Ji, Z. Tang, and M. Kejriwal, “Is persona enough for personality? using chatgpt to reconstruct an agent’s latent personality from simple descriptions,” *arXiv preprint arXiv:2406.12216*, 2024.
*   [30] G. V. Aher, R. I. Arriaga, and A. T. Kalai, “Using large language models to simulate multiple humans and replicate human subject studies,” in *International Conference on Machine Learning*.   PMLR, 2023, pp. 337–371.
*   [31] G. Kovač, M. Sawayama, R. Portelas, C. Colas, P. F. Dominey, and P.-Y. Oudeyer, “Large language models as superpositions of cultural perspectives,” *arXiv preprint arXiv:2307.07870*, 2023.
*   [32] T. Hu and N. Collier, “Quantifying the persona effect in llm simulations,” *arXiv preprint arXiv:2402.10811*, 2024.
*   [33] O. Sainz, J. A. Campos, I. García-Ferrero, J. Etxaniz, O. L. de Lacalle, and E. Agirre, “Nlp evaluation in trouble: On the need to measure llm data contamination for each benchmark,” *arXiv preprint arXiv:2310.18018*, 2023.
*   [34] M. Sclar, Y. Choi, Y. Tsvetkov, and A. Suhr, “Quantifying language models’ sensitivity to spurious features in prompt design or: How i learned to start worrying about prompt formatting,” *arXiv preprint arXiv:2310.11324*, 2023.
*   [35] M. Loya, D. A. Sinha, and R. Futrell, “Exploring the sensitivity of llms’ decision-making capabilities: Insights from prompt variation and hyperparameters,” *arXiv preprint arXiv:2312.17476*, 2023.

### -A Selected Prompts

TABLE V: Prompts Table

| Function | Prompts |
| Characters Define | You will play the role of Federal Reserve Chairman Jerome H. Powell, participating in the May 2018 FOMC meeting. In the subsequent dialogues, you will faithfully respond as this character. Stance: Focused on maintaining overall economic stability, particularly balancing between inflation and employment. Supports guiding the economy steadily forward through gradual interest rate increases, avoiding economic overheating or excessive tightening. |
| Socio-Demographic Information Input | Gender: Male. Educational Background: Bachelor’s degree in Politics from Princeton University; J.D. from Georgetown University. Past Positions: Former visiting scholar at the Bipartisan Policy Center; partner at The Carlyle Group; board member of charitable and educational institutions; Assistant Secretary and Undersecretary at the Treasury Department. |
| Personality Input | Personality: Humble and inclusive leadership style; values listening to different viewpoints and is committed to building consensus within the committee. Viewpoint: Keep interest rates unchanged. |
| Materials Learning | Below is the content of the Beige Book; please read and understand it carefully. After completing your reading and comprehension, please reply with “Completed”. |
| Monetary Policy Alternatives Generation | Currently, the federal funds rate is 1.50%, and the May 2018 FOMC meeting is in session. As an economist at the Federal Reserve, you have thoroughly studied the contents of Tealbook A and the Beige Book. Based on these materials, please generate three different interest rate adjustment proposals (increase, decrease, or maintain the current rate) and provide detailed economic justifications behind each proposal. |
| Personal Idea Generalization | Please stay true to your role at the Federal Reserve, and based on your understanding of the current meeting materials, specific economic data, and information, generate the genuine reasons for your own viewpoint (raise interest rates, lower interest rates, or keep them unchanged). |
| First Round Discussion | Please stay true to your role and begin your first-round speech at the May 2018 FOMC meeting, articulating your viewpoint (raise interest rates, lower interest rates, or keep them unchanged) based on specific data and information. |
| Second Round Discussion | Now we are in the free discussion session. Participants can refer to the three interest rate adjustment proposals put forward by the economists for in-depth discussion. Please stay true to your role, combining specific economic data, meeting materials, and the content of previous members’ speeches to make your own independent judgment and explain your reasons. Please pay special attention that the meeting time is set to May 2018, so ensure your response is based on data from May 2018\. You don’t have to maintain your previous viewpoint; instead, choose the most appropriate plan at this time. |
| Legal Expert Reviews | As a legal expert, you need to provide insights on legal and regulatory aspects to help evaluate the legal compliance of these proposals and their potential regulatory impacts. Please combine your professional expertise and the meeting materials to express your views and comment on the previous proposals by the committee members. |
| Final Vote | Now we are entering the voting stage. We will determine the final plan based on everyone’s votes. You can only choose one option (raise interest rates, lower interest rates, or keep the interest rate unchanged), and you do not need to explain your reasons at this stage. Please make your choice based on the previous discussion. |

### -B Ablation Studies

#### -B1 Examining Whether LLMs Have Been Trained on the Meeting Materials

We have employed the GPT-4o mini model as the foundational model for our agents. As the latest LLM released by OpenAI, we suspect that the GPT-4o mini may have been trained on the 2018 meeting materials, particularly the transcripts, which contain the voting results of each FOMC meeting participant. This could potentially lead to inaccuracies in the agents’ voting outcomes. To verify whether the model had been trained on the meeting materials prior to the commencement of our experiments, we randomly selected information from the Beige Book, which contains economic condition reports for each region and is publicly accessible online before each FOMC meeting. We then employed the following prompts to test the model’s memory before our experiments began:
Prompt: How did automobile dealers in the Cleveland area describe the demand for the automotive market and its underlying reasons during August and September 2018? This prompt contains detailed information about the automobile retail market in Cleveland, as described in the Beige Book. We then compared this information with the responses generated by the GPT-4o mini model and found that GPT-4o correctly answered the question, particularly regarding consumer demand for SUVs. Although this does not conclusively prove that GPT has been trained on all pertinent data related to this meeting, we conducted the following experiments to attempt to cleanse GPT’s memory regarding information about this meeting.

#### -B2 Cleanse GPT’s Memory with Simple Prompts

Given our suspicion that GPT has access to the meeting materials used in previous experiments, we have employed the following prompt to make GPT forget the information it had previously learned:
Forget any prior knowledge or information regarding the Federal Reserve’s FOMC meeting in September 2018\. From this point forward, you are unaware of any related information, processes, or outcomes of that meeting. Following our previous experiment, we tested GPT’s knowledge of the automobile market in the Cleveland region using the following prompt:
How did automobile dealers in the Cleveland area describe the demand for the automotive market and its underlying reasons during August and September 2018? GPT generated some responses in this instance; however, most of the information conflicted with the corresponding contents in the Beige Book. To ensure that the agents’ responses remained autonomous, we cleared all agents’ memories of the FOMC meetings before each experiment commenced using prompts.