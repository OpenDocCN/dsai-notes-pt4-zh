<!--yml
category: 未分类
date: 2025-01-11 12:12:28
-->

# RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance

> 来源：[https://arxiv.org/html/2410.01242/](https://arxiv.org/html/2410.01242/)

1^(st) Haolin Jin University of Sydney
hjin3177@uni.sydney.edu.au    2^(nd) Zechao Sun University of Sydney
zsun6058@uni.sydney.edu.au    1^* Huaming Chen University of Sydney
huaming.chen@sydney.edu.au

###### Abstract

Large Language Models (LLMs) have shown incredible potential in code generation tasks, and recent research in prompt engineering have enhanced LLMs’ understanding of textual information. However, ensuring the accuracy of generated code often requires extensive testing and validation by programmers. While LLMs can typically generate code based on task descriptions, their accuracy remains limited, especially for complex tasks that require a deeper understanding of both the problem statement and the code generation process. This limitation is primarily due to the LLMs’ need to simultaneously comprehend text and generate syntactically and semantically correct code, without having the capability to automatically refine the code. In real-world software development, programmers rarely produce flawless code in a single attempt based on the task description alone, they rely on iterative feedback and debugging to refine their programs. Inspired by this process, we introduce a novel architecture of LLM-based agents for code generation and automatic debugging: Refinement and Guidance Debugging (RGD). The RGD framework is a multi-LLM-based agent debugger that leverages three distinct LLM agents—Guide Agent, Debug Agent, and Feedback Agent. RGD decomposes the code generation task into multiple steps, ensuring a clearer workflow and enabling iterative code refinement based on self-reflection and feedback. Experimental results demonstrate that RGD exhibits remarkable code generation capabilities, achieving state-of-the-art performance with a 9.8% improvement on the HumanEval dataset and a 16.2% improvement on the MBPP dataset compared to the state-of-the-art approaches and traditional direct prompting approaches. We highlight the effectiveness of the RGD framework in enhancing LLMs’ ability to generate and refine code autonomously.

###### Index Terms:

Large Language Models, Code Generation, Automatic Debugging, Multi-Agent System, Code Debugging

## I Introduction

![Refer to caption](img/08bf37fc7731464fdd0be17189f32d38.png)

Figure 1: The overview of the RGD framework contains three processes: the code generation and debugging process, the evaluation process, and the reflection process. The Guide LLM is responsible for both generating guides and retrieving relevant information from the memory pool. The Debug LLM uses this information to generate code and incorporate failure analysis from the Feedback LLM to fix the code. The generated code is tested against both visible and hidden test cases to ensure comprehensive coverage and accuracy. This process is iterated until the code passes all visible and hidden test cases or reaches the maximum number of iterations.

Large Language Models (LLMs) have made significant advancements in the domain of automated code generation, showcasing their capability to translate natural language into functional code, generate code explanations [[1](https://arxiv.org/html/2410.01242v2#bib.bib1)], and even perform code-to-code translations across different programming languages [[2](https://arxiv.org/html/2410.01242v2#bib.bib2)] [[3](https://arxiv.org/html/2410.01242v2#bib.bib3)]. The most prevalent methods for employing LLMs in code generation rely heavily on prompt engineering, where well-crafted prompts are designed to guide the LLM in generating code snippets or interpreting existing code [[4](https://arxiv.org/html/2410.01242v2#bib.bib4)] [[5](https://arxiv.org/html/2410.01242v2#bib.bib5)]. This approach has proven effective in a range of scenarios, from generating code based on textual descriptions to converting code between different languages and frameworks.

However, the approach of generating code from text in a single pass has its limitations. Code generation is inherently a complex task, and a one-time generation approach often fails to account for the numerous edge cases and detailed task requirements that arise in software development, especially given the complexity and precision needed in software development tasks [[6](https://arxiv.org/html/2410.01242v2#bib.bib6)]. Consequently, researchers have introduced multi-round code generation frameworks that involve iterative refinement. These frameworks iteratively generate programs through multiple interactions, significantly improving the quality of program synthesis and making the development process more efficient and accurate [[7](https://arxiv.org/html/2410.01242v2#bib.bib7)]. Researchers have explored ways for LLMs to autonomously learn from errors and perform debugging and repairs. These frameworks leverage reflection, including failed test cases and error messages, and learns from these outcomes to improve subsequent code generations [[8](https://arxiv.org/html/2410.01242v2#bib.bib8)]. Although these approaches have demonstrated significant improvements, they cannot guarantee that every reflection result will lead the LLM to make effective changes based on failed test cases. As a result, LLMs often end up generating the same code over multiple iterations.

Moreover, the performance of LLMs in code generation is highly dependent on the clarity and completeness of the task description provided in the prompts, there is a high likelihood of the LLM overlooking critical edge cases or missing essential requirements. Previous studies have attempted to address this issue by introducing the concept where the LLM first reasoning the task and then proceeds with code generation [[9](https://arxiv.org/html/2410.01242v2#bib.bib9)] [[6](https://arxiv.org/html/2410.01242v2#bib.bib6)]. However, this strategy presents its own challenges; the LLM tends to rely heavily on the initial plan, leading to a lack of flexibility as it fails to incorporate further refinements or adjustments that may be necessary to address evolving requirements or unforeseen issues in the code.

In this paper, we introduce a novel framework called RGD (Refinement and Guidance Debugging) that leverages multiple LLMs in a collaborative manner to improve the quality of code generation. RGD incorporates multiple specialized LLMs, each with distinct roles, to simulate a comprehensive code repair process. The RGD framework utilizes three LLMs: the Guide LLM, responsible for generating a generation guide based on the task description, then passed the generation guide and the task description to the Debug LLM for initial code generation. The third LLM is tasked with consolidating the execution results and conducting failure analysis. The feedback LLM will use generated code and failed test cases to analysis the reason of failure and potential fix procedures. In addition to having the LLM analyze the reasons for failed test cases, our architecture also considers the test cases that pass. This step ensures that while the LLM analyzes the failed test cases, it is also aware of how the code passes the correct test cases, thereby preventing future code generations from failing previously passing test cases. This situation is very common, if there are only failed test sets and executed code, LLM will focus on trying to make the code successfully match the expected output and may not be able to actually find the problem in the code to debug, so the changed code will fail all test sets, and the failure analysis is also to avoid this scenario.

Furthermore, we draw inspiration from the RAG (Retrieval-Augmented Generation) approach, where we match similar content from a memory pool as additional information [[10](https://arxiv.org/html/2410.01242v2#bib.bib10)]. Unlike traditional applications where the retrieved content is often code, we store relevant generation guides and task descriptions. This allows the Guide LLM to generate guidance that avoids zero-shot learning scenarios and instead relies on past successful examples to produce high-quality responses. Figure [1](https://arxiv.org/html/2410.01242v2#S1.F1 "Figure 1 ‣ I Introduction ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance") shows the overall flow and RGD architecture.

To evaluate the RGD framework, we conducted tests on various benchmarks, including HumanEval [[4](https://arxiv.org/html/2410.01242v2#bib.bib4)], HumanEval-ET [[11](https://arxiv.org/html/2410.01242v2#bib.bib11)], MBPP [[12](https://arxiv.org/html/2410.01242v2#bib.bib12)], MBPP-ET [[11](https://arxiv.org/html/2410.01242v2#bib.bib11)], and APPS [[13](https://arxiv.org/html/2410.01242v2#bib.bib13)]. These datasets are all text-to-code generation tasks and contain tasks and test sets of varying difficulty levels. HumanEval-ET and MBPP-ET are extended versions with additional edge case test cases to address the limitations of the original test sets. Through our experiments, we evaluated the performance of GPT-4o and GPT-4o-mini under the RGD framework across multiple datasets. The results demonstrated significant improvements of RGD on all benchmarks, surpassing the current state-of-the-art frameworks including LDB [[14](https://arxiv.org/html/2410.01242v2#bib.bib14)].

## II Related Work

Large Language Models for Code Generation Recently, large language models (LLMs) have demonstrated outstanding performance multiple tasks especially in the code generation, showing great potential across various benchmarks [[4](https://arxiv.org/html/2410.01242v2#bib.bib4)] [[15](https://arxiv.org/html/2410.01242v2#bib.bib15)] [[16](https://arxiv.org/html/2410.01242v2#bib.bib16)] [[17](https://arxiv.org/html/2410.01242v2#bib.bib17)]. However, when faced with complex problems, these models are prone to generating hallucinated responses [[18](https://arxiv.org/html/2410.01242v2#bib.bib18)]. Prompt engineering [[19](https://arxiv.org/html/2410.01242v2#bib.bib19)] has reduced the need for extensive fine-tuning of LLMs on specific downstream tasks, which can better understand user requirements through contextual information. Chain-of-thought prompting [[6](https://arxiv.org/html/2410.01242v2#bib.bib6)], which guides LLMs to provide step-by-step responses, enhances their reasoning abilities, leading to significant improvements over baselines on multiple benchmarks [[20](https://arxiv.org/html/2410.01242v2#bib.bib20)]. Tree-of-thought (ToT) [[21](https://arxiv.org/html/2410.01242v2#bib.bib21)] framework enhances the reasoning capabilities of large language models by structuring the reasoning process as a tree search. Each node represents a possible reasoning state, and edges denote transitions between states. In addition, researchers have explored enabling LLMs to debug the code they generate on their own [[22](https://arxiv.org/html/2410.01242v2#bib.bib22)], allowing the models to autonomously fix errors by understanding the causes from generated outputs [[23](https://arxiv.org/html/2410.01242v2#bib.bib23)] [[8](https://arxiv.org/html/2410.01242v2#bib.bib8)]. Beyond self-repair based on execution results, these frameworks can also leverage external tools for dynamic and automatic code correction [[24](https://arxiv.org/html/2410.01242v2#bib.bib24)] [[25](https://arxiv.org/html/2410.01242v2#bib.bib25)] [[26](https://arxiv.org/html/2410.01242v2#bib.bib26)].

Information Retrieval is a popular strategy recently, which involves retrieving helpful information from local storage or the cloud to assist in generation tasks [[27](https://arxiv.org/html/2410.01242v2#bib.bib27)]. It has gained significant traction in LLM-based agents. For example, RAG [[28](https://arxiv.org/html/2410.01242v2#bib.bib28)] [[29](https://arxiv.org/html/2410.01242v2#bib.bib29)] and Microsoft’s GraphRAG [[30](https://arxiv.org/html/2410.01242v2#bib.bib30)] transform the original zero-shot learning generation environment into a few-shot learning scenario. Additionally, the matching algorithms can be adjusted to better fit specific task requirements. By leveraging web queries and utilizing tools to collect data in real-time from the internet [[31](https://arxiv.org/html/2410.01242v2#bib.bib31)], the capability of LLMs is further enhanced. RAG has shown remarkable performance in downstream tasks like code generation [[32](https://arxiv.org/html/2410.01242v2#bib.bib32)]. By ranking and sort the retrieved information, it increases reliability and reduces the impact of hallucinations [[33](https://arxiv.org/html/2410.01242v2#bib.bib33)].

## III Refinment and Guidance Debugging

### III-A Definitions

Following recent works [[14](https://arxiv.org/html/2410.01242v2#bib.bib14)] [[8](https://arxiv.org/html/2410.01242v2#bib.bib8)], we divide the benchmarking dataset HumanEval and MBPP into three components: $(Q,T_{v},T_{h},E)$. Here, $Q$ represents the task description, which includes code snippets and requirements in natural language. $T_{v}$ stands for visible test cases, $T_{h}$ for hidden test cases, and $E$ is the entry point. All test cases are executed starting from the entry point.

![Refer to caption](img/0f72f3a06dcc3a04b550fdf181518211.png)

Figure 2: Guide Agent Prompt

### III-B Guide LLM Agent

The Guide Agent is assigned a special role with the primary task of conducting initial reasoning and thinking. It dynamically adjusts the input information based on the task’s complexity and the current stage of execution. The Guide LLM is also responsible for selecting and applying relevant guides extracted from the Memory Pool. However, to avoid unnecessary token usage and excessive context that may lead to LLM overhead, the memory pool matching and extraction are not performed during the initial generation. As shown in Figure [2](https://arxiv.org/html/2410.01242v2#S3.F2 "Figure 2 ‣ III-A Definitions ‣ III Refinment and Guidance Debugging ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance"), different prompts are used by the Guide LLM at the initial stage and the debug stage. In the initial stage, the Guide Agent generates a Generation Guide based on the task description $Q$ and entry point $E$ without adding any additional information. During the debug stage, samples are matched from the memory pool based on task similarity. The system’s matching mechanism includes the following components:

*   •

    $Q$: The task description.

*   •

    $G$: The generated guide by the Guide LLM.

*   •

    $K$: A set of matching keywords extracted from the task description $Q$ and generated code $C$.

*   •

    $E$: The execution results, which include both visible test cases $T_{v}$ and hidden test cases $T_{h}$.

The memory pool, denoted as $\mathcal{M}$, stores tuples in the form of $(Q_{i},G_{i},K_{i})$, where:

*   •

    $Q_{i}$ is the task description.

*   •

    $G_{i}$ is the corresponding generation guide.

*   •

    $K_{i}$ is the set of keywords extracted by the GPT-4o-mini API after the task successfully passes both $T_{v}$ and $T_{h}$.

When a task is completed successfully (it passes both visible and hidden test cases), we call the GPT-4o-mini API to generate the set of keywords $K_{i}$ based on the task description $Q$ and the generated code $C$. This process is formulated as:

|  | $K_{i}=\text{ExtractKeywords}(Q,C)$ |  | (1) |

where ExtractKeywords is the function to extract relevant keywords from the task description and its corresponding solution. When processing a new task $Q_{\text{new}}$ during the Debug phase, the memory pool is queried for similar tasks using an SBERT [[34](https://arxiv.org/html/2410.01242v2#bib.bib34)] [[35](https://arxiv.org/html/2410.01242v2#bib.bib35)] for description match and BM25 [[36](https://arxiv.org/html/2410.01242v2#bib.bib36)] for the index term match, finally achieve hybrid retrieval systems [[37](https://arxiv.org/html/2410.01242v2#bib.bib37)]. The overall process shown in Algorithm [1](https://arxiv.org/html/2410.01242v2#alg1 "Algorithm 1 ‣ III-B Guide LLM Agent ‣ III Refinment and Guidance Debugging ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance"), which is the pseudo-code illustrates the memory pool matching functions, where the function Sim computes the similarity between $Q_{\text{new}}$ and $Q_{i}$, as well as the keywords.

The Guide LLM generates a final guide $G_{\text{final}}$ by augmenting the initial guide $G_{\text{init}}$ generated from $Q_{\text{new}}$ with the relevant guide $G_{i}$ retrieved from the memory pool based on the similarity score:

|  | $G_{\text{final}}=\text{GuideLLM}({G_{\text{init}},G_{i},Q,A})$ |  | (2) |

where $A$ represents the failure analysis generated by the Feedback Agent which will be introduced in (Section. [III-C](https://arxiv.org/html/2410.01242v2#S3.SS3 "III-C Debug LLM Agent ‣ III Refinment and Guidance Debugging ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance")).

Algorithm 1 Memory Pool Extraction

1:Memory pool $\mathcal{M}\leftarrow\emptyset$, Guide LLM model, similarity threshold list $\mathcal{T}\leftarrow[]$2:function RetrieveFromMemoryPool($\mathcal{M},Q^{\prime}$)3:    Input: Memory pool $\mathcal{M}$, New task description $Q^{\prime}$4:    $\mathcal{S}\leftarrow[]$ (Initialize similar guides list)5:    for all $(Q,G,K)\in\mathcal{M}$ do6:       $\tau\leftarrow\text{Sim}(Q^{\prime},Q,K)$ (Compute similarity)7:       if $|\mathcal{S}|<3$ then8:          $\mathcal{S}\leftarrow\mathcal{S}\cup\{(Q,G)\}$9:          $\mathcal{T}\leftarrow\mathcal{T}\cup\{\tau\}$10:       else11:          $\tau_{\min}\leftarrow\min(\mathcal{T})$12:          if $\tau>\tau_{\min}$ then13:Remove element with $\tau_{\min}$ from $\mathcal{S}$ and $\mathcal{T}$14:             $\mathcal{S}\leftarrow\mathcal{S}\cup\{(Q,G)\}$15:             $\mathcal{T}\leftarrow\mathcal{T}\cup\{\tau\}$16:          end if17:       end if18:    end for19:    Return top 3 guides $\mathcal{S}$ sorted by $S$ in descending order20:end function

The use of a memory pool to augment the Guide LLM with relevant past experiences contributes to enhanced contextual understanding. By retrieving and incorporating previously successful guides and related task information, the system can generate more accurate and context-aware guides, which leads to better quality in code generation. This enrichment ensures that the LLM is not starting from scratch for every task but instead builds upon a foundation of prior experience.

### III-C Debug LLM Agent

For the Debug Agent, after receiving the generated Generation Guide, it works in conjunction with the task description $Q$ and the entry point $E$ to facilitate code generation. The prompts used for the initial code generation and for fixing code are different, and during the initial generation phase, there is no matching with the memory pool or any failure analysis involved. This choice is similar to the approach in previous work [[14](https://arxiv.org/html/2410.01242v2#bib.bib14)], where seed code is generated before debugging to prioritize simpler tasks. However, if require running an initial execution to generate seed code for 500 samples before proceeding with debugging like MBPP dataset—this seed process can be extremely time-consuming. Especially in the case of MBPP-ET, where the length of the test set is several times longer than the original, it can lead to a significant waste of time on execution and computational costs, followed by the manual running of debugging procedures.

![Refer to caption](img/8e3960dee538b9fe1ceaa1ce4a73eed2.png)

Figure 3: Debug Agent Prompt

Conversely, for smaller dataset HumanEval, this approach can accelerate the overall debugging process. In RGD, simpler tasks are filtered out in the first round, thereby streamlining the debugging process. Figure [3](https://arxiv.org/html/2410.01242v2#S3.F3 "Figure 3 ‣ III-C Debug LLM Agent ‣ III Refinment and Guidance Debugging ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance") illustrates the corresponding prompts used at different stages. By efficiently identifying and eliminating simpler tasks early, the RGD framework ensures that computational resources are focused on more complex problems, enhancing both the speed and effectiveness of the debugging process.

![Refer to caption](img/4d1fe5496aec09950ed54d3d83f2f30b.png)

Figure 4: Feedback Agent Prompt

### III-D Feedback LLM Agent

The Feedback Agent is quite similar to the previous reflection strategy [[38](https://arxiv.org/html/2410.01242v2#bib.bib38)], where the LLM analyzes the code errors based on execution results to achieve self-repair. In the RGD architecture, the Feedback Agent plays a similar role by first recording both the failed and successful test cases and then consolidating them based on the execution results. If an exception occurs during execution, the type of exception is also recorded as part of the Real Execution Output. Subsequently, both the passed visible test cases and the failed visible test cases are provided to the Feedback LLM for analysis. This analysis also includes the failed code and the task description, as shown in Figure [4](https://arxiv.org/html/2410.01242v2#S3.F4 "Figure 4 ‣ III-C Debug LLM Agent ‣ III Refinment and Guidance Debugging ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance"). The final feedback generates a failure analysis, which serves as auxiliary information for both the Guide LLM Agent and the Debug LLM Agent.

Directly regenerating code based on execution results and failed code often results in output that is far inferior to handling a single task [[39](https://arxiv.org/html/2410.01242v2#bib.bib39)]. The Feedback Agent is responsible for independently processing the failure analysis and providing possible modification suggestions. By isolating the failure analysis and proposed corrections, the Feedback Agent enables the LLMs to perform better when dealing with complex tasks that require nuanced debugging and iteration.

## IV Experiment Setup

{tblr}

row1 = c, row4 = c, row5 = c, row6 = c, row7 = c, row8 = c, row9 = c, row10 = c, row11 = c, row12 = c, row13 = c, row14 = c, row15 = c, cell11 = r=3, cell12 = r=3, cell13 = c=16, cell24 = c=2c, cell26 = c, cell27 = c=2c, cell29 = c, cell210 = c=2c, cell212 = c, cell213 = c=2c, cell215 = c, cell216 = c=2c, cell218 = c, cell34 = c, cell35 = c, cell36 = c, cell37 = c, cell38 = c, cell39 = c, cell310 = c, cell311 = c, cell312 = c, cell313 = c, cell314 = c, cell315 = c, cell316 = c, cell317 = c, cell318 = c, cell41 = r=6, cell55 = fg=JapaneseLaurel, cell58 = fg=red, cell511 = fg=JapaneseLaurel, cell514 = fg=JapaneseLaurel1, cell517 = fg=JapaneseLaurel, cell65 = fg=JapaneseLaurel, cell68 = fg=JapaneseLaurel, cell611 = fg=red, cell614 = fg=JapaneseLaurel1, cell617 = fg=JapaneseLaurel, cell75 = fg=JapaneseLaurel, cell78 = fg=JapaneseLaurel, cell711 = fg=JapaneseLaurel1, cell714 = fg=JapaneseLaurel1, cell717 = fg=JapaneseLaurel, cell85 = fg=JapaneseLaurel, cell88 = fg=JapaneseLaurel, cell811 = fg=JapaneseLaurel, cell814 = fg=JapaneseLaurel1, cell95 = fg=JapaneseLaurel, cell98 = fg=JapaneseLaurel, cell911 = fg=JapaneseLaurel, cell914 = fg=JapaneseLaurel1, cell917 = fg=JapaneseLaurel, cell101 = r=6, cell115 = fg=JapaneseLaurel, cell118 = fg=JapaneseLaurel, cell1111 = fg=JapaneseLaurel, cell1114 = fg=JapaneseLaurel, cell1117 = fg=red, cell125 = fg=JapaneseLaurel, cell128 = fg=JapaneseLaurel, cell1211 = fg=JapaneseLaurel, cell1214 = fg=JapaneseLaurel, cell1217 = fg=JapaneseLaurel, cell135 = fg=JapaneseLaurel, cell138 = fg=JapaneseLaurel, cell1311 = fg=JapaneseLaurel, cell1314 = fg=JapaneseLaurel, cell1317 = fg=red, cell145 = fg=JapaneseLaurel, cell148 = fg=JapaneseLaurel, cell1411 = fg=JapaneseLaurel, cell1414 = fg=JapaneseLaurel, cell155 = fg=JapaneseLaurel, cell158 = fg=JapaneseLaurel, cell1511 = fg=JapaneseLaurel, cell1514 = fg=JapaneseLaurel, cell1517 = fg=JapaneseLaurel, hline1 = -, hline2,4 = 3-18, hline3 = 4-5,7-8,10-11,16-170.03em, hline3 = 13-14, hline4 = 1-20.03em, hline9,15 = 2-18dotted, hline10 = -0.05em, hline16 = -0.08em, Model & Approch Dataset
HumanEval HumanEval-ET MBPP MBPP-ET APPS-100
Acc $\uparrow$ $\Delta\uparrow$ Acc $\uparrow$ $\Delta\uparrow$ Acc $\uparrow$ $\Delta\uparrow$ Acc $\uparrow$ $\Delta\uparrow$ Acc $\uparrow$ $\Delta\uparrow$
GPT-4o Direct 87.8 84.7 67.2 56.2 56.5
COT 92.6 +4.8 82.3 -2.4 68.2 -1.0 58.6 +2.5 58.5 +3.0
Self-Planning 90.2 +2.4 90.8 +6.1 59.3 -7.9 55.8 +0.4 58.0 +1.5
Self-Debugging (+Trace) 89.0 +1.2 91.4 +6.7 66.2 +1.0 61.0 +4.8 60.0 +3.5
LDB 94.5 +6.7 91.4 +6.7 74.8 +7.6 71.0 +14.8 - -
RGD (ours) 97.6 +9.8 97.6 +12.9 83.4 +16.2 77.8 +21.6 63.0 +6.5
GPT-4o-mini Direct 87.8 77.4 57.7 44.6 47.4
COT 89.6 +1.8 78.6 +1.2 58.2 +0.6 45.3 +0.7 45.0 -2.4
Self-Planning 89.0 +1.2 88.3 +10.9 58.4 +0.7 48.2 +3.6 49.0 +1.6
Self-Debugging (+Trace) 88.4 +0.6 87.8 +10.4 64.4 +6.7 57.4 +12.8 45.0 -2.4
LDB 90.2 +2.4 89.6 +12.2 73.6 +15.9 65.8 +21.2 - -
RGD (ours) 96.9 +9.1 97.5 +20.1 80.6 +22.9 69.7 +25.1 56.0 +8.6

TABLE I: Pass@1 results for six approaches, the $\Delta$ denote the percentage improvement against the Direct baseline approach. COT [[5](https://arxiv.org/html/2410.01242v2#bib.bib5)], Self-Planning [[6](https://arxiv.org/html/2410.01242v2#bib.bib6)], Self-Debugging (+Trace) [[8](https://arxiv.org/html/2410.01242v2#bib.bib8)] tested all five benchmarks, where LDB framework tested only on HumanEval/ET and MBPP/ET benchmarks [[14](https://arxiv.org/html/2410.01242v2#bib.bib14)]

### IV-A Dataset

During the execution process, we use the test sets from various benchmarks to verify if the current code contains any bugs. In RGD, We utilized the HumanEval [[4](https://arxiv.org/html/2410.01242v2#bib.bib4)] and MBPP [[12](https://arxiv.org/html/2410.01242v2#bib.bib12)] datasets, and followed the approach in [[14](https://arxiv.org/html/2410.01242v2#bib.bib14)] for allocating visible and hidden test cases. In HumanEval, the visible tests are extracted from the task description, while the hidden tests are taken from the dataset’s own tests. In MBPP, the first test case is used as the visible test case.

Additionally, we use HumanEval-ET and MBPP-ET [[11](https://arxiv.org/html/2410.01242v2#bib.bib11)] to address the limitations of test cases from the original datasets. For these datasets, the ratio of visible to hidden test cases is set to 6:4\. The last dataset we used is APPS [[13](https://arxiv.org/html/2410.01242v2#bib.bib13)], from which we selected 100 samples for testing with difficulty levels categorized as introductory, interview, and competition in the ratio of 5:3:2\. Unlike the previous datasets, APPS relies on input-output results for validation, requiring a unique handling approach when processing the APPS dataset.

### IV-B Method

We compared RGD with five different approaches. Direct is the baseline measurement, where code is generated directly based on the task description. Chain-of-Thought [[5](https://arxiv.org/html/2410.01242v2#bib.bib5)] enhances the model’s self-planning ability by adding step-by-step prompts. Self-Planning [[6](https://arxiv.org/html/2410.01242v2#bib.bib6)] is based on the idea of planning first and then reasoning. Self-Debugging [[8](https://arxiv.org/html/2410.01242v2#bib.bib8)] involves self-reflection by incorporating feedback from execution results. Lastly, we also tested the LDB framework [[14](https://arxiv.org/html/2410.01242v2#bib.bib14)].

We conducted evaluations using the Pass@1 metric, In the set $k$ iterations (10 in our experiment), the problem is considered solved as long as it is solved successfully once. We primarily tested the currently most outperforming GPT-4o model and the GPT-4o-mini model.

## V Experiment Result and Analysis

### V-A Main Result

The main experimental results are presented in Table [I](https://arxiv.org/html/2410.01242v2#S4.T1 "TABLE I ‣ IV Experiment Setup ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance"). Here, $Acc$ represents the Pass@1 accuracy, and $\Delta$ denotes the percentage improvement over the direct code generation approach based on the task description. We can observe that RGD demonstrates state-of-the-art performance across all datasets, with further improvements compared to the LDB approach. For the HumanEval dataset with GPT-4o, RGD shows an improvement of 9.8% over the baseline and 3.1% over LDB. Our experiments reveal that RGD achieves particularly significant improvements on the HumanEval-ET and MBPP-ET benchmarks, which contain more test cases, with a maximum improvement of 25.1% on MBPP-ET. This performance boost is mainly due to the limited number of visible test cases in the original datasets, leading to situations where code that passes the specific tests still contains vulnerabilities and fails the hidden test cases, this highlights the critical role of the Feedback Agent’s analysis throughout the debugging process.

For the APPS dataset, we conducted experiments using a total of 100 samples. We did not test the LDB framework on the APPS benchmark because it would require significant modifications to its program. Overall, it is evident that RGD achieves enhancements over other methods, with a maximum improvement of 8.6% in the GPT-4o-mini model.

{tblr}

cells = c, cell11 = r=3, cell12 = c=8, cell22 = c=2, cell25 = c=2, cell28 = c=2, hline1,4,6 = -, hline2 = 2-9, hline3 = 2-3,5-6,8-9, Benchmark & Component Remove From RGD
Memory Pool Guide Agent Failure Feedback
Acc Drop Acc Drop Acc Drop
HumanEval 95.7% 1.9% 93.2% 4.4% 95.1% 2.5%
MBPP 78.4% 5.0% 77.0% 6.4% 73.6% 9.8%

TABLE II: Ablation study on the RGD framework using the GPT-4o model, evaluated on the HumanEval and MBPP

### V-B Ablations Studies

We conducted ablation studies to validate the effectiveness of our approach. We selected GPT-4o for experiments on two benchmarks, HumanEval and MBPP, and compared the results with the original RGD framework’s performance on these benchmarks, which were 97.6 pass@1 for HumanEval and 83.4 pass@1 for MBPP, respectively. We performed ablation tests under three different scenarios:

Memory Pool Removal: In this scenario, we removed the memory pool, and the Guide Agent no longer retrieved cases from it to generate and refine guides. As shown in Table [II](https://arxiv.org/html/2410.01242v2#S5.T2 "TABLE II ‣ V-A Main Result ‣ V Experiment Result and Analysis ‣ RGD: Multi-LLM Based Agent Debugger via Refinement and Generation Guidance"), this resulted in a drop of 1.9 percentage points on the HumanEval dataset and a drop of 5 percentage points on the MBPP dataset.

Guide Agent Removal: In this case, we removed the entire Guide Agent component, this led to a 4.4% drop on the HumanEval dataset and a 6.4% drop on the MBPP dataset. The results clearly demonstrate the significant impact of the Guide Agent on the overall performance of the RGD framework.

Failure Feedback Removal: Here, we removed the Failure Feedback component, and the Feedback LLM did not generate Failure Analysis to assist with debugging and refinement. The results showed performance drops on both datasets, especially on the MBPP dataset, where the decrease was as high as 9.8%.

The ablation study results confirm that the inclusion of the Memory Pool, Guide Agent, and Failure Feedback significantly enhances the overall performance of the RGD framework. Furthermore, we observe a trend from the drop percentages: datasets with more samples (e.g., MBPP) tend to experience a more substantial impact from the ablations. This also corroborates that the RGD framework maintains excellent performance when dealing with a larger number of tests. Both the Memory Pool and Failure Feedback influence subsequent refinement and debugging, and the information in the memory pool is accumulated during execution. This explains why the improvement on MBPP (500 samples) is greater than on HumanEval (163 samples).

## VI Conclusion

In this paper, we present RGD, a novel framework designed to enhance code generation by leveraging a Memory Pool for refining Generation Guides and incorporating feedback through a dedicated Feedback Agent. Our approach allows LLMs to iteratively refine generated code by utilizing previously stored memory and dynamic failure analysis. By selectively extracting relevant guides from the Memory Pool and continuously refining them based on task similarity, RGD significantly boosts the accuracy of code generation across multiple benchmarks. Experiments demonstrate that RGD achieves state-of-the-art performance in code generation and debugging. We anticipate that our work further demonstrates and enhances the capability of LLMs to learn from past, and efficiently adapt to new challenges.

## References

*   [1] A. Ni, P. Yin, Y. Zhao, M. Riddell, T. Feng, R. Shen, S. Yin, Y. Liu, S. Yavuz, C. Xiong, S. Joty, Y. Zhou, D. Radev, and A. Cohan, “L2ceval: Evaluating language-to-code generation capabilities of large language models,” 2023.
*   [2] R. Sun, S. Ö. Arik, A. Muzio, L. Miculicich, S. Gundabathula, P. Yin, H. Dai, H. Nakhost, R. Sinha, Z. Wang, et al., “Sql-palm: Improved large language model adaptation for text-to-sql (extended),” arXiv preprint arXiv:2306.00739, 2023.
*   [3] Q. Zheng, X. Xia, X. Zou, Y. Dong, S. Wang, Y. Xue, Z. Wang, L. Shen, A. Wang, Y. Li, T. Su, Z. Yang, and J. Tang, “Codegeex: A pre-trained model for code generation with multilingual benchmarking on humaneval-x,” 2024.
*   [4] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. D. O. Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, et al., “Evaluating large language models trained on code,” arXiv preprint arXiv:2107.03374, 2021.
*   [5] J. Wei, X. Wang, D. Schuurmans, M. Bosma, F. Xia, E. Chi, Q. V. Le, D. Zhou, et al., “Chain-of-thought prompting elicits reasoning in large language models,” Advances in neural information processing systems, vol. 35, pp. 24824–24837, 2022.
*   [6] X. Jiang, Y. Dong, L. Wang, F. Zheng, Q. Shang, G. Li, Z. Jin, and W. Jiao, “Self-planning code generation with large language models,” ACM Trans. Softw. Eng. Methodol., jun 2024. Just Accepted.
*   [7] E. Nijkamp, B. Pang, H. Hayashi, L. Tu, H. Wang, Y. Zhou, S. Savarese, and C. Xiong, “Codegen: An open large language model for code with multi-turn program synthesis,” 2023.
*   [8] X. Chen, M. Lin, N. Schärli, and D. Zhou, “Teaching large language models to self-debug,” arXiv preprint arXiv:2304.05128, 2023.
*   [9] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. Narasimhan, and Y. Cao, “React: Synergizing reasoning and acting in language models,” arXiv preprint arXiv:2210.03629, 2022.
*   [10] P. Lewis, E. Perez, A. Piktus, F. Petroni, V. Karpukhin, N. Goyal, H. Küttler, M. Lewis, W.-t. Yih, T. Rocktäschel, et al., “Retrieval-augmented generation for knowledge-intensive nlp tasks,” Advances in Neural Information Processing Systems, vol. 33, pp. 9459–9474, 2020.
*   [11] Y. Dong, J. Ding, X. Jiang, G. Li, Z. Li, and Z. Jin, “Codescore: Evaluating code generation by learning code execution,” arXiv preprint arXiv:2301.09043, 2023.
*   [12] J. Austin, A. Odena, M. Nye, M. Bosma, H. Michalewski, D. Dohan, E. Jiang, C. Cai, M. Terry, Q. Le, and C. Sutton, “Program synthesis with large language models,” 2021.
*   [13] D. Hendrycks, S. Basart, S. Kadavath, M. Mazeika, A. Arora, E. Guo, C. Burns, S. Puranik, H. He, D. Song, and J. Steinhardt, “Measuring coding challenge competence with apps,” NeurIPS, 2021.
*   [14] L. Zhong, Z. Wang, and J. Shang, “Ldb: A large language model debugger via verifying runtime execution step-by-step,” arXiv preprint arXiv:2402.16906, 2024.
*   [15] B. Roziere, J. Gehring, F. Gloeckle, S. Sootla, I. Gat, X. E. Tan, Y. Adi, J. Liu, T. Remez, J. Rapin, et al., “Code llama: Open foundation models for code,” arXiv preprint arXiv:2308.12950, 2023.
*   [16] B. Yetiştiren, I. Özsoy, M. Ayerdem, and E. Tüzün, “Evaluating the code quality of ai-assisted code generation tools: An empirical study on github copilot, amazon codewhisperer, and chatgpt,” arXiv preprint arXiv:2304.10778, 2023.
*   [17] H. Jin, L. Huang, H. Cai, J. Yan, B. Li, and H. Chen, “From llms to llm-based agents for software engineering: A survey of current, challenges and future,” 2024.
*   [18] J. Yang, H. Jin, R. Tang, X. Han, Q. Feng, H. Jiang, S. Zhong, B. Yin, and X. Hu, “Harnessing the power of llms in practice: A survey on chatgpt and beyond,” ACM Trans. Knowl. Discov. Data, vol. 18, apr 2024.
*   [19] J. White, Q. Fu, S. Hays, M. Sandborn, C. Olea, H. Gilbert, A. Elnashar, J. Spencer-Smith, and D. C. Schmidt, “A prompt pattern catalog to enhance prompt engineering with chatgpt,” arXiv preprint arXiv:2302.11382, 2023.
*   [20] D. Huang, Q. Bu, Y. Qing, and H. Cui, “Codecot: Tackling code syntax errors in cot reasoning for code generation,” 2024.
*   [21] S. Yao, D. Yu, J. Zhao, I. Shafran, T. Griffiths, Y. Cao, and K. Narasimhan, “Tree of thoughts: Deliberate problem solving with large language models,” Advances in Neural Information Processing Systems, vol. 36, 2024.
*   [22] X. Hu, K. Kuang, J. Sun, H. Yang, and F. Wu, “Leveraging print debugging to improve code generation in large language models,” 2024.
*   [23] Y. Ding, M. J. Min, G. Kaiser, and B. Ray, “Cycle: Learning to self-refine the code generation,” Proc. ACM Program. Lang., vol. 8, apr 2024.
*   [24] Q. Wu, G. Bansal, J. Zhang, Y. Wu, S. Zhang, E. Zhu, B. Li, L. Jiang, X. Zhang, and C. Wang, “Autogen: Enabling next-gen llm applications via multi-agent conversation framework,” arXiv preprint arXiv:2308.08155, 2023.
*   [25] T. X. Olausson, J. P. Inala, C. Wang, J. Gao, and A. Solar-Lezama, “Is self-repair a silver bullet for code generation?,” in The Twelfth International Conference on Learning Representations, 2023.
*   [26] S. Jiang, Y. Wang, and Y. Wang, “Selfevolve: A code evolution framework via large language models,” arXiv preprint arXiv:2306.02907, 2023.
*   [27] X. He, J. Zou, Y. Lin, M. Zhou, S. Han, Z. Yuan, and D. Zhang, “Conline: Complex code generation and refinement with online searching and correctness testing,” arXiv preprint arXiv:2403.13583, 2024.
*   [28] Y. Gao, Y. Xiong, X. Gao, K. Jia, J. Pan, Y. Bi, Y. Dai, J. Sun, and H. Wang, “Retrieval-augmented generation for large language models: A survey,” arXiv preprint arXiv:2312.10997, 2023.
*   [29] K. Guu, K. Lee, Z. Tung, P. Pasupat, and M. Chang, “Retrieval augmented language model pre-training,” in International conference on machine learning, pp. 3929–3938, PMLR, 2020.
*   [30] T. Procko, “Graph retrieval-augmented generation for large language models: A survey,” Available at SSRN, 2024.
*   [31] K. Zhang, H. Zhang, G. Li, J. Li, Z. Li, and Z. Jin, “Toolcoder: Teach code generation models to use api search tools,” arXiv preprint arXiv:2305.04032, 2023.
*   [32] W. Fan, Y. Ding, L. Ning, S. Wang, H. Li, D. Yin, T.-S. Chua, and Q. Li, “A survey on rag meeting llms: Towards retrieval-augmented large language models,” in Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, pp. 6491–6501, 2024.
*   [33] M. A. Islam, M. E. Ali, and M. R. Parvez, “Mapcoder: Multi-agent code generation for competitive problem solving,” arXiv preprint arXiv:2405.11403, 2024.
*   [34] N. Reimers and I. Gurevych, “Sentence-bert: Sentence embeddings using siamese bert-networks,” 2019.
*   [35] Z. Feng, D. Guo, D. Tang, N. Duan, X. Feng, M. Gong, L. Shou, B. Qin, T. Liu, D. Jiang, and M. Zhou, “Codebert: A pre-trained model for programming and natural languages,” 2020.
*   [36] J. Lin and X. Ma, “A few brief notes on deepimpact, coil, and a conceptual framework for information retrieval techniques,” 2021.
*   [37] V. Karpukhin, B. Oğuz, S. Min, P. Lewis, L. Wu, S. Edunov, D. Chen, and W. tau Yih, “Dense passage retrieval for open-domain question answering,” 2020.
*   [38] N. Shinn, F. Cassano, E. Berman, A. Gopinath, K. Narasimhan, and S. Yao, “Reflexion: Language agents with verbal reinforcement learning,” 2023.
*   [39] S. Moon, H. Chae, Y. Song, T. Kwon, D. Kang, K. T. iunn Ong, S. won Hwang, and J. Yeo, “Coffee: Boost your code llms by fixing bugs with feedback,” 2024.