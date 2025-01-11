<!--yml
category: 未分类
date: 2025-01-11 12:09:24
-->

# Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents

> 来源：[https://arxiv.org/html/2410.05130/](https://arxiv.org/html/2410.05130/)

Yuwei Hu
Renmin University of China
huyuweiyisui@ruc.edu.cn &Runlin Lei
Renmin University of China
runlin_lei@ruc.edu.cn
&Xinyi Huang
Renmin University of China
2022201342@ruc.edu.cn
\ANDZhewei Wei
Renmin University of China
zhewei@ruc.edu.cn
&Yongchao Liu
Alibaba Group
yongchao.ly@antgroup.com    Yuwei Hu¹, Runlin Lei¹, Xinyi Huang¹, Zhewei Wei¹, Yongchao Liu²
¹Renmin University of China, Beijing, China
²Alibaba Group, Beijing, China
huyuweiyisui@ruc.edu.cn, runlin_lei@ruc.edu.cn, 2022201342@ruc.edu.cn
zhewei@ruc.edu.cn, yongchao.ly@antgroup.com Corresponding Author.

###### Abstract

Recent research has explored the use of Large Language Models (LLMs) for tackling complex graph reasoning tasks. However, due to the intricacies of graph structures and the inherent limitations of LLMs in handling long text, current approaches often fail to deliver satisfactory accuracy, even on small-scale graphs and simple tasks. To address these challenges, we introduce GraphAgent-Reasoner, a fine-tuning-free framework that utilizes a multi-agent collaboration strategy for explicit and precise graph reasoning. Inspired by distributed graph computation theory, our framework decomposes graph problems into smaller, node-centric tasks that are distributed among multiple agents. The agents collaborate to solve the overall problem, significantly reducing the amount of information and complexity handled by a single LLM, thus enhancing the accuracy of graph reasoning. By simply increasing the number of agents, GraphAgent-Reasoner can efficiently scale to accommodate larger graphs with over 1,000 nodes. Evaluated on the GraphInstruct dataset, our framework demonstrates near-perfect accuracy on polynomial-time graph reasoning tasks, significantly outperforming the best available models, both closed-source and fine-tuned open-source variants. Our framework also demonstrates the capability to handle real-world graph reasoning applications such as webpage importance analysis.

## 1 Introduction

Graphs, as a crucial data structure for modeling complex real-world relationships, are ubiquitous across various scenarios, e.g. citation networks, recommendation networks. Many important applications like drug discovery (Stokes et al., [2020](https://arxiv.org/html/2410.05130v1#bib.bib27)), traffic forecasting (Jiang & Luo, [2022](https://arxiv.org/html/2410.05130v1#bib.bib16)), and financial detection (Motie & Raahemi, [2024](https://arxiv.org/html/2410.05130v1#bib.bib23)), require reasoning over graphs to be realized. Noticing the powerful general knowledge and language processing capabilities of Large Language Models (LLMs) (Brown et al., [2020](https://arxiv.org/html/2410.05130v1#bib.bib1)), a significant amount of works have focused on using LLMs to perform various reasoning tasks, such as mathematical formula derivation (Meadows et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib20)), commonsense reasoning (Madaan et al., [2022](https://arxiv.org/html/2410.05130v1#bib.bib18)), and multi-hop question answering (Creswell et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib7)). However, most of them primarily involve shallow or sequential reasoning. To bring the LLM reasoning closer to human thinking, it is necessary for LLMs to master deeper and more complex reasoning, such as graph reasoning.

Despite significant efforts by researchers to enable LLMs to memorize, comprehend, and perform basic reasoning on graph structures, several issues still persist: 1) The scale of graphs that can be handled is limited. Describing graph structures in natural language inevitably leads to excessively long inputs. Due to context length limitations and the shortcomings of LLMs in handling lengthy text (Liu et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib17)), previous works (Chai et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib2); Fatemi et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib9); Perozzi et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib25)) could only handle graphs of very limited size (e.g. fewer than 20 nodes and 100 edges). 2) The performance on graph reasoning tasks is relatively poor. Unlike text, which can tolerate some degree of semantic deviation, reasoning and computation on graphs must be highly precise. However, current works demonstrate poor accuracy (average 20$\sim$60%) in various graph reasoning tasks like connectivity and shortest path. 3) Lacking explicit reasoning paths. Taking the shortest path as an example, the responses of existing models resemble a heuristic search approach to finding the shortest path on a graph, rather than strictly executing an algorithm. This makes it difficult to determine whether LLMs are genuinely deriving the answer through correct reasoning or merely making educated guesses. Although GraphWiz (Chen et al., [2024a](https://arxiv.org/html/2410.05130v1#bib.bib4)) attempts to generate explicit reasoning paths through fine-tuning, it often fails due to the presence of incomplete or wrong reasoning paths in its training data. Furthermore, GraphWiz exhibits overfitting, where it tends to treat new or unrelated questions as one of the fine-tuned problems, which will be detailed in Section [5.3](https://arxiv.org/html/2410.05130v1#S5.SS3 "5.3 Experiment 3: Case Study ‣ 5 Experiments ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

Motivation. The ultimate goal of graph reasoning is to enable LLMs to leverage graph-related knowledge or algorithms to solve real-world graph problems. However, with the development of information science and hardware storage, the scale of graphs and information per node become too large for a single LLM to handle. To address this, a natural idea is to use distributed approaches, where a large graph is stored across multiple LLMs separately and compute collaboratively. Therefore, just as graph algorithms have generally evolved from non-distributed to distributed forms (Meng et al., [2024b](https://arxiv.org/html/2410.05130v1#bib.bib22))), we hope that LLMs can also learn the concept of distributed processing, thereby harnessing the power of swarm intelligence to solve graph problems in real-world scenarios.

![Refer to caption](img/8b533b6211768f3072505ca9b27757e7.png)

Figure 1: The current situation of LLMs in solving graph problems. Previous methods using a single LLM often failed due to the complex graph structures. In contrast, our approach leverages agents collaboration to effectively address graph problems.

Our Contribution. To address the above limitations, in this paper, we propose the GraphAgent-Reasoner(GAR) framework, which leverages the power of swarm intelligence to solve graph reasoning problems, as shown in Figure [1](https://arxiv.org/html/2410.05130v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents"). We follow a node-centric approach, assigning an agent to each node, allowing it to focus on processing its own information and communicate with neighbors. Thus, we can easily scale up the size of graphs that can be processed by simply increasing the number of agents. At the same time, under the direction of a Master LLM, graph problems are decomposed into smaller, node-centric tasks, which are assigned to agents for collaborative resolution. This approach significantly reduces the scale and complexity of information each agent needs to process, thereby greatly improving the overall accuracy. Furthermore, since agents must clearly transmit the processed information to neighboring agents, the reasoning process becomes transparent, demonstrating the framework solves graph reasoning problems through clear and correct reasoning, rather than lucky guessing. In summary, our contributions are as follows:

*   •

    We propose GraphAgent-Reasoner, the first LLM-based multi-agents framework for graph reasoning, which requires no fine-tuning and can utilize any LLM as the underlying reasoning model. Our framework achieves near-perfect accuracy on various polynomial-time tasks, significantly surpassing the performance of existing methods.

*   •

    Our framework expands the scale of graph reasoning tasks handled by LLMs from 100 nodes to 1,000 nodes, demonstrating exceptional scalability. Furthermore, as the graph size increases, our framework does not exhibit the significant performance degradation seen in other methods and maintains robust accuracy.

*   •

    We explore the performance of our framework in real-world applications like webpage importance analysis, showcasing its potential for addressing complex graph reasoning problems in real-life situations.

## 2 Preliminaries and Related Works

Preliminaries. In general scenarios, when discussing LLMs solving graph reasoning problems, the input is a ($\mathcal{G}$,$\mathcal{Q}$) pair. $\mathcal{G}$ is a graph represented as $\mathcal{G}=(\mathcal{V},\mathcal{E},\{s_{i}\},\{t_{i}\})$, where $\mathcal{V}$ is the node set and $\mathcal{E}$, the edge set. For each node $v_{i}\in\mathcal{V}$, a sequential text node feature $s_{i}$ is associated; similarly, for each edge $e_{i}\in\mathcal{E}$, a sequential text edge feature $t_{i}$ is assigned. The graph $\mathcal{G}$ is described in natural language, typically using edge or adjacency list representation. $\mathcal{Q}$ is a task-specific instruction or problem description. LLMs will process the ($\mathcal{G}$,$\mathcal{Q}$) pair and return an answer string $A$.

Large Language Models for Graph Reasoning. To further enhance the reasoning capabilities of LLMs, many works have attempted to improve the performance of LLMs in graph reasoning. Wang et al. ([2023](https://arxiv.org/html/2410.05130v1#bib.bib30)) first introduces the NLGraph Benchmark to evaluate the performance of LLMs on various graph reasoning tasks. Fatemi et al. ([2024](https://arxiv.org/html/2410.05130v1#bib.bib9)) explores the impact of different graph encoding methods and graph structure types on the performance of LLMs in graph reasoning tasks. Additionally, it introduces another benchmark called GraphQA. Considering the lengthy nature of describing graph structures in text, Chai et al. ([2023](https://arxiv.org/html/2410.05130v1#bib.bib2)) and Perozzi et al. ([2024](https://arxiv.org/html/2410.05130v1#bib.bib25)) respectively use Transformers and GNNs to encode graph structures and attempt to align them with LLMs. Inspired by how humans understand structural information through the visual modality, Wei et al. ([2024](https://arxiv.org/html/2410.05130v1#bib.bib31)) generates corresponding visual images based on graph structures and provides them to visual LLMs for graph reasoning. Chen et al. ([2024a](https://arxiv.org/html/2410.05130v1#bib.bib4)) conducted Supervised Fine-Tuning and Directly Prefered Optimization on LLMs, enhancing the performance of LLMs and encouraging them to output explicit reasoning paths.

Large Language Model based Multi-Agents. Recent advancements in LLMs have spurred interest in their application within multi-agent systems. LLM-based multi-agent frameworks leverage the natural language understanding and reasoning capabilities of LLMs to enable agents to collaborate, communicate, and solve complex tasks in a distributed manner. Existing multi-agents works for problem solving primarily focuses on applications such as Software Development (Dong et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib8); Hong et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib13); Qian et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib26)), Embodied Agents (Zhang et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib35); Mandi et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib19); Chen et al., [2024b](https://arxiv.org/html/2410.05130v1#bib.bib5)) and Science Debate (Xiong et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib32); Chan et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib3)). However, using LLM-based multi-agents to handle graph data has been less explored, especially in the areas of graph reasoning and graph computation tasks. This may be due to the hallucination issue inherent in LLMs (Huang et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib15)), where their responses are factually incorrect. This problem becomes more complex in a multi-agent setting, as the hallucinations of a single agent may propagate to other nodes by communication (Guo et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib12)). This requires the performance of individual agents be sufficiently stable to ensure the correct operation of the entire multi-agent system.

## 3 Limitations of Single LLM in Graph Reasoning

Although LLMs exhibit strong language processing and logical reasoning capabilities, problems with the Transformer architecture and Attention mechanism (Vaswani et al., [2017](https://arxiv.org/html/2410.05130v1#bib.bib29)) still limit the scale and accuracy when they process graph problems. There are two primary limitations:

![Refer to caption](img/28dcc416b75ee00b1bba4c46f80082b9.png)

Figure 2: The performance of a single LLM in memorizing first-order neighboring nodes. As the number of nodes increases, all models exhibit significant memory errors.

The graph structure is too complex to memorize and understand for a single LLM. Using adjacency or edge lists to describe graph structures in natural language is the most intuitive and direct method, facilitating the processing of graph data by LLMs through text. However, this approach inevitably leads to a lengthy context, as the number of edges can grow quadratically with the number of nodes. As the graph scales up and becomes denser, the graph structure becomes highly complex, requiring a large amount of tokens to describe the edge relationships. When the text becomes too lengthy, it becomes difficult for LLMs to properly allocate attention, and they may even struggle with simple tasks such as key-value pair matching Liu et al. ([2023](https://arxiv.org/html/2410.05130v1#bib.bib17)). This presents significant challenges for LLMs in identifying key information for graph reasoning tasks from the lengthy context. Figure [2](https://arxiv.org/html/2410.05130v1#S3.F2 "Figure 2 ‣ 3 Limitations of Single LLM in Graph Reasoning ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents") shows the performance of a single LLM in memorizing one-hop neighbor nodes. We observe that as the number of nodes in the graph increases, various LLMs exhibit a significant decline in accuracy. If a single LLM cannot even correctly recall basic graph structural information like node neighbors, it becomes difficult to proceed with more complex graph reasoning or computation.

Furthermore, the graph structure is described in a sequential manner. LLMs have to identify implicit graph structures from sequential text. Since the processing of LLMs is a black-box operation, it is difficult to assert that they truly construct graph structures implicitly and thereby understand them. Huang et al. ([2024](https://arxiv.org/html/2410.05130v1#bib.bib14)) conducted extensive experiments to explore whether LLMs treat the input prompts as graphs or merely as paragraphs with keywords on TAGs. The results show that the performance of LLMs in handling TAGs primarily stems from the context rather than the graph structure. LLMs tend to process the graph description as linearized paragraphs rather than graphs.

A single LLM struggles to solve reasoning problems in real-world scenarios. Researchers train LLMs on graph reasoning tasks to empower them to utilize learned graph-related knowledge or algorithms to tackle real-world graph problems. However, in practical scenarios, the amount of information associated with each node can be enormous. Take citation networks as an example: a single node represents a paper, and its node information includes the title, abstract, and references, which could amount to several thousand tokens. In addition to the complexity of graph structures, the need to handle a large amount of node information further exacerbates the burden on a single LLM and highlights its shortcomings in processing long contexts. Moreover, using a single LLM to handle the entire network is inefficient, as it cannot coherently process the entire network’s problems. Typically, it is necessary to manually compress or summarize the information for each node and then feed local subgraphs to the LLM for processing (Guo et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib11); Chen et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib6)).

Furthermore, many current works (Chen et al., [2024a](https://arxiv.org/html/2410.05130v1#bib.bib4); Perozzi et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib25)) require training GNNs or fine-tuning LLMs on individual or multiple graph reasoning tasks. However, when transferring to other graph tasks, a certain degree of performance degradation occurs, and retraining or fine-tuning for new graph tasks consumes a significant amount of time and resources. Whether LLMs can apply the graph knowledge and algorithms learned during the training process to actual graph reasoning also remains an open question. We explored this question in [5.3](https://arxiv.org/html/2410.05130v1#S5.SS3 "5.3 Experiment 3: Case Study ‣ 5 Experiments ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents") and observed significant overfitting in LLMs fine-tuned on specific graph reasoning tasks. Therefore, the ideal solution would be to leverage the powerful general knowledge acquired during the pre-training phase of LLMs through an appropriate approach, enabling them to handle graph reasoning tasks as naturally as they do with natural language problems.

## 4 GraphAgent-Reasoner

To solve the limitations above, we propose a novel framework based on multi-agent collaboration called GraphAgent-Reasoner as shown in Figure [3](https://arxiv.org/html/2410.05130v1#S4.F3 "Figure 3 ‣ 4 GraphAgent-Reasoner ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents"), aiming to solve graph reasoning problems explicitly and correctly. The interface of the framework is a Master LLM, which is responsible for processing the textual input of graph problems, constructing the agent network, directing them to collaboratively solve the problem, and finally aggregating the states of all agents to derive the solution. Its implementation is based on the React Agent proposed by Yao et al. ([2023](https://arxiv.org/html/2410.05130v1#bib.bib34)), which is capable of reasoning based on the environment and executing corresponding actions, as detailed later. The pipeline of GAR consists of four steps: Graph Construction, Algorithm Establishing, Distributed Execution and Master Summarization.

![Refer to caption](img/2a43ee988f767586c518e0e7b69f85b9.png)

Figure 3: The framework of GraphAgent-Reasoner. Given a graph problem, the Master LLM will first construct agents network according to graph strcutures. It then sequentially performs Algorithm Establishing, Distributed Execution and Master Summarization, as detailed in this section.

Graph Construction. Given an input pair ($\mathcal{G}$, $\mathcal{Q}$), the Master LLM first extracts the node and edge information from the textual description of graph $\mathcal{G}$. It then constructs an agent for each node and initializes the node’s state and neighbor information, forming an interconnected network of agents. Each agent independently maintains its state and neighbor data, communicates with adjacent agents based on instructions from the Master LLM, and updates its state in each round.

Algorithm Establishing. To accommodate diverse graph tasks and fully exploit the knowledge embedded in LLMs during pre-training, we propose a unified solution approach framed within a distributed paradigm as shown in Algorithm [1](https://arxiv.org/html/2410.05130v1#alg1 "In 4 GraphAgent-Reasoner ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents"). This approach requires the Master LLM to specify six core components for each problem: State, Message, Initialization, Send, Update, and Termination.

*   •

    State: The local information maintained by each node, representing its current state. This can include attributes like node features, labels, or any other task-specific data. The states evolve as nodes receive messages and update their information.

*   •

    Message: The data transmitted between nodes during the communication phase. Messages typically contain information that neighboring nodes need to perform updates, such as feature values, distances, or other task-relevant information.

*   •

    Initialization: At the start of the execution, each node initializes its state with predefined values, which may be based on node IDs, input features or task-specific requirements. This step ensures that the graph is ready to begin the communication process.

*   •

    Send: After initialization, each node generates messages based on its current state and sends them to its neighboring nodes. This step is repeated in each iteration, allowing nodes to continuously exchange information with their neighbors.

*   •

    Update: Upon receiving messages from its neighbors, each node updates its state by aggregating the incoming messages and combining them with its current state. This iterative process enables nodes to refine their information over time.

*   •

    Termination: The algorithm halts when a predefined stopping condition is met, such as reaching a fixed number of iterations, achieving convergence, or satisfying a task-specific criterion. Once the termination condition is reached, each node will send its final state to the Master LLM, and the execution terminates.

1 Input: Agent Nodes $\mathcal{A}$, each agent $a\in\mathcal{A}$ maintains a state $S_{a}$, the maximum iterations $I_{max}$ given by the Master LLM.2Output: Final state $S_{a}$ for each agent $a\in\mathcal{A}$/* Initialization */3 Each agent $a\in\mathcal{A}$ initializes its state $S_{a}$ based on Initialization rules.4Each agent $a$ sends an initial message $M_{a\rightarrow v}$ to each of its neighbors $v\in\text{Neighbors}(a)$ based on its current state $S_{a}$ and Send rules./* Communication */5 while *Iteration $i$ < $I_{max}$ and Termination not met* do      a. /* Receive */6      Each agent $a$ receives messages $M_{u\rightarrow a}$ from all neighboring agents $u$.      b. /* Update */7      Each agent $a$ updates its state $S_{a}$ based on the received messages $M$ and its own current state $S_{a}$ according to Update rules.      c. /* Send */8      Each agent $a$ sends updated messages $M_{a\rightarrow v}$ to each of its neighbors $v$ based on the updated state $S_{a}$ according to Send rules.Return: the final state $S_{a}$ for all agents $a\in\mathcal{A}$

Algorithm 1 Distributed Paradigm

Since LLMs lack prior knowledge of this distributed paradigm, to facilitate the Master LLM’s understanding and application of the framework, we develop a distributed algorithm library that adheres to this distributed paradigm, from which the Master LLM can query relevant algorithm templates to generate distributed solutions within this paradigm. Specifically, we selected classic distributed graph algorithms and documented their implementations under this distributed paradigm. Some examples are presented in Appendix [A.1](https://arxiv.org/html/2410.05130v1#A1.SS1 "A.1 Example of distributed algorithms in distributed algorithm library ‣ Appendix A Distributed algorithms under the distributed paradigm ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents"). Drawing on prior work (Zheng et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib36); Meng et al., [2024a](https://arxiv.org/html/2410.05130v1#bib.bib21)), we endeavor to write detailed reasoning steps of each part in the algorithm to encourage the agent to think step by step as much as possible, which plays an important role in enhancing the success rate of individual agents.

When receiving a problem input, the Master LLM first retrieves the $k$ algorithms most relevant to the problem description from the distributed algorithm library. If there are algorithms suitable for handling the problem, the Master LLM will adjust the algorithm according to the problem description, such as changing the initialization and termination conditions (e.g., the source node in the shortest path problem). If there are no appropriate algorithms, the Master LLM will design a distributed algorithm following the distributed paradigm based on the examples of the retrieved algorithms. For some generated examples, see Appendix [A.2](https://arxiv.org/html/2410.05130v1#A1.SS2 "A.2 Example of distributed algorithms designed by the Master LLM ‣ Appendix A Distributed algorithms under the distributed paradigm ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

Distributed Execution. After the distributed algorithm is designed, the Master LLM will relay the approach to each agent node for execution according to the process outlined in Algorithm [1](https://arxiv.org/html/2410.05130v1#alg1 "In 4 GraphAgent-Reasoner ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents"). Each agent will first initialize its state based on node information and algorithm rules and then send an initial message to neighboring agents. Subsequently, each agent will iteratively execute the operations of receiving messages, updating its state, and sending messages according to the algorithm rules, synchronizing progress after each communication round. Communication will continue until the maximum number of iterations is reached or the termination condition is met.

Master Summarization. Finally, the final state of all agent nodes will be aggregated to the Master LLM, which will summarize the results conclude based on the problem and return the final answer in natural language form.

## 5 Experiments

In this section, we summarize the key experiments conducted with GAR. We begin by highlighting some of the most exciting results from our analysis here:

*   •

    R1: GAR achieves near-perfect accuracy on polynomial-time graph reasoning problems, significantly surpassing existing closed-source models and open-source models fine-tuned on extensive data.

*   •

    R2: GAR maintains high accuracy on larger-scale graphs (up to 1000 nodes), demonstrating superior scalability. In contrast, as the number of nodes increases, other models exhibit a significant decline in performance or become incapable of handling the problem at all due to the context length limitation.

*   •

    R3: GAR showcases a robust understanding and application of graph algorithms in real-world graph reasoning scenarios, highlighting its potential for addressing complex graph problems encountered in daily life. In contrast, other open-source models that have undergone extensive fine-tuning on graph reasoning datasets fail to apply the learned graph reasoning knowledge when confronted with rephrased real-world graph problems.

Datasets. We conduct our experiments on the graph reasoning tasks proposed in GraphInstruct (Chen et al., [2024a](https://arxiv.org/html/2410.05130v1#bib.bib4)). This dataset contains nine graph reasoning problems with different time complexity, ranging from linear and polynomial complexity to NP-complete.

*   •

    Linear. Cycle Detection (Detect if a given graph $\mathcal{G}$ contains any cycles), Connectivity (Assess if two nodes $u$ and $v$ in a given graph $\mathcal{G}$ are connected via a path), Bipartite Graph Check (Judge if a given graph $\mathcal{G}$ is bipartite), and Topological Sort (Find a topological ordering of vertices in a directed acyclic graph $\mathcal{G}$).

*   •

    Polynomial. Shortest Path (Compute the shortest path between two specific nodes $u$ and $v$ in a given graph $\mathcal{G}$), Maximum Triangle Sum (Find the maximum sum of weights for any connected triplet of vertices in a given graph $\mathcal{G}$), and Maximum Flow (Calculate the maximum flow from a source node $s$ to a sink node $t$ in a directed graph $\mathcal{G}$).

Due to the complexity of NP-complete problems, there are currently no mature exact distributed algorithms available for their solution. Consequently, the Master LLM is unable to design correct and effective distributed algorithms based on the knowledge acquired during pre-training. Therefore, in our experiments, we only consider linear and polynomial-time problems. Detailed information of the dataset and partial test results for NP-complete problems will be presented in Appendix [B](https://arxiv.org/html/2410.05130v1#A2 "Appendix B The GraphInstruct Dataset ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

Setting. The underlying reasoning LLM of Agent Node used in our framework is ChatGPT-4o-mini-2024-07-18, and the base model of Master LLM is ChatGPT-4-turbo(OpenAI, [2023](https://arxiv.org/html/2410.05130v1#bib.bib24)). The temperature is consistently set to 0\. Our framwork is built upon AgentScope (Gao et al. ([2024](https://arxiv.org/html/2410.05130v1#bib.bib10))), an innovative platform to easily build reliable, high-performance multi-agent applications.

### 5.1 Experiment 1: Performance on GraphInstruct

In this experiment, we evaluate the performance of GAR on polynomial-time tasks of the GraphInstruct dataset. The results are shown in Table [1](https://arxiv.org/html/2410.05130v1#S5.T1 "Table 1 ‣ 5.1 Experiment 1: Performance on GraphInstruct ‣ 5 Experiments ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents"). We see GAR exhibits near-perfect results on these tasks, significantly outperforming other models. Especially on shortest and triangle tasks with high time complexity, GAR substantially improves the performance of LLMs. Problems that a single LLM struggles to solve have been effectively resolved through collaboration by agents after being decomposed into smaller, node-centric tasks.

As the number of nodes increases, the graph structures become more complex, making the solution of graph problems increasingly difficult. To investigate how the performance of models varies with increasing problem complexity, we conduct experiments on cycle detection and shortest path problems, gradually increasing the number of nodes from 5 to 100\. The results are presented in Figure [4](https://arxiv.org/html/2410.05130v1#S5.F4 "Figure 4 ‣ 5.1 Experiment 1: Performance on GraphInstruct ‣ 5 Experiments ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

![Refer to caption](img/8ecbe9b151774b21b498c89fc7abfcfd.png)

(a) Cycle Detection

![Refer to caption](img/a6cb80e743825c03675218e724994c02.png)

(b) Shortest Path

Figure 4: Performance of GraphAgent-Reasoner, GPT4(2 shot) and GraphWiz(Mistral 7B) on cycle detection and shortest path problems with different graph sizes.

We see with the number of nodes increasing, both ChatGPT-4 and Graphwiz exhibit a significant decline in performance. However, the accuracy of GAR remains stable, almost unaffected by the graph size, demonstrating robust scalability. Although the scale of the graph is increasing, the information processed by each agent has not significantly increased. Each agent still only handles its own information and communicates with neighboring agents. We observe that GAR occasionally makes errors in specific cases, likely due to the increasing communication rounds as the number of nodes and edges grows. Even when handling simple node-centric tasks, a single agent still has the potential to make mistakes. Therefore, as the number of agents and communication rounds increases, the overall likelihood of errors also rises. This can be improved by enhancing the capability of individual agents (such as using stronger LLMs as the underlying reasoning model) or by more finely designed prompts.

Table 1: Performance of GraphAgent-Reasoner and other models on polynomial-time tasks of GraphInstruct test set. Each task contains 400 test cases, with a maximum of 100 nodes. The first best result for each task is highlighted in bold, and the second best result is highlighted underlined.

 | Models | Linear | Polynomial | Average |
| cycle | connect | bipartite | topology | shortest | triangle |
| Closed-source Models |  |  |  |  |  |  |  |
| GPT-4 (zero-shot) | 38.75 | 17.00 | 65.25 | 5.00 | 9.25 | 5.75 | 23.50 |
| GhatGPT (2-shot) | 51.25 | 43.75 | 70.75 | 4.50 | 3.50 | 17.25 | 31.83 |
| GPT-4 (2-shot) | 52.50 | 62.75 | 74.25 | 25.25 | 18.25 | 31.00 | 44.00 |
| Fine-tuned Open-source Models |  |  |  |  |  |  |  |
| Naive SFT (LLaMA 2-7B) | 73.75 | 83.50 | 41.25 | 4.00 | 9.50 | 30.00 | 40.17 |
| Naive SFT (Mistral-7B) | 73.75 | 83.50 | 78.50 | 1.00 | 23.00 | 47.00 | 51.13 |
| GraphWiz (LLaMA 2-7B) | 91.50 | 87.00 | 74.00 | 18.00 | 28.00 | 38.25 | 56.13 |
| GraphWiz (Mistral-7B) | 92.00 | 89.50 | 72.00 | 19.00 | 31.25 | 38.75 | 57.08 |
| GraphWiz-DPO (LLaMA 2-7B) | 89.00 | 82.50 | 84.75 | 46.75 | 24.00 | 52.75 | 63.29 |
| GraphWiz-DPO (Mistral-7B) | 85.50 | 79.50 | 85.50 | 85.25 | 12.50 | 29.00 | 62.88 |
| GraphAgent-Reasoner | 99.50 | 100.00 | 100.00 | 96.50 | 99.75 | 93.25 | 98.00 | 

### 5.2 Experiment 2: Performance on Large-Scale Graphs

In this experiment, we evaluate the performance of current LLMs on large-scale graphs. The largest graph size handled by existing graph reasoning work is 100 nodes (Chen et al., [2024a](https://arxiv.org/html/2410.05130v1#bib.bib4)), which is still far from sufficient for real-world graph reasoning scenarios. To evaluate the reasoning performance of existing models on larger graphs, we conduct shortest path experiments on graphs with 100, 200, 500, and 1000 nodes. Due to the excessively long input text (reaching 16,000 tokens for 1000 nodes) and the money cost, we only create 20 test samples for each graph size. The results are shown in Table [2](https://arxiv.org/html/2410.05130v1#S5.T2 "Table 2 ‣ 5.2 Experiment 2: Performance on Large-Scale Graphs ‣ 5 Experiments ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

Table 2: Performance on large-scale graphs dealing with shortest path problems. x/20 indicates that out of 20 test samples, x samples are correct. NA signifies that testing could not be conducted due to the fact that the context length limit is exceeded.

 | Graph Size | 100 | 200 | 500 | 1000 |
| --- | --- | --- | --- | --- |
| Graphwiz (LLaMA 2-7B) | 0/20 | 0/20 | NA | NA |
| Graphwiz (LLaMA 2-7B-DPO) | 0/20 | 0/20 | NA | NA |
| Chatgpt-3.5-turbo-16k | 0/20 | 0/20 | 0/20 | 0/20 |
| Chatgpt-4-32k | 0/20 | 1/20 | 0/20 | 0/20 |
| GraphAgent-Reasoner | 20/20 | 20/20 | 20/20 | 18/20 | 

We see the two GraphWiz models fine-tuned on the LLaMA2-7B (Touvron et al., [2023](https://arxiv.org/html/2410.05130v1#bib.bib28)) base model are unable to handle graphs with 500 or more nodes due to the context length limitation (the context length limit for Llama2 is 4096 tokens). Although ChatGPT-3.5-turbo-16k and ChatGPT-4-32k can manage longer contexts, they output wrong answers in almost all test samples, with only ChatGPT-4-32k being correct in one 200 nodes test sample. In contrast, GAR maintains a high accuracy in large-scale graph, only failed in two 1000-node test samples, further demonstrating its robust scalability.

### 5.3 Experiment 3: Case Study

In this experiment, we explore the application of two graph reasoning models, Graphwiz and GAR, in real-world graph reasoning scenarios. We present a case study of webpage importance analysis in Figure [5](https://arxiv.org/html/2410.05130v1#S5.F5 "Figure 5 ‣ 5.3 Experiment 3: Case Study ‣ 5 Experiments ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

![Refer to caption](img/fb875a7ec1321ab558548cc6ff832773.png)

Figure 5: The importance analysis in webpage network. While the GraphWiz fails due to incorrect graph assessments, GAR correctly uses the PageRank algorithm to identify nodes 16, 14, and 5 as the most important.

Although GraphWiz performed well on fine-tuned tasks, it exhibits severe overfitting when faced with real-world graph problems, failing to apply the graph reasoning knowledge learned during the fine-tuning phase. Since GraphWiz uses a consistent graph node description, the sentence "The nodes are numbered from 0 to …" appears across all datasets during the mixed-task instruction tuning. When the actual problem has nodes numbered from 1 to 20, it still assumes the existence of node 0\. As a result, both GraphWiz models first output that the graph has 21 nodes and an incorrect number of edges. Furthermore, neither of the two GraphWiz models recognizes that this is a problem associated with web page importance ranking. Instead, they approach it as the bipartite graph check or topological sort problems they had been fine-tuned on. Additionally, neither model generates an explicit and correct reasoning path. These observations indicate that there is still a significant gap between excelling in classic graph reasoning tasks and effectively solving real-world graph reasoning problems. In contrast, GAR correctly identifies that the problem should be solved using knowledge related to PageRank (Yang et al., [2024](https://arxiv.org/html/2410.05130v1#bib.bib33)) and designs an algorithm that adhered to the distributed paradigm (Note: the distributed algorithm library does not contain a PageRank algorithm template). GAR then assigns the algorithm to agent nodes for execution, ultimately obtaining the PageRank value for each node and arriving at the correct conclusion. Through the distributed paradigm, GAR effectively bridges the powerful knowledge learned by LLMs with the solving of real-world graph reasoning problems, which enables it to flexibly handle practical issues in a distributed manner. This case study demonstrates the feasibility of using GAR to solve real-world graph reasoning problems, indicating its substantial practical applicability and offering researchers and practitioners a powerful framework to address such tasks.

## 6 Conclusion

We first summarize three key issues faced by existing LLMs in graph reasoning tasks: limited graph scale, poor performance, and the lack of explicit reasoning paths. We then reflect on the limitations of a single LLM in addressing graph reasoning problems, such as the graph structures being too complex to memorize and understand and the overwhelming information in real-world graph reasoning scenarios. To address these challenges, we propose GraphAgent-Reasoner, a framework based on multi-agent collaboration to solve graph reasoning problems. This framework demonstrates superior accuracy and scalability, significantly surpassing existing closed-source and fine-tuned open-source models. Our experiments show its robust scalability, maintaining high accuracy on large graphs (up to 1,000 nodes). Our case study on webpage importance analysis further illustrates its capability to handle real-world graph reasoning problems. Future work will focus on designing more accurate and scalable LLM-based multi-agent graph reasoning frameworks, aiming to apply them to larger and more complex real-world reasoning scenarios.

## References

*   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, et al. Language models are few-shot learners. In Hugo Larochelle, Marc’Aurelio Ranzato, Raia Hadsell, Maria-Florina Balcan, and Hsuan-Tien Lin (eds.), *Advances in Neural Information Processing Systems 33: Annual Conference on Neural Information Processing Systems 2020, NeurIPS 2020, December 6-12, 2020, virtual*, 2020. URL [https://proceedings.neurips.cc/paper/2020/hash/1457c0d6bfcb4967418bfb8ac142f64a-Abstract.html](https://proceedings.neurips.cc/paper/2020/hash/1457c0d6bfcb4967418bfb8ac142f64a-Abstract.html).
*   Chai et al. (2023) Ziwei Chai, Tianjie Zhang, Liang Wu, Kaiqiao Han, Xiaohai Hu, Xuanwen Huang, and Yang Yang. Graphllm: Boosting graph reasoning ability of large language model. *CoRR*, abs/2310.05845, 2023. URL [https://doi.org/10.48550/arXiv.2310.05845](https://doi.org/10.48550/arXiv.2310.05845).
*   Chan et al. (2024) Chi-Min Chan, Weize Chen, Yusheng Su, Jianxuan Yu, Wei Xue, Shanghang Zhang, Jie Fu, and Zhiyuan Liu. Chateval: Towards better llm-based evaluators through multi-agent debate. In *The Twelfth International Conference on Learning Representations, ICLR 2024, Vienna, Austria, May 7-11, 2024*. OpenReview.net, 2024. URL [https://openreview.net/forum?id=FQepisCUWu](https://openreview.net/forum?id=FQepisCUWu).
*   Chen et al. (2024a) Nuo Chen, Yuhan Li, Jianheng Tang, and Jia Li. Graphwiz: An instruction-following language model for graph computational problems. In Ricardo Baeza-Yates and Francesco Bonchi (eds.), *Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, KDD 2024, Barcelona, Spain, August 25-29, 2024*, pp.  353–364\. ACM, 2024a. URL [https://doi.org/10.1145/3637528.3672010](https://doi.org/10.1145/3637528.3672010).
*   Chen et al. (2024b) Yongchao Chen, Jacob Arkin, Yang Zhang, Nicholas Roy, and Chuchu Fan. Scalable multi-robot collaboration with large language models: Centralized or decentralized systems? In *IEEE International Conference on Robotics and Automation, ICRA 2024, Yokohama, Japan, May 13-17, 2024*, pp.  4311–4317\. IEEE, 2024b. URL [https://doi.org/10.1109/ICRA57147.2024.10610676](https://doi.org/10.1109/ICRA57147.2024.10610676).
*   Chen et al. (2023) Zhikai Chen, Haitao Mao, Hang Li, Wei Jin, Hongzhi Wen, Xiaochi Wei, Shuaiqiang Wang, Dawei Yin, Wenqi Fan, Hui Liu, and Jiliang Tang. Exploring the potential of large language models (llms)in learning on graphs. *SIGKDD Explor.*, 25(2):42–61, 2023. URL [https://doi.org/10.1145/3655103.3655110](https://doi.org/10.1145/3655103.3655110).
*   Creswell et al. (2023) Antonia Creswell, Murray Shanahan, and Irina Higgins. Selection-inference: Exploiting large language models for interpretable logical reasoning. In *The Eleventh International Conference on Learning Representations, ICLR 2023, Kigali, Rwanda, May 1-5, 2023*. OpenReview.net, 2023. URL [https://openreview.net/forum?id=3Pf3Wg6o-A4](https://openreview.net/forum?id=3Pf3Wg6o-A4).
*   Dong et al. (2023) Yihong Dong, Xue Jiang, Zhi Jin, and Ge Li. Self-collaboration code generation via chatgpt. *CoRR*, abs/2304.07590, 2023. URL [https://doi.org/10.48550/arXiv.2304.07590](https://doi.org/10.48550/arXiv.2304.07590).
*   Fatemi et al. (2024) Bahare Fatemi, Jonathan Halcrow, and Bryan Perozzi. Talk like a graph: Encoding graphs for large language models. In *The Twelfth International Conference on Learning Representations, ICLR 2024, Vienna, Austria, May 7-11, 2024*. OpenReview.net, 2024. URL [https://openreview.net/forum?id=IuXR1CCrSi](https://openreview.net/forum?id=IuXR1CCrSi).
*   Gao et al. (2024) Dawei Gao, Zitao Li, Weirui Kuang, Xuchen Pan, Daoyuan Chen, Zhijian Ma, Bingchen Qian, Liuyi Yao, Lin Zhu, Chen Cheng, Hongzhu Shi, Yaliang Li, Bolin Ding, and Jingren Zhou. Agentscope: A flexible yet robust multi-agent platform. *CoRR*, abs/2402.14034, 2024. URL [https://doi.org/10.48550/arXiv.2402.14034](https://doi.org/10.48550/arXiv.2402.14034).
*   Guo et al. (2023) Jiayan Guo, Lun Du, and Hengyu Liu. Gpt4graph: Can large language models understand graph structured data ? an empirical evaluation and benchmarking. *CoRR*, abs/2305.15066, 2023. URL [https://doi.org/10.48550/arXiv.2305.15066](https://doi.org/10.48550/arXiv.2305.15066).
*   Guo et al. (2024) Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, and Xiangliang Zhang. Large language model based multi-agents: A survey of progress and challenges. *CoRR*, abs/2402.01680, 2024. URL [https://doi.org/10.48550/arXiv.2402.01680](https://doi.org/10.48550/arXiv.2402.01680).
*   Hong et al. (2024) Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, and Jürgen Schmidhuber. Metagpt: Meta programming for A multi-agent collaborative framework. In *The Twelfth International Conference on Learning Representations, ICLR 2024, Vienna, Austria, May 7-11, 2024*. OpenReview.net, 2024. URL [https://openreview.net/forum?id=VtmBAGCN7o](https://openreview.net/forum?id=VtmBAGCN7o).
*   Huang et al. (2024) Jin Huang, Xingjian Zhang, Qiaozhu Mei, and Jiaqi Ma. Can llms effectively leverage graph structural information through prompts, and why? *Trans. Mach. Learn. Res.*, 2024, 2024. URL [https://openreview.net/forum?id=L2jRavXRxs](https://openreview.net/forum?id=L2jRavXRxs).
*   Huang et al. (2023) Lei Huang, Weijiang Yu, Weitao Ma, Weihong Zhong, Zhangyin Feng, Haotian Wang, Qianglong Chen, Weihua Peng, Xiaocheng Feng, Bing Qin, and Ting Liu. A survey on hallucination in large language models: Principles, taxonomy, challenges, and open questions. *CoRR*, abs/2311.05232, 2023. URL [https://doi.org/10.48550/arXiv.2311.05232](https://doi.org/10.48550/arXiv.2311.05232).
*   Jiang & Luo (2022) Weiwei Jiang and Jiayun Luo. Graph neural network for traffic forecasting: A survey. *Expert Syst. Appl.*, 207:117921, 2022. URL [https://doi.org/10.1016/j.eswa.2022.117921](https://doi.org/10.1016/j.eswa.2022.117921).
*   Liu et al. (2023) Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, and Percy Liang. Lost in the middle: How language models use long contexts. *Transactions of the Association for Computational Linguistics*, 12:157–173, 2023. URL [https://api.semanticscholar.org/CorpusID:259360665](https://api.semanticscholar.org/CorpusID:259360665).
*   Madaan et al. (2022) Aman Madaan, Shuyan Zhou, Uri Alon, Yiming Yang, and Graham Neubig. Language models of code are few-shot commonsense learners. In Yoav Goldberg, Zornitsa Kozareva, and Yue Zhang (eds.), *Proceedings of the 2022 Conference on Empirical Methods in Natural Language Processing, EMNLP 2022, Abu Dhabi, United Arab Emirates, December 7-11, 2022*, pp.  1384–1403\. Association for Computational Linguistics, 2022. URL [https://doi.org/10.18653/v1/2022.emnlp-main.90](https://doi.org/10.18653/v1/2022.emnlp-main.90).
*   Mandi et al. (2024) Zhao Mandi, Shreeya Jain, and Shuran Song. Roco: Dialectic multi-robot collaboration with large language models. In *IEEE International Conference on Robotics and Automation, ICRA 2024, Yokohama, Japan, May 13-17, 2024*, pp.  286–299\. IEEE, 2024. URL [https://doi.org/10.1109/ICRA57147.2024.10610855](https://doi.org/10.1109/ICRA57147.2024.10610855).
*   Meadows et al. (2023) Jordan Meadows, Marco Valentino, and André Freitas. Generating mathematical derivations with large language models. *CoRR*, abs/2307.09998, 2023. URL [https://doi.org/10.48550/arXiv.2307.09998](https://doi.org/10.48550/arXiv.2307.09998).
*   Meng et al. (2024a) Lingkai Meng, Yu Shao, Long Yuan, Longbin Lai, Peng Cheng, Xue Li, Wenyuan Yu, Wenjie Zhang, Xuemin Lin, and Jingren Zhou. A survey of distributed graph algorithms on massive graphs. *CoRR*, abs/2404.06037, 2024a. URL [https://doi.org/10.48550/arXiv.2404.06037](https://doi.org/10.48550/arXiv.2404.06037).
*   Meng et al. (2024b) Lingkai Meng, Yu Shao, Long Yuan, Longbin Lai, Peng Cheng, Xue Li, Wenyuan Yu, Wenjie Zhang, Xuemin Lin, and Jingren Zhou. A survey of distributed graph algorithms on massive graphs. *CoRR*, abs/2404.06037, 2024b. URL [https://doi.org/10.48550/arXiv.2404.06037](https://doi.org/10.48550/arXiv.2404.06037).
*   Motie & Raahemi (2024) Soroor Motie and Bijan Raahemi. Financial fraud detection using graph neural networks: A systematic review. *Expert Syst. Appl.*, 240:122156, 2024. URL [https://doi.org/10.1016/j.eswa.2023.122156](https://doi.org/10.1016/j.eswa.2023.122156).
*   OpenAI (2023) OpenAI. GPT-4 technical report. *CoRR*, abs/2303.08774, 2023. URL [https://doi.org/10.48550/arXiv.2303.08774](https://doi.org/10.48550/arXiv.2303.08774).
*   Perozzi et al. (2024) Bryan Perozzi, Bahare Fatemi, Dustin Zelle, Anton Tsitsulin, Seyed Mehran Kazemi, Rami Al-Rfou, and Jonathan Halcrow. Let your graph do the talking: Encoding structured data for llms. *CoRR*, abs/2402.05862, 2024. URL [https://doi.org/10.48550/arXiv.2402.05862](https://doi.org/10.48550/arXiv.2402.05862).
*   Qian et al. (2024) Chen Qian, Wei Liu, Hongzhang Liu, Nuo Chen, Yufan Dang, Jiahao Li, Cheng Yang, Weize Chen, Yusheng Su, Xin Cong, Juyuan Xu, Dahai Li, Zhiyuan Liu, and Maosong Sun. Chatdev: Communicative agents for software development. In Lun-Wei Ku, Andre Martins, and Vivek Srikumar (eds.), *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), ACL 2024, Bangkok, Thailand, August 11-16, 2024*, pp.  15174–15186\. Association for Computational Linguistics, 2024. URL [https://doi.org/10.18653/v1/2024.acl-long.810](https://doi.org/10.18653/v1/2024.acl-long.810).
*   Stokes et al. (2020) Jonathan M Stokes, Kevin Yang, Kyle Swanson, Wengong Jin, Andres Cubillos-Ruiz, Nina M Donghia, Craig R MacNair, Shawn French, Lindsey A Carfrae, Zohar Bloom-Ackermann, et al. A deep learning approach to antibiotic discovery. *Cell*, 180(4):688–702, 2020.
*   Touvron et al. (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, et al. Llama 2: Open foundation and fine-tuned chat models. *CoRR*, abs/2307.09288, 2023. URL [https://doi.org/10.48550/arXiv.2307.09288](https://doi.org/10.48550/arXiv.2307.09288).
*   Vaswani et al. (2017) Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, and Illia Polosukhin. Attention is all you need. In Isabelle Guyon, Ulrike von Luxburg, Samy Bengio, Hanna M. Wallach, Rob Fergus, S. V. N. Vishwanathan, and Roman Garnett (eds.), *Advances in Neural Information Processing Systems 30: Annual Conference on Neural Information Processing Systems 2017, December 4-9, 2017, Long Beach, CA, USA*, pp.  5998–6008, 2017. URL [https://proceedings.neurips.cc/paper/2017/hash/3f5ee243547dee91fbd053c1c4a845aa-Abstract.html](https://proceedings.neurips.cc/paper/2017/hash/3f5ee243547dee91fbd053c1c4a845aa-Abstract.html).
*   Wang et al. (2023) Heng Wang, Shangbin Feng, Tianxing He, Zhaoxuan Tan, Xiaochuang Han, and Yulia Tsvetkov. Can language models solve graph problems in natural language? In Alice Oh, Tristan Naumann, Amir Globerson, Kate Saenko, Moritz Hardt, and Sergey Levine (eds.), *Advances in Neural Information Processing Systems 36: Annual Conference on Neural Information Processing Systems 2023, NeurIPS 2023, New Orleans, LA, USA, December 10 - 16, 2023*, 2023. URL [http://papers.nips.cc/paper_files/paper/2023/hash/622afc4edf2824a1b6aaf5afe153fa93-Abstract-Conference.html](http://papers.nips.cc/paper_files/paper/2023/hash/622afc4edf2824a1b6aaf5afe153fa93-Abstract-Conference.html).
*   Wei et al. (2024) Yanbin Wei, Shuai Fu, Weisen Jiang, James T. Kwok, and Yu Zhang. Gita: Graph to visual and textual integration for vision-language graph reasoning. 2024. URL [https://api.semanticscholar.org/CorpusID:267413180](https://api.semanticscholar.org/CorpusID:267413180).
*   Xiong et al. (2023) Kai Xiong, Xiao Ding, Yixin Cao, Ting Liu, and Bing Qin. Examining inter-consistency of large language models collaboration: An in-depth analysis via debate. In Houda Bouamor, Juan Pino, and Kalika Bali (eds.), *Findings of the Association for Computational Linguistics: EMNLP 2023, Singapore, December 6-10, 2023*, pp.  7572–7590\. Association for Computational Linguistics, 2023. URL [https://doi.org/10.18653/v1/2023.findings-emnlp.508](https://doi.org/10.18653/v1/2023.findings-emnlp.508).
*   Yang et al. (2024) Mingji Yang, Hanzhi Wang, Zhewei Wei, Sibo Wang, and Ji-Rong Wen. Efficient algorithms for personalized pagerank computation: A survey. *IEEE Trans. Knowl. Data Eng.*, 36(9):4582–4602, 2024. URL [https://doi.org/10.1109/TKDE.2024.3376000](https://doi.org/10.1109/TKDE.2024.3376000).
*   Yao et al. (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R. Narasimhan, and Yuan Cao. React: Synergizing reasoning and acting in language models. In *The Eleventh International Conference on Learning Representations, ICLR 2023, Kigali, Rwanda, May 1-5, 2023*. OpenReview.net, 2023. URL [https://openreview.net/forum?id=WE_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X).
*   Zhang et al. (2024) Hongxin Zhang, Weihua Du, Jiaming Shan, Qinhong Zhou, Yilun Du, Joshua B. Tenenbaum, Tianmin Shu, and Chuang Gan. Building cooperative embodied agents modularly with large language models. In *The Twelfth International Conference on Learning Representations, ICLR 2024, Vienna, Austria, May 7-11, 2024*. OpenReview.net, 2024. URL [https://openreview.net/forum?id=EnXJfQqy0K](https://openreview.net/forum?id=EnXJfQqy0K).
*   Zheng et al. (2024) Xin Zheng, Qiming Zhu, Hongyu Lin, Yaojie Lu, Xianpei Han, and Le Sun. Executing natural language-described algorithms with large language models: An investigation. In Nicoletta Calzolari, Min-Yen Kan, Véronique Hoste, Alessandro Lenci, Sakriani Sakti, and Nianwen Xue (eds.), *Proceedings of the 2024 Joint International Conference on Computational Linguistics, Language Resources and Evaluation, LREC/COLING 2024, 20-25 May, 2024, Torino, Italy*, pp.  6752–6837\. ELRA and ICCL, 2024. URL [https://aclanthology.org/2024.lrec-main.596](https://aclanthology.org/2024.lrec-main.596).

## Appendix A Distributed algorithms under the distributed paradigm

### A.1 Example of distributed algorithms in distributed algorithm library

Shortest Path: See Figure [6](https://arxiv.org/html/2410.05130v1#A1.F6 "Figure 6 ‣ A.1 Example of distributed algorithms in distributed algorithm library ‣ Appendix A Distributed algorithms under the distributed paradigm ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

Connectivity: See Figure [7](https://arxiv.org/html/2410.05130v1#A1.F7 "Figure 7 ‣ A.1 Example of distributed algorithms in distributed algorithm library ‣ Appendix A Distributed algorithms under the distributed paradigm ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

![Refer to caption](img/2ed14e1d84045c77d48d6b2c49b42c2a.png)

Figure 6: Distributed algorithm for shortest path problem under the distributed paradigm.

![Refer to caption](img/23431a7b84198f7878f1590c8279487b.png)

Figure 7: Distributed algorithm for connectivity problem under the distributed paradigm.

### A.2 Example of distributed algorithms designed by the Master LLM

PageRank: See Figure [8](https://arxiv.org/html/2410.05130v1#A1.F8 "Figure 8 ‣ A.2 Example of distributed algorithms designed by the Master LLM ‣ Appendix A Distributed algorithms under the distributed paradigm ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

Hamilton Path: See Figure [9](https://arxiv.org/html/2410.05130v1#A1.F9 "Figure 9 ‣ A.2 Example of distributed algorithms designed by the Master LLM ‣ Appendix A Distributed algorithms under the distributed paradigm ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

Subgraph Matching: See Figure [10](https://arxiv.org/html/2410.05130v1#A1.F10 "Figure 10 ‣ A.2 Example of distributed algorithms designed by the Master LLM ‣ Appendix A Distributed algorithms under the distributed paradigm ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents").

![Refer to caption](img/780c3adb9c08499e900dffeafb93ff82.png)

Figure 8: Distributed algorithm for pagerank calculation under the distributed paradigm.

![Refer to caption](img/be982bc1d562074ee287bb30c472ef83.png)

Figure 9: Distributed algorithm for hamilton path problem under the distributed paradigm.

![Refer to caption](img/f7f43a6ea46e83ff5e5d169895890f65.png)

Figure 10: Distributed algorithm for subgraph matching problem under the distributed paradigm.

## Appendix B The GraphInstruct Dataset

The statistics and detailed information of GraphInstruct are shown in Table [3](https://arxiv.org/html/2410.05130v1#A2.T3 "Table 3 ‣ Appendix B The GraphInstruct Dataset ‣ Scalable and Accurate Graph Reasoning with LLM-based Multi-Agents"). Hamilton Path and Subgraph Matching are NP-complete problems.

Table 3: The detailed information of GraphInstruct dataset.

 | Problem | Definition | Node Range | Test Size |
| --- | --- | --- | --- |
| Cycle Detection | Detect if a given graph $\mathcal{G}$ contains any cycles. | [2, 100] | 400 |
| Connectivity | Assess if two nodes $u$ and $v$ in a given graph $\mathcal{G}$ are connected via a path. | [2, 100] | 400 |
| Bipartite Graph Check | Judge if a given graph $\mathcal{G}$ is bipartite. | [2, 100] | 400 |
| Topological Sort | Find a topological ordering of vertices in a directed acyclic graph $\mathcal{G}$. | [2, 50] | 400 |
| Shortest Path | Compute the shortest path between two specific nodes $u$ and $v$ in a given graph $\mathcal{G}$. | [2, 100] | 400 |
| Maximum Triangle Sum | Find the maximum sum of weights for any connected triplet of vertices in a given graph $\mathcal{G}$. | [2, 25] | 400 |
| Maximum Flow | Calculate the maximum flow from a source node $s$ to a sink node $t$ in a directed graph $\mathcal{G}$. | [2, 50] | 400 |
| Hamilton Path | Determine if a given graph $\mathcal{G}$ has a Hamiltonian path that visits each vertex exactly once. | [2, 50] | 400 |
| Subgraph Matching | Verify if there exists a subgraph in $\mathcal{G}$ that is isomorphic to a given graph $\mathcal{G}^{\prime}$. | [2, 30] | 400 | 

Hamilton Path Execution Example.

Problem Description:

Determine whether or not there is a Hamiltonian path in an undirected graph. In an undirected graph, (i,j) means that node i and node j are connected with an undirected edge. Given a graph, you need to output Yes or No, indicating whether there is a Hamiltonian path in the graph. Q: The nodes are numbered from 0 to 5, and the edges are: (0, 3) (0, 1) (0, 2) (0, 4) (1, 5) (1, 4) (1, 2) (1, 3) (2, 4) (2, 5) (3, 5) (3, 4). Is there a Hamiltonian path in this graph?

Execution Process:

```
### State
1\. ‘visited‘: Boolean indicating if the node has been visited.
2\. ‘path_length‘: Integer representing the current length of the path.
3\. ‘max_path_length‘: Integer tracking the longest path found.
 ### Initialization
Step 1: Set ‘visited = False‘, ‘path_length = 0‘, and ‘max_path_length = 1‘ for all nodes.
Step 2: For the initial node (e.g., Node Id: 0), set ‘visited = True‘ and
‘path_length = 1‘.
Distributed_Algorithm_Designer: ### Message
1\. ‘path_length‘: The current path length from the sender.
2\. ‘max_path_length‘: The maximum path length known to the sender.
3\. ‘visited_flag‘: Boolean indicating whether the sender has visited the node.
### Send
Step 1: For each neighbor (Node A, Node B), if ‘visited‘ is ‘False‘, construct the message as follows:
- Message to Node A: ‘(Node Id: <node_id>, path_length + 1, max(max_path_length, path_length + 1))‘.
- Message to Node B: ‘(Node Id: <node_id>, path_length + 1, max(max_path_length, path_length + 1))‘.
Step 2: Send the constructed messages to both neighbors.
### Example
## Input
Node Id: 0
State:
1\. visited: False
2\. path_length: 2
3\. max_path_length: 3
Neighbor Information:
Connected to:
Node A
Node B
## Process
Step 1: Check neighbors Node A and Node B. Since ‘visited‘ is
‘False‘, proceed to construct messages.
- For Node A:
  - Calculate new ‘path_length‘: ‘2 + 1 = 3‘
  - Calculate ‘max_path_length‘: ‘max(3, 2 + 1) = 3‘
- For Node B:
  - Same calculations apply as for Node A.
Step 2: Construct messages:
- Message to Node A: ‘(Node Id: 0, 3, 3)‘
- Message to Node B: ‘(Node Id: 0, 3, 3)‘
## Output
Message sent to Node A:
1\. path_length: 3
2\. max_path_length: 3
3\. visited_flag: False
Message sent to Node B:
1\. path_length: 3
2\. max_path_length: 3
3\. visited_flag: False
### Update
Step 1: For each received message, check the ‘visited_flag‘. If it is
‘False‘, proceed to the next steps.
Step 2: Update ‘visited‘ to ‘True‘ for the current node.
Step 3: Update ‘path_length‘ to the maximum of its current value and
the received ‘path_length‘ from the message, incrementing by 1 if the
message indicates a potential continuation of the path.
Step 4: Update ‘max_path_length‘ to the maximum of its current value
and the received ‘max_path_length‘ from the message.
Step 5: If any messages were processed, trigger a re-evaluation of
outgoing messages to neighbors, if applicable.
### Example
## Input
Node Id: 1
State:
1\. visited: False
2\. path_length: 1
3\. max_path_length: 2
Received Messages:
Message 1:
1\. path_length: 2
2\. max_path_length: 3
3\. visited_flag: False
Message 2:
1\. path_length: 3
2\. max_path_length: 4
3\. visited_flag: True
## Process
Step 1: Process Message 1\. Since ‘visited_flag‘ is ‘False‘, proceed:
- Update ‘visited‘ to ‘True‘.
- Update ‘path_length‘: ‘max(1, 2) + 1 = 3‘.
- Update ‘max_path_length‘: ‘max(2, 3) = 3‘.
Step 2: Process Message 2\. Since ‘visited_flag‘ is ‘True‘, ignore this
message and do not make any changes.
## Output
State:
1\. visited: True
2\. path_length: 3
3\. max_path_length: 3
### Termination
The algorithm continues until:
- A node reaches a ‘path_length‘ equal to the total number of nodes
(indicating a Hamiltonian path exists).
- Or until a predefined number of iterations equal to the total number
- of nodes (‘N‘) is reached without discovering a Hamiltonian path.
Master: ##################### ITER 3, STEP 1: REASONING ######################
Master: {’thought’: "The algorithm is designed to check for a
Hamiltonian path. Now let’s execute it to see if such a path exists.",
’speak’: "The algorithm is ready. Let’s execute it to determine if there is
a Hamiltonian path.", ’function’: [{’name’: ’Distributed_Execution’,
’arguments’: {}}]}
Master: ####################### ITER 3, STEP 2: ACTING #######################
0: State: 1\. visited: True 2\. path_length: 1 3\. max_path_length: 1
2: State: 1\. visited: False 2\. path_length: 0 3\. max_path_length: 1
5: State: 1\. visited: False 2\. path_length: 0 3\. max_path_length: 1
3: State: 1\. visited: False 2\. path_length: 0 3\. max_path_length: 1
4: State: 1\. visited: False 2\. path_length: 0 3\. max_path_length: 1
1: State: 1\. visited: False 2\. path_length: 0 3\. max_path_length: 1
Node 5 Send Message to Node 1: 1\. path_length: 1 2\. max_path_length:
1 3\. visited_flag: False
Node 5 Send Message to Node 2: 1\. path_length: 1 2\. max_path_length:
1 3\. visited_flag: False
Node 5 Send Message to Node 3: 1\. path_length: 1 2\. max_path_length:
1 3\. visited_flag: False
...
All agents’ state unchanged, terminating early...
>>> END
system: 1\. Execute function Distributed_Execution
   [ARGUMENTS]:

   [STATUS]: SUCCESS
   [RESULT]: Node: 0  State: 1\. visited: True 2\. path_length: 5 3\. max_path_length: 1
Node: 1  State: 1\. visited: True 2\. path_length: 2 3\. max_path_length: 1
Node: 2  State: 1\. visited: True 2\. path_length: 4 3\. max_path_length: 3
Node: 3  State: 1\. visited: True 2\. path_length: 5 3\. max_path_length: 3
Node: 4  State: 1\. visited: True 2\. path_length: 4 3\. max_path_length: 3
Node: 5  State: 1\. visited: True 2\. path_length: 5 3\. max_path_length: 3

Master: ##################### ITER 4, STEP 1: REASONING ######################
Master: {’thought’: ’Since the maximum path length found is 5, which is
less than the total number of nodes (6), there is no Hamiltonian path in
the graph.’, ’speak’: ’No, there is no Hamiltonian path in this graph.’,
’function’: []}

```

## Appendix C Execution examples of GraphAgent-Reasoner

Shortest Path Execution Example.

Problem Description:

Find the shortest distance from a source node to other nodes in an undirected graph. In an undirected graph, (i,j,k) means that node i and node j are connected with an undirected edge with weight k. The graph has 8 nodes, and the edges are: (0,7,9) (0,1,7) (0,4,9) (1,7,1) (2,7,7) (2,6,5) (2,5,8) (3,5,9) (3,4,8) (3,6,1) (4,7,7) (4,5,6) (5,7,6). Give the weight of the shortest distance from node 1 to other node.

Execution Process:

```
Master: ##################### ITER 1, STEP 1: REASONING ######################
Master: {’thought’: ’I need to find the shortest distances from node 1 to all
other nodes in the given undirected graph. First, I will build the graph using
the provided edges and then design a distributed algorithm to find the
shortest paths.’, ’speak’: ’Building the graph and designing the distributed
algorithm to find the shortest distances from node 1.’, ’function’: ’[{"name":
"Build_Graph", "arguments": {"Edges": [[0, 7, 9], [0, 1, 7], [0, 4, 9], [1, 7, 1], [2,
7, 7], [2, 6, 5], [2, 5, 8], [3, 5, 9], [3, 4, 8], [3, 6, 1], [4, 7, 7], [4, 5, 6], [5, 7, 6]],
"Nodes_Num": 8, "Directed": false}}]’}
Master: ####################### ITER 1, STEP 2: ACTING #######################
>>>     Edges: [[0, 7, 9], [0, 1, 7], [0, 4, 9], [1, 7, 1], [2, 7...
>>>     Nodes_Num: 8
>>>     Directed: False
>>> END
system: 1\. Execute function Build_Graph
   [ARGUMENTS]:
       Edges: [[0, 7, 9], [0, 1, 7], [0, 4, 9], [1, 7, 1], [2, 7, 7], [2, 6, 5], [2, 5, 8], [3,
       5, 9], [3, 4, 8], [3, 6, 1], [4, 7, 7], [4, 5, 6], [5, 7, 6]]
                Nodes_Num: 8
                Directed: False
   [STATUS]: SUCCESS
Master: ##################### ITER 2, STEP 1: REASONING ######################
Master: {’thought’: ’The graph has been successfully built. Now, I will design
a distributed algorithm to find the shortest distances from node 1 to all
other nodes.’, ’speak’: ’Designing a distributed algorithm to find the
shortest distances from node 1.’, ’function’: ’[{"name":
"Design_Distributed_Algorithm", "arguments": {"Problem_Str": "Find the
shortest distance from node 1 to all other nodes in an undirected graph
with 8 nodes and given edges."}}]’}
Master: ####################### ITER 2, STEP 2: ACTING #######################
>>> END
Master: ##################### ITER 3, STEP 1: REASONING ######################
Master: {’thought’: ’The distributed algorithm has been successfully
designed. Now, I will execute the algorithm to find the shortest distances
from node 1 to all other nodes.’, ’speak’: ’Executing the distributed
algorithm to calculate the shortest distances from node 1.’, ’function’:
’[{"name": "Distributed_Execution", "arguments": {}}]’}
Master: ####################### ITER 3, STEP 2: ACTING #######################
Initialization:
1: State: 1\. distance: 0
0: State: 1\. distance: \infinity
2: State: 1\. distance: \infinity
4: State: 1\. distance: \infinity
5: State: 1\. distance: \infinity
7: State: 1\. distance: \infinity
3: State: 1\. distance: \infinity
6: State: 1\. distance: \infinity
Node 6 Send Message to Node 2: 1\. new_distance: \infinity
Node 6 Send Message to Node 3: 1\. new_distance: \infinity
Node 1 Send Message to Node 0: 1\. new_distance: 7
Node 1 Send Message to Node 7: 1\. new_distance: 1
Node 3 Send Message to Node 5: 1\. new_distance: \infinity
....
All agents’ state unchanged, terminating early...
>>> END
system: 1\. Execute function Distributed_Execution
   [ARGUMENTS]:

   [STATUS]: SUCCESS
   [RESULT]: Node: 0  State: 1\. distance: 7
Node: 1  State: 1\. distance: 0
Node: 2  State: 1\. distance: 8
Node: 3  State: 1\. distance: 14
Node: 4  State: 1\. distance: 8
Node: 5  State: 1\. distance: 7
Node: 6  State: 1\. distance: 13
Node: 7  State: 1\. distance: 1

Master: ##################### ITER 4, STEP 1: REASONING ######################
The shortest distances from node 1 are as follows: Node 0: 7, Node 1: 0,
Node 2: 8, Node 3: 14, Node 4: 8, Node 5: 7, Node 6: 13, Node 7: 1.Master:
{’thought’: ’The distributed algorithm has been successfully executed, and I
have obtained the shortest distances from node 1 to all other nodes.’,
’speak’: ’The shortest distances from node 1 are as follows: Node 0: 7, Node
1: 0, Node 2: 8, Node 3: 14, Node 4: 8, Node 5: 7, Node 6: 13, Node 7: 1.’,
’function’: []}

```