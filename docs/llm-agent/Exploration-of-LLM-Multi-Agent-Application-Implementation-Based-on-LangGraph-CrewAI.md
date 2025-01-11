<!--yml
category: 未分类
date: 2025-01-11 11:53:06
-->

# Exploration of LLM Multi-Agent Application Implementation Based on LangGraph+CrewAI

> 来源：[https://arxiv.org/html/2411.18241/](https://arxiv.org/html/2411.18241/)

Zhihua Duan Intelligent Cloud Network Monitoring Department
China Telecom Shanghai Company Shanghai, China
duanzh.sh@chinatelecom.cn    Jialin Wang Executive Vice President
Ferret Relationship Intelligence Burlingame, CA 94010, USA
jialinwangspace@gmail.com

###### Abstract

With the rapid development of large model technology, the application of agent technology in various fields is becoming increasingly widespread, profoundly changing people’s work and lifestyles. In complex and dynamic systems, multi-agents achieve complex tasks that are difficult for a single agent to complete through division of labor and collaboration among agents. This paper discusses the integrated application of LangGraph and CrewAI. LangGraph improves the efficiency of information transmission through graph architecture, while CrewAI enhances team collaboration capabilities and system performance through intelligent task allocation and resource management. The main research contents of this paper are: (1) designing the architecture of agents based on LangGraph for precise control; (2) enhancing the capabilities of agents based on CrewAI to complete a variety of tasks. This study aims to delve into the application of LangGraph and CrewAI in multi-agent systems, providing new perspectives for the future development of agent technology, and promoting technological progress and application innovation in the field of large model intelligent agents.

###### Index Terms:

LangGraph, CrewAI, Agent, Large Language Model, Generative AI

## I Introduction

Driven by the rapid advancements in artificial intelligence technology, the application of agents in various fields is increasingly growing, profoundly affecting everyone’s work and lifestyle. The application scope of agent technology is broad; it can autonomously perceive the environment, conduct data analysis, and make decisions, thereby significantly enhancing efficiency and optimizing resource allocation. In complex and dynamic systems, the introduction of multi-agent systems enables multiple agents to collaborate and complete complex tasks that are difficult for a single agent to achieve.

The key advantage of multi-agent systems lies in their task decomposition capabilities, achieving goals through the collaborative action of agents, which not only enhances the system’s flexibility and adaptability but also improves its generalization ability. In environments characterized by uncertainty and dynamic changes, the ability of agents to collaborate and divide tasks is particularly crucial.

This paper primarily investigates two issues: (1) designing the architecture of agents based on LangGraph for more precise control; (2) enhancing the capabilities of agents based on CrewAI to accomplish different tasks. The aim is to delve into the application of AI multi-agent systems, explore the technological advantages brought about by the combination of LangGraph and CrewAI, and their potential application value in various fields. Through the analysis and research of these technologies, it is expected to provide new perspectives and ideas for the future development of agent technology, thereby promoting technological progress and application innovation in the field of large model intelligent agents.

## II Related Work

Autonomous agents are typically responsible for specific roles to accomplish various tasks.MetaGPT[[1](https://arxiv.org/html/2411.18241v1#bib.bib1)], ChatDev[[2](https://arxiv.org/html/2411.18241v1#bib.bib2)], and self-collaboration[[3](https://arxiv.org/html/2411.18241v1#bib.bib3)] pre-set various roles and corresponding responsibilities to foster collaboration among agents.

Autonomous agents based on Large Language Models (LLMs) incorporate mechanisms from human memory processes.RLP[[4](https://arxiv.org/html/2411.18241v1#bib.bib4)] is a conversational agent that maintains an internal state for both parties in a dialogue, achieving the agent’s short-term memory.SayPlan[[5](https://arxiv.org/html/2411.18241v1#bib.bib5)] is an agent designed for task planning and design.When faced with complex tasks, break them down into simpler subtasks and solve them. The Chain of Thought (CoT) [[6](https://arxiv.org/html/2411.18241v1#bib.bib6)] inputs reasoning steps for solving complex problems into the prompt.[[7](https://arxiv.org/html/2411.18241v1#bib.bib7)] proposes an intelligent personalized digital banking assistant based on LangGraph and Chain of Thoughts (COT), leveraging Large Language Models (LLM) and a multi-agent framework to enhance task efficiency. [[8](https://arxiv.org/html/2411.18241v1#bib.bib8)] improves advanced question-answering systems based on Retrieval-Augmented Generation (RAG) by leveraging graph technology, to overcome the limitations of existing RAG models and develop high-quality artificial intelligence services.

In this study, autonomous agents based on LLM can leverage the framework capabilities of LangGraph and CrewAI to automatically perform various tasks, endowing Agents with the ability to complete specific tasks, forming a comprehensive application system framework.

## III Methodology

### III-A Introduction to LangGraph

LangGraph is a framework designed for constructing multi-agent applications, enabling developers to utilize large language models (LLMs) to create agents and multi-agent workflows. Compared to other LLM frameworks, LangGraph offers the benefits of loops, controllability, and persistent memory. LangGraph provides granular control over the application’s processes and states, enabling the creation of reliable agents that support advanced human-computer interaction and memory capabilities. The flexible framework of LangGraph supports various control flows—single-agent, multi-agent, hierarchical, sequential and can robustly manage complex real-world scenarios.

### III-B CrewAI Framework

CrewAI is an open-source framework designed to coordinate AI agents with role-playing and autonomous operations to facilitate cooperation among agents in solving complex problems. It allows developers to define AI agents with specific roles, objectives, and tools. The main building blocks of CrewAI include Agent, Task, Tool, and Crew, offering a rich set of features that can be freely selected and combined according to specific needs to create multi-agent systems. CrewAI supports various APIs such as OpenAI and Ollama, and it has key features like role-customized agents, automatic task delegation, and flexibility in task management. CrewAI is aimed at handling complex tasks, such as multi-step workflows, decision-making, and problem-solving.

An Agent intelligent entity is an autonomous unit capable of performing tasks, making decisions, and communicating with other agents. In the CrewAI framework, a Task refers to the specific work carried out by an Agent, including details such as description, executing agents, and required tools. It supports multi-agent collaboration and optimizes team cooperation and efficiency through the process orchestration of the Crew.

![Refer to caption](img/4a4c914d52dca62537fa42e7e8d1973a.png)

Figure 1: Illustration of the email case based on LangGraph +CrewAI.

### III-C Integrating LangGraph with CrewAI

The integration of LangGraph+CrewAI framework provides powerful tools for complex task management, optimizing task execution and multi-agent systems through flexible workflows, inter-agent collaboration, and graph-structured management. It supports customized development by integrating with existing tools, meeting the evolving needs of AI applications. CrewAI ensures process efficiency through clear task allocation and role definition. Additionally, seamless integration with LangChain allows developers familiar with the framework to easily integrate independent agents, further enhancing the framework’s appeal.

Taking the example of the automatic email composition and sending case provided on the CrewAI official website, the LangGraph+CrewAI framework breaks down complex tasks into manageable steps and automates their execution.

As shown in Figure 1, with LangGraph, it is possible to clearly define and visualize the workflow for writing and sending emails, including checking new emails, composing new emails, and waiting for the next run.

As shown in Figures 2 to 4 ,this is a code example based on CrewAI+LangGraph. By integrating CrewAI with LangGraph, it becomes easy to implement functionalities such as email checking, composing, and automatic sending based on large models.

![Refer to caption](img/aa00a28531a8e46708c19a72dafa5e84.png)

Figure 2: Code example based on LangGraph + CrewAI.

![Refer to caption](img/8415365d620ebe8196c101713ea8fa8b.png)

Figure 3: Agent example based on CrewAI.

![Refer to caption](img/a82b9db352cfe27d6956f3dbef8fa99c.png)

Figure 4: Task example based on CrewAI.

## IV Application Case Practice

### IV-A Development Environment

In this study, we utilized the following development environment components to apply the LangGraph+CrewAI framework:

*   •

    Multi-agent: CrewAI, for organizing and coordinating collaborative work among AI agents.

*   •

    Graph workflow: LangGraph, used for managing state sharing and process control of graph nodes.

*   •

    Workflow tracking: LangSmith, for monitoring and auditing the execution of workflows.

*   •

    Embedding technology: ollama embedding model, utilizing embedded vector models to retrieve work order text.

*   •

    Vector database: Fasiss is used for storing and retrieving vector data, accelerating data access speed.

### IV-B Application Cases

As shown in Figure 5, based on the LangGraph+CrewAI framework, we have explored and implemented a multi-agent collaborative application that integrates code generation and code review capabilities. By achieving real-time sharing of status data and feedback mechanisms, the efficiency of code generation has been improved.

![Refer to caption](img/7b550122121eab779d48d1294180ebfd.png)

Figure 5: A code generation case based on LangGraph+CrewAI.

As shown in Figure 6, based on the CrewAI+LangGraph framework, we have explored and implemented a case for ticket auditing and forwarding features. By conducting an in-depth analysis of ticket text information, we have built multiple intelligent agents that can more accurately understand the content of the tickets, thereby enhancing the efficiency of ticket processing.

![Refer to caption](img/7d25b739dee93517a38a5d6bbb7c79cc.png)

Figure 6: Case study of automatic processing of work orders based on LangGraph+CrewAI

## V Conclusion

This article explores the combined application of LangGraph and CrewAI frameworks, demonstrating the powerful capabilities of these two frameworks in building complex multi-agent systems and workflow management. Through case analysis, the LangGraph+CrewAI framework has advantages in task management, inter-agent collaboration, graph workflow implementation, and integration with existing tools. This integrated approach not only improves the efficiency of task execution but also enhances the system’s flexibility and scalability through real-time status data sharing and feedback mechanisms. Our research results indicate that the combination of LangGraph and CrewAI provides a powerful toolkit for developing advanced AI applications, especially in scenarios that require handling complex tasks and multi-agent collaboration. By integrating with LangChain, existing independent agents can be easily incorporated into the CrewAI framework, providing developers with a unified platform for building and managing complex AI workflows.

## References

*   [1] Mingchen Zhuge Sirui Hong et al. Metagpt: Meta programming for multi-agent collaborative framework. International Conference on Learning Representations, 2024.
*   [2] Wei Liu Chen Qian et al. Chatdev: Communicative agents for software development. In Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 15174–15186.
*   [3] Xue Jiang Yihong Dong et al. Self-collaboration code generation via chatgpt. CoRR, abs/2304.07590, 2023.
*   [4] Kevin A. Fischer. Reflective linguistic programming (rlp): A stepping stone in socially-aware agi (socialagi). CoRR, abs/2305.12647, 2023.
*   [5] Jesse Haviland Krishan Rana et al. Sayplan: Grounding large language models using 3d scene graphs for scalable task planning. CoRR, 2023.
*   [6] Xuezhi Wang Jason Wei et al. Chain-of-thought prompting elicits reasoning in large language models. Advances in neural information processing systems, 35:24824–24837, 2022.
*   [7] Saha Sourav Arafat Md Easin et al. An intelligent llm-powered personalized assistant for digital banking using langgraph and chain of thoughts. IEEE 22nd Jubilee International Symposium on Intelligent Systems Informatics, pages 625–630, 2024.
*   [8] Cheonsu Jeong. A study on the implementation method of an agent-based advanced rag system using graph. CoRR, abs/2407.19994, 2024.