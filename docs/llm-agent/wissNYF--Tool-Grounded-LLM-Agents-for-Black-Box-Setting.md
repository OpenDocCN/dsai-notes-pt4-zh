<!--yml
category: 未分类
date: 2025-01-11 12:53:46
-->

# wissNYF: Tool Grounded LLM Agents for Black Box Setting

> 来源：[https://arxiv.org/html/2402.10051/](https://arxiv.org/html/2402.10051/)

Somnath Sendhil Kumar¹  Dhruv Jain¹  Eshaan Agarwal¹  Raunak Pandey¹  
¹ [Intelligence Group, IIT (BHU), Varanasi](https://cops-iitbhu.github.io/IG-website/)

###### Abstract

While Large Language Models (LLMs) have demonstrated enhanced capabilities in function-calling, these advancements primarily rely on accessing the functions’ responses. This methodology is practical for simpler APIs but faces scalability issues with irreversible APIs that significantly impact the system, such as a database deletion API. Similarly, processes requiring extensive time for each API call and those necessitating forward planning, like automated action pipelines, present complex challenges. Furthermore, scenarios often arise where a generalized approach is needed because algorithms lack direct access to the specific implementations of these functions or secrets to use them. Traditional tool planning methods are inadequate in these cases, compelling the need to operate within black-box environments. Unlike their performance in tool manipulation, LLMs excel in black-box tasks, such as program synthesis. Therefore, we harness the program synthesis capabilities of LLMs to strategize tool usage in black-box settings, ensuring solutions are verified prior to implementation. We introduce TOPGUN, an ingeniously crafted approach leveraging program synthesis for black box tool planning. Accompanied by SwissNYF, a comprehensive suite that integrates black-box algorithms for planning and verification tasks, addressing the aforementioned challenges and enhancing the versatility and effectiveness of LLMs in complex API interactions. The public code for SwissNYF is available at  [https://github.com/iclr-dummy-user/SwissNYF](https://github.com/iclr-dummy-user/SwissNYF)

## 1 Introduction

![Refer to caption](img/4c4fe3802f77a55527be1fe6ae08a089.png)

Figure 1: Illustration of different settings that an LLMs may require to manipulate tools.

Significant advancements in Large Language Models (LLMs) like GPT (Radford et al. ([2018](https://arxiv.org/html/2402.10051v1#bib.bib26)); Radford et al. ([2019](https://arxiv.org/html/2402.10051v1#bib.bib27)); Brown et al. ([2020](https://arxiv.org/html/2402.10051v1#bib.bib4)); Achiam et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib1))) and PaLM (Chowdhery et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib8));Anil et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib2));) have demonstrated profound abilities in reasoning and following instructions over an extensive array of tasks Huang & Chang ([2023](https://arxiv.org/html/2402.10051v1#bib.bib14)). The recent shift towards leveraging LLMs to interact with external tools for addressing complex real-world challenges marks a significant area of interest (Hao et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib13)); Zhang et al. ([2023a](https://arxiv.org/html/2402.10051v1#bib.bib36)); Zhuang et al. ([2023b](https://arxiv.org/html/2402.10051v1#bib.bib42)); Yang et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib33)); Schick et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib28));Lu et al. ([2023a](https://arxiv.org/html/2402.10051v1#bib.bib17));). In addressing intricate problems, autonomous agents powered by LLMs employ an amalgamation of LLMs and various external tools (APIs), crafting solutions that necessitate a sequence of intermediate reasoning steps (Schick et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib28));Lu et al. ([2023a](https://arxiv.org/html/2402.10051v1#bib.bib17));Lu et al. ([2023a](https://arxiv.org/html/2402.10051v1#bib.bib17));Patil et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib24));Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25))). When presented with a problem, These agents’ primary objective is to identify and execute a series of API function calls sequentially, leading to a coherent solution. These approaches are ineffective when queries lack transparency or when the APIs are irreversible.

We coin the term "black-box" settings in the context of tool planning as scenarios where the outcomes of an API or tool are not observable. This framework is especially pertinent in systems where using certain APIs poses risks, such as those causing inconsistencies by deleting or updating database entries, canceling jobs, or performing similar operations. It’s also relevant where API experimentation incurs high costs or when APIs require considerable time to execute, ensuring clarity and comprehensive coverage without redundancy, making it challenging to interpret their outcomes. We present a taxonomy of such systems Fig. [1](https://arxiv.org/html/2402.10051v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") into three branches:

1.  1.

    White Box Systems: In these settings, planners can invoke the API, receive responses, access the source code and understand its complex logic. This access enables the system to navigate complex inputs, intricacies and use cases efficiently.

2.  2.

    Gray Box Systems: Planners in these environments have descriptions of the tools at their disposal and the capability to call the API and receive responses. The system’s planning relies solely on the limited descriptions provided and the responses for each tool.

3.  3.

    Black Box Systems: In the most challenging scenarios, planners are confined to tool descriptions without access to actual tool outputs. Here, the planner must decipher the dynamics of each tool based solely on its description, making it a particularly demanding task to formulate responses to queries.

The Zhuang et al. ([2023a](https://arxiv.org/html/2402.10051v1#bib.bib41)) and Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25)) methods excel in straightforward scenarios where an agent can iterate over tools to identify the optimal path, yet they lack efficiency and necessitate extensive exploration. Approaches like Yao et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib34)) and Parisi et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib22)), subsets of this exploratory paradigm, offer enhanced efficiency yet frequently falter due to their constrained directionality in tool search, making them suitable predominantly for straightforward API challenges. In contrast, the Zhang et al. ([2023b](https://arxiv.org/html/2402.10051v1#bib.bib37)) approach is efficient regarding API execution costs by constraining the number of calls. However, it omits any form of verification for its proposed trajectory, diminishing its precision in practical applications.

These methodologies in tool application present a dichotomy between accuracy and computational overhead. While generally unsuitable for black-box settings, the Reverse chain approach exhibits potential for adaptation within such frameworks. On the other hand, program synthesis-based algorithms have been instrumental in exalting reasoning and decision-making capabilities within LLMs, offering a more naturally associative decision-making process than that afforded by mere text. Works like The Chain of Code Li et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib15)) and Program-of-thoughts Chen et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib7)) are great examples of using code generation to improve decision-making for answering general open-domain questions. To this end, few works also upheld the reasoning capability of LLMs using code like "TORA: A Tool-Integrated Reasoning Agent for Mathematical Problem Solving" Gou et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib12)), "Solving challenging math word problems using gpt-4 code interpreter with code-based self-verification" Zhou et al. ([2023b](https://arxiv.org/html/2402.10051v1#bib.bib40)) and "PAL: Program-aided Language Models" Gao et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib11)) have exploited code interpreters for zero-shot verified solving, substantially surpassing few-shot learning benchmarks by enabling semi-verification of proposed solutions.

However, works like Paranjape et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib21)), which employs code synthesis for tool usage, are restricted by their limited toolset and the scalability challenge posed by the need for extensive human feedback and interventions and the need for the human expert to be familiar with the whole toolset. Similarly, works such as Xu et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib32)), which deploys language models for real-time code generation and command execution within controlled environments, are limited by their narrow tool range and a deficit in generalizability. The state-of-the-art approaches on HumanEval Chen et al. ([2021](https://arxiv.org/html/2402.10051v1#bib.bib6)) and HumanEval-X Zheng et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib38)) datasets for code generation, like Reflexion Shinn et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib29)) and LATS Zhou et al. ([2023a](https://arxiv.org/html/2402.10051v1#bib.bib39)), which iterate upon code based on interpreter outputs and reflect over them, these approaches have yet to be experimented with in other domains associated with LLMs.

To bridge these gaps, we introduce the TOPGUN (Tool Orchestration and Program synthesis for Generalizing over UNknown systems) framework, which unifies code generation, reasoning, and strategic tool planning designed for complex tasks. TOPGUN also verifies the execution plans and does so with exceptional efficiency in API cost, effectively addressing the limitations of preceding models.

Key contributions of our work are summarized as follows:

1.  1.

    To the best of our knowledge, we are the First to coin the term Black Box setting for API usage and developed a suite to encourage the development of algorithms for such scenarios.

2.  2.

    We leverage the program synthesis capabilities of Large Language Models (LLMs) to augment their efficacy in tool usage substantially, showcasing a notable enhancement in performance.

3.  3.

    We present a robust and cost-efficient framework for scalable solutions across a wide array of open-domain queries, even when faced with limited knowledge of user data/tools. It is also publically hosted to demonstrate the same.¹¹1[https://swiss-nyf.azurewebsites.net/](https://swiss-nyf.azurewebsites.net/)

This paper details our methodology and its evaluation by first elucidating the background on Tool planning [2.1](https://arxiv.org/html/2402.10051v1#S2.SS1 "2.1 Problem Formulation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") and Code generation using LLM [2.2](https://arxiv.org/html/2402.10051v1#S2.SS2 "2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") followed by detailing individual components of the pipeline [3](https://arxiv.org/html/2402.10051v1#S3 "3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). Our evaluation is bifurcated into two segments: initially, we undertake a gray box [4.1](https://arxiv.org/html/2402.10051v1#S4.SS1 "4.1 Gray Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") across principal datasets, and subsequently, we delve into a black box setting [4.2](https://arxiv.org/html/2402.10051v1#S4.SS2 "4.2 Black Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). For the latter, we have curated a bespoke dataset employing Toolbench prompts, intentionally adjusting the dataset to include only limited documentation of widely used libraries. This adjustment aims to validate the generalizability of our approach. Additionally, we juxtapose our methodology with a tailored variant of the Reverse Chain method to scrutinize performance disparities.

## 2 Preliminaries

### 2.1 Problem Formulation

![Refer to caption](img/713246da1703b1bdbee92306537d5089.png)

Figure 2: Illustration of SwissNYF pipeline for tool usage in Black Box setting.

Tool planning within the context of a Large Language Model (LLM), denoted as $\rho$, involves leveraging a selection of tools from a pool of $n$ candidate tools in the corpus $\mathcal{C}$, represented as $\mathcal{C}=\{t_{0},t_{1},\ldots,t_{n}\}$, to effectively address a user’s query $q$. The primary goal is to formulate a meticulous plan, known as the Solution Trajectory $St$, for the orchestration of these tools. The Solution Trajectory $St$, which outlines the sequential execution of tools, is crafted to directly address the query $q$. The LLM agent, or planner $\mathcal{G}$, is responsible for planning or generating $St$ from $\mathcal{C}$, formalized as $St\leftarrow\mathcal{G}(q,\rho,\mathcal{C})$. This process ensures a structured and coherent response strategy, aligning the tools’ capabilities with the query’s specific requirements for an effective solution.

### 2.2 Code Generation

The integration of Reflexion Shinn et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib29)) with Large Language Models (LLM) $\rho$ and Python Interpreter $\mathcal{I}$ has significantly advanced coding tasks by enabling iterative code refinement. This approach leverages feedback $\mathcal{F}$ to iteratively address exceptions and enhance initial code output $c$, guided by test cases dynamically generated by $\rho$ itself. This ensures comprehensive verification and refinement within a Function Call module, leading to a finalized code $c_{n}$. This methodology enhances code quality and aligns with contemporary standards, marking a leap in automated code development and verification. This process of iterative code generation can be mathematically denoted as Eq. [1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")

|  | $\begin{gathered}c_{i}\leftarrow\rho(q,\textit{feedback}_{i-1},c_{i-1})\\ \textit{output}\leftarrow\mathcal{I}(c_{i})\\ \textit{feedback}_{i},\textit{verified}\leftarrow\mathcal{F}(output)\\ \end{gathered}$ |  | (1) |

## 3 SwissNYF

### 3.1 Overview

![Refer to caption](img/404ffdeabee906e3a5be1b13cf61a3a9.png)

Figure 3: Detailed pipeline of our proposed approach with TOPGUN in SwissNYF

In this section, we introduce SwissNYF, a suite that enables LLM-based agents to efficiently navigate the action space to identify a valid solution for problem-solving in a black box scenario. SwissNYF is composed of five major components i.e., Function Signature Generation $\mathcal{P}$, Corpus & Retriever $\mathcal{C}$, Planner $\mathcal{G}$, Verifier $\mathcal{F}$ and Parser $\mathcal{K}$ as in Fig. [2](https://arxiv.org/html/2402.10051v1#S2.F2 "Figure 2 ‣ 2.1 Problem Formulation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). We explain individual components of the pipeline in the subsequent subsections.

![Refer to caption](img/e61dfca571a01546cdbc1fbe222a6ba9.png)

(a) Example output of CodeSynth $\mathcal{P}$ Algorithm

![Refer to caption](img/bf57704e267a1ba6dc7ce250d54eafb4.png)

(b) Example output of TOPGUN $\mathcal{G}$ Algorithm

Figure 4: Illustration of pseudo function and tool planning generated by CodeSynth and TOPGUN, respectively.

### 3.2 Function Signature Generation

Function signatures, conceptualized as pseudo APIs, serve to emulate the behaviour of real API functions based on given tool descriptions. This emulation is crucial for two primary reasons in our tool planning methodology: firstly, they act as stand-ins for actual API calls, thereby enabling LLMs to plan and execute tasks with higher efficiency; secondly, they are treated as pre-defined functions, facilitating the transformation of tool augmentation into a task akin to code generation, using these pseudo functions. These function signatures are distinguished by their docstrings and an example return object that aligns with the tool description, equipping the planner with the necessary means to effectively address user queries. In the context of our SwissNYF implementation, we have adopted a straightforward yet effective method for generating these function signatures, termed CodeSynth. The efficacy of this approach is further analyzed in [4.3](https://arxiv.org/html/2402.10051v1#S4.SS3 "4.3 CodeSynth Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting").

#### 3.2.1 CodeSynth

For a given set of tool descriptions $t\in\mathcal{T}$, we direct the Large Language Model (LLM) $\rho$ to generate pseudo-function implementations, denoted as $\hat{t}$. Our primary objective is to ensure that the arguments and return types of these pseudo-functions remain consistent with their descriptions. Additionally, we craft detailed docstrings for each pseudo-function to facilitate subsequent processes. A critical aspect of CodeSynth is the inclusion of an example return value, which is designed to mimic all potential operations the returned object might undergo during the verification process. The output generated by CodeSynth is illustrated in Fig. [3(a)](https://arxiv.org/html/2402.10051v1#S3.F3.sf1 "3(a) ‣ Figure 4 ‣ 3.1 Overview ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). Moreover, the code generation facilitated by this block benefits from validation through Reflexion, as outlined in Eq. [1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). Ultimately, the methodologies applied within CodeSynth can be encapsulated in Algo. [1](https://arxiv.org/html/2402.10051v1#algorithm1 "1 ‣ 3.2.1 CodeSynth ‣ 3.2 Function Signature Generation ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting").

Input: $\rho$: large language model; $T$: tool descriptions; $\mathcal{I}$: python interpreter; $\mathcal{F}(\mathcal{I})$: reflexion feedback of $\mathcal{I}$; $\mathcal{C}$: empty corpus of pseudo toolsfor *$t=1,2,\cdots,T$* do       $\hat{t}_{0}\leftarrow\rho(t)$          //Pseudo code       $verified\leftarrow\mathcal{I}(\hat{t}_{0})$       while *not verified* do             $\hat{t}_{i}\leftarrow\rho(t,feedback_{i-1},\hat{t}_{i-1})$             $feedback_{i},\ verified\leftarrow\mathcal{F}(\mathcal{I}(\hat{t}_{i}))$      Update $\mathcal{C}\leftarrow\hat{t}_{n}$      // Update CorpusOutput: A corpus of verified psuedo functions $\mathcal{C}$

Algorithm 1 $\mathcal{P}$: CodeSynth

Utilizing the Function Calling module alongside the Interpreter, we rigorously test the pseudo-functions against a wide range of real-world scenarios. This approach guarantees that the test cases are comprehensive and reflective of actual function usage, allowing us to gather detailed feedback on the pseudo-functions’ performance. Such feedback is vital for the iterative improvement of the pseudo-functions, significantly enhancing their reliability and applicability in practical settings. Prompts for CodeSynth can be documented in [A.1](https://arxiv.org/html/2402.10051v1#A1.SS1 "A.1 Prompts ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting").

### 3.3 Corpus and Retriever

The function signatures, crucial components of our methodology, are systematically stored within a corpus for future utilization by any planning system. This corpus facilitates the indexing of tool descriptions, enabling the precise retrieval of the most appropriate tool based on the index. Notably, the literature documents several advanced retrieval systems designed for this purpose, demonstrating exceptional accuracy. These include ToolBench IR Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25)), APIRetriever Zan et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib35)), Instructor-XL Su et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib30)), and GEAR Lu et al. ([2023b](https://arxiv.org/html/2402.10051v1#bib.bib18)). Our framework incorporates these retrievers, with Instructor-XL set as the default option, owing to its proven efficacy. Furthermore, we are actively exploring the integration of AnyTool’s Hierarchical API Retriever Du et al. ([2024](https://arxiv.org/html/2402.10051v1#bib.bib9)), anticipating significant enhancements to our tool retrieval capabilities. This strategic inclusion of multiple retrievers ensures our system remains versatile and effective in identifying the most suitable tools for a given task, aligning with the latest advancements in retrieval technology.

### 3.4 Planner

We have implemented two planning approaches in our framework. The first leverages a modified Reverse Chain Zhang et al. ([2023b](https://arxiv.org/html/2402.10051v1#bib.bib37)) to support multiple end function calls by decomposing tasks into subtasks and creating sub-trees with the original reverse chain technique. The second, TOPGUN, is our proposed code-driven planning algorithm, designed for speed, efficiency, consistency, and accuracy, especially in black box scenarios. TOPGUN offers a streamlined alternative to traditional planning methods, optimizing for complex system navigation and task execution with greater reliability and cost-effectiveness.

#### 3.4.1 TOPGUN

Input: q: query; $\rho$: large language model; $T$: tool descriptions; $\mathcal{I}$: python interpreter; $\mathcal{F}(\mathcal{I})$: reflexion feedback of $\mathcal{I}$; $\mathcal{C}$: empty corpus of pseudo tools; $\mathcal{P}$: Codesynth, $\mathcal{K}$: parserInitialize $\mathcal{\hat{T}}\leftarrow\mathcal{P}(\rho,T,\mathcal{I},\mathcal{F},\mathcal% {C})$  // Pseudo tools$c_{0}\leftarrow\ \rho(q,\mathcal{\hat{T}},\mathcal{C})$        // Code for query$verified\leftarrow\mathcal{I}(c_{0},\mathcal{\hat{T}})$  // Verify with pseudo tools while *not verified* do       $c_{i}\leftarrow\ \rho(q,\mathcal{\hat{T}},\mathcal{\hat{C}},feedback_{i-1},c_{% i-1})$       $feedback_{i},\ verified\leftarrow\mathcal{F}(\mathcal{I}(c_{i},\mathcal{\hat{T% }}))$$\mathcal{S}t\leftarrow\mathcal{K}(c_{n})$        // Solution TrajectoryOutput: A solution trajectory $\mathcal{S}t$ and $c_{n}$ code for execution and evaluation

Algorithm 2 $\mathcal{G}$: TOPGUN

TOPGUN, an acronym for Tool Orchestration and Program synthesis for Generalizing over UNknown systems, redefines the approach to addressing user queries $q$ by framing the challenge as a task of code generation. Utilizing pseudo-functions $\mathcal{\hat{T}}$ as functions available to TOPGUN enables the agent to construct an accurate sequence of function calls $c_{0}\leftarrow\rho(q,\mathcal{\hat{T}},\mathcal{C})$, effectively depicted in Fig. [3(b)](https://arxiv.org/html/2402.10051v1#S3.F3.sf2 "3(b) ‣ Figure 4 ‣ 3.1 Overview ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). Leveraging Reflexion detailed in Eq.[1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"), the framework iteratively refines responses to the query. The synthesis of these components into the comprehensive algorithm is presented in Algo. [2](https://arxiv.org/html/2402.10051v1#algorithm2 "2 ‣ 3.4.1 TOPGUN ‣ 3.4 Planner ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") showcases TOPGUN’s capability to navigate through various solution paths. Unlike traditional traversal-based techniques, TOPGUN capitalizes on the inherent code-generation capabilities of LLMs, facilitating a more direct and efficient solution process. This distinction not only enhances efficacy by pinpointing issues with precision but also ensures adaptability in black box scenarios, simultaneously optimizing performance in gray box settings. A detailed pipeline overview with TOPGUN in place is given in Fig.[3(b)](https://arxiv.org/html/2402.10051v1#S3.F3.sf2 "3(b) ‣ Figure 4 ‣ 3.1 Overview ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). With prompts documented in [A.1](https://arxiv.org/html/2402.10051v1#A1.SS1 "A.1 Prompts ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting").

![Refer to caption](img/d47a80216d520a52d217f9809b6096fb.png)

Figure 5: Illustartion of Self-Reflection Mechanism in TOPGUN

### 3.5 Verifier

Verification is closely linked to the functionality of the Planner $\mathcal{G}$, relying on both the nature of $\mathcal{G}$’s output and its ability to incorporate feedback. Although verification initially serves as a preparatory step prior to parsing, it also plays a crucial role in refining outputs by providing feedback that $\mathcal{G}$ can use for subsequent iterations.

In our framework, we leverage Reflexion Shinn et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib29)), detailed in Eq. [1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") and depicted in Algo. [2](https://arxiv.org/html/2402.10051v1#algorithm2 "2 ‣ 3.4.1 TOPGUN ‣ 3.4 Planner ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"), to seamlessly integrate verification and feedback within the TOPGUN methodology. This eliminates the requirement for an additional function call module, concentrating instead on directly executing code pertinent to the user query. This approach is illustrated in Fig. [5](https://arxiv.org/html/2402.10051v1#S3.F5 "Figure 5 ‣ 3.4.1 TOPGUN ‣ 3.4 Planner ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"), providing a visual representation of the concept.

### 3.6 Parser

The Parser $\mathcal{K}$, akin to the Verifier $\mathcal{F}$, is intrinsically dependent on the Planner $\mathcal{G}$ for its functionality. Its pivotal output is a well-defined Solution Trajectory $St$, mapping out the sequence of tool applications devised to address the query. In employing the Reverse Chain technique, our methodology involves synthesizing individual sub-trees into a singular, comprehensive tree through the capabilities of LLM $\rho$. The process’s efficacy is markedly improved by the judicious reuse of elements from the individual trees during their amalgamation.

Conversely, for the TOPGUN methodology, we adopt the established Abstract Syntax Tree (AST) paradigm Fischer et al. ([2007](https://arxiv.org/html/2402.10051v1#bib.bib10)) to segment the program into fundamental function calls, alongside specifying their arguments and return values. This segmentation is instrumental in constructing a systematic series of tool invocations. This meticulously arranged series, denoted as $St$, is succinctly formalized as $St\leftarrow\mathcal{K}(c_{n})$.

The entire pipeline, as depicted in Figure [3](https://arxiv.org/html/2402.10051v1#S3.F3 "Figure 3 ‣ 3.1 Overview ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"), emerges from the integration of various components designed to effectively address user queries through the strategic orchestration of tools within the SwissNYF framework.

Table 1: Win Rate of different Candidate and Reference model over G1 set

| Candidate | Reference | G1-Instruction | G1-Tool | G1-Category |
| T.LLaMA ReACT | ChatGPT ReACT | 45.0 | 42.0 | 47.5 |
| T.LLaMA DFSDT | ChatGPT ReACT | 55.0 | 55.3 | 54.5 |
| T.LLaMA DFSDT+Ret | ChatGPT ReACT | 62.3 | 59.0 | 55.0 |
| ChatGPT DFSDT | ChatGPT ReACT | 60.5 | 62.0 | 57.3 |
| GPT4 ReACT | ChatGPT ReACT | 60.0 | 58.8 | 63.5 |
| GPT4 DFSDT | ChatGPT ReACT | 67.5 | 67.8 | 66.5 |
| GPT4 TOPGUN | ChatGPT ReACT | 88.192 | 87.46 | 87.15 |
| GPT4 TOPGUN | ChatGPT DFSDT | 78.49 | 77.55 | 76.24 |
| GPT4 TOPGUN | T.LLaMA ReACT | 86.72 | 82.94 | 80.80 |
| GPT4 TOPGUN | T.LLaMA DFSDT | 81.75 | 75.51 | 73.81 |
| GPT4 TOPGUN | T.LLaMA DFSDT+Ret | 80.35 | 77.11 | 75.39 |
| GPT4 TOPGUN | GPT4 ReACT | 82.996 | 79.956 | 77.633 |
| GPT4 TOPGUN | GPT4 DFSDT | 82.065 | 73.69 | 71.14 |

## 4 Experiments

Tool planning datasets, while diverse, often fall short in supporting multi-turn and multi-call dialogues, as seen in works by Schick et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib28)) and Tang et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib31)), and lack precise evaluation metrics, complicating thorough assessments. Even comprehensive datasets like ToolBench by Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25)) struggle with aligning to black-box settings, presenting significant challenges for evaluating tool planning in such scenarios.

Our evaluation employs the ToolBench benchmark Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25)) and a specially curated dataset for unchar codebases, assessed in both gray ([4.1](https://arxiv.org/html/2402.10051v1#S4.SS1 "4.1 Gray Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")) and black box ([4.2](https://arxiv.org/html/2402.10051v1#S4.SS2 "4.2 Black Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")) settings. We benchmark our TOPGUN approach against existing methods using win rate, token count, and success rate. Additionally, we scrutinize CodeSynth’s ($\mathcal{P}$) impact on the Planner’s ($\mathcal{G}$) performance and independently evaluate its ability to generate effective function signatures, acting as pseudo functions, detailed in Section [4.3](https://arxiv.org/html/2402.10051v1#S4.SS3 "4.3 CodeSynth Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting").

Table 2: Win Rate of different Candidate and Reference model over G2, G3 set and Average over all sets

| Candidate | Reference | G2-Instruction | G2-Category | G3-Instruction | Average |
| T.LLaMA ReACT | ChatGPT ReACT | 50.8 | 41.8 | 55.0 | 47.0 |
| T.LLaMA DFSDT | ChatGPT ReACT | 68.5 | 58.0 | 69.0 | 60.0 |
| T.LLaMA DFSDT+Ret | ChatGPT ReACT | 68.5 | 60.8 | 73.0 | 63.1 |
| ChatGPT DFSDT | ChatGPT ReACT | 72.0 | 64.8 | 69.0 | 64.3 |
| GPT4 ReACT | ChatGPT ReACT | 65.8 | 60.3 | 78.0 | 64.0 |
| GPT4 DFSDT | ChatGPT ReACT | 73.3 | 63.3 | 84.0 | 70.4 |
| GPT4 TOPGUN | ChatGPT ReACT | 87.59 | 78.78 | 90.05 | 86.54 |
| GPT4 TOPGUN | ChatGPT DFSDT | 81.63 | 73.07 | 85.26 | 78.71 |
| GPT4 TOPGUN | T.LLaMA ReACT | 86.24 | 77.71 | 93.23 | 84.61 |
| GPT4 TOPGUN | T.LLaMA DFSDT | 78.31 | 71.80 | 89.47 | 78.44 |
| GPT4 TOPGUN | T.LLaMA DFSDT+Ret | 83.07 | 72.92 | 87.82 | 79.44 |
| GPT4 TOPGUN | GPT4 ReACT | 78.61 | 73.75 | 93.68 | 80.27 |
| GPT4 TOPGUN | GPT4 DFSDT | 73.92 | 71.35 | 79.25 | 78.59 |

### 4.1 Gray Box Evaluation

To assess the performance of TOPGUN and compare it with other gray box methodologies such as ReACT and DFSDT, we maintain the integrity of our pipeline while adapting the evaluation process to incorporate actual functions in place of pseudo functions within the output solution trajectory. This approach effectively leaves our black box pipeline intact while converting it into a gray box evaluation framework. The necessity of responses and Final answers for evaluation purposes has led us to adopt this hybrid strategy. In practical scenarios, this mirrors the process where a generalist planner delivers a strategy to the client, who then substitutes pseudo-function implementations with their real functions for execution. For this evaluation, we employ ToolBench, as detailed by Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25)), and conduct our analysis across all problem categories provided in the dataset. Further elaboration on the precise evaluation methodology and the application of ToolBench is documented in [A.2](https://arxiv.org/html/2402.10051v1#A1.SS2 "A.2 ToolBench for Gray Box Evaluation ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting").

Results : Win Rate comparisons for ToolLLaMa-ReACT, ToolLLaMA-DFSDT, ChatGPT-DFSDT, GPT4-DFSDT, and GPT4-TOPGUN against ChatGPT-ReACT and GPT4-TOPGUN are summarized, with averages taken from 7 runs per model pair, detailed in Tables [1](https://arxiv.org/html/2402.10051v1#S3.T1 "Table 1 ‣ 3.6 Parser ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") and [2](https://arxiv.org/html/2402.10051v1#S4.T2 "Table 2 ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). TOPGUN significantly surpassed ReAct and DFSDT in all categories, achieving win rates of 80.27% versus GPT4-ReACT, 78.59% against GPT4-DFSDT, and 86.54% against ChatGPT-ReACT, showing improvements of 22.54% and 16.14% respectively. These results highlight TOPGUN’s superior ability to create tool plans that align with preference evaluation criteria across various conditions.

### 4.2 Black Box Evaluation

![Refer to caption](img/f5d4803bcd41b4ce97cfba1c0ab0e0ae.png)

Figure 6: Average Token Consumption of individual methodologies in Black Box setting.

Utilizing the Data Generation pipeline from Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25)), we constructed a black-box scenario dataset featuring 36 LLaMa-Hub LlamaIndex ([2023](https://arxiv.org/html/2402.10051v1#bib.bib16)) tools and unique functions from private libraries. Following Zan et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib35)), we converted Pandas and Numpy into Monkey and BeatNum packages, renaming all internal functions and structures to test planner generalizability without LLM prior knowledge. This dataset, detailed at [A.1](https://arxiv.org/html/2402.10051v1#A1.SS1 "A.1 Prompts ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"), focuses on accuracy of the solution trajectory, with each query designed for a single correct path. After manual annotation, it comprises 100 queries and 162 tools, with samples and TOPGUN outcomes at [A.3.2](https://arxiv.org/html/2402.10051v1#A1.SS3.SSS2 "A.3.2 Queries Example ‣ A.3 PrivateEval Dataset ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") and [A.5.2](https://arxiv.org/html/2402.10051v1#A1.SS5.SSS2 "A.5.2 PrivateEval ‣ A.5 TOPGUN Examples ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting").

Results : The black-box evaluation, featuring TOPGUN and a revised Reverse Chain, utilizes $\mathcal{P}$ function signatures for a comprehensive black-box methodology. TOPGUN surpasses Reverse Chain and undergoes comparison with GPT4-DFSDT and GPT4-ReACT within gray box evaluations, emphasizing output trajectories. Success rates, derived from exact trajectory matches with the ground truth and averaged over ten iterations, are documented in Table [4](https://arxiv.org/html/2402.10051v1#S4.T4 "Table 4 ‣ 4.3 CodeSynth Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). Figure [6](https://arxiv.org/html/2402.10051v1#S4.F6 "Figure 6 ‣ 4.2 Black Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") details the Average Token usage for each algorithm per query, underscoring TOPGUN’s effectiveness and efficiency in generating precise and resourceful tool plans in black-box scenarios, demonstrating its adaptability across diverse datasets.

Note: A black-box evaluation using ToolBench is infeasible, as ToolEval’s metrics, such as pass rate and win rate, rely on intermediate tool responses and the final answer.

### 4.3 CodeSynth Evaluation

To assess the quality of function signatures produced by CodeSynth, we adopt neuro-symbolic representations, as proposed by Parisotto et al. ([2017](https://arxiv.org/html/2402.10051v1#bib.bib23)) and Nye et al. ([2021](https://arxiv.org/html/2402.10051v1#bib.bib20)). These representations aim to capture the abstract semantic essence of a given program, aligning well with our objectives. Our evaluation spans the Python subset of HumanEval-X Zheng et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib38)) and MBPP Austin et al. ([2021](https://arxiv.org/html/2402.10051v1#bib.bib3)) dataset. Inspired by the semantic probing model introduced by Ma et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib19)), we construct semantic representations of both synthesized pseudo functions and ground truth code. Utilizing the tree-sitter Brunsfeld et al. ([2024](https://arxiv.org/html/2402.10051v1#bib.bib5)) package, we form the Abstract Syntax Tree, focusing our computation of the F1 score exclusively on the Function Definition block while excluding the body block. Hence, the final metric is precisely representative of our objective with CodeSynth. The appendix [A.4.1](https://arxiv.org/html/2402.10051v1#A1.SS4.SSS1 "A.4.1 HumanEval-X ‣ A.4 CodeSynth Examples ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") can be referred to for function signature examples synthesized with the HumanEval-X dataset.

Results: We evaluate CodeSynth across multiple reflection cycles, tracking the F1 score for each cycle to illustrate consistent enhancements in function signature quality, as depicted in Table [4](https://arxiv.org/html/2402.10051v1#S4.T4 "Table 4 ‣ 4.3 CodeSynth Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting"). CodeSynth significantly improved F1-scores on both HumanEval-X and MBPP datasets, achieving a perfect score of 1.0 by the fifth iteration from initial scores of 0.844 and 0.912, respectively. These findings highlight CodeSynth’s ability to produce function signatures closely resembling the semantics of the target function.

Table 3: Comparison of methodologies in Black Box Setting

| Method | Success Rate |
| GPT4-TOPGUN | 70.58 |
| GPT4-DFSDT | 61.45 |
| GPT4-ReAct | 45.45 |
| GPT4-ReverseChain | 43.75 |

| Dataset | F1 Score for max Reflexion Iteration |
| @1 | @2 | @3 | @4 | @5 |
| HumanEval-X | 0.844 | 0.894 | 0.965 | 0.983 | 1.00 |
| MBPP | 0.912 | 0.963 | 0.994 | 1.00 | 1.00 |

Table 3: Comparison of methodologies in Black Box Setting

Table 4: CodeSynth Evaluation for analyzing Reflexions improvement on Function Signature’s AST

## 5 Conclusion

In this work, we address the challenge of tool planning in black-box settings, where direct access to API calls and their implementations is not feasible, raising concerns about cost efficiency and privacy in API interactions. We introduce SwissNYF, a comprehensive framework designed to equip Large Language Models (LLMs) with the ability to navigate these scenarios effectively. Central to SwissNYF is the ingenious function signature generation that allows the planner to rely on tool descriptions, circumventing the need for actual API executions. We further introduce TOPGUN, a code-driven planning approach leveraging LLMs’ code generation capabilities to offer a robust solution for black-box environments. Our extensive evaluation across various toolsets and settings demonstrates the superior performance of our methodology against traditional tool planning strategies, validating its effectiveness and reliability. Through SwissNYF and TOPGUN, we establish an exciting and emerging paradigm in tool planning, We envision SwissNYF as a central hub for black-box tool usage, encouraging future advancements in developing strategies for black-box scenarios, thus making a significant leap towards efficient, privacy-conscious tool planning in the realm of LLM-enhanced applications.

## References

*   Achiam et al. (2023) Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, et al. Gpt-4 technical report. *arXiv preprint arXiv:2303.08774*, 2023.
*   Anil et al. (2023) Rohan Anil, Andrew M Dai, Orhan Firat, Melvin Johnson, Dmitry Lepikhin, Alexandre Passos, Siamak Shakeri, Emanuel Taropa, Paige Bailey, Zhifeng Chen, et al. Palm 2 technical report. *arXiv preprint arXiv:2305.10403*, 2023.
*   Austin et al. (2021) Jacob Austin, Augustus Odena, Maxwell Nye, Maarten Bosma, Henryk Michalewski, David Dohan, Ellen Jiang, Carrie Cai, Michael Terry, Quoc Le, et al. Program synthesis with large language models. *arXiv preprint arXiv:2108.07732*, 2021.
*   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, et al. Language models are few-shot learners. *Advances in neural information processing systems*, 33:1877–1901, 2020.
*   Brunsfeld et al. (2024) Max Brunsfeld, Andrew Hlynskyi, Amaan Qureshi, Patrick Thomson, Josh Vera, Phil Turnbull, Timothy Clem, Douglas Creager, Andrew Helwer, dundargoc, Rob Rix, Daumantas Kavolis, Hendrik van Antwerpen, Michael Davis, Ika, Tuan-Anh Nguyen, Amin Yahyaabadi, Stafford Brunk, Matt Massicotte, and George Fraser. tree-sitter/tree-sitter: v0.21.0-pre-release-1, 2024. URL [https://doi.org/10.5281/zenodo.10638807](https://doi.org/10.5281/zenodo.10638807).
*   Chen et al. (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, et al. Evaluating large language models trained on code. *arXiv preprint arXiv:2107.03374*, 2021.
*   Chen et al. (2022) Wenhu Chen, Xueguang Ma, Xinyi Wang, and William W Cohen. Program of thoughts prompting: Disentangling computation from reasoning for numerical reasoning tasks. *arXiv preprint arXiv:2211.12588*, 2022.
*   Chowdhery et al. (2023) Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann, et al. Palm: Scaling language modeling with pathways. *Journal of Machine Learning Research*, 24(240):1–113, 2023.
*   Du et al. (2024) Yu Du, Fangyun Wei, and Hongyang Zhang. Anytool: Self-reflective, hierarchical agents for large-scale api calls. *arXiv preprint arXiv:2402.04253*, 2024.
*   Fischer et al. (2007) Gregor Fischer, J Lusiardi, and J Wolff Von Gudenberg. Abstract syntax trees-and their role in model driven software development. In *International Conference on Software Engineering Advances (ICSEA 2007)*, pp.  38–38\. IEEE, 2007.
*   Gao et al. (2023) Luyu Gao, Aman Madaan, Shuyan Zhou, Uri Alon, Pengfei Liu, Yiming Yang, Jamie Callan, and Graham Neubig. Pal: Program-aided language models. In *International Conference on Machine Learning*, pp.  10764–10799\. PMLR, 2023.
*   Gou et al. (2023) Zhibin Gou, Zhihong Shao, Yeyun Gong, Yujiu Yang, Minlie Huang, Nan Duan, Weizhu Chen, et al. Tora: A tool-integrated reasoning agent for mathematical problem solving. *arXiv preprint arXiv:2309.17452*, 2023.
*   Hao et al. (2023) Shibo Hao, Tianyang Liu, Zhen Wang, and Zhiting Hu. Toolkengpt: Augmenting frozen language models with massive tools via tool embeddings. *arXiv preprint arXiv:2305.11554*, 2023.
*   Huang & Chang (2023) Jie Huang and Kevin Chen-Chuan Chang. Towards reasoning in large language models: A survey. In Anna Rogers, Jordan Boyd-Graber, and Naoaki Okazaki (eds.), *Findings of the Association for Computational Linguistics: ACL 2023*, pp.  1049–1065, Toronto, Canada, July 2023\. Association for Computational Linguistics. doi: [10.18653/v1/2023.findings-acl.67](10.18653/v1/2023.findings-acl.67). URL [https://aclanthology.org/2023.findings-acl.67](https://aclanthology.org/2023.findings-acl.67).
*   Li et al. (2023) Chengshu Li, Jacky Liang, Andy Zeng, Xinyun Chen, Karol Hausman, Dorsa Sadigh, Sergey Levine, Li Fei-Fei, Fei Xia, and Brian Ichter. Chain of code: Reasoning with a language model-augmented code emulator. *arXiv preprint arXiv:2312.04474*, 2023.
*   LlamaIndex (2023) LlamaIndex. Llamahub, 2023. URL [https://web.archive.org/web/20231229215448/https://llamahub.ai/](https://web.archive.org/web/20231229215448/https://llamahub.ai/).
*   Lu et al. (2023a) Pan Lu, Baolin Peng, Hao Cheng, Michel Galley, Kai-Wei Chang, Ying Nian Wu, Song-Chun Zhu, and Jianfeng Gao. Chameleon: Plug-and-play compositional reasoning with large language models. *arXiv preprint arXiv:2304.09842*, 2023a.
*   Lu et al. (2023b) Yining Lu, Haoping Yu, and Daniel Khashabi. Gear: Augmenting language models with generalizable and efficient tool resolution. *arXiv preprint arXiv:2307.08775*, 2023b.
*   Ma et al. (2023) Wei Ma, Mengjie Zhao, Xiaofei Xie, Qiang Hu, Shangqing Liu, Jie Zhang, Wenhan Wang, and Yang Liu. Are code pre-trained models powerful to learn code syntax and semantics?, 2023.
*   Nye et al. (2021) Maxwell Nye, Yewen Pu, Matthew Bowers, Jacob Andreas, Joshua B. Tenenbaum, and Armando Solar-Lezama. Representing partial programs with blended abstract semantics. In *International Conference on Learning Representations*, 2021. URL [https://openreview.net/forum?id=mCtadqIxOJ](https://openreview.net/forum?id=mCtadqIxOJ).
*   Paranjape et al. (2023) Bhargavi Paranjape, Scott Lundberg, Sameer Singh, Hannaneh Hajishirzi, Luke Zettlemoyer, and Marco Tulio Ribeiro. Art: Automatic multi-step reasoning and tool-use for large language models. *arXiv preprint arXiv:2303.09014*, 2023.
*   Parisi et al. (2022) Aaron Parisi, Yao Zhao, and Noah Fiedel. Talm: Tool augmented language models. *arXiv preprint arXiv:2205.12255*, 2022.
*   Parisotto et al. (2017) Emilio Parisotto, Abdel rahman Mohamed, Rishabh Singh, Lihong Li, Dengyong Zhou, and Pushmeet Kohli. Neuro-symbolic program synthesis. In *International Conference on Learning Representations*, 2017. URL [https://openreview.net/forum?id=rJ0JwFcex](https://openreview.net/forum?id=rJ0JwFcex).
*   Patil et al. (2023) Shishir G Patil, Tianjun Zhang, Xin Wang, and Joseph E Gonzalez. Gorilla: Large language model connected with massive apis. *arXiv preprint arXiv:2305.15334*, 2023.
*   Qin et al. (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. Toolllm: Facilitating large language models to master 16000+ real-world apis. *arXiv preprint arXiv:2307.16789*, 2023.
*   Radford et al. (2018) Alec Radford, Karthik Narasimhan, Tim Salimans, Ilya Sutskever, et al. Improving language understanding by generative pre-training. 2018.
*   Radford et al. (2019) Alec Radford, Jeffrey Wu, Rewon Child, David Luan, Dario Amodei, Ilya Sutskever, et al. Language models are unsupervised multitask learners. *OpenAI blog*, 1(8):9, 2019.
*   Schick et al. (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Luke Zettlemoyer, Nicola Cancedda, and Thomas Scialom. Toolformer: Language models can teach themselves to use tools. *arXiv preprint arXiv:2302.04761*, 2023.
*   Shinn et al. (2023) Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik R Narasimhan, and Shunyu Yao. Reflexion: Language agents with verbal reinforcement learning. In *Thirty-seventh Conference on Neural Information Processing Systems*, 2023.
*   Su et al. (2022) Hongjin Su, Weijia Shi, Jungo Kasai, Yizhong Wang, Yushi Hu, Mari Ostendorf, Wen-tau Yih, Noah A Smith, Luke Zettlemoyer, and Tao Yu. One embedder, any task: Instruction-finetuned text embeddings. *arXiv preprint arXiv:2212.09741*, 2022.
*   Tang et al. (2023) Qiaoyu Tang, Ziliang Deng, Hongyu Lin, Xianpei Han, Qiao Liang, and Le Sun. Toolalpaca: Generalized tool learning for language models with 3000 simulated cases. *arXiv preprint arXiv:2306.05301*, 2023.
*   Xu et al. (2023) Yiheng Xu, Hongjin Su, Chen Xing, Boyu Mi, Qian Liu, Weijia Shi, Binyuan Hui, Fan Zhou, Yitao Liu, Tianbao Xie, et al. Lemur: Harmonizing natural language and code for language agents. *arXiv preprint arXiv:2310.06830*, 2023.
*   Yang et al. (2023) Rui Yang, Lin Song, Yanwei Li, Sijie Zhao, Yixiao Ge, Xiu Li, and Ying Shan. Gpt4tools: Teaching large language model to use tools via self-instruction, 2023.
*   Yao et al. (2022) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. React: Synergizing reasoning and acting in language models. *arXiv preprint arXiv:2210.03629*, 2022.
*   Zan et al. (2022) Daoguang Zan, Bei Chen, Zeqi Lin, Bei Guan, Yongji Wang, and Jian-Guang Lou. When language model meets private library. *arXiv preprint arXiv:2210.17236*, 2022.
*   Zhang et al. (2023a) Beichen Zhang, Kun Zhou, Xilin Wei, Wayne Xin Zhao, Jing Sha, Shijin Wang, and Ji-Rong Wen. Evaluating and improving tool-augmented computation-intensive math reasoning, 2023a.
*   Zhang et al. (2023b) Yinger Zhang, Hui Cai, Yicheng Chen, Rui Sun, and Jing Zheng. Reverse chain: A generic-rule for llms to master multi-api planning. *arXiv preprint arXiv:2310.04474*, 2023b.
*   Zheng et al. (2023) Qinkai Zheng, Xiao Xia, Xu Zou, Yuxiao Dong, Shan Wang, Yufei Xue, Lei Shen, Zihan Wang, Andi Wang, Yang Li, et al. Codegeex: A pre-trained model for code generation with multilingual benchmarking on humaneval-x. In *Proceedings of the 29th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*, pp.  5673–5684, 2023.
*   Zhou et al. (2023a) Andy Zhou, Kai Yan, Michal Shlapentokh-Rothman, Haohan Wang, and Yu-Xiong Wang. Language agent tree search unifies reasoning acting and planning in language models. *arXiv preprint arXiv:2310.04406*, 2023a.
*   Zhou et al. (2023b) Aojun Zhou, Ke Wang, Zimu Lu, Weikang Shi, Sichun Luo, Zipeng Qin, Shaoqing Lu, Anya Jia, Linqi Song, Mingjie Zhan, et al. Solving challenging math word problems using gpt-4 code interpreter with code-based self-verification. *arXiv preprint arXiv:2308.07921*, 2023b.
*   Zhuang et al. (2023a) Yuchen Zhuang, Xiang Chen, Tong Yu, Saayan Mitra, Victor Bursztyn, Ryan A. Rossi, Somdeb Sarkhel, and Chao Zhang. Toolchain*: Efficient action space navigation in large language models with a* search, 2023a.
*   Zhuang et al. (2023b) Yuchen Zhuang, Yue Yu, Kuan Wang, Haotian Sun, and Chao Zhang. Toolqa: A dataset for llm question answering with external tools, 2023b.

## Appendix A Appendix

### A.1 Prompts

CodeSynth prompt for function signature generation

```
You are a Python code assistant that can generate a
pseudo-Python function given the name, description,
and arguments.

function name: {}
function description: {}

You have to generate a pseudo-Python function that only
contains docstring and a return example object for the
above-given information. Use dummy examples as return
objects.

Maintain the return datatype. Docsrting contains Args and
Returns. Maintain the arguments typing. The arguments are
optional and should be assigned relevant default values
according to their return type.

Only generate the def function itself as instructed above,
no typing imports or other code is needed.

```

TOPGUN prompt for code-based plan generation

```

You are a Python code assistant. Today, you are challenged
to generate a Python code for executing a query. You will
be given a list of pseudo functions that you will use in
your Python code to help you in solving the query correctly.

Understand the query properly and use the required
function to solve it.

We have the following pseudo functions:
=====
{}
=====

Let’s start

If the query is {}
Return the python code to execute it with the help of given
functions. Do not use double quotes; only use single quotes.
Always have to the code within ‘‘‘python\n<--Your Code-->\n‘‘‘
Always remember if a function is to input or output an object
assume the object to be a string.

```

Function Call Prompt for verification

```
You are a Python code assistant. You are given a function.
For the given function, write an executable function call
using dummy argument values.

Provided Libraries: {}

Details of the provided library can only be fetched using
the query engine tool, feel free to use it.

-You can import the required classes from one of the provided
 libraries, according to the function arguments and documentation.
-If any library is not provided, ignore any imports.
-Do not import {} function for which you generate the
 function call.
-Do not generate any unnecessary import statements.
-No print statements are needed.
-Always have to code within ‘‘‘python\n<--Your Code-->\n‘‘‘

Example:

Given Function:
  def add(a: int, b: int) -> int:
      ’’’
      Given integers a and b,
      return the total value of a and b.
      ’’’
      return a + b

Function Call:
  a = 1
  b = 4
  add(a, b)

The function name is: {}
The function description is: {}
The Function is: {}
Function Call:

```

Self-Reflection Prompt

```
You are a Python code assistant. You will be given your last
Python code implementation, and an error in your last
implementation will be provided. Taking the error into
account, refactor your Python code.

Use the query engine to export the information needed
to resolve.

Always have to code within ‘‘‘python\n<--Your Code-->\n‘‘‘

Previous python code implementation: {}
Self-reflection: {}

Refactored Python code:

```

CodeSynth prompt for function signature generation on PrivateEval

```
You are a Python code assistant that can generate a pseudo
Python function given its name, description, and arguments.

function name: {}
function description: {}
Provided Libraries: {}

Always remember to import the required classes from one of the
provided library, according to the function arguments and the
provided documentation.

Documentation is to be fetched using the query engine tool.

If any library is not provided, ignore any imports.

The function arguments and returns are clearly defined in the
function description. Use as provided in the description.

You have to generate a pseudo-Python function that only contains
docstring and a dummy return object matching the actual return
datatype. No need to use the provided arguments. Just return a
dummy object that matches the actual return datatype of the
function.

Maintain the actual return datatype in the return object.
Docsrting contains Args and Returns. Maintain the arguments
typing.

Only generate the def function as instructed above; no typing
imports or other code is needed.

Always have to the code within ‘‘‘python\n<--Your Code-->\n‘‘‘

Pseudo Function:

```

TOPGUN prompt for code-based plan generation on ToolBench

```
You are a Python code assistant. Today, you are challenged
to generate a Python code for executing a query. You will
be given a list of pseudo functions that you will use in
your Python code to help you in solving the query correctly.
Understand the query properly and use the required function
to solve it.

We have the following pseudo functions:
=====
{}
=====

You have to make sure to follow the below guardrails:
 - Do not use double quotes; only use single quotes.
 - You are not allowed to define any functions; you must
   always use the given functions in the code.
 - If in case you end up creating a function, please
   rememeber to have a decorator named @update_traverse_dict
   on them.
 - Do not create a main function script and using
   ’if __name__ == "__main__"’ is strictly prohibited.
 - Always have to the code within ‘‘‘python\n<--Your Code-->\n‘‘‘
 - Always remember to use .get() to fetch values from a
   dictionary or a JSON.
 - Always remember to replace the values in .get() of the
   generated code with a value that matches the description of
   its key and dictionary whose argument it is. Use your
   world knowledge to replace the value with a
   good, real example.
   Example:
     contact = company_info.get(’contact_number’, ’999991999’)
     name = company_info.get(’name’, ’ryanair’)

   Remember to Keep the values inside single quotes ’ ’.
 - This is also required when accessing the value of the list
   use try: except: and in except use a value that matches
   the description of the output.
 - Never use print statements. The user can use the variables
   in the code to infer the code.

You have to remember the following to solve the query:
 - Always remember if a function is to input or output an
   object assumes an object to be a string.
 - Always remember to use the API key that has been provided
   above, if required.

If the query is {}
Return the Python code to execute it with the help of the given
pseudo functions.

```

Prompt for query generation for PrivateEval  

```
You will be provided with several tools, tool descriptions, all of
each tool’s available API functions, the descriptions of these API
functions, and the parameters required for each API function. Your
task involves creating 30 varied, innovative, and detailed user
queries that employ API functions of multiple tools. For instance,
given three tools ‘azure speech’, ‘wikipedia’, and ‘google search’:
‘azure speech’ has API functions ’speech_to_text’ and
’text_to_speech’, ‘wikipedia’ has API functions ’search_data’ and
’read_search_data’, ‘google search’ has API functions
‘google_search’ and ‘read_google_search’. Your query should
articulate something akin to: ‘I recently found a banana with red
spots inside. Which plant disease is this? Can you find an Wikipedia
article on this and read it out to me.’ This query exemplifies how
to utilize API calls of all the given tools. A query that uses API
calls of only one tool will not be accepted. Additionally, you must
incorporate the input parameters required for each API call. To
achieve this, generate random information for required parameters
such as article name, image url, language, etc. For instance, don’t
merely say ‘example image url’, provide the exact link to a image.
Don’t just mention ‘language’, specify en, fr, it, etc. Don’t refer
to ‘dish’, use a real dish such as ‘lasagna’ instead. The first
twenty of the thirty queries should be very specific. Each single
query should combine API calls of different tools in various ways
and include the necessary parameters. Note that you shouldn’t ask
‘which API to use’, rather, simply state your needs that can be
addressed by these APIs. You should also avoid asking for the
input parameters required by the API call, but instead directly
provide the parameters in your query. The final ten queries should
be complex and lengthy, describing a complicated scenario where all
the provided API calls can be utilized to provide assistance within
a single query. You should first think about possible related API
combinations, then give your query. Related APIs are APIs that can
be used for a given query; those related APIs have to strictly come
from the provided API names. For each query, there should be
multiple related APIs; for different queries, overlap of related
APIs should be as little as possible. Deliver your response in
this format: [Query1: ...., ‘related apis’: [[tool name, api name],
[tool name, api name], [tool name, API name]...],Query2: ......,
‘related apis’:[[tool name,api name],[tool name, api name],
[tool name, api name]...] ,Query3: ......, ‘related apis’:
[[tool name, api name], [tool name, api name],
[tool name, api name]...], ...]

```

### A.2 ToolBench for Gray Box Evaluation

ToolBench is a diverse benchmark spanning over 16k APIs across 49 categories from RapidAPI Hub. It consists of three sets of instructions for tool augmentation evaluation: (1) Single-tool instruction (I1), (2) Intra-category multi-tool instruction (I2), and (3) Intra-collection multi-tool instructions. Such a rich set of APIs and instructions makes it a perfect ground to test our pipeline. ToolBench proposes ToolEval containing the evaluation procedure for this set of instructions. ToolEval designs two evaluation metrics using ChatGPT: (1) Pass Rate, calculated by the proportion of instructions completed within a limited budget; (2) Win Rate, measured by asking a ChatGPT evaluator to select its preference for two solution paths. We focus on Win Rate for the evaluation metric to draw comparisons between TOPGUN and other gray box approaches such as DFSDT and ReAct. ToolEval uses a tree-based representation of the responses to generate solution paths, which are then compared to calculate the win rate.

ToolEval response representation

```
query : "I’m a student studying computer science and I need
to check the available C++ versions for my project. Can you
fetch the versions using the CPP Code Compiler API?
Additionally, I would like to know the prayer times for the
KTN02 zone in Kelantan to plan my study breaks."

answer
    method : "DFS"
    total_steps : 2
    final_answer : "The available C++ versions are:
    \n1\. GCC 5.3.0\n2\. Zapcc 5.0.0\n3\. GCC 7.2.0\n4.
    GCC 8.1.0\n5\. GCC 9.1.0\n6\. GCC 11.1.0 The prayer
    times for the KTN02 zone in Kelantan are as follows:\n1.
    Imsak: 05:44:00\n2\. Subuh: 05:54:00\n3\. Syuruk: 07:08:00\n4.
    Zohor: 13:22:00\n5\. Asar: 16:46:00\n6\. Maghrib: 19:32:00\n7.
    Isyak: 20:42:00"

answer_details
    role : "tool"
    message
        name : "c_versions_for_cpp_code_compiler"
        arguments : "{}"
        response : "{"error": "", "response": "{’0’: {’id’: 0,
        ’name’: ’GCC 5.3.0’}, ’1’: {’id’: 1, ’name’:
        ’Zapcc 5.0.0’}, ’2’: {’id’: 2, ’name’: ’GCC 7.2.0’}, ’3’:
        {’id’: 3, ’name’: ’GCC 8.1.0’}, ’4’: {’id’: 4, ’name’:
        ’GCC 9.1.0’},’5’: {’id’: 5, ’name’: ’GCC 11.1.0’}}"}"

    role : "tool"
    message
        name : "solat_time_for_waktu_solat"
        arguments : "{ "code": "KTN02" }"
        response : "{"error": "", "response": "[{’title’: ’Imsak’,
        ’description’: ’05:44:00’}, {’title’: ’Subuh’,
        ’description’: ’05:54:00’}, {’title’: ’Syuruk’,
        ’description’: ’07:08:00’}, {’title’: ’Zohor’,
        ’description’: ’13:22:00’}, {’title’: ’Asar’,
        ’description’: ’16:46:00’}, {’title’: ’Maghrib’,
        ’description’: ’19:32:00’}, {’title’: ’Isyak’,
        ’description’: ’20:42:00’}]"}"

```

We ensure that the code plan generated by TOPGUN precisely aligns with this representation to harness ToolEval for win rate calculation. In our black-box inference phase, we lack the final answer and tool responses. However, we retrieve these values during gray-box evaluation involving actual API calls and populate the representation accordingly.

Black Box Inference output

```
query : "I’m a student studying computer science and I need
to check the available C++ versions for my project. Can you
fetch the versions using the CPP Code Compiler API?
Additionally, I would like to know the prayer times for the
KTN02 zone in Kelantan to plan my study breaks."

available_tools

answer
    method : "gpt4_topgun"
    total_steps : 2
    final_answer : ""

answer_details
    role : "tool"
    message
        name : "c_versions"
        arguments : "{}"
        response : ""

    role : "tool"
    message
        name : "solat_time"
        arguments : "{’code’: ’KTN02’}"
        response : ""

```

Gray Box Evaluation output

```
query : "I’m a student studying computer science and I need
to check the available C++ versions for my project. Can you
fetch the versions using the CPP Code Compiler API?
Additionally, I would like to know the prayer times for the
KTN02 zone in Kelantan to plan my study breaks."

available_tools

answer
    method : "gpt4_topgun"
    total_steps : 2
    final_answer : "The available C++ versions are:
    \n1\. GCC 5.3.0\n2\. Zapcc 5.0.0\n3\. GCC 7.2.0\n4.
    GCC 8.1.0\n5\. GCC 9.1.0\n6\. GCC 11.1.0 The prayer
    times for the KTN02 zone in Kelantan are as follows:
    \n1\. Imsak: 05:44:00\n2\. Subuh: 05:54:00\n3\. Syuruk:
    07:08:00\n4\. Zohor: 13:22:00\n5\. Asar: 16:46:00\n6.
    Maghrib: 19:32:00\n7\. Isyak: 20:42:00"

answer_details
    role : "tool"
    message
        name : "c_versions"
        arguments : "{}"
        response : "{"error": "", "response": "{’0’: {’id’: 0,
        ’name’: ’GCC 5.3.0’}, ’1’: {’id’: 1, ’name’:
        ’Zapcc 5.0.0’}, ’2’: {’id’: 2, ’name’: ’GCC 7.2.0’}, ’3’:
        {’id’: 3, ’name’: ’GCC 8.1.0’}, ’4’: {’id’: 4, ’name’:
        ’GCC 9.1.0’},’5’: {’id’: 5, ’name’: ’GCC 11.1.0’}}"}"

    role : "tool"
    message
        name : "solat_time"
        arguments : "{ "code": "KTN02" }"
        response : "{"error": "", "response": "[{’title’: ’Imsak’,
        ’description’: ’05:44:00’}, {’title’: ’Subuh’,
        ’description’: ’05:54:00’}, {’title’: ’Syuruk’,
        ’description’: ’07:08:00’}, {’title’: ’Zohor’,
        ’description’: ’13:22:00’}, {’title’: ’Asar’,
        ’description’: ’16:46:00’}, {’title’: ’Maghrib’,
        ’description’: ’19:32:00’}, {’title’: ’Isyak’,
        ’description’: ’20:42:00’}]"}"

```

We input the solution path representations from TOPGUN and other approaches into ToolEval’s preference test to compute the win rate for each query. These win rates are then averaged across different sets of instructions to determine the average win rate.

### A.3 PrivateEval Dataset

Here, we list some examples of tools and queries that we created for PrivateEval.

#### A.3.1 Tools

Moneky and BeatNum

```
’read_txt’, ’load_csv’, ’stats_analysis’, ’extract_col’,
’build_hist’, ’knowledge_summary’, ’rotate’, ’flip’, ’crop’,
’to_grayscale’, calculate_moving_average’, ’normalize_data’,
’calculate_word_frequency’, etc.

```

Llama Hub

```
’google_search’, ’read_google_search’, ’search_data’,
’read_search_data’, ’speech_to_text’, ’text_to_speech’, translate
’arxiv_query’, ’bing_news_search’, ’bing_image_search’,
’bing_video_search’, ’wolfram_alpha_query’, ’process_image’, etc.

```

#### A.3.2 Queries Example

1.  1.

    ```
    Could you help me load a multilingual dataset? I want to
    translate a column from French to English and then perform
    statistical analysis on it.

    ```

2.  2.

    ```
    Could you help me find the Chinchilla LLM paper? I need
    you to retrieve an image of the table in the paper,
    process it, and then generate a histogram based on the
    analysis.

    ```

3.  3.

    ```
    Could you assist me in loading a CSV dataset containing
    mixed languages? Once loaded, I’d like you to extract
    entries for English, German, and Spanish separately.
    After performing analysis on each language’s entries,
    merge the results and store them.

    ```

4.  4.

    ```
    Please retrieve Tesla stock price data from an online
    database. Next, calculate moving averages. Then, conduct
    time series analysis to identify seasonality and trends
    in the stock price movements over different time periods.
    Finally, summarize the findings.

    ```

5.  5.

    ```
    Could you please retrieve some images of dogs? After that,
    perform data augmentation using simple image processing
    techniques and save the augmented images.

    ```

6.  6.

    ```
    Could you search for papers on "artificial intelligence"
    on arXiv? Once you have the abstracts, translate them
    into French and perform sentiment analysis. Finally, we’ll
    visualize the distribution of sentiments.

    ```

7.  7.

    ```
    Please search for educational podcasts on "quantum physics".
    Once you have the podcasts, transcribe the audio content.
    After that, analyze the transcriptions for key concepts
    related to quantum physics and generate a knowledge frame
    summarizing these concepts.

    ```

8.  8.

    ```
    Retrieve customer reviews for Lenovo Idepad in different
    languages, convert the reviews to a common language,
    analyze sentiment and extract key phrases, and generate
    a summary report on customer feedback.

    ```

9.  9.

    ```
    Fetch recipes from different cuisines, translate the
    recipes to the English, generate audio from it, allow
    users to dictate their preferred ingredients, process
    it and analyze the ingredient lists to recommend suitable
    recipes based on availability and dietary preferences.

    ```

10.  10.

    ```
    Please search for a lasagna recipe. Once you have it,
    translate it from Italian to English. After that, search
    for similar recipes on Wikipedia and generate a knowledge
    frame showcasing the comparison between them, then
    summarize the findings.

    ```

11.  11.

    ```
    Please search for a TED talk speech. Once you have it,
    translate it from English to Mandarin. After that,
    generate a transcript of the translated speech. Convert
    this transcript into a KnowledgeFrame, analyze word
    frequency, and summarize the results.

    ```

12.  12.

    ```
    Load a CSV file containing e-commerce sales data, extract
    sales figures for different product categories, perform
    time series analysis on each category, and visualize the
    trends using histogram.

    ```

13.  13.

    ```
    Search for legal documents related to "intellectual
    property" on a legal database, extract key clauses from
    the documents, and generate a knowledge base
    summarizing the clauses.

    ```

14.  14.

    ```
    Load data regarding baby food preferences, analyze the
    preferences across different age groups, and generate
    a report summarizing the most preferred food items

    ```

### A.4 CodeSynth Examples

Examples of function signatures and calls generated by CodeSynth while evaluating with HumanEval-X and PrivateEval datasets.

#### A.4.1 HumanEval-X

1.  (a)

    ```
    Name: intersperse

    Description: Insert a number ’delimeter’ between every two
                 consecutive elements of input list ‘numbers’
                 >>> intersperse([], 4)
                 []
                 >>> intersperse([1, 2, 3], 4)
                 [1, 4, 2, 4, 3]

    ```

    [⬇](data:text/plain;base64,CmRlZiBpbnRlcnNwZXJzZShudW1iZXJzOiBMaXN0W2ludF0sIGRlbGltZXRlcjogaW50KSAtPiBMaXN0W2ludF06CiIiIgpBcmdzOgpudW1iZXJzIChMaXN0W2ludF0pOiBBIGxpc3Qgb2YgaW50ZWdlcnMKZGVsaW1ldGVyIChpbnQpOiBBbiBpbnRlZ2VyIHRvIGJlIGluc2VydGVkIGJldHdlZW4gZXZlcnkKdHdvIGNvbnNlY3V0aXZlIGVsZW1lbnRzIG9mIHRoZSBpbnB1dApsaXN0ClxwYXJSZXR1cm5zOgpMaXN0W2ludF06IEEgbmV3IGxpc3Qgd2l0aCB0aGUgZGVsaW1ldGVyIGluc2VydGVkIGJldHdlZW4KZXZlcnkgdHdvIGNvbnNlY3V0aXZlIGVsZW1lbnRzIG9mIHRoZSBpbnB1dApsaXN0CiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTYwMCAqKioqIiIiCnJldHVybiBbMF0gIyBEdW1teSByZXR1cm4gb2JqZWN0ClxwYXJccGFyIyBGdW5jdGlvbiBDYWxsOgpmcm9tIHR5cGluZyBpbXBvcnQgTGlzdApccGFybnVtYmVycyA9IFsxLCAyLCAzXQpkZWxpbWV0ZXIgPSA0CmludGVyc3BlcnNlKG51bWJlcnMsIGRlbGltZXRlcikK)def intersperse(numbers: List[int], delimeter: int) -> List[int]:"""Args:numbers (List[int]): A list of integersdelimeter (int): An integer to be inserted between everytwo consecutive elements of the inputlist\parReturns:List[int]: A new list with the delimeter inserted betweenevery two consecutive elements of the inputlist%**** iclr2024_conference.tex Line 1600 ****"""return [0] # Dummy return object\par\par# Function Call:from typing import List\parnumbers = [1, 2, 3]delimeter = 4intersperse(numbers, delimeter)
2.  (b)

    ```
    Name: pairs_sum_to_zero

    Description: pairs_sum_to_zero takes a list of integers
                 as an input. it returns True if there are
                 two distinct elements in the list that sum
                 to zero, and False otherwise.
                 >>> pairs_sum_to_zero([1, 3, 5, 0])
                 False
                 >>> pairs_sum_to_zero([1, 3, -2, 1])
                 False

    ```

    [⬇](data:text/plain;base64,CmRlZiBwYWlyc19zdW1fdG9femVybyhsOiBMaXN0W2ludF0pIC0+IGJvb2w6CiIiIgpBcmdzOgpsIChMaXN0W2ludF0pOiBBIGxpc3Qgb2YgaW50ZWdlcnMgYXMgYW4gaW5wdXQuClxwYXJSZXR1cm5zOgpib29sOiBUcnVlIGlmIHRoZXJlIGFyZSB0d28gZGlzdGluY3QgZWxlbWVudHMgaW4KdGhlIGxpc3QgdGhhdCBzdW0gdG8gemVybywgYW5kIEZhbHNlIG90aGVyd2lzZS4KIiIiCnJldHVybiBGYWxzZQpccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKbCA9IFsyLCA0LCAtNSwgMywgNSwgN10KcGFpcnNfc3VtX3RvX3plcm8obCkKXHBhcg==)def pairs_sum_to_zero(l: List[int]) -> bool:"""Args:l (List[int]): A list of integers as an input.\parReturns:bool: True if there are two distinct elements inthe list that sum to zero, and False otherwise."""return False\par\par# Function Call:l = [2, 4, -5, 3, 5, 7]pairs_sum_to_zero(l)\par
3.  (c)

    ```
    Name: vowels_count

    Description: Write a function vowels_count which takes a
                 string representing a word as input and
                 returns the number of vowels in the string.
                 Vowels in this case are ’a’, ’e’, ’i’, ’o’, ’u’.
                 Here, ’y’ is also a vowel, but only when it is
                 at the end of the given word.

                 Example:
                 >>> vowels_count("abcde")
                 2
                 >>> vowels_count("ACEDY")
                 3

    ```

    [⬇](data:text/plain;base64,CmRlZiB2b3dlbHNfY291bnQoczogc3RyKSAtPiBpbnQ6CiIiIgpBcmdzOgpzIChzdHIpOiBBIHN0cmluZyByZXByZXNlbnRpbmcgYSB3b3JkLgpccGFyUmV0dXJuczoKaW50OiBUaGUgbnVtYmVyIG9mIHZvd2VscyBpbiB0aGUgc3RyaW5nLgoiIiIKcmV0dXJuIDAKXHBhclxwYXIlKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDE2NzUgKioqKiMgRnVuY3Rpb24gQ2FsbDoKcyA9ICJleGFtcGxlIgp2b3dlbHNfY291bnQocykK)def vowels_count(s: str) -> int:"""Args:s (str): A string representing a word.\parReturns:int: The number of vowels in the string."""return 0\par\par%**** iclr2024_conference.tex Line 1675 ****# Function Call:s = "example"vowels_count(s)
4.  (d)

    ```
    Name: prod_signs

    Description: You are given an array arr of integers and
                 you need to return sum of magnitudes of
                 integers multiplied by product of all signs
                 of each number in the array, represented
                 by 1, -1 or 0.
                 Note: return None for empty arr.

                 Example:
                 >>> prod_signs([1, 2, 2, -4]) == -9
                 >>> prod_signs([0, 1]) == 0
                 >>> prod_signs([]) == None

    ```

    [⬇](data:text/plain;base64,CiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTcwMCAqKioqZGVmIHByb2Rfc2lnbnMoYXJyOiBMaXN0W2ludF0pIC0+IFVuaW9uW2ludCwgTm9uZV06CiIiIgpBcmdzOgphcnIgKExpc3RbaW50XSk6IEFuIGFycmF5IG9mIGludGVnZXJzLgpccGFyUmV0dXJuczoKVW5pb25baW50LCBOb25lXTogVGhlIHN1bSBvZiBtYWduaXR1ZGVzIG9mIGludGVnZXJzCm11bHRpcGxpZWQgYnkgdGhlIHByb2R1Y3Qgb2YgYWxsIHNpZ25zCm9mIGVhY2ggbnVtYmVyIGluIHRoZSBhcnJheSwgcmVwcmVzZW50ZWQKYnkgMSwgLTEgb3IgMC4gUmV0dXJucyBOb25lIGZvciBlbXB0eSBhcnIuClxwYXIiIiIKcmV0dXJuIDAgIyBEdW1teSByZXR1cm4gb2JqZWN0ClxwYXJccGFyIyBGdW5jdGlvbiBDYWxsOgpmcm9tIHR5cGluZyBpbXBvcnQgTGlzdCwgVW5pb24KXHBhcmFyciA9IFsxLCAyLCAyLCAtNF0KcHJvZF9zaWducyhhcnIpCg==)%**** iclr2024_conference.tex Line 1700 ****def prod_signs(arr: List[int]) -> Union[int, None]:"""Args:arr (List[int]): An array of integers.\parReturns:Union[int, None]: The sum of magnitudes of integersmultiplied by the product of all signsof each number in the array, representedby 1, -1 or 0. Returns None for empty arr.\par"""return 0 # Dummy return object\par\par# Function Call:from typing import List, Union\pararr = [1, 2, 2, -4]prod_signs(arr)
5.  (e)

    ```
    Name: will_it_fly

    Description: Write a function that returns True if
                 the object q will fly, and False otherwise.
                 The object q will fly if it’s balanced
                 (it is a palindromic list) and the sum of
                 its elements is less than or equal the maximum
                 possible weight w.

                 Example:
                 will_it_fly([1, 2], 5) -> False

                 will_it_fly([3, 2, 3], 1) -> False

    ```

    [⬇](data:text/plain;base64,CmRlZiB3aWxsX2l0X2ZseShxOiBMaXN0W2ludF0sIHc6IGludCkgLT4gYm9vbDoKIiIiCkFyZ3M6CnEgKExpc3RbaW50XSk6IEEgbGlzdCBvZiBpbnRlZ2VycyByZXByZXNlbnRpbmcgdGhlCm9iamVjdCdzIHdlaWdodCBkaXN0cmlidXRpb24uCncgKGludCk6IFRoZSBtYXhpbXVtIHBvc3NpYmxlIHdlaWdodCBmb3IgdGhlIG9iamVjdCB0byBmbHkuClxwYXJSZXR1cm5zOgolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDE3NTAgKioqKmJvb2w6IFRydWUgaWYgdGhlIG9iamVjdCB3aWxsIGZseSwgRmFsc2Ugb3RoZXJ3aXNlLgoiIiIKcmV0dXJuIFRydWUgIyBEdW1teSByZXR1cm4KXHBhclxwYXIjIEZ1bmN0aW9uIENhbGw6CmZyb20gdHlwaW5nIGltcG9ydCBMaXN0ClxwYXJxID0gWzMsIDIsIDNdCncgPSA5CndpbGxfaXRfZmx5KHEsIHcpCg==)def will_it_fly(q: List[int], w: int) -> bool:"""Args:q (List[int]): A list of integers representing theobject’s weight distribution.w (int): The maximum possible weight for the object to fly.\parReturns:%**** iclr2024_conference.tex Line 1750 ****bool: True if the object will fly, False otherwise."""return True # Dummy return\par\par# Function Call:from typing import List\parq = [3, 2, 3]w = 9will_it_fly(q, w)

#### A.4.2 PrivateEval

1.  (a)

    ```
    Name: stats_analysis

    Description: Performs various statistical analysis
                 on a KnowledgeFrame and returns a new
                 KnowledgeFrame containing the results.

                 Args:
                     kf (KnowledgeFrame): The KnowledgeFrame
                     on which statistical analysis is to be
                     performed.

                 Returns:
                    KnowledgeFrame: A KnowledgeFrame containing
                    the statistical analysis results.

    ```

    [⬇](data:text/plain;base64,CmRlZiBzdGF0c19hbmFseXNpcyhrbm93bGVkZ2VmcmFtZSk6CiIiIgpQZXJmb3JtcyB2YXJpb3VzIHN0YXRpc3RpY2FsIGFuYWx5c2VzIG9uIGEgS25vd2xlZGdlRnJhbWUKYW5kIHJldHVybnMgYSBuZXcgS25vd2xlZGdlRnJhbWUgY29udGFpbmluZyB0aGUgcmVzdWx0cy4KXHBhckFyZ3M6Cmtub3dsZWRnZWZyYW1lIChLbm93bGVkZ2VGcmFtZSk6IFRoZSBLbm93bGVkZ2VGcmFtZQpvbiB3aGljaCBzdGF0aXN0aWNhbCBhbmFseXNpcyB3aWxsIGJlIHBlcmZvcm1lZC4KXHBhclJldHVybnM6Cktub3dsZWRnZUZyYW1lOiBBIEtub3dsZWRnZUZyYW1lIGNvbnRhaW5pbmcgdGhlCnN0YXRpc3RpY2FsIGFuYWx5c2lzIHJlc3VsdHMuCiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTgwMCAqKioqIiIiCnJldHVybiBLbm93bGVkZ2VGcmFtZSgpICMgRHVtbXkgcmV0dXJuIG9iamVjdApccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKZnJvbSBtb25rZXkgaW1wb3J0IEtub3dsZWRnZUZyYW1lClxwYXIjIER1bW15IGRhdGEgZm9yIHRoZSBLbm93bGVkZ2VGcmFtZQpkYXRhID0gewonY29sdW1uMSc6IFsxLCAyLCAzXSwKJ2NvbHVtbjInOiBbNCwgNSwgNl0sCidjb2x1bW4zJzogWzcsIDgsIDldCn0KXHBhciMgQ3JlYXRlIGEgZHVtbXkgS25vd2xlZGdlRnJhbWUKa25vd2xlZGdlZnJhbWUgPSBLbm93bGVkZ2VGcmFtZShkYXRhKQpccGFyIyBDYWxsIHRoZSBzdGF0c19hbmFseXNpcyBmdW5jdGlvbiB3aXRoIHRoZSBkdW1teSBLbm93bGVkZ2VGcmFtZQpyZXN1bHQgPSBzdGF0c19hbmFseXNpcyhrbm93bGVkZ2VmcmFtZSkK)def stats_analysis(knowledgeframe):"""Performs various statistical analyses on a KnowledgeFrameand returns a new KnowledgeFrame containing the results.\parArgs:knowledgeframe (KnowledgeFrame): The KnowledgeFrameon which statistical analysis will be performed.\parReturns:KnowledgeFrame: A KnowledgeFrame containing thestatistical analysis results.%**** iclr2024_conference.tex Line 1800 ****"""return KnowledgeFrame() # Dummy return object\par\par# Function Call:from monkey import KnowledgeFrame\par# Dummy data for the KnowledgeFramedata = {’column1’: [1, 2, 3],’column2’: [4, 5, 6],’column3’: [7, 8, 9]}\par# Create a dummy KnowledgeFrameknowledgeframe = KnowledgeFrame(data)\par# Call the stats_analysis function with the dummy KnowledgeFrameresult = stats_analysis(knowledgeframe)
2.  (b)

    ```
    Name: knowledge_summary

    Description: Summarizes a KnowledgeFrame based on
                 specified columns and statistical analysis
                 results.

                 Args:
                    kf (KnowledgeFrame): The KnowledgeFrame
                    to be summarized.

                    columns (List[str]): The list of column
                    names to include in the summary.

                    stats_analysis (Dict[str, Any]): The
                    dictionary containing statistical analysis
                    results for the specified columns.

                 Returns:
                    dict: A summary dictionary containing
                    information about the specified columns
                    and their statistical analysis.

    ```

    [⬇](data:text/plain;base64,CmRlZiBrbm93bGVkZ2Vfc3VtbWFyeShrbm93bGVkZ2VmcmFtZSwgY29sdW1ucywgc3RhdHNfYW5hbHlzaXMpOgoiIiIKU3VtbWFyaXplcyBhIEtub3dsZWRnZUZyYW1lIGJhc2VkIG9uIHNwZWNpZmllZCBjb2x1bW5zIGFuZCBzdGF0aXN0aWNhbCBhbmFseXNpcyByZXN1bHRzLgolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDE4NTAgKioqKlxwYXJBcmdzOgprbm93bGVkZ2VmcmFtZSAoS25vd2xlZGdlRnJhbWUpOiBUaGUgS25vd2xlZGdlRnJhbWUgdG8KYmUgc3VtbWFyaXplZC4KXHBhcmNvbHVtbnMgKExpc3Rbc3RyXSk6IFRoZSBsaXN0IG9mIGNvbHVtbiBuYW1lcyB0byBpbmNsdWRlCmluIHRoZSBzdW1tYXJ5LgpccGFyc3RhdHNfYW5hbHlzaXMgKERpY3Rbc3RyLCBBbnldKTogVGhlIGRpY3Rpb25hcnkgY29udGFpbmluZwpzdGF0aXN0aWNhbCBhbmFseXNpcyByZXN1bHRzIGZvciB0aGUgc3BlY2lmaWVkIGNvbHVtbnMuClxwYXJSZXR1cm5zOgpkaWN0OiBBIHN1bW1hcnkgZGljdGlvbmFyeSBjb250YWluaW5nIGluZm9ybWF0aW9uIGFib3V0CnRoZSBzcGVjaWZpZWQgY29sdW1ucyBhbmQgdGhlaXIgc3RhdGlzdGljYWwgYW5hbHlzaXMuCiIiIgpyZXR1cm4geyJkdW1teV9rZXkiOiAiZHVtbXlfdmFsdWUifQpccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKXHBhciMgRHVtbXkgZnVuY3Rpb24gY2FsbCBmb3Iga25vd2xlZGdlX3N1bW1hcnkKa25vd2xlZGdlZnJhbWUgPSB7ImR1bW15X2tleSI6ICJkdW1teV92YWx1ZSJ9CmNvbHVtbnMgPSBbImNvbHVtbjEiLCAiY29sdW1uMiJdCnN0YXRzX2FuYWx5c2lzID0geyJjb2x1bW4xIjogeyJtZWFuIjogNSwgIm1lZGlhbiI6IDR9LCAiY29sdW1uMiI6IHsibWVhbiI6IDEwLCAibWVkaWFuIjogOH19ClxwYXIlKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDE4NzUgKioqKmtub3dsZWRnZV9zdW1tYXJ5KGtub3dsZWRnZWZyYW1lLCBjb2x1bW5zLCBzdGF0c19hbmFseXNpcykK)def knowledge_summary(knowledgeframe, columns, stats_analysis):"""Summarizes a KnowledgeFrame based on specified columns and statistical analysis results.%**** iclr2024_conference.tex Line 1850 ****\parArgs:knowledgeframe (KnowledgeFrame): The KnowledgeFrame tobe summarized.\parcolumns (List[str]): The list of column names to includein the summary.\parstats_analysis (Dict[str, Any]): The dictionary containingstatistical analysis results for the specified columns.\parReturns:dict: A summary dictionary containing information aboutthe specified columns and their statistical analysis."""return {"dummy_key": "dummy_value"}\par\par# Function Call:\par# Dummy function call for knowledge_summaryknowledgeframe = {"dummy_key": "dummy_value"}columns = ["column1", "column2"]stats_analysis = {"column1": {"mean": 5, "median": 4}, "column2": {"mean": 10, "median": 8}}\par%**** iclr2024_conference.tex Line 1875 ****knowledge_summary(knowledgeframe, columns, stats_analysis)
3.  (c)

    ```
    Name: to_grayscale

    Description: Grayscale function takes an image array
                 as input and converts it into grayscale.

                 Args:
                    image_array (beatnum.bdnumset): Input
                    image array to be converted to grayscale.

                 Returns:
                    beatnum.bdnumset: Grayscale image array.

    ```

    [⬇](data:text/plain;base64,CmRlZiB0b19ncmF5c2NhbGUoaW1hZ2VfYXJyYXkpOgoiIiIKR3JheXNjYWxlIGZ1bmN0aW9uIHRha2VzIGFuIGltYWdlIGFycmF5IGFzIGlucHV0IGFuZApjb252ZXJ0cyBpdCBpbnRvIGdyYXlzY2FsZS4KXHBhckFyZ3M6CiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTkwMCAqKioqaW1hZ2VfYXJyYXkgKGJlYXRudW0uYmRudW1zZXQpOiBJbnB1dCBpbWFnZSBhcnJheSB0bwpiZSBjb252ZXJ0ZWQgdG8gZ3JheXNjYWxlLgpccGFyUmV0dXJuczoKYmVhdG51bS5iZG51bXNldDogR3JheXNjYWxlIGltYWdlIGFycmF5LgoiIiIKZHVtbXlfc2hhcGUgPSAoMSwgMSkgIyBEdW1teSBzaGFwZSBmb3IgdGhlIGJkbnVtc2V0CnJldHVybiBiZWF0bnVtLmJkbnVtc2V0KGR1bW15X3NoYXBlKQpccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKZnJvbSBiZWF0bnVtIGltcG9ydCBiZG51bXNldApccGFyIyBEdW1teSBpbWFnZV9hcnJheQpkdW1teV9zaGFwZSA9ICgxLCAxKSAjIER1bW15IHNoYXBlIGZvciB0aGUgYmRudW1zZXQKaW1hZ2VfYXJyYXkgPSBiZG51bXNldChkdW1teV9zaGFwZSkKXHBhclxwYXIjIEZ1bmN0aW9uIGNhbGwKdG9fZ3JheXNjYWxlKGltYWdlX2FycmF5KQo=)def to_grayscale(image_array):"""Grayscale function takes an image array as input andconverts it into grayscale.\parArgs:%**** iclr2024_conference.tex Line 1900 ****image_array (beatnum.bdnumset): Input image array tobe converted to grayscale.\parReturns:beatnum.bdnumset: Grayscale image array."""dummy_shape = (1, 1) # Dummy shape for the bdnumsetreturn beatnum.bdnumset(dummy_shape)\par\par# Function Call:from beatnum import bdnumset\par# Dummy image_arraydummy_shape = (1, 1) # Dummy shape for the bdnumsetimage_array = bdnumset(dummy_shape)\par\par# Function callto_grayscale(image_array)
4.  (d)

    ```
    Name: flip

    Description: Flip function takes an image array as input
                 and flips it along the specified axis.

                 Arg:
                    image_array (beatnum.bdnumset): Input image
                    array to be flipped.

                    axis (int, optional): Axis along which to
                    flip the image array.

                 Returns:
                    beatnum.bdnumset: Flipped image array.

    ```

    [⬇](data:text/plain;base64,CmRlZiBmbGlwKGltYWdlX2FycmF5LCBheGlzPTEpOgoiIiIKRmxpcCBmdW5jdGlvbiB0YWtlcyBhbiBpbWFnZSBhcnJheSBhcyBpbnB1dCBhbmQgZmxpcHMKaXQgYWxvbmcgdGhlIHNwZWNpZmllZCBheGlzLgpccGFyQXJnczoKaW1hZ2VfYXJyYXkgKGJlYXRudW0uYmRudW1zZXQpOiBJbnB1dCBpbWFnZSBhcnJheSB0byBiZQpmbGlwcGVkLgpccGFyJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAxOTUwICoqKipheGlzIChpbnQsIG9wdGlvbmFsKTogQXhpcyBhbG9uZyB3aGljaCB0byBmbGlwIHRoZSBpbWFnZQphcnJheS4gRGVmYXVsdCBpcyAxIChob3Jpem9udGFsIGZsaXApLgpccGFyUmV0dXJuczoKYmVhdG51bS5iZG51bXNldDogRmxpcHBlZCBpbWFnZSBhcnJheS4KIiIiCmR1bW15X3NoYXBlID0gaW1hZ2VfYXJyYXkuc2hhcGUKcmV0dXJuIGJlYXRudW0uYmRudW1zZXQoc2hhcGU9ZHVtbXlfc2hhcGUpClxwYXIjIEZ1bmN0aW9uIENhbGw6CmltcG9ydCBiZWF0bnVtIGFzIGJuClxwYXJpbWFnZV9hcnJheSA9IGJuLmJkbnVtc2V0KHNoYXBlPSgyLCAyKSwgZHR5cGU9ZmxvYXQsIG9yZGVyPSdGJykKYXhpcyA9IDEKXHBhcmZsaXAoaW1hZ2VfYXJyYXksIGF4aXMpCg==)def flip(image_array, axis=1):"""Flip function takes an image array as input and flipsit along the specified axis.\parArgs:image_array (beatnum.bdnumset): Input image array to beflipped.\par%**** iclr2024_conference.tex Line 1950 ****axis (int, optional): Axis along which to flip the imagearray. Default is 1 (horizontal flip).\parReturns:beatnum.bdnumset: Flipped image array."""dummy_shape = image_array.shapereturn beatnum.bdnumset(shape=dummy_shape)\par# Function Call:import beatnum as bn\parimage_array = bn.bdnumset(shape=(2, 2), dtype=float, order=’F’)axis = 1\parflip(image_array, axis)
5.  (e)

    ```
    Name: translate

    Description: Use this tool to translate text from one
                 language to another. The source language will
                 be automatically detected. You need to specify
                 the target language using a two character
                 language code.

                 Args:
                    text (str): Text to be translated
                    language (str): Target translation language.
                    One of af, sq, am, ar, hy, as, az, bn, ba,
                    eu, bs, bg, ca, hr, cs, da, dv, nl, en, et,
                    fo, fj, fi, fr, gl, ka, de, el, gu, ht, he,
                    hi, hu, is, id, iu, ga, it, ja, kn, kk, km,
                    ko, ku, ky, lo, lv, lt, mk, mg, ms, ml, mt,
                    mi, mr, my, ne, nb, or, ps, fa, pl, pt, pa,
                    ro, ru, sm, sk, sl, so, es, sw, sv, ty, ta,
                    tt, te, th, bo, ti, to, tr, tk, uk, ur, ug,
                    uz, vi, cy, zu

    ```

    [⬇](data:text/plain;base64,CmRlZiB0cmFuc2xhdGUodGV4dDogc3RyLCBsYW5ndWFnZTogc3RyKSAtPiBzdHI6CiIiIgpUcmFuc2xhdGVzIHRleHQgZnJvbSBvbmUgbGFuZ3VhZ2UgdG8gYW5vdGhlci4gVGhlIHNvdXJjZQpsYW5ndWFnZSB3aWxsIGJlIGF1dG9tYXRpY2FsbHkgZGV0ZWN0ZWQuWW91IG5lZWQgdG8gc3BlY2lmeQp0aGUgdGFyZ2V0IGxhbmd1YWdlIHVzaW5nIGEgdHdvIGNoYXJhY3RlciBsYW5ndWFnZSBjb2RlLgpccGFyJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMDAwICoqKipBcmdzOgp0ZXh0IChzdHIpOiBUZXh0IHRvIGJlIHRyYW5zbGF0ZWQKXHBhcmxhbmd1YWdlIChzdHIpOiBUYXJnZXQgdHJhbnNsYXRpb24gbGFuZ3VhZ2UuIE9uZSBvZgphZiwgc3EsIGFtLCBhciwgaHksIGFzLCBheiwgYm4sIGJhLCBldSwgYnMsIGJnLCBjYSwKaHIsIGNzLCBkYSwgZHYsIG5sLCBlbiwgZXQsIGZvLCBmaiwgZmksIGZyLCBnbCwga2EsCmRlLCBlbCwgZ3UsIGh0LCBoZSwgaGksIGh1LCBpcywgaWQsIGl1LCBnYSwgaXQsIGphLAprbiwga2ssIGttLCBrbywga3UsIGt5LCBsbywgbHYsIGx0LCBtaywgbWcsIG1zLCBtbCwKbXQsIG1pLCBtciwgbXksIG5lLCBuYiwgb3IsIHBzLCBmYSwgcGwsIHB0LCBwYSwgdG8sCnJ1LCBzbSwgc2ssIHNsLCBzbywgZXMsIHN3LCBzdiwgdHksIHRhLCB0dCwgdGUsIHRoZSwKYm8sIHRpLCB0bywgdHIsIHRrLCB1aywgdXIsIHVnLCB1eiwgdmksIGN5LCB6dQpccGFyUmV0dXJuczoKc3RyOiBUcmFuc2xhdGVkIHRleHQKIiIiCnJldHVybiAiZHVtbXlfdHJhbnNsYXRlZF90ZXh0IgpccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKdGV4dCA9ICJIZWxsbywgaG93IGFyZSB5b3U/IgpsYW5ndWFnZSA9ICJmciIKdHJhbnNsYXRlKHRleHQsIGxhbmd1YWdlKQo=)def translate(text: str, language: str) -> str:"""Translates text from one language to another. The sourcelanguage will be automatically detected.You need to specifythe target language using a two character language code.\par%**** iclr2024_conference.tex Line 2000 ****Args:text (str): Text to be translated\parlanguage (str): Target translation language. One ofaf, sq, am, ar, hy, as, az, bn, ba, eu, bs, bg, ca,hr, cs, da, dv, nl, en, et, fo, fj, fi, fr, gl, ka,de, el, gu, ht, he, hi, hu, is, id, iu, ga, it, ja,kn, kk, km, ko, ku, ky, lo, lv, lt, mk, mg, ms, ml,mt, mi, mr, my, ne, nb, or, ps, fa, pl, pt, pa, to,ru, sm, sk, sl, so, es, sw, sv, ty, ta, tt, te, the,bo, ti, to, tr, tk, uk, ur, ug, uz, vi, cy, zu\parReturns:str: Translated text"""return "dummy_translated_text"\par\par# Function Call:text = "Hello, how are you?"language = "fr"translate(text, language)

### A.5 TOPGUN Examples

Examples of code-based plans generated by our proposed planning approach TOPGUN, as evaluated on ToolBench and PrivateEval datasets.

#### A.5.1 ToolBench

1.  (a)

    ```
    Query: My friends and I are eagerly awaiting the delivery of a
    package. Can you please track the package with the Pack & Send
    reference number ’ReferenceNumberHere’? Additionally, I’m
    interested in the latest status of the package with colis ID
    ’CA107308006SI’

    ```

    [⬇](data:text/plain;base64,CiMgSW1wb3J0IHRoZSByZXF1aXJlZCBsaWJyYXJpZXMKaW1wb3J0IHJlcXVlc3RzClxwYXIlKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDIwNTAgKioqKiMgVXNlIHRoZSBhcGlfdHJhY2tpbmcgZnVuY3Rpb24gdG8gdHJhY2sgdGhlIHBhY2thZ2Ugd2l0aAojIHRoZSBQYWNrICYgU2VuZCByZWZlcmVuY2UgbnVtYmVyCnJlZmVyZW5jZV9udW1iZXIgPSAnUmVmZXJlbmNlTnVtYmVySGVyZScKcGFja2FnZV90cmFja2luZ19pbmZvID0gYXBpX3RyYWNraW5nKHJlZmVyZW5jZV9udW1iZXIpClxwYXIjIFVzZSB0aGUgbGF0ZXN0IGZ1bmN0aW9uIHRvIGdldCB0aGUgbGF0ZXN0IHN0YXR1cyBvZiB0aGUKIyBwYWNrYWdlIHdpdGggY29saXMgSUQKY29saXNfaWQgPSAnQ0ExMDczMDgwMDZTSScKbGF0ZXN0X3N0YXR1cyA9IGxhdGVzdChjb2xpc19pZCkKXHBhciMgVGhlIHBhY2thZ2VfdHJhY2tpbmdfaW5mbyBhbmQgbGF0ZXN0X3N0YXR1cyB2YXJpYWJsZXMKIyBjb250YWluIHRoZSByZXF1aXJlZCBpbmZvcm1hdGlvbgo=)# Import the required librariesimport requests\par%**** iclr2024_conference.tex Line 2050 ****# Use the api_tracking function to track the package with# the Pack & Send reference numberreference_number = ’ReferenceNumberHere’package_tracking_info = api_tracking(reference_number)\par# Use the latest function to get the latest status of the# package with colis IDcolis_id = ’CA107308006SI’latest_status = latest(colis_id)\par# The package_tracking_info and latest_status variables# contain the required information
2.  (b)

    ```
    Query: I’m a movie critic and I need to write reviews for the
    latest movies. Can you provide me with a list of new arrivals
    on different platforms? It would be great if you could include
    the streaming platforms and the genres for each movie.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDIwNzUgKioqKlxwYXIjIEdldCB0aGUgbGF0ZXN0IGFycml2YWxzIGZyb20gZGlmZmVyZW50IHBsYXRmb3JtcwpuZXdfYXJyaXZhbHNfZGF0YSA9IG5ld19hcnJpdmFscyhyZWdpb249J1VTJykKXHBhciMgSW5pdGlhbGl6ZSBhbiBlbXB0eSBsaXN0IHRvIHN0b3JlIHRoZSBtb3ZpZSBkZXRhaWxzCm1vdmllX2RldGFpbHMgPSBbXQpccGFyIyBJdGVyYXRlIHRocm91Z2ggdGhlIG5ldyBhcnJpdmFscyBkYXRhCmZvciBtb3ZpZSBpbiBuZXdfYXJyaXZhbHNfZGF0YS5nZXQoJ3Jlc3VsdCcsIFtdKToKIyBHZXQgdGhlIElNRGIgSUQgb2YgdGhlIG1vdmllCmltZGJfaWQgPSBtb3ZpZS5nZXQoJ2ltZGJpZCcsICcnKQpccGFyIyBHZXQgdGhlIGJhc2ljIGluZm9ybWF0aW9uIG9mIHRoZSBtb3ZpZSB1c2luZyB0aGUgSU1EYiBJRAp0aXRsZV9kYXRhID0gdGl0bGVfZGV0YWlscyhpbWRiaWQ9aW1kYl9pZCkKXHBhciMgRXh0cmFjdCB0aGUgcmVxdWlyZWQgaW5mb3JtYXRpb24gZnJvbSB0aGUgdGl0bGUgZGF0YQptb3ZpZV90aXRsZSA9IHRpdGxlX2RhdGEuZ2V0KCd0aXRsZScsICcnKQpzdHJlYW1pbmdfcGxhdGZvcm1zID0gdGl0bGVfZGF0YS5nZXQoJ3BsYXRmb3JtcycsIHt9KQpnZW5yZXMgPSB0aXRsZV9kYXRhLmdldCgnZ2VucmUnLCAnJykKXHBhciMgQXBwZW5kIHRoZSBtb3ZpZSBkZXRhaWxzIHRvIHRoZSBtb3ZpZV9kZXRhaWxzIGxpc3QKbW92aWVfZGV0YWlscy5hcHBlbmQoewondGl0bGUnOiBtb3ZpZV90aXRsZSwKJ3N0cmVhbWluZ19wbGF0Zm9ybXMnOiBzdHJlYW1pbmdfcGxhdGZvcm1zLAonZ2VucmVzJzogZ2VucmVzCn0pClxwYXIjIFRoZSBtb3ZpZV9kZXRhaWxzIGxpc3Qgbm93IGNvbnRhaW5zIHRoZSBuZXcgYXJyaXZhbHMKIyBhbG9uZyB3aXRoIHRoZWlyIHN0cmVhbWluZyBwbGF0Zm9ybXMgYW5kIGdlbnJlcwo=)import requests%**** iclr2024_conference.tex Line 2075 ****\par# Get the latest arrivals from different platformsnew_arrivals_data = new_arrivals(region=’US’)\par# Initialize an empty list to store the movie detailsmovie_details = []\par# Iterate through the new arrivals datafor movie in new_arrivals_data.get(’result’, []):# Get the IMDb ID of the movieimdb_id = movie.get(’imdbid’, ’’)\par# Get the basic information of the movie using the IMDb IDtitle_data = title_details(imdbid=imdb_id)\par# Extract the required information from the title datamovie_title = title_data.get(’title’, ’’)streaming_platforms = title_data.get(’platforms’, {})genres = title_data.get(’genre’, ’’)\par# Append the movie details to the movie_details listmovie_details.append({’title’: movie_title,’streaming_platforms’: streaming_platforms,’genres’: genres})\par# The movie_details list now contains the new arrivals# along with their streaming platforms and genres
3.  (c)

    ```
    Query: I’m hosting a virtual movie night with my friends and
    I need some suggestions. Can you search for videos related to
    ’action’ on Vimeo? Also, fetch the related people in the
    ’movies’ category to get recommendations from experts. Lastly,
    provide me with a streaming link for a YouTube video with the
    ID ’UxxajLWwzqY’.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBTZWFyY2ggZm9yIHZpZGVvcyByZWxhdGVkIHRvICdhY3Rpb24nIG9uIFZpbWVvCmFjdGlvbl92aWRlb3MgPSBzZWFyY2h2aWRlb3MoZm9ybWF0PSdqc29uJywgcXVlcnk9J2FjdGlvbicsIHNvcnQ9J3JlbGV2YW50JykKXHBhciMgRmV0Y2ggdGhlIHJlbGF0ZWQgcGVvcGxlIGluIHRoZSAnbW92aWVzJyBjYXRlZ29yeQpyZWxhdGVkX3Blb3BsZSA9IGdldHJlbGF0ZWRwZW9wbGUoY2F0ZWdvcnk9J21vdmllcycsIGZvcm1hdD0nanNvbicpCiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMjEyNSAqKioqXHBhciMgUHJvdmlkZSBhIHN0cmVhbWluZyBsaW5rIGZvciBhIFlvdVR1YmUgdmlkZW8gd2l0aCB0aGUgSUQKIyAnVXh4YWpMV3d6cVknCnlvdXR1YmVfc3RyZWFtaW5nX2xpbmsgPSBkb3dubG9hZF9zdHJlYW0oaXNfaWQ9J1V4eGFqTFd3enFZJykK)import requests\par# Search for videos related to ’action’ on Vimeoaction_videos = searchvideos(format=’json’, query=’action’, sort=’relevant’)\par# Fetch the related people in the ’movies’ categoryrelated_people = getrelatedpeople(category=’movies’, format=’json’)%**** iclr2024_conference.tex Line 2125 ****\par# Provide a streaming link for a YouTube video with the ID# ’UxxajLWwzqY’youtube_streaming_link = download_stream(is_id=’UxxajLWwzqY’)
4.  (d)

    ```
    Query: I am a fitness enthusiast and I want to buy a fitness
    tracker. Can you suggest some top-rated fitness trackers
    available on Amazon along with their features and prices?

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBVc2UgdGhlIHNlYXJjaCBmdW5jdGlvbiB0byBmaW5kIHRvcC1yYXRlZCBmaXRuZXNzIHRyYWNrZXJzCiMgb24gQW1hem9uCnNlYXJjaF9yZXN1bHRzID0gc2VhcmNoKHR5cGU9J3NlYXJjaCcsIHNlYXJjaF90ZXJtPSdmaXRuZXNzIHRyYWNrZXInLCBhbWF6b25fZG9tYWluPSdhbWF6b24uY29tJywgc29ydF9ieT0nYXZlcmFnZV9yZXZpZXcnLCBleGNsdWRlX3Nwb25zb3JlZD1UcnVlKQpccGFyIyBFeHRyYWN0IHRoZSB0b3AgNSBmaXRuZXNzIHRyYWNrZXJzIGZyb20gdGhlIHNlYXJjaCByZXN1bHRzCnRvcF81X2ZpdG5lc3NfdHJhY2tlcnMgPSBzZWFyY2hfcmVzdWx0cy5nZXQoJ3Jlc3VsdHMnLCBbXSlbOjVdClxwYXIjIEdldCB0aGUgQVNJTnMgb2YgdGhlIHRvcCA1IGZpdG5lc3MgdHJhY2tlcnMKJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMTUwICoqKip0b3BfNV9hc2lucyA9IFt0cmFja2VyLmdldCgnYXNpbicsICcnKSBmb3IgdHJhY2tlciBpbiB0b3BfNV9maXRuZXNzX3RyYWNrZXJzXQpccGFyIyBSZXRyaWV2ZSB0aGUgcHJvZHVjdCBkZXRhaWxzIGZvciBlYWNoIG9mIHRoZSB0b3AgNSBmaXRuZXNzCiMgdHJhY2tlcnMKdG9wXzVfcHJvZHVjdF9kZXRhaWxzID0gW3Byb2R1Y3QodHlwZT0ncHJvZHVjdCcsIGFzaW49YXNpbiwgYW1hem9uX2RvbWFpbj0nYW1hem9uLmNvbScpIGZvciBhc2luIGluIHRvcF81X2FzaW5zXQpccGFyIyBFeHRyYWN0IHRoZSBmZWF0dXJlcyBhbmQgcHJpY2VzIG9mIHRoZSB0b3AgNSBmaXRuZXNzIHRyYWNrZXJzCnRvcF81X2ZlYXR1cmVzX2FuZF9wcmljZXMgPSBbXQpmb3IgcHJvZHVjdF9kZXRhaWwgaW4gdG9wXzVfcHJvZHVjdF9kZXRhaWxzOgp0cnk6CmZlYXR1cmVzID0gcHJvZHVjdF9kZXRhaWwuZ2V0KCdmZWF0dXJlcycsIFtdKQpleGNlcHQ6CmZlYXR1cmVzID0gW10KcHJpY2UgPSBwcm9kdWN0X2RldGFpbC5nZXQoJ3ByaWNlJywge30pLmdldCgndmFsdWUnLCAnTi9BJykKdG9wXzVfZmVhdHVyZXNfYW5kX3ByaWNlcy5hcHBlbmQoeydmZWF0dXJlcyc6IGZlYXR1cmVzLCAncHJpY2UnOiBwcmljZX0pClxwYXIjIFRoZSB0b3BfNV9mZWF0dXJlc19hbmRfcHJpY2VzIHZhcmlhYmxlIGNvbnRhaW5zIHRoZSBmZWF0dXJlcwojIGFuZCBwcmljZXMgb2YgdGhlIHRvcCA1IGZpdG5lc3MgdHJhY2tlcnMgb24gQW1hem9uCg==)import requests\par# Use the search function to find top-rated fitness trackers# on Amazonsearch_results = search(type=’search’, search_term=’fitness tracker’, amazon_domain=’amazon.com’, sort_by=’average_review’, exclude_sponsored=True)\par# Extract the top 5 fitness trackers from the search resultstop_5_fitness_trackers = search_results.get(’results’, [])[:5]\par# Get the ASINs of the top 5 fitness trackers%**** iclr2024_conference.tex Line 2150 ****top_5_asins = [tracker.get(’asin’, ’’) for tracker in top_5_fitness_trackers]\par# Retrieve the product details for each of the top 5 fitness# trackerstop_5_product_details = [product(type=’product’, asin=asin, amazon_domain=’amazon.com’) for asin in top_5_asins]\par# Extract the features and prices of the top 5 fitness trackerstop_5_features_and_prices = []for product_detail in top_5_product_details:try:features = product_detail.get(’features’, [])except:features = []price = product_detail.get(’price’, {}).get(’value’, ’N/A’)top_5_features_and_prices.append({’features’: features, ’price’: price})\par# The top_5_features_and_prices variable contains the features# and prices of the top 5 fitness trackers on Amazon
5.  (e)

    ```
    Query: I’m a cryptocurrency trader and I want to analyze the
    historical prices and market caps of popular cryptocurrencies
    like Bitcoin, Ethereum, and Stellar. Can you fetch this
    information for me using the Crypto Prices API? Additionally,
    I’m planning a trip to North America and I would like to know
    the subregions in North America using the Geography API.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBGZXRjaCBjcnlwdG9jdXJyZW5jeSBkYXRhCmNyeXB0b19kYXRhID0gcHJpY2VzX2FuZF91cF9hbmRfZG93bigpClxwYXIjIEZldGNoIHN1YnJlZ2lvbnMgb2YgTm9ydGggQW1lcmljYQpzdWJyZWdpb25zX2RhdGEgPSBnZXRfc3ViX3JlZ2lvbnMoJ05vcnRoIEFtZXJpY2EnKQpccGFyIyBBY2Nlc3Npbmcgc3BlY2lmaWMgY3J5cHRvY3VycmVuY3kgZGF0YQpiaXRjb2luX2RhdGEgPSBjcnlwdG9fZGF0YS5nZXQoJ0JpdGNvaW4nLCB7fSkKZXRoZXJldW1fZGF0YSA9IGNyeXB0b19kYXRhLmdldCgnRXRoZXJldW0nLCB7fSkKc3RlbGxhcl9kYXRhID0gY3J5cHRvX2RhdGEuZ2V0KCdTdGVsbGFyJywge30pClxwYXIjIEFjY2Vzc2luZyBzdWJyZWdpb25zIG9mIE5vcnRoIEFtZXJpY2EKdHJ5Ogpub3J0aF9hbWVyaWNhX3N1YnJlZ2lvbnMgPSBzdWJyZWdpb25zX2RhdGEuZ2V0KCdzdWJyZWdpb25zJywgW10pCmV4Y2VwdDoKbm9ydGhfYW1lcmljYV9zdWJyZWdpb25zID0gW10KJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMjAwICoqKipccGFyIyBZb3UgY2FuIG5vdyBhbmFseXplIHRoZSBjcnlwdG9jdXJyZW5jeSBkYXRhIGFuZCBwbGFuIHlvdXIKIyB0cmlwIHRvIE5vcnRoIEFtZXJpY2EgdXNpbmcgdGhlIHN1YnJlZ2lvbnMgaW5mb3JtYXRpb24uCg==)import requests\par# Fetch cryptocurrency datacrypto_data = prices_and_up_and_down()\par# Fetch subregions of North Americasubregions_data = get_sub_regions(’North America’)\par# Accessing specific cryptocurrency databitcoin_data = crypto_data.get(’Bitcoin’, {})ethereum_data = crypto_data.get(’Ethereum’, {})stellar_data = crypto_data.get(’Stellar’, {})\par# Accessing subregions of North Americatry:north_america_subregions = subregions_data.get(’subregions’, [])except:north_america_subregions = []%**** iclr2024_conference.tex Line 2200 ****\par# You can now analyze the cryptocurrency data and plan your# trip to North America using the subregions information.
6.  (f)

    ```
    Query: I need to find a tutorial on how to draw landscapes.
    Please provide me with the details of the most viewed
    landscape drawing tutorial video. Additionally, I would like
    to know the details of the channel that uploaded the video.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBTZWFyY2ggZm9yIGxhbmRzY2FwZSBkcmF3aW5nIHR1dG9yaWFsIHZpZGVvcwpzZWFyY2hfcmVzdWx0ID0gc2VhcmNoKCdsYW5kc2NhcGUgZHJhd2luZyB0dXRvcmlhbCcsIHR5cGU9J3ZpZGVvJywgc2FmZXNlYXJjaD1UcnVlKQpccGFyIyBGaW5kIHRoZSBtb3N0IHZpZXdlZCB2aWRlbwptb3N0X3ZpZXdlZF92aWRlbyA9IE5vbmUKbWF4X3ZpZXdzID0gMApmb3IgdmlkZW8gaW4gc2VhcmNoX3Jlc3VsdC5nZXQoJ2l0ZW1zJywgW10pOgp2aWV3cyA9IGludCh2aWRlby5nZXQoJ3N0YXRpc3RpY3MnLCB7fSkuZ2V0KCd2aWV3Q291bnQnLCAnMCcpKQolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDIyMjUgKioqKmlmIHZpZXdzID4gbWF4X3ZpZXdzOgptYXhfdmlld3MgPSB2aWV3cwptb3N0X3ZpZXdlZF92aWRlbyA9IHZpZGVvClxwYXIjIEdldCB0aGUgdmlkZW8gZGV0YWlscwp2aWRlb19pZCA9IG1vc3Rfdmlld2VkX3ZpZGVvLmdldCgnaWQnLCAndmlkZW9faWRfZXhhbXBsZScpCnZpZGVvX2RldGFpbHMgPSB2aWRlbyh2aWRlb19pZCkKXHBhciMgR2V0IHRoZSBjaGFubmVsIGRldGFpbHMKY2hhbm5lbF9pZCA9IG1vc3Rfdmlld2VkX3ZpZGVvLmdldCgnc25pcHBldCcsIHt9KS5nZXQoJ2NoYW5uZWxJZCcsICdjaGFubmVsX2lkX2V4YW1wbGUnKQpjaGFubmVsX2RldGFpbHMgPSB2aWRlbyhjaGFubmVsX2lkKQpccGFyIyBUaGUgdmlkZW9fZGV0YWlscyBhbmQgY2hhbm5lbF9kZXRhaWxzIHZhcmlhYmxlcyBjb250YWluIHRoZQojIHJlcXVpcmVkIGluZm9ybWF0aW9uCg==)import requests\par# Search for landscape drawing tutorial videossearch_result = search(’landscape drawing tutorial’, type=’video’, safesearch=True)\par# Find the most viewed videomost_viewed_video = Nonemax_views = 0for video in search_result.get(’items’, []):views = int(video.get(’statistics’, {}).get(’viewCount’, ’0’))%**** iclr2024_conference.tex Line 2225 ****if views > max_views:max_views = viewsmost_viewed_video = video\par# Get the video detailsvideo_id = most_viewed_video.get(’id’, ’video_id_example’)video_details = video(video_id)\par# Get the channel detailschannel_id = most_viewed_video.get(’snippet’, {}).get(’channelId’, ’channel_id_example’)channel_details = video(channel_id)\par# The video_details and channel_details variables contain the# required information

#### A.5.2 PrivateEval

1.  (a)

    ```
    Query: Could you help me load a multilingual dataset? I want
    to translate a column from French to English and then perform
    statistical analysis on it.

    ```

    [⬇](data:text/plain;base64,CiMgTG9hZCB0aGUgbXVsdGlsaW5ndWFsIGRhdGFzZXQgKGFzc3VtaW5nIGl0J3MgYSBDU1YgZmlsZSkKY3N2X2ZpbGUgPSAncGF0aC90by95b3VyL211bHRpbGluZ3VhbF9kYXRhc2V0LmNzdicKa25vd2xlZGdlZnJhbWUgPSBsb2FkX2Nzdihjc3ZfZmlsZSkKXHBhciMgRXh0cmFjdCB0aGUgRnJlbmNoIGNvbHVtbiAoYXNzdW1pbmcgaXQncyBuYW1lZCAnZnJlbmNoX2NvbHVtbicpCmZyZW5jaF9jb2x1bW4gPSBrbm93bGVkZ2VmcmFtZSgnZnJlbmNoX2NvbHVtbicpClxwYXIjIFRyYW5zbGF0ZSB0aGUgRnJlbmNoIGNvbHVtbiB0byBFbmdsaXNoCnRyYW5zbGF0ZWRfY29sdW1uID0gW10KZm9yIHRleHQgaW4gZnJlbmNoX2NvbHVtbjoKdHJhbnNsYXRlZF90ZXh0ID0gdHJhbnNsYXRlKHRleHQsIGxhbmd1YWdlPSdlbicpCnRyYW5zbGF0ZWRfY29sdW1uLmFwcGVuZCh0cmFuc2xhdGVkX3RleHQpClxwYXIjIEFkZCB0aGUgdHJhbnNsYXRlZCBjb2x1bW4gdG8gdGhlIEtub3dsZWRnZUZyYW1lCmtub3dsZWRnZWZyYW1lWydlbmdsaXNoX2NvbHVtbiddID0gdHJhbnNsYXRlZF9jb2x1bW4KXHBhciMgUGVyZm9ybSBzdGF0aXN0aWNhbCBhbmFseXNpcyBvbiB0aGUgdHJhbnNsYXRlZCBjb2x1bW4KYW5hbHlzaXNfa2YgPSBzdGF0c19hbmFseXNpcyhrbm93bGVkZ2VmcmFtZSkKJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMjc1ICoqKio=)# Load the multilingual dataset (assuming it’s a CSV file)csv_file = ’path/to/your/multilingual_dataset.csv’knowledgeframe = load_csv(csv_file)\par# Extract the French column (assuming it’s named ’french_column’)french_column = knowledgeframe(’french_column’)\par# Translate the French column to Englishtranslated_column = []for text in french_column:translated_text = translate(text, language=’en’)translated_column.append(translated_text)\par# Add the translated column to the KnowledgeFrameknowledgeframe[’english_column’] = translated_column\par# Perform statistical analysis on the translated columnanalysis_kf = stats_analysis(knowledgeframe)%**** iclr2024_conference.tex Line 2275 ****
2.  (b)

    ```
    Query: Load data regarding baby food preferences, analyze
    the preferences across different age groups, and generate
    a report summarizing the most preferred food items

    ```

    [⬇](data:text/plain;base64,CiMgTG9hZCB0aGUgQ1NWIGZpbGUgY29udGFpbmluZyB0aGUgc3VydmV5IGRhdGEKc3VydmV5X2RhdGEgPSBsb2FkX2NzdignYmFieV9mb29kX3ByZWZlcmVuY2VzLmNzdicpClxwYXIjIFBlcmZvcm0gc3RhdGlzdGljYWwgYW5hbHlzaXMgb24gdGhlIHN1cnZleSBkYXRhCmFuYWx5c2lzX3Jlc3VsdHMgPSBzdGF0c19hbmFseXNpcyhzdXJ2ZXlfZGF0YSkKXHBhciMgR2VuZXJhdGUgYSBoaXN0b2dyYW0gdG8gdmlzdWFsaXplIHRoZSBwcmVmZXJlbmNlcyBhY3Jvc3MgZGlmZmVyZW50IGFnZSBncm91cHMKYnVpbGRfaGlzdChhbmFseXNpc19yZXN1bHRzKQpccGFyIyBTdW1tYXJpemUgdGhlIG1vc3QgcHJlZmVycmVkIGZvb2QgaXRlbXMgaW4gYSByZXBvcnQKY29sdW1uc190b19pbmNsdWRlID0gWydhZ2VfZ3JvdXAnLCAnZm9vZF9pdGVtJywgJ3ByZWZlcmVuY2Vfc2NvcmUnXQpzdW1tYXJ5X3JlcG9ydCA9IGtub3dsZWRnZV9zdW1tYXJ5KHN1cnZleV9kYXRhLCBjb2x1bW5zX3RvX2luY2x1ZGUsIGFuYWx5c2lzX3Jlc3VsdHMpClxwYXIjIFRoZSBzdW1tYXJ5X3JlcG9ydCB2YXJpYWJsZSBub3cgY29udGFpbnMgdGhlIHJlcG9ydCBzdW1tYXJpemluZyB0aGUgbW9zdCBwcmVmZXJyZWQgZm9vZCBpdGVtcwo=)# Load the CSV file containing the survey datasurvey_data = load_csv(’baby_food_preferences.csv’)\par# Perform statistical analysis on the survey dataanalysis_results = stats_analysis(survey_data)\par# Generate a histogram to visualize the preferences across different age groupsbuild_hist(analysis_results)\par# Summarize the most preferred food items in a reportcolumns_to_include = [’age_group’, ’food_item’, ’preference_score’]summary_report = knowledge_summary(survey_data, columns_to_include, analysis_results)\par# The summary_report variable now contains the report summarizing the most preferred food items
3.  (c)

    ```
    Query: Could you help me find the Chinchilla LLM paper? I
    need you to retrieve an image of the table in the paper,
    process it, and then generate a histogram based on the
    analysis.

    ```

    [⬇](data:text/plain;base64,CmZyb20gbW9ua2V5IGltcG9ydCBLbm93bGVkZ2VGcmFtZQpmcm9tIHR5cGluZyBpbXBvcnQgTGlzdCwgRGljdCwgVW5pb24KXHBhciMgU2VhcmNoIGZvciAiQ2hpbmNoaWxsYSBMTE0iIHBhcGVyIG9uIGFyWGl2CnF1ZXJ5ID0gIkNoaW5jaGlsbGEgTExNIgpwYXBlcnMgPSBhcnhpdl9xdWVyeShxdWVyeSkKXHBhciMgQXNzdW1pbmcgZmlyc3QgcmVzdWx0IGlzIHJlbGV2YW50IGxvYWQgdGhlIFBERiBkYXRhCnBkZl9kYXRhID0gbG9hZF9wZGZfZGF0YShwYXBlcnNbMF0pClxwYXIjIFNlYXJjaCB0aGUgaW1hZ2Ugb2YgdGhlIHRhYmxlIGluIHRoZSAiQ2hpbmNoaWxsYSBMTE0iIHBhcGVyCmltYWdlX3VybCA9IGJpbmdfaW1hZ2Vfc2VhcmNoKCdDaGluY2hpbGxhIExMTSBwYXBlciB0YWJsZScpClxwYXIjIFByb2Nlc3MgdGhlIGltYWdlCnByb2Nlc3NlZF9pbWFnZSA9IHByb2Nlc3NfaW1hZ2UoaW1hZ2VfdXJsLCBmZWF0dXJlcz1bJ29iamVjdHMnXSkKJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMzI1ICoqKipccGFyIyBDb252ZXJ0IHRoZSBwcm9jZXNzZWQgaW1hZ2UgdG8gYSBLbm93bGVkZ2VGcmFtZQprbm93bGVkZ2VfZnJhbWUgPSByZWFkX3R4dChwcm9jZXNzZWRfaW1hZ2UuZ2V0KCdvYmplY3RzJywgJ2R1bW15X29iamVjdHNfdGV4dCcpKQpccGFyIyBQZXJmb3JtIHN0YXRpc3RpY2FsIGFuYWx5c2lzIG9uIHRoZSBLbm93bGVkZ2VGcmFtZQphbmFseXNpc19rZiA9IHN0YXRzX2FuYWx5c2lzKGtub3dsZWRnZV9mcmFtZSkKXHBhciMgQnVpbGQgYSBoaXN0b2dyYW0gYmFzZWQgb24gdGhlIGFuYWx5c2lzCmJ1aWxkX2hpc3QoYW5hbHlzaXNfa2YpCg==)from monkey import KnowledgeFramefrom typing import List, Dict, Union\par# Search for "Chinchilla LLM" paper on arXivquery = "Chinchilla LLM"papers = arxiv_query(query)\par# Assuming first result is relevant load the PDF datapdf_data = load_pdf_data(papers[0])\par# Search the image of the table in the "Chinchilla LLM" paperimage_url = bing_image_search(’Chinchilla LLM paper table’)\par# Process the imageprocessed_image = process_image(image_url, features=[’objects’])%**** iclr2024_conference.tex Line 2325 ****\par# Convert the processed image to a KnowledgeFrameknowledge_frame = read_txt(processed_image.get(’objects’, ’dummy_objects_text’))\par# Perform statistical analysis on the KnowledgeFrameanalysis_kf = stats_analysis(knowledge_frame)\par# Build a histogram based on the analysisbuild_hist(analysis_kf)
4.  (d)

    ```
    Query: Could you please retrieve some images of dogs? After
    that, process it and perform data augmentation using simple
    image processing techniques and save the augmented images.

    ```

    [⬇](data:text/plain;base64,CiMgSW1wb3J0IG5lY2Vzc2FyeSBsaWJyYXJpZXMKZnJvbSB0eXBpbmcgaW1wb3J0IExpc3QsIERpY3QsIFVuaW9uCmltcG9ydCBiZWF0bnVtCmZyb20gUElMIGltcG9ydCBJbWFnZQpccGFyIyBSZXRyaWV2ZSBpbWFnZXMgb2YgZG9ncwolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDIzNTAgKioqKmltYWdlX3VybHMgPSBiaW5nX2ltYWdlX3NlYXJjaChxdWVyeT0nZG9ncycpClxwYXIjIExvYWQgdGhlIGZpcnN0IGltYWdlCmltYWdlID0gcHJvY2Vzc19pbWFnZShpbWFnZV91cmxzWzBdKQppbWFnZV9hcnJheSA9IGJlYXRudW0uYmRudW1zZXQoaW1hZ2UpClxwYXIjIFBlcmZvcm0gZGF0YSBhdWdtZW50YXRpb24gdXNpbmcgc2ltcGxlIGltYWdlIHByb2Nlc3NpbmcgdGVjaG5pcXVlcwpyb3RhdGVkX2ltYWdlX2FycmF5ID0gcm90YXRlKGltYWdlX2FycmF5LCBkaXJlY3Rpb249J2Nsb2Nrd2lzZScpCmZsaXBwZWRfaW1hZ2VfYXJyYXkgPSBmbGlwKGltYWdlX2FycmF5LCBheGlzPTEpCmNyb3BwZWRfaW1hZ2VfYXJyYXkgPSBjcm9wKGltYWdlX2FycmF5LCBzY2FsZV9mYWN0b3I9MC41KQpncmF5c2NhbGVfaW1hZ2VfYXJyYXkgPSB0b19ncmF5c2NhbGUoaW1hZ2VfYXJyYXkpClxwYXIjIGNyZWF0aW5nIGltYWdlIG9iamVjdCBvZiBhYm92ZSBhcnJheQpyb3RhdGVkX2ltYWdlX2RhdGEgPSBJbWFnZS5mcm9tYXJyYXkocm90YXRlZF9pbWFnZV9hcnJheSkKZmxpcHBlZF9pbWFnZV9kYXRhID0gSW1hZ2UuZnJvbWFycmF5KGZsaXBwZWRfaW1hZ2VfYXJyYXkpCmNyb3BwZWRfaW1hZ2VfZGF0YSA9IEltYWdlLmZyb21hcnJheShjcm9wcGVkX2ltYWdlX2FycmF5KQpncmF5c2NhbGVfaW1hZ2VfZGF0YSA9IEltYWdlLmZyb21hcnJheShncmF5c2NhbGVfaW1hZ2VfYXJyYXkpClxwYXIjIFNhdmUgdGhlIGF1Z21lbnRlZCBpbWFnZXMKcm90YXRlZF9pbWFnZV9kYXRhLnNhdmUoJ3JvdGF0ZWRfZG9nX2ltYWdlLnBuZycpCmZsaXBwZWRfaW1hZ2VfZGF0YS5zYXZlKCdmbGlwcGVkX2RvZ19pbWFnZS5wbmcnKQpjcm9wcGVkX2ltYWdlX2RhdGEuc2F2ZSgnY3JvcHBlZF9kb2dfaW1hZ2UucG5nJykKZ3JheXNjYWxlX2ltYWdlX2RhdGEuc2F2ZSgnZ3JheXNjYWxlX2RvZ19pbWFnZS5wbmcnKQo=)# Import necessary librariesfrom typing import List, Dict, Unionimport beatnumfrom PIL import Image\par# Retrieve images of dogs%**** iclr2024_conference.tex Line 2350 ****image_urls = bing_image_search(query=’dogs’)\par# Load the first imageimage = process_image(image_urls[0])image_array = beatnum.bdnumset(image)\par# Perform data augmentation using simple image processing techniquesrotated_image_array = rotate(image_array, direction=’clockwise’)flipped_image_array = flip(image_array, axis=1)cropped_image_array = crop(image_array, scale_factor=0.5)grayscale_image_array = to_grayscale(image_array)\par# creating image object of above arrayrotated_image_data = Image.fromarray(rotated_image_array)flipped_image_data = Image.fromarray(flipped_image_array)cropped_image_data = Image.fromarray(cropped_image_array)grayscale_image_data = Image.fromarray(grayscale_image_array)\par# Save the augmented imagesrotated_image_data.save(’rotated_dog_image.png’)flipped_image_data.save(’flipped_dog_image.png’)cropped_image_data.save(’cropped_dog_image.png’)grayscale_image_data.save(’grayscale_dog_image.png’)