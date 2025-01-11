<!--yml
category: 未分类
date: 2025-01-11 11:58:19
-->

# AgentOps: Enabling Observability of LLM Agents

> 来源：[https://arxiv.org/html/2411.05285/](https://arxiv.org/html/2411.05285/)

\useunderLiming Dong, Qinghua Lu, Liming Zhu
Data61, CSIRO, Australia(Nov, 2024)

###### Abstract

Large language model (LLM) agents have demonstrated remarkable capabilities across various domains, gaining extensive attention from academia and industry. However, these agents raise significant concerns on AI safety due to their autonomous and non-deterministic behavior, as well as continuous evolving nature . From a DevOps perspective, enabling observability in agents is necessary to ensuring AI safety, as stakeholders can gain insights into the agents’ inner workings, allowing them to proactively understand the agents, detect anomalies, and prevent potential failures. Therefore, in this paper, we present a comprehensive taxonomy of AgentOps, identifying the artifacts and associated data that should be traced throughout the entire lifecycle of agents to achieve effective observability. The taxonomy is developed based on a systematic mapping study of existing AgentOps tools. Our taxonomy serves as a reference template for developers to design and implement AgentOps infrastructure that supports monitoring, logging, and analytics. thereby ensuring AI safety.

## 1 Introduction

An large language model (LLM) is a large-scale language model with tens of billions of parameters, pretrained on vast and diverse datasets, and applicable to various downstream tasks [[1](https://arxiv.org/html/2411.05285v2#bib.bib1)]. While LLMs demonstrate impressive capabilities, they also exhibit limitations in understanding and performing complex tasks. This has led to an increasing demand for LLM agents (including agentic systems which are designed to prompt an LLM multiple times using agent-like design patterns and exhibit varying degrees of agent-like behavior), which have the ability to handle complex tasks autonomously [[2](https://arxiv.org/html/2411.05285v2#bib.bib2)].

An LLM agent is an autonomous system powered by LLMs, capable of perceiving context, reasoning, planning, executing workflows, by leveraging external tools, knowledge bases and other agents to achieve human goals [[3](https://arxiv.org/html/2411.05285v2#bib.bib3)]. LLM agents have shown remarkable potential to enhance productivity across various domains, attracting widespread attention from academia and industry. For example, many agents are being successfully applied in the software engineering domain, such as Devin ¹¹1Devin, [https://www.cognition.ai/blog/introducing-devin](https://www.cognition.ai/blog/introducing-devin), ChatDev ²²2ChatDev. [https://github.com/OpenBMB/ChatDev](https://github.com/OpenBMB/ChatDev), SWE-agent ³³3SWE-agent, [https://github.com/princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent). Hereafter, we use ”agents” to refer specifically to LLM agents throughout this paper.

Despite the huge potential to enhance productivity, the adoption of LLM agents introduces unique challenges due to their inherent characteristics.

*   •

    Complex Artefacts and Pipelines: The agents are compound AI systems, integrating LLMs with various components (i.e., design time artifacts), such as context engine and external tools, and dynamically generating runtime artifacts, such as goals and plans. The operational pipelines typically include context processing, reasoning and planning, workflow execution, and continuous evolution based on the feedback. Throughout these processes, the pipelines may leverage external tools, knowledge bases, and other agents to achieve human goals.

*   •

    Autonomy: The agents operate with a high degree of autonomy, dynamically interact with external environments, including shifting context, external knowledge bases and tools. These interactions are not predetermined, which may increase the risk of unintended behaviors (such as selecting an external tool with vulnerability issues) and pose severe AI safety challenges [[4](https://arxiv.org/html/2411.05285v2#bib.bib4)].

*   •

    Non-Deterministic Behaviour: Due to the probabilistic nature of LLMs, the agents often exhibit non-deterministic behaviour, producing varied outputs even when given the same inputs. This lack of repeatability may lead to unintended consequences, making it challenging to ensure consistent and predictable outcomes.

*   •

    Continuous Evolution: The agents can evolve over time through continuous learning, adapting based on runtime evaluation results or human feedback. While this adaptability enhance the agents’ capabilities and skills, it also introduces further challenges in maintaining alignment with intended quality and safety goals over the agent’s lifecycle.

*   •

    Shared Accountability: The responsibility of the behaviour or decisions of the agents is often shared among multiple stakeholders, including the agent owner, the FM provider, and various providers of external tools/agents. This complicates the identification of failure sources and the assignment of accountability in the event of incidents.

To address the challenges outlined above, it is essential for DevOps tools to support observability features, enabling stakeholders to monitor agent behaviour, track the status of artifacts, log associated data, detect anomalies, trace the evolution of artifacts, and assign accountability if incidents occur. Observability refers to the ability to gain actionable insights into the inner workings of an agent by analysing the inputs and outputs (i.e., runtime artifacts) of different components (i.e., design time artifacts), as they flow through the operational pipelines [[5](https://arxiv.org/html/2411.05285v2#bib.bib5)]. However, most existing DevOps tools for agents primarily focus on LLM-specific metrics and prompt management, with limited support for the observability of agent-specific artifacts such as goals, plans, and tools. This limitation results in insufficient observability from both the system and pipeline perspectives.

To bridge this gap, we propose AgentOps, a specialised DevOps paradigm tailored for agents. AgentOps provides a holistic view of agent operations, enabling comprehensive observability by systematically tracing of agent artifacts and associated data. In this paper, we perform a systematic mapping study on the existing DevOps tools for monitoring agents and/or agentic systems to understand their features and limitations. Based on the study results, we first propose an artifact relationship model to identify key agent artifacts and their relationship. Then we present a comprehensive taxonomy of AgentOps, detailing the artifacts and associated data that should be traced. Our taxonomy serves as a reference template for developers to design and implement AgentOps tools to support monitoring, logging, and analytics of agents.

The remainder of the paper is organized as follows. Section [2](https://arxiv.org/html/2411.05285v2#S2 "2 Methodology ‣ AgentOps: Enabling Observability of LLM Agents") introduces the methodology. Section [3](https://arxiv.org/html/2411.05285v2#S3 "3 Mapping Study Result ‣ AgentOps: Enabling Observability of LLM Agents") presents the mapping study results of AgentOps tools and core features. Section [4](https://arxiv.org/html/2411.05285v2#S4 "4 Taxonomy of AgentOps ‣ AgentOps: Enabling Observability of LLM Agents") identifies the agent artifacts and their relationship and presents the taxonomy of AgentOps. Section [5](https://arxiv.org/html/2411.05285v2#S5 "5 Threats to Validity ‣ AgentOps: Enabling Observability of LLM Agents") addresses threats to validity, and Section [6](https://arxiv.org/html/2411.05285v2#S6 "6 Conclusion ‣ AgentOps: Enabling Observability of LLM Agents") concludes the paper and outlines future work.

## 2 Methodology

In this section, we introduce the methodology for this study. Figure[1](https://arxiv.org/html/2411.05285v2#S2.F1 "Figure 1 ‣ 2 Methodology ‣ AgentOps: Enabling Observability of LLM Agents") provides the overview of the search process. Following the guidelines outlined in [[6](https://arxiv.org/html/2411.05285v2#bib.bib6), [7](https://arxiv.org/html/2411.05285v2#bib.bib7)], we performed a systematic mapping study to analyse the existing tools related to AgentOps. This study aimed to understand their features and limitations and to identify the agent artifacts and associated data that should be traced to enable observability.

![Refer to caption](img/af0a7697061540e68b8b157da21275d0.png)

Figure 1: Search Process of AgentOps Relevant Tools

### 2.1 Data Sources

To identify tools relevant to AgentOps, we used multiple sources. Initially, we conducted a comprehensive search on GitHub⁴⁴4Github.[https://github.com](https://github.com) using carefully selected keywords. The repositories were then filtered based on predefined selection criteria to ensure relevance and quality. To complement the GitHub search and address potential gaps, we performed a targeted Google Search⁵⁵5Google Search.[https://google.com](https://google.com) to capture additional tools that may not have been available or visible in the GitHub search results. This multi-source approach ensured broad coverage of both open-source and proprietary AgentOps relevant tools.

### 2.2 Search String

The search terms we used in GitHub primarily focused on the keywords *((“AgentOps”) OR (“Agent” AND “DevOps”) OR (“Agent” AND “LLMOps”))*. Since the concept of AgentOps and the exploration of related tools are still in the early stages, we extended the scope of our search to include observability tools that provide comprehensive visibility into LLM applications, must including agents tracing and observability features as well. As a result, we also incorporated the search term *(“Agent” AND “Observability”)*. The search keyword term used in Google Search is *(“LLM” AND “Agent” AND “Observability” AND “Tool”)*.

Table 1: Selection Criteria

| ID | Inclusion Criteria |
| --- | --- |
| $I1$ | Support observability features (such as monitoring and tracing). |
| $I2$ | Support agent specific tracing or LLM application tracing that can be applied to agents, not just LLM-level tracing. |
| $I3$ | Formal release version. |
| $I4$ | Public online document available. |

### 2.3 Selection Criteria

We conducted the tool selection process from the multi-channel data sources mentioned above. For this paper, we focused on AgentOps relevant tools that met the criteria illustrated in Table 1. To ensure robust tracking and management of agent performance and interactions, it is essential that selected tools support core observability functionalities ($I1$) like monitoring and tracing etc. Furthermore, given that the focus of our study is on AgentOps rather than general LLMOps or model-level operations, the tools must support tracing at the agent level or LLM application level ($I2$) that allows us to observe a finer-grained understanding of agent-specific behaviors. To facilitate both implementation and community-wide adoption, it is essential that tools with formal release version ($I3$) and have accessible and public online documentation ($I4$).

### 2.4 Search Process

We began by querying GitHub with four specific keyword strings *“AgentOps”*, *“Agent” AND “DevOps”*, *“Agent” AND “LLMOps”*, and *“Agent” AND “Observability”*. Each query returned an initial set of repositories. For the search strings *(“AgentOps”) OR (“Agent” AND “LLMOps”) OR (“Agent” AND “Observability”)*, we obtained 37, 24, and 81 initial repositories, respectively. All repositories from these search results were screened to ensure relevance, leading to the inclusion of 4, 3, and 3 tools for further analysis, respectively.

For the *(“Agent” AND “DevOps”)* query, the search yielded a much larger set of 618 repositories. To manage this, we limited our screening to the top three pages of results, sorting repositories based on relevance and popularity. Most of these projects, however, were focused on agents for DevOps tasks rather than DevOps platforms for managing agents. As a result, only one additional tool was included from this query.

To ensure comprehensive coverage and address potential gaps in the GitHub search, we conducted an additional search using Google Search to capture proprietary tools. We used the keyword string *(“LLM” AND “Agent” AND “Observability” AND “Tool”)* and screened the top three pages of results. This search targeted relevant blogs, product announcements, and other online resources. After removing duplicates and irrelevant results, this process identified six additional tools.

The combined search process across GitHub and Google Search resulted in a total of 17 tools. This final set formed the basis for our systematic analysis of AgentOps relevant tools’ features and limitations.

### 2.5 Data Extraction

To better understand the current landscape of AgentOps tools, we systematically categorized and summarized 17 selected tools. Relevant data items were extracted from each tool’s GitHub repository, official product website, and available project documentation. The specific data items extracted are outlined in Table [2](https://arxiv.org/html/2411.05285v2#S2.T2 "Table 2 ‣ 2.5 Data Extraction ‣ 2 Methodology ‣ AgentOps: Enabling Observability of LLM Agents").

Table 2: Data Extraction

| ID | Data Item | Description |
| --- | --- | --- |
| $D1$ | Name | The name of AgentOps relevant tool. |
| $D2$ | Source | The data source used to identify the included tool. |
| $D3$ | GitHub Repo URL | The Github repository URL of included tool. |
| $D4$ | Star | The number of stars received on Github (as a proxy for popularity). |
| $D5$ | Scope | The scope of the tool, specifically whether it is designed for tracing agents or LLM applications. |
| $D6$ | Key Features | The primary features provided by the tool for AgentOps or LLM application observability. |
| $D7$ | Traceable Artifacts | The artifacts and associated data that the tool tracks. |

## 3 Mapping Study Result

In this section, we present the results of the systematic mapping study on tools relevant to AgentOps.

Table 3: List of Tools relevant to AgentOps

| Name | Source | Github Repo URL | Star | Scope |
| Agenta | Github | Agenta-AI/agenta | 1.3k | LLM applications |
| AgentNeo | Github | raga-ai-hubAgentNeo | 1k | Agents |
| AgentOps | Github | AgentOps-AI/agentops | 2.1k | Agents |
| AGIFlow | Github | AgiFlow/agiflow-sdks | 21 | Agents |
| Arize | Google | Arize-Phoenix | 3.9k | LLM applications |
| DataDog | Github | DataDog/datadog-agent | 2.9k | Agents |
| Dify | Github | langgenius/dify | 51.6k | LLM applications |
| Helicone | Github | Helicone/helicone | 1.9k | LLM applications |
| Laminar | Github | lmnr-ai/lmnr | 1.1k | LLM applications |
| Langfuse | Github | langfuse/langfuse | 6.5k | LLM applications |
| LangSmith | Github | langchain-ai/langsmith-sdk | 417 | LLM applications |
| LangTrace | Github | Scale3-Labs/langtrace | 552 | LLM applications |
| Lunary | Google | lunary-ai/lunary | 1.1k | LLM applications |
| PortKey | Google | Portkey-AI/gateway | 6.3k | LLM applications |
| TraceLoop | Google | traceloop/openllmetry | 3.4k | LLM applications |
| Trulens | Google | truera/trulens | 2.2k | LLM applications |

### 3.1 AgentOps-Relevant Tools

This study includes a diverse range of 17 tools relevant to AgentOps, designed to enable observability in agents or LLM applications. Table [3](https://arxiv.org/html/2411.05285v2#S3.T3 "Table 3 ‣ 3 Mapping Study Result ‣ AgentOps: Enabling Observability of LLM Agents") highlights the AgentOps relevant tools currently available in the market. An analysis of GitHub star rankings reveal that many of these tools have received significant attention, with 14 out of 17 tools accumulating thousands of stars. As of November 2024, several tools stand out as leaders in their respective categories, such as AgentOps (1.7k stars) and AgentNeo (1k stars). Among tools designed for LLM applications, Langfuse (6.5k stars), PortKey (6.3k stars), and Arize (3.9k stars), TraceLoop (3.4k stars), and DataDog (2.9k stars) stand out as leading solutions. It is worth noting that some tools may have been unintentionally excluded due to keyword mismatches or incomplete documentation. We aim to address this limitation by monitoring emerging tools and incorporating them into future work.

Table 4: Key Features of AgentOps relevant Tools

| Tool | Customisation | Prompt Mgmt | Evaluation | Feedback | Monitoring | Tracing | Guardrails |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Agenta | Yes | Yes | Yes | Yes | Yes | Yes | No |
| AgentNeo | Yes | No | Yes | No | Yes | Yes | No |
| AgentOps | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| AGIFlow | Yes | Yes | Yes | No | Yes | Yes | No |
| Arize | No | No | Yes | No | Yes | Yes | Yes |
| Datadog | No | No | No | No | Yes | Yes | No |
| Dify | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Helicone | No | No | Yes | No | Yes | Yes | No |
| Langfuse | No | Yes | Yes | Yes | Yes | Yes | No |
| LangTrace | No | No | Yes | No | Yes | Yes | No |
| LangSmith | No | Yes | Yes | Yes | Yes | Yes | Yes |
| Lunary | No | Yes | Yes | Yes | Yes | Yes | Yes |
| TraceLoop | No | No | No | No | Yes | Yes | No |
| Trulens | No | Yes | Yes | Yes | Yes | Yes | Yes |
| Portkey | No | No | Yes | No | Yes | Yes | No |

### 3.2 Key Features

We summarized the key features of the identified AgentOps relevant tool, as shown in Table [4](https://arxiv.org/html/2411.05285v2#S3.T4 "Table 4 ‣ 3.1 AgentOps-Relevant Tools ‣ 3 Mapping Study Result ‣ AgentOps: Enabling Observability of LLM Agents") and Table [5](https://arxiv.org/html/2411.05285v2#S3.T5 "Table 5 ‣ 3.2 Key Features ‣ 3 Mapping Study Result ‣ AgentOps: Enabling Observability of LLM Agents").

Table 5: AgentOps Relevant Tools Key Features

| Category | Features | Description |
| Customization | Provision, custom, spawn, and deploy autonomous agents | Create customisable and scalable autonomous agents. |
|  | Extend agent capabilities with toolkits | Add toolkits from marketplace to agent workflows. |
|  | Extend agent capabilities with multiple vector databases | Connect to multiple vector databases to improve agent’s performance. |
|  | Extend agent capabilities with fine-tuned models | Custom fine-tuned models for business specific use cases. |
| Prompt Management | Prompt versioning and management | Keep track of different versions of prompts used in agents. Useful for A/B testing and optimizing agent performance. |
|  | Prompt playground with model comparisons | Test and compare different prompts and models for agents before deployment. |
|  | Prompt injection detection | Identify potential code injection and secret leaks. |
| Evaluation | Test agents against benchmarks and leaderboards. | Create a dataset, define metrics, run evaluations, compare results, track results over time etc. |
|  | Evaluate agents in diverse step | Evaluate Final Response- Evaluate the agent’s final response. |
|  |  | Evaluate Single step-Evaluate any agent step in isolation (e.g., whether it selects the appropriate tool). |
|  |  | Evaluate Trajectory- Evaluate whether the agent took the expected path (e.g., of tool calls) to arrive at the final answer. |
| Feedback | Collect explicit feedback | Directly prompt the user to give feedback, this can be a thumb up or a thumb down. |
|  | Collect implicit feedback | Measure the user’s behavior, this can be time spent on a page, click-through rate. |
| Monitoring | Agent analytics dashboard | Monitor diverse level and dimension statistics metrics about agents. |
|  | LLM cost management and tracking | Track spend (token cost) with foundation model providers. |
| Tracing | LLM/agent tracing | Trace each agent span, e.g., the whole chain, retrieval, LLM call, tool call etc. |
|  |  | Trace evaluation span |
|  |  | Trace user feedback |
| Guardrails | Predefined rules and constraints | Set rules or constraints to limit agent actions, ensuring safe and predictable behavior. |
|  | Fallback and escalation paths | Provide safe defaults or redirect cases to human operators in ambiguous or risky scenarios. |

#### 3.2.1 Customization

When creating agents, existing tools extend agent capabilities by adding toolkits from marketplaces to agent workflows, connecting to multiple knowledge bases to improve performance, and integrating customized fine-tuned models for specific business use cases.

#### 3.2.2 Prompt Management

Prompt versioning and management allow developers to store and track different versions of prompts, making it invaluable for testing, optimizing, and reusing prompts across various stages of agent production. The prompt playground enables developers to edit, import, and test various prompt templates with different models, helping to compare performance before deployment. Additionally, consistent monitoring of prompts is also necessary for maintaining the reliability and security of agents, particularly in detecting and mitigating issues such as code injection attacks and secret leaks embedded within prompts.

#### 3.2.3 Evaluation

Evaluation is the process of assessing the behaviour and capabilities of an agent against specific criteria or general benchmarks. The typical evaluation process involves creating a suitable evaluation dataset, defining clear criteria and metrics, and conducting thorough testing based on these predefined metrics. It is important to test the agent’s performance against user requirements, standard leaderboards or comparable systems. For agents, evaluation goes beyond simply assessing the the final output. It is equally important to monitor and track the agent’s execution steps and evaluate intermediate outputs to ensure the entire process meets the intended goals and aligns with governance requirements.

LangSmith⁶⁶6LangSmith.[https://docs.smith.langchain.com/evaluation](https://docs.smith.langchain.com/evaluation) introduces two additional dimensions of agent evaluation: 1) Step-by-Step Evaluation: Assess each individual step the agent takes in isolation, such as determining whether it selects the appropriate tool. 2) Trajectory Evaluation: Examine whether the agent followed the expected sequence of actions, including the series of tool calls, to arrive at the final answer. This ensures that the decision-making process is sound, not just the outcome.

#### 3.2.4 Feedback

Human feedback plays a key role in evaluating the quality of an agent’s output. Feedback is collected as a score and attached to an execution trace or an individual LLM generation. Feedback can be also used to retrain/fine-tune LLM or improve the design of agents (e.g. stored in memory or used in prompts as positive or negative examples). Langfuse⁷⁷7Langfuse.[https://langfuse.com/docs/scores/user-feedback](https://langfuse.com/docs/scores/user-feedback) defined different types of feedback that can be collected that vary in quality, detail, and quantity: 1) Explicit Feedback: Directly prompt the user to give feedback, this can be a rating, a like, a dislike, a scale or a comment. While it is simple to implement, quality and quantity of the feedback is often low. More structured and fine-grained feedback is expected. 2) Implicit Feedback: Measure the user’s behavior, e.g., time spent on a page, click-through rate, accepting or rejecting final output. This type of feedback is more difficult to implement but is often more frequent and reliable.

#### 3.2.5 Monitoring

Developers can continuously monitor agent performance and enhance observability throughout the agent’s execution process by closely tracking its outputs. This involves keeping track of monitoring metrics (e.g., latency and cost), associating feedback with agent spans to evaluate performance, and debugging issues by diving into specific traces and spans where errors occurred. Monitoring also helps identify the root causes of unexpected results, errors, or latency issues, allowing developers to optimize performance based on real-time feedback.

#### 3.2.6 Tracing

AgentOps is designed to support developers in transitioning from prototype to production, ensuring that the work does not stop once the agent is created and initial tests are passed. Within the otool, agents execute increasingly complex tasks and iterative runs, such as chains, tool-assisted agents, and advanced prompts. By adding traces, AgentOps captures the entire process—from the moment a user sends a prompt to the final output—helping developers understand each step and identify the root causes of any issues. Execution tracing allows developers to follow the agent’s decision-making process step-by-step, providing insights into the flow of actions, decisions, and interactions. This helps identify where errors or unexpected behaviors originate, making it easier to isolate and address problems within complex agentic workflows. There is no doubt that tracing is the most direct way to enable observability in AgentOps platform. All tools listed in the Table [3](https://arxiv.org/html/2411.05285v2#S3.T3 "Table 3 ‣ 3 Mapping Study Result ‣ AgentOps: Enabling Observability of LLM Agents") have implemented tracing functionality.

#### 3.2.7 Guardrails

Guardrails for agent application are essential for ensuring AI-safety-by-design. Arize have already integrated Arize Guards⁸⁸8Arize Guards. [https://docs.arize.com/arize/llm-monitoring-and-guardrails/guardrails](https://docs.arize.com/arize/llm-monitoring-and-guardrails/guardrails). Additionally, an increasing number of tools (e.g., AgentNeo⁹⁹9AgentNeo. [https://github.com/raga-ai-hub/agentneo](https://github.com/raga-ai-hub/agentneo) ) have Guardrails implementation on their planned feature lists, aiming to enhance the safety of agents. AgentOps tools can trace the activation, execution process and outcomes of guardrails, which can be used to generate safety cases for auditing purposes.

## 4 Taxonomy of AgentOps

![Refer to caption](img/bb0daebc9a8cbea78656e6e5462edd31.png)

Figure 2: Entity-Relationship Model for Agent Artifacts

Once agent developer have deployed their agent, they need trace its execution process to monitor agent performance and evolution of agent artifacts. The AgentOps tool provides tracing functions to troubleshoot performance and correlate data throughout the product, enabling developer to find and resolve issues in agents.

In this section, we first introduce an entity-relationship model to describe various relationships between different agent artifacts. Then, we present a comprehensive taxonomy of Agentops, which serves as a template for developers to design and implement AgentOps tools for tracing agent artifacts and their associate data.

### 4.1 Entity-Relationship Model for Agent Artifacts

To clarify the relationships between traceable artifacts, Figure [2](https://arxiv.org/html/2411.05285v2#S4.F2 "Figure 2 ‣ 4 Taxonomy of AgentOps ‣ AgentOps: Enabling Observability of LLM Agents") illustrates the connections among nested spans in an agent trace.

When an agent receives a user goal, it can utilize zero or more knowledge bases to gather information or support decision-making. These knowledge bases supply essential data and contextual information for the agent span. A single reasoning span generates a plan, structured through logical processes and analysis.

An agent can generate multiple plan spans, each representing a structured plan to address specific objectives. The agent relies on these plans to organize and coordinate its tasks. Each plan span may call one or more LLM spans to leverage the LLM’s processing capabilities, aiding in the execution of planned actions. A plan is realized as a single Workflow, translating the strategic plan into practical execution.

A workflow comprises multiple tasks, with each task representing a specific action within the larger workflow. These task collectively fulfill the goals outlined in the workflow. Each task may utilise one or more tools, providing additional functionality or resources essential for task execution. Task may also call one or more LLM spans to access model-based functionalities, such as prediction or analysis, that are critical for task completion.

Evaluation spans assess either a specific agent, a plan or a single workflow, ensuring that the agent’s overall performance or the effectiveness of the workflow aligns with intended goals. The guardrail monitors all other spans in the agent’s lifecycle, enforcing constraints and ensuring compliance with predefined rules. This universal connection helps maintain the safety of the agent’s actions across all spans.

![Refer to caption](img/af6f98d2de2ea2b4d03661860f2237bb.png)

Figure 3: Taxonomy of AgentOps

### 4.2 Taxonomy of AgentOps

In AgentOps, a trace reveals the entire process from the moment a user submits a goal achieving request to the point when the final result is delivered. This includes the reasoning process, the plan generated, the workflow along with its associated tasks, the knowledge retrieved, the tools invoked, the evaluation and guardrails applied, and multiple LLM calls.

A trace consists of one or more spans. The first span represents the root span. Each root span represents a request from start to finish. The spans beneath the parent provide deeper context for what occurs during a request, detailing the steps that make up the request.

Aspan could consists of the following metadata attributes:

*   •

    Name: The label or identifier of the span, indicating the type of operation being performed.

*   •

    Start Timestamp: The exact time when the span begins, providing temporal context for tracking and performance analysis.

*   •

    Duration: The total time taken for the span’s operation, measured from start to finish. This metric helps in analyzing efficiency and identifying potential performance bottlenecks.

*   •

    Attributes:

    *   –

        Inputs and Outputs: Data fed into the span (e.g., user goal) and the resulting outputs (e.g., tool calling result or final result).

    *   –

        Error Type, Message, and Traceback: Information about any errors encountered, including the error type, a descriptive message, and traceback details for debugging.

    *   –

        Metrics: Quantitative data related to the span, such as input_tokens, output_tokens, evaluation metrics, monitoring metrics, which measure and track cost usage and agent performance.

*   •

    Events: Specific occurrences or a detailed timeline of actions and transitions within each span, providing detailed insights into the span’s activities or any notable events during its lifecycle.

*   •

    Parent ID: An identifier that links this span to its parent span, establishing a hierarchical relationship between spans and helping to track nested operations.

*   •

    Links: Connections to other spans or external references, which can help in understanding dependencies and relationships between different spans in a complex workflow.

In the context of agent spans, where a trace consists of a series of nested spans representing different operational spans or runs, each type of span involved in agent span plays a distinct role. The agent span includes the following nested spans:

*   •

    Agent Span: The agent span metadata includes the agent’s role and persona, which significantly influence how the agent performs tasks, interacts with users and makes decisions. The agent span meta data includes the following.

    *   –

        Agent Role: The scope or responsibility of the agent.

    *   –

        Agent Persona: The behavioral characteristics and interaction style the agent adopts.

*   •

    Reasoning Span: Reasoning span, captures the reasoning processes of the agent. Reasoning Span metadata including:

    *   –

        Context: The relevant information or situational data that informs the reasoning process. This may include inputs from previous spans or external factors that influence the agent’s reasoning.

    *   –

        Retrieved Knowledge: Information or data that the agent retrieves and references during reasoning, possibly from external sources or memory, to support its reasoning.

    *   –

        Inference Rules and Boundary: The logical rules, constraints, or boundaries applied during the reasoning. This could include specific guidelines or conditions under which the agent performs reasoning.

    *   –

        Outcome: The thoughts generated or conclusion reached after the reasoning process.

*   •

    Planning Span: Planning span records the planning phase, where the agent outlines the steps or objectives required to achieve the goal. It defines the intended sequence of operations, setting up a structured path for the agent’s activities. Planning Span metadata including:

    *   –

        Goal: The specific objective or desired outcome that the agent intends to achieve through this plan.

    *   –

        Constraints: The limitations or restrictions within which the plan must operate. These constraints can include time limits, resource limitations, or predefined rules that shape the planning process.

    *   –

        Context: Relevant situational or environmental information that informs the plan.

    *   –

        Historical Plans: Records of previous plans that may influence the current planning process. This includes past strategies or actions taken in similar contexts, which can provide valuable insights or best practices for the current plan.

*   •

    Workflow Span: Within the workflow span, the trace documents the organization of tasks through task spans and tool Spans. The workflow span, supported by an LLMspan, details how individual tasks are broken down and managed. The task span represents specific actions within the workflow, while the tool span (nested within the task span) logs interactions with tools or external resources the agent uses to fulfill specific parts of the workflow.

    The metadata of the workflow span includes the following.

    *   –

        Tasks: A list of task span that need to be completed as part of the workflow. This defines the specific actions or steps the agent needs to perform within this span.

    *   –

        Task Dependencies: Information about dependencies between tasks, indicating the order in which tasks should be executed or any prerequisites required for specific tasks. This helps manage the sequence and ensures that tasks are executed in a logical, efficient manner.

    *   –

        Operational Context: The situational information or environment details relevant to executing the workflow. This can include real-time conditions, status updates from other spans, or external factors that might influence task execution.

    *   –

        Past Execution History: Records of previous executions of similar workflows or tasks in (long-term) memory module, which can provide insights into best practices, potential pitfalls, or optimisation opportunities for the current workflow.

*   •

    Task Span: The task span represents a discrete unit of work or action within a workflow. It is a fundamental part of the workflow structure, defining individual tasks that the agent needs to execute in a sequence or parallel arrangement. The task span metadata includes:

    *   –

        Task Description: Specific information about the task to be performed, including task objectives, instructions, and parameters needed for execution.

    *   –

        Task Status: The current status (e.g., pending, in progress, completed) and the result of the task, which could include success, failure, or a specific output generated by the task.

*   •

    Tool Span: The tool span represents interactions with external tools or resources that assist in task execution. This span captures details about tools utilized by the agent, logging their configurations, responses, and any intermediate outputs. Tool Span metadata includes:

    *   –

        Tool Name: The name of the tool.

    *   –

        Tool Version: The version of the tool.

    *   –

        Configuration Settings: Parameters or settings configured for the tool during its use, such as version restriction, input formats, timeouts, or resource limits that may influence its behavior.

*   •

    Evaluation Span: The evaluation span assesses the correctness and quality of the agent’s actions and outputs performance against predefined criteria. It verifies whether the agent’s actions align with expectations or quality standards, providing a feedback mechanism to ensure that outputs meet the intended goals.

    The metadata of the evaluation span includes the following.

    *   –

        Test Cases: Specific scenarios or conditions under which the agent’s performance or outputs are evaluated. These cases provide a structured way to assess the agent’s actions and outcomes against expected behavior [[8](https://arxiv.org/html/2411.05285v2#bib.bib8)].

    *   –

        Testing Metrics: Quantitative or qualitative measures used to assess the agent’s performance. These metrics could include accuracy, efficiency, relevance, or other criteria that define the quality of the agent’s output.

    *   –

        Testing Results: The actual outcomes or findings from the evaluation process, indicating how well the agent’s performance aligns with the expected standards defined in the test cases and metrics.

*   •

    Guardrail Span: The guardrail span defines the guardrails applied to ensure the agent’s operations are aligned with expected governance requirements [[9](https://arxiv.org/html/2411.05285v2#bib.bib9)]. It helps prevent errors or unintended actions by setting boundaries on the agent’s behavior. Guardrail span metadata includes:

    *   –

        Guardrail Actions: Guardrails actions triggered, such as block, validation, filter, etc.

    *   –

        Guardrail Targets: Specific agent artifacts that guardrails are applied, such as goals and tools.

*   •

    LLM Span: The LLM Span captures interactions with an LLM, where the model processes language-based inputs to generate responses or insights. This span is essential for tasks involving natural language understanding, generation, or interpretation. LLM Span metadata could includes:

    *   –

        LLM Name: The name of the LLM.

    *   –

        LLM Version: The version of the LLM.

    *   –

        LLM Parameters: Settings applied to the LLM, such as temperature (affecting randomness), max_tokens (limiting response length), and other relevant hyper-parameters that shape the output.

## 5 Threats to Validity

Tool Selection Limitations: Due to the rapid proliferation of various tools and AI platforms, it is possible that not all relevant AgentOps tools were identified. To address this limitation, we selected tools from multiple data sources. The identified tools include both open-source AgentOps tools, such as AgentOps and Langfuse, as well as commercial observability platforms like Datadog.

Data Coverage Limitations: The comprehensive overview of traceable artifacts throughout the AgentOps life-cycle provided in this work may not encompass all possible data attributes related to AI agents. To ensure broader coverage of important traceable data across the entire life-cycle of an agent and enrich on the data attributes outlined in our paper, we have drawn on some relevant academic literature [[10](https://arxiv.org/html/2411.05285v2#bib.bib10), [9](https://arxiv.org/html/2411.05285v2#bib.bib9), [11](https://arxiv.org/html/2411.05285v2#bib.bib11)] to support our findings as well. However, some potentially valuable data, such as trace links and interactions between different steps, may have been missed. Future work will aim to explore these gaps further.

## 6 Conclusion

In this paper, we proposed AgentOps, a specialised DevOps paradigm tailored for LLM agents to enable observability. Through a systematic mapping study of existing AgentOps relevant tools, we first proposed an entity-relationship model to understand the key agent artifacts and their relationship. Then we presented a comprehensive taxonomy of AgentOps, offering a structured template for developers to design and implement AgentOps tools. Future research will focus on validating the proposed taxonomy through real-world case studies and the development of a AgentOps tool prototype.

## References

*   [1] R. Bommasani, D. A. Hudson, E. Adeli, R. Altman, S. Arora, S. von Arx, M. S. Bernstein, J. Bohg, A. Bosselut, E. Brunskill *et al.*, “On the opportunities and risks of foundation models,” *arXiv preprint arXiv:2108.07258*, 2021.
*   [2] Y. Liu, S. K. Lo, Q. Lu, L. Zhu, D. Zhao, X. Xu, S. Harrer, and J. Whittle, “Agent design pattern catalogue: A collection of architectural patterns for foundation model based agents,” 2024\. [Online]. Available: [https://arxiv.org/abs/2405.10467](https://arxiv.org/abs/2405.10467)
*   [3] Q. Lu, L. Zhu, X. Xu, Z. Xing, S. Harrer, and J. Whittle, “Towards responsible generative ai: A reference architecture for designing foundation model based agents,” in *2024 IEEE 21st International Conference on Software Architecture Companion (ICSA-C)*.   IEEE, 2024, pp. 119–126.
*   [4] Q. Lu, L. Zhu, J. Whittle, X. Xu *et al.*, *Responsible AI: Best Practices for Creating Trustworthy AI Systems*.   Addison-Wesley, 2023.
*   [5] L. Bass, Q. Lu, I. Weber, and L. Zhu, *Engineering AI Systems: Architecture and DevOps Essentials*.   Addison-Wesley, 2025.
*   [6] B. Kitchenham and S. Charters, “Guidelines for performing systematic literature reviews in software engineering version 2.3,” Software Engineering Group, School of Computer Science and Mathematics, Keele University and Department of Computer Science University of Durham, Tech. Rep., 2007.
*   [7] V. Garousi, M. Felderer, and M. V. Mäntylä, “Guidelines for including grey literature and conducting multivocal literature reviews in software engineering,” *Inf. Softw. Technol.*, vol. 106, pp. 101–121, 2019\. [Online]. Available: [https://doi.org/10.1016/j.infsof.2018.09.006](https://doi.org/10.1016/j.infsof.2018.09.006)
*   [8] B. Xia, Q. Lu, L. Zhu, Z. Xing, D. Zhao, and H. Zhang, “An evaluation-driven approach to designing llm agents: Process and architecture,” 2024\. [Online]. Available: [https://arxiv.org/abs/2411.13768](https://arxiv.org/abs/2411.13768)
*   [9] M. Shamsujjoha, Q. Lu, D. Zhao, and L. Zhu, “Designing multi-layered runtime guardrails for foundation model based agents: Swiss cheese model for ai safety by design,” 2024\. [Online]. Available: [https://arxiv.org/abs/2408.02205](https://arxiv.org/abs/2408.02205)
*   [10] S. Schulhoff, M. Ilie, N. Balepur, K. Kahadze, A. Liu, C. Si, Y. Li, A. Gupta, H. Han, S. Schulhoff, P. S. Dulepet, S. Vidyadhara, D. Ki, S. Agrawal, C. Pham, G. Kroiz, F. Li, H. Tao, A. Srivastava, H. D. Costa, S. Gupta, M. L. Rogers, I. Goncearenco, G. Sarli, I. Galynker, D. Peskoff, M. Carpuat, J. White, S. Anadkat, A. Hoyle, and P. Resnik, “The prompt report: A systematic survey of prompting techniques,” 2024\. [Online]. Available: [https://arxiv.org/abs/2406.06608](https://arxiv.org/abs/2406.06608)
*   [11] A. Chan, C. Ezell, M. Kaufmann, K. Wei, L. Hammond, H. Bradley, E. Bluemke, N. Rajkumar, D. Krueger, N. Kolt *et al.*, “Visibility into ai agents,” in *The 2024 ACM Conference on Fairness, Accountability, and Transparency*, 2024, pp. 958–973.