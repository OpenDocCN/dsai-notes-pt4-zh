<!--yml
category: 未分类
date: 2025-01-11 12:47:56
-->

# Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents

> 来源：[https://arxiv.org/html/2403.05307/](https://arxiv.org/html/2403.05307/)

Jinyang Li^($\clubsuit$)${}^{*}$, Nan Huo^($\clubsuit$)${}^{*}$, Yan Gao^($\heartsuit$), Jiayi Shi^($\spadesuit$), Yingxiu Zhao, Ge Qu^($\clubsuit$),
Yurong Wu, Chenhao Ma^($\spadesuit$), Jian-Guang Lou^($\heartsuit$), Reynold Cheng^($\clubsuit$)
^($\clubsuit$)The University of Hong Kong, ^($\heartsuit$)Microsoft Research Asia
^($\spadesuit$)The Chinese University of Hong Kong, Shenzhen
{jl0725,huonan,quge}@connect.hku.hk, ckcheng@cs.hku.hk
{yan.gao,jlou}@microsoft.com 

###### Abstract

Interactive Data Analysis, a collaboration between humans and LLM agents, enables real-time data exploration for informed decision-making. The challenges and costs of collecting realistic interactive logs for data analysis hinder the quantitative evaluation of Large Language Model (LLM) agents in this task. To mitigate this issue, we introduce Tapilot-Crossing, a new benchmark to evaluate LLM agents on interactive data analysis. Tapilot-Crossing contains 1024 interactions, covering 4 practical scenarios: Normal, Action, Private, and Private Action. Notably, Tapilot-Crossing is constructed by an economical multi-agent environment, Decision Company, with few human efforts. We evaluate popular and advanced LLM agents in Tapilot-Crossing, which underscores the challenges of interactive data analysis. Furthermore, we propose Adaptive Interaction Reflection (AIR), a self-generated reflection strategy that guides LLM agents to learn from successful history. Experiments demonstrate that Air can evolve LLMs into effective interactive data analysis agents, achieving a relative performance improvement of up to 44.5%.²²2[https://tapilot-crossing.github.io/](https://tapilot-crossing.github.io/)

¹¹footnotetext: Equal contribution.

## 1 Introduction

The exponential growth of the big data calls for accessible data analysis or data science techniques that cater to a range of technical backgrounds in various data-driven domains, such as healthcare, games, and entertainment (Khanbabaei et al., [2018](https://arxiv.org/html/2403.05307v1#bib.bib23); Han et al., [2011](https://arxiv.org/html/2403.05307v1#bib.bib16); Fayyad et al., [1996](https://arxiv.org/html/2403.05307v1#bib.bib11)). Recently, the fast development of Large Language Model (LLM) agents (Liu et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib34); Xu et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib48); Zeng et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib58); Xu et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib47); Deng et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib8); Si et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib41)) have been recevied much attention. They are capable of understanding natural language queries and generating corresponding code or analysis for data manipulation and visualization by reasoning (Huang and Chang, [2023](https://arxiv.org/html/2403.05307v1#bib.bib19); Wei et al., [2022](https://arxiv.org/html/2403.05307v1#bib.bib45); Yao et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib51)) and tool calls (Li et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib30); Huang et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib21); Qin et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib39)).

![Refer to caption](img/f005bc3f0080301b774f9f60f9b5744b.png)

Figure 1: This is an overview of the four interaction modes in Tapilot-Crossing. Notable Opponents is ambiguous which requires clarification in multi-turn interaction.

SheetCopilot (Li et al., [2024c](https://arxiv.org/html/2403.05307v1#bib.bib28)), TableGPT (Zha et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib59)), Data-Copilot (Zhang et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib60)) showcase the potential of tabular data analysis agents through automatic workflow given the user queries. However, the dynamic and uncertain nature of real-world analysis obliges effective human-agent interaction. This is because user intents can often be ambiguous (De Vries et al., [2020](https://arxiv.org/html/2403.05307v1#bib.bib7); Yan et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib49); Wang et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib43)), and users may need to adjust their analysis strategies based on intermediate results (Yan et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib49); Yao et al., [2020](https://arxiv.org/html/2403.05307v1#bib.bib52)). For example, in Figure [1](https://arxiv.org/html/2403.05307v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"), the notable opponents could refer to a variety of interpretations, such as the opponents with the highest wins, or the most frequent opponents. To this end, a thorough benchmark for gauging their capability for interactive user engagement in data analysis scenarios is indispensable.

In this paper, we introduce Tapilot-Crossing, a new benchmark for evaluating LLM agents in interactive data analysis tasks. Tapilot-Crossing is designed to simulate real-world data analysis scenarios, where users interact with LLM agents to generate codes for data exploration and decision makings. It includes 1024 user-machine interactions with 1176 user intents, spanning four practical scenarios: 1) Normal, where all questions and user requirements are explicit, requiring no actions from agents; 2) Action, where agents must respond to diverse user feedback or instructions; 3) Private, which examines the true semantic parsing capability of agents when encountering unseen packages during the pre-training phase (Zan et al., [2022](https://arxiv.org/html/2403.05307v1#bib.bib57)); and 4) Private Action, a mode that combines the features of Private and Action, more closely reflecting real-world data analysis. There are two answer types: 1) Code Generation, which can test whether the agent can correctly interpret the user’s query and generate the corresponding code for data analysis, and 2) Multiple-Choice questions, which can evaluate the agent’s ability to understand the returned results being executed and provide appropriate insights for users.

The conventional construction of datasets or benchmarks based on crowdsourcing, especially for high-quality and interactive scenarios, is time-consuming and costly due to the significant human effort and expertise required (Yu et al., [2019a](https://arxiv.org/html/2403.05307v1#bib.bib54); Li et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib29); Guo et al., [2021](https://arxiv.org/html/2403.05307v1#bib.bib15); Li et al., [2023d](https://arxiv.org/html/2403.05307v1#bib.bib32); Zhang et al., [2023c](https://arxiv.org/html/2403.05307v1#bib.bib62)). In this case, we design a novel multi-agent environment, Decision Company, to construct Tapilot-Crossing. Decision Company is a simulated environment where 4 GPT-4 agents communicate with each other to perform data analysis tasks. By using this environment, two PhD students are able to construct Tapilot-Crossing within a span of one months at a cost of less than 100 US dollars.

We evaluate the popular advanced LLM agents on Tapilot-Crossing. The results underscore the challenges of interactive data analysis and fuel the need for more advanced LLM agents that can handle diverse user intents and feedback.

To further evolve the LLMs towards effective interactive data analysis agents, we propose Adaptive Interaction Reflection (AIR), which guides LLM agents to learn from successful history via self-generated pseudo logic reflection.

Our experiments demonstrate that AIR can significantly enhance the performance of LLMs, in which GPT-4 can gain relative improvement of 44.5% compared to its model base. offering an insight of how to actively improve the interaction between human and LLM agents in data analysis tasks.

In summary, our contributions are threefold:

*   •

    We introduce Tapilot-Crossing, a new benchmark for evaluating LLM agents in interactive data analysis tasks. Tapilot-Crossing is constructed by our designed multi-agent environment within few human efforts, Decision Company, and covers a wide range of practical scenarios.

*   •

    We evaluate popular LLM agents on Tapilot-Crossing, highlighting the challenges of interactive data analysis and the need for more advanced LLM agents.

*   •

    We propose AIR, an effective and efficient reflection strategy that significantly improves the performance of LLM agents in interactive data analysis tasks.

## 2 Preliminaries

![Refer to caption](img/a4db563d645a0cca93a4927c5d942d69.png)

Figure 2: This figure provides an overview of action types in Tapilot-Crossing, illustrated by examples. We emphasize the keywords specific to each category, and demonstrate the relevant sections of the associated queries, as well as the agent actions. The number of ![Refer to caption](img/f03f4f5e31a3f1b0bb5539086a425d9b.png) symbols represents the relative difficulty of each action.

### 2.1 Interactive Data analysis

The task of interactive data analysis with LLM agents involves a user and an LLM agent engaging in a sequence of user-agent turns, denoted as $[(u_{1},a_{1}),(u_{2},a_{2}),...,(u_{n},a_{n})]$, where $n$ is the number of turns in the dialog. Each user-agent turn is a tuple $(u,a)$, where $u$ is the user’s query and $a$ is the agent’s response. The user’s query $u$ can be a natural language instruction or a feedback to the user’s or agent’s previous response. The agent’s response $a$ can be a code snippet for data analysis or the correct answer chosen from a list of options provided with the question. Each dialog starts with the user’s initial query $u_{1}$ and ends with the agent’s final response $a_{n}$.

### 2.2 Agent Actions in Interactions

In Tapilot-Crossing, we summarize 6 common agent actions in interactions to deal with the user intents and the contexts as described in Figure [2](https://arxiv.org/html/2403.05307v1#S2.F2 "Figure 2 ‣ 2 Preliminaries ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). We denote the agent action set as $A$, which is a function of the user’s query $u$, the dialog history of the interaction $H$, and $T$ refers to the table contents. In this work, we mainly focus on data analysis of tabular data.

Formally, we represent the agent action set as $A=f_{\theta}(u,H,T)$, where $f_{\theta}$ refers to agent based on LLMs with parameter $\theta$. In Tapilot-Crossing, we evaluate performance of LLM agents separately in each Action mode of set $A$.

## 3 Tapilot-Crossing Dataset

![Refer to caption](img/1382c33cba2f72e3c78b927bee1690d7.png)

Figure 3: This describes the construction pipeline of Tapilot-Crossing by the AI Agent Sandbox Decision Company. ![Refer to caption](img/742b6484926ba0a7c9e1811948741e5f.png) denotes human intervention during construction. For a more detailed describing, please refer to Section [3](https://arxiv.org/html/2403.05307v1#S3 "3 Tapilot-Crossing Dataset ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents").

### 3.1 Dataset Construction.

The construction of Tapilot-Crossing is mainly based on the AI Agent Sandbox, Decision Company, as depicted in Figure [3](https://arxiv.org/html/2403.05307v1#S3.F3 "Figure 3 ‣ 3 Tapilot-Crossing Dataset ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). Decision Company is a multi-agent environment where four GPT-4 agents (Administrator, Client, Data Scientist and AI Chatbot) interact with each other to perform data analysis tasks. The construction process involves the following steps: Data Acquisition & Preprocessing, Client Persona Generation, Analysis Scenario Generation, Plan Discussion and Interaction Log Simulation. During these stages, human intervention may be required to correct errors or eliminate harmful or biased content. Ultimately, the prototype data is adapted to private and action settings.

#### Data Acquisition & Prepossessing.

The first step in the construction of Tapilot-Crossing is the acquisition and preprocessing of data. We collect 5 open-source tables from Kaggle ^*^**[https://www.kaggle.com/](https://www.kaggle.com/), a popular data sicence platform. These datasets span diverse domains, namely ATP Tennis, Credit Card, Fast Food, Laptop Price, and Melbourne Housing. Then the Administrator Agent will generate column meanings and value illustrations.

#### Client Persona Generation.

The construction of Tapilot-Crossing proceeds to the generation of client personas. These personas with specific tasks and topics related to the data are founded by the Administrator Agent. Each persona is defined by a Name, Location, Job, and Background with diverse range of interests and backgrounds.

#### Simulation of Analysis Scenarios.

Then, the Administrator Agent conducts interviews with each Client Agent to ask about their Scenario description, Scenario Name, and the Goal of using the dataset for the scenario. In the Tapilot-Crossing dataset, human annotators will interrupt here and select the most reasonable or interesting scenarios. This ensures that the scenarios included in the Tapilot-Crossing dataset are meaningful, not too general across different clients. For instance, in B.3 of Figure [3](https://arxiv.org/html/2403.05307v1#S3.F3 "Figure 3 ‣ 3 Tapilot-Crossing Dataset ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"), we select Court Condition Impact because Player Performance Analysis is too general and Sponsor Attraction requires too much additional information out of the table contents, which leads to too many unawserable questions.

#### Plan Discussion.

In this process, the Client Agent and Data Scientist Agent collaborate to convert the client’s requirements into a set of specific data analysis questions. Each question is provided by an expected result type, such as dataframes, lists, or various plot types, which helps reduce question ambiguity and ease the pressure on evaluation metrics (Yin et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib53); He et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib17); Zhang et al., [2023d](https://arxiv.org/html/2403.05307v1#bib.bib63)). The dialogue between the agents further refines the questions with specific conditions. For example, as depicted in Figure [3](https://arxiv.org/html/2403.05307v1#S3.F3 "Figure 3 ‣ 3 Tapilot-Crossing Dataset ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") B.4, the client Garcia’s question could be further elaborated on the basis of his following responses, making all questions more answerable. In particular, Agent Garcia, fully cognizant of his persona created in B.2, adds the condition grass, reflecting his London location. This implies that the role-playing aspect of the agent can be instrumental in generating a wider range of questions that are both diverse and reasonable (Li et al., [2024a](https://arxiv.org/html/2403.05307v1#bib.bib26); Park et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib36)).

#### Interaction Log Annotation.

Following the plan discussion, the interaction simulation phase begins. Here, the AI Chatbot Agent takes the lead, executing the data analysis plan agreed on during the previous stage. The Chatbot Agent interacts with the Data Scientist Agent to answer a series of questions defined in the plan by generating codes and analyzing returned results.

### 3.2 Human Calibration

While the Decision Company can generate a wealth of data analysis interactions in a zero-shot prompting manner, human intervention is indispensable to ensure the quality of the data set (Lu et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib35)). Our observations indicate that only 23.5% of the original codes produced by Chatbot Agent can be directly used as reference codes. Therefore, human experts, two PhD students, participate in each stage of the generation process to calibrate the errors and meaningless interactions. While human intervention is required, it is worth noting that modifying existing answers or codes is more efficient than creating them from scratch. We preserve all natural and meaningful interactions, both agent-to-agent and human-to-machine, throughout the action setting collection.

### 3.3 Private Lib Mode Evolution

Data analyst frequently relies on their private libraries (Zan et al., [2022](https://arxiv.org/html/2403.05307v1#bib.bib57)). These libraries, often tailored to their specific needs, allow for more efficient and customized data processing and analysis. Furthermore, generating code through user-defined packages can test the true semantic parsing abilities of agents rather than merely testing their memorization of standard syntax from libraries such as Pandas(Lai et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib24)). It also evaluates their ability to understand and implement custom functions, which is a crucial aspect of real-world data analysis. In this work, we prompt GPT-4 to autonomously convert prototype codes with pre-trained functions like Pandas or Numpy into private codes. The details can refer to Appendix [J.5](https://arxiv.org/html/2403.05307v1#A10.SS5 "J.5 Private Lib Evolution ‣ Appendix J Decision Company Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents").

## 4 Data Statistics & Metrics

### 4.1 Dataset Statistics

 | Statistic | Number |
| Total Interactions | 1024 |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) clear interactions | 284 |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) action interactions | 485 |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) private lib. interactions | 206 |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) private act. interactions | 49 |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) # of pirvate lib functions | 137 |
| Answer Types |  |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) # of code generation answers | 594 |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) # of multi-choice answers | 430 |
| Quality & Cost |  |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) inner-agreement | 93.64 |
| ![[Uncaptioned image]](img/d9ec1e6e0d4bcdfd89cb715af1abdada.png) total costs (USD) | 66.7 | 

Table 1: The statistics of Tapilot-Crossing.

 | Dataset | # Q &#124; # Intents | # Toks. / Q | # Toks. / Code | Code Type | Analysis | Multi-Turn | Private Lib | Multi-modal | Evaluation |
| HumanEval (Chen et al., [2021](https://arxiv.org/html/2403.05307v1#bib.bib2)) | 164 &#124; 164 | 0060.9 | 024.4 | ![[Uncaptioned image]](img/aa0023fbe73731a1ed1ebaeb745f5292.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | Test Cases |
| MBPP (Austin et al., [2021](https://arxiv.org/html/2403.05307v1#bib.bib1)) | 974 &#124; 974 | 0014.5 | 024.2 | ![[Uncaptioned image]](img/aa0023fbe73731a1ed1ebaeb745f5292.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | Test Cases |
| Spider (Yu et al., [2018](https://arxiv.org/html/2403.05307v1#bib.bib55)) | 1034 &#124; 1034 | 0012.4 | 018.3 | ![[Uncaptioned image]](img/8218d0fe15e25799ce47ba38725c1529.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | Acc + EM |
| BIRD (Li et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib29)) | 1534 &#124; 1534 | 0014.5 | 049.6 | ![[Uncaptioned image]](img/8218d0fe15e25799ce47ba38725c1529.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | Acc + VES |
| DS-1000 (Lai et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib24)) | 1000 &#124; 1000 | 0282.4 | 042.1 | ![[Uncaptioned image]](img/aa0023fbe73731a1ed1ebaeb745f5292.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | Test Cases + SFC |
| SparC (Yu et al., [2019b](https://arxiv.org/html/2403.05307v1#bib.bib56)) | 1203 &#124; 1203 | 009.4 | 026.3 | ![[Uncaptioned image]](img/8218d0fe15e25799ce47ba38725c1529.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | Acc |
| CoSQL (Yu et al., [2019a](https://arxiv.org/html/2403.05307v1#bib.bib54)) | 1008 &#124; 1008 | 0013.1 | 031.4 | ![[Uncaptioned image]](img/8218d0fe15e25799ce47ba38725c1529.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | Acc |
| ARCADE (Yin et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib53)) | 1066 &#124; 1066 | 0019.2 | 048.2 | ![[Uncaptioned image]](img/aa0023fbe73731a1ed1ebaeb745f5292.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | ![[Uncaptioned image]](img/3828ea6c1754b4301c04e6b83983c6e6.png) | Acc + Fuzzy |
| Tapilot-Crossing | 1024 &#124; 1176 | 0273.6 | 110.6 | ![[Uncaptioned image]](img/aa0023fbe73731a1ed1ebaeb745f5292.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | ![[Uncaptioned image]](img/56b0b1a18270b6b5bb74baba770ca46b.png) | Acc + AccR | 

Table 2: Comparison of Tapilot-Crossing to other data analysis datasets. The first 5 datasets are single-turn data analysis sets featuring both SQL and Python codes. The following 3 benchmarks are multi-turn or interactive data analysis datasets. Tapilot-Crossing represents a challenging dataset in data analysis with more comprehensive settings. ![[Uncaptioned image]](img/aa0023fbe73731a1ed1ebaeb745f5292.png) represents that the end code is Python. ![[Uncaptioned image]](img/8218d0fe15e25799ce47ba38725c1529.png) means the target code is SQL.

Table [1](https://arxiv.org/html/2403.05307v1#S4.T1 "Table 1 ‣ 4.1 Dataset Statistics ‣ 4 Data Statistics & Metrics ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") provides key statistics for our dataset, while Table [2](https://arxiv.org/html/2403.05307v1#S4.T2 "Table 2 ‣ 4.1 Dataset Statistics ‣ 4 Data Statistics & Metrics ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") offers a comparison between Tapilot-Crossing and other datasets related to data analysis or science. To ensure a fair comparison regarding question and code length, we utilize tiktoken to compute the number of tokens for each dataset. As shown in Table [2](https://arxiv.org/html/2403.05307v1#S4.T2 "Table 2 ‣ 4.1 Dataset Statistics ‣ 4 Data Statistics & Metrics ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"), Tapilot-Crossing includes the comprehensive evaluation settings across private library, multi-turn, and multi-modal interactions. Additionally, the complexity of this dataset, reflected in the extended lengths of questions and associated code snippets, is further amplified by the inclusion of multi-intent queries. These queries, encapsulating multiple intents within a single question, require a versatile array of computational strategies for effective handling. For example, the query, "Please provide mean, median, mode, range, and histogram plots for age, employment status, and credit history." demands both statistical computations and data visualization. Finally, despite Tapilot-Crossing comprising 1024 data analysis interactions, it only incurs a cost of 66.7 USD, making it an economical choice for dataset generation. The inner-agreement promises the high quality of the dataset.

### 4.2 Evaluation Metrics

#### Accuracy (Acc).

Acc is a thorough metric that evaluates ability of agents to generate code that executes correctly and to accurately answer multi-choice questions. It is defined as the proportion of instances where the predicted outputs match the expected reference output, across all evaluated tasks. For a given dataset with $N$ instances, where $C_{i}$ is the expected outcome (either execution result or correct answer) and $\hat{C}_{i}$ is the predicted outputs for the $i^{th}$ instance, Acc is calculated as follows:

|  | $\mathrm{Acc}=\frac{1}{N}\sum_{i=1}^{N}\mathbf{I}(C_{i}=\hat{C}_{i}),$ |  | (1) |

where $\mathbf{I}$ is an indicator function that returns $1$ if $C_{i}=\hat{C}_{i}$, and $0$ otherwise.

#### Acc with Private Lib Recall (AccR).

Recognizing the importance of accurately leveraging specific user-defined libraries in code generation, we extend Acc to include a recall-based adjustment for instances involving private libraries. This ensures that AccR not only evaluates the direct accuracy of code execution and question answering but also evaluates the inclusion and correct usage of private library functions. AccR can be computed as follows:

|  | $\mathrm{AccR}=\frac{1}{N}\sum_{i=1}^{N}\mathbf{I}(C_{i}=\hat{C}_{i})\cdot% \mathbf{R}(C_{i},\hat{C}_{i}),$ |  | (2) |

|  | $\mathbf{R}(C{i},\hat{C}{i})=\frac{&#124;\mathbf{F}(C{i})\cap\mathbf{F}(\hat{C}{i})&#124;% }{&#124;\mathbf{F}(C{i})&#124;},$ |  | (3) |

where $\mathbf{R}(C_{i},\hat{C}_{i})$ quantifies the recall rate of relevant library functions in the predicted code. $\mathbf{F}(C_{i})$ and $\mathbf{F}(\hat{C}_{i})$ denote the set of private library functions in the reference codes and the set actually utilized by agents in the predicted codes, respectively.

## 5 Evolving LLMs Towards Interactive Data Analysis Agents

In this section, we discuss our approach of equipping LLMs as data analysis agents with tools and reasoning. Then, we introduce our self-generated reflection strategy Air to enhancing their performance in interactive settings.

### 5.1 Toolkit

Our tool sets include an executor, a user simulator, and a chart-to-table converter. The executor provides an environment for models to observe real-time feedback on their intermediate code results (Xie et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib46); Wang et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib43)). The user simulator (Wang et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib43); Yan et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib49)), powered by GPT-4-Turbo, tests the agents’ ability to generate code after clarifying details when facing under-specific questions. The chart-to-table (Liu et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib33)) converter mitigates the prevalent issue of LLMs’ inability to comprehend plots by converting them into tables. Detailed descriptions of these tool sets are provided in the Appendix [E.1](https://arxiv.org/html/2403.05307v1#A5.SS1 "E.1 Toolkit ‣ Appendix E Agent Implementation ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents").

### 5.2 Reasoning

Reasoning is a critical process in transitioning LLMs into data analysis agents (Huang and Chang, [2023](https://arxiv.org/html/2403.05307v1#bib.bib19)). In Tapilot-Crossing, we incorporate two primary reasoning methods for code generation and multiple-choice answers. The first is the Chain-of-Thought (COT) prompting technique (Wei et al., [2022](https://arxiv.org/html/2403.05307v1#bib.bib45)), which enhances the complex reasoning abilities of LLMs by dividing the reasoning path into multiple steps. The second method is Reasoning & Action (ReAct), which enables models to make decisions by generating reasoning traces and actions in an interleaved manner, inducing writing codes, executing and understanding results, and make decision based on analysis (Yao et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib51)).

### 5.3 Adapative Interaction Reflection (Air)

Successful interactions are important since they encapsulate the logic necessary to meet user requirements and ensure correct steps of analysis or code generation. Motivated by this, we propose an Adaptive Interaction Reflection (Air) approach to enable data analysis agents to learn from successful user-code histories with two steps.

#### Pseudo Code Logic Generation.

First, given the last previous history $(\mathbf{u}_{t-1};\mathbf{a}_{t-1})$, when $t>1$, we prompt the data analysis agent to reflect and generate its underlying logic $\mathbf{m}_{t-1}=f_{\theta}(\mathbf{u}_{t-1};\mathbf{a}_{t-1})$, where $f_{\theta}$ refers to agent based on LLMs with parameter $\theta$. And $(x;y)$ represents two elements $x$ and $y$ are concatenated in the prompt. In our work, we consider pseudo code as $\mathbf{m}$ since it is an intermediate logics between natural language queries and codes.

#### Re-Org One-Shot Reasoning.

Second, we re-organize them into a self-generated one-shot example with the order: $\mathbf{p}_{t-1}=(\mathbf{u}_{t-1};\{\mathbf{m}_{t-1};\mathbf{a}_{t-1}\})$, which represents the scenario where the input $\mathbf{u}_{t-1}$ is given, the agent should generate a logic $\mathbf{m}_{t-1}$ first, generate the answers $\mathbf{a}_{t-1}$. Finally, data analysis agent can learn from $\mathbf{p}_{t-1}$ to first generate logic $\mathbf{m}_{t}=f_{\theta}(\mathbf{p}_{t-1};\mathbf{u}_{t})$ and generate answer $\mathbf{a}_{t}=f_{\theta}(\mathbf{u}_{t};\mathbf{m}_{t})$ in the current turn $t$. When $t=1$, we keep the same reasoning method of the original agent. Figure [7](https://arxiv.org/html/2403.05307v1#A4.F7 "Figure 7 ‣ Appendix D Air Implementation ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") provides a detailed example.

 | Model |  | Interaction Mode | Result Type | Overall |
| Normal | Action | Private | Pri-Act | Code | Choice |
| Code-LLama-34B | Model Base | 27.5 | 18.7 | 2.4 | 0.0 | 15.0 | 19.8 | 16.7 |
| Agent | 18.5 | 22.3 | 1.0 | 0.0 | 9.9 | 24.4 | 15.2 |
| Inter-Agent | 28.8 | 22.9 | 2.1 | 0.2 | 15.7 | 24.8 | 19.2 |
| Claude-2.1 | Model Base | 20.2 | 16.8 | 1.5 | 4.5 | 11.4 | 17.9 | 13.7 |
| Agent | 23.4 | 18.0 | 3.9 | 0.0 | 13.6 | 19.1 | 15.6 |
| Inter-Agent | 24.8 | 18.0 | 5.1 | 0.6 | 15.1 | 18.2 | 16.4 |
| GPT-4-Turbo | Model Base | 27.6 | 17.5 | 5.3 | 4.3 | 17.8 | 16.1 | 17.2 |
| Agent | 29.1 | 21.7 | 10.1 | 3.6 | 20.2 | 20.6 | 20.4 |
| Inter-Agent | 29.2 | 22.8 | 12.0 | 7.9 | 20.6 | 23.1 | 21.5 |
| GPT-4-32k | Model Base | 29.7 | 24.2 | 7.1 | 0.0 | 17.8 | 25.4 | 20.9 |
| Agent | 23.4 | 39.2 | 9.1 | 5.3 | 16.6 | 38.8 | 25.9 |
| Inter-Agent | 32.2 | 41.3 | 10.6 | 9.8 | 21.6 | 42.1 | 30.2 | 

Table 3: Overall results of LLMs in base, agent, and inter-agent modes on the Tapilot-Crossing dataset. Pri-Act refers to private library + action evaluation mode.

## 6 Experiment

We introduce experiment setup in Section [6.1](https://arxiv.org/html/2403.05307v1#S6.SS1 "6.1 Experiment Setup ‣ 6 Experiment ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"), experimental results in Section [6.2](https://arxiv.org/html/2403.05307v1#S6.SS2.SSS0.Px1 "Overall Results. ‣ 6.2 Experimental Results ‣ 6 Experiment ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"), and analysis in Section [6.3](https://arxiv.org/html/2403.05307v1#S6.SS3 "6.3 Private Mode Analysis ‣ 6 Experiment ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") and [6.4](https://arxiv.org/html/2403.05307v1#S6.SS4 "6.4 Model Shortcoming Analysis ‣ 6 Experiment ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents").

### 6.1 Experiment Setup

#### Models.

Our experiments primarily involve popular Large Language Models (LLMs) that are capable of generating code and following complex human instructions since this is a basic in data analysis. Therefore, we investigate performance of GPT-4-Turbo^†^††gpt-4-1106-preview, GPT-4-32k, Claude-2.1 and CodeLlama-34B ^‡^‡‡codellama-34b-instruct-hf (Roziere et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib40)).

#### Implementation details.

Each LLM is implemented with three settings. 1) Model-Base refers to the LLM itself without any tool calls and reasonings. 2) Agent mode involves tool usage and reasoning. We employ zero-shot COT for guiding the LLM in code generation tasks since it can allow us to test the pure code generation ability of agents in data analysis. For multi-choice question answering, we utilize one-shot ReAct. 3) Inter-Agent mode incorporates Air as described in Section [5.3](https://arxiv.org/html/2403.05307v1#S5.SS3 "5.3 Adapative Interaction Reflection (Air) ‣ 5 Evolving LLMs Towards Interactive Data Analysis Agents ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") beyond the agent. Each model is provided with the last up to 5 turns of user-code histories. Further details can be found in Appendix [F](https://arxiv.org/html/2403.05307v1#A6 "Appendix F Implementation Details ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). For implementation of Private settings, we follow (Li et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib30); Zan et al., [2022](https://arxiv.org/html/2403.05307v1#bib.bib57)) to prompt agents to retrieve private libraries first then generate the code with retrieved packages.

### 6.2 Experimental Results

![Refer to caption](img/bd153aa4694e03108fb60282cd5d06d8.png)

Figure 4: Visualization of the performance of GPT-4-32k across various categories in Action Mode. It includes a comparative analysis of the base, agent, and inter_agent versions.

#### Overall Results.

Table [3](https://arxiv.org/html/2403.05307v1#S5.T3 "Table 3 ‣ Re-Org One-Shot Reasoning. ‣ 5.3 Adapative Interaction Reflection (Air) ‣ 5 Evolving LLMs Towards Interactive Data Analysis Agents ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") illustrates the comprehensive performance of all LLM agents and their base models on the Tapilot-Crossing dataset. From the results, we can deduce the following: 1) Most models with Agent version outperform their base version, highlighting the crucial role of tools and reasoning in enhancing the performance of Language Learning Models (LLMs) under complex tasks (Liu et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib34); Xie et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib46)). 2) All models exhibit obvious improvements in the Inter-Agent mode with Air. This indicates that the underlying logic of successful interaction histories is instrumental in guiding LLMs to become more proficient data analysis agents in interactive settings. 3) Despite GPT-4-Turbo’s performance being nearly on par with GPT-4 in code generation, its overall performance still falls short of GPT-4\. This indicates that beyond code writing, understanding results, and analysis are equally important. Fortunately, the comprehensive settings of Tapilot-Crossing can assist users in selecting models for data analysis tasks. 4) It’s superising to see exceptional performance of CodeLlama in the Normal code generation setting. We observe that CodeLlama frequently defines functions automatically and applies these in the following code, thereby improving readability and logic. This is particularly beneficial in tasks related to data-analysis code generation. Such tasks often require the composition of API functions, which demands a profound understanding of the context and the capability to extract common patterns into reusable functions. By defining and reusing symbolic functions, CodeLlama can streamline complex contexts, making them more logical, which is an advantage for resolving complex tasks (Gu et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib13)).

#### Fine-Grained Results on Action Modes.

![Refer to caption](img/86b3718e36a200b821e8628fd8c35727.png)

Figure 5: Visualization of the performance of LLMs on Normal, Private, Private w/ Recall. The Scores of first two modes are Acc. The last score is AccR.

Figure [4](https://arxiv.org/html/2403.05307v1#S6.F4 "Figure 4 ‣ 6.2 Experimental Results ‣ 6 Experiment ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") provides a comparative evaluation of three GPT-4 model variants across various Action modes, as detailed in Section [2.2](https://arxiv.org/html/2403.05307v1#S2.SS2 "2.2 Agent Actions in Interactions ‣ 2 Preliminaries ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). The interactive data analysis agent, Inter-Agent, obviously outperforms in most areas, especially in managing Fast_Fail queries and executing Update_Code actions. However, it falls short in the Best_Guess action compared to the Agent. We note that Air tends to make agents overly tractable in re-org one-shot example $\mathbf{p}_{t-1}$ and current generated logics $\mathbf{m}_{t}$. If $\mathbf{p}_{t-1}$ and $\mathbf{m}_{t}$ do not contain instructions on making assumptions, agents tend to select None of Above. This observation suggests that excessive reliance on historical data may hinder the inherent ability of models to conjecture based on the instant user behaviors. Therefore, striking a trade-off between user-code history exploration and real-time user interaction, especially facing under-specific questions is crucial for improving the performance of LLM agents in interactive settings.

### 6.3 Private Mode Analysis

#### Overall Results.

Table [3](https://arxiv.org/html/2403.05307v1#S5.T3 "Table 3 ‣ Re-Org One-Shot Reasoning. ‣ 5.3 Adapative Interaction Reflection (Air) ‣ 5 Evolving LLMs Towards Interactive Data Analysis Agents ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") and Figure [5](https://arxiv.org/html/2403.05307v1#S6.F5 "Figure 5 ‣ Fine-Grained Results on Action Modes. ‣ 6.2 Experimental Results ‣ 6 Experiment ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") indicates that the Private setting presents a considerable obstacle, with the best performing GPT-4-Turbo Inter-Agent only achieving 12%. This demonstrates that understanding and implementing user-specific functions is a critical and urgent skill for LLM agents in real-world data analysis tasks (Zan et al., [2022](https://arxiv.org/html/2403.05307v1#bib.bib57)).

#### The Critical Role of Function Relative Recall.

Notably, CodeLlama outperforms GPT-4-Turbo in Acc within the Private setting. However, its performance declines significantly relative to GPT-4-Turbo upon the consideration of private library relative recall in the generated codes, as measured by the AccR metric. This observation suggests that CodeLlama tends to reply less on user-defined private functions, aiming to reduce risk of code errors. Therefore, AccR metric can spotlight the balance required between proficient code generation and the meticulous integration of user-specified private libraries to foster safer and more satisfying code production.

### 6.4 Model Shortcoming Analysis

#### Long-Context Challenges.

The challenge of handling long-contexts is considerable in Tapilot-Crossing, especially for models with shorter maximum input lengths. Models such as Codellama-34B, which has a maximum input length of 16k, are particularly affected. For example, it is essential for LLMs to access all private function descriptions and codes for effective code generation with retrieved functions. The statistics shows that the average number of prompt tokens for Private is 15.7k, and notably, 20.4% of their prompts surpass the 16k length.

#### Instruction Following.

Our experiments reveal that GPT-4-32k requires minimal efforts in prompt design due to their exceptional ability to follow human instructions. To be specific, only 3.4% of their results deviate from the provided instructions. However, other models exhibit a higher proportion of unexpected result types. For instance, extracting generated codes or answers from Claude-2.1 proves to be extremely challenging since it often embeds the answer in the middle of outputs rather than at the end as defined. We also observe that GPT-4-Turbo tends to generate longer codes in any settings. While this characteristic enhances its performance in code generation, it also results in 60.3% of the code generated during ReAct reasoning being non-executable, thereby leading to incorrect answers. Furthermore, CodeLlama-34B-Instruct exhibits a lack of robustness when faced with longer or more complex prompts. With the addition of COT, the performance of CodeLlama significantly drops from 27.5% with simpler instructions to 18.5% in Normal code generation.

## 7 Related Work

#### Large Language Models for Data Analysis.

The use of LLMs for data analysis has been a topic of interest in recent years. LLMs powered by In-Context Learning (Yang et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib50); Dai et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib6); Dong et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib10)) have been employed in various data analysis tasks, such as SQL query generation (Pourreza and Rafiei, [2024a](https://arxiv.org/html/2403.05307v1#bib.bib37); Gao et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib12); Lei et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib25); Zhang et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib61); Gu et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib14); Wang et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib42); Pourreza and Rafiei, [2024b](https://arxiv.org/html/2403.05307v1#bib.bib38); Li et al., [2024b](https://arxiv.org/html/2403.05307v1#bib.bib27)), pandas or python code generation (Jain et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib22); Chen et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib4), [2023a](https://arxiv.org/html/2403.05307v1#bib.bib3); Li et al., [2024c](https://arxiv.org/html/2403.05307v1#bib.bib28); Zha et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib59); Zhang et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib60); Zheng et al., [2024b](https://arxiv.org/html/2403.05307v1#bib.bib65)), and data visualization (Chen et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib5); Huang et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib20)). However, most of these works focus on single-turn setting, where the user’s query is explicit and does not require any interaction or clarification. Recently, there has been a growing interest in interactive data analysis, where the user intents may need to be clarified or refined through interactive communication (De Vries et al., [2020](https://arxiv.org/html/2403.05307v1#bib.bib7); Yan et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib49); Wang et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib43)).

#### Data Analysis Benchmarks.

The development of benchmarks for data analysis tasks has been a crucial factor in driving the progress of LLMs in data science. Existing benchmarks can be broadly categorized into single-turn and multi-turn benchmarks. Single-turn benchmarks, such as HumanEval (Chen et al., [2021](https://arxiv.org/html/2403.05307v1#bib.bib2)), MBPP (Austin et al., [2021](https://arxiv.org/html/2403.05307v1#bib.bib1)), Spider (Yu et al., [2018](https://arxiv.org/html/2403.05307v1#bib.bib55)), BIRD (Li et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib29)), Text2Analysis (He et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib17)), DABench (Hu et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib18)) and DS-1000 (Lai et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib24)), focus on generating code snippets or closed-form insight summaries for data analysis given a single user query. To explore interactive nature of real-world data analysis scenarios, where the user’s intent may need to be clarified or refined through interactive communication, several multi-turn benchmarks have been proposed, including CoSQL (Yu et al., [2019a](https://arxiv.org/html/2403.05307v1#bib.bib54)), and ARCADE (Yin et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib53)). However, these benchmarks are primarily focused on code generation and do not cover other aspects of data analysis, such as data visualization and understanding based on intermediate results. Our work extends the existing literature by introducing a new benchmark, Tapilot-Crossing, for evaluating LLM agents in interactive data analysis tasks across wide range of data analysis settings.

#### Multi-Agent Environments for Data Generation.

LLMs have proven to be effective in constructing multi-agent environments for automatic data generation. For instance, Lu et al. ([2023](https://arxiv.org/html/2403.05307v1#bib.bib35)) and Ding et al. ([2023](https://arxiv.org/html/2403.05307v1#bib.bib9)) simulate dialogs for QA and text generation tasks. Also Li et al. ([2023b](https://arxiv.org/html/2403.05307v1#bib.bib30)) generates data about API calls using multi-agent environments. This is because LLM agents can simulate believable human actions when placed in an environment with dynamically updating knowledge and memory (Park et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib36)). Inspired by this, we also created Decision Company to generate interaction log data for data analysis with more believable behaviors. Unlike most of previous work on training dataset generation, our research pioneers the construction of the interactive benchmark with a specific focus on interactive data analysis agent evaluation.

## 8 Conclusion

We introduce Tapilot-Crossing, a new benchmark for evaluating LLM agents in interactive data analysis tasks. Tapilot-Crossing is constructed using a cost-effective multi-agent environment, Decision Company, and covers a wide range of practical scenarios. We evaluate data analysis agents based on popular LLMs on Tapilot-Crossing, highlighting the challenges of interactive data analysis and the need for more advanced interactive data analysis agents. We also propose AIR, an effective reflection strategy for interactive data analysis agent evolution. Our experiments demonstrate that AIR can obviously enhance the performance of LLM agents.

## 9 Limitations

#### Dataset Limitations.

1) The Tapilot-Crossing dataset assumes that all human-machine interaction history is clean and correct. However, in real-world scenarios, the interaction history is often not clean, and may contain noise or require multi-turn clarifications for a single question. Therefore, future work should consider a more realistic, noisy interaction benchmark in data analysis. It is also worth noting that even with a clean history, the most capable model, GPT-4-32k, only achieves a score of 30.2 in the Inter-Agent mode. 2) The creation of our dataset is both cost-effective and efficient; however, the evaluation phase demands considerable efforts due to the inherent complexity and unpredictability of data analysis questions. Given it is challenging to discern subtle differences in performance among data analysis agents, especially in the context of long-form code generation and execution accuracy. This difficulty is exacerbated by the fact that the execution results of code with a single error (i.e., one-line error) and a completely incorrect code (just one-line output) are also determined as 0\. Therefore, a soft-metric evaluation system should be introduced in the future as Appendix [H.5](https://arxiv.org/html/2403.05307v1#A8.SS5 "H.5 Code Similarity Equivalance (CSE) ‣ Appendix H Evaluation Metric Details ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). It would improve our ability to accurately gauge how close an answer is to the expected output, even when the executed output is zero, thereby providing a more fine-grained observation of code generation capabilities. 3) Finally, our work only concentrate on tabular data based analysis, while in the future, we would like to involve relational database (RDB)-based analysis with the programming language of SQLs.

#### Method Limitations.

Our proposed reflection strategy can make LLMs more effective interactive data analysis agent, but relies heavily on the accuracy of previous interactions. This reliance becomes less reliable in instances where the historical dialogue is cluttered with errors, suggesting the need for retrieval-augmented tools or methods to identify successful past interactions. Additionally, this strategy does not enhance agent performance in initial interactions due to the absence of historical data. Finally, while effective in many Action settings, this focus on interaction history may limit the inferential capabilities of Large Language Models (LLMs) by prioritizing past interactions over present context. Future efforts will be directed towards refining this approach to better balance the benefits of leveraging historical interactions against the need to maintain or enhance the inferential capabilities of LLMs.

#### Model Limitations.

In this work, we only test four popular and advanced models in comprehensive settings. Many key open-sourced models actually show limited human instruction following capability on our datasets leading to very poor performance on reasoning. Specifically, as these models are mainly trained on code, they often fail to adequately follow user instructions, as illustrated in section [6.4](https://arxiv.org/html/2403.05307v1#S6.SS4 "6.4 Model Shortcoming Analysis ‣ 6 Experiment ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). These models simply return the instruction itself and continue to generate meaningless following dialogue when the resources and instructions are more complex. Therefore, how to plan more weaker LLMs to become effective interactive data analysis agents would be one of our important future work.

## 10 Ethical Statement

The application of Large Language Models (LLMs) for automatic data generation requires a rigorous examination of ethical implications. The primary concern is the potential for LLMs to generate contents that could be considered harmful or biased. To mitigate these risks, human annotators (two PhD students) already filter and fix all problematic cases in Section [3.2](https://arxiv.org/html/2403.05307v1#S3.SS2 "3.2 Human Calibration ‣ 3 Tapilot-Crossing Dataset ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). Also, LLMs may disseminate private or sensitive information. Therefore, we employ anonymization techniques wherein personal identifiers are systematically altered. For example, the name strings are replaced randomly, and any information of personas are switched as well. And the geographical locations of John Smith will be replaced with locations of Carlos Garcia to prevent any linkage to real-world individuals or entities. These procedures are conducted in Section [3](https://arxiv.org/html/2403.05307v1#S3 "3 Tapilot-Crossing Dataset ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). Moreover, we are committed to ensuring that the outputs generated by our LLM, referred to as Tapilot-Crossing, are free from political or sexual biases. To this end, each output, including conclusions and generated responses, is rigorously reviewed by the authors. In a nutshell, our ethical framework is built on a foundation of transparency, accountability, and a proactive stance towards mitigating any ethical concerns associated with the use of LLMs. The measures we have implemented reflect our commitment to upholding the highest standards of ethical research practice with LLMs.

## References

*   Austin et al. (2021) Jacob Austin, Augustus Odena, Maxwell Nye, Maarten Bosma, Henryk Michalewski, David Dohan, Ellen Jiang, Carrie Cai, Michael Terry, Quoc Le, et al. 2021. Program synthesis with large language models. *arXiv preprint arXiv:2108.07732*.
*   Chen et al. (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, et al. 2021. Evaluating large language models trained on code. *arXiv preprint arXiv:2107.03374*.
*   Chen et al. (2023a) Xinyun Chen, Renat Aksitov, Uri Alon, Jie Ren, Kefan Xiao, Pengcheng Yin, Sushant Prakash, Charles Sutton, Xuezhi Wang, and Denny Zhou. 2023a. Universal self-consistency for large language model generation. *arXiv preprint arXiv:2311.17311*.
*   Chen et al. (2024) Xinyun Chen, Maxwell Lin, Nathanael Schärli, and Denny Zhou. 2024. Teaching large language models to self-debug. In *The Twelfth International Conference on Learning Representations*.
*   Chen et al. (2023b) Zhutian Chen, Chenyang Zhang, Qianwen Wang, Jakob Troidl, Simon Warchol, Johanna Beyer, Nils Gehlenborg, and Hanspeter Pfister. 2023b. Beyond generating code: Evaluating gpt on a data visualization course. *arXiv preprint arXiv:2306.02914*.
*   Dai et al. (2023) Damai Dai, Yutao Sun, Li Dong, Yaru Hao, Shuming Ma, Zhifang Sui, and Furu Wei. 2023. Why can GPT learn in-context? language models secretly perform gradient descent as meta-optimizers. In *Findings of the Association for Computational Linguistics: ACL 2023*, pages 4005–4019, Toronto, Canada. Association for Computational Linguistics.
*   De Vries et al. (2020) Harm De Vries, Dzmitry Bahdanau, and Christopher Manning. 2020. Towards ecologically valid research on language user interfaces. *arXiv preprint arXiv:2007.14435*.
*   Deng et al. (2024) Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Sam Stevens, Boshi Wang, Huan Sun, and Yu Su. 2024. Mind2web: Towards a generalist agent for the web. *Advances in Neural Information Processing Systems*, 36.
*   Ding et al. (2023) Ning Ding, Yulin Chen, Bokai Xu, Yujia Qin, Shengding Hu, Zhiyuan Liu, Maosong Sun, and Bowen Zhou. 2023. Enhancing chat language models by scaling high-quality instructional conversations. In *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing, EMNLP 2023, Singapore, December 6-10, 2023*, pages 3029–3051\. Association for Computational Linguistics.
*   Dong et al. (2023) Qingxiu Dong, Lei Li, Damai Dai, Ce Zheng, Zhiyong Wu, Baobao Chang, Xu Sun, Jingjing Xu, and Zhifang Sui. 2023. A survey for in-context learning. *arXiv preprint arXiv:2301.00234*.
*   Fayyad et al. (1996) Usama M. Fayyad, Gregory Piatetsky-Shapiro, and Padhraic Smyth. 1996. From data mining to knowledge discovery in databases. *AI Mag.*, 17(3):37–54.
*   Gao et al. (2023) Dawei Gao, Haibin Wang, Yaliang Li, Xiuyu Sun, Yichen Qian, Bolin Ding, and Jingren Zhou. 2023. Text-to-sql empowered by large language models: A benchmark evaluation. *arXiv preprint arXiv:2308.15363*.
*   Gu et al. (2023) Yu Gu, Xiang Deng, and Yu Su. 2023. Don’t generate, discriminate: A proposal for grounding language models to real-world environments. In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), ACL 2023, Toronto, Canada, July 9-14, 2023*, pages 4928–4949\. Association for Computational Linguistics.
*   Gu et al. (2024) Yu Gu, Yiheng Shu, Hao Yu, Xiao Liu, Yuxiao Dong, Jie Tang, Jayanth Srinivasa, Hugo Latapie, and Yu Su. 2024. Middleware for llms: Tools are instrumental for language agents in complex environments. *arXiv preprint arXiv:2402.14672*.
*   Guo et al. (2021) Jiaqi Guo, Ziliang Si, Yu Wang, Qian Liu, Ming Fan, Jian-Guang Lou, Zijiang Yang, and Ting Liu. 2021. Chase: A large-scale and pragmatic chinese dataset for cross-database context-dependent text-to-sql. In *Proceedings of the 59th Annual Meeting of the Association for Computational Linguistics and the 11th International Joint Conference on Natural Language Processing, ACL/IJCNLP 2021, (Volume 1: Long Papers), Virtual Event, August 1-6, 2021*, pages 2316–2331\. Association for Computational Linguistics.
*   Han et al. (2011) Jiawei Han, Micheline Kamber, and Jian Pei. 2011. *Data Mining: Concepts and Techniques, 3rd edition*. Morgan Kaufmann.
*   He et al. (2023) Xinyi He, Mengyu Zhou, Xinrun Xu, Xiaojun Ma, Rui Ding, Lun Du, Yan Gao, Ran Jia, Xu Chen, Shi Han, et al. 2023. Text2analysis: A benchmark of table question answering with advanced data analysis and unclear queries. *arXiv preprint arXiv:2312.13671*.
*   Hu et al. (2024) Xueyu Hu, Ziyu Zhao, Shuang Wei, Ziwei Chai, Guoyin Wang, Xuwu Wang, Jing Su, Jingjing Xu, Ming Zhu, Yao Cheng, et al. 2024. Infiagent-dabench: Evaluating agents on data analysis tasks. *arXiv preprint arXiv:2401.05507*.
*   Huang and Chang (2023) Jie Huang and Kevin Chen-Chuan Chang. 2023. Towards reasoning in large language models: A survey. In *Findings of the Association for Computational Linguistics: ACL 2023, Toronto, Canada, July 9-14, 2023*, pages 1049–1065\. Association for Computational Linguistics.
*   Huang et al. (2023a) Kung-Hsiang Huang, Mingyang Zhou, Hou Pong Chan, Yi R Fung, Zhenhailong Wang, Lingyu Zhang, Shih-Fu Chang, and Heng Ji. 2023a. Do lvlms understand charts? analyzing and correcting factual errors in chart captioning. *arXiv preprint arXiv:2312.10160*.
*   Huang et al. (2023b) Yue Huang, Jiawen Shi, Yuan Li, Chenrui Fan, Siyuan Wu, Qihui Zhang, Yixin Liu, Pan Zhou, Yao Wan, Neil Zhenqiang Gong, et al. 2023b. Metatool benchmark for large language models: Deciding whether to use tools and which to use. *arXiv preprint arXiv:2310.03128*.
*   Jain et al. (2023) Naman Jain, Tianjun Zhang, Wei-Lin Chiang, Joseph E Gonzalez, Koushik Sen, and Ion Stoica. 2023. Llm-assisted code cleaning for training accurate code generators. *arXiv preprint arXiv:2311.14904*.
*   Khanbabaei et al. (2018) Mohammad Khanbabaei, Farzad Movahedi Sobhani, Mahmood Alborzi, and Reza Radfar. 2018. Developing an integrated framework for using data mining techniques and ontology concepts for process improvement. *J. Syst. Softw.*, 137:78–95.
*   Lai et al. (2023) Yuhang Lai, Chengxi Li, Yiming Wang, Tianyi Zhang, Ruiqi Zhong, Luke Zettlemoyer, Wen-Tau Yih, Daniel Fried, Sida I. Wang, and Tao Yu. 2023. DS-1000: A natural and reliable benchmark for data science code generation. In *International Conference on Machine Learning, ICML 2023, 23-29 July 2023, Honolulu, Hawaii, USA*, volume 202 of *Proceedings of Machine Learning Research*, pages 18319–18345\. PMLR.
*   Lei et al. (2023) Fangyu Lei, Qian Liu, Yiming Huang, Shizhu He, Jun Zhao, and Kang Liu. 2023. S3eval: A synthetic, scalable, systematic evaluation suite for large language models. *arXiv preprint arXiv:2310.15147*.
*   Li et al. (2024a) Guohao Li, Hasan Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. 2024a. Camel: Communicative agents for" mind" exploration of large language model society. *Advances in Neural Information Processing Systems*, 36.
*   Li et al. (2024b) Haoyang Li, Jing Zhang, Hanbing Liu, Ju Fan, Xiaokang Zhang, Jun Zhu, Renjie Wei, Hongyan Pan, Cuiping Li, and Hong Chen. 2024b. Codes: Towards building open-source language models for text-to-sql. *arXiv preprint arXiv:2402.16347*.
*   Li et al. (2024c) Hongxin Li, Jingran Su, Yuntao Chen, Qing Li, and ZHAO-XIANG ZHANG. 2024c. Sheetcopilot: Bringing software productivity to the next level through large language models. *Advances in Neural Information Processing Systems*, 36.
*   Li et al. (2023a) Jinyang Li, Binyuan Hui, GE QU, Jiaxi Yang, Binhua Li, Bowen Li, Bailin Wang, Bowen Qin, Ruiying Geng, Nan Huo, Xuanhe Zhou, Chenhao Ma, Guoliang Li, Kevin Chang, Fei Huang, Reynold Cheng, and Yongbin Li. 2023a. Can LLM already serve as a database interface? a BIg bench for large-scale database grounded text-to-SQLs. In *Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track*.
*   Li et al. (2023b) Minghao Li, Yingxiu Zhao, Bowen Yu, Feifan Song, Hangyu Li, Haiyang Yu, Zhoujun Li, Fei Huang, and Yongbin Li. 2023b. Api-bank: A comprehensive benchmark for tool-augmented llms. In *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing, EMNLP 2023, Singapore, December 6-10, 2023*, pages 3102–3116\. Association for Computational Linguistics.
*   Li et al. (2023c) Ruosen Li, Teerth Patel, and Xinya Du. 2023c. Prd: Peer rank and discussion improve large language model based evaluations. *arXiv preprint arXiv:2307.02762*.
*   Li et al. (2023d) Yunshui Li, Binyuan Hui, Xiaobo Xia, Jiaxi Yang, Min Yang, Lei Zhang, Shuzheng Si, Junhao Liu, Tongliang Liu, Fei Huang, et al. 2023d. One shot learning as instruction data prospector for large language models. *arXiv preprint arXiv:2312.10302*.
*   Liu et al. (2023a) Fangyu Liu, Julian Martin Eisenschlos, Francesco Piccinno, Syrine Krichene, Chenxi Pang, Kenton Lee, Mandar Joshi, Wenhu Chen, Nigel Collier, and Yasemin Altun. 2023a. Deplot: One-shot visual language reasoning by plot-to-table translation. In *Findings of the Association for Computational Linguistics: ACL 2023, Toronto, Canada, July 9-14, 2023*, pages 10381–10399\. Association for Computational Linguistics.
*   Liu et al. (2023b) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, et al. 2023b. Agentbench: Evaluating llms as agents. *arXiv preprint arXiv:2308.03688*.
*   Lu et al. (2023) Bo-Ru Lu, Nikita Haduong, Chia-Hsuan Lee, Zeqiu Wu, Hao Cheng, Paul Koester, Jean Utke, Tao Yu, Noah A Smith, and Mari Ostendorf. 2023. Dialgen: collaborative human-lm generated dialogues for improved understanding of human-human conversations. *arXiv preprint arXiv:2307.07047*.
*   Park et al. (2023) Joon Sung Park, Joseph C. O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. 2023. Generative agents: Interactive simulacra of human behavior. In *Proceedings of the 36th Annual ACM Symposium on User Interface Software and Technology, UIST 2023, San Francisco, CA, USA, 29 October 2023- 1 November 2023*, pages 2:1–2:22\. ACM.
*   Pourreza and Rafiei (2024a) Mohammadreza Pourreza and Davood Rafiei. 2024a. Din-sql: Decomposed in-context learning of text-to-sql with self-correction. *Advances in Neural Information Processing Systems*, 36.
*   Pourreza and Rafiei (2024b) Mohammadreza Pourreza and Davood Rafiei. 2024b. Dts-sql: Decomposed text-to-sql with small large language models. *arXiv preprint arXiv:2402.01117*.
*   Qin et al. (2023) Yujia Qin, Shengding Hu, Yankai Lin, Weize Chen, Ning Ding, Ganqu Cui, Zheni Zeng, Yufei Huang, Chaojun Xiao, Chi Han, et al. 2023. Tool learning with foundation models. *arXiv preprint arXiv:2304.08354*.
*   Roziere et al. (2023) Baptiste Roziere, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Tal Remez, Jérémy Rapin, et al. 2023. Code llama: Open foundation models for code. *arXiv preprint arXiv:2308.12950*.
*   Si et al. (2023) Shuzheng Si, Wentao Ma, Haoyu Gao, Yuchuan Wu, Ting-En Lin, Yinpei Dai, Hangyu Li, Rui Yan, Fei Huang, and Yongbin Li. 2023. SpokenWOZ: A large-scale speech-text benchmark for spoken task-oriented dialogue agents. In *Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track*.
*   Wang et al. (2023a) Bing Wang, Changyu Ren, Jian Yang, Xinnian Liang, Jiaqi Bai, Qian-Wen Zhang, Zhao Yan, and Zhoujun Li. 2023a. Mac-sql: Multi-agent collaboration for text-to-sql. *arXiv preprint arXiv:2312.11242*.
*   Wang et al. (2024) Xingyao Wang, Zihan Wang, Jiateng Liu, Yangyi Chen, Lifan Yuan, Hao Peng, and Heng Ji. 2024. MINT: Evaluating LLMs in multi-turn interaction with tools and language feedback. In *The Twelfth International Conference on Learning Representations*.
*   Wang et al. (2023b) Yue Wang, Hung Le, Akhilesh Deepak Gotmare, Nghi DQ Bui, Junnan Li, and Steven CH Hoi. 2023b. Codet5+: Open code large language models for code understanding and generation. *arXiv preprint arXiv:2305.07922*.
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting elicits reasoning in large language models. *Advances in Neural Information Processing Systems*, 35:24824–24837.
*   Xie et al. (2023) Tianbao Xie, Fan Zhou, Zhoujun Cheng, Peng Shi, Luoxuan Weng, Yitao Liu, Toh Jing Hua, Junning Zhao, Qian Liu, Che Liu, et al. 2023. Openagents: An open platform for language agents in the wild. *arXiv preprint arXiv:2310.10634*.
*   Xu et al. (2023a) Binfeng Xu, Xukun Liu, Hua Shen, Zeyu Han, Yuhan Li, Murong Yue, Zhiyuan Peng, Yuchen Liu, Ziyu Yao, and Dongkuan Xu. 2023a. Gentopia.ai: A collaborative platform for tool-augmented llms. In *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing, EMNLP 2023 - System Demonstrations, Singapore, December 6-10, 2023*, pages 237–245\. Association for Computational Linguistics.
*   Xu et al. (2023b) Yiheng Xu, Hongjin Su, Chen Xing, Boyu Mi, Qian Liu, Weijia Shi, Binyuan Hui, Fan Zhou, Yitao Liu, Tianbao Xie, et al. 2023b. Lemur: Harmonizing natural language and code for language agents. *arXiv preprint arXiv:2310.06830*.
*   Yan et al. (2023) Hao Yan, Saurabh Srivastava, Yintao Tai, Sida I. Wang, Wen-tau Yih, and Ziyu Yao. 2023. Learning to simulate natural language feedback for interactive semantic parsing. In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), ACL 2023, Toronto, Canada, July 9-14, 2023*, pages 3149–3170\. Association for Computational Linguistics.
*   Yang et al. (2023) Jiaxi Yang, Binyuan Hui, Min Yang, Binhua Li, Fei Huang, and Yongbin Li. 2023. Iterative forward tuning boosts in-context learning in language models. *arXiv preprint arXiv:2305.13016*.
*   Yao et al. (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R. Narasimhan, and Yuan Cao. 2023. React: Synergizing reasoning and acting in language models. In *The Eleventh International Conference on Learning Representations, ICLR 2023, Kigali, Rwanda, May 1-5, 2023*. OpenReview.net.
*   Yao et al. (2020) Ziyu Yao, Yiqi Tang, Wen-tau Yih, Huan Sun, and Yu Su. 2020. An imitation game for learning semantic parsers from user interaction. In *Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing, EMNLP 2020, Online, November 16-20, 2020*, pages 6883–6902\. Association for Computational Linguistics.
*   Yin et al. (2023) Pengcheng Yin, Wen-Ding Li, Kefan Xiao, Abhishek Rao, Yeming Wen, Kensen Shi, Joshua Howland, Paige Bailey, Michele Catasta, Henryk Michalewski, Oleksandr Polozov, and Charles Sutton. 2023. Natural language to code generation in interactive data science notebooks. In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), ACL 2023, Toronto, Canada, July 9-14, 2023*, pages 126–173\. Association for Computational Linguistics.
*   Yu et al. (2019a) Tao Yu, Rui Zhang, Heyang Er, Suyi Li, Eric Xue, Bo Pang, Xi Victoria Lin, Yi Chern Tan, Tianze Shi, Zihan Li, Youxuan Jiang, Michihiro Yasunaga, Sungrok Shim, Tao Chen, Alexander R. Fabbri, Zifan Li, Luyao Chen, Yuwen Zhang, Shreya Dixit, Vincent Zhang, Caiming Xiong, Richard Socher, Walter S. Lasecki, and Dragomir R. Radev. 2019a. Cosql: A conversational text-to-sql challenge towards cross-domain natural language interfaces to databases. In *Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing and the 9th International Joint Conference on Natural Language Processing, EMNLP-IJCNLP 2019, Hong Kong, China, November 3-7, 2019*, pages 1962–1979\. Association for Computational Linguistics.
*   Yu et al. (2018) Tao Yu, Rui Zhang, Kai Yang, Michihiro Yasunaga, Dongxu Wang, Zifan Li, James Ma, Irene Li, Qingning Yao, Shanelle Roman, Zilin Zhang, and Dragomir R. Radev. 2018. Spider: A large-scale human-labeled dataset for complex and cross-domain semantic parsing and text-to-sql task. In *Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing, Brussels, Belgium, October 31 - November 4, 2018*, pages 3911–3921\. Association for Computational Linguistics.
*   Yu et al. (2019b) Tao Yu, Rui Zhang, Michihiro Yasunaga, Yi Chern Tan, Xi Victoria Lin, Suyi Li, Heyang Er, Irene Li, Bo Pang, Tao Chen, Emily Ji, Shreya Dixit, David Proctor, Sungrok Shim, Jonathan Kraft, Vincent Zhang, Caiming Xiong, Richard Socher, and Dragomir R. Radev. 2019b. Sparc: Cross-domain semantic parsing in context. In *Proceedings of the 57th Conference of the Association for Computational Linguistics, ACL 2019, Florence, Italy, July 28- August 2, 2019, Volume 1: Long Papers*, pages 4511–4523\. Association for Computational Linguistics.
*   Zan et al. (2022) Daoguang Zan, Bei Chen, Zeqi Lin, Bei Guan, Yongji Wang, and Jian-Guang Lou. 2022. When language model meets private library. In *Findings of the Association for Computational Linguistics: EMNLP 2022, Abu Dhabi, United Arab Emirates, December 7-11, 2022*, pages 277–288\. Association for Computational Linguistics.
*   Zeng et al. (2023) Aohan Zeng, Mingdao Liu, Rui Lu, Bowen Wang, Xiao Liu, Yuxiao Dong, and Jie Tang. 2023. Agenttuning: Enabling generalized agent abilities for llms. *arXiv preprint arXiv:2310.12823*.
*   Zha et al. (2023) Liangyu Zha, Junlin Zhou, Liyao Li, Rui Wang, Qingyi Huang, Saisai Yang, Jing Yuan, Changbao Su, Xiang Li, Aofeng Su, et al. 2023. Tablegpt: Towards unifying tables, nature language and commands into one gpt. *arXiv preprint arXiv:2307.08674*.
*   Zhang et al. (2023a) Wenqi Zhang, Yongliang Shen, Weiming Lu, and Yueting Zhuang. 2023a. Data-copilot: Bridging billions of data and humans with autonomous workflow. *arXiv preprint arXiv:2306.07209*.
*   Zhang et al. (2023b) Yunjia Zhang, Jordan Henkel, Avrilia Floratou, Joyce Cahoon, Shaleen Deep, and Jignesh M Patel. 2023b. Reactable: Enhancing react for table question answering. *arXiv preprint arXiv:2310.00815*.
*   Zhang et al. (2023c) Zhehao Zhang, Xitao Li, Yan Gao, and Jian-Guang Lou. 2023c. CRT-QA: A dataset of complex reasoning question answering over tabular data. In *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing*, pages 2131–2153, Singapore. Association for Computational Linguistics.
*   Zhang et al. (2023d) Zhehao Zhang, Xitao Li, Yan Gao, and Jian-Guang Lou. 2023d. CRT-QA: A dataset of complex reasoning question answering over tabular data. In *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing, EMNLP 2023, Singapore, December 6-10, 2023*, pages 2131–2153\. Association for Computational Linguistics.
*   Zheng et al. (2024a) Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric Xing, et al. 2024a. Judging llm-as-a-judge with mt-bench and chatbot arena. *Advances in Neural Information Processing Systems*, 36.
*   Zheng et al. (2024b) Tianyu Zheng, Ge Zhang, Tianhao Shen, Xueling Liu, Bill Yuchen Lin, Jie Fu, Wenhu Chen, and Xiang Yue. 2024b. Opencodeinterpreter: Integrating code generation with execution and refinement. *arXiv preprint arXiv:2402.14658*.

## Appendix A License

### A.1 Tapilot-Crossing

Our Tapilot-Crossing data is available under the lisense CC BY NC 4.0.^§^§§[https://creativecommons.org/licenses/by-nc/4.0/](https://creativecommons.org/licenses/by-nc/4.0/)

### A.2 Kaggle Tabular Data

The tabular data that we used to create Tapilot-Crossing are following open-source licenses: 1) Public Domain: Public Domain Mark 2) CC-BY: Creative Commons Attribution 4.0 International.

## Appendix B Dynamic History Combination

### B.1 History Relational Database (H-RDB)

We split the User-AI interaction into several single-turn user queries and AI answers stored in a relational database, indexed by the conversational order as shown in figure [6](https://arxiv.org/html/2403.05307v1#A2.F6 "Figure 6 ‣ B.1 History Relational Database (H-RDB) ‣ Appendix B Dynamic History Combination ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"). This storage is subject to dynamic combinations for different scenarios.

![Refer to caption](img/8861d28407397d8110308dc2a7397299.png)

Figure 6: The screenshot of History Relational Database (H-RDB).

### B.2 History Retrieval Queries

When retrieving the stored history information, we use sqlite3^¶^¶¶[https://docs.python.org/3/library/sqlite3.html](https://docs.python.org/3/library/sqlite3.html) python package. The search query is provided in sqlite3 format, for example: SELECT {Prompt} FROM {table} WHERE 1=1 AND Domain = ? AND ...

### B.3 Tapilot-Alpha

As stated in Section [9](https://arxiv.org/html/2403.05307v1#S9 "9 Limitations ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents"), the current Tapilot-Crossing involves only clean and accurate code revisions, which we refer to as the Alpha version. Looking ahead, we are considering the incorporation of noisy data or the integration of user interactions in Action mode into the code history. This potential expansion aims to simulate more realistic development environments and challenges. Even within the constraints of a curated and error free interaction history, the experimental results show that there still are substantial opportunities for optimization and improvements.

## Appendix C Dialog Types

Tapilot-Crossing can be categorized into Statement- (longer) and Colloquial- (shorter) dialogs. The statement-dialogs are more formal, resulting in more complex user instructions and code generations, which are commonly found in computational notebooks (Yin et al., [2023](https://arxiv.org/html/2403.05307v1#bib.bib53)). On the other hand, colloquial dialogs involve shorter and simpler user questions, but exhibit more colloquial and interactive characteristics. This category of dialogs is primarily constructed through the process of prompting GPT-4 to segment and reinterpret the existing statement-dialogs.

## Appendix D Air Implementation

![Refer to caption](img/bf305e43f4f2b6b333d20da85af37fc3.png)

Figure 7: This is an overview of our proposed method, Air. The areas highlighted in purple represent results generated by the agents.

Figure [7](https://arxiv.org/html/2403.05307v1#A4.F7 "Figure 7 ‣ Appendix D Air Implementation ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") presents the detailed steps of Air.

## Appendix E Agent Implementation

### E.1 Toolkit

#### Executor

To get the execution results of code generated by LLMs, we adopt Python Executor exec() which is implemented in Python ^∥^∥∥[https://docs.python.org/3/library/functions.html#exec](https://docs.python.org/3/library/functions.html#exec), within a isolated Python environment. The output of the code execution, whether it be any return values, print statements, or error messages, is then captured by the Executor. This output is subsequently returned to the LMs, providing them with feedback on the results of their code generation to make a better next-step action or decision.

#### User Simulator

In addressing the clarification action type, LLMs are permitted to request clarification when they feel ambigous about conditions from user queires. Therefore, we employ GPT-4 Turbo to emulate the user’s question-answering behavior, considering that GPT-4 has been demonstrated to provide feedback of equivalent quality to human responses (Wang et al., [2024](https://arxiv.org/html/2403.05307v1#bib.bib43)).

#### Chart-to-Table

We employ deplot (Liu et al., [2023a](https://arxiv.org/html/2403.05307v1#bib.bib33)) to convert images into a table. Given the table, then LLMs can reason and answer the questions.

### E.2 Reasoning

#### COT

To evaluate the pure code generalization capability of data analysis, we restrict LLMs from executing code during generation. Therefore, we employ a zero-shot COT for the reasoning of the Agent mode. The key prompt to implement such COT is:
... write a step-by-step outline and then write the code:

#### ReAct

To evaluate analytical capabilities beyond mere code generation, we employ ReAct for multiple-choice questions. Specifically, we set the MAX STEP for ReAct reasoning to 5, with the Executor serving as the primary tool. Data analysis agents are tasked to generate, analyze, and draw conclusions about their results. If the result contains bugs, the corresponding message is returned to the agent for rectification, although this process may consume additional reasoning steps. We also manually provide a one-shot example to guide agents on how to react in Tapilot-Crossing. To prevent data leakage, we cross-reference examples across different tabular data. For instance, an example curated from ATP_Tennis could be used to guide LLMs in the Laptop Pricing dataset.

## Appendix F Implementation Details

### F.1 General Implementation

The temperature parameter is set to 0.0 for Claude 2.1, GPT-4, and GPT-4-Turbo.

## Appendix G Action Description

In this section, we categorize and formalize the action types in Tapilot-Crossing, identifying five distinct sub-categories that correspond to different types of user queries.

### G.1 Update_Code

The Update_Code action refers to instances where the user requests corrections for bugs or refinements to the conditions of previous queries.

### G.2 Fast_Fail

Fast_Fail is an action that alerts users when the current data contents or resources are insufficient to meet their requests, or when user queries contain factual errors.

### G.3 Clarification

Clarification is a common action in response to under-specified questions, which are frequent in data-analysis queries. In this action, agents make the conditions of the question more specific and clear by seeking additional information from users.

### G.4 Best_Guess

While Clarification is an effective action to reduce the uncertainty, it can lead to issues such as user impatience due to unsteadily asking, and long dialog histories that result in attention distraction and long-context problems. Therefore, the Best_Guess action can address these issues by making appropriate assumptions based on data contents, domain knowledge, and commonsense knowledge for under-specific questions. However, there is also a risk that incorrect guesses can lead to hallucinations.

### G.5 Plot_QA

In real data analysis settings, agents are also expected to answer user questions about insights derived from plots. The Plot_QA action can assist users in better understanding the contents of plots for decision making.

### G.6 Insight_Mining

Beyond generating codes for users to retrieve expected results, code agents are also tasked with summarizing executed results from the environment to assist users in making informed decisions. This process, known as Insight_Mining, plays an important role in data analysis since it contributes to the evolution of code agents into comprehensive data analysis agents.

## Appendix H Evaluation Metric Details

### H.1 DataFrame Comparison

The function compares two dataframes (df_1 and df_2) by checking their indices, column presence, and column data. It uses np.allclose() for numeric data and direct comparison for non-numeric data. If a column in df_1 is absent in the original dataframe, it searches for a matching column in df_2. The function returns True if df_1 and df_2 are equivalent, otherwise False. Please note, the column names will not be computed since different LLMs may have their only preference names. For example, the win_ratio generated by GPT-4 could be called winning ratio by Claude 2.1.

### H.2 Visualization Comparison

We note that it’s hard to compare the closed-form results for visualization-based code generation since parameters of plots may be varied significant across models. For instance, GPT-4 generated plots may be the same with CodeLlama while their title names may be different, which leads to false negatives. Therefore we utilize PIL package to compute similarity between plots. To be specific, the function compare_plots takes two image file paths as inputs (ai_output and reference_output), resizes them to 800x600 pixels using the LANCZOS method, and saves them. The images are then read in grayscale mode to avoid the difference brought by colors. The function computes and returns the Structural Similarity Index (SSIM), a measure of image similarity, between the two images. This function can be used to compare an AI model’s output with a reference output. Finally, the code generated will be considered as correct if the similarity is larger than 0.6.

### H.3 Multi-Intent Evaluation

In this work, we evaluate the code generation performance on intent manner, which means if one user query contains multiple intents, then the total scores of this query will be the number of intents. We evaluate each intent separately and sum up the scores of all intents as the denominator when calculate the performance of each model in percentage.

### H.4 Private Function Recall

We notice that some LLMs tend to import as many as possible private functions while not using all of them. Thus, to extract all indeed used private functions in the customized function library, we utilize AST package. After extracting the used private functions, we calculate the recall coefficient according to Equation [3](https://arxiv.org/html/2403.05307v1#S4.E3 "3 ‣ Acc with Private Lib Recall (AccR). ‣ 4.2 Evaluation Metrics ‣ 4 Data Statistics & Metrics ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents").

### H.5 Code Similarity Equivalance (CSE)

In the context of Tapilot-Crossing, the complexity of code generation tasks—many of which yield a score of zero—presents huge challenges in evaluating performance through Acc or AccR only. This is particularly evident when distinguishing between codes that differ by merely a single line of error or output, both of which would result in an Acc or AccR of zero, despite their obvious differences in code generation capabilities. To overcome this limitation, we propose the introduction of Code Similarity Equivalence (CSE), a nuanced evaluation metric designed to assess the similarity between generated codes and reference codes. Given that these codes originate based on identical user instructions, a high degree of similarity is expected. Our approach leverages a hybrid combination of models to reduce the bias, incorporating CodeT5++ and OpenAI Ada (text-embedding-ada-002) models, which are affordable and available for most institutes. This combination has demonstrated a strong correlation with human evaluative preferences, offering a more refined and accurate measure of code generation performance.

#### Details.

We introduce here about how to conduct more nuanced evaluation of Acc or AccR with CSE. 1) We collect 180 instances of code generation including both Normal and Private. To evaluation the quality of these codes, we enlist two additional PhD students who are proficient in data science and Python as evaluation committee.
2) They evaluate code generated by several models, including GPT-4-32k, GPT-4-Turbo, Claude-2.1, CodeLlama-Instruct (ranging from 7B to 34B parameters), StarCoder, and DeepSeek-Coder-Instruct (also from 7B to 34B parameters). Each evaluator is provided with comprehensive user code histories, tabular contents, the current query, access to the decision_company private library. Please note that evaluations are conducted only based on their expertise and experience, without any predefined guidelines and discussion, to avoid bias.
3) We ask for a relative ranking of generated codes among models over absolute scoring to avoid potential variability in scoring preferences among the evaluators.
4) In cases of parts of divergent rankings, the evaluators engage in discussions regarding the specific code samples until a consensus was reached. This step ensures a more reliable and agreed-upon evaluation outcome.
5) The evaluation committee then examine various open-source and readily available embedding models to measure code similarity, aiming to closely match their ranking preferences. Our exploration identifies that the score system consisting of CodeT5+ (Wang et al., [2023b](https://arxiv.org/html/2403.05307v1#bib.bib44)) and Ada (text-embedding-ada-002) most closely aligned with human evaluative preferences.

#### Introduction of a Mixed Evaluation Metric (AccSE & AccSER).

To accurately reflect the nuanced capabilities of code generation models, we propose a composite metric that integrates Code Similarity Evaluation (CSE) with Accuracy (Acc), termed Accuracy for Similarity Evaluation (AccSE). This metric is concisely defined as:

|  | $\text{AccSE}=\begin{cases}1.0,&\text{if }C=\hat{C},\\ 0.5,&\text{if }S_{1}>0.85\land S_{2}>0.9,\\ 0.25,&\text{if }(S_{1}>0.85\land S_{2}\leq 0.9)\\ &\lor(S_{1}\leq 0.85\land S_{2}>0.9),\\ 0,&\text{otherwise.}\end{cases}$ |  | (4) |

Where:

*   •

    $C$ and $\hat{C}$ represent the reference and generated code execution outcomes, respectively.

*   •

    $S_{1}$ denotes the CSE score based on CodeT5+.

*   •

    $S_{2}$ denotes the CSE score based on Ada.

This formulation succinctly captures the evaluation criteria for AccSE, with symbols $S_{1}$ and $S_{2}$ representing the CSE scores based on CodeT5+ and Ada, respectively. The logical operators $\land$ and $\lor$ are used for "and" and "or" conditions, respectively, to further compact the notation. AccSER is computed in the similar way just times recall score for each value as Eq. [3](https://arxiv.org/html/2403.05307v1#S4.E3 "3 ‣ Acc with Private Lib Recall (AccR). ‣ 4.2 Evaluation Metrics ‣ 4 Data Statistics & Metrics ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents").

We hold this for future evaluation system of Tapilot-Crossing when we conduct more extensive cases with involved with more expert volunteers.

#### Rationale Against GPT-4-Based and Multi-Agent Evaluation Methods.

While existing research suggests that GPT-4-based soft evaluation could enhance the assessment of complex generative tasks, such approaches are deemed unsuitable for Tapilot-Crossing due to several critical reasons:

1) Bias Concerns: The prototype annotations and questions in our study originate from a GPT-4-based agent environment. Employing GPT-4 for evaluation purposes could inadvertently introduce a self-enhancement bias (Zheng et al., [2024a](https://arxiv.org/html/2403.05307v1#bib.bib64)), compromising fairness across model evaluations.

2) Cost Concerns: Although multi-agent evaluation frameworks, incorporating diverse families of Large Language Models (LLMs), is to mitigate bias (Li et al., [2023c](https://arxiv.org/html/2403.05307v1#bib.bib31)), the economical and computational overhead is obvious. Specifically, evaluations in such settings require at least twice the token consumption than that used in generation alone, rendering it impractically expensive in Tapilot-Crossing.

Given these considerations, our research proposes an alternative evaluation methodology that is both cost-effective and reliable for evaluating the accuracy of complex data science code generation at this time. We demonstrate that CodeT5+, a remarkably efficient code embedding model, can obviously distinguish between varying performance levels and accurately identify correct code logic. Crucially, this model offers a pragmatic balance between evaluation thoroughness and resource efficiency.

### H.6 Other Value Types

For other result types, such as dictionry, set, list, we directly compute the exectued results and determine whether they are equal or not.

### H.7 Case-by-Case Evaluation

While we categorize instances according to result types and provide evaluation codes for each type, some scenarios requires a case-by-case evaluation script. For instance, in most dataframe or matrix comparisons, we employ np.close() and string match for result comparison. However, in some cases, such as using a dataframe or matrix to display a classifier’s Confusion Matrix, the predicted code is deemed correct if its f1-score surpasses that of the referenced code, even if their f1-scores are not similar. For the evaluation script of Tapilot-Crossing, we manually review and adjust the scripts to accommodate each case.

## Appendix I Action Evaluation Mode

### I.1 Correction

#### Update_Code

This could be evaluated within a static setting where the bug feedback is embedded into user-code history. Agents are requried to update the previous code via user feedback.

### I.2 Unawserable

#### Fast_Fail

In Decision Company, we keep the original unanswerable questions and categorize them as multi-choice questions. This is done to evaluate if agents can identify these questions based on their analysis of table contents and commonsense knowledge. To prevent any biased setting, such as specially designed prompts that might mislead agents into determining a question as unanswerable, we sample an equal number of under-specified problems and answerable questions. We then reformulate their choices, enabling the model to decide whether a question is answerable with clarification or assumption, or to directly classify it as unanswerable.

### I.3 Under_Specific

#### Clarification

To evaluate the performance of agents on clarification action, we employ a dynamic setting that incorporates a User Simulator. This simulator mimics user feedback based on the ground truth code or answer. Initially, interactive data analysis agents are expected to pose questions for clarification, simulator will answer it according to the ground truth answers. Subsequently, these agents are tasked with generating the final code, understanding both the original history and the history of clarifications. This setup provides a robust assessment of the agents’ ability to interact, clarify ambiguities, and generate accurate code.

#### Best_Guess

We aim to evaluate the ability of interactive data analysis agents to make accurate assumptions when faced with ambiguous questions, without resorting to constant clarification, which could potentially frustrate users. We believe that an agent’s best guess should not impact the final decision and this evaluation metric should be somehow flexible. For instance, in a credit card application scenario, the term young people could refer to individuals aged 20-40 or 25-45, making it challenging to be evaluated by fixed metrics. Therefore, we opt to use multiple-choice questions to assess the agents’ assumption-making capabilities. We posit that an assumption is appropriate only if it does not influence the final decision-making process.

### I.4 Visualization

#### Plot_QA

We evaluate the analysis capability of agents around plot in Tapilot-Crossing. The end format of answer would be multiple choices.

### I.5 Analysis

#### Insight_Mining

We evaluate the analysis capability of agents generally in Tapilot-Crossing. We opt to use multi-choice questions to evaluate it.

## Appendix J Decision Company Prompt

![Refer to caption](img/095e329bab18c1f68d5db1dd5999ee2b.png)

Figure 8: The prompt of Client Persona Generation

![Refer to caption](img/e6bc317a26b412c3eff664d1926056e2.png)

Figure 9: The prompt of analysis scenario generation

![Refer to caption](img/fb8830881b5d03ad1651cfa3aeb51354.png)

Figure 10: The example of plan discussion. The final output should be a plan of analysis invovlving questions and their or result types.

![Refer to caption](img/5ce78e4791021aa51e7229c59796c2c7.png)

Figure 11: The prompt of Data Science Agent in interaction log generation.

![Refer to caption](img/c7819bc4561f4296a97ce5f2af1835c7.png)

Figure 12: The prompt of Chatbot Agent in interaction log generation.

![Refer to caption](img/1512700ebaa5c3d1711367f003deef9c.png)

Figure 13: The prompt of Conversion from prototype code towards the code with private libraries.

### J.1 Client Persona Generation

The Figure [8](https://arxiv.org/html/2403.05307v1#A10.F8 "Figure 8 ‣ Appendix J Decision Company Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt Administrator agent to generate meaningful personas.

### J.2 Simulation of Analysis Scenarios

The Figure [9](https://arxiv.org/html/2403.05307v1#A10.F9 "Figure 9 ‣ Appendix J Decision Company Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") shows how we create diverse scenarios via In-Context Learning (ICL).

### J.3 Plan Discussion

The Figure [10](https://arxiv.org/html/2403.05307v1#A10.F10 "Figure 10 ‣ Appendix J Decision Company Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") presents how conversation between Data Scientist agent and Client agent can generate the series of analysis plans.

### J.4 Interaction Log Annotation

The prompt to drive interaction log annotation is presented by Figure [11](https://arxiv.org/html/2403.05307v1#A10.F11 "Figure 11 ‣ Appendix J Decision Company Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") and Figure [12](https://arxiv.org/html/2403.05307v1#A10.F12 "Figure 12 ‣ Appendix J Decision Company Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") from the view of Data Scientist agent and Chatbot agent, respectively.

### J.5 Private Lib Evolution

Figure [13](https://arxiv.org/html/2403.05307v1#A10.F13 "Figure 13 ‣ Appendix J Decision Company Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") shows the framework of how to prompt GPT-4 to generate code automatically. The human efforts are introduced to reduce the bias and fix errors.

## Appendix K Implementation Prompt

### K.1 Code Generation

The Figure [14](https://arxiv.org/html/2403.05307v1#A11.F14 "Figure 14 ‣ K.1 Code Generation ‣ Appendix K Implementation Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt LLM model to generate code to answer user queries. And Figure [15](https://arxiv.org/html/2403.05307v1#A11.F15 "Figure 15 ‣ K.1 Code Generation ‣ Appendix K Implementation Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt LLM in Agent to generate code to answer user queries following with chain-of-thought Wei et al. ([2022](https://arxiv.org/html/2403.05307v1#bib.bib45)). Finally, Figure [16](https://arxiv.org/html/2403.05307v1#A11.F16 "Figure 16 ‣ K.1 Code Generation ‣ Appendix K Implementation Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt LLM in Inter-Agent to generate code to answer user queries with our proposed AIR.

![Refer to caption](img/4d7604e02bc4bf1ea60ef0e034e5cb0e.png)

Figure 14: The prompt of LLM in Model-Base version in Code Generation mode.

![Refer to caption](img/6ac5e4c43bb28259aac45c51e196dc63.png)

Figure 15: The prompt of LLM with data analysis agent in Code Generation mode. The COT prompt text is in red color.

![Refer to caption](img/73fe26aa30b82b452fc0b4a7efd6113a.png)

Figure 16: The prompt of LLM with interactive data analysis agent in Code Generation mode. The AIR prompt text are in green color, which are generated by LLM itself by learning from successful history.

### K.2 Multi-choice

The Figure [17](https://arxiv.org/html/2403.05307v1#A11.F17 "Figure 17 ‣ K.2 Multi-choice ‣ Appendix K Implementation Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt LLM to answer user queries. And Figure [18](https://arxiv.org/html/2403.05307v1#A11.F18 "Figure 18 ‣ K.2 Multi-choice ‣ Appendix K Implementation Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt LLM in Agent to answer user queries following with ReAct Yao et al. ([2023](https://arxiv.org/html/2403.05307v1#bib.bib51)). Finally, Figure [19](https://arxiv.org/html/2403.05307v1#A11.F19 "Figure 19 ‣ K.2 Multi-choice ‣ Appendix K Implementation Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt LLM w/ Inter-Agent to answer user queries with our proposed AIR.

![Refer to caption](img/c764fde54aac92048311332485924fb1.png)

Figure 17: The prompt of LLM in Model-Base version in Multi-choice mode.

![Refer to caption](img/a4a93cb94bbb098356702d3d8b483861.png)

Figure 18: The prompt of LLM with data analysis agent in Multi-choice mode.

![Refer to caption](img/16f28131b3dbb6b5d91f8e6786a99bba.png)

Figure 19: The prompt of LLM with interactive data analysis agent in Multi-choice mode. The AIR prompt text are in green color. And the pseudocode is generated by LLM itself by learning from successful history

### K.3 Clarification Action

The Figure [20](https://arxiv.org/html/2403.05307v1#A11.F20 "Figure 20 ‣ K.3 Clarification Action ‣ Appendix K Implementation Prompt ‣ Tapilot-Crossing: Benchmarking and Evolving LLMs Towards Interactive Data Analysis Agents") describes how we prompt LLM in Model-Base version to ask for clarification.

![Refer to caption](img/1c897f28f2c63b4bf3444de981a8ae06.png)

Figure 20: The prompt of LLM in Model-Base version in clarification mode.