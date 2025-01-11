<!--yml
category: 未分类
date: 2025-01-11 12:39:01
-->

# AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents

> 来源：[https://arxiv.org/html/2405.06907/](https://arxiv.org/html/2405.06907/)

Shuyuan Xu
Rutgers University
shuyuan.xu@rutgers.edu
Zelong Li ¹¹footnotemark: 1
Rutgers University
zelong.li@rutgers.edu
Kai Mei
Rutgers University
kai.mei@rutgers.edu
Yongfeng Zhang
Rutgers University
yongfeng.zhang@rutgers.edu Both authors contributed equally to this work.Corresponding author

###### Abstract

Since their inception, programming languages have trended towards greater readability and lower barriers for programmers. Following this trend, natural language can be a promising type of programming language that provides great flexibility and usability and helps towards the democracy of programming. However, the inherent vagueness, ambiguity, and verbosity of natural language pose significant challenges in developing an interpreter that can accurately understand the programming logic and execute instructions written in natural language. Fortunately, recent advancements in Large Language Models (LLMs) have demonstrated remarkable proficiency in interpreting complex natural language. Inspired by this, we develop a novel system for Code Representation and Execution (CoRE), which employs LLM as interpreter to interpret and execute natural language programs (NLPg). The proposed system unifies natural language programming, pseudo-code programming, and flow programming under the same representation for constructing language agents, while LLM serves as the interpreter to interpret and execute the agent programs. In this paper, we begin with defining the programming syntax that structures natural language instructions logically. During the execution, we incorporate external memory to minimize redundancy. Furthermore, we equip the designed interpreter with the capability to invoke external tools, compensating for the limitations of LLM in specialized domains or when accessing real-time information. This work is open-source at [https://github.com/agiresearch/CoRE](https://github.com/agiresearch/CoRE), [https://github.com/agiresearch/OpenAGI](https://github.com/agiresearch/OpenAGI), and [https://github.com/agiresearch/AIOS](https://github.com/agiresearch/AIOS).

## 1 Introduction

Figure 1: In our CoRE system, we design the CoRE language to unify natural language programming, pseudo-code programming, and flow programming in the same syntax representative. We use the program for OpenAGI [[14](https://arxiv.org/html/2405.06907v2#bib.bib14)] platform as an example.

Programming is crucial for computers as it enables them to execute specific tasks based on a predefined set of instructions. It allows us to utilize logical algorithms to enable computers to solve problems. Programming has evolved significantly since its inception, with new technologies and innovations driving its growth. Initially, programming languages were based on binary machine language, such as punched cards, which can be directly executed by the machine. However, machine language was hardly readable to humans. Subsequently, low-level programming languages, such as assembly language, use mnemonic instructions and operands to represent machine code, which enhances the readability [[18](https://arxiv.org/html/2405.06907v2#bib.bib18)]. However, due to the requirement of controlling memory locations and registers, assembly language still has a high entry barrier for programmers. With the design of high-level programming languages like C/C++, Java and Python, coding has become more user-friendly and efficient. They offer programmers a more productive and accessible approach, leading to increased participation in programming and software development. Consequently, programming languages are becoming more integrated into everyday life.

From the history of programming languages, we can observe a clear trend toward increased usability, readability, and democracy of programming. Following this trend, natural language can be a desirable choice for coding due to its accessibility, readability, and minimal training requirements for programmers. However, the application of natural language programming presents challenges due to the inherent vagueness, ambiguity, and verbosity of natural language. The recently emerged Large Language Models (LLMs) serve as a solution to this challenge due to their extraordinary capability in language understanding [[38](https://arxiv.org/html/2405.06907v2#bib.bib38), [8](https://arxiv.org/html/2405.06907v2#bib.bib8)], tool use and function calling [[14](https://arxiv.org/html/2405.06907v2#bib.bib14), [42](https://arxiv.org/html/2405.06907v2#bib.bib42)], as well as interacting with human or environments [[43](https://arxiv.org/html/2405.06907v2#bib.bib43), [12](https://arxiv.org/html/2405.06907v2#bib.bib12)]. In this work, we propose a novel system for Code Representation and Execution (CoRE), which takes LLM as the interpreter to interpret and execute the instructions in natural language, enabling agent programming in natural language.

CoRE can be used for natural language programming, pseudo-code programming, and flow programming, as the three forms of agent programs unify into our CoRE language, as shown by the example in Figure [1](https://arxiv.org/html/2405.06907v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents"). In the realm of programming, the fundamental task involves designing and developing logically structured instructions to address specific problems. Natural language programming offers a method where instructions are formulated in everyday language, making the code intuitive and accessible. When we structure all natural language instructions in a logical way, it inherently mirrors the essence of pseudo-code programming. Pseudo-code, by design, simplifies the coding process by stripping down syntax complexities and focusing on the algorithmic logic for easy understanding. Therefore, when the instructions are expressed in natural language, the structured instructions can be identified as pseudo-code. Moreover, pseudo-code shares a direct relationship with flow programming, as it essentially represents the algorithm’s logic that can seamlessly be visualized as a workflow. Workflow, in turn, provides a graphical representation of the step-by-step execution of programs, emphasizing the decision-making visualization process and the flow of control across the program.

We face several significant challenges when designing the novel system for natural language programming with LLM as an interpreter. First, how to represent the logic of the program using natural language instructions. To tackle this issue, we design a set of programming syntax to logically structure natural language instructions, and unify the natural language programming, pseudo-code programming, and flow programming in the same representation. Second, given that the programs consist of step-by-step instructions, it is crucial to make sure that each step is executed according to its corresponding instruction. To ensure precise execution of the instructions in natural language for each step, we design two additional components: one for retrieving information from memory, and the other for invoking external tools. Considering the LLM’s limitation on the number of input tokens (context window size), including all runtime information in the input prompt is impractical. To address this problem, we store a large volume of intermediate results in temporary memory, retrieving relevant information as needed in subsequent steps [[48](https://arxiv.org/html/2405.06907v2#bib.bib48), [34](https://arxiv.org/html/2405.06907v2#bib.bib34), [27](https://arxiv.org/html/2405.06907v2#bib.bib27), [4](https://arxiv.org/html/2405.06907v2#bib.bib4)]. Besides, while LLMs excel at processing textual information, they often fall short in tasks that require domain-specific knowledge or up-to-date information [[15](https://arxiv.org/html/2405.06907v2#bib.bib15)]. To mitigate these limitations, we enable the LLM to utilize external tools to solve the problems [[14](https://arxiv.org/html/2405.06907v2#bib.bib14), [42](https://arxiv.org/html/2405.06907v2#bib.bib42), [30](https://arxiv.org/html/2405.06907v2#bib.bib30), [2](https://arxiv.org/html/2405.06907v2#bib.bib2)]. Finally, when executing the natural language program, incorrectly determining the next step can lead to different final results. We solve this problem by demanding the LLM interpreter to evaluate the current results so as to identify the most suitable subsequent step. The overall execution pipeline of CoRE is depicted in Figure [2](https://arxiv.org/html/2405.06907v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents").

Figure 2: An example showing how the CoRE system executes one step.

In summary, the key contributions of the work are listed as follows:

*   •

    We design a CoRE language that unifies natural language programming, pseudo-code programming and flow programming. The CoRE language logically structures natural language instructions.

*   •

    We propose the CoRE system, which utilizes Large Language Model (LLM) as an interpreter to interpret and execute instructions step-by-step. During execution, the LLM follows the instructions and leverages both information retrieval and external tools to enhance its effectiveness.

*   •

    We verify the effectiveness and efficiency of our system based on public benchmark datasets. Specifically, we employ our proposed system for agent task solving based on natural language programs, showcasing its practical capabilities.

In the following part of this paper, we first provide the related work in Section [2](https://arxiv.org/html/2405.06907v2#S2 "2 Related Work ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents"). In Section [3](https://arxiv.org/html/2405.06907v2#S3 "3 The CoRE System ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents") we present the CoRE framework and how the framework can be applied to LLM agents. We provide the experimental results in Section [4](https://arxiv.org/html/2405.06907v2#S4 "4 Experiments ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents"), and conclude the work together with future directions in Section [5](https://arxiv.org/html/2405.06907v2#S5 "5 Conclusions and Future Work ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents").

## 2 Related Work

### 2.1 Natural Language Programming

Research in natural language programming [[17](https://arxiv.org/html/2405.06907v2#bib.bib17), [5](https://arxiv.org/html/2405.06907v2#bib.bib5), [46](https://arxiv.org/html/2405.06907v2#bib.bib46), [28](https://arxiv.org/html/2405.06907v2#bib.bib28), [10](https://arxiv.org/html/2405.06907v2#bib.bib10), [13](https://arxiv.org/html/2405.06907v2#bib.bib13)] primarily focus on addressing the ambiguity in translating natural language into programming language statements. [Heidorn](https://arxiv.org/html/2405.06907v2#bib.bib17) [[17](https://arxiv.org/html/2405.06907v2#bib.bib17)] proposes to adopt heuristic NLP encoding and decoding rules to develop an automatic programming system that can accept natural language dialogues. [Vadas and Curran](https://arxiv.org/html/2405.06907v2#bib.bib46) [[46](https://arxiv.org/html/2405.06907v2#bib.bib46)] introduce a prototype system that can translate certain English instructions into executable Python code using Combinatory Categorial Grammar (CCG) parser, which uses unrestricted syntax to cover a wide range of user instruction semantics. [Mihalcea et al.](https://arxiv.org/html/2405.06907v2#bib.bib35) [[35](https://arxiv.org/html/2405.06907v2#bib.bib35)] implement a procedural natural language programming system to convert natural language to programming language. Early natural language programming techniques are restricted in extensibility by the need to create domain-specific languages (DSLs). To avoid the problems of repeatedly designing new DSLs, [Desai et al.](https://arxiv.org/html/2405.06907v2#bib.bib10) [[10](https://arxiv.org/html/2405.06907v2#bib.bib10)] propose a general generative framework for constructing a program that takes natural language input and produces the expressions in the target DSL. Further, [Ernst](https://arxiv.org/html/2405.06907v2#bib.bib13) [[13](https://arxiv.org/html/2405.06907v2#bib.bib13)] leverages neural networks, i.e., the recurrent neural networks (RNN), to convert English specifications of file system operations into corresponding bash commands.

### 2.2 Large Language Models and AI Agents for Problem Solving

Large Language Models (LLMs) have emerged as powerful tools for problem solving, encompassing tasks in reasoning, planning, and code generation. LLM reasoning typically involves decomposing a complex task into a sequence of steps, also known as a reasoning chain [[49](https://arxiv.org/html/2405.06907v2#bib.bib49)]. Prominent approaches in LLM reasoning include Chain-of-Thought (CoT) and its derivatives [[49](https://arxiv.org/html/2405.06907v2#bib.bib49), [26](https://arxiv.org/html/2405.06907v2#bib.bib26)]. To further improve the reasoning ability of LLM, several work has been proposed. The Self-consistency method [[47](https://arxiv.org/html/2405.06907v2#bib.bib47)] samples multiple reasoning paths and selects the most consistent outcome by voting. Additionally, classical data structures like trees and graphs are utilized to enhance reasoning efficiency and accuracy in fewer steps [[52](https://arxiv.org/html/2405.06907v2#bib.bib52), [3](https://arxiv.org/html/2405.06907v2#bib.bib3)]. Apart from reasoning, planning is also an important task that can be used to solve problems. LLM Planning involves generating a series of actions to achieve the predefined goals [[16](https://arxiv.org/html/2405.06907v2#bib.bib16)]. Recent advancements include direct prompting of LLMs for planning tasks, showing promising results [[20](https://arxiv.org/html/2405.06907v2#bib.bib20), [45](https://arxiv.org/html/2405.06907v2#bib.bib45), [11](https://arxiv.org/html/2405.06907v2#bib.bib11)]. Finite state machines have been integrated into LLM to enhance the planning ability [[29](https://arxiv.org/html/2405.06907v2#bib.bib29), [51](https://arxiv.org/html/2405.06907v2#bib.bib51)]. ReAct [[53](https://arxiv.org/html/2405.06907v2#bib.bib53)] proposes to leverage external tools like search engine to enhance the LLM planning. Besides, considering the powerful ability of LLM in programming, recent work propose to generate programming code to solve problems [[32](https://arxiv.org/html/2405.06907v2#bib.bib32), [22](https://arxiv.org/html/2405.06907v2#bib.bib22), [31](https://arxiv.org/html/2405.06907v2#bib.bib31), [6](https://arxiv.org/html/2405.06907v2#bib.bib6), [23](https://arxiv.org/html/2405.06907v2#bib.bib23), [40](https://arxiv.org/html/2405.06907v2#bib.bib40), [36](https://arxiv.org/html/2405.06907v2#bib.bib36)]. Furthermore, the “self-reflection” mechanism [[33](https://arxiv.org/html/2405.06907v2#bib.bib33), [39](https://arxiv.org/html/2405.06907v2#bib.bib39), [44](https://arxiv.org/html/2405.06907v2#bib.bib44)] enables LLMs to critique their own outputs, significantly enhancing performance in tasks such as reasoning [[3](https://arxiv.org/html/2405.06907v2#bib.bib3)] and code generation [[7](https://arxiv.org/html/2405.06907v2#bib.bib7)]. In contrast to existing methods that directly use LLMs for generating solutions, the proposed CoRE system utilizes LLMs as interpreters, executing solutions designed by humans to address complex questions. This approach leverages human creativity in solution design, coupled with LLM’s ability, to enhance problem-solving capabilities in natural language programming contexts.

Figure 3: An overview of the CoRE LLM interpreter system.

## 3 The CoRE System

In this section, we will introduce how we define the natural language programming syntax and how to use LLM as an interpreter to interpret and execute natural language programs.

### 3.1 CoRE Language Syntax

To organize natural language instructions, we define the basic structural representation for each step, which consists of four components. An example can be found in Figure [1](https://arxiv.org/html/2405.06907v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents").

*   •

    Step Name: Each step in the program is uniquely identified by a step name. This identifier is analogous to function identifiers in traditional programming languages, which facilitates navigation and reference within the program structure, ensuring that each operation within the program can be distinctly addressed and accessed.

*   •

    Step Type: The step type categorizes the nature of the operation being performed in each step, analogous to control structures in conventional programming. We define three primary step types:

    *   –

        Process: Akin to a procedural statement in traditional programming, this step type executes a specific operation and transitions to the next specified step.

    *   –

        Decision: Corresponding to conditional statements (e.g., “if-else”), this step involves branching the program flow based on evaluated conditions, leading to multiple potential paths.

    *   –

        Terminal: Similar to the “end” or “return” statement, this step marks the conclusion of the program, indicating that no further steps are to be executed.

*   •

    Step Instruction: The step instruction explicates the task to be conducted at a step. This component is integral as it provides the instruction and content for execution, paralleling the statement block in traditional programming languages. By demonstrating operations in natural language, NLPg lowers the barrier to programming, making it more readable for non-expert programmers.

*   •

    Step Connection: Step connections define the progression from one step to another, establishing the flow of the program execution. In process steps, a single subsequent step is specified. In decision steps, multiple pathways are delineated based on conditions. Terminal steps, by definition, do not lead to any future steps, indicating the end of program execution.

For each step in the program, the above four components are separated by “:::” (as illustrated in the CoRE language in Figure [1](https://arxiv.org/html/2405.06907v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")). Other special tokens can also be used to separate different components.

Figure 4: An example showing how the CoRE system retrieves relevant information.

In programming languages, there are three basic control constructs in programming [[9](https://arxiv.org/html/2405.06907v2#bib.bib9), [41](https://arxiv.org/html/2405.06907v2#bib.bib41)]: sequence, selection and iteration. These three basic constructs can be easily designed within the CoRE language.

*   •

    Sequence: Sequence in programming is the execution of statements in a linear order, with each statement leading to the next. In the CoRE framework, this construct is designed by setting the “Step Connection” to point to the subsequent step. Each step operates under the Process type until the sequence concludes.

*   •

    Selection: Selection in programming languages facilitates conditional branching, allowing the program to execute different sequences of steps based on specific conditions. This is implemented using the Decision step type where the “Step Connection” part explicitly outlines multiple potential paths. Each branch is defined by a condition stated within the “Step Connection” part, guiding the program flow to various steps depending on the conditions.

*   •

    Iteration: Iteration involves repeating a set of operations until a certain condition is met, akin to loops in conventional programming. In the CoRE framework, we utilize a step with the Decision type to assess whether the loop condition has been fulfilled. At the end of one loop cycle, the “Step Connection” is configured to point back to the previous Decision step, thereby enabling the continuation of the loop.

### 3.2 LLM as Interpreter

In this section, we will discuss how the CoRE system utilizes a Large Language Model (LLM) as an interpreter to execute programs written in the CoRE language. We will demonstrate the execution of a single step within the CoRE system, which is illustrated in Figure [3](https://arxiv.org/html/2405.06907v2#S2.F3 "Figure 3 ‣ 2.2 Large Language Models and AI Agents for Problem Solving ‣ 2 Related Work ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents"). More specifically, the system executes a single step in four procedures. First of all, the interpreter determines the useful information to execute the current step. Then the interpreter will integrate all relevant information to construct the prompt. Based on the generated prompt, the interpreter will generate response and may utilize tools to execute the current step. Finally, after executing the current step, the interpreter will determine the next step based on step type and execution results. We will introduce the four parts in details.

#### 3.2.1 Observation Retrieval from Memory

This initial procedure is critical since it sets the stage for the entire execution process of the current step. Figure [4](https://arxiv.org/html/2405.06907v2#S3.F4 "Figure 4 ‣ 3.1 CoRE Language Syntax ‣ 3 The CoRE System ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents") shows an example. The system’s memory serves as a repository of all prior observations related to the program, where the observation represents the results of tool execution, such as search results. During this phase, the interpreter scans the memory to identify records that are relevant to the current instruction. This selective retrieval ensures that the interpreter’s decisions are informed by accurate and contextually relevant data, which is crucial for the successful execution of the program.

Figure 5: An example showing how the CoRE system analyze the output from the LLM interpreter.

#### 3.2.2 Input Prompt Construction

Constructing the prompt is essentially about synthesizing the information into a comprehensive and coherent query that the LLM can understand and respond to effectively. This involves combining multiple information into a single, structured prompt that guides the LLM towards generating the most appropriate and contextually relevant response. In the CoRE system, the interpreter constructs a detailed prompt with four elements:

*   •

    Task Description: The query that defines the entire program, acting as the primary input to guide the system’s operations.

*   •

    Current Progress: Summarizes the previous steps including what has been done or decided, helping maintain a narrative flow.

*   •

    Observation: This part may not be included in every step. When relevant information is retrieved from the memory by the interpreter, it is incorporated here.

*   •

    Current Instruction: Specifies the action to be taken in natural language, directing the interpreter on how to proceed in the current step.

#### 3.2.3 Output Analysis

While the LLM can generate direct responses, complex tasks may require capabilities beyond its immediate scope. Incorporating the use of specialized tools when necessary extends the LLM’s capabilities, allowing the system to handle a broader range of tasks effectively. A demonstrative example of the execution process is shown in Figure [5](https://arxiv.org/html/2405.06907v2#S3.F5 "Figure 5 ‣ 3.2.1 Observation Retrieval from Memory ‣ 3.2 LLM as Interpreter ‣ 3 The CoRE System ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents"). In the CoRE system, the interpreter will make a decision about if or not to employ specialized tools based on the LLM’s initial response and the demands of the task at hand, which ensures that the system remains highly functional and versatile, actively solving problems rather than merely processing the language prompt for the current step. Specifically, if tool usage is warranted, the system will select the suitable tool, configure it with the necessary parameters, execute it, and integrate the output into the ongoing process.

#### 3.2.4 Branching Analysis

Figure 6: An example showing how the CoRE system determines the next step in the flow.

Determining the appropriate next step in the program is critical, especially in multi-branch scenarios where different outcomes can lead to different subsequent actions. Figure [6](https://arxiv.org/html/2405.06907v2#S3.F6 "Figure 6 ‣ 3.2.4 Branching Analysis ‣ 3.2 LLM as Interpreter ‣ 3 The CoRE System ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents") shows an example. In the CoRE language interpreter, the Decision steps indicate multiple branches with the corresponding conditions. The interpreter uses LLM to decide if the prompt satisfies the natural language described branching condition or not and which next step to take. This adaptive approach allows the system to navigate through decision points effectively, ensuring logical progression toward the program’s goals.

## 4 Experiments

### 4.1 Backbone Large Language Model (LLM)

We conduct experiments on both closed-source and open-source LLMs:

*   •

    GPT-4 [[37](https://arxiv.org/html/2405.06907v2#bib.bib37)] (Closed-source) is a generative pre-trained transformer of OpenAI. In this work, we use the GPT-4-1106-preview version.

*   •

    Mixtral-8x7B [[21](https://arxiv.org/html/2405.06907v2#bib.bib21)] (Open-source) is a pre-trained generative Sparse Mixture of Experts with 46.7 billion parameters.

### 4.2 Planning Schema of LLMs

We adopt the following LLM-based agent planning schema:

*   •

    Zero-shot Learning (Zero) directly inputs the query to the LLM.

*   •

    Chain-of-Thought (CoT) [[49](https://arxiv.org/html/2405.06907v2#bib.bib49)] induces the LLM to generate a coherent language sequence that serves as a meaningful intermediate step bridging the input query and the output answer.

*   •

    Few-shot Learning (Few) presents a set of high-quality demonstrations in the prompt, each consisting of both input and desired output on the target task.

*   •

    CoRE is our natural language programming method with LLM as an interpreter.

### 4.3 Benchmark Datasets

We conduct experiments on a benchmark dataset, OpenAGI [[14](https://arxiv.org/html/2405.06907v2#bib.bib14)]. The OpenAGI benchmark tasks are categorized based on their output type and ground-truth label type (Task 1, 2, and 3). Then, based on different task types, different metrics are employed to gauge the performance: CLIP Score [[19](https://arxiv.org/html/2405.06907v2#bib.bib19)], assessing the similarity between text and image, is utilized for Text-to-Image tasks; BERT Score [[55](https://arxiv.org/html/2405.06907v2#bib.bib55)], evaluating text generation with BERT, is applied when both data labels and the expected outputs are texts; and ViT Score [[50](https://arxiv.org/html/2405.06907v2#bib.bib50)] gauges the similarity between the image label and image output.

### 4.4 Implementation Details

Our framework and all baselines are implemented by PyTorch, an open-source library. We follow the implementation setting of the OpenAGI platform [[14](https://arxiv.org/html/2405.06907v2#bib.bib14)] for Zero-shot and few-shot learnings. We leverage the DSPy framework [[24](https://arxiv.org/html/2405.06907v2#bib.bib24), [25](https://arxiv.org/html/2405.06907v2#bib.bib25)] to apply the CoT strategy to the OpenAGI platform. We also tried Program-of-Thought [[6](https://arxiv.org/html/2405.06907v2#bib.bib6)] and ReAct [[54](https://arxiv.org/html/2405.06907v2#bib.bib54)] strategies on the OpenAGI platform. However, the ReAct strategy requires text observation, which is unsuitable for our OpenAGI task since some observations are in image format, and Program-of-Thought cannot generate executable codes. Thus, we did not include them as the baselines.

### 4.5 Experimental Analysis

The experiment results on the OpenAGI benchmark are shown in Table [1](https://arxiv.org/html/2405.06907v2#S4.T1 "Table 1 ‣ 4.5 Experimental Analysis ‣ 4 Experiments ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents"). Each row stands for a type of task, each column represents the planning schema of an LLM interpreter, and every four columns are the results of the same LLM interpreter. From the results, we can see that our CoRE planning schema is better on average performance than any baseline under both Mixtral and GPT-4 as the interpreters. When using Mixtral as the interpreter, CoRE outperforms Zero-shot and CoT under each type of task, and is better than Few-shot learning on Task 2 and average score, though worse on Task 3 and slightly worse on Task 1\. When using GPT-4 as the interpreter, CoT, Few-shot has similar performance on Task 1 and Task 3, while on Task 2 and average score, CoRE is still the best. It may be worth noting that it is unfair to compare CoRE with Few-shot learning since we do not directly provide the output format and output example in the prompt. However, even without using such examples, the CoRE planning strategy is still better than the Few-shot strategy on average. We also find that even for the same CoRE program, the system may perform differently when using different LLM as interpreters, which means that the performance of natural language programming depends on the natural language understanding ability of the LLM interpreter.

| Metrics / Task | Mixtral (open source) as LLM interpreter | GPT-4 (closed-source) as LLM interpreter |
| Zero | CoT | Few | CoRE (Ours) | Zero | CoT | Few | CoRE (Ours) |
| Task 1 (CLIP Score) | 0.0 | 0.0 | $0.1839$ | 0.1825 | 0.0 | 0.2732 | 0.1837 | $0.3030$ |
| Task 2 (BERT Score) | 0.1092 | 0.1987 | 0.0687 | $0.2593$ | 0.2076 | 0.2266 | 0.5277 | $0.5756$ |
| Task 3 (ViT Score) | 0.1949 | 0.1562 | $0.5501$ | 0.2437 | 0.5058 | 0.6736 | $0.6916$ | 0.6611 |
| Average over tasks | 0.1206 | 0.1736 | 0.1887 | $0.2483$ | 0.2378 | 0.3359 | 0.5391 | $0.5744$ |
| % of Valid Plans | 23.08 | 38.46 | 46.15 | $56.92$ | 53.85 | 60.00 | 83.08 | $92.31$ |

Table 1: OpenAGI [[14](https://arxiv.org/html/2405.06907v2#bib.bib14)] benchmark task performances under different settings. Zero is for Zero-shot Learning, Few is for Few-shot Learning. The boldface numbers denote the highest score under each task type using the same LLM.

## 5 Conclusions and Future Work

In this study, we introduce a novel system, CoRE, for Code Representation and Execution. CoRE is designed to bridge natural language programming, pseudo-code, and flow programming through the development of a unified CoRE language for the construction of AI Agents. CoRE leverages natural language as the programming interface, which lowers the programming barrier and advocates the democracy of programming, so that even ordinary users can create their AI Agents. Our system leverages Large Language Models (LLMs) as interpreters to process and execute natural language instructions. Throughout execution, the interpreter dynamically retrieves necessary information, utilizes appropriate external tools, and navigates through instructions based on previous outputs. The experimental outcomes validate the efficacy of the CoRE system in natural language programming.

While CoRE demonstrates promising results, it currently relies on manually crafted programs, which may introduce inefficiencies due to the inherent ambiguities of natural language. To address this, future research could explore the development of automated systems for generating natural language programming instructions. This automation would help standardize instruction clarity and precision, potentially improving system performance. Additionally, a future direction is to expand CoRE’s language support to facilitate international use and implement real-time debugging features to aid in education and assist novice programmers, further broadening the system’s utility and accessibility.

## References

*   [1]
*   Ahn et al. [2022] Michael Ahn, Anthony Brohan, Noah Brown, Yevgen Chebotar, Omar Cortes, Byron David, Chelsea Finn, Chuyuan Fu, Keerthana Gopalakrishnan, Karol Hausman, et al. 2022. Do as i can, not as i say: Grounding language in robotic affordances. *arXiv preprint arXiv:2204.01691* (2022).
*   Besta et al. [2024] Maciej Besta, Nils Blach, Ales Kubicek, Robert Gerstenberger, Michal Podstawski, Lukas Gianinazzi, Joanna Gajda, Tomasz Lehmann, Hubert Niewiadomski, Piotr Nyczyk, et al. 2024. Graph of thoughts: Solving elaborate problems with large language models. In *Proceedings of the AAAI Conference on Artificial Intelligence*, Vol. 38\. 17682–17690.
*   Borgeaud et al. [2022] Sebastian Borgeaud, Arthur Mensch, Jordan Hoffmann, Trevor Cai, Eliza Rutherford, Katie Millican, George Bm Van Den Driessche, Jean-Baptiste Lespiau, Bogdan Damoc, Aidan Clark, et al. 2022. Improving language models by retrieving from trillions of tokens. In *International conference on machine learning*. PMLR, 2206–2240.
*   Bruckman and Edwards [1999] Amy Bruckman and Elizabeth Edwards. 1999. Should we leverage natural-language knowledge? An analysis of user errors in a natural-language-style programming language. In *Proceedings of the SIGCHI conference on Human Factors in Computing Systems*. 207–214.
*   Chen et al. [2023b] Wenhu Chen, Xueguang Ma, Xinyi Wang, and William W. Cohen. 2023b. Program of Thoughts Prompting: Disentangling Computation from Reasoning for Numerical Reasoning Tasks. *Transactions on Machine Learning Research* (2023).
*   Chen et al. [2023a] Xinyun Chen, Maxwell Lin, Nathanael Schärli, and Denny Zhou. 2023a. Teaching large language models to self-debug. *arXiv preprint arXiv:2304.05128* (2023).
*   Chung et al. [2024] Hyung Won Chung, Le Hou, Shayne Longpre, Barret Zoph, Yi Tay, William Fedus, Yunxuan Li, Xuezhi Wang, Mostafa Dehghani, Siddhartha Brahma, et al. 2024. Scaling instruction-finetuned language models. *Journal of Machine Learning Research* 25, 70 (2024), 1–53.
*   Dahl et al. [1972] Ole-Johan Dahl, Edsger Wybe Dijkstra, and Charles Antony Richard Hoare. 1972. *Structured programming*. Academic Press Ltd.
*   Desai et al. [2016] Aditya Desai, Sumit Gulwani, Vineet Hingorani, Nidhi Jain, Amey Karkare, Mark Marron, and Subhajit Roy. 2016. Program synthesis using natural language. In *Proceedings of the 38th International Conference on Software Engineering*. 345–356.
*   Ding et al. [2023] Yan Ding, Xiaohan Zhang, Chris Paxton, and Shiqi Zhang. 2023. Task and motion planning with large language models for object rearrangement. In *2023 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)*. IEEE, 2086–2092.
*   Driess et al. [2023] Danny Driess, Fei Xia, Mehdi SM Sajjadi, Corey Lynch, Aakanksha Chowdhery, Brian Ichter, Ayzaan Wahid, Jonathan Tompson, Quan Vuong, Tianhe Yu, et al. 2023. Palm-e: An embodied multimodal language model. *arXiv preprint arXiv:2303.03378* (2023).
*   Ernst [2017] Michael D Ernst. 2017. Natural language is a programming language: Applying natural language processing to software development. In *2nd Summit on Advances in Programming Languages (SNAPL 2017)*. Schloss-Dagstuhl-Leibniz Zentrum für Informatik.
*   Ge et al. [2023a] Yingqiang Ge, Wenyue Hua, Kai Mei, Jianchao Ji, Juntao Tan, Shuyuan Xu, Zelong Li, and Yongfeng Zhang. 2023a. OpenAGI: When LLM Meets Domain Experts. *In Advances in Neural Information Processing Systems (NeurIPS)* (2023).
*   Ge et al. [2023b] Yingqiang Ge, Yujie Ren, Wenyue Hua, Shuyuan Xu, Juntao Tan, and Yongfeng Zhang. 2023b. LLM as OS, Agents as Apps: Envisioning AIOS, Agents and the AIOS-Agent Ecosystem. *arXiv e-prints* (2023), arXiv–2312.
*   Hao et al. [2023] Shibo Hao, Yi Gu, Haodi Ma, Joshua Jiahua Hong, Zhen Wang, Daisy Zhe Wang, and Zhiting Hu. 2023. Reasoning with language model is planning with world model. *arXiv preprint arXiv:2305.14992* (2023).
*   Heidorn [1976] George E Heidorn. 1976. Automatic programming through natural language dialogue: A survey. *IBM Journal of research and development* 20, 4 (1976), 302–313.
*   Hennessy and Patterson [2011] John L Hennessy and David A Patterson. 2011. *Computer architecture: a quantitative approach*. Elsevier.
*   Hessel et al. [2021] Jack Hessel, Ari Holtzman, Maxwell Forbes, Ronan Le Bras, and Yejin Choi. 2021. CLIPScore: A Reference-free Evaluation Metric for Image Captioning.
*   Huang et al. [2022] Wenlong Huang, Fei Xia, Ted Xiao, Harris Chan, Jacky Liang, Pete Florence, Andy Zeng, Jonathan Tompson, Igor Mordatch, Yevgen Chebotar, et al. 2022. Inner monologue: Embodied reasoning through planning with language models. *arXiv preprint arXiv:2207.05608* (2022).
*   Jiang et al. [2024] Albert Q Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, et al. 2024. Mixtral of experts. *arXiv preprint arXiv:2401.04088* (2024).
*   Jojic et al. [2023] Ana Jojic, Zhen Wang, and Nebojsa Jojic. 2023. Gpt is becoming a turing machine: Here are some ways to program it. *arXiv preprint arXiv:2303.14310* (2023).
*   Josifoski et al. [2023] Martin Josifoski, Lars Klein, Maxime Peyrard, Yifei Li, Saibo Geng, Julian Paul Schnitzler, Yuxing Yao, Jiheng Wei, Debjit Paul, and Robert West. 2023. Flows: Building blocks of reasoning and collaborating ai. *arXiv preprint arXiv:2308.01285* (2023).
*   Khattab et al. [2022] Omar Khattab, Keshav Santhanam, Xiang Lisa Li, David Hall, Percy Liang, Christopher Potts, and Matei Zaharia. 2022. Demonstrate-Search-Predict: Composing Retrieval and Language Models for Knowledge-Intensive NLP. *arXiv preprint arXiv:2212.14024* (2022).
*   Khattab et al. [2023] Omar Khattab, Arnav Singhvi, Paridhi Maheshwari, Zhiyuan Zhang, Keshav Santhanam, Sri Vardhamanan, Saiful Haq, Ashutosh Sharma, Thomas T. Joshi, Hanna Moazam, Heather Miller, Matei Zaharia, and Christopher Potts. 2023. DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines. *arXiv preprint arXiv:2310.03714* (2023).
*   Kojima et al. [2022] Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. 2022. Large language models are zero-shot reasoners. *Advances in neural information processing systems* 35 (2022), 22199–22213.
*   Lewis et al. [2020] Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, et al. 2020. Retrieval-augmented generation for knowledge-intensive nlp tasks. *Advances in Neural Information Processing Systems* 33 (2020), 9459–9474.
*   Li and Hovy [2015] Jiwei Li and Eduard Hovy. 2015. The NLP engine: A universal turing machine for nlp. *arXiv preprint arXiv:1503.00168* (2015).
*   Li et al. [2024] Zelong Li, Wenyue Hua, Hao Wang, He Zhu, and Yongfeng Zhang. 2024. Formal-LLM: Integrating Formal Language and Natural Language for Controllable LLM-based Agents. *arXiv:2402.00798* (2024).
*   Liang et al. [2023] Yaobo Liang, Chenfei Wu, Ting Song, Wenshan Wu, Yan Xia, Yu Liu, Yang Ou, Shuai Lu, Lei Ji, Shaoguang Mao, et al. 2023. Taskmatrix. ai: Completing tasks by connecting foundation models with millions of apis. *arXiv preprint arXiv:2303.16434* (2023).
*   Liu et al. [2023] Bo Liu, Yuqian Jiang, Xiaohan Zhang, Qiang Liu, Shiqi Zhang, Joydeep Biswas, and Peter Stone. 2023. Llm+ p: Empowering large language models with optimal planning proficiency. *arXiv preprint arXiv:2304.11477* (2023).
*   Lyu et al. [2023] Qing Lyu, Shreya Havaldar, Adam Stein, Li Zhang, Delip Rao, Eric Wong, Marianna Apidianaki, and Chris Callison-Burch. 2023. Faithful chain-of-thought reasoning. *arXiv preprint arXiv:2301.13379* (2023).
*   Madaan et al. [2024] Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang, et al. 2024. Self-refine: Iterative refinement with self-feedback. *Advances in Neural Information Processing Systems* 36 (2024).
*   Mei et al. [2024] Kai Mei, Zelong Li, Shuyuan Xu, Ruosong Ye, Yingqiang Ge, and Yongfeng Zhang. 2024. AIOS: LLM Agent Operating System. *arXiv* (2024).
*   Mihalcea et al. [2006] Rada Mihalcea, Hugo Liu, and Henry Lieberman. 2006. NLP (natural language processing) for NLP (natural language programming). In *Computational Linguistics and Intelligent Text Processing: 7th International Conference, CICLing 2006, Mexico City, Mexico, February 19-25, 2006\. Proceedings 7*. Springer, 319–330.
*   Nijkamp et al. [2022] Erik Nijkamp, Bo Pang, Hiroaki Hayashi, Lifu Tu, Huan Wang, Yingbo Zhou, Silvio Savarese, and Caiming Xiong. 2022. Codegen: An open large language model for code with multi-turn program synthesis. *arXiv preprint arXiv:2203.13474* (2022).
*   OpenAI [2023] Josh et al OpenAI. 2023. GPT-4 Technical Report. arXiv:2303.08774 [cs.CL]
*   Ouyang et al. [2022] Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, et al. 2022. Training language models to follow instructions with human feedback. *Advances in neural information processing systems* 35 (2022), 27730–27744.
*   Paul et al. [2023] Debjit Paul, Mete Ismayilzada, Maxime Peyrard, Beatriz Borges, Antoine Bosselut, Robert West, and Boi Faltings. 2023. Refiner: Reasoning feedback on intermediate representations. *arXiv preprint arXiv:2304.01904* (2023).
*   Poesia et al. [2022] Gabriel Poesia, Oleksandr Polozov, Vu Le, Ashish Tiwari, Gustavo Soares, Christopher Meek, and Sumit Gulwani. 2022. Synchromesh: Reliable code generation from pre-trained language models. *arXiv preprint arXiv:2201.11227* (2022).
*   Prather [1997] Ronald E Prather. 1997. Regular expressions for program computations. *The American mathematical monthly* 104, 2 (1997), 120–130.
*   Qin et al. [2023] Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. 2023. Toolllm: Facilitating large language models to master 16000+ real-world apis. *arXiv preprint arXiv:2307.16789* (2023).
*   Ross et al. [2023] Steven I Ross, Fernando Martinez, Stephanie Houde, Michael Muller, and Justin D Weisz. 2023. The programmer’s assistant: Conversational interaction with a large language model for software development. In *Proceedings of the 28th International Conference on Intelligent User Interfaces*. 491–514.
*   Shinn et al. [2023] Noah Shinn, Beck Labash, and Ashwin Gopinath. 2023. Reflexion: an autonomous agent with dynamic memory and self-reflection. *arXiv preprint arXiv:2303.11366* (2023).
*   Singh et al. [2023] Ishika Singh, Valts Blukis, Arsalan Mousavian, Ankit Goyal, Danfei Xu, Jonathan Tremblay, Dieter Fox, Jesse Thomason, and Animesh Garg. 2023. Progprompt: Generating situated robot task plans using large language models. In *2023 IEEE International Conference on Robotics and Automation (ICRA)*. IEEE, 11523–11530.
*   Vadas and Curran [2005] David Vadas and James R Curran. 2005. Programming with unrestricted natural language. In *Proceedings of the Australasian Language Technology Workshop 2005*. 191–199.
*   Wang et al. [2022] Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, Sharan Narang, Aakanksha Chowdhery, and Denny Zhou. 2022. Self-consistency improves chain of thought reasoning in language models. *arXiv preprint arXiv:2203.11171* (2022).
*   Wang et al. [2023] Yubo Wang, Xueguang Ma, and Wenhu Chen. 2023. Augmenting black-box llms with medical textbooks for clinical question answering. *arXiv preprint arXiv:2309.02233* (2023).
*   Wei et al. [2022] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting elicits reasoning in large language models. *Advances in neural information processing systems* 35 (2022), 24824–24837.
*   Wu et al. [2020] Bichen Wu, Chenfeng Xu, Xiaoliang Dai, Alvin Wan, Peizhao Zhang, Zhicheng Yan, Masayoshi Tomizuka, Joseph Gonzalez, Kurt Keutzer, and Peter Vajda. 2020. Visual Transformers: Token-based Image Representation and Processing for Computer Vision. arXiv:2006.03677 [cs.CV]
*   Wu et al. [2024] Yiran Wu, Tianwei Yue, Shaokun Zhang, Chi Wang, and Qingyun Wu. 2024. StateFlow: Enhancing LLM Task-Solving through State-Driven Workflows. *arXiv preprint arXiv:2403.11322* (2024).
*   Yao et al. [2024] Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, and Karthik Narasimhan. 2024. Tree of thoughts: Deliberate problem solving with large language models. *Advances in Neural Information Processing Systems* 36 (2024).
*   Yao et al. [2022] Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2022. React: Synergizing reasoning and acting in language models. *arXiv preprint arXiv:2210.03629* (2022).
*   Yao et al. [2023] Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2023. ReAct: Synergizing Reasoning and Acting in Language Models. In *International Conference on Learning Representations (ICLR)*.
*   Zhang et al. [2020] Tianyi Zhang, Varsha Kishore, Felix Wu, Kilian Q. Weinberger, and Yoav Artzi. 2020. BERTScore: Evaluating Text Generation with BERT.