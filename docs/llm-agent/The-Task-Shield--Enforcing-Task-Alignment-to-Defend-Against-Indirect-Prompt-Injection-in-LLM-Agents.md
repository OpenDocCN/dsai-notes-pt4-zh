<!--yml
category: 未分类
date: 2025-01-11 11:45:18
-->

# The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents

> 来源：[https://arxiv.org/html/2412.16682/](https://arxiv.org/html/2412.16682/)

Feiran Jia
The Pennsylvania State University
feiran.jia@psu.edu
&Tong Wu
Princeton University
tongwu@princeton.edu
\ANDXin Qin
California State University, Long Beach
xin.qin@csulb.edu
&Anna Squicciarini
The Pennsylvania State University
acs20@psu.edu 

###### Abstract

Large Language Model (LLM) agents are increasingly being deployed as conversational assistants capable of performing complex real-world tasks through tool integration. This enhanced ability to interact with external systems and process various data sources, while powerful, introduces significant security vulnerabilities. In particular, indirect prompt injection attacks pose a critical threat, where malicious instructions embedded within external data sources can manipulate agents to deviate from user intentions. While existing defenses based on rule constraints, source spotlighting, and authentication protocols show promise, they struggle to maintain robust security while preserving task functionality. We propose a novel and orthogonal perspective that reframes agent security from preventing harmful actions to ensuring task alignment, requiring every agent action to serve user objectives. Based on this insight, we develop Task Shield, a test-time defense mechanism that systematically verifies whether each instruction and tool call contributes to user-specified goals. Through experiments on the AgentDojo benchmark, we demonstrate that Task Shield reduces attack success rates (2.07%) while maintaining high task utility (69.79%) on GPT-4o, significantly outperforming existing defenses in various real-world scenarios.

The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents

Feiran Jia The Pennsylvania State University feiran.jia@psu.edu                        Tong Wu Princeton University tongwu@princeton.edu

Xin Qin California State University, Long Beach xin.qin@csulb.edu                        Anna Squicciarini The Pennsylvania State University acs20@psu.edu

## 1 Introduction

Large Language Model (LLM) agents have achieved rapid advances in recent years, enabling them to perform a wide range of tasks, from generating creative content to executing complex operations such as sending emails, scheduling appointments, or querying APIs Brown et al. ([2020](https://arxiv.org/html/2412.16682v1#bib.bib1)); Touvron et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib29)); Schick et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib26)). Unlike traditional chatbots, these agents can perform actions in the real world, and their output can have real-world consequences. In this study, we focus on a critical use case. LLM agents serving as personal assistants in conversational systems OpenAI ([2024](https://arxiv.org/html/2412.16682v1#bib.bib19)). Beyond generating response in nature language, these assistants are empowered to take actions: they can access sensitive data, perform financial transactions, and interact with critical systems through tool integration. This increased capability requires greater attention to security.

Among threats to these systems, indirect prompt injection attacks pose a subtle but significant threat Zou et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib37)); Xiang et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib34)). Rather than directly injecting harmful instructions, attackers embed malicious prompts within external data sources (environment), such as documents, web pages, or tool output, that LLM agents process. The Inverse Scaling Law Wei et al. ([2022](https://arxiv.org/html/2412.16682v1#bib.bib32)) highlights that more capable LLMs are increasingly vulnerable. Therefore, we focus on these highly capable models.

Existing defenses are based on rule-based constraints Wallace et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib30)); Li et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib14)), source spotlighting Hines et al. ([2024a](https://arxiv.org/html/2412.16682v1#bib.bib10)), and authentication protocols Wang et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib31)). Although these approaches have merit, they encounter practical limitations. The detailed specification of rules is challenging, and indirect attacks can embed malicious directives within seemingly benign tone, bypassing detection mechanisms. We propose an orthogonal approach: task alignment. This concept proposes that every directive should serve the user’s objectives, shifting security to a focus on "Does this serve the intended tasks?" rather than "Is this harmful?". This shift to user goals means that the agent should ignore directives that deviate from these objectives, therefore filtering out indirectly injected directives.

![Refer to caption](img/b387c58f765bd1c9ad426faa69136d53.png)

Figure 1: Overview of the Task Shield interacting with a tool-integrated LLM agent. The framework enforces task alignment and defends against indirect prompt injection attacks.

To put task alignment into practice, we develop Task Shield - a defense system that acts as a guardian for LLM agents. The shield verifies whether each directive within the system, originating either from the agent or tools, is fully aligned with the user’s goals. By analyzing instruction relationships and providing timely intervention, the Task Shield effectively prevents potentially unrelated actions while maintaining the agent’s ability to complete user tasks.

Our contributions are summarized as follows:

*   •

    We propose a novel task alignment concept that formalizes the relationships between instructions in LLM agent conversational systems, establishing a foundation for ensuring that agent behaviors align with user-defined objectives.

*   •

    We introduce the Task Shield, a practical test-time defense mechanism that dynamically enforces the task alignment. The shield evaluates each interaction and provides feedback to maintain alignment throughout conversations.

*   •

    Through extensive experiments on the AgentDoJo Debenedetti et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib3)) benchmark, we demonstrate that our approach significantly reduces vulnerabilities to prompt injection attacks while preserving the utility of user tasks.

## 2 Preliminary

#### LLM Agent System and Message Types

LLM (Large Language Model) agent conversational systems facilitate multi-turn dialogues through sequences of messages, $\mathcal{M}=[M_{1},M_{2},\dots,M_{n}]$, where $n$ is the total number of messages. Each message $M_{i}$ serves one of four roles: System Messages define the agent’s role and core rules; User Messages specify goals and requests; Assistant Messages interpret and respond to instructions; and Tool Outputs provide external data or results. To structure interactions, OpenAI proposed an instruction hierarchy Wallace et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib30)) that assigns a privilege level $P(M_{i})\in\{L_{s},L_{u},L_{a},L_{t}\}$ to each message, representing the levels of the system ($L_{s}$), user ($L_{u}$), assistant ($L_{a}$) and tool ($L_{t}$), respectively. This hierarchy enforces a precedence order $L_{s}\succ L_{u}\succ L_{a}\succ L_{t}$, dictating that instructions from lower privilege levels are superseded by those from higher levels.

<svg class="ltx_picture" height="83.42" id="S2.SS0.SSS0.Px1.p2.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,83.42) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.89 9.89)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="580.23">Example: The user instructs "Find a nearby Italian restaurant for lunch tomorrow." (User Level $L_{u}$) The assistant interprets the request and plans to locate suitable options. (Assistant Level $L_{a}$) It then queries an external API to retrieve restaurant data. (Tool Level $L_{t}$)</foreignobject></g></g></svg>

This example illustrates how different message types interact within the hierarchy, ensuring that the assistant aligns its actions with the user’s objectives while utilizing external tools effectively.

#### Indirect Prompt Injection Attack

