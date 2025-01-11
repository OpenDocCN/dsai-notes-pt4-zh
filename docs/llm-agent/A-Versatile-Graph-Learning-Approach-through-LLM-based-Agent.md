<!--yml
category: 未分类
date: 2025-01-11 13:07:12
-->

# A Versatile Graph Learning Approach through LLM-based Agent

> 来源：[https://arxiv.org/html/2309.04565/](https://arxiv.org/html/2309.04565/)

\useunder

\ul

Lanning Wei^(1,2), Huan Zhao^(3,∗), Xiaohan Zheng⁴ Zhiqiang He^(1,5), Quanming Yao^(4,∗) ¹Institute of Computing Technology, Chinese Academy of Sciences ²University of Chinese Academy of Sciences ³Noumena AI, ⁴Department of Electronic Engineering, Tsinghua University, ⁵LenovoBeijingChina [weilanning@163.com; zhaohuan@noumena.com.cn; qyaoaa@tsinghua.edu.cn](mailto:weilanning@163.com;%20zhaohuan@noumena.com.cn;%20qyaoaa@tsinghua.edu.cn)

###### Abstract.

Designing versatile graph learning approaches is important, considering the diverse graphs and tasks existing in real-world applications. Existing methods have attempted to achieve this target through automated machine learning techniques, pre-training and fine-tuning strategies, and large language models. However, these methods are not versatile enough for graph learning, as they work on either limited types of graphs or a single task. In this paper, we propose to explore versatile graph learning approaches with LLM-based agents, and the key insight is customizing the graph learning procedures for diverse graphs and tasks. To achieve this, we develop several LLM-based agents, equipped with diverse profiles, tools, functions and human experience. They collaborate to configure each procedure with task and data-specific settings step by step towards versatile solutions, and the proposed method is dubbed GL-Agent. By evaluating on diverse tasks and graphs, the correct results of the agent and its comparable performance showcase the versatility of the proposed method, especially in complex scenarios.The low resource cost and the potential to use open-source LLMs highlight the efficiency of GL-Agent.

## 1\. Introduction

Graph-structured data have been widely employed in various real-world domains, such as social networks (Hamilton et al., [2017](https://arxiv.org/html/2309.04565v2#bib.bib17)), e-commerce graphs (Lü et al., [2012](https://arxiv.org/html/2309.04565v2#bib.bib29)), and chemistry or biomedical molecules (Zhang et al., [2023c](https://arxiv.org/html/2309.04565v2#bib.bib70); Gilmer et al., [2017](https://arxiv.org/html/2309.04565v2#bib.bib13)). These applications, which are based on graphs, display a significant amount of diversity in terms of the domain knowledge and learning tasks contained in the graph. These diversities on graphs and learning tasks demands different settings in graph learning procedures toward effective graph mining (You et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib62); Rossi et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib34); Gao et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib12); Yao et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib60)).

Existing methods are not versatile enough on graph learning. To be specific, automated machine learning (AutoML) techniques have been explored towards handling diverse graphs with a single task (Luo et al., [2019](https://arxiv.org/html/2309.04565v2#bib.bib32); Wang et al., [2022c](https://arxiv.org/html/2309.04565v2#bib.bib50); Gao et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib12); You et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib62); Zhang et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib71)). It is achieved by customizing the machine learning pipeline in a data-driven manner, which alleviate the heavy efforts when facing diverse graphs. To handling diverse graph learning tasks, pre-training and fine-tuning paradigm (Lu et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib30)), especially the prompt tuning strategy from large language models (LLMs) (Sun et al., [2022](https://arxiv.org/html/2309.04565v2#bib.bib38); Liu et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib28)), have been widely explored in recent years. The pre-trained model can adapt to the diverse downstream tasks by tuning with task-specific data. Furthermore, motivated by the emergent ability in LLMs (Brown et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib3); Touvron et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib42); Chowdhery et al., [2022](https://arxiv.org/html/2309.04565v2#bib.bib6)), flatten-based methods transform graph learning problems into natural language question-answer problems, and then use LLMs to obtain the predictions (Fatemi et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib9); Li et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib23)). These methods have difficulties in describing large-scale graphs considering the limited context length of LLMs.

![Refer to caption](img/c7cfceb89b595bfa1fdb87dcd8add3d8.png)

Figure 1\. The framework designed to customize the graph learning procedures with LLM-based agents.

In this paper, we propose GL-Agent, which leverage the insight from the versatility of LLMs, to explore the versatile graph learning approaches. Learning on diverse tasks and graphs requires different settings in machine learning pipeline, as shown in Fig. [1](https://arxiv.org/html/2309.04565v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ A Versatile Graph Learning Approach through LLM-based Agent"), i.e., designing different strategies for data, model and hyper-parameters based on the human expertise. It represents that versatile solutions can be obtained by configuring the pipeline with task- and data-specific settings. However, it is difficult to achieve this by LLMs directly due to the complex procedures (Wei et al., [2022a](https://arxiv.org/html/2309.04565v2#bib.bib52); Valmeekam et al., [2024](https://arxiv.org/html/2309.04565v2#bib.bib43)). Considering this, we design LLM-based agent in each procedure to configure and complete the pipeline step by step, and the proposed method is dubbed GL-Agent (Graph Learning Agent).

As shown in Fig. [1](https://arxiv.org/html/2309.04565v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ A Versatile Graph Learning Approach through LLM-based Agent"), started with users’ instructions of graph learning requirements, the manager agent extracts the graph learning keywords from the instructions, e.g., data, learning target, model design preferences. Then, in graph learning procedures, AutoML is adopted due to its effectiveness in designing data-specific approaches  (Yao et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib60); Pham et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib33); Gao et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib12)). Four graph learning agents are provided, each specializing for data, AutoML configuration, data-specific model selection, and hyper-parameter optimization, respectively. The final response is generated based on the graph learning results, after which the user obtains the response according to the given instructions. The agent is equipped with well-defined profile, external tools, functions that may used, and expertise description when executing the given procedure. Then, LLMs are able to configure and complete the procedure step by step.

To validate the proposed GL-Agent, we adopt 11 widely used datasets from node, link and graph levels. The versatility and effectiveness are demonstrated by the correct agent outputs and comparable model performance on these datasets, particularly with complex non-homophilous graphs and complex instructions. The low time and economic cost involved in designing versatile graph learning solution represent the efficiency of GL-Agent.

The main contributions are summarized as follows: 1) We propose a method GL-Agent to explore the versatile graph learning approaches with LLM-based agent. 2) Given diverse tasks and graphs, we develop well-equipped manager, graph learning and response agents to configure and conduct the graph learning procedures step by step towards versatile graph learning solutions. 3) Extensive experiments have been conducted on diverse tasks and graphs. The comparable performance and correct agent outputs demonstrate the versatility and effectiveness of the proposed GL-Agent. The low resource cost and potential for using open-source LLMs highlight the efficiency of the proposed method.

## 2\. Related Work

### 2.1\. Versatile Graph Learning Methods

Existing methods have widely explored in handling the diverse tasks and data in graph learning problems.

1) AutoML is the representative technique in handling different data by designing data-specific solutions (Yao et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib60)). Existing methods have explored the data-specific feature selection strategies (Wang et al., [2022c](https://arxiv.org/html/2309.04565v2#bib.bib50); Luo et al., [2019](https://arxiv.org/html/2309.04565v2#bib.bib32)), feature dimension design methods (Ginart et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib14); Zhaok et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib74); Liu et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib25)). GNNs are automatically designed from the aspects of operations (Gao and Ji, [2019](https://arxiv.org/html/2309.04565v2#bib.bib10); Zhao et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib72), [2021](https://arxiv.org/html/2309.04565v2#bib.bib73); Wei et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib55); You et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib62); Wang et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib51)) and skip-connections (Wei et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib54); Li and King, [2020](https://arxiv.org/html/2309.04565v2#bib.bib22)). These methods have the ability in handing different graphs while limited in single task as represented in Table [1](https://arxiv.org/html/2309.04565v2#S2.T1 "Table 1 ‣ 2.1\. Versatile Graph Learning Methods ‣ 2\. Related Work ‣ A Versatile Graph Learning Approach through LLM-based Agent").

Table 1\. The comparisons of existing methods in handling diverse tasks and graphs.

| Methods | Diverse Tasks | Diverse graphs |
| --- | --- | --- |
| GL-Agent | $\checkmark$ | $\checkmark$ |
| AutoML | $\times$ | $\checkmark$ |
| Pre-training and tuning | $\checkmark$ | $\times$ |
| Flatten-based methods | $\checkmark$ | $\checkmark$(small-scale) |

2) Pre-training and fine-tuning paradigm, especially prompt tuning, are widely used in recent years to handling the different graph learning tasks, motivated by the versatility of LLMs (Liu et al., [2023a](https://arxiv.org/html/2309.04565v2#bib.bib26); Lu et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib30); Liu et al., [2023c](https://arxiv.org/html/2309.04565v2#bib.bib27); Xiong et al., [2024b](https://arxiv.org/html/2309.04565v2#bib.bib58)). These methods learn the task-specific vector in tuning stage with downstream task data, and then the pre-trained models could applied to different graph learning tasks (Liu et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib28); Sun et al., [2022](https://arxiv.org/html/2309.04565v2#bib.bib38), [2023a](https://arxiv.org/html/2309.04565v2#bib.bib39); Yu et al., [2024](https://arxiv.org/html/2309.04565v2#bib.bib64)). However, the gap between two stages are significant, which representing the deficiency in handling diverse graphs. 3) Flatten-based graph prediction methods are proposed motivated by the emerging ability of LLMs (Fatemi et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib9); Wang et al., [2024](https://arxiv.org/html/2309.04565v2#bib.bib45); Li et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib23)) These methods describe the graphs with natural language, and then use LLMs to predict the results. They have limitations in the large-scale graph given the context window length.

Compared with these methods, the designed GL-Agent could benefit from the AutoML techniques and versality of LLMs when handling different tasks and graphs. It is achieved by configuring the graph learning procedures in task- and data-specific settings with the assistance of LLM-based agent.

### 2.2\. LLM-empowered Machine Learning

LLMs have achieved great success in wide applications with their ability in achieving human-like intelligence (Wang et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib47); weng, [2023](https://arxiv.org/html/2309.04565v2#bib.bib56); Hong et al., [2024](https://arxiv.org/html/2309.04565v2#bib.bib21); Guo et al., [2024](https://arxiv.org/html/2309.04565v2#bib.bib16)). They are used to serve as the excellent artificial general intelligence (AGI) in real world, with applications including question answering in natural language processing tasks, automatic solving compute vision tasks, and reasoning graph properties  (Guo et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib15); Wang et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib47); Zhang, [2023](https://arxiv.org/html/2309.04565v2#bib.bib67); Shen et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib37); Tornede et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib41)).

On the one hand, LLM could empower the machine learning tasks by automatically decomposing and completing complex tasks, and they are proposed to bringing more convenient, comprehensive and reliable decisions when facing diverse applications and tasks (Shen et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib37); Zhang et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib69); Wang et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib47); Zhang et al., [2023a](https://arxiv.org/html/2309.04565v2#bib.bib66); Yuan et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib65)). For instance, HuggingGPT (Shen et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib37)) responds to the user request with task planning, model selection, model execution and response generation procedures. Each of them are executed automatically following the managing of LLM-based agents. Similarly, AutoML-GPT (Zhang et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib69)) tackles diverse datasets and tasks automatically via conducting experiments from data processing to model architecture, hyper-parameter tuning. These procedures are also managed by LLMs. On the other hand, LLM can enhance machine learning tasks from a model perspective, which is also widely used on graph learning. This includes, but is not limited to, make a better understanding on graphs (Guo et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib15); Zhang, [2023](https://arxiv.org/html/2309.04565v2#bib.bib67)), design neural architectures (Yu et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib63); Zheng et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib75); Zhang et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib69); Wang et al., [2023a](https://arxiv.org/html/2309.04565v2#bib.bib46)), or directly predict the labels with LLMs (Zhang, [2023](https://arxiv.org/html/2309.04565v2#bib.bib67); Sun et al., [2023b](https://arxiv.org/html/2309.04565v2#bib.bib40); Gao et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib11); Chen et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib4)).

In this paper, we propose to explore versatile graph learning approaches, and LLM-based agents are utilized to configuring and executing the graph learning pipelines step by step, which bring flexibility for users.

## 3\. Method

For graph learning problems with different tasks and graphs, they have the same learning pipeline while different configurations as shown in Fig. [1](https://arxiv.org/html/2309.04565v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ A Versatile Graph Learning Approach through LLM-based Agent") (Yao et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib60)). Motivated by the versatility of LLMs, we propose to design versatile graph learning approaches with LLMs by configuring the pipeline with specific settings learned from the users, tasks and graphs. It is non-trivial to achieve this with LLMs considering the complex learning procedures and the absence of expertise  (Wei et al., [2022a](https://arxiv.org/html/2309.04565v2#bib.bib52); Shen et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib37); Valmeekam et al., [2024](https://arxiv.org/html/2309.04565v2#bib.bib43)). To this end, we propose to address this challenge by providing well-equipped LLM-based agents for each procedure to accomplish the complex pipeline step by step. With the assistance of LLMs, the graph learning problems with different tasks and graphs can be solved following the users’ instructions, instead of heavily relying on the human efforts.

![Refer to caption](img/0087bc5739955abe1709389a2b6bffee.png)

Figure 2\. The introduction of procedures and agents used when learning on graphs.

As shown in Fig. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"), a set of well-equipped agents are co-operated to conduct the graph learning procedures along with the black arrows. Firstly, the manager agent extracts the graph learning information from these instructions, on top of which the real-world learning information is transformed into general graph learning settings that can be used in the following agents. Then, four graph learning agents, i.e., data, configuration, searching and tuning agent, are deployed to assist the graph learning procedures. Finally, responses are generated for user based on the searched results and the instructions, which achieved by the proposed the response agent. For the agents, the profiles, functions, tools, and expertise needed for conducting the procedures are provided. Then, the agent can be configured and executed to produce the final response. In the following sections, we will introduce each agent, and more details can be found in Appendix [A](https://arxiv.org/html/2309.04565v2#A1 "Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").

### 3.1\. Preliminary on Technical Support

The designed procedures necessitate agents to comprehend requests, invoke APIs and functions, use external plugin tools, execute code, and perform question reasoning based on the documents. In this paper, we utilize the LangChain ¹¹1https://github.com/langchain-ai/langchain, an LLM-driven tool to develop agents and interact with external resources and models, to assist in different procedures in graph learning Additionally, we use PyG ²²2https://github.com/pyg-team/pytorch_geometric, HyperOPT and PyTorch packages in the graph learning pipeline, and the interactions with these packages are achieved using LLMs.

### 3.2\. Instruction and Manager Agent

The graph learning pipeline begins by extracting learning tasks from real-world applications and designing the graph learning techniques used in the subsequent procedures. The user’s instructions may encompass various keywords, which include but are not limited to descriptions of the data, learning task, evaluation metric, human expertise on the learning task, and hardware device. More details of instructions can be found in Appendix [A.1](https://arxiv.org/html/2309.04565v2#A1.SS1 "A.1\. Instructions ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").

The manager agent is responsible for extracting the relevant information from the user’s instruction. With the pre-defined profile and expertise of graph learning keywords, the agent can extract and summarize the natural language instructions. The results are formatted as a structured Python dictionary and saved in a file that will be used by following agents.

### 3.3\. Graph Learning Agents

We provide four graph learning agents assisted by AutoML to generate graph learning solutions given diverse tasks and graphs, considering the following reasons: (1) It is a challenge for LLMs to generate effective graph learning solutions directly given their outdated training data (“LLM-GNN” baseline in experiments), (2) AutoML has advantages in designing data-specific and effective graph learning solutions (Zhang et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib71); Wang et al., [2022a](https://arxiv.org/html/2309.04565v2#bib.bib49)), which is achieved by preserving diverse candidates and then selecting one like human beings (You et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib62)). Therefore, we use LLM-based agents to configure the graph learning procedures under AutoML, which benefits from both the effectiveness and versatility. As shown in Fig. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"), four graph learning agents are designed to conduct graph processing, AutoML configuration, model search and hyper-parameter fine-tuning procedures, respectively.

#### 3.3.1\. Data Agent

The data agent is designed to prepare the data and conduct the feature engineering processes in the graph learning pipeline. Depending on the learning task and evaluation metric, the agent’s objective is to select appropriate feature engineering strategy. The agent browses the website of PyG document, decides to invoke the correct feature engineering API, and then saves the decisions in the memory that are used in the following agents, in the same way as human experts.

#### 3.3.2\. Configuration Agent

Following the general pipeline when designing architectures based on AutoML, i.e., neural architecture search, we first need to set the configurations of the search space and search algorithm. The agent assists the configuration procedure by browsing the documentation and memory, and then making decisions like human experts do.

$\bullet$Search Space. The search space is constructed by preparing a set of candidate operations within the designed architecture backbone, in which the agent plays a crucial role in making decisions. As shown in Figure. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"), four functions are designed to configure the search space with LLM-based agents. (1):Module selection. The agent selects the proper operation modules to construct the models based on the learning task $\mathcal{T}$, e.g., selecting aggregation and activation module for node classification task. It is achieved by reasoning on the vast maintained knowledge in LLMs. (2): Operation preparation. To prepare the potential operations for the selected module, the agent looks up the corresponding documents in PyG. Then, the available candidate operations in each module can be extracted, e.g., by collecting all implemented convolution operations provided in PyG for aggregation module. (3): Candidate selection. Considering that the different preferences $\mathcal{P}$ and constrains $\mathcal{C}$ of the hardware device which may affect the model design, the candidate operations used in each module are selected based on these constrains as shown in Fig. [3](https://arxiv.org/html/2309.04565v2#A2.F3 "Figure 3 ‣ B.1.1\. Node-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), relying on the ability of LLMs to make human-like decisions. (4): Formatting output. Based on the selected candidate operations, the results are formatted and preserved in a new file. With the results from previous function, these four functions are conducted sequentially to obtained the configured search space, i.e., a set of candidates in each selected module as shown in Fig. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent").

$\bullet$Search algorithm. The search algorithm is proposed to explore the designed search space and find the suitable architecture under given constrains (Elsken et al., [2019](https://arxiv.org/html/2309.04565v2#bib.bib8); Zoph and Le, [2017](https://arxiv.org/html/2309.04565v2#bib.bib77)). As shown in Fig. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"), we define two functions in this agent when configuring the search algorithm. (1): Identifying requirements. The agent looks up the instruction and summarizes the efficiency requirement. (2) Algorithm selection. For simplicity, we provide two variants in the search algorithm: random search, and differentiable search algorithm, which is more efficient but has constraints on the search space. The agent selects search algorithm based on the summarized requirements and the designed search space.

#### 3.3.3\. Searching Agent

Based on the configurations of the search space and algorithm, we construct and search the data-specific models within the search space. Given the AutoML code, the searching agent helps configure the arguments, execute the code, and formulate the output that is used by the following agent. As shown in Fig. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"), four steps are implemented in the searching agent. (1) Arguments configuration. The searching agent loads the AutoML code and then formats the configurations as the arguments. (2) Code execution. Based on the configured arguments, the agent helps to execute the code. (3) Model construction. After the code execution is finished, the searching agent extracts the data-specific models by looking up the logging file. (4) Summary generation. Based on the searched models and the logging file, the LLM-based searching agent summarizes this procedure and then saves the results in a new document. Through these collaborative steps, we obtain the searched architecture that fulfills the specified design objectives and requirements.

#### 3.3.4\. Tuning Agent

After obtaining the searched GNNs, we can fine-tune the hyper-parameters to obtain the final results. Similar to the searching agent, the tuning agent has steps as shown in Fig. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"). For simplicity, we utilize HyperOPT (Bergstra et al., [2013](https://arxiv.org/html/2309.04565v2#bib.bib2)) to assist the tuning agent. Then, in arguments configuration, the hyper-parameters, such as learning rate, dropout ratio, weight decay ratio, and normalization, are configured based on the instructions and searched models, and it is achieved by reading the code and matching the arguments with the agent. Then, the similar Code execution, Hyper-parameter preparationand Summary generation steps are provided to execute code and generate the final output. Finally, we can obtain the final performance following the users’ instructions.

### 3.4\. Response Agent

After obtaining the searched models and the corresponding performance from the completed graph learning procedures, we utilize a response agent to generate a comprehensive summary based on the shared memory and the logging files of these agents. The summary may contain the user instructions, the searched models $\bm{\alpha}$, the hyper-parameters $\mathcal{H}$, the prediction performance $\mathcal{R}$ and so on.

### 3.5\. Discussion

GL-Agent is proposed to explore versatile graph learning approaches given diverse tasks and graphs, leveraging insights from LLMs. Beyond the versatility introduced in Section [2.1](https://arxiv.org/html/2309.04565v2#S2.SS1 "2.1\. Versatile Graph Learning Methods ‣ 2\. Related Work ‣ A Versatile Graph Learning Approach through LLM-based Agent"), GL-Agent also offers additional benefits in flexibility and extensibility.

Flexibility. Compared with existing methods which require extensive human effort when addressing graph learning problems, GL-Agent could achieve the target with instructions alone. This approach is more flexible for users and could lower the learning and usage threshold for non-expert users unfamiliar with graph learning.

Extensibility. GL-Agent uses AutoML techniques in designing graph learning solutions, and it is configured with LLM-based agents. It indicates that this method can be extended to new graphs to obtain data-specific and effective graph learning solutions. Besides, the widely-used graph learning package PyG is considered in this paper. Therefore, the user-defined GNNs implemented based on PyG can be integrated into this method, indicating the extension of user-specific expertise.

## 4\. Experiments

In the experiments, we first evaluate the versatility of GL-Agent from the perspective of agent output and performance over different tasks and graphs. Then we show the ability of GL-Agent in handling complex scenarios, including predictions on large-scale non-homophilous graphs and the use of complex instructions. Finally, we show the resource cost and the potential of using other LLMs.

### 4.1\. Implementation Instance

In this paper, we develop LLM-based agents to assist the configuration and execution of graph learning pipeline using AutoML technique. We provide three AutoML implementation instances over three tasks: a) Node-level. The instance is constructed based on the differentiable automated GNN design method F2GNN (Wei et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib54)), which focus on designing the aggregation operations and the GNN topology, i.e., the connections between these aggregation operations. b) Graph-level The graph-level implementation instance is constructed based on LRGNN (Wei et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib53)), which searches to capture the long-range dependency with deep stacked GNNs. The framework is constructed by several cells and in which the selection, fusion and aggregation modules are provided. A readout module is provided to generate the graph representations for this task at the end of framework. c) Link-level For link-level, the instance is constructed based on method Prof-CF (Wang et al., [2022a](https://arxiv.org/html/2309.04565v2#bib.bib49)), which searches for GNN-based two-tower collaborative filtering functions. The framework contains diverse design modules, including message function, aggregation, layer combinations, and it designs the search space by pruning operations. The manager agent chooses according instance based on the user instruction, and the detailed introduction of these agents are shown in Appendix [B.1](https://arxiv.org/html/2309.04565v2#A2.SS1 "B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent").

### 4.2\. Experimental settings

#### 4.2.1\. Datasets

To evaluate the versatility of designed GL-Agent in handling the diverse tasks and graphs, we adopt the node classification, graph classification and item ranking task in recommendation systems in this paper. The widely used datasets are adopted: (a) Node classification: Cora (Sen et al., [2008](https://arxiv.org/html/2309.04565v2#bib.bib35)), Physics (Shchur et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib36)), as well as the Computers and Photo datasets from (Shchur et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib36)), and large-scale non-homophilous dataset genius (Lim et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib24)); (b) Graph classification: DD and PROTEINS datasets from (Dobson and Doig, [2003](https://arxiv.org/html/2309.04565v2#bib.bib7)), as well as NCI1 and NCI109 datasets from (Wale and Karypis, [2006](https://arxiv.org/html/2309.04565v2#bib.bib44)); (c) item ranking task: Epinions and Amazon-Sports. The detailed introductions of these datasets are provided in Appendix [B.2](https://arxiv.org/html/2309.04565v2#A2.SS2 "B.2\. Datasets ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent").

#### 4.2.2\. Baselines

In this paper, LLM-based agents focus on extracting information, configuring, executing and summarizing graph learning procedures with AutoML technique. To evaluate the proposed GL-Agent, three types methods are adopted as baselines for each task, i.e., the human-designed methods, the NAS-based methods and baselines that LLM suggested. In the node classification task, we adopt the human designed methods GCN, GIN, GPR-GNN (Chien et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib5)) and ACM-GCN (Luan et al., [2022](https://arxiv.org/html/2309.04565v2#bib.bib31)), and the NAS-based methods SANE (Zhao et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib73)) and F2GNN (Wei et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib54)). In the graph classification task, we adopt the human-designed methods GCN, GIN, DGCNN (Zhang et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib68)) and DiffPool (Ying et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib61)), as well as the NAS-based method LRGNN (Wei et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib53)). In the item ranking task, we adopt the human-designed methods NCF (He et al., [2017](https://arxiv.org/html/2309.04565v2#bib.bib20)), NGCF (Wang et al., [2019](https://arxiv.org/html/2309.04565v2#bib.bib48)) and LightGCN (He et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib19)), and the NAS-based methd Prof-CF (Wang et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib51)). The detailed introduction of these methods and the LLM-GNN is provided in Appendix [B.3](https://arxiv.org/html/2309.04565v2#A2.SS3 "B.3\. Baselines ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent").

As mentioned in Section [3.1](https://arxiv.org/html/2309.04565v2#S3.SS1 "3.1\. Preliminary on Technical Support ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"), the proposed GL-Agent is constructed based on LangChain and PyG. We use the GPT-3.5-turbo as the deployment model in the agents. The detailed prompts and answers of agents are shown in the Appendix [A](https://arxiv.org/html/2309.04565v2#A1 "Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").

Table 2\. The comparisons of agent outputs on different tasks.

|  | Cora (Node) | NCI1 (Graph) | Epinions(Link) |
| --- | --- | --- | --- |
| Instructions | I have a dataset $\mathcal{D}$ which is a citation network like Cora dataset. In this network, the nodes $\cdots$ and the edge $\cdots$ . My goal is to accurately predict the domains of these papers. | I have a dataset $\mathcal{D}$, like NCI1 dataset in graph benchmark, and each graph is a chemical compounds. I want to find one GNN that has better accuracy when predicting the molecule property. | I currently possess a social network dataset like Epinions, denoted as $\mathcal{D}$. In this dataset, the nodes $\cdots$, and edges $\cdots$. My objective is to recommend potential friends for each user. |
| Prompted output of manager agent | $\mathcal{D}$: citation network Cora; $\mathcal{T}$: node classification; Evaluation metric $\mathcal{M}$: accuracy. $\cdots$ | $\mathcal{D}$: chemical compounds NCI1; $\mathcal{T}$: graph classification; Evaluation metric $\mathcal{M}$: accuracy. $\cdots$ | $\mathcal{D}$: social network Epinions; $\mathcal{T}$: link prediction; Evaluation metric $\mathcal{M}$: Recall@20\. $\cdots$ |
| Formatted output of configuration agent | The response list is [‘Aggregation’,‘Selection’,‘Fusion’] | The response list is [‘Aggregation’, ‘Pooling’, ‘Readout’,‘Selection’,‘Fusion’] | The response list is [‘Embedding dim’, ‘Message function’, $\cdots$, ‘Layer combination’, ‘Interaction function’] |

### 4.3\. Evaluation of Versatility

GL-Agent explores versatile graph learning solutions for diverse tasks and graphs using LLM-based agents. We evaluate versatility by showing the agent outputs and performance on different datasets across three tasks.

#### 4.3.1\. Correctness of Agent Output

As shown in Table [2](https://arxiv.org/html/2309.04565v2#S4.T2 "Table 2 ‣ 4.2.2\. Baselines ‣ 4.2\. Experimental settings ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), we compare the agent results when facing different tasks and graphs in graph learning. For simplicity, we only provide the instructions and outputs of manager agents, along with the configured search space. It is clear that the agent can extract useful information accurately and make correct predictions like human experts, demonstrating its versatility in handling different learning tasks and data based on diverse instructions. The details of the agents are provided in Appendix [A](https://arxiv.org/html/2309.04565v2#A1 "Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent"), and the complete case study on different tasks is provided in Appendix [B.5](https://arxiv.org/html/2309.04565v2#A2.SS5 "B.5\. Case Study ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent").

Table 3\. Performance comparisons on the node classification task. We report the average test accuracy and the standard deviation with 10 random splits.

|  | Cora | Photo | genius |
| # Nodes | 2,708 | 7,487 | 421,961 |
| GL-Agent | 86.81(0.40) | 96.40(0.16) | 90.51(0.15) |
| GCN | 85.68(0.61) | 93.13(0.27) | 89.10(0.13) |
| GIN | 83.83(1.36) | 92.67(0.57) | 85.61(0.06) |
| GPRGNN | 87.62(0.48) | 91.93(0.26) | 90.05(0.31) |
| ACM-GCN | 86.67(0.14) | 94.35(0.65) | 90.08(0.05) |
| SANE | 86.40(0.38) | 94.53(0.22) | OOM |
| F2GNN | 87.42(0.42) | 95.38(0.30) | 90.93(0.02) |
| LLM-GNN | 84.64(1.04) | 93.73(0.38) | 89.31(0.17) |

Table 4\. Performance comparisons on graph classification task. We report the mean test accuracy and the standard deviations based on the 10-fold cross-validation data.

|  | Proteins | DD |
| GL-Agent | 75.38(5.03) | 78.10(3.21) |
| GCN | 74.84(3.07) | 73.59(4.17) |
| GIN | 74.50(4.10) | 74.62(2.74) |
| DGCNN | 73.95(3.04) | 61.63(5.33) |
| DiffPool | 75.11(2.14) | 77.85(3.53) |
| LRGNN | 75.39(4.40) | 78.18(2.02) |
| LLM-GNN | 74.47(3.65) | 75.12(3.44) |

Table 5\. Performance comparisons on the item ranking task. We report the mean test Recall@20 and the standard deviations based on the 10 repeats.

|  | Epinions | Amazon-Sports |
| GL-Agent | 0.0453(0.0018) | 0.0829(0.0006) |
| NCF | 0.0344(0.0007) | 0.0636(0.0011) |
| NGCF | 0.0290(0.0002) | 0.0609(0.0011) |
| LightGCN | 0.0321(0.0003) | 0.0776(0.0006) |
| Prof-CF | 0.0472(0.0011) | 0.1023(0.0021) |
| LLM-GNN | 0.0379(0.0014) | 0.0151(0.0001) |

#### 4.3.2\. Performance Comparisons Over Different Tasks

We show the performance comparisons with the baselines to evaluate the effectiveness of GL-Agent. As shown in Table [3](https://arxiv.org/html/2309.04565v2#S4.T3 "Table 3 ‣ 4.3.1\. Correctness of Agent Output ‣ 4.3\. Evaluation of Versatility ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), on the node-level task, GL-Agent achieves comparable performance with AutoML-based method F2GNN. Since GL-Agent is implemented based on F2GNN, the comparable performance represents the ability of agents in understanding and handling the node-level tasks. For LLM-GNN, where the GNN and hyper-parameters are directly suggested by LLMs, the performance is subpar compared to the GCN baseline due to inappropriate hyper-parameters. This highlights the challenge LLMs face in handling different graphs and different graph learning procedures. As shown in Table [4](https://arxiv.org/html/2309.04565v2#S4.T4 "Table 4 ‣ 4.3.1\. Correctness of Agent Output ‣ 4.3\. Evaluation of Versatility ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent") and Table [5](https://arxiv.org/html/2309.04565v2#S4.T5 "Table 5 ‣ 4.3.1\. Correctness of Agent Output ‣ 4.3\. Evaluation of Versatility ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), we can obtain the same conclusion that GL-Agent has comparable performance with its basic AutoMl-based method LRGNN and prof-CF (Wang et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib51)) on the graph classification and item-ranking tasks, respectively. This conclusion is consistent across full datasets on three tasks, and detailed results and analysis on different tasks are provided in Appendix [B.4](https://arxiv.org/html/2309.04565v2#A2.SS4 "B.4\. Performance Comparisons ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent").

In summary, the proposed GL-Agent explores versatile graph learning approaches given diverse tasks and graphs. The proposed agents can generate task- and data-specific results to configure the graph learning procedures, as well as obtain comparable or higher performance than these baselines. The results highlight the versatility and effectiveness of GL-Agent in handling different tasks and data within complex instructions.

### 4.4\. Evaluations on Complex Scenarios

We show the ability of InstructionGL to achieve complex tasks that require high-level human expertise, on which the robustness can be evaluated.

#### 4.4.1\. Evaluations on Complex Graph Learning Task

We consider the large-scale non-homophilous scenario that connected nodes have different labels, which is a challenge to design effective models (Zhu et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib76); Luan et al., [2022](https://arxiv.org/html/2309.04565v2#bib.bib31)). As shown in Table [3](https://arxiv.org/html/2309.04565v2#S4.T3 "Table 3 ‣ 4.3.1\. Correctness of Agent Output ‣ 4.3\. Evaluation of Versatility ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), for the large-scale dataset genius, LLM-GNN has comparable performance with GCN baseline, even though we have added the dataset descriptions and experimental observations in the instruction (introduced in Appendix [B.3](https://arxiv.org/html/2309.04565v2#A2.SS3 "B.3\. Baselines ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent")). However, AutoML based method, i.e., F2GNN and GL-Agent, could achieve higher performance by designing data-specific solutions. Therefore, by using the AutoML method, its effectiveness in handling diverse graphs has been expanded into the proposed GL-Agent.

#### 4.4.2\. Evaluations with Complex Instructions

In the proposed framework, instructions contain all the information about data and learning tasks. The clarity of instructions will have a large influence on the agent’s outputs. Then, we provide three strategies to evaluate robustness on complex instructions, motivated by (Xiong et al., [2024a](https://arxiv.org/html/2309.04565v2#bib.bib57)). 1) Simple: Instructions organized with simplified sentences, keeping the agent output unchanged; 2) Complex: Using GPT-4 to paraphrase instructions and agent outputs with similar semantics but with complex words; 3) Misleading: Using GPT-4 to add misleading information to the original instructions and outputs. As shown in Table [6](https://arxiv.org/html/2309.04565v2#S4.T6 "Table 6 ‣ 4.4.2\. Evaluations with Complex Instructions ‣ 4.4\. Evaluations on Complex Scenarios ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), we show the accuracy of agent outputs when facing different instructions and inputs. We can clearly observe that comparable performance between simple and complex variants, which demonstrates the robustness of the proposed GL-Agent in extracting useful information from the input. Besides, misleading information has a large influence on configuration agent due to the complex tools and functions of this agent. Further improvements can be expected by revising the outputs based on feedback from reflection or human experts, and this will be considered in future work. The detailed instructions and results can be found in Table [24](https://arxiv.org/html/2309.04565v2#A2.T24 "Table 24 ‣ B.6\. Interpretability of Decisions in Graph Learning Tasks ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent") in appendix.

Table 6\. Performance comparisons on different instructions, and resource cost comparisons on different agents.

|  | Instructions | Resource Cost |
| Agents | Simple | Complex | Misleading | Time (s) | Token | Money (USD$) |
| Manager | 0.7 | 0.6 | 0.5 | 3.23 | 683.5 | 0.014 |
| Data | 1.0 | 1.0 | 0.9 | 4.95 | 1054 | 0.021 |
| Configuration | 0.7 | 0.6 | 0.3 | 11.57 | 6672.4 | 0.133 |
| Searching | 0.6 | 0.7 | 0.7 | 93.29 | 6462.5 | 0.129 |
| Tuning | 0.6 | 0.7 | 0.7 | 329.12 | 711.1 | 0.014 |

### 4.5\. The Evaluations of Resource Cost

GL-Agent designs versatile graph learning solutions with closed-source LLMs GPT-3.5-Turbo, where the time and economic cost cannot be ignored. Then, we empirically evaluate the search cost of GL-Agent when using APIs. As shown in Table [6](https://arxiv.org/html/2309.04565v2#S4.T6 "Table 6 ‣ 4.4.2\. Evaluations with Complex Instructions ‣ 4.4\. Evaluations on Complex Scenarios ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), the agents take less than one minute to generate the outputs, which can be ignored compared with the long time required for code execution. Additionally, it costs 0.31 USD to conduct one procedure, which is also economically efficient. These results indicate that GL-Agent can handle graph learning problems with diverse tasks and graphs in an efficient and economic manner.

Table 7\. The comparisons of agent results based on different LLMs.

| Functions | LLaMA2-7B | GPT-3.5-turbo + LangChain (ours) |
| --- | --- | --- |
| Module Selection | Possible operations: Aggregation: $\cdots$, Pooling: $\cdots$, Readout: $\cdots$ | The response list is [‘aggregation’, ‘pooling’, ‘readout’,‘selection’,‘fusion’] |
| Operation Preparation; Candidate Selection | To address your request: 1\. Justification: $\cdots$, 2\. Finding the corresponding class: $\cdots$, 3\. Returning the class name and module name $\cdots$. | ’ChebConv’, ’torch_geometric.nn.conv.cheb _conv.ChebConv’ |
| Identifying Requirements; Algorithm Selection | Firstly, let me clarify the difference between them. $\cdots$. Then, Lets evaluate the options based on these principles. $\cdots$ | You should use“differentiable search algorithm". |

To further lower the cost, we examine the different LLMs in exploring the versatile graph learning solutions. As illustrated in Table [7](https://arxiv.org/html/2309.04565v2#S4.T7 "Table 7 ‣ 4.5\. The Evaluations of Resource Cost ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), we show the comparisons with LLaMA2 7B and focus on procedures that yield inconsistent answers among these LLMs. Since LLaMA2 70B and GPT-4 obtained correct results, we move the comparisons to Table [26](https://arxiv.org/html/2309.04565v2#A2.T26 "Table 26 ‣ B.6\. Interpretability of Decisions in Graph Learning Tasks ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent") in appendix. Based on the carefully designed functions and tools, the agents can provide accurate predictions in most procedures except for the cell highlighted in gray, which may be caused by fewer parameters (7B). This suggests that we can substitute GPT-3.5-turbo with the more affordable open-source LLMs, which offer comparable performance in our method.

## 5\. Conclusion

We propose a method GL-Agent to explore versatile graph learning solutions given the different tasks and graphs in real-world applications. Firstly, we propose manager agent, four graph learning agents, and response agent that are equipped with profile, functions, external tools, and human experience. Then, based on the user instructions, the well-equipped agents understand user requirements in natural languages, configure and complete the graph learning procedures step-by-step. Finally, we obtained the customized graph learning procedures with task- and data-specific settings. In the experiments, GL-Agent can make correct decisions over different tasks, achieve comparable performance compared with baselines, even if on the complex scenarios. Besides, the low resource cost and potential for using open-source LLMs demonstrate its efficiency in accomplishing the graph learning task. In the future, we will improve the robustness and reliability of GL-Agent. More specifically, by adding the reflection and multi-round dialogues into the framework, the agents will revise their outputs with human feedback to achieve the target.

## References

*   (1)
*   Bergstra et al. (2013) James Bergstra, Daniel Yamins, and David Cox. 2013. Making a science of model search: Hyperparameter optimization in hundreds of dimensions for vision architectures. In *International conference on machine learning*. PMLR, 115–123.
*   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, et al. 2020. Language models are few-shot learners. *Advances in neural information processing systems* 33 (2020), 1877–1901.
*   Chen et al. (2023) Zhikai Chen, Haitao Mao, Hang Li, Wei Jin, Hongzhi Wen, Xiaochi Wei, Shuaiqiang Wang, Dawei Yin, Wenqi Fan, Hui Liu, et al. 2023. Exploring the potential of large language models (llms) in learning on graphs. *arXiv preprint arXiv:2307.03393* (2023).
*   Chien et al. (2021) Eli Chien, Jianhao Peng, Pan Li, and Olgica Milenkovic. 2021. Adaptive universal generalized pagerank graph neural network. *ICLR* (2021).
*   Chowdhery et al. (2022) Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann, et al. 2022. Palm: Scaling language modeling with pathways. *arXiv preprint arXiv:2204.02311* (2022).
*   Dobson and Doig (2003) Paul D Dobson and Andrew J Doig. 2003. Distinguishing enzyme structures from non-enzymes without alignments. *Journal of molecular biology (JMB)* 330, 4 (2003), 771–783.
*   Elsken et al. (2019) Thomas Elsken, Jan Hendrik Metzen, and Frank Hutter. 2019. Neural architecture search: A survey. *The Journal of Machine Learning Research* 20, 1 (2019), 1997–2017.
*   Fatemi et al. (2023) Bahare Fatemi, Jonathan Halcrow, and Bryan Perozzi. 2023. Talk like a graph: Encoding graphs for large language models. *arXiv preprint arXiv:2310.04560* (2023).
*   Gao and Ji (2019) Hongyang Gao and Shuiwang Ji. 2019. Graph u-nets. *ICML*, 2083–2092.
*   Gao et al. (2023) Jun Gao, Huan Zhao, Changlong Yu, and Ruifeng Xu. 2023. Exploring the feasibility of chatgpt for event extraction. *arXiv preprint arXiv:2303.03836* (2023).
*   Gao et al. (2020) Yang Gao, Hong Yang, Peng Zhang, Chuan Zhou, and Yue Hu. 2020. Graphnas: Graph neural architecture search with reinforcement learning. In *IJCAI*.
*   Gilmer et al. (2017) Justin Gilmer, Samuel S Schoenholz, Patrick F Riley, Oriol Vinyals, and George E Dahl. 2017. Neural Message Passing for Quantum Chemistry. In *ICML*. 1263–1272.
*   Ginart et al. (2021) Antonio A Ginart, Maxim Naumov, Dheevatsa Mudigere, Jiyan Yang, and James Zou. 2021. Mixed dimension embeddings with application to memory-efficient recommendation systems. In *2021 IEEE International Symposium on Information Theory (ISIT)*. IEEE, 2786–2791.
*   Guo et al. (2023) Jiayan Guo, Lun Du, and Hengyu Liu. 2023. GPT4Graph: Can Large Language Models Understand Graph Structured Data? An Empirical Evaluation and Benchmarking. *arXiv preprint arXiv:2305.15066* (2023).
*   Guo et al. (2024) Siyuan Guo, Cheng Deng, Ying Wen, Hechang Chen, Yi Chang, and Jun Wang. 2024. DS-Agent: Automated Data Science by Empowering Large Language Models with Case-Based Reasoning. *ICML*.
*   Hamilton et al. (2017) Will Hamilton, Zhitao Ying, and Jure Leskovec. 2017. Inductive representation learning on large graphs. In *NeurIPS*. 1024–1034.
*   He and McAuley (2016) Ruining He and Julian McAuley. 2016. Ups and downs: Modeling the visual evolution of fashion trends with one-class collaborative filtering. In *proceedings of the 25th international conference on world wide web*. 507–517.
*   He et al. (2020) Xiangnan He, Kuan Deng, Xiang Wang, Yan Li, Yongdong Zhang, and Meng Wang. 2020. Lightgcn: Simplifying and powering graph convolution network for recommendation. In *Proceedings of the 43rd International ACM SIGIR conference on research and development in Information Retrieval*. 639–648.
*   He et al. (2017) Xiangnan He, Lizi Liao, Hanwang Zhang, Liqiang Nie, Xia Hu, and Tat-Seng Chua. 2017. Neural collaborative filtering. In *Proceedings of the 26th international conference on world wide web*. 173–182.
*   Hong et al. (2024) Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, et al. 2024. Metagpt: Meta programming for multi-agent collaborative framework. *ICLR* (2024).
*   Li and King (2020) Yaoman Li and Irwin King. 2020. AutoGraph: Automated Graph Neural Network. In *ICONIP*. 189–201.
*   Li et al. (2023) Yuhan Li, Zhixun Li, Peisong Wang, Jia Li, Xiangguo Sun, Hong Cheng, and Jeffrey Xu Yu. 2023. A survey of graph meets large language model: Progress and future directions. *arXiv preprint arXiv:2311.12399* (2023).
*   Lim et al. (2021) Derek Lim, Xiuyu Li, Felix Hohne, and Ser-Nam Lim. 2021. New benchmarks for learning on non-homophilous graphs. *arXiv preprint arXiv:2104.01404* (2021).
*   Liu et al. (2020) Haochen Liu, Xiangyu Zhao, Chong Wang, Xiaobing Liu, and Jiliang Tang. 2020. Automated embedding size search in deep recommender systems. In *Proceedings of the 43rd International ACM SIGIR Conference on Research and Development in Information Retrieval*. 2307–2316.
*   Liu et al. (2023a) Jiawei Liu, Cheng Yang, Zhiyuan Lu, Junze Chen, Yibo Li, Mengmei Zhang, Ting Bai, Yuan Fang, Lichao Sun, Philip S Yu, et al. 2023a. Towards graph foundation models: A survey and beyond. *arXiv preprint arXiv:2310.11829* (2023).
*   Liu et al. (2023c) Pengfei Liu, Weizhe Yuan, Jinlan Fu, Zhengbao Jiang, Hiroaki Hayashi, and Graham Neubig. 2023c. Pre-train, prompt, and predict: A systematic survey of prompting methods in natural language processing. *Comput. Surveys* 55, 9 (2023), 1–35.
*   Liu et al. (2023b) Zemin Liu, Xingtong Yu, Yuan Fang, and Xinming Zhang. 2023b. Graphprompt: Unifying pre-training and downstream tasks for graph neural networks. In *Proceedings of the ACM Web Conference 2023*. 417–428.
*   Lü et al. (2012) Linyuan Lü, Matúš Medo, Chi Ho Yeung, Yi-Cheng Zhang, Zi-Ke Zhang, and Tao Zhou. 2012. Recommender systems. *Physics reports* 519, 1 (2012), 1–49.
*   Lu et al. (2021) Yuanfu Lu, Xunqiang Jiang, Yuan Fang, and Chuan Shi. 2021. Learning to pre-train graph neural networks. In *Proceedings of the AAAI conference on artificial intelligence*, Vol. 35\. 4276–4284.
*   Luan et al. (2022) Sitao Luan, Chenqing Hua, Qincheng Lu, Jiaqi Zhu, Mingde Zhao, Shuyuan Zhang, Xiao-Wen Chang, and Doina Precup. 2022. Revisiting Heterophily For Graph Neural Networks. In *NeurIPS*.
*   Luo et al. (2019) Yuanfei Luo, Mengshuo Wang, Hao Zhou, Quanming Yao, Wei-Wei Tu, Yuqiang Chen, Wenyuan Dai, and Qiang Yang. 2019. Autocross: Automatic feature crossing for tabular data in real-world applications. In *Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining*. 1936–1945.
*   Pham et al. (2018) Hieu Pham, Melody Guan, Barret Zoph, Quoc Le, and Jeff Dean. 2018. Efficient Neural Architecture Search via Parameter Sharing. In *ICML*. 4092–4101.
*   Rossi et al. (2020) Emanuele Rossi, Fabrizio Frasca, Ben Chamberlain, Davide Eynard, Michael Bronstein, and Federico Monti. 2020. Sign: Scalable inception graph neural networks. *arXiv preprint arXiv:2004.11198* 7 (2020), 15.
*   Sen et al. (2008) Prithviraj Sen, Galileo Namata, Mustafa Bilgic, Lise Getoor, Brian Galligher, and Tina Eliassi-Rad. 2008. Collective classification in network data. *AI magazine* 29, 3 (2008), 93–93.
*   Shchur et al. (2018) Oleksandr Shchur, Maximilian Mumme, Aleksandar Bojchevski, and Stephan Günnemann. 2018. Pitfalls of graph neural network evaluation. *arXiv preprint arXiv:1811.05868* (2018).
*   Shen et al. (2023) Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu, and Yueting Zhuang. 2023. Hugginggpt: Solving ai tasks with chatgpt and its friends in huggingface. *arXiv preprint arXiv:2303.17580* (2023).
*   Sun et al. (2022) Mingchen Sun, Kaixiong Zhou, Xin He, Ying Wang, and Xin Wang. 2022. Gppt: Graph pre-training and prompt tuning to generalize graph neural networks. In *Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*. 1717–1727.
*   Sun et al. (2023a) Xiangguo Sun, Hong Cheng, Jia Li, Bo Liu, and Jihong Guan. 2023a. All in one: Multi-task prompting for graph neural networks. In *Proceedings of the 29th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*. 2120–2131.
*   Sun et al. (2023b) Xiaofei Sun, Xiaoya Li, Jiwei Li, Fei Wu, Shangwei Guo, Tianwei Zhang, and Guoyin Wang. 2023b. Text Classification via Large Language Models. *arXiv preprint arXiv:2305.08377* (2023).
*   Tornede et al. (2023) Alexander Tornede, Difan Deng, Theresa Eimer, Joseph Giovanelli, Aditya Mohan, Tim Ruhkopf, Sarah Segel, Daphne Theodorakopoulos, Tanja Tornede, Henning Wachsmuth, et al. 2023. AutoML in the Age of Large Language Models: Current Challenges, Future Opportunities and Risks. *arXiv preprint arXiv:2306.08107* (2023).
*   Touvron et al. (2023) Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, et al. 2023. Llama: Open and efficient foundation language models. *arXiv preprint arXiv:2302.13971* (2023).
*   Valmeekam et al. (2024) Karthik Valmeekam, Matthew Marquez, Alberto Olmo, Sarath Sreedharan, and Subbarao Kambhampati. 2024. Planbench: An extensible benchmark for evaluating large language models on planning and reasoning about change. *Advances in Neural Information Processing Systems* 36 (2024).
*   Wale and Karypis (2006) N Wale and G Karypis. 2006. Comparison of Descriptor Spaces for Chemical Compound Retrieval and Classification. In *ICDM*. 678–689.
*   Wang et al. (2024) Heng Wang, Shangbin Feng, Tianxing He, Zhaoxuan Tan, Xiaochuang Han, and Yulia Tsvetkov. 2024. Can language models solve graph problems in natural language? *Advances in Neural Information Processing Systems* 36 (2024).
*   Wang et al. (2023a) Haishuai Wang, Yang Gao, Xin Zheng, Peng Zhang, Hongyang Chen, and Jiajun Bu. 2023a. Graph Neural Architecture Search with GPT-4. *arXiv preprint arXiv:2310.01436* (2023).
*   Wang et al. (2023b) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. 2023b. A Survey on Large Language Model based Autonomous Agents. *arXiv preprint arXiv:2308.11432* (2023).
*   Wang et al. (2019) Xiang Wang, Xiangnan He, Meng Wang, Fuli Feng, and Tat-Seng Chua. 2019. Neural graph collaborative filtering. In *Proceedings of the 42nd international ACM SIGIR conference on Research and development in Information Retrieval*. 165–174.
*   Wang et al. (2022a) Xin Wang, Ziwei Zhang, and Wenwu Zhu. 2022a. Automated Graph Machine Learning: Approaches, Libraries and Directions. *arXiv preprint arXiv:2201.01288* (2022).
*   Wang et al. (2022c) Yejing Wang, Xiangyu Zhao, Tong Xu, and Xian Wu. 2022c. Autofield: Automating feature selection in deep recommender systems. In *Proceedings of the ACM Web Conference 2022*. 1977–1986.
*   Wang et al. (2022b) Zhenyi Wang, Huan Zhao, and Chuan Shi. 2022b. Profiling the Design Space for Graph Neural Networks based Collaborative Filtering. In *WSDM*. 1109–1119.
*   Wei et al. (2022a) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022a. Chain-of-thought prompting elicits reasoning in large language models. *Advances in neural information processing systems* 35 (2022), 24824–24837.
*   Wei et al. (2023) Lanning Wei, Zhiqiang He, Huan Zhao, and Quanming Yao. 2023. Search to Capture Long-range Dependency with Stacking GNNs for Graph Classification. In *Proceedings of the ACM Web Conference 2023*. 588–598.
*   Wei et al. (2022b) Lanning Wei, Huan Zhao, and Zhiqiang He. 2022b. Designing the Topology of Graph Neural Networks: A Novel Feature Fusion Perspective. In *The WebConf*. 1381–1391.
*   Wei et al. (2021) Lanning Wei, Huan Zhao, Quanming Yao, and Zhiqiang He. 2021. Pooling architecture search for graph classification. In *CIKM*. 2091–2100.
*   weng (2023) Lilian weng. June 23, 2023. LLM Powered Autonomous Agents. [https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/).
*   Xiong et al. (2024a) Miao Xiong, Zhiyuan Hu, Xinyang Lu, Yifei Li, Jie Fu, Junxian He, and Bryan Hooi. 2024a. Can llms express their uncertainty? an empirical evaluation of confidence elicitation in llms. *ICLR* (2024).
*   Xiong et al. (2024b) Siheng Xiong, Ali Payani, Ramana Kompella, and Faramarz Fekri. 2024b. Large language models can learn temporal reasoning. *arXiv preprint arXiv:2401.06853* (2024).
*   Xu et al. (2018) Keyulu Xu, Chengtao Li, Yonglong Tian, Tomohiro Sonobe, Ken-ichi Kawarabayashi, and Stefanie Jegelka. 2018. Representation learning on graphs with jumping knowledge networks. In *ICML*. 5453–5462.
*   Yao et al. (2018) Quanming Yao, Mengshuo Wang, Yuqiang Chen, Wenyuan Dai, Yu-Feng Li, Wei-Wei Tu, Qiang Yang, and Yang Yu. 2018. Taking human out of learning applications: A survey on automated machine learning. *arXiv preprint arXiv:1810.13306* (2018).
*   Ying et al. (2018) Zhitao Ying, Jiaxuan You, Christopher Morris, Xiang Ren, Will Hamilton, and Jure Leskovec. 2018. Hierarchical graph representation learning with differentiable pooling. In *NeurIPS*. 4800–4810.
*   You et al. (2020) Jiaxuan You, Rex Ying, and Jure Leskovec. 2020. Design Space for Graph Neural Networks. In *NeurIPS*.
*   Yu et al. (2023) Caiyang Yu, Xianggen Liu, Chenwei Tang, Wentao Feng, and Jiancheng Lv. 2023. GPT-NAS: Neural Architecture Search with the Generative Pre-Trained Model. *arXiv preprint arXiv:2305.05351* (2023).
*   Yu et al. (2024) Xingtong Yu, Chang Zhou, Yuan Fang, and Xinming Zhang. 2024. MultiGPrompt for Multi-Task Pre-Training and Prompting on Graphs. *The WebConf*.
*   Yuan et al. (2023) Quan Yuan, Mehran Kazemi, Xin Xu, Isaac Noble, Vaiva Imbrasaite, and Deepak Ramachandran. 2023. TaskLAMA: Probing the Complex Task Understanding of Language Models. *arXiv preprint arXiv:2308.15299* (2023).
*   Zhang et al. (2023a) Hongxin Zhang, Weihua Du, Jiaming Shan, Qinhong Zhou, Yilun Du, Joshua B Tenenbaum, Tianmin Shu, and Chuang Gan. 2023a. Building Cooperative Embodied Agents Modularly with Large Language Models. *arXiv preprint arXiv:2307.02485* (2023).
*   Zhang (2023) Jiawei Zhang. 2023. Graph-ToolFormer: To Empower LLMs with Graph Reasoning Ability via Prompt Augmented by ChatGPT. *arXiv preprint arXiv:2304.11116* (2023).
*   Zhang et al. (2018) Muhan Zhang, Zhicheng Cui, Marion Neumann, and Yixin Chen. 2018. An end-to-end deep learning architecture for graph classification. In *AAAI*.
*   Zhang et al. (2023b) Shujian Zhang, Chengyue Gong, Lemeng Wu, Xingchao Liu, and Mingyuan Zhou. 2023b. AutoML-GPT: Automatic Machine Learning with GPT. *arXiv preprint arXiv:2305.02499* (2023).
*   Zhang et al. (2023c) Xuan Zhang, Limei Wang, Jacob Helwig, Youzhi Luo, Cong Fu, Yaochen Xie, Meng Liu, Yuchao Lin, Zhao Xu, Keqiang Yan, et al. 2023c. Artificial Intelligence for Science in Quantum, Atomistic, and Continuum Systems. *arXiv preprint arXiv:2307.08423* (2023).
*   Zhang et al. (2021) Ziwei Zhang, Xin Wang, and Wenwu Zhu. 2021. Automated Machine Learning on Graphs: A Survey. *arXiv preprint arXiv:2103.00742* (2021).
*   Zhao et al. (2020) Huan Zhao, Lanning Wei, and Quanming Yao. 2020. Simplifying Architecture Search for Graph Neural Network.
*   Zhao et al. (2021) Huan Zhao, Quanming Yao, and Weiwei Tu. 2021. Search to aggregate neighborhood for graph neural network. In *ICDE*.
*   Zhaok et al. (2021) Xiangyu Zhaok, Haochen Liu, Wenqi Fan, Hui Liu, Jiliang Tang, Chong Wang, Ming Chen, Xudong Zheng, Xiaobing Liu, and Xiwang Yang. 2021. Autoemb: Automated embedding dimensionality search in streaming recommendations. In *2021 IEEE International Conference on Data Mining (ICDM)*. IEEE, 896–905.
*   Zheng et al. (2023) Mingkai Zheng, Xiu Su, Shan You, Fei Wang, Chen Qian, Chang Xu, and Samuel Albanie. 2023. Can GPT-4 Perform Neural Architecture Search? *arXiv preprint arXiv:2304.10970* (2023).
*   Zhu et al. (2020) Jiong Zhu, Yujun Yan, Lingxiao Zhao, Mark Heimann, Leman Akoglu, and Danai Koutra. 2020. Beyond homophily in graph neural networks: Current limitations and effective designs. In *NeurIPS*.
*   Zoph and Le (2017) Barret Zoph and Quoc V Le. 2017. Neural architecture search with reinforcement learning. *ICLR* (2017).

## Appendix A Detailed Introduction of Agents in GL-Agent

In this section, we provide the implementation details of each agent within the GL-Agent framework and how they collaborate to achieve the automated process of generating graph learning models from user instructions.

### A.1\. Instructions

As shown in Table [8](https://arxiv.org/html/2309.04565v2#A1.T8 "Table 8 ‣ A.1\. Instructions ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent"), we provide five keys when constructing the instructions, which contains the descriptions of data, learning task, evaluation metric and user preferences. It is needed to say that more complex instructions can be constructed by introducing error information or more personalized keys, which is out-of-scope of this paper.

Table 8\. The keywords in user instruction.

| Keywords | Content |
| --- | --- |
| Data $\mathcal{D}$ | The introduction of graph or the used dataset. |
| Task $\mathcal{T}$ | The learning target on the graph in natural language, e.g., recommending items to particular users or predicting the properties of specific molecular graphs., |
| Evaluation metric $\mathcal{M}$ | The evaluations of the learning target. In general, it would be the test performance or the response time. |
| Preference $\mathcal{P}$ | The prior knowledge of users in setting up the graph learning procedures, either on the feature engineering, architecture design or hyper-parameter selections., |
| Constrains $\mathcal{C}$ | The constrains on model, device, or training strategy. |

### A.2\. Manager Agent

The manager agent serves as the starting point of the entire GL-Agent framework and is responsible for parsing the user’s natural language instructions to extract key information about the graph learning task. To achieve this goal, we have adopted a structured prompt template named PromptTemplate. The design of this template aims to guide LLMs in analyzing user requests and generating responses in a predefined format. The specific prompt design is illustrated in Table [9](https://arxiv.org/html/2309.04565v2#A1.T9 "Table 9 ‣ A.4\. Response Agent ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent"). The expected LLM output and its content are detailed in Table [8](https://arxiv.org/html/2309.04565v2#A1.T8 "Table 8 ‣ A.1\. Instructions ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent"). Based on the above prompt, the manager agent will invoke LLM to extract relevant information from the user request and store the information in key-value pair for use by subsequent agents.

### A.3\. Graph Learning Agents

In the data agent, configuration agent, searching agent, and tuning agent, we also use the structured prompt template mentioned above to guide and standardize the LLMs’ output.

The Data Agent plays a crucial role by actively browsing through the PyTorch Geometric (PyG) document to identify and select the most appropriate data pre-processing methods and feature engineering techniques in line with the specific requirements of the task. This step ensures that the data is provided in the most suitable form for model learning, the prompt used is shown in Table [10](https://arxiv.org/html/2309.04565v2#A1.T10 "Table 10 ‣ A.4\. Response Agent ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").

In the Configuration Agent, we first select suitable modules and operations from the PyTorch Geometric (PyG) document based on user preferences to configure the search space (specific modules and operations are provided in Appendix [B.1](https://arxiv.org/html/2309.04565v2#A2.SS1 "B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent")). Then, using the configured search space along with the formatted instructions preserved in the shared memory, we choose the appropriate search algorithm. the prompt used is shown in Table [11](https://arxiv.org/html/2309.04565v2#A1.T11 "Table 11 ‣ A.4\. Response Agent ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").. The configured search space and search algorithm are also stored in memory in the form of key-value pairs, available for use by subsequent agents. The searched GNN architecture is also stored in memory for subsequent use.

In the searching agent, we create bash scripts to externally interact with AutoML methods. These scripts are specifically designed to invoke AutoML’s APIs for conducting Neural Architecture Search (NAS), based on the search space and search algorithm previously set up by the Configuration Agent and feature engineering functions selected by Data Agent. After the search process is completed, we parse the generated logs and output files, extract detailed information about the searched GNN architecture, summarize the search process and results using the LLM, and generate an easily understandable report, the prompt used is shown in Table [12](https://arxiv.org/html/2309.04565v2#A1.T12 "Table 12 ‣ A.4\. Response Agent ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").

After searching the optimal architecture, the tuning agent continues to refine the hyper-parameters through interactions with AutoML by generating bash scripts, aiming to further enhance the model’s performance. Once the fine-tuning process is completed, key performance metrics and model configurations are extracted through LLM. This includes the best hyper-parameter combinations found during fine-tuning, the model’s performance on validation and test sets, and any specific model behaviors or observed issues. The prompt used for this agent is detailed in Table [13](https://arxiv.org/html/2309.04565v2#A1.T13 "Table 13 ‣ A.4\. Response Agent ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").

### A.4\. Response Agent

Finally, the response agent is responsible for integrating all the outputs from the previous steps and generating the final model summary report. This report provides a detailed description of the model’s architecture, hyper-parameter configuration, performance evaluation results, and possible improvement recommendations, offering users a comprehensive overview of the model, the prompt used is shown in Table [14](https://arxiv.org/html/2309.04565v2#A1.T14 "Table 14 ‣ A.4\. Response Agent ‣ Appendix A Detailed Introduction of Agents in GL-Agent ‣ A Versatile Graph Learning Approach through LLM-based Agent").

Table 9\. The Prompt Design of Manager Agent.

| The Prompt Design of Manager Agent |
| --- |
| # Profile |
| You are a Graph Learning Specialist with exceptional abilities in interpreting complex user instructions and translating them into structured, actionable data formats. |
|  |
| # Objective |
| Your task is to deconstruct user instructions related to graph learning into detailed, executable plans, ensuring all elements of the request are accurately captured and clearly categorized. |
|  |
| # Functions |
| 1\. **Task Plan Function** |
| - **Purpose**: To meticulously parse and interpret user instructions, identifying key information pertinent to graph learning tasks. |
| - **Input**: User instructions. |
| - **Output**: A Python Dictionary of the task highlighting its level (node, link, or graph), type of learning task (classification, regression, etc.), the name of dataset, and evaluation metrics. |
|  |
| # Human Expertise |
| Human expertise in this context involves a deep understanding of how to interpret complex user requests related to graph learning. The expertise includes: |
| 1\. **Task Categorization**: Identifying the level of graph learning tasks (node, link, or graph-level) based on the user’s description. |
| 2\. **Task Type Determination**: Distinguishing between different types of learning tasks, such as classification or regression, based on the details provided in the user’s request. |
| 3\. **Evaluation Metric Selection**: Selecting appropriate evaluation metrics. |
| 4\. **Preference Identification**: Noting any specific operational preferences mentioned by the user. |
| 5\. **Dataset Identification**: Accurately identifying the relevant datasets mentioned in the user’s request. |
|  |

Table 10\. The Prompt Design of Data Agent.

| The Prompt Design of Data Agent. |
| --- |
| # Profile |
| You are a Graph Learning Specialist with specialized skills in navigating and utilizing document structures for optimizing machine learning workflows. Your expertise includes extracting, parsing, and interpreting complex data from documents formats to assist in feature engineering. |
|  |
| # Objective |
| Your task is to analyze the PyG documentation and user requests to identify and select appropriate feature engineering techniques that are most effective for the specified task plan. This involves extracting relevant techniques from the documentation that directly align with the user’s objectives and requirements, thereby enhancing the model’s performance. |
|  |
| # Functions |
| 1\. **Feature Engineering Selection Function** |
| - **Purpose**: To determine the best feature engineering techniques from the provided documentation that align with the user’s request and task requirements. |
| - **Input**: User request ${}^{\prime}user\_req^{\prime}$, task plan details ${}^{\prime}task\_plan^{\prime}$, and specific documentation content ${}^{\prime}content^{\prime}$. |
| - **Output**: A list of up to three selected feature engineering techniques, formatted as a Python Dictionary ensuring they are directly applicable and beneficial for enhancing the model’s performance. |
|  |
| # Human Expertise |
| Human experts are crucial in the process of selecting effective feature engineering techniques from PyG documentation. Their expertise involves analyzing the task requirements, understanding the types of datasets involved, and comprehending detailed descriptions of feature engineering. This knowledge enables them to choose the most relevant and beneficial feature engineering functions that align with the specific needs of the task and enhance the overall performance of the model. These decisions are based on a deep understanding of how different techniques can affect the efficiency and effectiveness of graph neural network models in various contexts. |
|  |

Table 11\. The Prompt Design of Configuration Agent.

| The Prompt Design of Configuration Agent |
| --- |
| # Profile |
| You are a Graph Learning Specialist specialized in navigating and utilizing configuration options. Your expertise allows you to effectively parse documentation, extract operational data, and apply this information to configure search space and search algorithm. |
|  |
| # Objective |
| Your primary objective is to orchestrate the configuration process of graph neural networks by selecting appropriate modules, preparing operation candidates, evaluating these candidates, and finally selecting an optimal configuration that enhances the model’s effectiveness and efficiency. |
|  |
| # Functions |
| 1\. **Module Selection Function**: Aimed at identifying the best modules for inclusion in the graph neural network based on the task’s specifics. |
| - **Input**: Task requirements and available module options. |
| - **Output**: List of modules deemed most suitable for the task. |
|  |
| 2\. **Operation Preparation Function**: Prepares the detailed list of operations from specific documentation content ${}^{\prime}content^{\prime}$ that can be performed by the selected modules. |
| - **Input**: Selected modules and specific documentation content. |
| - **Output**: Detailed operations capable of being executed by these modules. |
|  |
| 3\. **Candidate Selection Function**: Evaluates the prepared operations and selects the most promising candidates for final deployment. |
| - **Input**: List of prepared operations. |
| - **Output**: Shortlist of candidate operations for search space. |
|  |
| 4\. **Construct Search Space Function**: Constructs a comprehensive search space where different configurations can be tested and evaluated. |
| - **Input**: Candidate operations. |
| - **Output**: A structured search space. |
|  |
| 5\. **Algorithm Selection Function**: Selects the most suitable algorithm through based on the identified requirements. |
| - **Input**: The constructed search space, the task plan and the selected modules. |
| - **Output**: The most effective search algorithm for finding the network architecture. |
|  |
| # Human Expertise |
| The configuration of graph neural networks involves a sequential process. Initially, human experts select modules that align with the specific demands of the task. Following this, they outline potential operations for these modules and narrow down the choices to the most effective ones for candidate selection. The next step is constructing a search space based on the selected modules and selected candidates. Finally, human experts select an algorithm that best navigates this space to find the optimal architecture of the network. Throughout this process, human expertise ensures that each step is tailored to meet the task-specific goals and technical requirements efficiently. |
|  |

Table 12\. The Prompt Design of Searching Agent.

| The Prompt Design of Searching Agent |
| --- |
| # Profile |
| You are a Graph Learning Specialist with advanced capabilities in automating neural architecture search (NAS) for graph neural networks. Your skills include configuring execution code, performing neural architecture searching operation, and generating insightful summaries from the search processes. |
|  |
| # Objective |
| Your primary task is to streamline the process of efficiently executing search procedures to discover optimal network architectures, and effectively summarize the outcomes using the details derived from search logs. |
|  |
| # Functions |
| 1\. **Argument Configuration** |
| - **Purpose**: Sets up the parameters and arguments that control the search process. |
| - **Input**: User-defined parameters including search space, feature engineering functions, GPU preferences. |
| - **Output**: A configured execution script ready for the NAS (Neural Architecture Search) process. |
|  |
| 2\. **Code Execution** |
| - **Purpose**: Employs advanced AutoML techniques to systematically explore network architectures. |
| - **Input**: The configured execution script. |
| - **Output: Search log from the exploration of network architectures. |
|  |
| 3\. **Summary Generation** |
| - **Purpose**: Analyzes logs from the NAS (Neural Architecture Search) process to construct detailed summaries. |
| - **Input**: Logs and data generated during the NAS process. |
| - **Output**: Summaries that capture key results and strategic insights, aiding in the evaluation of the search outcomes. |
|  |
| # Human Expertise |
| Human experts are essential in setting up and configuring the execution codes tailored to the chosen feature engineering functions and the constructed search space. Following the completion of the AutoML process, they critically assess and interpret the log data to create comprehensive summaries that encapsulate the search findings and provide valuable insights into the effectiveness of the tested architectures. |

Table 13\. The Prompt Design of Tuning Agent.

| The Prompt Design of Tuning Agent |
| --- |
| # Profile |
| You are a Graph Learning Specialist with advanced capabilities in automating the fine-tuning of the searched architectures for graph neural networks. Your expertise includes configuring execution parameters, managing fine-tuning processes, and synthesizing outcomes into actionable insights. |
|  |
| # Objective |
| Your primary task is to automate the fine-tuning of neural architectures, setting up fine-tuning execution code, executing the fine-tuning code, and generating detailed summaries of the outcomes. |
|  |
| # Functions |
| 1\. **Argument Configuration** |
| - **Purpose**: Sets up the parameters and arguments that control the fine-tune process. |
| - **Input**: The search space, feature engineering functions and other parameters. |
| - **Output**: Configured execution script ready for fine-tuning execution. |
|  |
| 2\. **Code Execution** |
| - **Purpose**: Runs the fine-tuning scripts to fine-tune the searched neural network. |
| - **Input**: The configured execution script. |
| - **Output**: Tuning log from the fine-tuning of the searched network architecture. |
|  |
| 3\. **Summary Generation** |
| - **Purpose**: Synthesizes data from the fine-tuning process into insightful summaries. |
| - **Input**: Logs and outputs generated during the fine-tuning phase. |
| - **Output**: Reports summaries that highlight improvements, effectiveness, and optimization areas of the fine-tuned architectures. |
|  |
| # Human Expertise |
| Human experts play a crucial role in configuring the execution codes to align with the selected feature engineering functions and the search space. After executing the fine-tuning process, they analyze the results to generate detailed summaries that highlight key outcomes and insights, thus providing a deeper understanding of the effectiveness of the tested network architectures. |

Table 14\. The Prompt Design of Response Agent.

| The Prompt Design of Response Agent |
| --- |
| # Profile |
| You are a Graph Learning Specialist tasked with synthesizing various information from neural network tuning and architecture search into a structured format. Your capabilities include interpreting complex data outputs, generating comprehensive summaries, and structuring these into actionable insights. |
|  |
| # Objective |
| Your primary objective is to consolidate the results of graph neural network tuning and architecture searches into a clear, structured summary that outlines prediction outcomes, architecture details, hyperparameters, and resource consumption. |
|  |
| # Functions |
| 1\. ** Response Generation** |
| - **Purpose**: To aggregate and synthesize information and generate a comprehensive response. |
| - **Input**: Data from tuning results, architecture files, and other relevant agents. |
| - **Output**: A Python Dictionary that clearly delineates prediction results, architecture specifics, optimized hyperparameters, and resource usage. |
|  |
| # Human Expertise |
| Human experts guide the process by ensuring the accurate interpretation of data, the applicability of the synthesized information, and the correctness of the output format. |
|  |

## Appendix B Experiments

### B.1\. Implementation Instance

#### B.1.1\. Node-level

The instance is constructed based on the differentiable automated GNN design method F2GNN (Wei et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib54)), which focus on designing the aggregation operations and the GNN topology, i.e., the connections between these operations. The implementation details are provided in the following.

Architecture backbone. We begin by presenting the architecture backbone that is tailored for node-level tasks. As illustrated in Fig. [3](https://arxiv.org/html/2309.04565v2#A2.F3 "Figure 3 ‣ B.1.1\. Node-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), F2GNN proposed selection $f_{s}$ and fusion $f_{f}$ module to design the GNN topology, in which the former is designed for each aggregation operation to select inputs from previous operations, and the latter is designed to integrate these inputs that could be used by the following aggregation operations. Finally, the aggregation operation is provided to update node representations H, and it is relevant for all tasks.

![Refer to caption](img/98488104d80a5f4f3733fd12d48ce22f.png)

Figure 3\. The designed architecture backbone from (Wei et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib54)).

Configurations on search space and hyper-parameters. Based on these operation modules, we pre-define basic operations in Table [15](https://arxiv.org/html/2309.04565v2#A2.T15 "Table 15 ‣ B.1.1\. Node-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), and it can be further extend or modified by the configuration agent. As shown in Fig. [2](https://arxiv.org/html/2309.04565v2#S3.F2 "Figure 2 ‣ 3\. Method ‣ A Versatile Graph Learning Approach through LLM-based Agent"), the configuration agent constructs the search space and hyper-parameters by selecting proper modules and then revising their candidates according to the corresponding documents. Compared with the fixed search space used in existing NAS-based methods, the proposed method can configure the search space based on the prior knowledge from users’ instructions automatically, leading to better generalization to real-world data.

When facing node-level tasks, we implement GL-Agent based on the method F2GNN (Wei et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib54)). The basic operations used in the module are provided in Table [15](https://arxiv.org/html/2309.04565v2#A2.T15 "Table 15 ‣ B.1.1\. Node-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent").

Table 15\. The candidates of architectures and hyper-parameters used in node-level tasks.

|  | Module | Candidates |
| Model | Aggregation | GCN, SAGE |
| Selection | ZERO, IDENTITY |
| Fusion | sum, mean |
| Hyper- parameters | Learning rate | [0.001, 0.005] |
| Weight decay | [0.0001, 0.0005] |
| Dropout rate | [0, 0.5] |
| Activation | ReLU |

#### B.1.2\. Graph-level

The graph-level implementation instance is constructed based on method LRGNN (Wei et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib53)), which search to capture the long-range dependency with deep stacked GNNs. As shown in Fig. [4](https://arxiv.org/html/2309.04565v2#A2.F4 "Figure 4 ‣ B.1.2\. Graph-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), the framework are constructed by several cells and in which the selection, fusion and aggregation modules are provided in the same way as shown in Fig. [3](https://arxiv.org/html/2309.04565v2#A2.F3 "Figure 3 ‣ B.1.1\. Node-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"). A readout module is provided to generate the graph representations for this task at the end of framework.

![Refer to caption](img/d96c3dcede30169ad230fec6fce73a46.png)

Figure 4\. The framework used in LRGNN (Wei et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib53)), in which the yellow color nodes represent the pre- and port-processing operations and the green nodes represent the module of selection, fusion and aggregation.

Based on the given framework, we set the basic operations as shown in Table [16](https://arxiv.org/html/2309.04565v2#A2.T16 "Table 16 ‣ B.1.2\. Graph-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), on which the LLM-based configuration agents will add or remove operations following the user instructions.

Table 16\. The candidates of architectures and hyper-parameters used in graph-level tasks.

|  | Module | Candidates |
| --- | --- | --- |
| Model | Aggregation | GCN, SAGE |
| Selection | ZERO, IDENTITY |
| Fusion | sum, mean |
| Readout | global_sum, Global_mean |
| Hyper- parameters | Learning rate | [0.01, 0.05] |
| Weight decay | [0.001, 0.005] |
| Dropout rate | [0, 0.5] |
| Activation | ReLU |

#### B.1.3\. Link-level

For link-level, the instance is constructed based on method Prof-CF (Wang et al., [2022a](https://arxiv.org/html/2309.04565v2#bib.bib49)), which search for GNN-based two-tower collaborative filtering functions. As shown in Fig. [5](https://arxiv.org/html/2309.04565v2#A2.F5 "Figure 5 ‣ B.1.3\. Link-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), the framework contains diverse design modules, including message function, aggregation, and activation in each GNN layer, as well as the layer combination, component combination, and interaction functions used beyond layer.

![Refer to caption](img/3d6216c5700dacc5f1c6d305f032e80a.png)

Figure 5\. The framework used in Prof-CF (Wang et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib51)).

Based on the framework and the constructed search space in Prof-CF, we design a basic candidates as shown in Table [17](https://arxiv.org/html/2309.04565v2#A2.T17 "Table 17 ‣ B.1.3\. Link-level ‣ B.1\. Implementation Instance ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), on top of which the configuration agents will update the candidates following the instructions.

Table 17\. The candidates of architectures and hyper-parameters used in link-level tasks.

|  | Module | Candidates |
| Model | Message Function | Identity, Hadamard |
| Aggregation | NONE, GCN, SAGE |
| Layer Number | 1, 3 |
| Layer Combination | STACK, SUM |
|  | Component Number | 1,2,3,4 |
|  | Componnet Combination | MEAN |
|  | Interaction Function | DOT PRODUCT, CONCAT+MLP |
| Hyper- parameters | Learning rate | [0.01, 0.05] |
| Weight decay | [0.001, 0.005] |
| Dropout rate | [0, 0.5] |
| Activation | ReLU,IDENTITY |

### B.2\. Datasets

In the node classification task, Cora (Sen et al., [2008](https://arxiv.org/html/2309.04565v2#bib.bib35)) is the citation network where each node represents a paper, and each edge represents the citation relation between two papers; Computers and Photo (Shchur et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib36)) are the Amazon co-purchase graphs where nodes represent goods that are linked by an edge if these goods are frequently bought together; Physics (Shchur et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib36)) is a co-authorship graph where nodes are authors who are connected by an edge if they co-author a paper.

The statistics of these datasets are provided in Table [20](https://arxiv.org/html/2309.04565v2#A2.T20 "Table 20 ‣ B.2\. Datasets ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"). In the graph classification task, NCI1 and NCI109 (Wale and Karypis, [2006](https://arxiv.org/html/2309.04565v2#bib.bib44)) are datasets of chemical compounds; DD and PROTEINS (Dobson and Doig, [2003](https://arxiv.org/html/2309.04565v2#bib.bib7)) are datasets both protein graphs. The statistics of these datasets are provided in Table [18](https://arxiv.org/html/2309.04565v2#A2.T18 "Table 18 ‣ B.2\. Datasets ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"). In the item ranking task, two datasets are adopted. Epinions is a graph of consumer reviews, in which node represent the users and the edges represent trust relationship between users. Amazon-Sports is a e-commerce dataset which contains the subset of sports category from (He and McAuley, [2016](https://arxiv.org/html/2309.04565v2#bib.bib18)). The statistics of these datasets are provided in Table [19](https://arxiv.org/html/2309.04565v2#A2.T19 "Table 19 ‣ B.2\. Datasets ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent").

Table 18\. The statistics of four graph classification datasets.

| Dataset | # Graphs | # Feature | # Classes | Avg.# of Nodes | Avg.# of Edges |
| --- | --- | --- | --- | --- | --- |
| NCI1 (Wale and Karypis, [2006](https://arxiv.org/html/2309.04565v2#bib.bib44)) | 4,110 | 89 | 2 | 29.87 | 32.3 |
| NCI109 (Wale and Karypis, [2006](https://arxiv.org/html/2309.04565v2#bib.bib44)) | 4,127 | 38 | 2 | 29.69 | 32.13 |
| DD (Dobson and Doig, [2003](https://arxiv.org/html/2309.04565v2#bib.bib7)) | 1,178 | 89 | 2 | 384.3 | 715.7 |
| PROTEINS (Dobson and Doig, [2003](https://arxiv.org/html/2309.04565v2#bib.bib7)) | 1,113 | 3 | 2 | 39.1 | 72.8 |

Table 19\. The statistics of two datasets used in item ranking task.

|  | Epinions | Amazon-Sports |
| --- | --- | --- |
| # Users | 40,163 | 11,435 |
| # Items | 139,738 | 5, 405 |
| # Interactions | 664, 824 | 108, 004 |
| # Rating Scale | [1,5] | [1, 5] |
| Density | 0.012% | 0.17% |

Table 20\. The statistics of four node classification datasets.

| Datasets | #Nodes | #Edges | #Features | #Classes |
| --- | --- | --- | --- | --- |
| Cora (Sen et al., [2008](https://arxiv.org/html/2309.04565v2#bib.bib35)) | 2,708 | 5,278 | 1,433 | 7 |
| Computers (Shchur et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib36)) | 13,381 | 245,778 | 767 | 10 |
| Photo (Shchur et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib36)) | 7,487 | 238,162 | 745 | 8 |
| Physics (Shchur et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib36)) | 34,493 | 495,924 | 8,415 | 5 |
| genius (Lim et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib24)) | 421,961 | 984, 979 | 12 | 2 |

### B.3\. Baselines

We use three types of baselines in this paper to evaluate the versatility of GL-Agent, i.e., the human-designed widely used baselines and SOTA methods used in recent two years; the AutoML-based method; and the baselines that suggested by LLMs directly. In the following, we introduce the baselines used in each task, and then analyze the construction of LLL-GNN methods in each dataset.

Node-level. We adopt the (1) human-designed method. Four-layer GCN and GIN baseline; GPR-GNN (Chien et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib5)) that learns the weights of each layer based on generalized PageRank, and ACM-GCN (Luan et al., [2022](https://arxiv.org/html/2309.04565v2#bib.bib31)) that adaptive mixing the channel information from low-/high-/full-frequency, and the configuration of these two methods are followed the original paper. (2) NAS-based method SANE (Zhao et al., [2021](https://arxiv.org/html/2309.04565v2#bib.bib73)) that learns the connections based on JKNet (Xu et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib59)) and F2GNN (Wei et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib54)) that designs the network topology from the feature fusion perspective. We employ the official code and then search on the target dataset.

Graph-level. We adopt the (1) the human-designed global pooling method. Four-layer GCN and GIN baseline with global add readout operations; DGCNN (Zhang et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib68)) that learn the graph representation based on the selected top-ranked nodes; and the hierarchical pooling methods DiffPool (Ying et al., [2018](https://arxiv.org/html/2309.04565v2#bib.bib61)) to learn the hierarchy nature in the graphs; (2) NAS-based method. LRGNN (Wei et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib53)) that design the network topology to capture the long-range dependency. The experiments are conducted based on the official code provided by this paper.

Link-level. In the item ranking task, we adopt (1) the human-designed method. NCF (He et al., [2017](https://arxiv.org/html/2309.04565v2#bib.bib20)) leverages neural networks to learn a more expressive interaction functions; NGCF (Wang et al., [2019](https://arxiv.org/html/2309.04565v2#bib.bib48)) and LightGCN (He et al., [2020](https://arxiv.org/html/2309.04565v2#bib.bib19)) are the representative GNN-based collaborative filtering methods, where the former adopts hadamard product and the latter uses identity function in the message function. (2) NAS-based method. Prof-CF (Wang et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib51)) aim to design an expressive search space by pruning operations. We conduct the experiments following the settings and using the codes provided by this paper.

The construction of LLM-GNN baseline. Apart from that , we provide the LLM-GNN baseline, for which the GNN and hyper-parameters are suggested by LLM (GPT-3.5-turbo) given given the dataset description and statistics, the details can be found in Table [21](https://arxiv.org/html/2309.04565v2#A2.T21 "Table 21 ‣ B.3\. Baselines ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"). They are obtained based on the following prompts:

Your are an expert on graph learning. Firstly, could you please describe the dataset {$\mathcal{D}$} used in the task {$\mathcal{T}$}. Then, you can suggest one GNN that could achieve better performance on this dataset. Here are the design dimensions you can refer to: the aggregation operation (message-passing layer), the activation function, the layer numbers of the designed GNN, the skip-connections beyond layer, the function that integrating the features from different layers, the hyper-parameters, and etc.

For the non-homophilous dataset genius, we further provide the dataset introduction and experimental observations as shown in the following:

This is a non-homophilous dataset that connected nodes may have different labels. For this dataset, using MLPs may have better performance than general GNNs.

For the item ranking task, we provide one additional sentence in the prompt:

Multiple GNNs can be added when extracting the results, and then you can choose the numbers of GNNs and the combination function.

Table 21\. The suggested solutions used in LLM-GNN baseline.

| Data | Suggestions |
| --- | --- |
| Cora | GNN: Two-layer stakcing-based GCN. Hyper-parameters: { hidden size:16, dropout ratio:0, learning rate:0.01, weight decay:5e-4 } |
| Computer | GNN: Two-layer stakcing-based GCN. Hyper-parameters: { ’hidden’:256, ’dropout’:0.1, ’lr’:0.01, ’l2’:5e-4,’ } |
| Photo | GNN: Two-layer GCN in which the aggregation results are concatenated to predict the node labels. Hyper-parameters: { ’hidden’:128, ’dropout’:0.1, ’lr’:0.005, ’l2’:5e-4, } |
| Physics | GNN: Two-layer GCN in which the aggregation results are concatenated to predict the node labels. Hyper-parameters: { ’hidden’:128, ’dropout’:0.1, ’lr’:0.005, ’l2’:5e-4, } |
| genius | GNN: Two-layer GraphSAGE with mean aggregator. Hyper-parameters: {‘activation’: ReLU, hidden size: 128, lr: 0.01, ’l2’:5e-4}. |
| DD | GNN: Three-layer GIN in which the aggregation results are added, and graph representation vector is obtained with Global sum readout operation. Hyper-parameters: { ’hidden’:128, ’dropout’:0.1, ’lr’:0.01, ’l2’:5e-4 }. |
| PROTEINS | GNN: Two-layer stakcing-based GIN, and graph representation vector is obtained with Global sum readout operation. Hyper-parameters: { ’hidden’:32, ’dropout’:0, ’lr’:0.01, ’l2’:5e-4 }. |
| NCI1 | GNN: Three-layer GIN in which the aggregation results are added, and graph representation vector is obtained with Global sum readout operation. Hyper-parameters: { ’hidden’:64, ’dropout’:0.0, ’lr’:0.01, ’l2’:5e-4 }. |
| NCI109 | GNN: Three-layer GIN in which the aggregation results are added, and graph representation vector is obtained with Global sum readout operation. Hyper-parameters: { ’hidden’:128, ’dropout’:0.1, ’lr’:0.01, ’l2’:5e-4 } |
| Epsonion | Three components are used in the GNN. In each component, two-layer stakcing-based GCN are employed, and these components are concatenated to obtain the node representations. The user and item rep- resentations are first concatenated or summed up, and then fed into an MLP for prediction |
| Amazon-Sports | GNN: Two-layer GAT in which the aggregation results are concatenated to formulate the final node representations. The Hadamard operation is adopted when calculating the messages. The final prediction is obtained based on the dot product of the user and item representations. |

### B.4\. Performance Comparisons

In this section, we show the performance comparisons with the baselines to evaluate the effectiveness of the proposed method and the implemented instance in graph learning over different tasks. As indicated in Table [22](https://arxiv.org/html/2309.04565v2#A2.T22 "Table 22 ‣ B.4\. Performance Comparisons ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), our method outperforms all others across four node classification datasets, even when compared to NAS-based methods that focus on either architecture topology or aggregation operation. For LLM-GNN, where the GNN and hyper-parameters are directly suggested by LLM, the performance is subpar compared to the GCN baseline due to inappropriate hyper-parameters. They fail to outperform well-designed baselines provided by research, such as GPRGNN and ACM-GNN used in this table. This highlights the challenge LLMs face in keeping up-to-date with the latest knowledge in graph learning. The suboptimal performance of the suggested GCN baseline further supports this conclusion. In contrast, our method, which designs the flexible and versatile graph learning approach with the assistance of LLM-based agents and AutoML, rather than solely relying on LLMs, exhibits superior performance. Similar results are observed in the graph classification datasets in Table [23](https://arxiv.org/html/2309.04565v2#A2.T23 "Table 23 ‣ B.4\. Performance Comparisons ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"). LLM-GNN struggles to outperform existing methods, whether they overlook hierarchical information or use models with insufficient depth (Wei et al., [2023](https://arxiv.org/html/2309.04565v2#bib.bib53), [2021](https://arxiv.org/html/2309.04565v2#bib.bib55)).

For the item ranking task, we evaluate the Recall@20 performance on two datasets as shown in Table [5](https://arxiv.org/html/2309.04565v2#S4.T5 "Table 5 ‣ 4.3.1\. Correctness of Agent Output ‣ 4.3\. Evaluation of Versatility ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"). It can be observed that GL-Agent achieves higher performance than human-designed methods, which demonstrates its effectiveness. When compared to Prod-CF (Wang et al., [2022b](https://arxiv.org/html/2309.04565v2#bib.bib51)), which GL-Agent is based on, it neglects multiple components in GNNs as shown in Table [2](https://arxiv.org/html/2309.04565v2#S4.T2 "Table 2 ‣ 4.2.2\. Baselines ‣ 4.2\. Experimental settings ‣ 4\. Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent"), resulting in inferior performance. Furthermore, when we use LLM to directly suggest GNNs, we mention multiple components in the prompt, but only Epinions considers this design dimension when designing GNNs. These findings empirically demonstrate the outdated information maintained in LLMs and their limited ability to design more powerful GNNs based on the latest prior knowledge.

The evaluations conducted on three tasks demonstrate the feasibility of GL-Agent in managing diverse user requirements that may arise in real-world scenarios. It has the potential to achieve superior or comparable performance to human-designed methods, using only instructions. This makes it a flexible and user-friendly tool. Given its low requirements for coding ability and domain knowledge of graph learning, GL-Agent is particularly accessible to non-expert users, including those unfamiliar with graph learning. From this perspective, it serves as a versatile tool for designing effective graph learning solutions.

Table 22\. The performance comparisons on the node classification task.

|  | Cora | Photo | Computer | Physics | Avg. Rank |
| GL-Agent | 86.81(0.40) | 96.40(0.16) | 92.47(0.20) | 96.93(0.98) | 2.25 |
| GCN | 85.68(0.61) | 93.13(0.27) | 90.52(0.42) | 95.97(0.14) | 5.5 |
| GIN | 83.83(1.36) | 92.67(0.57) | 88.67(1.21) | 95.79(0.16) | 7.5 |
| GPRGNN | 87.62(0.48) | 91.93(0.26) | 88.90(0.37) | 97.51(0.21) | 4.5 |
| ACM-GCN | 86.67(0.14) | 94.35(0.65) | 88.58(0.39) | 97.89(0.23) | 4.5 |
| SANE | 86.40(0.38) | 94.53(0.22) | 90.25(0.31) | 98.28(0.13) | 3.25 |
| F2GNN | 87.42(0.42) | 95.38(0.30) | 91.42(0.26) | 96.92(0.06) | 2.75 |
| LLM-GNN | 84.64(1.04) | 93.73(0.38) | 89.20(1.16) | 96.34(0.11) | 5.75 |

Table 23\. The performance comparisons on the graph classification task.

|  | Proteins | DD | NCI1 | NCI109 | Avg. Rank |
| GL-Agent | 75.38(5.03) | 78.10(3.21) | 82.14(1.74) | 81.25(1.53) | 2 |
| GCN | 74.84(3.07) | 73.59(4.17) | 76.96(3.07) | 75.70(4.03) | 4.25 |
| GIN | 74.50(4.10) | 74.62(2.74) | 77.95(1.95) | 73.25(2.67) | 4.5 |
| DGCNN | 73.95(3.04) | 61.63(5.33) | 76.08(1.03) | 74.58(3.99) | 5.75 |
| DiffPool | 75.11(2.14) | 77.85(3.53) | 75.04(1.98) | 71.48(2.46) | 4.75 |
| LRGNN | 75.39(4.40) | 78.18(2.02) | 82.51(1.37) | 81.39(1.92) | 1 |
| LLM-GNN | 74.47(3.65) | 75.12(3.44) | 71.70(2.58) | 73.04(3.65) | 5.75 |

### B.5\. Case Study

In this section, we will present specific case studies to demonstrate the feasibility and versatility of the GL-Agent framework. Fig. [6](https://arxiv.org/html/2309.04565v2#A2.F6 "Figure 6 ‣ B.6\. Interpretability of Decisions in Graph Learning Tasks ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent") and Fig. [7](https://arxiv.org/html/2309.04565v2#A2.F7 "Figure 7 ‣ B.6\. Interpretability of Decisions in Graph Learning Tasks ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent") illustrate the entire process, from the specific request posed by the user to the final results of searched GNN models. In these figures, we clearly observe that the correctness of decisions and execution.

### B.6\. Interpretability of Decisions in Graph Learning Tasks

User requirements may involve data structures and diverse analysis goals. The interpretability of the decisions made by LLM-based agents is self-evidently important to ensure that users can clearly understand the model recommendations. Therefore, we explore in detail the thinking of LLM when parsing input and formulating strategies accordingly. Table [25](https://arxiv.org/html/2309.04565v2#A2.T25 "Table 25 ‣ B.6\. Interpretability of Decisions in Graph Learning Tasks ‣ Appendix B Experiments ‣ A Versatile Graph Learning Approach through LLM-based Agent") shows the decision-making process of each agent when handling different tasks.

Table 24\. The various user instructions.

| Number | Original User Instruction | User Instruction Paraphrased by ChatGPT-4 | User Instruction Misled by ChatGPT-4 |
| --- | --- | --- | --- |
| 1 | I have a graph, it is saved on the dataset file: Cora, in which node represent the paper and edges represent the citation relationship. The node attribute is the keywords mentioned in the paper.I want to predict the domain of the given paper. | I’m working with a graph dataset called Cora. In this dataset, each node corresponds to an academic paper, while the edges between nodes signify citation relationships among the papers. Additionally, each node is associated with attributes that capture the keywords mentioned in the respective papers. My goal is to leverage this dataset to predict the academic domain or field of study to which a given paper belongs. | I have a graph, it is stored in the dataset file: Cora, where each node supposedly represents a scientific study, and edges falsely suggest a co-authorship relation. The node attribute misleadingly includes the main topics discussed in the study. My goal is erroneously stated as predicting the academic field of the corresponding study based on these misrepresented connections. |
| 2 | I have a graph, it is saved on the dataset file: Computer. This dataset is based on the ’Computers’ section of the Amazon co-purchase graph. In this graph, nodes represent products, and edges connect products that are frequently bought together. I am looking to leverage this graph to build a recommendation system that suggests products based on co-purchase history. | I’m working with a graph dataset named "Computer", derived from the "Computers" section of the Amazon co-purchase graph. In this dataset, each node represents a product, and edges between nodes indicate products that are frequently purchased together. My objective is to utilize this graph to develop a recommendation system that suggests products based on their co-purchase history. | I possess a graph housed in the dataset named ’Computer,’ extracted from the Amazon co-purchase data focused on electronics. Within this structure, each node misleadingly symbolizes a consumer, and the edges wrongly imply financial transactions between them. My aim is to develop a system that predicts future buying trends of these consumers based on their supposed transaction history. |
| 3 | I have a graph, it is saved on the dataset file: physics. It shows authors as nodes, and if two authors wrote a paper together, they’re connected by an edge. I want to use this to find groups of authors who work closely and guess who might team up on future projects. | I’m working with a graph dataset named "physics", where nodes represent authors, and edges connect authors who have collaborated on writing a paper together. My aim is to analyze this graph to identify clusters of authors who collaborate closely and predict potential future collaborations among them. | I have a dataset stored in the ’physics’ file, where each node is mistakenly thought to represent a scientific publication, and the edges erroneously indicate that these publications are based on similar research topics. My goal is to predict future collaborative research topics among these publications, assuming they might combine their findings. |
| 4 | I have a graph, it is saved on the dataset file: actor, where each point is an actor, and a line between two actors means they’re mentioned together on a Wikipedia page. I want to look at how these actors are connected, find groups, and see how they relate to each other. | I’m analyzing a graph dataset named "actor", wherein each node represents an actor, and an edge between two actors indicates that they are mentioned together on a Wikipedia page. My objective is to examine the connections between these actors, identify groups or clusters within the network, and explore the relationships between them. | I have accessed a graph data file named ’actor’, where each point misleadingly represents a movie, and the lines between points incorrectly imply that actors from these movies have worked together on the same film set. My objective is to analyze the movie genres to find potential future collaborations among these actors based on their genre appearances. |
| 5 | I have a graph, it is saved on the dataset file: wisconsin. This graph is a map of web pages, where each node is a page and each edge is a link to another page.The pages have features based on the content and hyperlinks. My task is to categorize the pages. | I am working with a dataset titled ’Wisconsin,’ which consists of a graph stored at /data/wisconsin/. In this graph, each node corresponds to a web page, and each edge represents a hyperlink that connects one page to another. Attributes of each page include content details and hyperlink information. My task is to categorize these web pages into distinct classes based on their attributes and the structure of their hyperlinks. | I have a dataset called ’wisconsin’ that is inaccurately described as a geographic map of physical locations, with each node representing a location and each edge depicting roads connecting them. The features of these nodes supposedly include demographic data and transport links. My goal is to classify these locations based on their demographic characteristics. |
| 6 | I have a graph, it is saved on the dataset file: NCI1\. In the NCI1 dataset, each graph represents a chemical compound, where nodes correspond to atoms within the compound, and edges represent the chemical bonds between atoms. Node attributes can include chemical properties such as atom type and charge, which are key features of the atoms or compounds. I want to find one GNN that has better accuracy. | I’m working with a graph dataset named "NCI1". In this dataset, each graph represents a chemical compound, with nodes corresponding to atoms within the compound and edges representing the chemical bonds between atoms. Additionally, node attributes capture important chemical properties such as atom type and charge, which are crucial features for characterizing the atoms or compounds. My goal is to identify a Graph Neural Network (GNN) model that achieves higher accuracy in analyzing this chemical compound dataset. | I have a graph dataset named ’NCI1’. Each graph in this dataset represents a chemical compound, with nodes corresponding to atoms and edges representing the chemical bonds between them. Node attributes include important chemical properties like atom type and charge, which are essential for characterizing the atoms or compounds. I aim to identify a Graph Neural Network (GNN) model that offers superior accuracy in analyzing these chemical structures. |
| 7 | I have a graph, it is saved on the dataset file: Epinions, in which nodes represent the users and the edges represent the trust relationship between users. The node attribute could include user activity metrics, ratings, or other relevant information that signifies the user’s influence or trustworthiness. I want to predict the potential of a trust relationship forming between two users. | I’m analyzing a graph dataset named "Epinions", where nodes represent users and edges represent the trust relationships between users. Node attributes may include user activity metrics, ratings, or other relevant information indicating the user’s influence or trustworthiness. My objective is to develop a predictive model that can anticipate the likelihood of a trust relationship forming between two users based on the available data in the graph. | I am working with a graph dataset named ’Epinions’, stored in a specific dataset file. In this graph, each node represents a user, and the edges denote the trust relationships existing between users. The node attributes might include various user activity metrics, ratings, or other pertinent information that highlights a user’s influence or trustworthiness. My objective is to develop a predictive model that assesses the potential for forming a trust relationship between two users. |
| 8 | In a movie recommendation system, the data is stored at the path: /data/movies/. Within this system, user nodes represent users, and edges represent their social connections. I want to predict users’ movie preferences, i.e., which types of movies users are likely to enjoy. | Within our movie recommendation system, the dataset is located at /data/movies/. In this system, user nodes represent individual users, while edges symbolize their social connections. My goal is to predict users’ movie preferences, specifically identifying the types of movies users are likely to enjoy. | In our movie recommendation system, the dataset is stored at the location /data/movies/. In this system, each node within the graph represents a user, while the edges illustrate the social connections between these users. The goal is to predict the movie preferences of these users, specifically determining which types of movies are likely to appeal to each user based on their social connections. |
| 9 | In a protein-protein interaction network, the dataset is located at /data/proteins/. Nodes represent proteins and edges represent interactions between them. I’m interested in classifying proteins based on their functions. The node attributes include the type of protein and its biological properties. I believe GCNConv might be a good fit for this task. | In the protein-protein interaction network, the dataset is stored at /data/proteins/. Nodes correspond to proteins, while edges represent interactions between them. My objective is to classify proteins based on their functions. The node attributes include the type of protein and its biological properties. I believe utilizing the GCNConv (Graph Convolutional Network Convolution) method might be suitable for achieving this task. | In the protein-protein interaction network housed at /data/proteins/, each node is a protein and each edge signifies an interaction between proteins. My focus is on classifying these proteins based on their functions, utilizing the node attributes that detail each protein’s type and biological properties. I am considering using the Graph Convolutional Network (GCNConv) model, as it might be well-suited for analyzing the complex relationships and properties encapsulated in this dataset. |
| 10 | For a chemical reaction network located at /data/chemical_reactions/, nodes are reactants/products and edges are reaction pathways. The goal is to predict reaction outcomes. | In the chemical reaction network stored at /data/chemical_reactions/, nodes represent reactants and products, while edges denote reaction pathways. The objective is to predict reaction outcomes. | In the chemical reaction network stored at /data/chemical_reactions/, each node represents either a reactant or a product, and the edges delineate the pathways of chemical reactions between these nodes. The objective of this study is to predict the outcomes of these reactions by analyzing how reactants transform into products along the defined pathways. This analysis aims to enhance our understanding of chemical reaction dynamics and potentially predict new reaction outcomes based on existing data. |

![Refer to caption](img/4a254145a63febbacb5342b984ceb4af.png)

Figure 6\. Case Study on Cora Dataset.

![Refer to caption](img/e9b9ce371fc02ec10774ce945fcc287b.png)

Figure 7\. Case Study on NCI1 Dataset

Table 25\. Interpretation of Agents Decisions in Diverse Graph Learning Scenarios

| Agents | Input information | Reason of decision | Agent output |
| Manager agent | User Request : I have a graph, it is saved on the path:Cora, in which node represent the paper and edges represent the citation relationship. The node attribute is the keywords mentioned in the paper.I want to predict the domain of the given paper. | {’Learning_tasks_on_graph_reason’: ’The user wants to predict the domain of the given paper. The node attribute is the keywords mentioned in the paper. Therefore, the node-level learning task is chosen.’, ’Learning_task_types_reason’: ’The user wants to predict the domain of the given paper. Therefore, the classification learning task is chosen.’, ’Evaluation_metric_reason’: ’The user wants to predict the domain of the given paper. Therefore, the accuracy metric is chosen.’} | {’Data’: ’Cora’, ’Learning_tasks_on_graph’: ’node-level’, ’Learning_task_types’: ’classification’, ’Evaluation_metric’: ’accuracy’, ’Preference’: ’None’} |
| Manager agent | User Request : In an environmental monitoring network at /data/environment_network/, nodes are sensor locations and edges represent ecological connections. The goal is to predict the overall health of an ecosystem. Graph-level attributes include biodiversity indices and pollution levels. | {’Learning_tasks_on_graph_reason’: ’The goal is to predict the overall health of an ecosystem, which is a graph-level attribute. Therefore, a graph-level learning task is chosen.’, ’Learning_task_types_reason’: ’The goal is to predict a continuous variable (overall health), which is a regression task.’, ’Evaluation_metric_reason’: ’R-squared is a common metric for regression tasks, and it is preferred because it provides a measure of how well the model fits the data.’} | {’Data’: ’/data/environment_network/’, ’Learning_tasks_on_graph’: ’graph-level’, ’Learning_task_types’: ’regression’, ’Evaluation_metric’: ’R-squared’, ’Preference’: ’None’} |
| Manager agent | User Request : In a book co-authorship network stored at /data/book_network/, nodes are authors and edges are co-authorships. The goal is to identify influential authors in different genres. I’m leaning towards using GCNConv. | {’Learning_tasks_on_graph_reason’: ’The goal is to identify influential authors in different genres. ’, ’Learning_task_types_reason’: ’The user wants to identify influential authors in different genres. Classification is used.’, ’Evaluation_metric_reason’: ’Accuracy is used as the evaluation metric because the user wants to identify influential authors in different genres.’} | {’Data’: ’/data/book_network/’, ’Learning_tasks_on_graph’: ’node-level’, ’Learning_task_types’: ’classification’, ’Evaluation_metric’: ’accuracy’, ’Preference’: ’ GCNConv ’} |
| Data agent | User Request : I have a graph, it is saved on the dataset file:Cora, in which node represent the paper and edges represent the citation relationship.The node attribute is the keywords mentioned in the paper.I want to predict the domain of the given paper. Task plan : {’Data’: ’Cora’, ’Learning_tasks_on_graph’: ’node-level’, ’Learning_task_types’: ’classification’, ’Evaluation_metric’: ’accuracy’, ’Preference’: ’None’} | {"feature_engineering_reason": "The AddSelfLoops transform is used to add self-loops to the given homogeneous or heterogeneous graph. This is useful for cases where the graph does not have self-loops, but the model requires them to be present."} | {"feature_engineering": "AddSelfLoops"} |
| Data agent | User Request : In an online gaming network at /data/gaming_network/, nodes are players and edges are interactions. The goal is to identify user engagement patterns. Node attributes include playtime and activity data. |  |  |
| Task plan : {’Data’: ’/data/gaming_network/’, ’Learning_tasks_on_graph’: ’Node-level’, ’Learning_task_types’: ’Classification’, ’Evaluation_metric’: ’Accuracy’, ’Preference’: ’None’} | {"feature_engineering_reason": "The gaming network is undirected, hence the ToUndirected feature engineering technique is suitable."} | {feature_engineering": "ToUndirected"} |  |
| Configuration agent | Task plan : {’Data’: ’Cora’, ’Learning_tasks_on_graph’: ’Node-level’, ’Learning_task_types’: ’Classification’, ’Evaluation_metric’: ’Accuracy’, ’Preference’: ’None’} | {’reason’: ’The task is related to graph representation learning, and the hidden size cannot adopt the differentiable search algorithm. The design dimension does not contain any coarsening operations. Therefore, the differentiable search algorithm is suitable for this task.’} | {’algorithm’: ’Differentiable Search’} |

Table 26\. The comparisons of agent results based on different LLMs.

|  |  | LLaMA2-7B | LLaMA2 70B | GPT-3.5-turbo + LangChain (ours) | GPT-4 |
| Configuration | Module Selection | Possible operations: Aggregation: $\cdots$, Pooling: $\cdots$, Readout: $\cdots$ | Here are my suggestions for the three operations provided, along with their justifications, for the graph-level task: Convolution: $\cdots$, Pooling: $\cdots$, Readout: $\cdots$ | The response list is [‘aggregation’, ‘pooling’, ‘readout’,‘selection’,‘fusion’] | The graph-level task involves learning a representation for the entire graph, which is often used for graph classification or regression. Convolution: Yes. $\cdots$, Pooling: Yes. $\cdots$, Readout: Yes. $\cdots$ .So, all the operations [convolution, pooling, readout] can be used for graph-level tasks. |
| Operation Preparation & Candidate Selection | To address your request: 1\. Justification: $\cdots$, 2\. Finding the corresponding class: $\cdots$, 3\. Returning the class name and module name $\cdots$. | The class name for the convolution operation would be ChebConv. The module name would be pyg.nn. | ’ChebConv’, ’torch_geometric.nn.conv.cheb _conv.ChebConv’ | This function is part of the convolution module in PyTorch Geometric. So, the class name is ChebConv and the module name is convolution. |
| Formatted output | { ’aggregation’: [’GCNConv’, ’GATConv’, ’ChebConv’], "READOUT":$\cdots$ } | { ’aggregation’: [’GCNConv’, ’GATConv’, ’ChebConv’], "READOUT":$\cdots$ } | {‘aggregation’: [ GCN, SAGE, ChebConv ],‘readout’: $\cdots$} | { ’aggregation’: [’GCNConv’, ’GATConv’, ’ChebConv’], "READOUT":$\cdots$ } |
| Algorithm | Identifying Requirements & Algorithm Selection | Firstly, let me clarify the difference between them. $\cdots$. Then, Lets evaluate the options based on these principles. $\cdots$ | Recommendation: Differentiable Search Algorithm | You should use“differentiable search algorithm". | $\cdots$ So, in this case, both differentiable and random search algorithms should be used depending on the specific operation. |