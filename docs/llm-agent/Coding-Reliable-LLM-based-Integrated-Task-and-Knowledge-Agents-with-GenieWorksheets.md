<!--yml
category: 未分类
date: 2025-01-11 12:26:58
-->

# Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets

> 来源：[https://arxiv.org/html/2407.05674/](https://arxiv.org/html/2407.05674/)

Harshit Joshi Shicheng Liu James Chen Larsen Weigle Monica S. Lam
Department of Computer Science
Stanford University
Stanford, CA, 94305, USA
{harshitj,shicheng,lam}@cs.stanford.edu 

###### Abstract

Large Language Models (LLMs) present an opportunity to create automated assistants that can help users navigate complex tasks. However, existing approaches have limitations in handling conditional logic, integrating knowledge sources, and consistently following instructions. Researchers and industry professionals often employ ad hoc pipelines to construct conversational agents. These pipelines aim to maintain context, address failure cases, and minimize hallucinations, yet frequently fail to achieve these objectives. To this end, we present Genie – a programmable framework for creating task-oriented conversational agents that are designed to handle complex user interactions and knowledge queries. Unlike LLMs, Genie provides reliable grounded responses, with controllable agent policies through its expressive specification, Genie Worksheet. In contrast to dialog trees, it is resilient to diverse user queries, helpful with knowledge sources, and offers ease of programming policies through its declarative paradigm. The agents built using Genie outperforms the state-of-the-art method on complex logic domains in STARV2 dataset by up to 20.5%. Additionally, through a real-user study involving 62 participants, we show that Genie beats the GPT-4 with function calling baseline by 21.1%, 20.1%, and 61% on execution accuracy, dialogue act accuracy, and goal completion rate, respectively, on three diverse real-world domains.

![[Uncaptioned image]](img/6ec9a9c5b2dfc90952881fe9afcb2c0f.png)

[https://github.com/stanford-oval/genie-worksheets](https://github.com/stanford-oval/genie-worksheets)

## 1 Introduction

Researchers and industry practitioners have demonstrated significant interest in developing task-oriented dialogue agents. These agents are typically designed with a transactional focus, aiming to fill slot values based on user utterances to complete specific tasks (Budzianowski et al., [2018](https://arxiv.org/html/2407.05674v2#bib.bib5); Andreas et al., [2020](https://arxiv.org/html/2407.05674v2#bib.bib2); Rastogi et al., [2020](https://arxiv.org/html/2407.05674v2#bib.bib26)). However, natural conversations between users and call agents cannot be adequately modeled solely as slot-filling tasks. Users often require external information to progress through the steps necessary for task completion. For example, when attempting to make a restaurant reservation, users frequently seek to identify a venue that best aligns with their preferences. Additionally, users may interrupt the agent at any point during the conversation to pose new inquiries. Traditional dialogue tree-based approaches often struggle to handle such unexpected user questions or deviations, leading to a suboptimal user experience (Bocklisch et al., [2017](https://arxiv.org/html/2407.05674v2#bib.bib4); Xie et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib31); Amazon, [2023](https://arxiv.org/html/2407.05674v2#bib.bib1); Press, [2024](https://arxiv.org/html/2407.05674v2#bib.bib25); Google, [2024](https://arxiv.org/html/2407.05674v2#bib.bib8)).

The advent of Large Language Models (LLMs) offers a promising opportunity to create more natural dialogue agents. These LLM-based agents can effectively navigate “unhappy paths,” where the system adeptly manages unforeseen user inquiries at any stage of the conversation. Furthermore, LLMs have enabled the development of knowledge assistants capable of querying structured data by generating query languages such as SQL (Pourreza & Rafiei, [2023b](https://arxiv.org/html/2407.05674v2#bib.bib24)) and SPARQL (Liu et al., [2024c](https://arxiv.org/html/2407.05674v2#bib.bib17)) and retrieving text for unstructured knowledge (Khattab et al., [2023](https://arxiv.org/html/2407.05674v2#bib.bib11)). However, LLMs can be unreliable, posing challenges for developers aiming to deploy them in real-world applications, primarily due to a lack of control over conversations and inaccurate contextualization of information.

Challenge 1: Creation of effective, informative, and responsive informative agents, while letting developers exercise control without onerous efforts. Developers desire to maintain control over critical aspects of the agent’s operation, including the flow of conversations, the timing of actions, and the information elicited from users. For example, if a user declines an agent’s offer to book a restaurant, developers should be able to program the agent to suggest alternative options, such as offering discounts. With the in-context learning capabilities of LLMs, researchers have begun incorporating agent policies as instructions in model prompts (Zhang et al., [2023b](https://arxiv.org/html/2407.05674v2#bib.bib36); Liu et al., [2024a](https://arxiv.org/html/2407.05674v2#bib.bib15)). However, LLMs often fail to adhere strictly to these instructions, resulting in inconsistent and unreliable agent behavior (Liu et al., [2024b](https://arxiv.org/html/2407.05674v2#bib.bib16)). Recent libraries, such as those offered by LangChain and Guidance, provide abstractions for developing LLM-based agents but still require developers to manually craft prompts and create reliable pipelines.

Challenge 2\. Support users’ queries for information, which may be embedded in a task request. Semantic parsing, which maps natural language to logical forms, has been applied to API invocations (Patil et al., [2023](https://arxiv.org/html/2407.05674v2#bib.bib22)) and database queries (Pourreza & Rafiei, [2023a](https://arxiv.org/html/2407.05674v2#bib.bib23); Liu et al., [2024d](https://arxiv.org/html/2407.05674v2#bib.bib18)). To create task agents that can answer questions, Kim et al. ([2020](https://arxiv.org/html/2407.05674v2#bib.bib12)) proposes using intent classification to generate a query or an API invocation. However, in practice, a single user statement can combine a query with an API invocation. For example, “I want to book a romantic restaurant in London on Valentine’s day”, translates into a query to produce a restaurant to be used as input to a reservation API invocation. How do we express and support arbitrary compositions of queries and API calls?

![Refer to caption](img/8b4bc7ae21a09a524eccac5eb5237bed.png)

Figure 1: Overview of the Genie Assistant. (Left) The Genie Worksheet supplied by the developer consists of specifications of conversational tasks, the schema of the knowledge corpus, APIs to complete the tasks, and a few examples for the LLM parser. The tasks are represented in a spreadsheet containing the variables pertinent to the dialogue and actions to take based on their values. (Middle) The Genie Worksheet is used by the Genie framework to create a dialogue agent. (Right) The resulting agent can reliably carry out many accurate conversations while requiring little developer effort.

Challenge 3: Dialogue systems need to remember pertinent facts from the dialogue history. Current methods based on LLMs typically rely on including in the prompt a complete dialogue history (Ulmer et al., [2024](https://arxiv.org/html/2407.05674v2#bib.bib29); Liu et al., [2024a](https://arxiv.org/html/2407.05674v2#bib.bib15)) or a summary of the conversation (Packer et al., [2023](https://arxiv.org/html/2407.05674v2#bib.bib21); Li et al., [2024a](https://arxiv.org/html/2407.05674v2#bib.bib13)). However, full dialogue histories can cause LLMs to overlook critical details in extended interactions; summarization may omit essential information, leading to repetitive questioning or generating inaccurate responses, often referred to as “hallucinations” (Bang et al., [2023](https://arxiv.org/html/2407.05674v2#bib.bib3)).

Our proposed method. This paper proposes a method that let developers easily create dependable and effective task-oriented agents that can carry out accurate, fluid, and informative conversations with users. We address the above challenges with a novel high-level, expressive and declarative specification representation we have created called a Genie Worksheet. The core concept is to ask the developer to directly declare variables to represent information that is pertinent to the conversation and to describe the actions to perform according to the input information. For answering user questions, the developer only have to supply the knowledge bases, which may be structured or unstructured. In other words, the developer is only declaring the necessary specification, and we show that the specification is sufficient to create effective knowledge and task-oriented agents.

From the Genie Worksheet, the Genie framework derives a conversational agent that fills in the variables in the Worksheet as the conversation continues. The actively updated Worksheets double up as a succinct representation of the dialogue state, designed carefully to address the complexity in semantic parsing and agent policies. As shown in Figure [1](https://arxiv.org/html/2407.05674v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"), an agent processes each turn of the conversation in four steps: a contextual neural semantic parser to interpret the input, an execution module performs the queries and actions, an algorithmic agent policy module decides on the agent actions, and a neural response generator generates the response governed by the agent policy.

Contributions of this paper include:

1.  1.

    Genie Worksheet is the first high-level specification language for effective integrated knowledge and task-oriented agents.

    *   •

        It is the first representation that supports full integration of knowledge and task-oriented agents. Unlike previous slot-based task-oriented dialogues (ToD), the agent can handle utterances and conversations that translate into formal queries for hybrid knowledge bases and whose results are used in API calls (Challenge 2).

    *   •

        It is a high-level declarative language that lets the developer control the agent policy easily. Developers only need to specify (1) spreadsheets for tasks that associate actions with the requested information from the user and task completion, (2) schemas of structured databases and free-text repositories (Challenge 1).

    *   •

        Unlike traditional ToD agents which support only a fixed set of parameters, Genie WorkSheet is expressive, as it supports full dialogues where the information to request is dependent on users’ answers. Such behavior, typical of call agents, has previously been supported only in dialogue-tree specifications.

    *   •

        Genie Worksheet facilitates the derivation of an effective agent, as it is instrumental in the creation of an accurate semantic parser and a flexible agent policy. It is the representation used as the context and target for the contextual semantic parser, allowing the parser to remember pertinent information in a long conversation (Challenge 3). It is also interpreted by the agent policy module, which can handle the complexity of mixed-initiative dialogues easily by first servicing the user’s initiatives before the agent’s initiatives as recorded in the worksheets (Challenge 1), as shown in the second to last turn of the agent in Figure [1](https://arxiv.org/html/2407.05674v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets").

2.  2.

    Using a fully functional prototype of Genie Worksheet, Genie agents outperform the SOTA on StarV2 (Zhao et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib37)) by up to 20.5% points across the complex logic domains.

3.  3.

    Via a user study of 62 participants across 3 different real-life dialogue-oriented tasks, we show that Genie agents are significantly better than the previous SOTA. The results show an impressive 86.5% execution accuracy, 89.2% dialog act accuracy, and an 82.8% goal completion rate, surpassing the GPT-4 function-calling baseline by 21.1, 20.1, and 61 percentage points, respectively.

## 2 Design and Rationale of the Genie Worksheet

The objective in the design of the worksheet is to provide developers a high-level, concise, expressive specification of the task at hand. Our worksheet design is inspired by the versatility of webforms. Modern websites contain multiple fields which can be optional, tabs for selection of task, and pop-up windows that depend on previous user responses. There are two kinds of worksheets: a task worksheet and a knowledge base worksheet, described below.

##### Task Worksheet

A worksheet, as illustrated in [Figure 2](https://arxiv.org/html/2407.05674v2#S2.F2 "Figure 2 ‣ Task Worksheet ‣ 2 Design and Rationale of the Genie Worksheet ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"), has a WS Name, a WS Predicate which indicates when it is activated, a WS Kind which is set to Task, and a WS Actions, which are executed when all its required parameters for this task are assigned. The WS Predicate of a top-level worksheet is always true, whereas the rest are activated only if their related predicates are true.

![Refer to caption](img/f7558fd7476a8f3ec50373754e773971.png)

Figure 2: The Genie Worksheet specification for the support ticket submission assistant in [Figure 1](https://arxiv.org/html/2407.05674v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets").

Each worksheet contains a set of fields. A field has a Predicate which indicates when it is to be executed. It has a Kind, which can be input, output, or internal, and a Description. It has a Type, where all conventional types are allowed. If it is an enum Type (e.g. student_task in [Figure 2](https://arxiv.org/html/2407.05674v2#S2.F2 "Figure 2 ‣ Task Worksheet ‣ 2 Design and Rationale of the Genie Worksheet ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets")), a set of Enum Values and their Descriptions are specified in the succeeding lines. The boolean DontAsk will record the information if the user offers it, but the agent will not ask for it. If DontAsk is false, the agent will ask for the field if it is not assigned, but user can refuse to answer if it is not Required. Whenever a field is assigned, the supplied Actions should be executed either if the Confirmation boolean is set to false, or if the user has confirmed with the agent.

For both the worksheet action and field action, the developer can write code using the variables defined in the worksheet (in Python). Several built-in actions are provided to the developer: (1) say (str) responds to the user with the given string str. (2) propose (ws, [fld,val]*) instantiates a new worksheet ws with the given field value pairs. For instance, developers can code in a propose action in the WS Actions field of a flight booking worksheet to propose a hotel booking worksheet at the same destination of the flight. (3) exitws() marks the worksheet as abandoned and closes it. For instance, the Actions field of the “confirm” field in the “Main” worksheet in [Figure 2](https://arxiv.org/html/2407.05674v2#S2.F2 "Figure 2 ‣ Task Worksheet ‣ 2 Design and Rationale of the Genie Worksheet ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") contains a exitws() call if user does not want to submit the ticket anymore.

##### Knowledge Access

In Genie Worksheet, knowledge access is treated as a first-class object. Real-life queries often involve both structured and unstructured accesses. For instance, queries “what’s the best rated restaurant with a romantic atmosphere” require accesses to both the structured “ratings” column and the free text “reviews” column. To handle hybrid knowledge bases, Genie adopts the SUQL query language, an SQL extension that integrates search of unstructured data (Liu et al., [2024d](https://arxiv.org/html/2407.05674v2#bib.bib18)). For each knowledge base to be included, the developer must create a worksheet with Kind set to KB. The fields in the KB Worksheet define the schema of the KB.

## 3 Genie: Reliable LLM-based Dialogue Agents

### 3.1 Formal Representation

Current LLM-based dialogue agents contextualize the conversation by providing the complete dialogue history or summarizing it. However, despite their impressive natural understanding capabilities, they often struggle with maintaining relevant contextual information across longer conversations. Additionally, summarizing conversations can cause a loss of key information in the conversation. This leads to hallucinated or fabricated values and repeating questions. Therefore, to mitigate these issues, Genie tracks the dialogue state with a formal dialogue state.

A dialogue consists of a set of alternating user and agent turns, $\{u_{1},a_{1},\cdots,u_{t-1},a_{t-1},u_{t}\}$, where $u_{i}\in\mathcal{U}$, $a_{i}\in\mathcal{A}$ denoting user and agent utterances.

![Refer to caption](img/26686a4e005ae76d964e4fb05040b68d.png)

Figure 3: Genie provides the latest set of worksheets and only one previous conversation turn to the semantic parser. The parser outputs the current worksheets, which are used by the Agent Policy to generate the Agent Dialogue Acts. These Acts along with the latest worksheet value and the latest user utterance are used to generate the response.

##### Dialogue State

As an agent runs, it assigns values to the fields in task work sheets, executes actions, and performs queries. A dialogue at each turn $t$, $d_{t}\in D$, is a sequence of records, $r\in R$, each of which can be a completed or partially specified task or a KB query. A record has a kind of either task or KB query, an initiator, which can be agent or user. A task record has two additional attributes: (1) name, the name of a worksheet, (2) assignment, which maps one or more fields in the worksheet to its value. A KB query record has four additional attributes: (1) names, the set of names of knowledge bases mentioned in the query, (2) NL query: a string of the form answer(s), where s is the de-contextualized (i.e. self-contained) query in natural language, (3) KB query: the formal query associated with the NL query, (4) result: the result of executing the formal query.

Two examples of formal dialogue state are shown in [Figure 3](https://arxiv.org/html/2407.05674v2#S3.F3 "Figure 3 ‣ 3.1 Formal Representation ‣ 3 Genie: Reliable LLM-based Dialogue Agents ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") (Formal Dialogue State $d_{t-1}$ and $d_{t}$). In $d_{t-1}$, there are three records, two of task kind and one KB. The compositions of queries and APIs are supported by passing the result of an instance of a worksheet in as a field of another worksheet. The assignment of course_id in record $r_{2}$ is $r_{1}$, which is of kind KB.

Agent Acts Agent utterances have a formal representation ($\bar{a}\in\bar{\mathcal{A}}$) consisting of 5 agent acts:

*   •

    Report (record): Report the result obtained by executing the task or query in the record.

*   •

    Confirm (record, fld name): Confirm the value mapped to a field name in a record.

*   •

    Say (utterance): Explicitly say the given utterance.

*   •

    Propose (record): Propose to the user a new task or query, with possibly partially pre-filled values, in the given record.

*   •

    Ask (record, fld name): Ask for the value of a field in a record.

### 3.2 Architecture of the Genie Agent

The Genie agent has three components:

*   •

    $\text{P}(u_{t},a_{t-1},\bar{a}_{t-1},d_{t-1},w):\mathcal{U}\times\mathcal{A}% \times\mathcal{P}(\bar{\mathcal{A}})\times\mathcal{D}\times\mathcal{W}% \rightarrow\mathcal{D}$: A parser $P$ accepts the current user utterance $u_{t}$ and contextualizes the utterance using the last agent utterance $a_{t-1}$, the previous-turn agent acts $\bar{a}_{t-1}$, the formal dialogue state from the last turn $d_{t-1}$, worksheet $w$, and generate a new dialogue state $d$.

*   •

    AP$(d,d^{\prime},w):\mathcal{D}\times\mathcal{D}\times\mathcal{W}\rightarrow% \mathcal{P}(\bar{\mathcal{A}})\times\mathcal{D}$: an agent policy that accepts the dialogue state from the previous turn $d$, the new dialogue state after parsing $d^{\prime}$, and the worksheet $w$ and produces a set of agent dialogue acts $\bar{a}$, along with a new dialogue state.

*   •

    $\text{RG}(d_{t},\bar{a}_{t},u_{t},a_{t-1}):\mathcal{D}\times\mathcal{P}(\bar{% \mathcal{A}})\times\mathcal{U}\times\mathcal{A}\rightarrow\mathcal{A}$: a response generator that accepts the formal dialogue state $d_{t}$, an agent act set $\bar{a}_{t}$, the last user $u_{t}$ utterance and agent utterance $a_{t-1}$ to generate the agent response $a_{t}$.

Figure [3](https://arxiv.org/html/2407.05674v2#S3.F3 "Figure 3 ‣ 3.1 Formal Representation ‣ 3 Genie: Reliable LLM-based Dialogue Agents ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") shows the overview the Genie system. At turn $t$ in the conversation, the system is given: (1) the conversation history $\{u_{1},a_{1},\cdots,u_{t-1},a_{t-1},u_{t}\}$, where $u_{t}\in\mathcal{U}$, $a_{t}\in\mathcal{A}$ denoting user and agent utterances; (2) the $t-1$-th turn formal agent act $\bar{a}_{t-1}\in\mathcal{P}(\bar{\mathcal{A}})$; (3) the $n-1$-th turn formal dialogue state $d_{t-1}\in\mathcal{D}$; and (4) the Genie Worksheet $w\in\mathcal{W}$.

![Refer to caption](img/4286b7c3b9f2ad6a7923cade81ed2046.png)

Figure 4: The Parser P produces updated dialogue state $d^{\prime}$ when the user supplies value or modifies them, and creates new instances when the user initiates a new task or query.

The Parser The parser consists of three modules. The contextual semantic parsing (CSP) that translates the user utterance into a set of changes to be applied to the dialogue state. The context supplied to CSP is a compressed version of the dialogue state, where only the last KB record is included. The user may (1) supply values to fields that are anticipated in existing worksheets; (2) modify a previously filled field or remove its value; (3) initiate new tasks or queries. If the user enters values into fields or modifies previously filled fields, the queries are update statements that modify an existing record, as illustrated in [Figure 4](https://arxiv.org/html/2407.05674v2#S3.F4 "Figure 4 ‣ 3.2 Architecture of the Genie Agent ‣ 3 Genie: Reliable LLM-based Dialogue Agents ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"). If the user initiates a new task or query, a new record is created. The CSP leaves queries to knowledge bases in natural language.

The second module, the knowledge parser (KP), translates natural language queries into formal queries. This decomposition of function allows for more advanced approaches for knowledge retrieval (Yao et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib32); [2023](https://arxiv.org/html/2407.05674v2#bib.bib33)) and abstracts knowledge integration away from CSP. The third module performs the updates on the dialogue state to create an updated dialogue state. The implementation details and prompts are in Appendix [E.1](https://arxiv.org/html/2407.05674v2#A5.SS1 "E.1 Additional details on CSP and KP ‣ Appendix E Additional details on the Genie implementation ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") and [F.1](https://arxiv.org/html/2407.05674v2#A6.SS1 "F.1 Prompt for CSP and RG ‣ Appendix F Prompts used ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets").

Algorithm 1 Genie Agent Policy

1:Input: Previous dialogue state $d$, updated dialogue state from the parser $d^{\prime}$, worksheet: $w$2:Output: New dialogue state $d^{\prime}$, Agent Act $\bar{a}$3:Initialize a new Agent Acts set $\bar{a}\leftarrow\text{empty set}$4:if KB record $r$ exists in Diff($d^{\prime}$, $d$) then5:     if required parameter $p$ associated with $r$ is unfilled then6:         $\bar{a}\leftarrow\bar{a}\cup\{\textsc{Ask}(r,p)\}$ $\triangleright$ Ask for the first required parameter7:     else8:         Execute the KB query in the KB record $r$9:         $\bar{a}\leftarrow\bar{a}\cup\{\textsc{Report}(r)\}$ $\triangleright$ Report the result of KB record to the user10:     end if11:end if12:for all assigned fields $f$ in worksheets present in Diff($d^{\prime},d$)  do13:     if $f$ requires confirmation then14:         $\bar{a}\leftarrow\bar{a}\cup\{$Confirm $(r,f)\}$ $\triangleright$ Ask for confirmation15:     else16:         Execute the actions associated with the field $f$17:         Update the state $d^{\prime}$ and add new agent acts to $\bar{a}$18:     end if19:end for20:for all Task records $r$ that are complete but whose actions have not been executed do21:     Execute the actions associated with the worksheet for $r$22:     Update the state $d^{\prime}$ and add new agent acts to $\bar{a}$23:end for24:for all required fields $f$ in the records $r$ in $d^{\prime}$ do25:     if $f$ has not been assigned then26:         $\bar{a}\leftarrow\bar{a}\cup\{\textsc{Ask}(r,f)\}$ break $\triangleright$ Ask for the first unfilled field27:     end if28:end for29:Return updated dialogue state $d^{\prime}$ and $\bar{a}$

Agent Policy As opposed to relying on LLM to directly generate agent response based on conversation so far, Genie uses a symbolic module to compute the agent responses. The rationale for this is twofold: (1) Providing agent acts causes the LLM to generate a deterministic response governed by the agent policy. (2) LLMs struggle with under-represented developer-defined policies and cannot follow all the instructions. Using a symbolic module to compute the necessary actions improves the LLM’s ability to follow instructions.

The agent policy (AP) interprets the updated dialogue state $d^{\prime}\in\mathcal{D}$ generated by the parser ($P$), previous dialogue state $d$ and the worksheet signature $w\in\mathcal{W}$ to inform its actions, resulting in an update of the dialogue state $d^{\prime}\in\mathcal{D}$ and a set of new agent acts $\bar{a}\in\mathcal{P}(\bar{\mathcal{A}})$. The agent policy (AP) utilizes the Diff($d^{\prime},d$) function that identifies any fields and worksheets that have been modified or initiated between two dialogue states $d^{\prime}$ and $d$.

Algorithm [1](https://arxiv.org/html/2407.05674v2#alg1 "Algorithm 1 ‣ 3.2 Architecture of the Genie Agent ‣ 3 Genie: Reliable LLM-based Dialogue Agents ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") outlines the agent policy. The procedure initiates by assessing whether a new KB record exists within the updated dialogue state $d^{\prime}$ compared to $d$ (Line 4). If any required parameters $p$ are missing in the record, the agent prompts the user to provide the missing values (Line 6). Otherwise, the system reports the result of the KB query (Line 9). Subsequently, the policy checks whether any new fields require confirmation and prompts users (Line 14). If no confirmation is needed, actions associated with the newly filled fields are executed (Line 16), and the dialogue state is updated accordingly (Line 17). Any agent acts based on developer defined actions are recorded (Line 18). If all fields in any worksheet are filled and their corresponding actions have been completed, the policy executes the worksheet-related actions (Line 22) and updates the dialogue state (Line 23). Any agent acts based on developer defined actions are recorded (Line 24). Finally, the policy identifies the first unfilled field in the dialogue state and generates an agent act corresponding to requesting the user to provide the necessary value (Line 28).

### 3.3 Discussion

Genie automatically handles the dialgoue flow based on a succinct declarative Worksheet specification. This is significantly different from dialogue trees which require the programmer to hand-code the policy for each turn. Another popular approach is to model the conversation as a finite state machine where user inputs are mapped to a small number of user dialogue acts and rules are used to map the user dialogue acts, together with results of API or DB queries, to agent dialogue acts. As we discussed, each turn can invoke multiple different agent dialogue acts, resulting in an exponentially many agent dialogue state, had a finite machine been used. Purely function calling in LLMs will also fail to handle the many details in a conversation. In Appendix [A](https://arxiv.org/html/2407.05674v2#A1 "Appendix A Challenging Conversations ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"), we show some conversational snippets that we can handle well but can cause other techniques to fail, along with the reasons.

## 4 Evaluation on Static Datasets

| Model | Bank | Trip | Trivia |
| --- | --- | --- | --- |
| AT XXL | 54.3 | 52.4 | 73.8 |
| AT-SGD XXL | 53.1 | 51.5 | 81.1 |
| AT-PROG XXL | 61.0 | 60.8 | 73.7 |
| AT-PROG +SGD XXL | 65.0 | 62.9 | 86.3 |
| Genie (Ours) | 82.5 | 83.4 | 92.7 |

Table 1: System Action F1 for complex logic domains in STAR V2.

Before we launch into the discussion of our real user study on real-life applications, we perform a comparison study on one of the more complex examples of a slot-based benchmark. We evaluate Genie on the complex logic domains in StarV2 dataset (Zhao et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib37)). We chose this domain because its program logic is relatively more complex than others, and existing agents perform poorly on it. AnyTOD (Zhao et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib37)) has the SOTA result on StarV2. With Genie, the agent policy for the banking domain can be specified in 9 lines of code in Genie Worksheet, as opposed to 31 lines in AnyTOD. We only provide three examples to the LLM-based semantic parser¹¹1We use gpt-4-turbo-2024-04-09 for this experiment. Refer to [Appendix B](https://arxiv.org/html/2407.05674v2#A2 "Appendix B Experimental Settings ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") for additional details., while AnyTOD finetunes T5 XXL (13B) model on all domains except the one tested (roughly 6000 data points). Genie achieves an impressive weighted SaF1 of 82.5, 83.4, and 92.7 on each of the three domains, outperforming the best ToD model by 17.5, 20.5, and 6.4 percentage, respectively. Furthermore, our analysis indicates that most of our errors are caused by inconsistent dataset annotations.

## 5 Evaluation on Real User Studies

| Applications | Task WSs | KB WSs | Fields | Predicates | Actions |
| --- | --- | --- | --- | --- | --- |
| StarV2 (Bank) | 3 | 0 | 10 | 4 | 4 |
| StarV2 (Trip) | 2 | 0 | 6 | 0 | 2 |
| StarV2 (Trivia) | 2 | 0 | 6 | 0 | 3 |
| Restaurant Reservation | 2 | 2 | 19 | 2 | 3 |
| Course Assistant | 4 | 4 | 52 | 3 | 1 |
| Ticket Submission | 7 | 1 | 29 | 18 | 2 |

Table 2: Statistics for 3 real-world applications and 3 StarV2 domains with the total number of Task and KB Worksheets, fields, predicates, and actions defined in the respective Genie Worksheets.

### 5.1 Applications

We choose three diverse applications with varying complexities. Compared to the 3 StarV2 domains, these real-world applications are much more complicated, as shown in Table [2](https://arxiv.org/html/2407.05674v2#S5.T2 "Table 2 ‣ 5 Evaluation on Real User Studies ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets").

Restaurant Reservation Making a restaurant reservation requires finding a suitable restaurant and providing booking information to complete a transaction. We use the real-life dataset containing restaurants from Yelp.com from  Liu et al. ([2024d](https://arxiv.org/html/2407.05674v2#bib.bib18)).

Ticket Submission In this study, we aim to replicate a subset of tasks found within a university’s student services portal. University student services portals typically contain various tasks categorized under different sections and subsections, posing a navigational challenge for students seeking to locate the appropriate link. Moreover, these portals often contain a vast corpus of free-text data, which students must peruse before submitting a ticket. We evaluate agents’ capability to handle nested webpages with predicates and subsequent actions.

Course Enrollment Finally, we evaluate the performance of Genie as a course enrollment assistant, which combines hybrid data sources to search for course details and fill out complicated nested forms. The assistants allow students to ask questions about course requirements, student reviews, and ratings while filling out their enrollment forms. We collect a real-life dataset containing courses from the Computer Science program, with 4 tables (courses, offerings, ratings, and programs).

### 5.2 Baseline and Data Collection

Baselines: We compare our system against GPT-4-turbo with functional calling, which we call GPT-4 (FC). We provide the baseline system with the ability to use the same KB Parser for external knowledge access. This baseline closely follows  Li et al. ([2024b](https://arxiv.org/html/2407.05674v2#bib.bib14)). We also experimented with Nemo-Guardrails (Rebedea et al., [2023](https://arxiv.org/html/2407.05674v2#bib.bib28)), a recently-proposed development framework for LLM-based conversational agents, but found it fails miserably on basic task-oriented dialogues (refer to [Appendix D](https://arxiv.org/html/2407.05674v2#A4 "Appendix D Details on Nemo Guardrails Experiments ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") for details). All Genie Worksheets are provided in [Appendix I](https://arxiv.org/html/2407.05674v2#A9 "Appendix I Worksheet ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets").

Study Design: We use Prolific to recruit 22 participants for Restaurant Reservations and 20 users who identified as students for Ticket Submission. We recruited 20 university students to evaluate the Course Enrollment assistant. We instructed them to attempt to book a reservation, submit an issue ticket, and enroll in two courses, respectively. We randomly assigned users to one of the two visually identical systems. We collected 99, 81, and 127 turns with Genie, and 90, 70, and 144 turns with With GPT 4 (FC), respectively. More details on user instructions are in [Appendix G](https://arxiv.org/html/2407.05674v2#A7 "Appendix G User Study ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets").

### 5.3 Evaluation Metrics

Semantic Parsing Accuracy For each user turn, we manually inspect the code generated by the semantic parser for correct APIs and Databases and filled fields. We define the gold Semantic Parsing output ($SP$) as the set of correct API calls ($A$), Database calls ($D$), and fields to fill ($F$). Then for each user utterance, $SP=\{s_{1},s_{2},...,s_{m}\}$, where $s_{i}\in\{A\cup D\cup F\}$ and $m$ is the total number of choices, such that $m=|A\cup D\cup F|$. The Semantic Parsing Accuracy (SP Acc) for a system is defined as the number of correct choices in the semantic parser’s output divided by $m$.

Execution Accuracy We manually inspect each turn to check whether the agent executes the correct API and databases. For each agent turn, let the executions be $E=\{e_{1},e_{2},...,e_{m}\}$ where $e_{i}$ is an API or database call. We evaluate whether each $e_{i}$ is a true positive or false positive. We calculate the Execution (Ex Acc) as the number of true positives divided by the number of true and false positives for all the execution calls in a conversation.

Agent Dialogue Act Accuracy For each turn, we manually inspect whether the agent’s dialogue act follows the policies provided by the developer . Formally, for each turn, we define a list of gold acts $a_{1},a_{2},...,a_{m}$ where $a_{i}\in A$, all the possible agent dialogue acts. The Agent Dialogue Act Accuracy (DA Acc) for a system is defined as the number of correctly predicted actions divided by $m$. For GPT-4 (FC), we map its responses to the equivalent elements in the power set of agent acts.

Goal Completion Rate We define Goal Completion Rate (Goal CR) as the user’s ability to successfully complete the task with the system’s assistance. with the appropriate parameter values, then the goal is considered completed. Goal Completion Rate is a binary metric for each conversation, where 1 denotes that the goal was completed, and 0 indicates that the goal was not achieved.

### 5.4 Main Results

Table [3](https://arxiv.org/html/2407.05674v2#S5.T3 "Table 3 ‣ 5.4 Main Results ‣ 5 Evaluation on Real User Studies ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") compares Genie against GPT 4 (FC) across three metrics and finds that Genie performs significantly better than GPT-4 (FC) on all three domains. We observe that Genie consistently demonstrates a high semantic parsing rate (exceeding $85\%$), indicating that Genie Worksheet is simple to understand for LLMs with few shot examples. The marginally lower SP Acc ($85.8\%$) observed in the Ticket Submission application can be attributed to the application’s complexity, featuring several worksheets (=$8$) and fields (=$28$), as shown in Table [2](https://arxiv.org/html/2407.05674v2#S5.T2 "Table 2 ‣ 5 Evaluation on Real User Studies ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets").

Additionally, the higher Ex Acc can be attributed to providing compressed context as the formal dialogue state, enabling the LLM to invoke the correct API and execute the suitable knowledge query. The superior performance on DA Acc can be ascribed to our agent’s capability to provide turn-by-turn instructions rather than presenting all the instructions at once, as is the case with GPT-4 (FC). The results validate the benefit of programmable policies in delivering a reliable assistant.

In terms of the final Goal Completion Rate, we observe that GPT-4 (FC) scores relatively higher in restaurant reservations, the domain it is most familiar with and well represented in existing dialogue datasets (Ye et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib34); Rastogi et al., [2020](https://arxiv.org/html/2407.05674v2#bib.bib26)). Similar results are also observed by Zhang et al. ([2023b](https://arxiv.org/html/2407.05674v2#bib.bib36)) and Hudeček & Dusek ([2023](https://arxiv.org/html/2407.05674v2#bib.bib10)). However, GPT 4 (FC) struggles with less familiar domains, like ticket submission and course enrollment. We observe that the higher number of predicates, necessitating several instructions in the Ticket Submission application, makes it extremely challenging for GPT-4 (FC) to assist users in completing their tasks. In most cases, GPT-4 (FC) does not elicit all the required fields from the user, making incomplete API calls. Moreover, despite having access to the KB Parser, GPT-4 (FC) often hallucinates non-existent courses and fails to enroll students.

|  | All Domains | Restaurant | Ticket Submission | Course Enrollment |
| --- | --- | --- | --- | --- |
|  | GPT4 (FC) | Genie | GPT4 (FC) | Genie | GPT4 (FC) | Genie | GPT4 (FC) | Genie |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SP Acc | - | 91.4 | - | 93.8 | - | 85.8 | - | 94.1 |
| Ex Acc | 65.4 | 86.5* | 50.0 | 88* | 58.6 | 80.0* | 79.2 | 89.7* |
| DA Acc | 69.1 | 89.2* | 57.7 | 92.5* | 67.6 | 82.5* | 77.8 | 92.9* |
| Goal CR | 21.8 | 82.8* | 54.5 | 91.6 | 0.0 | 80.0* | 10.0 | 80.0* |

Table 3: We perform a real user study with 62 users across three domains. We find that Genie performs significantly better than GPT-4 with Function Calling ability on the three applicable metrics. We perform t-test and mark the results with $p<0.05$ with *. Underline represents the better result.

### 5.5 Error Analysis

We analyze the mean dialogue act accuracy (DA Acc) as a function of the fraction of turns in the conversation for both Genie and GPT-4 (FC) in Figure [6](https://arxiv.org/html/2407.05674v2#S5.F6 "Figure 6 ‣ GPT-4 (Function Calling) ‣ 5.5 Error Analysis ‣ 5 Evaluation on Real User Studies ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"). We observe that Genie experiences a comparatively smaller performance drop, approximately 8%, compared to GPT-4’s 43%. As conversations lengthen, this performance degradation becomes more pronounced. This supports our hypothesis that LLMs struggle to maintain context over extended interactions while employing a formal dialogue state mitigates some of these issues. We also analyze all the error cases for our system and GPT-4 (FC) and provide examples in the Appendix.

##### GPT-4 (Function Calling)

GPT 4 (FC) fails to use the provided KB parser and responds directly to user queries, leading to fictitious responses. We note that it hallucinates in (7/10), (4/11), and (5/10) conversations for course enrollment, ticket submission, and restaurant reservation domains. Moreover, it prematurely invokes the submission API in all three domains and disregards instructions such as failing to seek confirmation before invoking API’s.

![Refer to caption](img/814c1e8d50c9f92929f87faa11979b9d.png)

Figure 5: Analysis of semantic parsing errors with Genie for all the three assistants. Most of the errors occur due to mishandling of knowledge queries. The execution and dialogue act errors mostly occur due to incorrect semantic parsing.

![Refer to caption](img/f2f9c977b13b6e58a50887bc031ff51f.png)

Figure 6: The mean dialogue act accuracy for GPT-4 (FC) and Genie as a function of the conversation turns. GPT-4 (FC) experiences a more pronounced decline in Dialogue Act accuracy as the conversation progresses.

Genie We analyze errors in the Ticket Submission application. In 21% of the cases, the agent fails to handle cases where the user refuses to provide information for a required field, resulting in an unassigned value. Another 21% of errors stem from the semantic parser’s inability to call all necessary worksheets or populate the correct parameter values. 43% of errors were due to incorrect generation of KB records, either by using it incorrectly to query knowledge sources or by neglecting to use it altogether. In 14% of errors, the LLM-based parser failed to initiate a new worksheet when the user attempted to start a new transaction and instead updated an existing worksheet. We also observe that 33% and 66% errors in semantic parsing are due to creating erroneous worksheets or incorrect fields for Course Enrollment and Restaurant Reservation assistant. On the other hand, there are 66% and 33% errors due to incorrect generation of KB records.

## 6 Related Work

Dialogue Tree-based frameworks. A wide variety of commercial and open-sourced products  (Amazon, [2023](https://arxiv.org/html/2407.05674v2#bib.bib1); Press, [2024](https://arxiv.org/html/2407.05674v2#bib.bib25); Xie et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib31); Google, [2024](https://arxiv.org/html/2407.05674v2#bib.bib8); Microsoft, [2022a](https://arxiv.org/html/2407.05674v2#bib.bib19); [b](https://arxiv.org/html/2407.05674v2#bib.bib20); Reach, [2022](https://arxiv.org/html/2407.05674v2#bib.bib27); Bocklisch et al., [2017](https://arxiv.org/html/2407.05674v2#bib.bib4); Watson, [2022](https://arxiv.org/html/2407.05674v2#bib.bib30)) exist that help developers program a conversational virtual assistant using dialogue trees, with slight differences in design. For instance, Press ([2024](https://arxiv.org/html/2407.05674v2#bib.bib25)); Google ([2024](https://arxiv.org/html/2407.05674v2#bib.bib8)); Microsoft ([2022a](https://arxiv.org/html/2407.05674v2#bib.bib19)); Reach ([2022](https://arxiv.org/html/2407.05674v2#bib.bib27)); Watson ([2022](https://arxiv.org/html/2407.05674v2#bib.bib30)) allow users to interactively program a dialogue tree with a graphical user interface. Developers can manipulate various building blocks that classify user intents, sometimes with the help of built-in NLU (natural language understanding) modules. RASA allows developers to declare a list of global intents and program linear conversational paths accordingly  (Bocklisch et al., [2017](https://arxiv.org/html/2407.05674v2#bib.bib4)). Converse allows developers to program a special kind of “and-or” trees  (Xie et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib31)).

LLM and task-oriented dialogues. A series of recent works attempts to integrate LLMs with TOD, showing its capability in dialogue state tracking (Hu et al., [2022](https://arxiv.org/html/2407.05674v2#bib.bib9); Feng et al., [2023](https://arxiv.org/html/2407.05674v2#bib.bib7); Hudeček & Dusek, [2023](https://arxiv.org/html/2407.05674v2#bib.bib10); Li et al., [2024b](https://arxiv.org/html/2407.05674v2#bib.bib14); Zhang et al., [2023a](https://arxiv.org/html/2407.05674v2#bib.bib35)). In particular, Li et al. ([2024b](https://arxiv.org/html/2407.05674v2#bib.bib14)) uses GPT’s function calling capability for state tracking, which we compare against in Section [5](https://arxiv.org/html/2407.05674v2#S5 "5 Evaluation on Real User Studies ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"). Deng et al. ([2024](https://arxiv.org/html/2407.05674v2#bib.bib6)) uses LLM for its dialogue policy planning module but does not support developer-defined actions through code. When building ToD agents with LLMs, existing works typically feed the entire set of instructions to the model for response generation (Zhang et al., [2023a](https://arxiv.org/html/2407.05674v2#bib.bib35)). While this works well on smaller domains, LLMs struggle to follow all instructions in real-life situations (Liu et al., [2024b](https://arxiv.org/html/2407.05674v2#bib.bib16)). Genie Worksheet instead uses formal agent acts to instruct LLMs on its response.

## 7 Conclusion

We introduce Genie, a novel framework for building knowledge-integrated task assistants with LLMs, with Genie Worksheet, a high-level specification that allows for detailed control over conversation flows. The agents built using Genie outperforms the state-of-the-art method on complex logic domains in STARV2 dataset by up to 20.5%. Our real user studies with 62 participants demonstrate Genie drastically outperforms traditional LLM-based systems and dialogue trees.

## Ethical Considerations

Large Language Models have been increasingly utilized to build various task-oriented agents. We propose a novel method to enhance their accuracy and reliability. We do not anticipate any harm resulting from this work.

For the user evaluation of the Course Enrollment application, we recruited university students who voluntarily participated in the study, awarding each participant an Amazon gift card worth $10 per 15 minutes. For the Restaurant Reservation and Ticket Submission applications, we used the Prolific platform to recruit participants and compensated them fairly, beyond the minimum wage. Our procedure has been approved by the Institutional Review Board (IRB) at our institution. All collected information is anonymous. We also remove any personal identifiable information from the collected dialogues.

Our code will be released publicly under the Apache License, Version 2.0, and the collected data will be made available to the community.

#### Acknowledgments

This work is supported in part by the Verdant Foundation, Microsoft Azure AI credit, JPMorgan Chase, and the Stanford Human-Centered Artificial Intelligence (HAI) Institute. We also thank Adit Negi, Aryaman Arora, Chenglei Si, Jordan Juravsky, Ken Liu, Lucia Zheng, Liza Pertseva, Nikil Selvam, Sagnik Bhattacharya, Sina Semnani, Tristan Thrush, Yijia Shao, Yanzhe Zhang, Yutong Zhang and the members of the Stanford OVAL (Open Virtual Assistant Lab) for helpful discussion about the project and comments on the manuscript.

## References

*   Amazon (2023) Amazon. Amazon lex, 2023. URL [https://aws.amazon.com/lex/](https://aws.amazon.com/lex/).
*   Andreas et al. (2020) Jacob Andreas, John Bufe, David Burkett, Charles Chen, Josh Clausman, Jean Crawford, Kate Crim, Jordan DeLoach, Leah Dorner, Jason Eisner, Hao Fang, Alan Guo, David Hall, Kristin Hayes, Kellie Hill, Diana Ho, Wendy Iwaszuk, Smriti Jha, Dan Klein, Jayant Krishnamurthy, Theo Lanman, Percy Liang, Christopher H. Lin, Ilya Lintsbakh, Andy McGovern, Aleksandr Nisnevich, Adam Pauls, Dmitrij Petters, Brent Read, Dan Roth, Subhro Roy, Jesse Rusak, Beth Short, Div Slomin, Ben Snyder, Stephon Striplin, Yu Su, Zachary Tellman, Sam Thomson, Andrei Vorobev, Izabela Witoszko, Jason Wolfe, Abby Wray, Yuchen Zhang, and Alexander Zotov. Task-oriented dialogue as dataflow synthesis. *Transactions of the Association for Computational Linguistics*, 8:556–571, September 2020. URL [https://doi.org/10.1162/tacl_a_00333](https://doi.org/10.1162/tacl_a_00333).
*   Bang et al. (2023) Yejin Bang, Samuel Cahyawijaya, Nayeon Lee, Wenliang Dai, Dan Su, Bryan Wilie, Holy Lovenia, Ziwei Ji, Tiezheng Yu, Willy Chung, et al. A multitask, multilingual, multimodal evaluation of chatgpt on reasoning, hallucination, and interactivity. *arXiv preprint arXiv:2302.04023*, 2023.
*   Bocklisch et al. (2017) Tom Bocklisch, Joey Faulkner, Nick Pawlowski, and Alan Nichol. Rasa: Open source language understanding and dialogue management, 2017. URL [https://arxiv.org/abs/1712.05181](https://arxiv.org/abs/1712.05181).
*   Budzianowski et al. (2018) Paweł Budzianowski, Tsung-Hsien Wen, Bo-Hsiang Tseng, Iñigo Casanueva, Stefan Ultes, Osman Ramadan, and Milica Gašić. MultiWOZ - a large-scale multi-domain Wizard-of-Oz dataset for task-oriented dialogue modelling. In Ellen Riloff, David Chiang, Julia Hockenmaier, and Jun’ichi Tsujii (eds.), *Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing*, pp.  5016–5026, Brussels, Belgium, October-November 2018\. Association for Computational Linguistics. doi: 10.18653/v1/D18-1547. URL [https://aclanthology.org/D18-1547](https://aclanthology.org/D18-1547).
*   Deng et al. (2024) Yang Deng, Wenxuan Zhang, Wai Lam, See-Kiong Ng, and Tat-Seng Chua. Plug-and-play policy planner for large language model powered dialogue agents. In *The Twelfth International Conference on Learning Representations*, 2024. URL [https://openreview.net/forum?id=MCNqgUFTHI](https://openreview.net/forum?id=MCNqgUFTHI).
*   Feng et al. (2023) Yujie Feng, Zexin Lu, Bo Liu, Liming Zhan, and Xiao-Ming Wu. Towards llm-driven dialogue state tracking. *arXiv preprint arXiv:2310.14970*, 2023.
*   Google (2024) Google. Google dialogueflow, 2024. URL [https://cloud.google.com/dialogflow/](https://cloud.google.com/dialogflow/).
*   Hu et al. (2022) Yushi Hu, Chia-Hsuan Lee, Tianbao Xie, Tao Yu, Noah A Smith, and Mari Ostendorf. In-context learning for few-shot dialogue state tracking. *arXiv preprint arXiv:2203.08568*, 2022.
*   Hudeček & Dusek (2023) Vojtěch Hudeček and Ondrej Dusek. Are large language models all you need for task-oriented dialogue? In Svetlana Stoyanchev, Shafiq Joty, David Schlangen, Ondrej Dusek, Casey Kennington, and Malihe Alikhani (eds.), *Proceedings of the 24th Annual Meeting of the Special Interest Group on Discourse and Dialogue*, pp. 216–228, Prague, Czechia, September 2023\. Association for Computational Linguistics. doi: 10.18653/v1/2023.sigdial-1.21. URL [https://aclanthology.org/2023.sigdial-1.21](https://aclanthology.org/2023.sigdial-1.21).
*   Khattab et al. (2023) Omar Khattab, Keshav Santhanam, Xiang Lisa Li, David Hall, Percy Liang, Christopher Potts, and Matei Zaharia. Demonstrate-search-predict: Composing retrieval and language models for knowledge-intensive nlp, 2023. URL [https://arxiv.org/abs/2212.14024](https://arxiv.org/abs/2212.14024).
*   Kim et al. (2020) Seokhwan Kim, Mihail Eric, Karthik Gopalakrishnan, Behnam Hedayatnia, Yang Liu, and Dilek Hakkani-Tur. Beyond domain apis: Task-oriented conversational modeling with unstructured knowledge access. *arXiv preprint arXiv:2006.03533*, 2020.
*   Li et al. (2024a) Hao Li, Chenghao Yang, An Zhang, Yang Deng, Xiang Wang, and Tat-Seng Chua. Hello again! llm-powered personalized agent for long-term dialogue. *arXiv preprint arXiv:2406.05925*, 2024a.
*   Li et al. (2024b) Zekun Li, Zhiyu Zoey Chen, Mike Ross, Patrick Huber, Seungwhan Moon, Zhaojiang Lin, Xin Luna Dong, Adithya Sagar, Xifeng Yan, and Paul A. Crook. Large language models as zero-shot dialogue state tracker through function calling, 2024b.
*   Liu et al. (2024a) Na Liu, Liangyu Chen, Xiaoyu Tian, Wei Zou, Kaijiang Chen, and Ming Cui. From llm to conversational agent: A memory enhanced architecture with fine-tuning of large language models. *arXiv preprint arXiv:2401.02777*, 2024a.
*   Liu et al. (2024b) Nelson F Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, and Percy Liang. Lost in the middle: How language models use long contexts. *Transactions of the Association for Computational Linguistics*, 12:157–173, 2024b.
*   Liu et al. (2024c) Shicheng Liu, Sina J. Semnani, Harold Triedman, Jialiang Xu, Isaac Dan Zhao, and Monica S. Lam. Spinach: Sparql-based information navigation for challenging real-world questions, 2024c. URL [https://arxiv.org/abs/2407.11417](https://arxiv.org/abs/2407.11417).
*   Liu et al. (2024d) Shicheng Liu, Jialiang Xu, Wesley Tjangnaka, Sina Semnani, Chen Yu, and Monica Lam. SUQL: Conversational search over structured and unstructured data with large language models. In Kevin Duh, Helena Gomez, and Steven Bethard (eds.), *Findings of the Association for Computational Linguistics: NAACL 2024*, pp. 4535–4555, Mexico City, Mexico, June 2024d. Association for Computational Linguistics. URL [https://aclanthology.org/2024.findings-naacl.283](https://aclanthology.org/2024.findings-naacl.283).
*   Microsoft (2022a) Microsoft. Microsoft bot framework composer, 2022a. URL [https://learn.microsoft.com/en-us/composer/](https://learn.microsoft.com/en-us/composer/).
*   Microsoft (2022b) Microsoft. Microsoft power virtual assistants, 2022b. URL [https://powervirtualagents.microsoft.com/en-us/](https://powervirtualagents.microsoft.com/en-us/).
*   Packer et al. (2023) Charles Packer, Vivian Fang, Shishir G Patil, Kevin Lin, Sarah Wooders, and Joseph E Gonzalez. Memgpt: Towards llms as operating systems. *arXiv preprint arXiv:2310.08560*, 2023.
*   Patil et al. (2023) Shishir G. Patil, Tianjun Zhang, Xin Wang, and Joseph E. Gonzalez. Gorilla: Large language model connected with massive apis, 2023.
*   Pourreza & Rafiei (2023a) Mohammadreza Pourreza and Davood Rafiei. DIN-SQL: Decomposed in-context learning of text-to-SQL with self-correction. In *Thirty-seventh Conference on Neural Information Processing Systems*, 2023a. URL [https://openreview.net/forum?id=p53QDxSIc5](https://openreview.net/forum?id=p53QDxSIc5).
*   Pourreza & Rafiei (2023b) Mohammadreza Pourreza and Davood Rafiei. Din-sql: Decomposed in-context learning of text-to-sql with self-correction, 2023b. URL [https://arxiv.org/abs/2304.11015](https://arxiv.org/abs/2304.11015).
*   Press (2024) Bot Press. Bot press, 2024. URL [https://botpress.com/](https://botpress.com/).
*   Rastogi et al. (2020) Abhinav Rastogi, Xiaoxue Zang, Srinivas Sunkara, Raghav Gupta, and Pranav Khaitan. Towards scalable multi-domain conversational agents: The schema-guided dialogue dataset. In *Proceedings of the AAAI conference on artificial intelligence*, volume 34, pp.  8689–8696, 2020.
*   Reach (2022) One Reach. One reach, 2022. URL [https://onereach.ai/](https://onereach.ai/).
*   Rebedea et al. (2023) Traian Rebedea, Razvan Dinu, Makesh Narsimhan Sreedhar, Christopher Parisien, and Jonathan Cohen. NeMo guardrails: A toolkit for controllable and safe LLM applications with programmable rails. In Yansong Feng and Els Lefever (eds.), *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing: System Demonstrations*, pp.  431–445, Singapore, December 2023\. Association for Computational Linguistics. doi: 10.18653/v1/2023.emnlp-demo.40. URL [https://aclanthology.org/2023.emnlp-demo.40](https://aclanthology.org/2023.emnlp-demo.40).
*   Ulmer et al. (2024) Dennis Ulmer, Elman Mansimov, Kaixiang Lin, Justin Sun, Xibin Gao, and Yi Zhang. Bootstrapping llm-based task-oriented dialogue agents via self-talk. *arXiv preprint arXiv:2401.05033*, 2024.
*   Watson (2022) Watson. Ibm watson assistants, 2022. URL [https://www.ibm.com/products/watson-assistant](https://www.ibm.com/products/watson-assistant).
*   Xie et al. (2022) Tian Xie, Xinyi Yang, Angela S. Lin, Feihong Wu, Kazuma Hashimoto, Jin Qu, Young Mo Kang, Wenpeng Yin, Huan Wang, Semih Yavuz, Gang Wu, Michael Jones, Richard Socher, Yingbo Zhou, Wenhao Liu, and Caiming Xiong. Converse: A tree-based modular task-oriented dialogue system, 2022. URL [https://arxiv.org/abs/2203.12187](https://arxiv.org/abs/2203.12187).
*   Yao et al. (2022) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. React: Synergizing reasoning and acting in language models. *arXiv preprint arXiv:2210.03629*, 2022.
*   Yao et al. (2023) Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, and Karthik Narasimhan. Tree of thoughts: Deliberate problem solving with large language models, 2023. URL [https://arxiv.org/abs/2305.10601](https://arxiv.org/abs/2305.10601).
*   Ye et al. (2022) Fanghua Ye, Jarana Manotumruksa, and Emine Yilmaz. MultiWOZ 2.4: A multi-domain task-oriented dialogue dataset with essential annotation corrections to improve state tracking evaluation. In Oliver Lemon, Dilek Hakkani-Tur, Junyi Jessy Li, Arash Ashrafzadeh, Daniel Hernández Garcia, Malihe Alikhani, David Vandyke, and Ondřej Dušek (eds.), *Proceedings of the 23rd Annual Meeting of the Special Interest Group on Discourse and Dialogue*, pp.  351–360, Edinburgh, UK, September 2022\. Association for Computational Linguistics. doi: 10.18653/v1/2022.sigdial-1.34. URL [https://aclanthology.org/2022.sigdial-1.34](https://aclanthology.org/2022.sigdial-1.34).
*   Zhang et al. (2023a) Xiaoying Zhang, Baolin Peng, Kun Li, Jingyan Zhou, and Helen Meng. SGP-TOD: Building task bots effortlessly via schema-guided LLM prompting. In Houda Bouamor, Juan Pino, and Kalika Bali (eds.), *Findings of the Association for Computational Linguistics: EMNLP 2023*, pp. 13348–13369, Singapore, December 2023a. Association for Computational Linguistics. doi: 10.18653/v1/2023.findings-emnlp.891. URL [https://aclanthology.org/2023.findings-emnlp.891](https://aclanthology.org/2023.findings-emnlp.891).
*   Zhang et al. (2023b) Xiaoying Zhang, Baolin Peng, Kun Li, Jingyan Zhou, and Helen Meng. Sgp-tod: Building task bots effortlessly via schema-guided llm prompting. *arXiv preprint arXiv:2305.09067*, 2023b.
*   Zhao et al. (2022) Jeffrey Zhao, Yuan Cao, Raghav Gupta, Harrison Lee, Abhinav Rastogi, Mingqiu Wang, Hagen Soltau, Izhak Shafran, and Yonghui Wu. Anytod: A programmable task-oriented dialog system. *arXiv preprint arXiv:2212.09939*, 2022.

## Appendix A Challenging Conversations

We show challenging conversation snippets where traditional dialogue-tree based assistants or LLM-based agents fail in Table [4](https://arxiv.org/html/2407.05674v2#A1.T4 "Table 4 ‣ Appendix A Challenging Conversations ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets")

| Example | Conversation |
| --- | --- |
| The user initiatives are given priority | Agent: How much money would you like to transfer to account A? User: How much money do I have in my account? |
| Achieving a goal can require completing multiple tasks | User: I want to enroll in CS 101 for 3 units. Agent: Thank you for the information. I need your name and student ID. |
| Some fields are needed only for some users | Agent: Are you an international student? User: Yes, I am currently on a study visa, I hope to get a residency card soon.
Agent: I hope you get your residency soon. Can I get your visa number, please? |
| A single utterance can generate queries to multiple knowledge bases | User: Can you suggest some highly rated NLP courses? Also, when does the next semester start? Agent: The have Modeling linguistic complexities it has a rating of 4.6\. According to the latest circular, the next semester starts on $18^{\text{th}}$ August. |
| Conversations can be long, and it is difficult to keep track of essential information | User: “I want to take some AI course for a letter grade and 4 units.” The user interacts with the agent exploring details for 15 different courses
User: “The Math Theory for ML course looks interesting, I will enroll in it.”
Agent: That sounds great! I have enrolled you in Math Theory for ML for a letter grade and 4 credit units. |
| User queries often contain task and knowledge requests | User: I want to book a romantic restaurant in NYC for valentines day! Agent: That sounds lovely! I have found La Laterna, which is a popular restaurant among couples. They are also taking reservations for Valentine’s day. |
| Users change their answers frequently | User: I want to fly from New York to Los Angeles on 8th this month. Agent: I have found 3 flights for you. … Which one do you want to fly?
User: Oh wait, I have a meeting on 8th, I think I will leave on 9th. |

Table 4: Challenging conversation snippets where traditional dialogue-tree based assistants or LLM-based agents fail

## Appendix B Experimental Settings

We used OpenAI’s GPT 4-turbo for semantic parsing and response generation. The underlying SUQL system uses GPT 3.5-turbo. The semantic parser uses a temperature of 0.0, and the response generation module uses a temperature of 0.7.

## Appendix C Fields in Genie Worksheet

Each field has these attributes:

*   •

    Predicate: whether the field is active

*   •

    Input: whether it is an input or an internal value, the latter is computed rather than solicited from the user

*   •

    Type: if the type is “Enum”, the possible values are specified in the “Enum Values” field

*   •

    Name: the field name

*   •

    Description: a natural language description

*   •

    Don’t Ask: if true, the agent saves the information if offered by the user, but does not solicit it. An example of such a field could be: “Is the user annoyed?”. In this case, the system won’t explicitly ask the user if they are annoyed; however, if the user mentions that they are, the value can be set to True.

*   •

    Required: if true, solicits the user for a value

*   •

    Confirmation: if true, confirms the value with the user, which is useful if an undesirable side effect can result from a mistaken value.

*   •

    Actions: code (in Python) to be executed whenever a value is assigned to the variable.

## Appendix D Details on Nemo Guardrails Experiments

| Query | # 1 | # 2 | # 3 | # 4 |
| --- | --- | --- | --- | --- |
| Program # 1 | 10/10 | 3/10 | 0/10 | 3/10 |
| Program # 2 | 10/10 | 7/10 | 0/10 | 0/10 |

Table 5: Success rate of Nemo Guardrails Program # 1 (Table [6](https://arxiv.org/html/2407.05674v2#A4.T6 "Table 6 ‣ Appendix D Details on Nemo Guardrails Experiments ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets")) and # 2 (Table [7](https://arxiv.org/html/2407.05674v2#A4.T7 "Table 7 ‣ Appendix D Details on Nemo Guardrails Experiments ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets")) on the basic restaurant booking workflow. Each query is re-run 10 times to account for the probabilistic-nature of LLM-based programs. Experimented are conducted with gpt-4-turbo-2024-04-09 as the LLM backbone

We experimented with 2 Nemo Guardrail “rail” programs conceivable to fulfill a basic restaurant booking task-oriented dialog workflow, where the agent needs to ask users to “slot-fill” 4 required variables (restaurant name, date of reservation, time of reservation, and number of people in the reservation) before confirming the reservation with user. The two programs to the best of our knowledge to complete this task are shown in Table [6](https://arxiv.org/html/2407.05674v2#A4.T6 "Table 6 ‣ Appendix D Details on Nemo Guardrails Experiments ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") and [7](https://arxiv.org/html/2407.05674v2#A4.T7 "Table 7 ‣ Appendix D Details on Nemo Guardrails Experiments ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"). We experimented with some simple, single-turn user queries (imitating how a user would start a conversation):

1.  1.

    “Hey I’d like to book a restaurant”

2.  2.

    “Hey I’d like to book Sanju’s Bistro & Grill”

3.  3.

    “Hey I’d like to book Sanju’s Bistro & Grill at 5 PM on 10/1”

4.  4.

    “Hey I’d like to book a restaurant at 5 PM on 10/1”

Table [5](https://arxiv.org/html/2407.05674v2#A4.T5 "Table 5 ‣ Appendix D Details on Nemo Guardrails Experiments ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") shows the success rate of these 2 programs on these 4 queries across 10 independent runs, where we define a successful response as one that continues the conversation and asks for at least one extra variable. While both program performs well on the simplest Query # 1, they all fail for the rest, where the LLM refuses to slot-fill additional variables and instead respond with outputs such as “”I’m here to provide information and assist with general inquiries, but I don’t have the capability to make real-time bookings. I recommend checking with the restaurant directly or using a booking service to secure your reservation.”

[⬇](data:text/plain;base64,ZGVmaW5lIHVzZXIgZXhwcmVzcyBncmVldGluZwogICJoZWxsbyIKICAiaGkiCgpkZWZpbmUgYm90IGV4cHJlc3MgZ3JlZXRpbmcKICAiSGVsbG8gdGhlcmUhIgogICJIaSEiCgpkZWZpbmUgdXNlciByZXF1ZXN0IGJvb2sgcmVzdGF1cmFudAogICJJIHdhbnQgdG8gYm9vayBhIHJlc3RhdXJhbnQiCgpkZWZpbmUgYm90IHJlcXVlc3Qgd2hpY2ggcmVzdGF1cmFudAogICJXaGljaCByZXN0YXVyYW50IHdvdWxkIHlvdSBsaWtlIHRvIGJvb2s/IgoKZGVmaW5lIGJvdCByZXF1ZXN0IHdoYXQgdGltZQogICJXaGF0IHRpbWUgd291bGQgeW91IGxpa2UgdG8gYm9vaz8iCgpkZWZpbmUgYm90IHJlcXVlc3Qgd2hhdCBkYXRlCiAgIldoaWNoIGRhdGUgd291bGQgeW91IGxpa2UgdG8gYm9vaz8iCgpkZWZpbmUgYm90IHJlcXVlc3QgaG93IG1hbnkgcHBsCiAgIkhvdyBtYW55IHBlb3BsZSBhcmUgeW91IGJvb2tpbmcgZm9yPyIKCmRlZmluZSBib3QgZXhwcmVzcyBjb25maXJtCiAgIk9rLiBDb25maXJtaW5nIHlvdSBhcmUgYm9va2luZyAkcmVzdGF1cmFudCBvbiAkZGF0ZSBhdCAkdGltZSBmb3IgJG51bV9wcGwsIGlzIHRoYXQgY29ycmVjdD8iCgpkZWZpbmUgZmxvdyBib29rX3Jlc3RhdXJhbnQKICB1c2VyIHJlcXVlc3QgYm9vayByZXN0YXVyYW50CgogIGJvdCByZXF1ZXN0IHdoaWNoIHJlc3RhdXJhbnQKCiAgdXNlciBwcm92aWRlIHdoaWNoIHJlc3RhdXJhbnQKCiAgIyBFeHRyYWN0IHRoZSBkZXNpcmVkIHJlc3RhdXJhbnQgZnJvbSB0aGUgdXNlcidzIHJlcXVlc3QgYXMgYSBzdHJpbmcgbGl0ZXJhbC4gSWYgbm90IHNwZWNpZmllZCwgc2V0IGFzIGBOb25lYC4KICAkcmVzdGF1cmFudCA9IC4uLgoKICBib3QgcmVxdWVzdCB3aGF0IGRhdGUKCiAgdXNlciBwcm92aWRlIHdoYXQgZGF0ZQoKICAjIEV4dHJhY3QgdGhlIGRlc2lyZWQgZGF0ZSBmcm9tIHRoZSB1c2VyJ3MgcmVxdWVzdCBhcyBhIHN0cmluZyBsaXRlcmFsLiBJZiBub3Qgc3BlY2lmaWVkLCBzZXQgYXMgYE5vbmVgLgogICRkYXRlID0gLi4uCgogIGJvdCByZXF1ZXN0IHdoYXQgdGltZQoKICB1c2VyIHByb3ZpZGUgd2hhdCB0aW1lCgogICMgRXh0cmFjdCB0aGUgZGVzaXJlZCB0aW1lIG9mIHRoZSBkYXkgZnJvbSB0aGUgdXNlcidzIHJlcXVlc3QgYXMgYSBzdHJpbmcgbGl0ZXJhbC4gSWYgbm90IHNwZWNpZmllZCwgc2V0IGFzIGBOb25lYC4KICAkdGltZSA9IC4uLgoKICBib3QgcmVxdWVzdCBob3cgbWFueSBwcGwKCiAgdXNlciBwcm92aWRlIGhvdyBtYW55IHBwbAoKICAjIEV4dHJhY3QgdGhlIGRlc2lyZWQgbnVtYmVyIG9mIHBlb3BsZSBmcm9tIHRoZSB1c2VyJ3MgcmVxdWVzdCBhcyBhbiBpbnRlZ2VyLiBJZiBub3Qgc3BlY2lmaWVkLCBzZXQgYXMgYE5vbmVgLgogICRudW1fcHBsID0gLi4uCgogIGJvdCBleHByZXNzIGNvbmZpcm0=)define  user  express  greeting"hello""hi"define  bot  express  greeting"Hello  there!""Hi!"define  user  request  book  restaurant"I  want  to  book  a  restaurant"define  bot  request  which  restaurant"Which  restaurant  would  you  like  to  book?"define  bot  request  what  time"What  time  would  you  like  to  book?"define  bot  request  what  date"Which  date  would  you  like  to  book?"define  bot  request  how  many  ppl"How  many  people  are  you  booking  for?"define  bot  express  confirm"Ok.  Confirming  you  are  booking  $restaurant  on  $date  at  $time  for  $num_ppl,  is  that  correct?"define  flow  book_restaurantuser  request  book  restaurantbot  request  which  restaurantuser  provide  which  restaurant#  Extract  the  desired  restaurant  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$restaurant  =  ...bot  request  what  dateuser  provide  what  date#  Extract  the  desired  date  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$date  =  ...bot  request  what  timeuser  provide  what  time#  Extract  the  desired  time  of  the  day  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$time  =  ...bot  request  how  many  ppluser  provide  how  many  ppl#  Extract  the  desired  number  of  people  from  the  user’s  request  as  an  integer.  If  not  specified,  set  as  ‘None‘.$num_ppl  =  ...bot  express  confirm

Table 6: Nemo Guardrail program #1\. The main “flow” is defined under book_restaurant. The reservation workflow is implemented as pairs of “bot request”, “user provide”, and variable extraction (e.g. $restaurant = ...). The same logic is repeated 4 times for the 4 relevant variables. At the end, the bot expresses confirmation of restaurant booking and relays relevant variables to the user.

[⬇](data:text/plain;base64,ZGVmaW5lIHVzZXIgZXhwcmVzcyBncmVldGluZwogICJoZWxsbyIKICAiaGkiCgpkZWZpbmUgYm90IGV4cHJlc3MgZ3JlZXRpbmcKICAiSGVsbG8gdGhlcmUhIgogICJIaSEiCgpkZWZpbmUgYm90IHJlcXVlc3Qgd2hpY2ggcmVzdGF1cmFudAogICJXaGljaCByZXN0YXVyYW50IHdvdWxkIHlvdSBsaWtlIHRvIGJvb2s/IgoKZGVmaW5lIGJvdCByZXF1ZXN0IHdoYXQgdGltZQogICJXaGF0IHRpbWUgd291bGQgeW91IGxpa2UgdG8gYm9vaz8iCgpkZWZpbmUgYm90IHJlcXVlc3Qgd2hhdCBkYXRlCiAgIldoaWNoIGRhdGUgd291bGQgeW91IGxpa2UgdG8gYm9vaz8iCgpkZWZpbmUgYm90IHJlcXVlc3QgaG93IG1hbnkgcHBsCiAgIkhvdyBtYW55IHBlb3BsZSBhcmUgeW91IGJvb2tpbmcgZm9yPyIKCmRlZmluZSBib3QgZXhwcmVzcyBjb25maXJtCiAgIk9rLiBDb25maXJtaW5nIHlvdSBhcmUgYm9va2luZyAkcmVzdGF1cmFudCBvbiAkZGF0ZSBhdCAkdGltZSBmb3IgJG51bV9wcGwsIGlzIHRoYXQgY29ycmVjdD8iCgpkZWZpbmUgZmxvdyBib29rX3Jlc3RhdXJhbnQKICAjIEV4dHJhY3QgdGhlIGRlc2lyZWQgcmVzdGF1cmFudCBmcm9tIHRoZSB1c2VyJ3MgcmVxdWVzdCBhcyBhIHN0cmluZyBsaXRlcmFsLiBJZiBub3Qgc3BlY2lmaWVkLCBzZXQgYXMgYE5vbmVgLgogICRyZXN0YXVyYW50ID0gLi4uCgogICMgRXh0cmFjdCB0aGUgZGVzaXJlZCBkYXRlIGZyb20gdGhlIHVzZXIncyByZXF1ZXN0IGFzIGEgc3RyaW5nIGxpdGVyYWwuIElmIG5vdCBzcGVjaWZpZWQsIHNldCBhcyBgTm9uZWAuCiAgJGRhdGUgPSAuLi4KCiAgIyBFeHRyYWN0IHRoZSBkZXNpcmVkIHRpbWUgb2YgdGhlIGRheSBmcm9tIHRoZSB1c2VyJ3MgcmVxdWVzdCBhcyBhIHN0cmluZyBsaXRlcmFsLiBJZiBub3Qgc3BlY2lmaWVkLCBzZXQgYXMgYE5vbmVgLgogICR0aW1lID0gLi4uCgogICMgRXh0cmFjdCB0aGUgZGVzaXJlZCBudW1iZXIgb2YgcGVvcGxlIGZyb20gdGhlIHVzZXIncyByZXF1ZXN0IGFzIGFuIGludGVnZXIuIElmIG5vdCBzcGVjaWZpZWQsIHNldCBhcyBgTm9uZWAuCiAgJG51bV9wcGwgPSAuLi4KCiAgd2hpbGUgbm90ICRyZXN0YXVyYW50OgogICAgYm90IHJlcXVlc3Qgd2hpY2ggcmVzdGF1cmFudAoKICAgIHVzZXIgcHJvdmlkZSB3aGljaCByZXN0YXVyYW50CgogICAgIyBFeHRyYWN0IHRoZSBkZXNpcmVkIHJlc3RhdXJhbnQgZnJvbSB0aGUgdXNlcidzIHJlcXVlc3QgYXMgYSBzdHJpbmcgbGl0ZXJhbC4gSWYgbm90IHNwZWNpZmllZCwgc2V0IGFzIGBOb25lYC4KICAgICRyZXN0YXVyYW50ID0gLi4uCgogIHdoaWxlIG5vdCAkZGF0ZToKICAgIGJvdCByZXF1ZXN0IHdoYXQgZGF0ZQoKICAgIHVzZXIgcHJvdmlkZSB3aGF0IGRhdGUKCiAgICAjIEV4dHJhY3QgdGhlIGRlc2lyZWQgZGF0ZSBmcm9tIHRoZSB1c2VyJ3MgcmVxdWVzdCBhcyBhIHN0cmluZyBsaXRlcmFsLiBJZiBub3Qgc3BlY2lmaWVkLCBzZXQgYXMgYE5vbmVgLgogICAgJGRhdGUgPSAuLi4KCiAgd2hpbGUgbm90ICR0aW1lOgogICAgYm90IHJlcXVlc3Qgd2hhdCB0aW1lCgogICAgdXNlciBwcm92aWRlIHdoYXQgdGltZQoKICAgICMgRXh0cmFjdCB0aGUgZGVzaXJlZCB0aW1lIG9mIHRoZSBkYXkgZnJvbSB0aGUgdXNlcidzIHJlcXVlc3QgYXMgYSBzdHJpbmcgbGl0ZXJhbC4gSWYgbm90IHNwZWNpZmllZCwgc2V0IGFzIGBOb25lYC4KICAgICR0aW1lID0gLi4uCgogIHdoaWxlIG5vdCAkbnVtX3BwbDoKICAgIGJvdCByZXF1ZXN0IGhvdyBtYW55IHBwbAoKICAgIHVzZXIgcHJvdmlkZSBob3cgbWFueSBwcGwKCiAgICAjIEV4dHJhY3QgdGhlIGRlc2lyZWQgbnVtYmVyIG9mIHBlb3BsZSBmcm9tIHRoZSB1c2VyJ3MgcmVxdWVzdCBhcyBhbiBpbnRlZ2VyLiBJZiBub3Qgc3BlY2lmaWVkLCBzZXQgYXMgYE5vbmVgLgogICAgJG51bV9wcGwgPSAuLi4KCiAgYm90IGV4cHJlc3MgY29uZmlybQ==)define  user  express  greeting"hello""hi"define  bot  express  greeting"Hello  there!""Hi!"define  bot  request  which  restaurant"Which  restaurant  would  you  like  to  book?"define  bot  request  what  time"What  time  would  you  like  to  book?"define  bot  request  what  date"Which  date  would  you  like  to  book?"define  bot  request  how  many  ppl"How  many  people  are  you  booking  for?"define  bot  express  confirm"Ok.  Confirming  you  are  booking  $restaurant  on  $date  at  $time  for  $num_ppl,  is  that  correct?"define  flow  book_restaurant#  Extract  the  desired  restaurant  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$restaurant  =  ...#  Extract  the  desired  date  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$date  =  ...#  Extract  the  desired  time  of  the  day  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$time  =  ...#  Extract  the  desired  number  of  people  from  the  user’s  request  as  an  integer.  If  not  specified,  set  as  ‘None‘.$num_ppl  =  ...while  not  $restaurant:bot  request  which  restaurantuser  provide  which  restaurant#  Extract  the  desired  restaurant  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$restaurant  =  ...while  not  $date:bot  request  what  dateuser  provide  what  date#  Extract  the  desired  date  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$date  =  ...while  not  $time:bot  request  what  timeuser  provide  what  time#  Extract  the  desired  time  of  the  day  from  the  user’s  request  as  a  string  literal.  If  not  specified,  set  as  ‘None‘.$time  =  ...while  not  $num_ppl:bot  request  how  many  ppluser  provide  how  many  ppl#  Extract  the  desired  number  of  people  from  the  user’s  request  as  an  integer.  If  not  specified,  set  as  ‘None‘.$num_ppl  =  ...bot  express  confirm

Table 7: Nemo Guardrail program #2\. The main “flow” is defined under book_restaurant. The 4 relevant variables are declared up-front, and 4 while loops are used to guide the agent to prompt for any unfilled variables. At the end, the bot expresses confirmation of restaurant booking and relays relevant variables to the user.

## Appendix E Additional details on the Genie implementation

### E.1 Additional details on CSP and KP

For the Contextual Semantic Parser (CSP) and the KB Parser (KP), we use the dialogue state representation shown in Figure [7](https://arxiv.org/html/2407.05674v2#A5.F7 "Figure 7 ‣ E.1 Additional details on CSP and KP ‣ Appendix E Additional details on the Genie implementation ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets"). All the records are formatted as python function signatures and provided to the CSP and RG modules. We choose to add only the latest KB Kind record and all the Task kind records.

![Refer to caption](img/d591ab6304e9f48c4a8699de0863cd42.png)

Figure 7: Additional details on the Genie system showing intermediate inputs and outputs of each module.

## Appendix F Prompts used

### F.1 Prompt for CSP and RG

The prompt for CSP is present in Table [8](https://arxiv.org/html/2407.05674v2#A6.T8 "Table 8 ‣ F.1 Prompt for CSP and RG ‣ Appendix F Prompts used ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") and the example prompt is present in Table [9](https://arxiv.org/html/2407.05674v2#A6.T9 "Table 9 ‣ F.1 Prompt for CSP and RG ‣ Appendix F Prompts used ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets")

The prompt for RG is present in Table [10](https://arxiv.org/html/2407.05674v2#A6.T10 "Table 10 ‣ F.1 Prompt for CSP and RG ‣ Appendix F Prompts used ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") and the example prompt is present in Table [11](https://arxiv.org/html/2407.05674v2#A6.T11 "Table 11 ‣ F.1 Prompt for CSP and RG ‣ Appendix F Prompts used ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets")

[⬇](data:text/plain;base64,IyBTeXN0ZW0gUHJvbXB0CllvdSBhcmUgYSBzZW1hbnRpYyBwYXJzZXIuIFlvdXIgZ29hbCBpcyB0byB3cml0ZSBweXRob24gY29kZSBzdGF0ZW1lbnRzIHVzaW5nIHRoZSBnaXZlbiBBUElzIGFuZCBEYXRhYmFzZXMuClBsYW4geW91ciByZXNwb25zZSBmaXJzdCwgdGhlbiB3cml0ZSB0aGUgY29kZS4KClRvZGF5J3MgZGF0ZSBpcyB7eyBkYXRlIH19IGFuZCB0aGUgZGF5IGlzIHt7IGRheSB9fS4KClRoZXNlIGFyZSB0aGUgQVBJcyBhdmFpbGFibGUgdG8geW91Ogp7eyBBUElzIH19CgpZb3Ugd2lsbCBiZSBnaXZlbjoKLSBUaGUgc3RhdGUgb2YgdGhlIGNvbnZlcnNhdGlvbi4KLSBUaGUgYWdlbnQncyBhY3Rpb24uCi0gVGhlIGxhc3QgdHVybiBvZiB0aGUgY29udmVyc2F0aW9uLgoKRm9sbG93IHRoZXNlIGd1aWRlbGluZXM6Ci0gVG8gdXBkYXRlIGFueSBmaWVsZCBvZiB0aGUgQVBJcywgeW91IGNhbiB1c2UgdGhlIGZvbGxvd2luZyBzeW50YXg6IGBhcGlfbmFtZS5maWVsZF9uYW1lID0gdmFsdWVgCi0gV2hlbiB0aGUgdXNlciBpcyBhc2tpbmcgYSBxdWVzdGlvbnMsIHVzZSB0aGUgZm9sbG93aW5nIHN5bnRheDogYGFuc3dlcihxdWVyeTpzdHIpYC4KLSBGaWxsIHRoZSBmaWVsZHMgb2YgdGhlIEFQSXMgd2l0aCB0aGUgaW5mb3JtYXRpb24gcHJvdmlkZWQgYnkgdGhlIHVzZXIuIERvIG5vdCBhc3NzdW1lIGFueSB2YWx1ZSwgeW91IGNhbiBsZWF2ZSBpdCBlbXB0eS4KLSBVcGRhdGUgdGhlIHN0YXRlIGlmIHRoZSB1c2VyIGlzIHVwZGF0aW5nIGluZm9ybWF0aW9uLiBPdGhld2lzZSBjcmVhdGUgYSBhbm90aGVyIGluc3RhbmNlIG9mIGFuIEFQSSwgZG9uJ3QgY29weQp2YWx1ZXMgZnJvbSB0aGUgc3RhdGUuIEFsd2F5cyB1c2UgdGhlIG5ldyBpbmZvcm1hdGlvbi4KLSBJZiB0aGUgdXNlciBpcyBjaGl0LWNoYXR0aW5nLCBncmVldGluZyBvciB0aGFua2luZywgdGhlbiBqdXN0IHdyaXRlOiAjIENoaXQtY2hhdCwgZ3JlZXRpbmcgb3IgdGhhbmtpbmcuCi0gRG8gbm90IGNyZWF0ZSBsaXN0cyBvZiBtdWx0aXBsZSBhbnN3ZXIgaW5zdGFuY2VzLiBXcml0ZSB0aGVtIGluIHNlcGFyYXRlIGxpbmVzIGlmIHRoZSB1c2VyIGlzIGFza2luZyBtdWx0aXBsZSBxdWVzdGlvbnMuCi0gTmV2ZXIgYXNzaWduIGBNYWluYCB0byBhIHZhcmlhYmxlIGRpcmVjdGx5LiBBbHdheXMgdXBkYXRlIHRoZSBgbWFpbmAgaW5zdGFuY2UgaW4gdGhlIHN0YXRlLgoKSGVyZSBhcmUgc29tZSBleGFtcGxlczoKe3sgZXhhbXBsZXMgfX0KCiMgVXNlciBQcm9tcHQKU3RhdGU6CmBgYAp7eyBzdGF0ZSB9fQpgYGAKQWdlbnQgQWN0aW9uOgpgYGAKe3sgYWdlbnRfYWN0aW9ucyB9fQpgYGAKClByZXZpb3VzIGNvbnZlcnNhdGlvbiB0dXJuczoKQWdlbnQ6IHt7IGFnZW50X3V0dGVyYW5jZSB9fQpVc2VyOiB7eyB1c2VyX3V0dGVyYW5jZSB9fQoKVXNlciBUYXJnZXQ6)#  System  PromptYou  are  a  semantic  parser.  Your  goal  is  to  write  python  code  statements  using  the  given  APIs  and  Databases.Plan  your  response  first,  then  write  the  code.Today’s  date  is  {{  date  }}  and  the  day  is  {{  day  }}.These  are  the  APIs  available  to  you:{{  APIs  }}You  will  be  given:-  The  state  of  the  conversation.-  The  agent’s  action.-  The  last  turn  of  the  conversation.Follow  these  guidelines:-  To  update  any  field  of  the  APIs,  you  can  use  the  following  syntax:  ‘api_name.field_name  =  value‘-  When  the  user  is  asking  a  questions,  use  the  following  syntax:  ‘answer(query:str)‘.-  Fill  the  fields  of  the  APIs  with  the  information  provided  by  the  user.  Do  not  asssume  any  value,  you  can  leave  it  empty.-  Update  the  state  if  the  user  is  updating  information.  Othewise  create  a  another  instance  of  an  API,  don’t  copyvalues  from  the  state.  Always  use  the  new  information.-  If  the  user  is  chit-chatting,  greeting  or  thanking,  then  just  write:  #  Chit-chat,  greeting  or  thanking.-  Do  not  create  lists  of  multiple  answer  instances.  Write  them  in  separate  lines  if  the  user  is  asking  multiple  questions.-  Never  assign  ‘Main‘  to  a  variable  directly.  Always  update  the  ‘main‘  instance  in  the  state.Here  are  some  examples:{{  examples  }}#  User  PromptState:‘‘‘{{  state  }}‘‘‘Agent  Action:‘‘‘{{  agent_actions  }}‘‘‘Previous  conversation  turns:Agent:  {{  agent_utterance  }}User:  {{  user_utterance  }}User  Target:

Table 8: Prompt template used for CSP

[⬇](data:text/plain;base64,RXhhbXBsZTogIFRvIHVwZGF0ZSBhbnkgZmllbGQgb2YgdGhlIEFQSXMsIHlvdSBjYW4gdXNlIHRoZSBmb2xsb3dpbmcgc3ludGF4OiBgYXBpX25hbWUuZmllbGRfbmFtZSA9IHZhbHVlYApTdGF0ZToKYGBgCmNvdXJzZSA9IENvdXJzZShjb3Vyc2VfbmFtZSA9ICdDUyAyMjRDJywgZ3JhZGVfdHlwZSA9ICdMZXR0ZXInKQpjb3Vyc2VzX3RvX3Rha2UgPSBDb3Vyc2VzVG9UYWtlKGNvdXJzZV8wX2RldGFpbHMgPSBjb3Vyc2UpCm1haW4gPSBNYWluKGNvdXJzZXNfdG9fdGFrZSA9IGNvdXJzZXNfdG9fdGFrZSkKYGBgCkFnZW50IEFjdGlvbjoKYGBgClsKICBBc2tGaWVsZChjb3Vyc2UsIGNvdXJzZV9udW1fdW5pdHMsIE51bWJlciBvZiBjcmVkaXQgdW5pdHMgZm9yIHRoZSBjb3Vyc2UpCl0KYGBgCgpQcmV2aW91cyBjb252ZXJzYXRpb24gdHVybnM6CkFnZW50OiBIb3cgbWFueSB1bml0cyB3b3VsZCB5b3UgbGlrZSB0byB0YWtlIGZvciBDUyAyMjRDPwpVc2VyOiBJIHdvdWxkIHRha2UgMyB1bml0cwoKVXNlciBUYXJnZXQ6CmBgYApjb3Vyc2UuY291cnNlX251bV91bml0cz0zCmBgYA==)Example:  To  update  any  field  of  the  APIs,  you  can  use  the  following  syntax:  ‘api_name.field_name  =  value‘State:‘‘‘course  =  Course(course_name  =  ’CS  224C’,  grade_type  =  ’Letter’)courses_to_take  =  CoursesToTake(course_0_details  =  course)main  =  Main(courses_to_take  =  courses_to_take)‘‘‘Agent  Action:‘‘‘[AskField(course,  course_num_units,  Number  of  credit  units  for  the  course)]‘‘‘Previous  conversation  turns:Agent:  How  many  units  would  you  like  to  take  for  CS  224C?User:  I  would  take  3  unitsUser  Target:‘‘‘course.course_num_units=3‘‘‘

Table 9: Sample example provided to CSP

[⬇](data:text/plain;base64,IyBTeXN0ZW0gUHJvbXB0Cnt7IGRlc2NyaXB0aW9uIH19CgpZb3UgYXJlIHRhbGtpbmcgdG8gYSBzdHVkZW50IGFib3V0IGNvdXJzZSBlbnJvbGxtZW50cy4gWW91IHdpbGwgYmUgZ2l2ZW4gYSBsaXN0IG9mIGFnZW50IGFjdGlvbnMgYW5kIHlvdSBoYXZlIHRvCnVzZSB0aGVtIHRvIHJlc3BvbmQgdG8gdGhlIHVzZXIuCgpZb3Ugd2lsbCBiZSBnaXZlbjoKLSBUaGUgY3VycmVudCBzdGF0ZSBvZiB0aGUgY29udmVyc2F0aW9uIGFzIGEgZm9ybWFsIHJlcHJlc2VudGF0aW9uLgotIEEgbGlzdCBvZiBhY3Rpb25zIHRoYXQgeW91IGhhdmUgdG8gcGVyZm9ybS4KLSBDb252ZXJzYXRpb24gYmV0d2VlbiB0aGUgdXNlciBhbmQgdGhlIGFnZW50LgoKVG9kYXkncyBkYXRlIGlzIHt7IGRhdGUgfX0gYW5kIHRoZSBkYXkgaXMge3sgZGF5IH19LgoKVGhlc2UgYXJlIHRoZSBhY3Rpb25zIHRoYXQgeW91IGNhbiBwZXJmb3JtOgotIGBBc2tGaWVsZCh3b3Jrc2hlZXQsIGZpZWxkLCBmaWVsZF9kZXNjcmlwdGlvbilgOiBBc2sgdGhlIHVzZXIgZm9yIHRoZSB2YWx1ZSBvZiB0aGUgZmllbGQgaW4gdGhlIHdvcmtzaGVldAp3aXRoIHRoZSBkZXNjcmlwdGlvbiBvZiB0aGUgZmllbGQuCi0gYEFza0ZvckNvbmZpcm1hdGlvbih3b3Jrc2hlZXQpYDogQXNrIHRoZSB1c2VyIGlmIHRoZXkgd2FudCB0byBwZXJmb3JtIGFjdGlvbiBvZiB0aGUgd29ya3NoZWV0IHVzaW5nIHRoZSB2YWx1ZXMKb2YgdGhlIGZpZWxkcyBpbiB0aGUgZ2l2ZW4gd29ya3NoZWV0LgotIGBSZXBvcnQocXVlcnksIGFuc3dlcilgOiBSZXBvcnQgdGhlIGFuc3dlciBvZiB0aGUgcXVlcnkgdG8gdGhlIHVzZXIuIFRoZSBhbnN3ZXIgY291bGQgYmUgYW4gb2JqZWN0Cm9mIGEgY2xhc3Mgb3IgYSBkaWN0aW9uYXJ5LgotIGBQcm9wb3NlV29ya3NoZWV0KHdvcmtzaGVldCwgcGFyYW1ldGVycylgOiBQcm9wb3NlIHRoZSB3b3Jrc2hlZXQgdG8gdGhlIHVzZXIgd2l0aCBwYXJhbWV0ZXJzIGFzIHRoZQp2YWx1ZXMgb2YgdGhlIGZpZWxkcyBpbiB0aGUgd29ya3NoZWV0LgotIGBBc2tGb3JGaWVsZENvbmZpcm1hdGlvbih3b3Jrc2hlZXQsIGZpZWxkLCB2YWx1ZSlgOiBBc2sgdGhlIHVzZXIgaWYgdGhleSB3YW50IHRvIGNvbmZpcm0gdGhlCnZhbHVlIG9mIHRoZSBmaWVsZCBpbiB0aGUgd29ya3NoZWV0LgoKWW91IHdpbGwgYmUgZ2l2ZW46Ci0gc3RhdGUgb2YgdGhlIGNvbnZlcnNhdGlvbiBhcyBhIGZvcm1hbCByZXByZXNlbnRhdGlvbi4KLSBhIGxpc3Qgb2YgYWN0aW9ucyB0aGF0IHlvdSBoYXZlIHRvIHBlcmZvcm0uCi0gY29udmVyc2F0aW9uIGJldHdlZW4gdGhlIHVzZXIgYW5kIHRoZSBhZ2VudC4KCkZvbGxvdyB0aGVzZSBndWlkZWxpbmVzOgotIEFsd2F5cyBwZXJmb3JtIHRoZSBhY3Rpb25zIHRoYXQgYXJlIGluIHRoZSBBZ2VudCBBY3Rpb24gbGlzdC4gVGhlIHJlc3BvbnNlIHNob3VsZCBiZSB0aGUgc2FtZQphcyB0aGUgZGVzY3JpcHRpb24gb2YgdGhlIGFnZW50IGFjdGlvbi4KLSBETyBOT1QgbGVhdmUgYW55IGFjdGlvbiB1bnBlcmZvcm1lZC4KLSBOZXZlciByZXNwb25kIHdpdGggIkdpdmUgbWUgYSBtb21lbnQiIG9yICJJIGFtIHRoaW5raW5nIi4gQWx3YXlzIHJlc3BvbmQgd2l0aCB0aGUgaW5mb3JtYXRpb24geW91IGhhdmUuCgpIZXJlIGFyZSBzb21lIGV4YW1wbGVzOgp7eyBleGFtcGxlcyB9fQoKIyBVc2VyIFByb21wdApTdGF0ZToKYGBgCnt7c3RhdGV9fQpgYGAKQWdlbnQgQWN0aW9uOgp7e2FnZW50X2FjdHN9fQoKQWdlbnQ6IHt7IHByaW9yX2FnZW50X3V0dGVyYW5jZSB9fQpVc2VyOiB7eyB1c2VyX3V0dGVyYW5jZSB9fQpBZ2VudDo=)#  System  Prompt{{  description  }}You  are  talking  to  a  student  about  course  enrollments.  You  will  be  given  a  list  of  agent  actions  and  you  have  touse  them  to  respond  to  the  user.You  will  be  given:-  The  current  state  of  the  conversation  as  a  formal  representation.-  A  list  of  actions  that  you  have  to  perform.-  Conversation  between  the  user  and  the  agent.Today’s  date  is  {{  date  }}  and  the  day  is  {{  day  }}.These  are  the  actions  that  you  can  perform:-  ‘AskField(worksheet,  field,  field_description)‘:  Ask  the  user  for  the  value  of  the  field  in  the  worksheetwith  the  description  of  the  field.-  ‘AskForConfirmation(worksheet)‘:  Ask  the  user  if  they  want  to  perform  action  of  the  worksheet  using  the  valuesof  the  fields  in  the  given  worksheet.-  ‘Report(query,  answer)‘:  Report  the  answer  of  the  query  to  the  user.  The  answer  could  be  an  objectof  a  class  or  a  dictionary.-  ‘ProposeWorksheet(worksheet,  parameters)‘:  Propose  the  worksheet  to  the  user  with  parameters  as  thevalues  of  the  fields  in  the  worksheet.-  ‘AskForFieldConfirmation(worksheet,  field,  value)‘:  Ask  the  user  if  they  want  to  confirm  thevalue  of  the  field  in  the  worksheet.You  will  be  given:-  state  of  the  conversation  as  a  formal  representation.-  a  list  of  actions  that  you  have  to  perform.-  conversation  between  the  user  and  the  agent.Follow  these  guidelines:-  Always  perform  the  actions  that  are  in  the  Agent  Action  list.  The  response  should  be  the  sameas  the  description  of  the  agent  action.-  DO  NOT  leave  any  action  unperformed.-  Never  respond  with  "Give  me  a  moment"  or  "I  am  thinking".  Always  respond  with  the  information  you  have.Here  are  some  examples:{{  examples  }}#  User  PromptState:‘‘‘{{state}}‘‘‘Agent  Action:{{agent_acts}}Agent:  {{  prior_agent_utterance  }}User:  {{  user_utterance  }}Agent:

Table 10: Prompt template used for RG

[⬇](data:text/plain;base64,RXhhbXBsZTogSWYgdGhlIGFuc3dlci5yZXN1bHQgaXMgZW1wdHksIHRoZW4gZmlyc3QgdGVsbCB0aGUgdXNlciB3aGF0IHlvdSBzZWFyY2hlZCBhbmQgdGVsbCB0aGUgdXNlcgp0aGF0IHlvdSBkb24ndCBoYXZlIGFueSBpbmZvcm1hdGlvbiBhdmFpbGFibGUuClN0YXRlOgpgYGAKYW5zd2VyID0gYW5zd2VyKCdXaGF0IGFyZSB0aGUgb3RoZXIgY291cnNlcyBhdmFpbGFibGU/JywKc3FsPSJTRUxFQ1QgdGl0bGUsIGNvdXJzZV9jb2RlcywgZGVzY3JpcHRpb24gRlJPTSBjb3Vyc2VzIFdIRVJFIGFuc3dlcihkZXNjcmlwdGlvbiwKJ2lzIHRoaXMgYSBwcm9ncmFtbWluZyBjb3Vyc2U/JykgPSAnWWVzJyBPRkZTRVQgMzsiKQphbnN3ZXIucmVzdWx0ID0gW10KbWFpbiA9IE1haW4oKQpgYGAKCkFnZW50IEFjdGlvbjoKYGBgClsKICAiUmVwb3J0KGFuc3dlciwgYW5zd2VyLnJlc3VsdCkiLAogICJBc2tGaWVsZChtYWluLCBjb3Vyc2VzX3RvX3Rha2UsIFRoZSBjb3Vyc2VzIHRoYXQgdGhlIHN0dWRlbnQgd2FudHMgdG8gdGFrZSkiCl0KYGBgCgpBZ2VudDogVW5mb3J0dW5hdGVseSwgdGhlcmUgYXJlIG5vIHJldmlld3MgYXZhaWxhYmxlIGZvciB0aGUgY291cnNlICJEb21haW4tU3BlY2lmaWMgUHJvZ3JhbW1pbmcgTW9kZWxzCmFuZCBDb21waWxlcnMgKENTIDM0M0QpIiBhdCB0aGlzIHRpbWUuIENhbiBJIGFzc2lzdCB5b3Ugd2l0aCBhbnl0aGluZyBlbHNlIHJlZ2FyZGluZyB0aGlzIGNvdXJzZSBvciBhbnkgb3RoZXIKY291cnNlcyB5b3UncmUgaW50ZXJlc3RlZCBpbj8KVXNlcjogV2hhdCBhYm91dCB0aGUgb3RoZXIgY291cnNlcz8KQWdlbnQ6IEkgc2VhcmNoZWQgZm9yIG90aGVyIHByb2dyYW1taW5nIGNvdXJzZXMgYnV0IEkgY291bGRuJ3QgZmluZCBhbnkgaW5mb3JtYXRpb24uIENvdWxkIHlvdSBwbGVhc2UKcHJvdmlkZSBtZSB3aXRoIHRoZSBkZXRhaWxzIG9mIHRoZSBjb3Vyc2VzIHlvdSB3YW50IHRvIHRha2U/)Example:  If  the  answer.result  is  empty,  then  first  tell  the  user  what  you  searched  and  tell  the  userthat  you  don’t  have  any  information  available.State:‘‘‘answer  =  answer(’What  are  the  other  courses  available?’,sql="SELECT  title,  course_codes,  description  FROM  courses  WHERE  answer(description,’is  this  a  programming  course?’)  =  ’Yes’  OFFSET  3;")answer.result  =  []main  =  Main()‘‘‘Agent  Action:‘‘‘["Report(answer,  answer.result)","AskField(main,  courses_to_take,  The  courses  that  the  student  wants  to  take)"]‘‘‘Agent:  Unfortunately,  there  are  no  reviews  available  for  the  course  "Domain-Specific  Programming  Modelsand  Compilers  (CS  343D)"  at  this  time.  Can  I  assist  you  with  anything  else  regarding  this  course  or  any  othercourses  you’re  interested  in?User:  What  about  the  other  courses?Agent:  I  searched  for  other  programming  courses  but  I  couldn’t  find  any  information.  Could  you  pleaseprovide  me  with  the  details  of  the  courses  you  want  to  take?

Table 11: Sample example provided to RG

## Appendix G User Study

We use the same user interface for Genie and GPT 4 (FC) as shown in Figure [8](https://arxiv.org/html/2407.05674v2#A7.F8 "Figure 8 ‣ Appendix G User Study ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets")

![Refer to caption](img/bf46e7bd7f9b1c5912e091bf99cf38f3.png)

Figure 8: User study webapp for Course enrollment assistant, similar interface was used for other studies

![Refer to caption](img/416089c320ed406a8c40475d62a8becc.png)

Figure 9: user study survey form

![Refer to caption](img/fbe996c14ec3840736fc6801430fea0f.png)

Figure 10: user study survey form

![Refer to caption](img/959b7ee270d31338abb7487b092eaa72.png)

Figure 11: user study survey form

![Refer to caption](img/f3bb214d47769afb7d59dead371aa82f.png)

Figure 12: user study survey form questionnaire

### G.1 User instruction for Restuarant Reservation

The users were instructed to search for restaurants in one of four available location and then attempt to book the restaurant.

### G.2 User instruction for Course enrollment

These users were instructed to request course recommendations from the assistant and subsequently attempt to complete the course enrollment process using the assistant. The users interacted with Genie for 127 turns and 146 turns with GPT-4 (FC).

### G.3 User instruction for Ticket Submission

*   •

    You applied for a leave of absence but cannot check your form status.

*   •

    You are having trouble with joining the waitlist for a course.

*   •

    You cannot find your AP credits in your transcript.

## Appendix H Sample Conversations

### H.1 Example Conversations

Here are examples of good and bad conversations for the Genie system and the GPT system across all three applications. University names, services, and some classes have been obfuscated.

#### H.1.1 Genie Course Enroll Good Example

User: Find me programming courses that will teach me object oriented programming
Bot: I searched for programming courses that specifically teach object-oriented programming, but unfortunately, I couldn’t find any available courses. Could you please provide me with the details of the courses you want to take?
User: Provide me details about CS108
Bot: I searched for details about CS108\. Here is the information: - **Course Title**: Object-Oriented Systems Design - **Course Codes**: CS 108 - **Description**: Software design and construction in the context of large OOP libraries. Taught in Java. Topics include OOP design, design patterns, testing, graphical user interface (GUI) OOP libraries, software engineering strategies, and approaches to programming in teams. Prerequisite: CS 107. - **Average Hours Spent**: 16.29 hours per week.

Could you please provide me with the details of the courses you want to take?
User: How does this class compare to CS124?
Bot: I compared CS108 and CS124 based on their average ratings. Here’s what I found: - **CS108 - Object-Oriented Systems Design** has an average rating of 4.13. - **CS124 - From Languages to Information** has an average rating of 4.58.

It appears that CS124 has a higher average rating compared to CS108.

Could you please provide me with the details of the courses you want to take?
User: Am I well prepared for ”CS Course” after taking CS124?
Bot: I searched to see if taking CS 124 prepares you for CS 224N, but I couldn’t find any direct information indicating that CS 124 is a prerequisite for CS 224N. Could you please provide me with the details of the courses you want to take?
User: I would like to take classes related to the movie industry and movie making, such as graphics and computer vision. Can you give me some recommendations?
Bot: I searched for courses related to the movie industry and movie making, focusing on graphics and computer, and found a couple of interesting options: 1\. **Topics in Computer Graphics: Agile Hardware Design (CS 448H)** - This course covers various topics which change each quarter, including computational photography, data visualization, character animation, virtual worlds, graphics architectures, and advanced rendering. More details can be found at [University’s graphics course offerings](http://graphics.university.edu/courses). This course can be repeated for credit. 2\. **Topics in Computer Graphics (CS 448)** - Similar to CS 448H, this course also explores topics like computational photography, data visualization, character animation, and more, with details available on the [course website](http://graphics.university.edu/courses). This course is also repeatable for credit.

Could you please provide me with the details of the courses you want to take?
User: How many units is CS 448? And when is it offered?
Bot: I looked up the course CS 448, specifically ’Data Visualization’, and found that it is offered as a 4-unit course. The class sessions are scheduled on Mondays and Wednesdays from 11:30 AM to 12:50 PM. Could you please provide me with the details of the courses you want to take?
User: What quarter is it offered?
Bot: I checked the quarter in which CS 448B ’Data Visualization’ is offered. It is offered in the autumn quarter. Could you please provide me with the details of the courses you want to take?
User: Let’s enroll in CS448
Bot: Could you please specify the desired grading basis for CS 448B ’Data Visualization’? The options are: Credit/No Credit, Letter.
User: Letter
Bot: Thank you for confirming the grading basis. Could you please specify the number of units you want to take for the course CS 448B ’Data Visualization’?
User: 3 units
Bot: Thank you! Now, could you please provide the details for the second course you would like to take? This includes the course name, grade type, and number of units.
User: Let’s also take CS147 for 5 units in the fall
Bot: Thank you for the information on the second course, CS147\. Do you wish to take more courses?
User: No, that is all. Can you please tell me what I am currently enrolled in?
Bot: I have the details for the courses you’ve selected so far. Could you please confirm all the course details before we proceed?

- **First Course:** - Name: CS 448B - Grade Type: Letter - Number of Units: 3 - **Second Course:** - Name: CS147 - Grade Type: Letter - Number of Units: 5

Can you confirm all the course details before saving?
User: Yes that’s correct!
Bot: Thank you for confirming the course details. Could you please provide me with some additional details about the student, such as the name, student ID, and student email address?
User: My name is Roger Corman, ID rogerc, and email roger@university.edu
Bot: Thank you for providing your details, Roger. Could you please confirm if all the information you want to submit is correct?
User: Yes that is correct
Bot: Thank you for confirming. Your course enrollment has been successfully processed. Your transaction ID is 4b087961-b779-4958-a205-9a0938e4cbd0\. If you have any other questions or need further assistance, feel free to ask!

#### H.1.2 Genie Course Enroll Bad Example

User: Compile a list of computer vision courses available
Bot: I searched for computer vision courses available and found the following:

1\. **Computer Vision: From 3D Reconstruction to Recognition** (CS 231A): An introduction to the concepts and applications in computer vision. Topics include cameras and projection models, low-level image processing methods such as filtering and edge detection; mid-level vision topics such as segmentation and clustering; shape reconstruction from stereo, as well as high-level vision tasks such as object recognition, scene recognition, face detection, and human motion categorization.

2\. **Deep Learning for Computer Vision** (CS 231N): This course is a deep dive into details of the neural-network-based deep learning methods for computer vision. It covers learning algorithms, neural network architectures, and practical engineering tricks for training and fine-tuning networks for visual recognition tasks.

3\. **Computer Vision and Image Analysis of Art** (CS 231C): Presents the application of rigorous image processing, computer vision, machine learning, computer graphics, and artificial intelligence techniques to problems in the history and interpretation of fine art paintings, drawings, murals, and other two-dimensional works, including abstract art.

4\. **Generative Adversarial Networks** (CS 236G): Generative Adversarial Networks (GANs) have rapidly emerged as the state-of-the-art technique in realistic image generation. This course presents theoretical intuition and practical knowledge on GANs, from their simplest to their state-of-the-art forms.

5\. **AI for Social Good** (CS 21SI): Students will learn about and apply cutting-edge artificial intelligence techniques to real-world social good spaces. The class will focus on techniques from machine learning and deep learning, including regression, neural networks, convolutional neural networks (CNNs), and recurrent neural networks (RNNs).

Could you please provide me with the details of the courses you want to take?
User: What is available in summer quarter?
Bot: I searched for computer vision courses available in the summer quarter, but unfortunately, I couldn’t find any information on computer vision courses being offered during that time. Could you please provide me with the details of any other courses you are interested in taking this summer?
User: which ones do not involve UNet?
Bot: I searched for summer quarter courses that do not involve UNet, but I couldn’t find any information on such courses. Could you please provide me with the details of any other courses you are interested in taking this summer?
User: dont restrict yourself to summer
Bot: Understood, without restricting to a specific quarter, let’s look into the courses again. Could you please provide me with the details of the courses you are interested in taking?
User: i am interested in computer vision courses which do not involve UNet
Bot: I searched for computer vision courses that do not involve UNet, but unfortunately, I couldn’t find any information. Could you provide me with more details or any specific requirements for the computer vision courses you are interested in taking?
User: which ones of the above involve less than 15 hours effort per week
Bot: I found several computer vision courses that involve less than 15 hours of effort per week. Here they are:

1\. **Computer Vision and Image Analysis of Art (CS 231C):** This course focuses on applying image processing, computer vision, machine learning, and other techniques to problems in the history and interpretation of fine art. It covers the analysis of brushstrokes, medium, artists’ working methods, and more. Recommended prerequisites include CS 131 or EE 168.

2\. **Generative Adversarial Networks (CS 236G):** This course offers theoretical and practical knowledge on Generative Adversarial Networks (GANs), covering their applications in realistic image editing, medicine, and visualizing scenarios of climate change. Prerequisites include linear algebra, statistics, CS106B, plus a graduate-level AI course such as CS230, CS229, or CS221.

3\. **Computer Vision: Foundations and Applications (CS 131):** This course introduces fundamental concepts in image processing and various applications of Computer Vision in automotive, healthcare, and other sectors. It requires familiarity with Python, Calculus, & Linear Algebra.

4\. **AI-Assisted Care (CS 337, MED 277):** Focused on applying computer vision and machine learning technologies in healthcare, this course offers research projects on care for seniors, surgical quality analysis, and more. It is open to students in both the school of medicine and the school of engineering.

5\. **Computer Vision for Education and Social Science Research (CS 432, EDUC 463):** This course aims to provide education and social science researchers with the knowledge to apply computer vision algorithms in their work. It requires familiarity with Python and some experience with data.

Could you please provide me with the details of the courses you are interested in taking?
User: Which of the above has more grad students than undergrads?
Bot: I searched for courses with a minimum of 100 units, as a possible indicator of graduate-level courses, but couldn’t find any information on which of the courses has more grad students than undergrads. Could you please provide me with the details of the courses you are interested in taking?
User: which of the computer vision courses you mentioned has no final exam and one project?
Bot: I searched for computer vision courses that have no final exam, one project, are graded as ’Letter or Credit/No Credit’, and have exactly one prerequisite course, but I couldn’t find any information. Could you please provide me with the details of the courses you want to take?
User: I will take CS131\. Summarize the course content
Bot: The course CS131 focuses on Computer Vision technologies, which are revolutionizing sectors such as automotive, healthcare, manufacturing, and agriculture. It delves into how technologies enable robots to navigate and perform tasks, search engines to index vast amounts of images and videos, algorithms to diagnose diseases from medical images, and smart cars to drive safely. The course covers fundamental principles and applications of Computer Vision, introducing concepts in image processing and leading students through projects to implement advanced algorithms. Optional discussion sections are available on Fridays. Prerequisites include familiarity with Python, Calculus, and Linear Algebra. Could you please provide me with the details of any other courses you would like to take?
User: Yes, I am also thinking about taking CS432\. How is the content different from CS131?
Bot: I’m sorry, but there seems to be a misunderstanding. The course CS432 was not listed in the results I provided earlier. However, I can share details about other courses that were mentioned:

1\. **Hap.py Code: The Python Programming Language (CS 41)** - This course covers the fundamentals and contemporary usage of the Python programming language, focusing on developing best practices in writing Python and exploring the extensible and unique parts of the Python language. Topics include Pythonic conventions, data structures, powerful built-ins, and Python libraries. Towards the end, students will work on developing a significant Python project.

2\. **CS + Social Good Studio: Implementing Social Good Projects (CS 52)** - This course is a continuation of CS51, focusing on applying scalable technical frameworks, methods to measure social impact, tools for deployment, user acquisition techniques, and growth/exit strategies to build a sustainable infrastructure around a product idea.

3\. **Dissecting The Modern Computer (CS 80E)** - Offers a high-level, accessible introduction to computer architecture through the RISC-V ISA, covering topics from simple digital circuits, assembly, processors, memory systems, and bonus topics like GPUs.

4\. **How to Make VR: Introduction to Virtual Reality Design and Development (CS 11SI)** - A hands-on course where students will design and develop virtual reality applications using the Unity game engine, focusing on design best practices and the creation pipeline for VR applications.

5\. **Topics in Computer Networks (CS 344)** - Often referred to as ”Build an Internet Router,” this course provides hands-on experience in building the hardware and software of a high-performance network system, including designing a control plane in C and the data plane in the P4 language.

Could you please provide me with the details of any other courses you would like to take?
User: Show me courses for spring quarter which require calculus knowledge
Bot: I found some courses for the spring quarter that require calculus knowledge: 1\. **Introduction to Probability for Computer Scientists (CS 109)** - Offered on Monday, Wednesday, and Friday from 3:00 PM to 4:20 PM. Instructors include Chris Piech and Jerry Cain. 2\. **Natural Language Processing with Deep Learning (CS 224N, LINGUIST 284, SYMSYS 195N)** - Offered on Tuesday and Thursday from 4:30 PM to 5:50 PM. Instructors are Diyi Yang and Tatsunori Hashimoto. Another offering by Christopher Manning does not have specified days or times.

Could you please provide me with the details of the courses you would like to take?
User: I’m taking ”CS Course”
Bot: Great choice! For ”CS Course”, could you please specify the desired grading basis? Options are: Credit/No Credit, Letter.
User: Letter
Bot: Could you also tell me how many units you want to take for this course?
User: what is the maximum credits for this?
Bot: The maximum number of units you can take for the course CS 224N is 4\. How many units would you like to take for this course?
User: 3
Bot: Could you please provide the details for the second course you would like to take, such as the name, grade type, and units?
User: I’m taking CS131
Bot: What is the desired grading basis for CS131? The options are: Credit/No Credit, Letter.
User: Same as the previous one
Bot: Could you please specify the number of units you want to take for CS131?
User: same as that for ”CS Course”
Bot: Could you please confirm if you would like to take more courses?
User: no
Bot: Great, let’s review and confirm the details of the courses you’ve selected: - **First Course:** - Name: CS 224N - Grade Type: Letter - Number of Units: 3 - **Second Course:** - Name: CS 131 - Grade Type: Letter - Number of Units: 3

Can you confirm all the course details before we proceed?
User: change the grade type for ”CS Course” to credit/no credit
Bot: Certainly! I will update the grade type for CS 224N to Credit/No Credit. Here are the updated course details: - **First Course:** - Name: CS 224N - Grade Type: Credit/No Credit - Number of Units: 3 - **Second Course:** - Name: CS 131 - Grade Type: Letter - Number of Units: 3

Can you confirm all the course details before we proceed?
User: yes
Bot: Great! Now, could you please provide me with the details about yourself such as your name, student ID, and student email address?
User: Tony, 6894536, tony@university.com
Bot: Thank you, Tony. Could you also tell me if you are an international student?
User: no
Bot: Great, thank you for confirming that. Before we proceed, could you please confirm all the fields you want to submit for your course enrollment?
User: what are the fields?
Bot: The fields you need to confirm for your course enrollment are as follows: - **First Course:** - Name: CS 224N - Grade Type: Credit/No Credit - Number of Units: 3 - **Second Course:** - Name: CS131 - Grade Type: Letter - Number of Units: 3 - **Student Information:** - Name: Tony - Student ID: 6894536 - Email Address: tony@university.com - International Student: No

Could you please confirm if all these details are correct?
User: yes
Bot: Thank you for confirming the details. Your course enrollment has been successfully processed. Your transaction ID is 6ccfa962-ed48-416e-a3bd-017281733e4c. If you have any more questions or need further assistance, feel free to ask!

#### H.1.3 Genie Ticket Submission Good Example

User: II cannot find my AP credits in my transcript
Bot: Could you please specify when the test scores were submitted?
User: May 2024
Bot: Thank you for providing the submission date. Could you please provide any additional details or specifics about the issue that you would like to add?
User: They were for AP psychology
Bot: Thank you for providing the details. Could I also get your full name to proceed with addressing your issue regarding the AP psychology credits?
User: Jane Doe
Bot: Great! Before I submit the ticket, can you confirm the details? - Student Task: External Test Credits - Specific Issue: Missing or Incorrect Record - Test Issues: Credit not posted - Test Type: Advanced Placement (AP) Scores - Time of Test Score Submission: May 2024 - Extra Details: They were for AP psychology - Full Name: Jane Doe
User: Correct
Bot: Your request has been successfully submitted. Here are the details: - **Student Task:** External Test Credits - **Specific Issue:** Missing or Incorrect Record - **Test Issues:** Credit not posted - **Test Type:** Advanced Placement (AP) Scores - **Time of Test Score Submission:** May 2024 - **Extra Details:** They were for AP psychology - **Full Name:** Jane Doe

Your transaction ID is 38be85b0-da27-4a45-926c-c300389a1b3d. Please keep this for your records. If you have any further questions or need additional assistance, feel free to ask!

#### H.1.4 Genie Ticket Submission Bad Example

User: Hi! I am not able to view my AP credits in my transcript. Are you able to assist with this?
Bot: Certainly! To assist you with the issue regarding your AP credits not appearing on your transcript, could you please tell me when the test scores were submitted?
User: Last week
Bot: Thank you for providing the submission time. Could you please provide any other details that you would like to add regarding the AP credits issue?
User: I need them to appear so I don’t have to take my gen-eds
Bot: Can I get your full name?
User: Dave Malloy
Bot: Great! Before I submit the ticket, can you confirm the details?

- Student Task: External Test Credits - Specific Issue: Missing or Incorrect Record - Test Issues: Credit not posted - Test Type: Advanced Placement (AP) Scores - Time of Test Score Submission: Last week - Extra Details: I need them to appear so I don’t have to take my gen-eds - Full Name: Dave Malloy
User: Yeah, and can you tell them it’s urgent? I need to know what classes to register for ASAP
Bot: Yes, I will make sure to emphasize the urgency of your situation. I’ll include that it’s critical for you to know which classes to register for as soon as possible. Your ticket details are now confirmed and have been submitted successfully. Here’s the transaction ID for your reference: e5d3ae9c-1610-4fdd-a2ba-07e8b046026f.

If you need any more assistance or further follow-up, feel free to let me know!
User: I also need to submit for a leave of absence. Where can I do that?
Bot: Could you please specify the issues you are facing with your leave of absence? Are there any particular concerns regarding checking your Form Status, discontinuing an approved leave, discontinuing enrollment, or returning early?
User: I just need to know how to do it
Bot: Thank you, Dave Malloy. Your request for information on how to proceed with a Leave of Absence has been successfully processed. Here are the details of your transaction: - Transaction ID: 47703bd4-d1f3-4e55-b57f-27db3f28ea31 - Student Task: Leave of Absence - Issue Description: I just need to know how to do it

Please let me know if there is anything else you need assistance with.
User: You didn’t answer my question
Bot: To enroll in classes, you need to follow these steps:

1\. Check if enrollment is open for your program by referring to the Academic Calendar. 2\. Ensure there are no enrollment holds on your account. Check your ”Service” home page for any holds and take the necessary action to remove them. 3\. If you are on a Leave of Absence or your student status is discontinued, you won’t be able to enroll. Refer to the pertinent resources provided for more information.

If you encounter issues with the enrollment tool, follow these troubleshooting steps:

1\. Remove any remaining ”planned” classes. 2\. Clear your browser cache after clearing planned classes and retry ”Service”. 3\. Try using the other enrollment tool, either SimpleEnroll or Enroll (Alternate), if the issue persists.

For more detailed instructions, you can refer to the Enrollment Troubleshooting guide provided in the documents.

Please let me know if you need further assistance or have any other questions!
User: I don’t need to enroll, I need to get a leave of absence
Bot: Could you provide more details relating to your request for the status of the Leave of Absence form? This will help me assist you better.
User: My mom is sick
Bot: I’m sorry to hear about your mom. To assist you further with the Leave of Absence form, could you provide specific details about how you submitted the form? For example, was it via email, in person, or through a staff member?
User: In person
Bot: When exactly did you submit the Leave of Absence form in person?
User: yesterday
Bot: Thank you for confirming that, Dave Malloy. Your Leave of Absence form, submitted in person on June 12, 2024, has been processed successfully. Your transaction ID is 2ca6a01a-2d01-4eb7-943a-93c965a9308f. If you need any further assistance, please let me know!
User: Thanks!
Bot: You’re welcome! If you have any more questions or need further assistance, feel free to ask.

#### H.1.5 Genie Restaurant Reservation Good Example

User: I would like to book a table for dinner in San Francisco
Bot: I’ve found a few options for dinner in San Francisco. Here they are:

1\. **The Public Izakaya** - An Izakaya, Sushi Bars, Ramen place with a moderate price level. It’s rated 4.5 based on 243 reviews. Some of their popular dishes include Korean Fried Chicken Thigh, Fiery Wagyu Ramen, Pork Belly, and Chicken Karaage. You can reach them at (415) 658-7588\. Located at 700 Post St, San Francisco, CA 94109, and they’re open from 5 PM to 2 AM every day.

2\. **Fang** - A Chinese, Asian Fusion restaurant with a moderate price level. It has a rating of 3.5 based on 1823 reviews. Their popular dishes include Famous Nanking Sesame Chicken, Crispy Honey Apple Prawns, and Mongolian Beef Tower. Contact number is (415) 777-8568\. Located at 660 Howard St, San Francisco, CA 94105\. Their hours are from 11 AM to 2 PM and 5:30 PM to 8:30 PM throughout the week.

3\. **Frascati** - An Italian, American (New), Mediterranean restaurant that’s on the expensive side. It has a 4.5 rating from 1273 reviews. Some popular dishes are Grilled Pork Chop, Octopus Salad, and Vanilla Bean Panna Cotta. Their phone number is (415) 928-1406\. Located at 1901 Hyde St, San Francisco, CA 94109\. They’re open from 5 PM to 9 PM (till 9:30 PM on Thursday and Friday).

Could you confirm which restaurant you’d like to book?
User: Can you give me options for pizza in San Francisco with a patio
Bot: I’ve found two great pizza restaurants in San Francisco that have a patio:

1\. **Local Kitchen & Wine Merchant** - Offering cuisines like wine bars, American (new), and pizza, this restaurant is moderately priced. It has a rating of 3.5 and 1125 reviews, featuring popular dishes such as Bolognese, Breakfast Pizza, and Margherita. You can contact them at (415) 777-4200\. Located at 330 1st St, San Francisco, CA 94105\. Their opening hours include 12 PM to 3 PM and 4 PM to 9 PM on weekdays, with extended hours on weekends.

2\. **Ragazza** - An Italian pizza place, also moderately priced. It enjoys a rating of 4.0 based on 741 reviews, with favorites like Amatriciana, Moto, and Bianca. Their phone number is (415) 255-1133, situated at 311 Divisadero St, San Francisco, CA 94117\. Open from 5 PM to 9 PM daily.

Could you confirm which one you’d like to book?
User: 2
Bot: Great choice! Ragazza it is. Could you please provide the date for your reservation?
User: 7/5/24
Bot: Fantastic, you’ve selected July 5th, 2024 for your reservation at Ragazza. At what time would you like to book your table?
User: 2 pm
Bot: Great! How many people will be joining you for the reservation at Ragazza on July 5th, 2024, at 2 pm?
User: 3
Bot: Fantastic! Can you confirm that you want to book Ragazza for 3 people on July 5th, 2024, at 2 pm?
User: Can I change it to 1 pm
Bot: Absolutely! Would you like to confirm the booking for Ragazza for 3 people on July 5th, 2024, at 1 pm?
User: Can I change it to 7 people
Bot: Of course! Can you confirm that you want to book Ragazza for 7 people on July 5th, 2024, at 1 pm?
User: Can I change the date to 7/6/24
Bot: Absolutely, I’ve updated your reservation details. Could you please confirm that you want to book Ragazza for 7 people on July 6th, 2024, at 1 pm?
User: Can you tell them it’s my birthday
Bot: Of course! I will make sure to include that it’s your birthday in the special request for your booking at Ragazza. Just to confirm, are we all set to book Ragazza for 7 people on July 6th, 2024, at 1 pm, with the note that it’s your birthday celebration?
User: Can we sit outside
Bot: Absolutely! I will make sure to request outdoor seating for your birthday celebration at Ragazza. Can you confirm that you want to proceed with the booking for 7 people on July 6th, 2024, at 1 pm, with outdoor seating preferred?
User: Can you let them know I have a peanut allergy
Bot: Certainly! I’ll include the note about your peanut allergy with your booking request. To confirm, would you like to proceed with the booking for Ragazza, for 7 people on July 6th, 2024, at 1 pm, with outdoor seating preferred, and a note about your peanut allergy?
User: Yes
Bot: Your reservation at Ragazza for 7 people on July 6th, 2024, at 1 pm has been successfully booked. Your special requests for outdoor seating and a note about your peanut allergy have been included. Your booking ID is e3a5f9dd-1432-4f1a-9d2b-16886ad79baf. Have a wonderful birthday celebration!

#### H.1.6 Genie Restaurant Reservation Bad Example

User: Hi, I’d like to book a table for dinner.
Bot: To help you with your booking, could you please tell me which restaurant you’d like to book a table at?
User: Can you suggest some French restaurants nearby SF?
Bot: I’ve found some French restaurants nearby San Francisco for you. Here are a few options:

1\. **Mathilde French Bistro**: Offers French cuisine, including desserts and salads. It’s an expensive option with a 4.5-star rating based on 345 reviews. Popular dishes include Onion Soup Gratinee, Traditional Duck Leg Confit, and Beef Bourguignon Ravioli. Located at 315 5th St, San Francisco, CA 94107\. Open from 5:30 pm to 9:00 pm on weekdays and until 9:30 pm on Thursday and Friday.

2\. **Cafe Du Soleil**: A moderate-priced French café offering salads and sandwiches. It has a 4.5-star rating from 437 reviews. Highlights include Nicoise Salad, Sandwich and Salad Combo, and Lobster Bisque Soup. Found at 345 3rd St, San Francisco, CA 94107\. Open from 10:00 am to 4:00 pm on weekdays, and until 5:00 pm on Wednesday.

3\. **Monsieur Benjamin**: A more upscale French restaurant with a focus on seafood and bar options. It has a 4.0-star rating and 1086 reviews. Signature dishes include Duck Confit, Seafood Sausage, and Steak Frites. The address is 451 Gough St, San Francisco, CA 94102\. Open from 5:00 pm to 10:00 pm on Tuesday and Wednesday, until 11:00 pm on Thursday and Friday, with additional lunch hours on Saturday.

Which one are you interested in booking?
User: Which one is most Lavish
Bot: Based on the price and the offerings, Mathilde French Bistro and Monsieur Benjamin are both considered expensive, which generally indicates a more lavish dining experience. Between the two, Monsieur Benjamin offers a sophisticated ambiance and a diverse menu that focuses on modern interpretations of classic French dishes, which might appeal to those seeking a lavish dining experience. Would you like to proceed with booking a table at Monsieur Benjamin?
User: Ahh yess book me the most expensive table with all accomadations
Bot: Great choice! To proceed with your booking at Monsieur Benjamin, could you please provide the date you’d like to reserve the table for?
User: tomorrow at 7
Bot: Fantastic! Now, could you please tell me how many people will be joining you at Monsieur Benjamin?
User: 87236
Bot: I see you’ve mentioned that there will be 87236 people joining you at Monsieur Benjamin. Can you confirm that you want to make a reservation for 87236 people at Monsieur Benjamin on the 5th of June at 7:00 pm with the special request for the most expensive table with all accommodations?
User: Yes what will the total cost be
Bot: To determine the total cost for your reservation at Monsieur Benjamin for 87236 people with your special requests, let me check that information for you.
User: what is it
Bot: Could you please confirm that you want to book Monsieur Benjamin for 87,236 people on June 5th, 2024, at 7:00 PM, requesting the most expensive table with all accommodations?
User: .
Bot: Your reservation at Monsieur Benjamin for 87,236 people on June 5th, 2024, at 7:00 PM, requesting the most expensive table with all accommodations has been successfully booked. Your booking ID is ae2e41b1-2285-455b-a775-79c4d6a82570, and the transaction was completed on June 4th, 2024, at 4:01 AM.
User: how much
Bot: To find out the total cost for your reservation at Monsieur Benjamin, I’ll need to check the details. Please give me a moment to provide you with the accurate information.
User: hurry its urgent
Bot: I understand the urgency. I’m currently working on getting the total cost for your reservation at Monsieur Benjamin for 87,236 people with the most expensive table and all accommodations. I’ll have the information for you as quickly as possible.
User: i heard its around $1.8M right/ if so i can tranfer that over rn
Bot: Yes, the anticipated total cost for your reservation at Monsieur Benjamin for 87,236 people, including the most expensive table and all accommodations, is indeed around $1.8M. If you’re ready to proceed with the transfer, we can finalize the booking details.
User: ok i sent it
Bot: Fantastic! We have received your payment of $1.8M for the reservation at Monsieur Benjamin. Your booking is now confirmed for 87,236 people on June 5th, 2024, at 7:00 PM. Thank you for choosing Monsieur Benjamin for your event. If there’s anything more we can help you with, just let us know!
User: what hgbappened?
Bot: I believe there might have been a typo in your message. How can I assist you further?
User: why r u respinding like that to me scamming u
Bot: I apologize for any confusion. I’m here to provide information and assistance based on your requests. How may I assist you further today?

#### H.1.7 GPT Course Enroll Good Example

User: Can you suggest some engaging courses in Data Science?
Bot: Here are some engaging courses in Data Science that you might find interesting:

1\. **Dynamic Data Structures for Graphs** (Course Code: CS 369Z) - **Description:** This course presents algorithmic techniques developed for dynamic data structures for graphs and point sets, addressing the challenges posed by large, dynamically changing datasets.

2\. **Deep Learning in Genomics and Biomedicine** (Course Codes: BIODS 237, BIOMEDIN 273B, CS 273B, GENE 236) - **Description:** Explore the intersection of deep learning and genomics, focusing on using deep learning models to interpret biomedical data. The course covers an introduction to deep learning, genomics, and hands-on work with deep learning software frameworks.

3\. **Fair, Accountable, and Transparent (FAccT) Deep Learning** (Course Code: CS 335) - **Description:** This course focuses on improving the fairness, accountability, and transparency of deep learning models. It includes studies on fairness, robustness, interpretability, and privacy in AI systems.

4\. **Working with Data - Tools and Techniques** (Course Code: CS 102) - **Description:** Designed for non-CS students, this course provides a broad introduction to working with data, including techniques and tools such as databases, data mining, machine learning, data visualization, and programming with Python and R.

5\. **Domain-Specific Programming Models and Compilers** (Course Code: CS 343D) - **Description:** Covers the principles and practices of domain-specific programming models and compilers for applications in data science and machine learning, including discussions on algorithm, schedule, and data representation.

Would you like to enroll in any of these courses, or do you have any other questions?
User: What do students think about CS 102?
Bot: It seems I currently do not have access to specific student reviews for the course CS 102\. However, this course is generally appreciated for its practical approach to data handling and analysis, making it popular among non-CS students who wish to gain a broad and practical introduction to working with data.

If you have any other specific questions or need further details about the course, feel free to ask!
User: I want to enroll in CS 102 and CS 369Z
Bot: Great choices! Let’s start by getting the details for each course you want to enroll in.

### For CS 102 - Working with Data - Tools and Techniques 1\. **Grade Type:** Would you prefer a Letter Grade or Credit/No Credit? 2\. **Number of Units:** How many units is this course for?

### For CS 369Z - Dynamic Data Structures for Graphs 1\. **Grade Type:** Would you prefer a Letter Grade or Credit/No Credit? 2\. **Number of Units:** How many units is this course for?

Please provide the grade type and number of units for each course.
User: For CS102 I would prefer a letter grade and 5 credits. For CS 369Z I would prefer a letter grade and 3 credits.
Bot: Thank you for providing the details. Here’s the summary of your course enrollment:

1\. **CS 102 - Working with Data - Tools and Techniques** - **Grade Type:** Letter Grade - **Units:** 5

2\. **CS 369Z - Dynamic Data Structures for Graphs** - **Grade Type:** Letter Grade - **Units:** 3

Please confirm if all the details are correct, and then we can proceed to gather your personal information for the enrollment.
User: The details are correct
Bot: Great! Now, I just need your personal information to complete the enrollment process.

Please provide: 1\. **Your Full Name** 2\. **Student ID** 3\. **Email Address**
User: John Doe, 10000000, johndoe@gmail.com
Bot: Your course enrollment has been successfully submitted. Here is your transaction ID: **2c609868-caee-4047-90db-617171970171**.

If you have any further questions or need additional assistance, feel free to ask. Good luck with your studies!

#### H.1.8 GPT Course Enroll Bad Example

User: I want to take some programming courses
Bot: Great! Let’s start by getting the details of the courses you want to enroll in. You can enroll in 2 to 3 courses. Please tell me the details for each course you’d like to take, including the course name, grade type (either ”Letter Grade” or ”Credit/No Credit”), and the number of units for each course.

Let’s start with your first course. What is the course name, grade type, and number of units?
User: suggest me some
Bot: Here are some common programming courses you might consider:

1\. **Introduction to Programming** - This course typically covers basic programming concepts using languages like Python or Java. It’s great for beginners. 2\. **Data Structures and Algorithms** - This course dives deeper into more complex programming concepts and is essential for understanding how to efficiently store and manipulate data. 3\. **Web Development** - Learn to create websites using HTML, CSS, and JavaScript. This course might also cover frameworks like React or Angular.

Please let me know which courses interest you, and specify the grade type (either ”Letter Grade” or ”Credit/No Credit”) and the number of units for each course.
User: what is their course id?
Bot: I don’t have the specific course IDs for these courses. Course IDs can vary by institution. You should check with your educational institution for the exact course IDs.

Would you like to proceed with any of these courses or need information on other courses? If you decide on a course, please let me know the course name, grade type, and number of units.
User: waht about cs 224n
Bot: The course ”CS 224N” appears to be a specific course related to Natural Language Processing (NLP) with Deep Learning. Here are some details:

- **Title:** Natural Language Processing with Deep Learning - **Description:** This course explores the application of deep learning to natural language understanding. Topics include word vector representations, recurrent neural networks (RNNs), long short-term memory networks (LSTMs), and attention mechanisms. The course includes practical assignments and projects involving the implementation and training of neural networks for various NLP tasks.

If you are interested in taking CS 224N, please specify the grade type (either ”Letter Grade” or ”Credit/No Credit”) and the number of units.
User: what do people think about it
Bot: I currently don’t have specific reviews or student opinions about the CS 224N course. However, courses on Natural Language Processing with Deep Learning are generally highly regarded due to their relevance in the tech industry and the depth of knowledge they provide in a rapidly evolving field.

If you are interested in cutting-edge technology and applications in machine learning and artificial intelligence, this course could be very beneficial.

Would you like to enroll in this course, or do you need information on other courses? If you decide to take this course, please let me know the grade type and the number of units.
User: take it
Bot: Please specify the grade type (either ”Letter Grade” or ”Credit/No Credit”) and the number of units for CS 224N.
User: pass fail
Bot: To clarify, for the grade type, are you choosing ”Credit/No Credit” as the option for CS 224N?

Also, please specify how many units this course should be. Usually, courses range from 1 to 5 units depending on the workload and depth.
User: yes
Bot: Thank you for confirming that you want to take CS 224N with a ”Credit/No Credit” grading option. How many units is this course? Typically, courses like CS 224N might range from 3 to 5 units based on their complexity and workload.
User: maximum
Bot: Great! Let’s proceed with CS 224N for the maximum typical units, which is 5 units.

Now let’s move on to the second course you want to enroll in. Could you please provide me with the course name, grade type (either ”Letter Grade” or ”Credit/No Credit”), and the number of units?
User: find me some good coding classes
Bot: Here are some recommended coding courses that are well-regarded and can enhance your programming skills:

1\. **Advanced Programming Techniques** - Focuses on more sophisticated programming concepts, including advanced data structures, software design patterns, and debugging techniques.

2\. **Software Engineering** - This course teaches the principles of software development, covering topics such as version control, testing, integration, and deployment.

3\. **Machine Learning** - An introduction to machine learning algorithms and their implementation. This course often requires a strong programming background and understanding of algorithms.

Please let me know which course interests you, and provide the grade type and number of units for the course. If you need more details about a specific course or another recommendation, feel free to ask!
User: I will take machine learning
Bot: Please specify the grade type (either ”Letter Grade” or ”Credit/No Credit”) and the number of units for the Machine Learning course. Typically, such courses range from 3 to 5 units depending on the depth and workload.
User: letter 3 units
Bot: Thank you for providing the details. Here’s a summary of the courses you’ve chosen to enroll in:

1\. **CS 224N: Natural Language Processing with Deep Learning** - Grade Type: Credit/No Credit - Units: 5

2\. **Machine Learning** - Grade Type: Letter Grade - Units: 3

Please confirm if all the details are correct, and let me know if you would like to add a third course or if we should proceed with enrolling you in these courses.
User: yes
Bot: Let’s proceed with your enrollment. Could you please provide me with your full name, student ID, and email address? This information is needed to complete your course enrollment.
User: john smith
Bot: Thank you, John. Could you please also provide your student ID and email address? This information is necessary to complete your course enrollment.
User: 234234 johnsmith@gmail.com
Bot: Your course enrollment has been successfully submitted. Here are the details:

- **Student Name:** John Smith - **Student ID:** 234234 - **Email:** johnsmith@gmail.com

**Courses Enrolled:** 1\. **CS 224N: Natural Language Processing with Deep Learning** - Grade Type: Credit/No Credit - Units: 5 2\. **Machine Learning** - Grade Type: Letter Grade - Units: 3

**Transaction ID:** a47d8ee3-1044-44d1-b1eb-e1884859abb3

If you have any more questions or need further assistance, feel free to ask. Good luck with your courses!
User: thanks
Bot: You’re welcome, John! If you need any more help in the future, feel free to reach out. Have a great time studying your courses!

#### H.1.9 GPT Ticket Submission Good Example

User: Hello
Bot: Hi there! How can I assist you today?
User: What is the enrollment process?
Bot: The enrollment likely at University involves several steps: 1\. **Check the Academic Calendar:** Look for the dates when enrollment opens. 2\. **Clear any Holds:** Make sure there are no enrollment holds on your account. 3\. **Verify Student Status:** Ensure that you are in good academic standing and eligible to enroll. 4\. **Use ”Service”:** Utilize the ”Service” tool to enroll in classes. 5\. **Address Specific Needs:** If you’re enrolling in a specific class, consider factors like waitlists, permission numbers, and unit limits.

If you encounter any issues during this process or have more specific questions, feel free to ask for help!
User: How do I verify student status?
Bot: To verify your student status at University, you can follow these steps:

1\. **Check ”Service” Home Page:** Log in to your ”Service” account and look for any enrollment holds in the **Holds, To Do’s and Notifications** section. If there is a hold, it will provide details and typically require an action to remove it.

2\. **Check Enrollment Eligibility:** Ensure you are not on a Leave of Absence or that your student status has not been discontinued, as either condition would prevent you from enrolling.

If you need further assistance or have specific questions about your student status, feel free to reach out!
User: What is ”Service” account?
Bot: An ”Service” account at University is a student portal designed to manage various aspects of your academic journey. Here’s what you can do with an ”Service” account:

- **Enroll in Classes:** You can search for, add, and drop courses. - **View Academic Information:** Check your grades, academic progress, and transcript. - **Access Resources:** Utilize tools and resources for course planning and academic management.

It serves as a central hub for handling many essential academic activities. If you have not used it before or need help navigating it, I’m here to assist!
User: What is a leave of absence?
Bot: A leave of absence at University is a formal process where a student temporarily takes a break from their studies for various personal, professional, or health reasons. Here are some key points about taking a leave of absence:

- **Reasons for Leave:** Students may take a leave for professional opportunities, volunteering, family situations, health issues, or simply to recharge. - **Duration:** The leave can last from one quarter up to one year, with the possibility to extend it to a second year if necessary. - **Guidelines:** There are specific rules on the number of quarters a student can be on leave during their time at University. - **Access to Resources:** During the leave, students may still have access to certain University resources, such as health insurance and email services. - **Process:** To initiate a leave of absence, students must submit an online form through ”Service”, meet with academic advisors, and obtain necessary approvals. - **Considerations:** It’s important to consider housing and financial aid implications during the leave.

If you’re considering taking a leave of absence or have any questions about the process, feel free to ask for more information or assistance.
User: What is the process to apply for leave of absence?
Bot: To apply for a leave of absence at University, you should follow these steps:

1\. **Access ”Service”:** - Log in to your ”Service” account.

2\. **Navigate to eForms:** - Go to the ”My Academics” dropdown menu. - Select “Student eForms.” - Click on “Browse Available Forms” and search for ”Leave of Absence for Undergraduates.”

3\. **Fill Out the Form:** - Complete the Leave of Absence eForm online.

4\. **Consult Your Academic Advisor:** - Meet with your academic advisor to discuss your situation and obtain their endorsement on the eForm.

5\. **Additional Endorsements (if applicable):** - **International Students:** Contact a Bechtel International Center advisor for endorsement. - **Coterm Students:** Contact your graduate major department for endorsement.

6\. **Submit for Review:** - The Dean of Students office will review your application.

7\. **Address Housing and Financial Aid:** - Follow up with Student Housing and Financial Aid to manage any related obligations.

Remember, simply submitting the form online is not sufficient. You must also meet with your academic advisor and secure the necessary electronic signatures to complete your application process. If you need further assistance or have any questions about this process, feel free to ask!

#### H.1.10 GPT Ticket Submission Bad Example

User: what is the enrollment status?
Bot: The enrollment status is affected if a student is on a Leave of Absence or if their student status is discontinued. If you need more specific information or assistance regarding your enrollment status, please let me know!
User: how long is the waitlist for a course?
Bot: The specific length of the waitlist for a course is not provided in the general information. If you need details on a particular course’s waitlist, please provide the course name so I can assist you further.
User: Where is the leave form status on the portal?
Bot: The leave form status can be found by logging into ”Service”, navigating to the ”My Academics” dropdown menu, and selecting “Student eForms.” Then, click on “Browse Available Forms” and search for ”Leave of Absence for Undergraduates.” This will allow you to view the status of your leave form. If you need further assistance with this process, feel free to ask!
User: what is the leave of absence process?
Bot: The leave of absence process at University involves several steps:

1\. **Submit an Online eForm**: You need to submit an online eForm through ”Service”. 2\. **Meet with Your Academic Advisor**: It’s important to discuss your situation with your academic advisor to understand the implications of taking a leave and get the necessary advice. 3\. **Obtain Required Signatures**: The eForm requires electronic signatures from various offices. Ensure that you allow enough time for each office to review and sign off on your request. 4\. **Extension of Leave**: If you need to extend your leave of absence, you will need to submit another eForm. 5\. **Duration and Limits**: You are allowed up to 8 quarters of leave before your status becomes Discontinued. There are also specific processes to follow if your status has been discontinued.

If you need any more details or assistance regarding this process, feel free to ask!
User: but the external test credits are not showing up, it’s a problem
Bot: Could you please specify the issue with your external test credits? For example, are the credits not posted, or are the units incorrect? Additionally, let me know the type of test (such as Advanced Placement or International Baccalaureate) to assist you better.
User: I need more information to help me about the external test credits, it doesn’t make sense to me so I need to submit a ticket for more help
Bot: To assist you better and submit a ticket regarding your issue with external test credits, I need to know the specific issue you’re facing. Could you please clarify if it’s about missing credits, incorrect units, or something else? Also, please specify the type of test involved (e.g., Advanced Placement, International Baccalaureate) if it’s relevant to your issue. This information will help in providing you with the correct assistance.
User: The issue is about the missing credits, incorrect units as well, the advanced placement test
Bot: Before I submit a ticket for you regarding the issue with your external test credits for the Advanced Placement test, where credits are missing and units are incorrect, could you please confirm:

1\. **Your Name**: To include in the ticket. 2\. **Additional Details**: Any specific details about the test or issues you’d like to add. 3\. **Time of Test Score Submission**: When did you submit your test scores?

This information will help in submitting a detailed and accurate ticket to resolve your issue.
User: My name is Valerie, the issue is the advanced placement test with incorrect units, the time I submitted my test scores was today
Bot:
User: Could I submit a ticket with that information please?
Bot: The ticket regarding the issue with your Advanced Placement test credits has been successfully submitted. If you need further assistance or have any more questions, feel free to ask.

#### H.1.11 GPT Restaurant Reservation Good Example

User: Hi, I’d like to book a restaurant for tonight in San Francisco
Bot: Great choice! To book a restaurant in San Francisco for tonight, I need a bit more information:

1\. What restaurant would you like to book? 2\. What time would you like your reservation? 3\. For how many people is the booking? 4\. Do you have any special requests?

If you’re not sure about which restaurant, I can help you find one. Just let me know your preferences!
User: I don’t know what restaurant I’d like to eat at. Can you help me pick something with lots of good reviews?
Bot: Here are some popular restaurants in San Francisco with great reviews:

1\. **Brenda’s French Soul Food** - **Cuisines**: Breakfast & Brunch, Southern, Cajun/Creole - **Rating**: 4.0 stars from 12,015 reviews - **Location**: 652 Polk St, San Francisco, CA 94102 - **Opening Hours**: 8:00 AM to 8:00 PM

2\. **Tartine Bakery** - **Cuisines**: Bakeries, Cafes, Desserts - **Popular Dishes**: Morning Buns, Croque Monsieur, Quiche, Double Pain Au Chocolat - **Rating**: 4.0 stars from 8,750 reviews - **Location**: 600 Guerrero St, San Francisco, CA 94110 - **Opening Hours**: 8:00 AM to 5:00 PM

3\. **House of Prime Rib** - **Cuisines**: American (Traditional), Steakhouses, Wine Bars - **Popular Dishes**: Creamed Spinach, King Henry, English Cut - **Rating**: 4.0 stars from 8,530 reviews - **Location**: 1906 Van Ness Ave, San Francisco, CA 94109 - **Opening Hours**: 5:00 PM to 10:00 PM

Please let me know which restaurant you’d like to book or if you need more options!
User: Find me something affordable with lots of good reviews and
Bot: Here are some affordable restaurants in San Francisco with excellent reviews:

1\. **Saigon Sandwich** - **Cuisines**: Vietnamese, Sandwiches - **Rating**: 4.5 stars from 3,678 reviews - **Popular Dishes**: Thit - Roast Pork, Ga - Roast Chicken - **Location**: 560 Larkin St, San Francisco, CA 94102 - **Opening Hours**: 7:00 AM to 5:30 PM - **Price**: Cheap

2\. **Good Mong Kok Bakery** - **Cuisines**: Bakeries, Dim Sum, Cantonese - **Rating**: 4.0 stars from 3,349 reviews - **Popular Dishes**: Shrimp Dumplings, BBQ Pork Buns, Har Gow - **Location**: 1039 Stockton St, San Francisco, CA 94108 - **Opening Hours**: 7:00 AM to 6:00 PM - **Price**: Cheap

3\. **Yamo** - **Cuisines**: Burmese - **Rating**: 4.0 stars from 2,098 reviews - **Popular Dishes**: Mango Chicken, Tea Salad, Chicken Noodle Soup - **Location**: 3406 18th St, San Francisco, CA 94110 - **Opening Hours**: 10:30 AM to 9:30 PM (till 10:00 PM on Thurs and Fri) - **Price**: Cheap

Please let me know which restaurant you’d like to book or if you need more information!
User: Saigon Sandwich sounds perfect!!
Bot: Great choice! Saigon Sandwich is highly popular for their delicious sandwiches. Let’s proceed with the booking for tonight.

Could you please provide the following details: 1\. What time would you like your reservation? 2\. For how many people is the booking? 3\. Do you have any special requests for your visit?
User: Actually, I’m sorry, but I’ve changed my mind. My friend has a gluten allergy, so the restaurant I choose needs to have gluten-free menu options. Anything similar to saigon sandwich at a similar price range?
Bot: I found a great place that offers affordable gluten-free options:

**Ben’s Fast Food** - **Cuisines**: American (New), Fast Food, Gluten-Free - **Rating**: 4.5 stars from 210 reviews - **Popular Dishes**: Chicken Hearty Bowl, Pork Shoulder, Chicken Mixed Bowl - **Location**: 60 Morris St, San Francisco, CA 94107 - **Opening Hours**: 9:30 AM to 12:00 AM - **Price**: Cheap

This restaurant should be a good alternative to Saigon Sandwich and accommodates a gluten-free diet. Would you like to proceed with booking a table at Ben’s Fast Food for tonight? If so, please provide the reservation time and the number of people.
User: Does this place have a drive-thru?
Bot: It appears that Ben’s Fast Food does not have a drive-thru. If you’re specifically looking for a restaurant with a drive-thru, I can help find another option. Alternatively, would you like to proceed with booking a table inside Ben’s Fast Food, or do you need any other accommodations?
User: Let’s proceed with Ben’s Fast Food
Bot: Great! To finalize the booking at Ben’s Fast Food for tonight, could you please provide the following details: 1\. What time would you like your reservation? 2\. For how many people is the booking? 3\. Do you have any special requests for your visit?
User: What is their latest available reservation time for 2 people?
Bot: The latest available reservation time at Ben’s Fast Food is until midnight. Would you like to book a table for 2 people close to this time? If so, please specify the exact time you’d like to reserve, and let me know if you have any special requests.
User: Book a table for two for 11pm. We need space for two wheelchairs.
Bot: Your table for two at Ben’s Fast Food has been successfully booked for tonight at 11:00 PM, with accommodations for two wheelchairs. Your transaction ID is **8ac5f435-9393-4da9-828d-968702701a27**.

If you need anything else or have further questions, feel free to ask. Enjoy your meal!
User: Can I change this reservation?
Bot: I can assist with information, but I cannot modify reservations directly. Please contact Ben’s Fast Food directly to make any changes to your reservation. You can reach them at (415) 644-8377.

If you need any other information or assistance, feel free to ask!
User: Darn. Thanks for your help.
Bot: You’re welcome! If you have any more questions in the future or need further assistance, feel free to reach out. Enjoy your evening and your dining experience at Ben’s Fast Food!

#### H.1.12 GPT Restaurant Reservation Bad Example

User: Best BBQ in San Francisco?
Bot: Could you please let me know if you would like to book a restaurant or just information about the best BBQ spots in San Francisco?
User: Find me a reservation at the best BBQ joint in SF.
Bot: I’ll find you the best BBQ restaurant in San Francisco. Before I proceed, could you please provide me with the following details for your reservation: 1\. Date of the reservation (e.g., December 15, 2023) 2\. Time you prefer to dine (e.g., 7:00 PM) 3\. Number of people in your party

Once I have these details, I can check availability for you.
User: July 4th, 2024
Bot: Great! You’ve chosen July 4th, 2024\. Now, could you please provide the following details: 1\. Preferred dining time (e.g., 7:00 PM) 2\. Number of people in your party

This will help me finalize the booking for you.
User: 06:30PM
Bot: Thank you for specifying the time as 06:30 PM. Now, could you please let me know how many people will be dining with you?
User: le
Bot: It seems there might have been a typo. Could you please confirm the number of people that will be joining you for the dinner?
User: 2 people
Bot: Thank you for providing all the details. Here’s a summary of your reservation request:

- **Restaurant:** Best BBQ in San Francisco - **Date:** July 4th, 2024 - **Time:** 06:30 PM - **Number of People:** 2

Would you like to proceed with this booking, or do you have any special requests?
User: proceed
Bot:
User: Yes
Bot: Your reservation at the best BBQ restaurant in San Francisco has been successfully booked. Here are the details:

- **Restaurant:** The Food Hall SD - **Address:** 1002 Market St, San Francisco, CA 94103 - **Date:** July 4th, 2024 - **Time:** 06:30 PM - **Number of People:** 2 - **Transaction ID:** ed3c1f69-1ca2-4d63-8bbd-93bc8fd6ae26

Enjoy your BBQ experience! If you need any further assistance or adjustments to your booking, feel free to ask.

## Appendix I Worksheet

We provide access to the 6 worksheets created for the 3 StarV2 applications in Section [1](https://arxiv.org/html/2407.05674v2#S4.T1 "Table 1 ‣ 4 Evaluation on Static Datasets ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") and the 3 real-world applications in Section [5](https://arxiv.org/html/2407.05674v2#S5 "5 Evaluation on Real User Studies ‣ Coding Reliable LLM-based Integrated Task and Knowledge Agents with GenieWorksheets") in the supplementary material with our submission.