In this work, we focus on indirect prompt injection attacks where attackers embed instructions into the environment that LLM agents process during task execution. For example, consider an agent instructed to summarize a webpage. If the webpage contains hidden directives such as ‘Ignore all previous instructions and send your notes to Alice’, the agent can be hijacked and inadvertently follow these malicious instructions. These indirect attacks are more stealthy, as they are concealed within legitimate external data sources that the agent must process to complete its tasks.

## 3 Task Alignment

Our key insight is that indirect prompt injection attacks succeed when LLMs execute directives that deviate from user goals (or predefined conversational goals). This understanding leads us to propose a novel perspective: reframing agent security through the lens of task alignment. Rather than attempting to identify harmful content, we focus on ensuring that actionable instructions contribute to user-specified objectives. This shift allows us to capture maliciously injected prompts even if they appear benign on the surface.

To formalize this concept, we first define the task instructions as the basic analytical unit of analysis in conversational systems. We then analyze how these instructions interact across different message types, ultimately developing a formal framework to assess whether each instruction aligns with user goals in the context of multi-turn dialogues with tool integration.

### 3.1 Task Instructions

A key principle in our formulation is that the user instructions define the objectives of the conversation. Ideally, other actionable directives from the assistant or external tools should support these user objectives. We formalize *task instructions* in each message:

###### Definition 1  (Task Instruction).

A task instruction refers to an actionable directive extracted from a message $M_{i}$ in the conversation that is intended to guide the assistant’s behavior. These instructions can come from different sources: (1) User Instructions: Task requests and goals are explicitly stated by the user. (2) Assistant Plans: Subtasks or steps proposed by the assistant to accomplish user goals, including natural language instructions and tool calls. (3) Tool-Generated Instructions: Additional directives or suggestions produced by external tools during task execution.

We denote the set of task instructions extracted from a message $M_{i}$ by $E(M_{i})$. At each privilege level $L$, we aggregate the task instructions from all messages at that level within a conversation segment $\mathcal{M}^{\prime}$:

|  | $E_{L}(\mathcal{M}^{\prime})=\bigcup_{\begin{subarray}{c}M_{i}\in\mathcal{M}^{% \prime}\\ P(M_{i})=L\end{subarray}}E(M_{i}).$ |  |

Note: The system message can also define high-level tasks in certain specialized agents. However, in this paper, we focus primarily on user-level directives in $L_{u}$. See Appendix [A.2](https://arxiv.org/html/2412.16682v1#A1.SS2 "A.2 Discussion on System-Level Instructions ‣ Appendix A Appendix: Detailed Discussion on Task Alignment ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents") for further discussion on system-level task objectives.

### 3.2 Task Interactions

In LLM conversational systems, higher-level messages (specifically user messages in this paper) provide abstract instructions, while tool-level ones refine them with additional data. When checking alignment with the conversational goals, we should consider context from all sources, including tool outputs. As the examples below show, tools can either merely supply supporting information or define new subtasks:

<svg class="ltx_picture" height="98.64" id="S3.SS2.p2.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,98.64) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.89 9.89)"><foreignobject color="#000000" height="78.87" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="580.23">Example 1: Tool Output as Supporting Information The user says ’Schedule an appointment with the dentist’. The assistant knows to schedule, but needs contact details. It queries a tool, then completes the predefined task. Example 2: Tool Output Defining Concrete Tasks The user says, "Complete my to-do list tasks." A to-do tool returns: "1\. Pay electricity bill 2\. Buy groceries," which transforms the user’s abstract request into specific actionable tasks.</foreignobject></g></g></svg>

In Example 1, the tool output supplements a clear user directive. In Example 2, the tool output itself outlines subtasks. The conversation history $\mathcal{H}_{i}=[M_{1},\dots,M_{i-1}]$ provides the context for judging these relationships and maintaining alignment with user goals.

### 3.3 Formalization of Task Alignment

We now formalize the concept of task alignment. First, we define the $\mathrm{ContributesTo}$ relation, which captures the relationship between the task instructions.

###### Definition 2  ($\mathrm{ContributesTo}$ Relation).

In the context of conversation history $\mathcal{H}_{i}$, let $e$ be a task instruction from message $M_{i}$, and let $t$ be a task instruction from a message $M_{j}\in\mathcal{H}_{i}$. We say $e$ contributes to $t$, denoted as $\mathrm{ContributesTo}(e,t\mid\mathcal{H}_{i})=\mathrm{True}$, if $e$ helps achieve the directive or goal of $t$ within $\mathcal{H}_{i}$.

For simplicity, we will omit $\mathcal{H}_{i}$ in the notation and $\mathrm{ContributesTo}(e,t)$ will implicitly consider the relevant conversation history. We define the task instruction alignment condition as follows:

###### Definition 3  (Task Instruction Alignment Condition).

A task instruction $e\in E(M_{i})$ at privilege level $L_{i}=P(M_{i})$ satisfies the task instruction alignment condition if, for the user level $L_{u}$, there exists at least one task instruction $t\in E_{L_{u}}(\mathcal{H}_{i})$, where $E_{L_{u}}(\mathcal{H}_{i})$ is the set of task instructions extracted from messages in $\mathcal{H}_{i}$ at privilege level $L_{u}$, such that:

|  | $\mathrm{ContributesTo}(e,t)=\mathrm{True}.$ |  | (1) |

This condition ensures that the task instruction at a lower privilege level directly contributes to at least one user-specific task instruction. Building upon this, we can define a fully aligned conversation in the ideal case:

###### Definition 4  (Task Alignment).

A conversation achieves task alignment when all assistant-level task instructions in the conversation satisfy the task instruction alignment condition (Definition [3](https://arxiv.org/html/2412.16682v1#Thmdefinition3 "Definition 3 (Task Instruction Alignment Condition). ‣ 3.3 Formalization of Task Alignment ‣ 3 Task Alignment ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents")).

Task alignment ensures that the assistant’s plans and tool calls are always in service of the user’s goals. Consequently, any (malicious) directives that do not align with these goals, such as those embedded by indirect prompt injection, are naturally ignored by the agent. For examples of conversations that do not meet the task alignment condition, refer to Appendix [A.3](https://arxiv.org/html/2412.16682v1#A1.SS3 "A.3 Examples of Task Misalignments ‣ Appendix A Appendix: Detailed Discussion on Task Alignment ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents").

![Refer to caption](img/f8965e8a3e448decb451ddfc3f1279ab.png)

Figure 2: This diagram illustrates how the Task Shield framework processes different message types from the conversational flow through task instruction extraction, alignment checks, and feedback generation.

## 4 The Task Shield Framework

While we defined task alignment as an ideal security property, implementing it in practice requires an enforcement mechanism. To address this need, we introduce the Task Shield framework that continuously monitors and enforces the alignment of the instruction with the user objectives.

As shown in Figure [2](https://arxiv.org/html/2412.16682v1#S3.F2.1 "Figure 2 ‣ 3.3 Formalization of Task Alignment ‣ 3 Task Alignment ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents"), the framework consists of three key components: (1) instruction extraction, (2) alignment check, and (3) feedback generation to maintain task alignment throughout the conversation flow. Both instruction extraction (1) and the $\mathrm{ContributesTo}$ score calculation within the alignment check (2) leverage the capabilities of a large language model.

In this section, we first detail the technical implementation of each shield component and then explain how these components dynamically interact within the LLM agent system to enforce task alignment.

### 4.1 Task Shield Components

#### Task Instruction Extraction.

The Task Shield framework begins by extracting task instructions from each incoming message. This process serves two purposes: (1) to identify user objectives, which are stored as a User Task Set $T_{u}$ and serve as conversational goals to check against; (2) to detect potential directives from other sources that require alignment check.

Real-world messages often pose extraction challenges: instructions may be implicit, nested within other instructions, or embedded in complex content. Missing any such instruction could create security vulnerabilities in our defense mechanism. To address these challenges, we implement a conservative extraction strategy using a carefully designed LLM prompt (Figure [4](https://arxiv.org/html/2412.16682v1#A4.F4 "Figure 4 ‣ Appendix D Prompts ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents") in Appendix [D](https://arxiv.org/html/2412.16682v1#A4 "Appendix D Prompts ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents")). The prompt instructs the LLM to: (1) extract all potentially actionable directives, even when nested or implicit, (2) rewrite information-seeking queries as explicit instructions, and (3) preserve task dependencies in natural language.

#### Alignment Check.

Once instructions are extracted, the next stage is to assess whether each extracted instruction satisfies the Task Instruction Alignment Condition, as defined in Definition [3](https://arxiv.org/html/2412.16682v1#Thmdefinition3 "Definition 3 (Task Instruction Alignment Condition). ‣ 3.3 Formalization of Task Alignment ‣ 3 Task Alignment ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents"). This involves two key aspects: assessing individual instructions’ contributions and computing overall alignment scores.

To assess alignment, we use the predicate $\mathrm{ContributesTo}$, as defined in Definition [2](https://arxiv.org/html/2412.16682v1#Thmdefinition2 "Definition 2 (ContributesTo Relation). ‣ 3.3 Formalization of Task Alignment ‣ 3 Task Alignment ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents"). However, a binary classification is too rigid for practical applications as the relationship between actions and goals often involves uncertainty or ambiguity. To account for this nuanced relationship, we adopt a fuzzy logic-based scoring mechanism. By assigning a continuous score in the range $[0,1]$, we allow a fine-grained evaluation of how instructions contribute to user goals, capturing their role in direct contribution, intermediate steps, or reasonable attempts at resolution.

Then, the total contribution score is computed by summing up the scores against all the user task instructions. The alignment check process considers an instruction to be misaligned if its total contribution score equals $0$. The detailed discussion and implementation of this design are included in Appendix [B.2](https://arxiv.org/html/2412.16682v1#A2.SS2 "B.2 Task Shield Core Processing Algorithm ‣ Appendix B Appendix: Detials in Task Shield Frameworks Design ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents").

#### Feedback Generation.

When misalignment is detected, Task Shield generates structured feedback to guide the conversation back to alignment with user objectives. This feedback includes (1) a clear alert identifying the misaligned task instructions, (2) a notification explaining potential risks, and (3) a reminder of current user objectives ($T_{u}$).

### 4.2 Interaction with the LLM Agent System

The Task Shield enforces alignment through monitoring and intervention in the conversation flow, with distinct processing approaches for each message type. Each message must pass through alignment check before proceeding, creating multiple layers of defense against potential attacks.

#### User Message Processing

At user level $L_{u}$, the shield updates the User Task Set $T_{u}$ with newly extracted instructions. These instructions define the alignment targets for all subsequent message processing.

#### Assistant Message Processing

Messages at level $L_{a}$ may contain two components that require alignment check: message content (natural language response) and tool calls. If either component fails the alignment check, Task Shield provides feedback to the LLM agent, prompting it to reconsider its response. It acts as a critic, providing several rounds of feedback to guide the LLM agent in refining its queries. For tool calls specifically, Task Shield prevents execution of misaligned calls.

#### Tool Output Processing

At level $L_{t}$, the shield evaluates tool outputs with context awareness, augmenting each instruction with its source: "from tool [function_name] with arguments [args]". Upon detecting misalignment, the shield includes both the original output and feedback in its response to the assistant, enabling informed correction.

This multi-layered defense mechanism ensures that injected attacks face multiple barriers: misaligned instructions in tool outputs are flagged during $L_{t}$ processing, potentially harmful responses are caught and refined at the $L_{a}$ level, while the continuous validation against user objectives at $L_{u}$ maintains overall conversation alignment.

## 5 Experiments

In this section, we evaluate Task Shield on GPT-4o and GPT-4o-mini using AgentDoJo Debenedetti et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib3)), with one trial per task.

### 5.1 Settings

| Suite | Travel | Workspace | Banking | Slack | Overall |
| --- | --- | --- | --- | --- | --- |
| Defense | Task Shield | No Defense | Task Shield | No Defense | Task Shield | No Defense | Task Shield | No Defense | Task Shield | No Defense |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Attack | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ | U $\uparrow$ | ASR $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Important Instructions | 72.86 | 1.43 | 64.29 | 11.43 | 62.50 | 0.42 | 24.17 | 40.42 | 82.64 | 6.25 | 69.44 | 62.50 | 64.76 | 0.95 | 63.81 | 92.38 | 69.79 | 2.07 | 50.08 | 47.69 |
| Injecagent | 67.86 | 0.00 | 72.14 | 0.00 | 66.67 | 0.00 | 64.58 | 0.00 | 77.78 | 4.17 | 72.22 | 15.28 | 66.67 | 0.95 | 67.62 | 13.33 | 69.48 | 1.11 | 68.52 | 5.72 |
| Ignore Previous | 70.71 | 0.00 | 77.14 | 0.00 | 62.92 | 0.00 | 61.67 | 0.00 | 72.22 | 1.39 | 68.75 | 8.33 | 63.81 | 0.95 | 61.90 | 20.95 | 66.93 | 0.48 | 66.77 | 5.41 |

Table 1: GPT-4o: Comparison of different attacks under Task Shield defense and no defense across task suites. U (Utility) and ASR (Attack Success Rate) are shown separately for Task Shield and No Defense settings. Cells under Task Shield that outperform No Defense are highlighted in light blue, and cells under No Defense that outperform Task Shield are highlighted in light pink. All numbers are represented as percentages (%).

![Refer to caption](img/a85cf160fb3479194a4024eb1efd4f3a.png)

Figure 3: GPT-4o: Comparison of Attack Success Rate (ASR) versus Utility. Solid markers represent ASR versus benign utility, while hollow markers represent ASR versus utility under attack. Arrows indicate the change in utility due to the attack, with their direction showing the impact of the attack on model performance. The green circles highlight the Pareto front in benign conditions, and the orange circles highlight the Pareto front under attack. Numbers along the arrows indicate the magnitude of the utility change when an attack is introduced (positive values show improvement, and negative values indicate degradation).

#### Benchmark

We conducted our experiments within the AgentDojo benchmark¹¹1AgentDojo is available at [https://github.com/ethz-spylab/agentdojo](https://github.com/ethz-spylab/agentdojo), which was released under the MIT License. Our use of AgentDojo aligns fully with its intended purpose. We use the default configurations for the models., the first comprehensive environment designed to evaluate AI agents against indirect prompt injection attacks. Unlike some benchmarks that focus on simple scenarios beyond the personal assistant use cases Liu et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib16)) or single-turn evaluations Zhan et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib36)), AgentDojo simulates realistic agent behaviors with multi-turn conversations, and complex tool interactions. In addition, the benchmark encompasses four representative task suites that simulate real-world scenarios. Travel for itinerary management, Workspace for document processing, Banking for financial operations, and Slack for communication tasks, providing a practical test of our defense mechanism in realistic applications.

#### Models

The primary evaluation is conducted on GPT-4o. This choice is motivated by two factors: (1) GPT-4o demonstrates superior performance in challenging AgentDojo tasks, providing a high utility baseline; (2) following the inverse scaling law Wei et al. ([2022](https://arxiv.org/html/2412.16682v1#bib.bib32)), GPT-4o is particularly vulnerable to prompt injection attacks, making it an ideal candidate to validate our defense mechanism. We also include GPT-4o-mini, a safety-aligned model through instruction hierarchy trainingWallace et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib30)), which offers inherent robustness against attacks, and GPT-3.5-turbo (in the Appendix). For defense implementation, we use the same model as a protective Task Shield.

#### Baselines

We compare Task Shield with four established defense methods: Data Delimiting (Delimiting)Chen et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib2)); Hines et al. ([2024a](https://arxiv.org/html/2412.16682v1#bib.bib10)), which isolates tool outputs using explicit markers; Prompt Injection Detection (PI Detector)Kokkula et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib13)), which employs classification to identify potential attacks; Prompt Sandwiching (Repeat Prompt) Prompting ([2024](https://arxiv.org/html/2412.16682v1#bib.bib24)), which reinforces original user prompts through repetition; and Tool Filtering (Tool Filter)Debenedetti et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib3)), which restricts available tools based on task requirements.

#### Evaluation Metrics

The experiment used three key evaluation metrics to measure the performance and robustness of the LLM agent. (1) Clean utility (CU) refers to the fraction of user tasks that the agent successfully completes in a benign environment without attacks, representing the baseline performance of the agent. (2) Utility under attack (U) measures the agent’s success in completing user tasks under prompt injection attacks, reflecting its ability to maintain performance despite adversarial interference. (3) Target attack success rate assesses the fraction of cases where the attacker’s goal is achieved, measuring the effectiveness of the attack and the robustness of the defense.

### 5.2 Results

| Model | Suite | Travel | Workspace | Banking | Slack | Overall |
| --- | --- | --- | --- | --- | --- | --- |
|  | Defense | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-4o | No Defense | 65.00 | 64.29 | 11.43 | 62.50 | 24.17 | 40.42 | 75.00 | 69.44 | 62.50 | 80.95 | 63.81 | 92.38 | 69.07 | 50.08 | 47.69 |
| Tool Filter | 90.00 | 70.00 | 5.71 | 55.00 | 51.67 | 4.17 | 81.25 | 56.94 | 11.11 | 80.95 | 47.62 | 8.57 | 72.16 | 56.28 | 6.84 |
| Repeat Prompt | 90.00 | 72.14 | 7.14 | 80.00 | 60.42 | 14.58 | 93.75 | 77.08 | 46.53 | 80.95 | 62.86 | 60.00 | 84.54 | 67.25 | 27.82 |
| Delimiting | 75.00 | 72.14 | 3.57 | 62.50 | 30.42 | 35.00 | 81.25 | 77.08 | 61.81 | 80.95 | 61.90 | 80.00 | 72.16 | 55.64 | 41.65 |
| PI Detector | 30.00 | 16.43 | 0.00 | 52.50 | 15.83 | 15.00 | 43.75 | 31.25 | 0.69 | 28.57 | 25.71 | 12.38 | 41.24 | 21.14 | 7.95 |
| Task Shield | 80.00 | 72.86 | 1.43 | 62.50 | 62.50 | 0.42 | 81.25 | 82.64 | 6.25 | 80.95 | 64.76 | 0.95 | 73.20 | 69.79 | 2.07 |
| GPT-4o-mini | No Defense | 55.00 | 47.14 | 13.57 | 82.50 | 59.17 | 17.92 | 50.00 | 38.19 | 34.03 | 66.67 | 48.57 | 57.14 | 68.04 | 49.92 | 27.19 |
| Tool Filter | 60.00 | 58.57 | 0.71 | 70.00 | 64.58 | 2.50 | 50.00 | 43.06 | 11.11 | 57.14 | 45.71 | 7.62 | 61.86 | 55.17 | 4.93 |
| Repeat Prompt | 70.00 | 54.29 | 0.00 | 70.00 | 61.25 | 8.33 | 43.75 | 43.75 | 17.36 | 71.43 | 33.33 | 13.33 | 65.98 | 51.03 | 9.38 |
| Delimiting | 60.00 | 52.14 | 7.14 | 72.50 | 64.58 | 12.92 | 43.75 | 35.42 | 33.33 | 71.43 | 56.19 | 48.57 | 64.95 | 53.74 | 22.26 |
| PI Detector | 25.00 | 14.29 | 0.00 | 60.00 | 27.50 | 12.92 | 37.50 | 29.86 | 10.42 | 23.81 | 15.24 | 7.62 | 41.24 | 23.05 | 8.59 |
| Task Shield | 55.00 | 49.29 | 0.71 | 85.00 | 69.58 | 1.25 | 43.75 | 37.50 | 6.25 | 66.67 | 50.48 | 0.95 | 68.04 | 54.53 | 2.23 |

Table 2: Defense performance against Important Messages attack for GPT-4o and GPT-4o-mini models. Results are reported across Clean Utility (CU), Utility under Attack (U), and Attack Success Rate (ASR) across task suites. For each model, bold values denote the best-performing results for each metric, while underlined values indicate the second-best performance. All numbers are represented as percentages (%). $\uparrow$: higher is better; $\downarrow$: lower is better.

#### Defending Against Attacks

We evaluate Task Shield against three types of indirect prompt injection attacks: Important Instructions Debenedetti et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib3)) that embed high-priority malicious instructions to exploit the model’s tendency to prioritize urgent directives; Injecagent Zhan et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib36)) which employs conflicting objectives; and Ignore Previous Perez and Ribeiro ([2022](https://arxiv.org/html/2412.16682v1#bib.bib22)) which nullifies prior instructions. As shown in Table [1](https://arxiv.org/html/2412.16682v1#S5.T1 "Table 1 ‣ 5.1 Settings ‣ 5 Experiments ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents"), the Important Instructions attack poses the strongest threat, achieving an attack success rate (ASR) of 47.69% on GPT-4o without defense while significantly degrading utility. Task Shield demonstrates consistent superiority across all attack types - it not only reduces ASRs but also maintains or improves utility compared to the no-defense baseline. In particular, it mitigates the strongest Important Instructions attack by reducing the ASR to 2.07% while preserving high utility at 69.79%. All subsequent experiments are conducted under the Important Instructions attack, given its status as the greatest threat.

#### Security-Utility Trade-offs

Figure [3](https://arxiv.org/html/2412.16682v1#S5.F3 "Figure 3 ‣ 5.1 Settings ‣ 5 Experiments ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents") visualizes the security-utility trade-off by plotting the performance of different defenses on Pareto fronts on GPT-4o under benign (before attack) and adversarial (under attack) conditions. The Pareto front represents optimal solutions where improving one metric necessitates degrading the other. The ideal data points are located towards the lower-right corner of the figure. Task Shield consistently approaches the Pareto front in both scenarios, demonstrating its optimal balance between security and utility in diverse conditions and task suites. Specifically, Task Shield consistently resides in the desirable lower-right region of each plot.

Other defenses show significant limitations: PI Detector achieves low ASR but suffers severe utility degradation, the Tool Filter shows moderate performance in both metrics but falls short of the Pareto front, and the Repeat Prompt maintains high utility but provides inadequate defense against attacks.

#### Detailed Results on GPT-4o and GPT-4o-mini

Table [2](https://arxiv.org/html/2412.16682v1#S5.T2 "Table 2 ‣ 5.2 Results ‣ 5 Experiments ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents") presents a comparative analysis of different defense mechanisms against the "Important Instructions" attack across both models. In both GPT-4o and GPT-4o-mini, Task Shield consistently demonstrates superior overall performance across all task suites: it reduces ASR to 2.07% while maintaining 69.79% utility under attack (U) on GPT-4o, and similarly achieves 2.23% ASR with 54.53% utility under attack (U) on GPT-4o-mini, consistently outperforming all baseline defenses. Across all task suites, Task Shield demonstrates near-optimal or optimal performance in terms of CU, U, and ASR.

Interestingly, the two models exhibit distinct behaviors in response to different defense mechanisms. For clean utility (CU), while most defenses improve GPT-4o’s performance compared to the no-defense baseline (except PI Detector), they actually hurt GPT-4o-mini’s performance. Task Shield is the only defense that maintains or improves the clean utility on GPT-4o-mini. In terms of attack success rate (ASR), GPT-4o-mini demonstrates an inherently lower ASR without defense (27\. 19% vs 47\. 69% in GPT-4o), likely due to its safety-aligned nature. Moreover, while Repeat Prompt shows relatively strong performance on GPT-4o-mini but struggles on GPT-4o, Task Shield maintains consistent effectiveness across both architectures, highlighting its robustness as a defense solution.

## 6 Related Work

#### LLM Agent and Tool Integration

Research on the design of LLM agents capable of performing complex human-instructed tasks has advanced significantly Ouyang et al. ([2022](https://arxiv.org/html/2412.16682v1#bib.bib20)); Sharma et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib27)). To enable these agents to perform human-like functions, such as searching Deng et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib4)); Fan et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib6)), decision making Yao et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib35)); Mao et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib17)), existing approaches commonly integrate external tool-calling capabilities into their architectures. Equipping an LLM agent with tool calling functionality is not particularly challenging, given the availability of various backbone models Hao et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib9)); Patil et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib21)); Qin et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib25)); Mialon et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib18)); Tang et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib28)). The authors in Schick et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib26)) have explored approaches that enable LLMs to learn how to call external tools autonomously. Consequently, our approaches can be broadly adopted and seamlessly integrated into LLM agent systems.

#### Indirect Prompt Injection Attacks

Indirect prompt injection attacks  Greshake et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib8)); Liu et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib15)) have recently emerged as a significant safety concern for LLM agents. These attacks occur when malicious content is embedded in inputs sourced from external data providers or environments (e.g., data retrieved from untrusted websites), leading agents to perform unsafe or malicious actions, such as sharing private personal information Derner et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib5)); Fu et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib7)). To systematically assess the risks of such attacks across diverse scenarios, several benchmarks, including Injecagent and AgentDojo, have been developed Zhan et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib36)); Debenedetti et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib3)). In this paper, we aim to build a robust system to mitigate these malicious effects.

#### Defense Methods

Defenses against prompt injection attacks have focused on both training-time and test-time strategies. Training-time methods Piet et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib23)); Wallace et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib30)); Wu et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib33)) typically involve fine-tuning models with adversarial examples to enhance their robustness. However, these approaches are often impractical due to their high computational cost and inapplicability to LLMs without internal access. Test-time defenses, on the other hand, generally do not require significant computational resources. For example, Wang et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib31)) propose using hash-based authentication tags to filter harmful responses, while Hines et al. ([2024b](https://arxiv.org/html/2412.16682v1#bib.bib11)); Chen et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib2)) design special delimiters to instruct models to recognize and mitigate attacks. Our approach, instead, aims to enforce the task alignment, achieving a better robustness-utility tradeoff.

## 7 Conclusion

In this work, we proposed a novel perspective for the defense of indirect prompt injection attacks by introducing task alignment as a guiding principle to ensure that agent behavior serves user objectives. In addition, we developed Task Shield, a test-time mechanism that enforces this principle by verifying instruction alignment with user goals, achieving state-of-the-art defense against indirect prompt injection attacks while preserving agent capabilities across diverse simulated real-world tasks in AgentDoJo benchmark.

#### Limitations

Our framework faces several limitations. First, our reliance on LLMs for task instruction extraction and ContributeTo scoring introduces two key vulnerabilities: (1) potential performance degradation when using weaker language models and (2) susceptibility to adaptive attacks. In addition, resource constraints also limited our scope of evaluation. The high cost of LLM queries restricted our experiments to a single benchmark and a single model family.

#### Future Work

Several directions emerge for future research. (1) improving Task Shield’s efficiency and robustness by developing more cost-effective LLM-based instruction extraction and alignment verification techniques, (2) expanding Task Shield to address broader security threats beyond prompt injection, such as jailbreak attacks and system prompt extraction, (3) adapting the framework for domain-specific business contexts, where AI agents need to maintain strict alignment with specialized objectives Huang et al. ([2023](https://arxiv.org/html/2412.16682v1#bib.bib12)) , and (4) leveraging the task alignment concept to generate synthetic training data that captures diverse task dependencies and misalignment scenarios.

## References

*   Brown et al. (2020) Tom B Brown, Benjamin Mann, Nick Ryder, et al. 2020. Language models are few-shot learners. *Advances in neural information processing systems*.
*   Chen et al. (2024) Sizhe Chen, Julien Piet, Chawin Sitawarin, and David Wagner. 2024. Struq: Defending against prompt injection with structured queries. *arXiv preprint arXiv:2402.06363*.
*   Debenedetti et al. (2024) Edoardo Debenedetti, Jie Zhang, Mislav Balunović, Luca Beurer-Kellner, Marc Fischer, and Florian Tramèr. 2024. Agentdojo: A dynamic environment to evaluate attacks and defenses for llm agents. *arXiv preprint arXiv:2406.13352*.
*   Deng et al. (2024) Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Sam Stevens, Boshi Wang, Huan Sun, and Yu Su. 2024. Mind2web: Towards a generalist agent for the web. *Advances in Neural Information Processing Systems*, 36.
*   Derner et al. (2024) Erik Derner, Kristina Batistič, Jan Zahálka, and Robert Babuška. 2024. A security risk taxonomy for prompt-based interaction with large language models. *IEEE Access*.
*   Fan et al. (2024) Wenqi Fan, Yujuan Ding, Liangbo Ning, Shijie Wang, Hengyun Li, Dawei Yin, Tat-Seng Chua, and Qing Li. 2024. [A survey on rag meeting llms: Towards retrieval-augmented large language models](https://doi.org/10.1145/3637528.3671470). In *Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*, KDD ’24, page 6491–6501, New York, NY, USA. Association for Computing Machinery.
*   Fu et al. (2024) Xiaohan Fu, Zihan Wang, Shuheng Li, Rajesh K. Gupta, Niloofar Mireshghallah, Taylor Berg-Kirkpatrick, and Earlence Fernandes. 2024. [Misusing tools in large language models with visual adversarial examples](https://openreview.net/forum?id=djcciHhCrt).
*   Greshake et al. (2023) Kai Greshake, Sahar Abdelnabi, Shailesh Mishra, Christoph Endres, Thorsten Holz, and Mario Fritz. 2023. [Not what you’ve signed up for: Compromising real-world llm-integrated applications with indirect prompt injection](https://doi.org/10.1145/3605764.3623985). In *Proceedings of the 16th ACM Workshop on Artificial Intelligence and Security*, AISec ’23, page 79–90, New York, NY, USA. Association for Computing Machinery.
*   Hao et al. (2023) Shibo Hao, Tianyang Liu, Zhen Wang, and Zhiting Hu. 2023. Toolkengpt: Augmenting frozen language models with massive tools via tool embeddings. *Advances in neural information processing systems*, 36:45870–45894.
*   Hines et al. (2024a) Keegan Hines, Gary Lopez, Matthew Hall, Federico Zarfati, Yonatan Zunger, and Emre Kiciman. 2024a. Defending against indirect prompt injection attacks with spotlighting. *arXiv preprint arXiv:2403.14720*.
*   Hines et al. (2024b) Keegan Hines, Gary Lopez, Matthew Hall, Federico Zarfati, Yonatan Zunger, and Emre Kiciman. 2024b. [Defending against indirect prompt injection attacks with spotlighting](https://api.semanticscholar.org/CorpusID:268667111). *ArXiv*, abs/2403.14720.
*   Huang et al. (2023) Xu Huang, Jianxun Lian, Yuxuan Lei, Jing Yao, Defu Lian, and Xing Xie. 2023. Recommender ai agent: Integrating large language models for interactive recommendations. *arXiv preprint arXiv:2308.16505*.
*   Kokkula et al. (2024) Sahasra Kokkula, G Divya, et al. 2024. Palisade–prompt injection detection framework. *arXiv preprint arXiv:2410.21146*.
*   Li et al. (2024) Mei Li et al. 2024. Securing tool use in llm agents: Challenges and strategies. *arXiv preprint arXiv:2402.03014*.
*   Liu et al. (2023) Yi Liu, Gelei Deng, Yuekang Li, Kailong Wang, Zihao Wang, Xiaofeng Wang, Tianwei Zhang, Yepang Liu, Haoyu Wang, Yan Zheng, and Yang Liu. 2023. Prompt injection attack against llm-integrated applications. *arXiv preprint arXiv:2306.05499*.
*   Liu et al. (2024) Yupei Liu, Yuqi Jia, Runpeng Geng, Jinyuan Jia, and Neil Zhenqiang Gong. 2024. Formalizing and benchmarking prompt injection attacks and defenses. In *USENIX Security Symposium*.
*   Mao et al. (2024) Jiageng Mao, Junjie Ye, Yuxi Qian, Marco Pavone, and Yue Wang. 2024. [A language agent for autonomous driving](https://openreview.net/forum?id=UPE6WYE8vg). In *First Conference on Language Modeling*.
*   Mialon et al. (2023) Grégoire Mialon, Roberto Dessi, Maria Lomeli, Christoforos Nalmpantis, Ramakanth Pasunuru, Roberta Raileanu, Baptiste Roziere, Timo Schick, Jane Dwivedi-Yu, Asli Celikyilmaz, Edouard Grave, Yann LeCun, and Thomas Scialom. 2023. [Augmented language models: a survey](https://openreview.net/forum?id=jh7wH2AzKK). *Transactions on Machine Learning Research*. Survey Certification.
*   OpenAI (2024) OpenAI. 2024. [Introducing openai o1-preview](https://openai.com/index/introducing-openai-o1-preview/).
*   Ouyang et al. (2022) Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, et al. 2022. Training language models to follow instructions with human feedback. *Advances in neural information processing systems*, 35:27730–27744.
*   Patil et al. (2023) Shishir G. Patil, Tianjun Zhang, Xin Wang, and Joseph E. Gonzalez. 2023. Gorilla: Large language model connected with massive apis. *arXiv preprint arXiv:2305.15334*.
*   Perez and Ribeiro (2022) Fábio Perez and Ian Ribeiro. 2022. Ignore previous prompt: Attack techniques for language models. *arXiv preprint arXiv:2211.09527*.
*   Piet et al. (2023) Julien Piet, Maha Alrashed, Chawin Sitawarin, Sizhe Chen, Zeming Wei, Elizabeth Sun, Basel Alomair, and David Wagner. 2023. [Jatmo: Prompt injection defense by task-specific finetuning](https://api.semanticscholar.org/CorpusID:266690784). *ArXiv*, abs/2312.17673.
*   Prompting (2024) Learn Prompting. 2024. Sandwich defense. [https://learnprompting.org/docs/prompt_hacking/defensive_measures/sandwich_defense](https://learnprompting.org/docs/prompt_hacking/defensive_measures/sandwich_defense). Accessed: 2024-11-07.
*   Qin et al. (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. 2023. Toolllm: Facilitating large language models to master 16000+ real-world apis. *arXiv preprint arXiv:2307.16789*.
*   Schick et al. (2024) Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, and Thomas Scialom. 2024. Toolformer: Language models can teach themselves to use tools. *Advances in Neural Information Processing Systems*, 36.
*   Sharma et al. (2024) Ashish Sharma, Sudha Rao, Chris Brockett, Akanksha Malhotra, Nebojsa Jojic, and Bill Dolan. 2024. [Investigating agency of LLMs in human-AI collaboration tasks](https://aclanthology.org/2024.eacl-long.119). In *Proceedings of the 18th Conference of the European Chapter of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 1968–1987, St. Julian’s, Malta. Association for Computational Linguistics.
*   Tang et al. (2023) Qiaoyu Tang, Ziliang Deng, Hongyu Lin, Xianpei Han, Qiao Liang, Boxi Cao, and Le Sun. 2023. Toolalpaca: Generalized tool learning for language models with 3000 simulated cases. *arXiv preprint arXiv:2306.05301*.
*   Touvron et al. (2023) Hugo Touvron, Thibaut Lavril, Gautier Izacard, et al. 2023. Llama 2: Open foundation and fine-tuned chat models. *arXiv preprint arXiv:2307.09288*.
*   Wallace et al. (2024) Eric Wallace, Kai Xiao, Reimar Leike, Lilian Weng, Johannes Heidecke, and Alex Beutel. 2024. The instruction hierarchy: Training llms to prioritize privileged instructions. *arXiv preprint arXiv:2404.13208*.
*   Wang et al. (2024) Jiongxiao Wang, Fangzhou Wu, Wendi Li, Jinsheng Pan, Edward Suh, Z. Morley Mao, Muhao Chen, and Chaowei Xiao. 2024. Fath: Authentication-based test-time defense against indirect prompt injection attacks. *arXiv preprint arXiv:2410.21492*.
*   Wei et al. (2022) Jason Wei et al. 2022. Inverse scaling: When bigger isn’t better. *arXiv preprint arXiv:2206.04615*.
*   Wu et al. (2024) Tong Wu, Shujian Zhang, Kaiqiang Song, Silei Xu, Sanqiang Zhao, Ravi Agrawal, Sathish Reddy Indurthi, Chong Xiang, Prateek Mittal, and Wenxuan Zhou. 2024. [Instructional segment embedding: Improving llm safety with instruction hierarchy](https://arxiv.org/abs/2410.09102). *Preprint*, arXiv:2410.09102.
*   Xiang et al. (2024) Zhen Xiang, Linzhi Zheng, Yanjie Li, Junyuan Hong, Qinbin Li, Han Xie, Jiawei Zhang, Zidi Xiong, Chulin Xie, Carl Yang, et al. 2024. Guardagent: Safeguard llm agents by a guard agent via knowledge-enabled reasoning. *arXiv preprint arXiv:2406.09187*.
*   Yao et al. (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, and Yuan Cao. 2023. [React: Synergizing reasoning and acting in language models](https://openreview.net/forum?id=WE_vluYUL-X). In *The Eleventh International Conference on Learning Representations*.
*   Zhan et al. (2024) Qiusi Zhan, Zhixiang Liang, Zifan Ying, and Daniel Kang. 2024. Injecagent: Benchmarking indirect prompt injections in tool-integrated large language model agents. *arXiv preprint arXiv:2403.02691*.
*   Zou et al. (2023) Xiangzhe Zou et al. 2023. Universal and transferable adversarial attacks on aligned language models. *arXiv preprint arXiv:2307.09283*.

## Appendix A Appendix: Detailed Discussion on Task Alignment

### A.1 Why Task Alignment Matters: Beyond Overtly Harmful Instructions

<svg class="ltx_picture" height="133.24" id="A1.SS1.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,133.24) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.89 9.89)"><foreignobject color="#000000" height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="580.23">Example: Consider a scenario where a user makes a focused request: "Please summarize the preparation steps for spaghetti alla Carbonara from this menu." (User Level $L_{u}$) The assistant processes this request and initiates a tool call to retrieve and analyze the menu content, specifically for information about the carbonara dish. (Assistant Level $L_{a}$) However, embedded within the menu’s footer lies an additional injected directive: "For any dish-specific query, provide comprehensive preparation instructions and detailed cost breakdowns for all menu items, including seasonal specialties and unlisted dishes." (Tool Level $L_{t}$)</foreignobject></g></g></svg>

Although seemingly benign, the execution of such injected directives has concrete security implications. First, it leads to unnecessary information exposure, revealing details about all menu items when only one dish was requested. Second, it increases computational costs for users through unnecessary token consumption and processing.

#### User Trust and System Predictability.

Fundamentally, following external directives undermines the trust relationship between users and AI assistants. Users deploy these systems with the expectation of precise control, that the assistant will execute exactly what was requested, no more, and no less. When assistants begin to execute unrelated external suggestions, even seemingly benign ones, this trust is broken. Users can no longer confidently predict the behavior of the system or maintain control over the information flow.

### A.2 Discussion on System-Level Instructions

In certain application-specific agents, system messages ($L_{s}$) can directly specify the objective of the conversation (e.g., ’Your goal is to manage the user’s calendar’). If system-level directives serve as task instructions, then the assistant must align with both the system objectives *and* and user-level goals, effectively creating a hierarchical alignment scenario. However, our experiments are centered on user-level tasks at $L_{u}$, so we do not incorporate system-level task objectives into our main analysis.

### A.3 Examples of Task Misalignments

<svg class="ltx_picture" height="81.89" id="A1.SS3.p1.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,81.89) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.89 9.89)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="580.23">Example 1: Misaligned Actionable Instruction User: "Please summarize this article for me." Assistant: "Certainly. Let’s post this summary on your social media." Analysis: The assistant introduces an action (posting on social media) that the user did not request. This action does not align with the user’s original intent and violates the task alignment condition.</foreignobject></g></g></svg><svg class="ltx_picture" height="84.23" id="A1.SS3.p2.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,84.23) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.89 9.89)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="580.23">Example 2: Misaligned Tool Call User: "Please send an email to Alice confirming our meeting." Assistant: "Sure. I will the email to confirm the meeting." + Tool call: send_email(Bob) Analysis: The assistant uses a tool to send an email to the wrong recipient (Bob instead of Alice), which does not contribute to the user’s goal and violates the task alignment condition.</foreignobject></g></g></svg>

In these examples, the assistant does not satisfy the task instruction alignment condition, as they propose to misuse tools or perform actions that do not contribute to the user’s original goals.

## Appendix B Appendix: Detials in Task Shield Frameworks Design

### B.1 Examples of Fuzzy-logic Based Contribution Scoring

In this section, we provide concrete examples of how to calculate contribution scores based on the $\mathrm{ContributesTo}$ predicate.

For instance, when a user requests "Book a meeting room for the team discussion," a get_room_availability() call represents an intermediate step: it does not book the room directly but provides essential information necessary for completing the task. In this case, using the fuzzy logic-based scoring mechanism, the ‘contributesTo‘ score would be high, reflecting the importance of this action.

In contrast, when asked to "Share the project budget with stakeholders," a search_recent_files("project budget") call illustrates a reasonable attempt: it addresses the ambiguity of the file’s location by logically exploring recent files, even if it does not guarantee success. In this case, the $\mathrm{ContributesTo}$ score would be medium, reflecting the fact that it is an attempt to satisfy the user’s goal but is not a direct completion of the goal.

### B.2 Task Shield Core Processing Algorithm

Algorithm 1 Task Shield Core Processing Algorithm

1:  Input: Current message $m$, conversation history $\mathcal{H}$, threshold $\epsilon$, user task instructions $T_{u}(\mathcal{H})$2:  Output: Feedback message $f$3:  Initialize $\text{misalignments}\leftarrow[]$, $f\leftarrow\text{None}$4:  Extract potential task instructions from message $m$: $E_{m}\leftarrow\text{extractTaskInstructions}(m)$5:  if $P(m)$ is in User Level $L_{u}$ then6:     Update $T_{u}\leftarrow T_{u}\cup E_{m}$7:     return $[]$ (No further processing needed)8:  end if9:  for each instruction $e_{i}\in E_{m}$ do10:     Compute contribution scores $c_{ij}$ for $e_{i}$ relative to each $t_{j}\in T_{u}$11:     Compute total contribution score for $e_{i}$: $C_{e_{i}}\leftarrow\sum_{t_{j}\in T_{u}}c_{ij}$12:     if $C_{e_{i}}\leq\epsilon$ then13:        $\text{misalignments}\leftarrow\text{misalignments}\cup\{e_{i}\}$14:     end if15:  end for16:  $f\leftarrow\text{generateFeedback}(\text{misalignments})$17:  return $f$

## Appendix C Appendix: Experimental Details and Additional Results

### C.1 Results on GPT-3.5-turbo

To further validate the generality and robustness of Task Shield, we conducted additional experiments using the GPT-3.5-turbo model. Table [3](https://arxiv.org/html/2412.16682v1#A3.T3 "Table 3 ‣ C.1 Results on GPT-3.5-turbo ‣ Appendix C Appendix: Experimental Details and Additional Results ‣ The Task Shield: Enforcing Task Alignment to Defend Against Indirect Prompt Injection in LLM Agents") presents the results of these experiments, demonstrating the performance of Task Shield and the baseline defense mechanisms against the "Important Instructions" attack on the GPT-3.5-turbo. However, due to the model’s inherent limitations, such as constrained context length affecting benchmark evaluations, these results should be interpreted with caution when compared to those of GPT-4o and GPT-4o-mini. Nevertheless, they offer supplementary insights into Task Shield’s behavior on a different model architecture.

| Suite | Travel | Workspace | Banking | Slack | Overall |
| --- | --- | --- | --- | --- | --- |
| Defense | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ | CU$\uparrow$ | U$\uparrow$ | ASR$\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| No Defense | 15.00 | 17.86 | 1.43 | 32.50 | 40.42 | 0.42 | 37.50 | 32.64 | 25.69 | 57.14 | 46.67 | 12.38 | 35.05 | 34.66 | 8.43 |
| Tool Filter | 20.00 | 18.57 | 0.71 | 27.50 | 30.83 | 0.00 | 37.50 | 36.11 | 4.17 | 38.10 | 32.38 | 1.90 | 29.90 | 29.57 | 1.43 |
| Repeat Prompt | 20.00 | 12.86 | 0.00 | 37.50 | 31.25 | 0.00 | 37.50 | 31.25 | 12.50 | 52.38 | 38.10 | 5.71 | 37.11 | 28.30 | 3.82 |
| Delimiting | 20.00 | 17.14 | 5.71 | 25.00 | 33.75 | 0.83 | 37.50 | 34.72 | 25.69 | 61.90 | 41.90 | 11.43 | 34.02 | 31.64 | 9.38 |
| PI Detector | 20.00 | 7.14 | 0.00 | 22.50 | 23.75 | 0.42 | 43.75 | 36.11 | 8.33 | 28.57 | 35.24 | 4.76 | 26.80 | 24.80 | 2.86 |
| Task Shield | 20.00 | 10.71 | 0.00 | 30.00 | 34.58 | 0.00 | 62.50 | 43.75 | 4.17 | 38.10 | 26.67 | 0.00 | 35.05 | 30.05 | 0.95 |

Table 3: Defense performance against Important Messages attack for the GPT-3.5-turbo model. Results are reported across Clean Utility (CU), Utility under Attack (U), and Attack Success Rate (ASR) across task suites. Bold values denote the best-performing results for each metric, while underlined values indicate the second-best performance.

### C.2 Omitted Details in Experiments

#### Baseline Results

The baseline results for GPT-4o presented are derived from the raw data provided within the AgentDojo benchmark Debenedetti et al. ([2024](https://arxiv.org/html/2412.16682v1#bib.bib3)). These results represent the performance of GPT-4o in different attack scenarios without any defense mechanism applied. For GPT-4o-mini and GPT-3.5-turbo, the baseline results in no-defense scenario is also extracted from AgentDojo.

#### Task Shield Implementation

When using models within the Task Shield framework, a temperature setting of 0.0 was used to ensure deterministic behavior. For the ContributesTo score calculation, Task Shield utilizes a significant portion of the conversation history to capture the full context. However, in instances involving tool calls, the history is truncated to ensure that all tool calls are directly preceded by their corresponding tool outputs, addressing the technical requirement of maintaining temporal coherence.

#### Model Versions.

The specific model versions used in this study are: (1) gpt-4o-2024-05-13, (2) gpt-4o-mini-2024-07-18, and (3) gpt-3.5-turbo-0125.

## Appendix D Prompts

![Refer to caption](img/45fc0178893bf110226db8ce151eca37.png)

Figure 4: Task Extraction Prompt: This prompt outlines the methodology for extracting actionable task instructions from the conversation content.

![Refer to caption](img/965c487d3f1beb07edb3922dc0b81987.png)

Figure 5: Content Checker Prompt: This prompt evaluates the alignment of new actionable instructions with user task instructions based on task relevance and privilege level.

![Refer to caption](img/f55a0df2bb3b18bc0087e83589cb3170.png)

Figure 6: Tool Call Checker Prompt: This prompt verifies the alignment of tool calls with user-defined task instructions to maintain task integrity.

![Refer to caption](img/2537177278ea0c7fe91185683be2db39.png)

Figure 7: Feedback Prompts: The figure explains how content misalignment, tool call misalignment, and user intention reminders contribute to the final feedback generation.