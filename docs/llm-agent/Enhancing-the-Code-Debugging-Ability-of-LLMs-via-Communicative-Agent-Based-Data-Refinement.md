<!--yml
category: 未分类
date: 2025-01-11 12:19:34
-->

# Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement

> 来源：[https://arxiv.org/html/2408.05006/](https://arxiv.org/html/2408.05006/)

Weiqing Yang *indicates both authors contributed equally to this work. Northeastern University
Shenyang, China    Hanbin Wang Peking University
Beijing, China    Zhenghao Liu^† ^†indicates corresponding author. Northeastern University
Shenyang, China    Xinze Li Northeastern University
Shenyang, China    Yukun Yan Tsinghua University
Beijing, China    Shuo Wang Tsinghua University
Beijing, China    Yu Gu Northeastern University
Shenyang, China    Minghe Yu Northeastern University
Shenyang, China    Zhiyuan Liu Tsinghua University
Beijing, China    Ge Yu Northeastern University
Shenyang, China

###### Abstract

Debugging is a vital aspect of software development, yet the debugging capabilities of Large Language Models (LLMs) remain largely unexplored. This paper first introduces DebugEval, a comprehensive benchmark designed to evaluate the debugging capabilities of LLMs. DebugEval collects data from existing high-quality datasets and designs four different tasks to evaluate the debugging effectiveness, including BUG Localization, BUG Identification, Code Review, and Code Repair. Additionally, to enhance the code debugging ability of LLMs, this paper proposes a CoMmunicative Agent BaSed DaTa REfinement FRamework (MASTER), which generates the refined code debugging data for supervised finetuning. Specifically, MASTER employs the Code Quizzer to generate refined data according to the defined tasks of DebugEval. Then the Code Learner acts as a critic and reserves the generated problems that it can not solve. Finally, the Code Teacher provides a detailed Chain-of-Thought based solution to deal with the generated problem. We collect the synthesized data and finetune the Code Learner to enhance the debugging ability and conduct the NeuDebugger model. Our experiments evaluate various LLMs and NeuDebugger in the zero-shot setting on DebugEval. Experimental results demonstrate that these 7B-scale LLMs have weaker debugging capabilities, even these code-oriented LLMs. On the contrary, these larger models (over 70B) show convincing debugging ability. Our further analyses illustrate that MASTER is an effective method to enhance the code debugging ability by synthesizing data for Supervised Fine-Tuning (SFT) LLMs. All data and codes are available at https://github.com/NEUIR/DebugEval.

###### Index Terms:

Code Debugging, Large Language Models, DebugEval Benchmark, Data Refinement, Agents

## I Introduction

In the realm of software development, code debugging is an indispensable process for ensuring the functionality and reliability of applications [[1](https://arxiv.org/html/2408.05006v1#bib.bib1), [2](https://arxiv.org/html/2408.05006v1#bib.bib2)]. With the complexity of software systems grows, traditional debugging methods, which often rely on heuristics [[3](https://arxiv.org/html/2408.05006v1#bib.bib3), [4](https://arxiv.org/html/2408.05006v1#bib.bib4)] and predefined patterns [[5](https://arxiv.org/html/2408.05006v1#bib.bib5), [6](https://arxiv.org/html/2408.05006v1#bib.bib6)], are reaching their limitations. The emergent ability of Large Language Models (LLMs) [[7](https://arxiv.org/html/2408.05006v1#bib.bib7), [8](https://arxiv.org/html/2408.05006v1#bib.bib8)] has opened up new horizons in automated debugging, offering a more flexible and comprehensive approach to identifying and rectifying code errors [[9](https://arxiv.org/html/2408.05006v1#bib.bib9)].

The capabilities of LLMs have been extensively explored for code-related tasks such as code generation and translation [[10](https://arxiv.org/html/2408.05006v1#bib.bib10), [11](https://arxiv.org/html/2408.05006v1#bib.bib11), [12](https://arxiv.org/html/2408.05006v1#bib.bib12), [13](https://arxiv.org/html/2408.05006v1#bib.bib13), [14](https://arxiv.org/html/2408.05006v1#bib.bib14)]. However, their debugging capabilities remain relatively underexplored. Recently, researchers have begun to focus on using LLMs for self-debugging to repair buggy code iteratively [[9](https://arxiv.org/html/2408.05006v1#bib.bib9), [15](https://arxiv.org/html/2408.05006v1#bib.bib15), [16](https://arxiv.org/html/2408.05006v1#bib.bib16)]. To better evaluate the code debugging ability, researchers are now building benchmarks to assess the code debugging capabilities of LLMs [[17](https://arxiv.org/html/2408.05006v1#bib.bib17), [18](https://arxiv.org/html/2408.05006v1#bib.bib18)]. Nevertheless, existing code debugging benchmarks face two main issues: 1) They primarily design tasks around code repair, which is insufficient for a comprehensive evaluation of code debugging ability. 2) Constructing buggy code using GPT-4 [[19](https://arxiv.org/html/2408.05006v1#bib.bib19)] fails to capture the complexity and diversity of code errors encountered in real development environments.

To address these challenges, this paper introduces DebugEval, a comprehensive benchmark designed to evaluate the debugging capabilities of LLMs. DebugEval introduces four tasks: BUG Localization, BUG Identification, Code Review, and Code Repair, which are designed to test the LLMs’ ability not only to identify and classify errors but also to provide correct code solutions. These tasks are of different difficulty levels and represent common debugging scenarios in real software development environments, making the evaluation both representative and challenging. Besides, each task in DebugEval includes Python, C++, and Java. Additionally, DebugEval incorporates the real user-written buggy codes and GPT-4 generated buggy codes to better simulate real-world software development. While collecting user-written buggy codes, we also implement strict quality control to prevent data leakage in DebugEval.

Additionally, this paper proposes a coMmunicative Agent baSed daTa rEfinement fRamework (MASTER), which focuses on improving the debugging ability of Large Language Models (LLMs). To ensure the quality of the Supervised Fine-Tuning (SFT) data, MASTER defines three agents: Code Quizzer, Code Learner, and Code Teacher, which collaborate to synthesize high-quality code debugging data for finetuning the Code Learner. Specifically, the Code Quizzer generates a diverse set of code debugging problems, guiding the LLM to acquire debugging knowledge during SFT. The Code Learner then acts as a critic, evaluating the educational value of the synthesized problems. Problems that the Code Learner answers incorrectly are collected as SFT data, and the Code Teacher provides detailed solutions and explanations for these problems. Finally, we finetune the Code Learner using the synthesized SFT data and develop our NeuDebugger model.

![Refer to caption](img/4fbeb37eabe039c186712ca56c82b22e.png)

Figure 1: The performance of LLMs on DebugEval Benchmark. Left: We evaluate the code debugging capability of LLMs on four tasks: BUG Localization, BUG Identification, Code Review, and Code Repair. The radar chart shows the distribution of performance on each task, reflecting the strengths and weaknesses of different LLMs for these debugging tasks. Right: The average performance of open-source and closed-source models on four tasks. The same color indicates that the number of parameters belongs to the same level.

We benchmark 13 open-source and closed-source LLMs on DebugEval and evaluate the overall performance of NeuDebugger. The results are shown in Figure [1](https://arxiv.org/html/2408.05006v1#S1.F1 "Figure 1 ‣ I Introduction ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), indicating that: 1) Models with 7 billion parameters exhibit relatively weaker debugging capabilities, whereas 70 billion parameter models and closed-source models perform better. 2) The DeepSeek series models demonstrate superior performance, with the open-source model DeepSeek-Coder-V2-0724 outperforming the closed-source model GPT-4o-mini. 3) NeuDebugger-DS-6.7B and NeuDebugger-Llama3-8B, based on DeepSeek-Coder-6.7B-Ins and Llama3-8B-Ins respectively, achieve improvements of 27.7% and 4.1% when trained on data synthesized by MASTER, indicating that MASTER can significantly refine SFT data to enhance model performance on debugging tasks.

Further analysis reveals that collecting data solely for SFT does not enhance the debugging ability of LLMs [[20](https://arxiv.org/html/2408.05006v1#bib.bib20)]. The Code Teacher effectively teaches the Code Student in three tasks: BUG Localization, BUG Identification, and Code Review, by generating the Chain-of-Thought (CoT) [[21](https://arxiv.org/html/2408.05006v1#bib.bib21)]. However, CoT outcomes decrease the performance of LLMs in the Code Repair task, as they introduce additional noise and disrupt the code structure. During SFT, different models exhibit distinct learning behaviors. We find that synthesized data can significantly improve the performance of code-oriented LLMs, such as DeepSeek-Coder-6.7B-Ins, than general LLMs, such as Llama3-8B-Int. These experimental findings provide important insights and directions for future research on enhancing the debugging capabilities of LLMs.

## II Related Work

This section first introduces some backbone models for code understating and generation and then introduces the related work of code debugging and repair.

Code-Oriented Language Models. To tailor language models for code understanding, related work mainly focuses on pretrained language models and guides them to learn the code semantics of syntax, and idiomatic [[22](https://arxiv.org/html/2408.05006v1#bib.bib22), [23](https://arxiv.org/html/2408.05006v1#bib.bib23), [24](https://arxiv.org/html/2408.05006v1#bib.bib24)]. CodeBERT masks tokens of Natural Language (NL) and Program Language (PL) and then asks the pretrained language models to fill-in the masked spans. Then CodeBERT follows the ELECTRA method [[25](https://arxiv.org/html/2408.05006v1#bib.bib25)] and pretrains language models to detect whether the tokens are replaced, which helps models to better capture the code semantics [[26](https://arxiv.org/html/2408.05006v1#bib.bib26)]. DOBF [[27](https://arxiv.org/html/2408.05006v1#bib.bib27)] goes a step further by taking into account the unique attributes of code-related tasks, which focuses more on masking and generating the class, function, and variable names of code segments. CodeT5 [[28](https://arxiv.org/html/2408.05006v1#bib.bib28)] continuously pretrains T5 models [[29](https://arxiv.org/html/2408.05006v1#bib.bib29)] using the span masking strategy and also refines the masking strategy by focusing more on the identifiers within code. Such a pretraining method asks the T5 [[29](https://arxiv.org/html/2408.05006v1#bib.bib29)] model to generate these identifiers, thereby enhancing its ability to identify and understand identifier information in code-related tasks. Furthermore, some researchers also incorporate multi-modal data sources, such as code, comments, and abstract syntax trees (AST), to pretrain language models, which also helps to improve the code understanding ability by aligning the semantic between code semantics and natural language [[30](https://arxiv.org/html/2408.05006v1#bib.bib30), [31](https://arxiv.org/html/2408.05006v1#bib.bib31)].

Recently, Large Language Models (LLMs), such as ChatGPT [[7](https://arxiv.org/html/2408.05006v1#bib.bib7)] and Llama [[8](https://arxiv.org/html/2408.05006v1#bib.bib8)], have demonstrated their emergency ability in dealing with different tasks, especially for code understanding and generation tasks. To enhance the code generation ability, some widely used LLMs, such as ChatGPT [[7](https://arxiv.org/html/2408.05006v1#bib.bib7)], also mix some code data in the pretraining corpus, which has proven its advantage in enhancing the reasoning ability of LLMs [[32](https://arxiv.org/html/2408.05006v1#bib.bib32), [33](https://arxiv.org/html/2408.05006v1#bib.bib33), [34](https://arxiv.org/html/2408.05006v1#bib.bib34)]. Some typical code-based LLMs also collect some instruction data of different code-related tasks to supervised finetune LLMs, which significantly improves the code generation ability of LLMs [[35](https://arxiv.org/html/2408.05006v1#bib.bib35), [36](https://arxiv.org/html/2408.05006v1#bib.bib36), [37](https://arxiv.org/html/2408.05006v1#bib.bib37)]. Even though LLMs have strong effectiveness in generating code segments, the code segments usually contain bugs [[9](https://arxiv.org/html/2408.05006v1#bib.bib9)], decreasing the pass rate of generated codes. To alleviate these problems, the existing efforts primarily concentrate on employing an iterative code repair approach to continuously refine generated code segments [[9](https://arxiv.org/html/2408.05006v1#bib.bib9), [16](https://arxiv.org/html/2408.05006v1#bib.bib16), [15](https://arxiv.org/html/2408.05006v1#bib.bib15)].

Code Debugging and Repair. Early debugging models primarily rely on feature-based methods, such as using templates [[5](https://arxiv.org/html/2408.05006v1#bib.bib5), [6](https://arxiv.org/html/2408.05006v1#bib.bib6)], heuristic rules [[3](https://arxiv.org/html/2408.05006v1#bib.bib3), [4](https://arxiv.org/html/2408.05006v1#bib.bib4)], or constraints [[38](https://arxiv.org/html/2408.05006v1#bib.bib38), [39](https://arxiv.org/html/2408.05006v1#bib.bib39)] to correct the buggy codes. However, the effectiveness of these feature-based debugging methods is hard to broaden to correcting different bug errors and dealing with more complex code bugs, because of the limited patterns or rules that need predefinition by researchers.

With the development of Pre-trained Language Models (PLMs), the work also follows the pretraining and then finetuning strategy to build the debugging model and deal with various code bugs occurred in real life. For example, Xia et al. [[40](https://arxiv.org/html/2408.05006v1#bib.bib40)] use the code-oriented pretrained model, CodeX [[41](https://arxiv.org/html/2408.05006v1#bib.bib41)], to explore the capabilities of PLMs in debugging. It shows that CodeX [[41](https://arxiv.org/html/2408.05006v1#bib.bib41)] achieves convinced code repair performance, especially on both Python and Java program languages. Kolak et al. [[42](https://arxiv.org/html/2408.05006v1#bib.bib42)] also use GPT-2 [[43](https://arxiv.org/html/2408.05006v1#bib.bib43)] and CodeX [[41](https://arxiv.org/html/2408.05006v1#bib.bib41)] to evaluate their effectiveness in generating the correct patch line when given corresponding code prefix. All the research has proven that the effectiveness of PLM based debugging models mainly thrives on the code understanding ability obtained during pretraining.

Different from previous debugging work, recent research focuses more on correcting the bugs generated by LLMs. Self-Debug [[9](https://arxiv.org/html/2408.05006v1#bib.bib9)] prompts LLMs to generate the code reviews to aid themselves to refine generated codes, while Self-Repair [[44](https://arxiv.org/html/2408.05006v1#bib.bib44)] incorporates human-provided feedback for repairing the buggy code. Furthermore, Self-Edit [[15](https://arxiv.org/html/2408.05006v1#bib.bib15)] trains an additional fault-aware editor to repair codes by leveraging the error messages from the test case evaluation and generated code segments. Wang et al. [[16](https://arxiv.org/html/2408.05006v1#bib.bib16)] further explore the effectiveness of interactive Chain-of-Repair (CoR), which uses LLMs to generate the guidelines for repairing codes by incorporating the generated codes and error messages from the code compiler. It is evident that the effectiveness of these code repair models mainly relies on the debugging ability of LLMs.

To improve the debugging ability of LLMs, recent work focuses more on generating data for supervised fine-tuning. InstructCoder [[45](https://arxiv.org/html/2408.05006v1#bib.bib45)] employs the Self-Instruct method [[46](https://arxiv.org/html/2408.05006v1#bib.bib46)] to build an instruction-tuning dataset to improve the effectiveness of LLMs in code debugging. Li et al. [[47](https://arxiv.org/html/2408.05006v1#bib.bib47)] further construct the APR-INSTRUCTION dataset and utilize the dataset to finetune LLMs using four different Parameter-Efficient Fine-Tuning (PEFT) methods LoRA [[48](https://arxiv.org/html/2408.05006v1#bib.bib48)], p-tuning [[49](https://arxiv.org/html/2408.05006v1#bib.bib49)], prefix-tuning [[50](https://arxiv.org/html/2408.05006v1#bib.bib50)] and $(IA)^{3}$ [[51](https://arxiv.org/html/2408.05006v1#bib.bib51)]. To further evaluate the debugging capabilities of LLMs, some researchers focus on building benchmarks to evaluate LLMs. Wang et al. [[16](https://arxiv.org/html/2408.05006v1#bib.bib16)] collect real user-submitted buggy code from the Atcoder website to construct CodeError, which is used to evaluate the repair abilities of LLMs on Python codes. Tian et al. [[17](https://arxiv.org/html/2408.05006v1#bib.bib17)] further propose DebugBench to explore the debugging capabilities of LLMs in Python, C++, and Java. They use GPT-4 [[19](https://arxiv.org/html/2408.05006v1#bib.bib19)] to insert some code errors into code segments to synthesize the buggy codes and ask LLMs to repair codes. Nevertheless, only code repair task is insufficient to evaluate the debugging ability of LLMs.

![Refer to caption](img/8e7d29dc76c7cfe9b6df35deddd7c532.png)

Figure 2: DebugEval Benchmark. DebugEval includes four tasks: BUG Localization, BUG Identification, Code Review, and Code Repair. The first three tasks are multiple-choice questions and are evaluated with Accuracy. In Code Review task, we swap the contents of options A and B to avoid any potential bias. The Code Repair task is evaluated using Pass@$1$.

## III Evaluating the Debugging Ability of LLMs with DebugEval

In this section, we introduce the benchmark DebugEval, which is built to evaluate the debugging capabilities of Large Language Models (LLMs) from different aspects. We first describe the task definition of the designed task in DebugEval (Sec. [III-A](https://arxiv.org/html/2408.05006v1#S3.SS1 "III-A Task Definition ‣ III Evaluating the Debugging Ability of LLMs with DebugEval ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement")). Then we detail the process of constructing the DebugEval benchmark (Sec. [III-B](https://arxiv.org/html/2408.05006v1#S3.SS2 "III-B Details of Data Construction ‣ III Evaluating the Debugging Ability of LLMs with DebugEval ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement")).

### III-A Task Definition

DebugEval introduces four different tasks to evaluate the debugging ability of LLMs. The evaluation tasks include BUG Localization, BUG Identification, Code Review, and Code Repair.

The data statistics of DebugEval are shown in Table [I](https://arxiv.org/html/2408.05006v1#S3.T1 "TABLE I ‣ III-A Task Definition ‣ III Evaluating the Debugging Ability of LLMs with DebugEval ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"). For each task, there are questions in Python, C++, and Java to evaluate LLMs’ debugging performance on different programming languages. There are 578, 2320, 2400, and 414 test instances respectively for BUG Localization, BUG Identification, Code Review, and Code Repair tasks, nearly evenly distributed among the three programming languages. In the rest of this subsection, we present the illustrations of these four evaluation tasks in Figure [2](https://arxiv.org/html/2408.05006v1#S2.F2 "Figure 2 ‣ II Related Work ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement") and delve deeper to describe each evaluation task.

TABLE I: Data Statistics of DebugEval.

 | Evaluation Tasks | Language | #Test |
| BUG Localization: Identify which code fragm- ent in the buggy code causing the error. | Python | 178 |
| C++ | 195 |
| Java | 205 |
| BUG Identification: Identify the type of error in your code. | Python | 760 |
| C++ | 800 |
| Java | 760 |
| Code Review: Determine which code is incor- rect. | Python | 800 |
| C++ | 800 |
| Java | 800 |
| Code Repair: Fixing the buggy code to passes all the test cases. | Python | 138 |
| C++ | 138 |
| Java | 138 | 

BUG Localization. The BUG Localization task focuses on identifying the specific line(s) of code that contains the error. It evaluates the ability of LLMs to point out the exact location where the error occurs within a code snippet, which is usually regarded as the first step in the debugging process. For each test instance of the BUG Localization task, we give a buggy code $P$, extract four code snippets $\{S_{A},S_{B},S_{C},S_{D}\}$ from $P$, and then ask LLMs to identify the golden code snippet $S_{E}$, which contains error.

BUG Identification. In this task, the LLMs should classify the type of error that occurred in the code. Specifically, given a program $P$ with code error(s), we ask LLMs to classify the error type $E$ from four choices, including SyntaxError, ReferenceError, LogicError, and MultiErrors. SyntaxError indicates that the code contains a syntax error. ReferenceError in programming typically occurs when code attempts to access a variable, function, or object that has not been declared or is out of scope. The LogicError represents that the code is usually syntactically correct but contains logical error, not getting the expected output. MultiErrors indicates that the code segment contains various errors of SyntaxError, ReferenceError, and LogicError.

Code Review. For the Code Review task, we give the correct code $C_{i}$ and the error code $E_{i}$ to ask LLMs to distinguish the error one. Specifically, only a few code snippets are different between $C_{i}$ and $E_{i}$. Moreover, we swap the choice identifiers (A and B) of $C_{i}$ and $E_{i}$ in the experiment to avoid any potential bias.

Code Repair. The Code Repair [[16](https://arxiv.org/html/2408.05006v1#bib.bib16), [17](https://arxiv.org/html/2408.05006v1#bib.bib17)] task requires to generate the corrected version $P^{\prime}$ for the given buggy code $P$, which is the ultimate test of the model’s debugging capabilities. The code repair task is more difficult among these four debugging tasks. It not only involves code error detection/identification but also needs to correct the error of the code. After generating the corrected code $P^{\prime}$, we evaluate the correctness of $P^{\prime}$ by using $n$ test cases $X=\{(x_{1},y_{1}),...,(x_{n},y_{n})\}$. Specifically, we feed the input $x_{i}$ of the test case $(x_{i},y_{i})$ to the corrected code $P^{\prime}$ and then get the execution result $P^{\prime}(x_{i})$. If there exists the test case $(x_{i},y_{i})\in X$ that satisfies $P^{\prime}(x_{j})\neq y_{j}$, it indicates that the code $P^{\prime}$ still contains errors that need to be addressed. The Code Repair task aims to transform the faulty program $P$ to $P^{\prime}$ that pass all test cases ($\forall(x_{i},y_{i})\in(x_{i},y_{i}),P^{\prime}(x_{i})=y_{i}$).

### III-B Details of Data Construction

In this subsection, we elaborate on the source data collection and construction method for the DebugEval dataset.

To ensure the quality of DebugEval, we collect high-quality data from reliable data sources, such as DebugBench [[17](https://arxiv.org/html/2408.05006v1#bib.bib17)] and LiveCodeBench [[52](https://arxiv.org/html/2408.05006v1#bib.bib52)], and the AtCoder website¹¹1[https://atcoder.jp](https://atcoder.jp). The DebugBench focuses on the code repair task, which needs to call LeetCode API to evaluate the correctness of the generated code. Each test instance of DebugBench consists of a buggy code and corresponding error type. The LiveCodeBench is a code generation dataset, each test instance contains a task description, correct code, and test cases to evaluate the correctness of generated codes. In this paper, we only use the questions from LiveCodeBench and construct one correct code and buggy code pair for each question from the AtCoder website.

Bug Localization. For the Bug Localization task, we sample test instances from DebugBench. In detail, we built the dataset for the Bug Localization task by sampling up to 20 instances of each of the 15 single code error types for each programming language. We compare the buggy code with the correct code to find the code snippet that contains errors as the golden choice $S_{E}$. To construct the other confusing choices, we discard the error code snippet and randomly sample one code line as one of the confusing choices.

Bug Identification. To conduct the evaluation on the Bug Identification task, we also sample the test instances from DebugBench. Specifically, the choices of the task are divided into four types: SyntaxError, ReferenceError, LogicError, and MultiErrors, thus it is important to ensure the balance of each choice (code error type). For each programming language, we compare the sizes of the test sample sets for different types of code errors and then select the smallest set size as the sampling number. Finally, we sample test instances from the different code error test sets according to this sampling number.

TABLE II: A Comparison between DebugEval and Other Code Repair Benchmarks. Size only represents the size of the test set. CE indicates compilation errors (e.g., SyntaxError). Source of Buggy Code indicates how the buggy code was constructed. Against Data Leakage indicates whether there is a data leakage. Part of the data of DebugEval is from the DebugBench, and there is no data leakage. The data collected by us is all after 2023-09-01, which also avoids data leakage.

 | Dataset | Language | Task | Size | Error Type | Source of Bugs | Against Data Leakage |
| DeepFix [[53](https://arxiv.org/html/2408.05006v1#bib.bib53)] | C | Code Repair | 6,971 | CE Only | User Submission | ✗ |
| Review4Repair [[54](https://arxiv.org/html/2408.05006v1#bib.bib54)] | Java | Code Repair | 2,961 | All | User Submission | ✗ |
| Bug2Fix [[55](https://arxiv.org/html/2408.05006v1#bib.bib55)] | Java | Code Repair | 5,835 | All | User Submission | ✗ |
| Github-Python [[53](https://arxiv.org/html/2408.05006v1#bib.bib53)] | Python | Code Repair | 15K | CE Only | User Submission | ✗ |
| FixEval [[56](https://arxiv.org/html/2408.05006v1#bib.bib56)] | Java/Python | Code Repair | 43k/243k | All | User Submission | ✗ |
| CodeError [[16](https://arxiv.org/html/2408.05006v1#bib.bib16)] | Python | Code Repair | 4,463 | All | User Submission | ✗ |
| DebugBench [[17](https://arxiv.org/html/2408.05006v1#bib.bib17)] | C++/Java/Python | Code Repair | 1,438/1,401/1,414 | All | GPT-4 Generation | ✓ |
| CodeEditorBench [[18](https://arxiv.org/html/2408.05006v1#bib.bib18)] | C++/Java/Python | Code Repair | 676/515/716 | All | GPT-4 Generation | ✓ |
| DebugEval | C++/Java/Python |  

&#124; BUG Localization &#124;
&#124; BUG Identification &#124;
&#124; Code Review &#124;
&#124; Code Repair &#124;

 |  

&#124; 195/205/178 &#124;
&#124; 800/760/760 &#124;
&#124; 800/800/800 &#124;
&#124; 138/138/138 &#124;

 | All |  

&#124; User Submission &#124;
&#124; GPT-4 Generation &#124;

 | ✓ | 

Code Review. We first mix the test instances from both DebugBench and LiveCodeBench. Then we collect the test pairs from the mixed dataset by randomly sampling 800 test instances from each programming language. Each test instance consists of the buggy code and the correct code to ask LLMs to choose the buggy one.

Code Repair. For code repair, we first collect 138 newly released programming competition problems from Atcoder from September 1, 2023, to April 1, 2024, which reduces the risk of data leakage. Then we collect buggy code submissions from real users and respectively choose one buggy code of Python, C++, and Java. Lots of buggy codes (82.7%) have the following submissions that correct the buggy codes and pass all test cases, which are regarded as the golden answer of the buggy code. For the rest of these buggy codes (17.3%), we use different LLMs to correct the buggy codes until the corrected code passes all test cases.

Summary. As shown in Table [II](https://arxiv.org/html/2408.05006v1#S3.T2 "TABLE II ‣ III-B Details of Data Construction ‣ III Evaluating the Debugging Ability of LLMs with DebugEval ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), different from other debugging benchmarks, DebugEval is a multi-lingual and multi-task debugging benchmark, which conducts more complete evaluations on the debugging ability of LLMs. Besides, DebugEval contains the buggy codes that are generated by GPT-4 and humans, making the evaluation more convincing, realistic, and authentic.

![Refer to caption](img/b2bff13baa235adb24d80a9c15749f43.png)

Figure 3: Illustration of coMmunicative Agent baSed daTa rEfinement fRamework (MASTER).

## IV CoMmunicative Agent BaSed DaTa REfinement FRamework

Supervised Fine-Tuning (SFT) has been widely adopted to enhance the performance of LLMs in specific domains [[37](https://arxiv.org/html/2408.05006v1#bib.bib37), [57](https://arxiv.org/html/2408.05006v1#bib.bib57)] using human-labeled datasets or LLM-generated data [[15](https://arxiv.org/html/2408.05006v1#bib.bib15)]. However, SFT’s reliance on the availability and quality of labeled data limits its overall effectiveness. In this case, this paper introduces the coMmunicative Agent baSed daTa rEfinement fRamework (MASTER), which automatically refines code debugging data for Supervised Fine-Tuning. As shown in Figure [3](https://arxiv.org/html/2408.05006v1#S3.F3 "Figure 3 ‣ III-B Details of Data Construction ‣ III Evaluating the Debugging Ability of LLMs with DebugEval ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), we give the task examples to LLMs and then prompt LLMs to play different roles to synthesize the SFT data (Sec. [IV-A](https://arxiv.org/html/2408.05006v1#S4.SS1 "IV-A Agent Building ‣ IV CoMmunicative Agent BaSed DaTa REfinement FRamework ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement")). Finally, MASTER employs different agents to refine the synthesized data to guarantee the quality of the SFT data (Sec. [IV-B](https://arxiv.org/html/2408.05006v1#S4.SS2 "IV-B SFT Data Refinement with Communicative Multi-Agents ‣ IV CoMmunicative Agent BaSed DaTa REfinement FRamework ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement")).

### IV-A Agent Building

To conduct the data refinement, MASTER constructs three agents that collaboratively generate and refine the debugging problem data to synthesize high-quality SFT datasets. As illustrated in Figure [4](https://arxiv.org/html/2408.05006v1#S4.F4 "Figure 4 ‣ IV-A Agent Building ‣ IV CoMmunicative Agent BaSed DaTa REfinement FRamework ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), we employ different prompts to guide LLMs to play the roles of Code Quizzer, Code Learner, and Code Teacher. The details of each agent are described below.

Code Quizzer. The Code Quizzer is designed to generate high-quality problems for the SFT data. It uses a stronger LLM as the backbone model and provides the instruction: “You are a code debugging expert, skilled in generating code debugging problems to challenge programmers”. This setup enables the Code Quizzer to generate tailored problems by analyzing examples of debugging tasks. These problems are intended to evaluate the Code Learner’s ability to solve the corresponding debugging tasks.

Code Learner. The Code Learner shares the same backbone model as the SFT model and serves as the critic to evaluate the educational value of the problems generated by the Code Quizzer. Using the prompt: “You are a student, please provide an answer to the following code debugging question using your own knowledge”, the Code Learner is tasked with solving the problem based on its memorized knowledge. The educational value of the problem is then assessed by determining whether the Code Learner can correctly solve it, thereby contributing to the finetuning process.

Code Teacher. Inspired by previous work [[16](https://arxiv.org/html/2408.05006v1#bib.bib16)], we also develop a Code Teacher by prompting the same LLM used for the Code Quizzer with the instruction: “You are an experienced and insightful debugger”. This prompt directs the LLM to act as a proficient code debugger, generating chain-of-thought outcomes [[58](https://arxiv.org/html/2408.05006v1#bib.bib58)] as detailed solutions to the problems, which can better guide LLMs during the SFT process.

![Refer to caption](img/d05a71d8aedb1e084aa077dbb44229d0.png)

Figure 4: Illustrations of Prompts Used in MASTER to Build the Agents. Within MASTER, there are three LLM-based agents Code Quizzer, Code Learner, and Code Teacher. We utilize specific instructions to ensure that they play the correct roles and carry out the intended tasks.

### IV-B SFT Data Refinement with Communicative Multi-Agents

MASTER achieves automated data refinement through multi-agent collaboration, leveraging the expertise of stronger models to enhance the capabilities of weaker ones. The data refinement process consists of three main steps.

In the initial step (Step A), the Code Quizzer synthesizes various code debugging problems based on examples from the debugging task. To ensure diversity in the synthesized data, we instruct the Code Quizzer to generate different debugging problems aligned with the tasks defined in DebugEval, including Bug Localization, Bug Identification, Code Review, and Code Repair. The data synthesis process of each debugging task is guided by a single example, which serves as the demonstrations [[59](https://arxiv.org/html/2408.05006v1#bib.bib59)]. This approach ensures that the synthesized data encompasses a range of error types and difficulty levels, which is crucial for distilling debugging knowledge from the Code Quizzer/Teacher model during SFT Code Learner.

After synthesizing the debugging problems, we proceed to Step B. At this step, the Code Learner attempts to solve the problems provided by the Code Quizzer. In this case, the Code Learner acts as a critic, assessing the educational value of each synthesized problem for the Code Learner. If the Code Learner solves the problem correctly, it indicates that the learner already possesses the necessary knowledge to solve the problem, and thereby the problem is discarded. On the other hand, if the Code Learner provides an incorrect solution, the problem is reserved as the SFT data, due to its educational value for guiding the Code Learner.

Finally, in Step C, the Code Teacher reviews the reserved problems and generates detailed explanations and solutions. These Chain-of-Thought based explanations may include error type identification, error explanations, and the correct solutions to solve the problems. This feedback is essential for the Code Learner to comprehend the problems and refine their solutions. The responses generated by the Code Teacher are treated as the final outputs for the synthesized problems, forming the SFT data to fine-tune our NeuDebugger model.

TABLE III: Data Statistics of Different Supervised Fine-Tuning Strategies. We collect high-quality instruction tuning data from UltraInteract, InstructCoder and RepairLlama to conduct the Vanilla SFT setting.

 | SFT Data | Data Source | #Instance |
| Human/GPT-4 | UltraInteract [[35](https://arxiv.org/html/2408.05006v1#bib.bib35)] | 154,347 |
| InstructCoder [[60](https://arxiv.org/html/2408.05006v1#bib.bib60)] | 6,913 |
| RepairLlama [[61](https://arxiv.org/html/2408.05006v1#bib.bib61)] | 64,643 |
| MASTER | BUG Localization data | 4,681 |
| BUG Identification data | 4,474 |
| Code Review data | 4,420 |
| Code Repair data | 11,317 | 

## V Experimental Methodology

In this section, we describe the Supervised Fine-Tuning (SFT) strategies, evaluation metrics, details of evaluated foundation models, and implementation details of our experiments.

Supervised Fine-Tuning Strategies. As shown in Table [III](https://arxiv.org/html/2408.05006v1#S4.T3 "TABLE III ‣ IV-B SFT Data Refinement with Communicative Multi-Agents ‣ IV CoMmunicative Agent BaSed DaTa REfinement FRamework ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), we describe the experimental details of different supervised fine-tuning strategies, including Vanilla SFT and MASTER. For Vanilla SFT, we collect SFT data from UltraInteract [[35](https://arxiv.org/html/2408.05006v1#bib.bib35)], InstructCoder [[60](https://arxiv.org/html/2408.05006v1#bib.bib60)], and RepairLlama [[61](https://arxiv.org/html/2408.05006v1#bib.bib61)] to finetune LLMs. These datasets are generated by GPT-4 or annotated by humans, which are of high-quality. For MASTER, we synthesize the SFT data by employing multi-agents.

Evaluation Metrics. We first introduce the evaluation metrics used to evaluate different models on DebugEval.

For BUG Localization, BUG Identification, and Code Review tasks, LLMs need to give one answer from multiple choices of the given question. Thus, we use Accuracy as the evaluation metric for these three tasks, which is the same as the previous work [[62](https://arxiv.org/html/2408.05006v1#bib.bib62)]. In particular, for the Code Review task, since the model is asked to choose between two options, we scrambled the order of the options to avoid potential Bias. For each piece of test data, the model can only be considered correct if both orders are answered correctly. For the Code Repair task, we follow previous work [[63](https://arxiv.org/html/2408.05006v1#bib.bib63), [12](https://arxiv.org/html/2408.05006v1#bib.bib12), [11](https://arxiv.org/html/2408.05006v1#bib.bib11), [64](https://arxiv.org/html/2408.05006v1#bib.bib64)] and we use Pass@$k$ [[63](https://arxiv.org/html/2408.05006v1#bib.bib63)] to evaluate the effectiveness of different LLMs:

|  | $\text{Pass@}k:=\underset{\text{Problems}}{\operatorname*{\mathbb{E}}}\left[1-% \frac{\binom{n-c}{k}}{\binom{n}{k}}\right],$ |  | (1) |

where $n$ is the total number of samples, $c$ is the number of correct samples, and $k$ is the number of top samples. In our experiments, we set $k=1$.

TABLE IV: Evaluation Results of Different LLMs on DebugEval. The three tasks, including BUG Localization, BUG Identification, and Code Review, are evaluated using Accuracy. Code Repair is evaluated with Pass$@1$. Here, we use DS to represent the DeepSeek model and PY to denote Python.

 | Model | BUG Localization | BUG Identification | Code Review | Code Repair | Avg. |
| PY | C++ | JAVA | Avg. | PY | C++ | JAVA | Avg. | PY | C++ | JAVA | Avg. | PY | C++ | JAVA | Avg. |
| GPT-4o-mini-0718 [[65](https://arxiv.org/html/2408.05006v1#bib.bib65)] | 84.8 | 81.0 | 81.5 | 82.4 | 53.3 | 48.5 | 48.9 | 50.2 | 85.4 | 90.9 | 91.0 | 89.1 | 65.2 | 67.2 | 67.4 | 66.6 | 72.1 |
| GPT-3.5-Turbo-0125 [[7](https://arxiv.org/html/2408.05006v1#bib.bib7)] | 40.4 | 47.2 | 52.2 | 46.9 | 35.5 | 33.3 | 34.1 | 34.3 | 79.4 | 82.4 | 84.0 | 81.9 | 57.2 | 52.9 | 61.6 | 57.2 | 55.1 |
| DeepSeek-V2-0628 [[66](https://arxiv.org/html/2408.05006v1#bib.bib66)] | 82.0 | 81.0 | 85.9 | 83.0 | 62.0 | 61.0 | 61.3 | 61.4 | 77.4 | 83.9 | 80.5 | 80.6 | 65.2 | 63.0 | 63.5 | 63.9 | 72.2 |
| DeepSeek-Coder-V2-0724 [[67](https://arxiv.org/html/2408.05006v1#bib.bib67)] | 88.8 | 83.1 | 89.8 | 87.2 | 58.7 | 58.9 | 60.8 | 59.4 | 87.9 | 94.9 | 93.3 | 92.0 | 66.7 | 63.1 | 62.3 | 64.0 | 75.7 |
| Llama3-70B-Ins [[68](https://arxiv.org/html/2408.05006v1#bib.bib68)] | 74.2 | 75.9 | 82.0 | 77.5 | 42.8 | 42.3 | 44.9 | 43.3 | 73.9 | 61.6 | 63.3 | 66.3 | 44.9 | 44.2 | 45.7 | 44.9 | 58.0 |
| Qwen2-72B-Ins [[69](https://arxiv.org/html/2408.05006v1#bib.bib69)] | 79.8 | 69.2 | 74.6 | 74.4 | 45.8 | 45.0 | 41.3 | 44.1 | 61.5 | 75.8 | 70.4 | 69.2 | 43.5 | 42.0 | 42.8 | 42.8 | 57.6 |
| DSCoder-33B-Ins [[64](https://arxiv.org/html/2408.05006v1#bib.bib64)] | 52.2 | 50.3 | 51.7 | 51.4 | 24.9 | 26.0 | 30.9 | 27.2 | 24.8 | 27.0 | 30.5 | 27.4 | 46.4 | 50.7 | 54.3 | 50.5 | 39.1 |
| Llama2-7B-Ins [[8](https://arxiv.org/html/2408.05006v1#bib.bib8)] | 18.0 | 20.0 | 22.4 | 20.2 | 24.9 | 27.0 | 25.8 | 25.9 | 2.3 | 0.6 | 2.0 | 1.6 | 4.3 | 11.7 | 19.6 | 11.9 | 14.9 |
| CodeLlama-7B-Ins [[37](https://arxiv.org/html/2408.05006v1#bib.bib37)] | 27.0 | 20.0 | 23.9 | 23.5 | 26.1 | 23.0 | 23.8 | 24.3 | 48.1 | 60.5 | 65.6 | 58.1 | 18.8 | 23.2 | 23.2 | 21.7 | 31.9 |
| CodeQwen1.5-7B-Ins [[36](https://arxiv.org/html/2408.05006v1#bib.bib36)] | 29.2 | 30.8 | 38.0 | 32.9 | 27.6 | 25.9 | 28.8 | 27.4 | 26.9 | 34.4 | 37.1 | 32.8 | 39.1 | 49.3 | 52.9 | 47.1 | 35.1 |
| DeepSeek-LLM-7B-Ins [[70](https://arxiv.org/html/2408.05006v1#bib.bib70)] | 27.0 | 19.0 | 25.9 | 23.9 | 30.5 | 28.5 | 30.9 | 29.9 | 35.9 | 36.6 | 46.0 | 39.5 | 21.0 | 24.1 | 14.5 | 19.9 | 28.3 |
| DSCoder-6.7B-Ins [[64](https://arxiv.org/html/2408.05006v1#bib.bib64)] | 22.5 | 25.6 | 33.7 | 27.5 | 26.6 | 26.0 | 25.9 | 26.2 | 15.5 | 17.8 | 27.8 | 20.3 | 31.9 | 43.5 | 46.4 | 40.6 | 28.7 |
| Llama3-8B-Ins [[68](https://arxiv.org/html/2408.05006v1#bib.bib68)] | 55.6 | 55.9 | 61.0 | 57.6 | 36.8 | 38.1 | 34.6 | 36.6 | 69.1 | 77.4 | 78.1 | 74.9 | 26.1 | 34.3 | 28.3 | 29.6 | 49.7 |
| NeuDebugger-DS-6.7B | 62.4 | 55.4 | 59.0 | 58.8 | 42.6 | 46.9 | 47.8 | 45.8 | 71.0 | 71.4 | 71.4 | 71.3 | 43.5 | 48.6 | 56.5 | 49.5 | 56.4 |
| NeuDebugger-Llama3-8B | 64.6 | 57.9 | 61.0 | 61.1 | 38.6 | 29.9 | 33.3 | 33.8 | 75.3 | 78.0 | 82.4 | 78.5 | 38.4 | 41.3 | 45.7 | 41.8 | 53.8 | 

Foundation Models. We evaluated 13 LLMs on DebugEval, including both closed-source and open-source models.

OpenAI GPTs. GPT-4o-mini-0718 [[65](https://arxiv.org/html/2408.05006v1#bib.bib65)] and GPT-3.5-Turbo-0125 [[7](https://arxiv.org/html/2408.05006v1#bib.bib7)] are two popular and powerful LLMs, which belong to different variants of the GPT family, developed by OpenAI. GPT-4o-mini-0718 is a lightweight version of the GPT-4o model, but it still inherits the core advantages of the GPT-4o, including powerful text generation, logical reasoning, and code generation. Both models are black-box models, which supply commercial APIs for usage.

Meta Llama. Llama2-7B-Ins [[8](https://arxiv.org/html/2408.05006v1#bib.bib8)] is an open-sourced LLM. It is trained with up to 1.4 trillion tokens, where 4.5% of them are code tokens from Github. CodeLlama-7B-Ins [[37](https://arxiv.org/html/2408.05006v1#bib.bib37)] conducts additional instruction-tuning stage to adapt Llama2 [[8](https://arxiv.org/html/2408.05006v1#bib.bib8)] to improve the effectiveness in code-related tasks. Recently, Llama3 [[68](https://arxiv.org/html/2408.05006v1#bib.bib68)] models are released, which is a major leap over Llama2 models and establish a new state-of-the-art.

Aliyun Qwen. Qwen2-72B-Ins is a 72 billion parameter scale LLM. Qwen2-72B employs a variety of automated methods for obtaining high-quality instruction and preference data, making it perform well on code and maths tasks. CodeQwen1.5-7B-Ins [[36](https://arxiv.org/html/2408.05006v1#bib.bib36)] is the code-oriented version of Qwen1.5-7B [[71](https://arxiv.org/html/2408.05006v1#bib.bib71)]. CodeQwen1.5-7B-Ins has been tuned with around 3 trillion tokens of code-related data. It supports 92 programming languages and supports long context understanding and generation with a context length of 64K tokens.

DeepSeek. The DeepSeek series models are released by High-Flyer. DeepSeek-LLM-7B [[70](https://arxiv.org/html/2408.05006v1#bib.bib70)] is trained from scratch with 2 trillion tokens in both English and Chinese. DeepSeek-LLM-7B-Ins [[70](https://arxiv.org/html/2408.05006v1#bib.bib70)] is initialized by DeepSeek-LLM-7B and tuned with an additional 1 million instruction data. DSCoder-6.7B-Ins [[64](https://arxiv.org/html/2408.05006v1#bib.bib64)] and DSCoder-33B-Ins [[64](https://arxiv.org/html/2408.05006v1#bib.bib64)] are trained from scratch on 2T tokens, which consist of 87% code and 13% natural language. DeepSeek-V2-0628 [[66](https://arxiv.org/html/2408.05006v1#bib.bib66)] contains 236B parameters and employs the Mixture-of-Experts (MoE) [[72](https://arxiv.org/html/2408.05006v1#bib.bib72)] architecture to conduct efficient training and inference. It is trained on a high-quality corpus comprising 8.1 trillion tokens. DeepSeek-Coder-V2-0724 [[67](https://arxiv.org/html/2408.05006v1#bib.bib67)] is also an open-sourced MoE based LLM, which achieves comparable performance with GPT4-Turbo in code-related tasks. DeepSeek-Coder-V2 starts from an intermediate checkpoint of DeepSeek-V2 and is tuned using 6 trillion tokens.

Implementation Details. For all LLMs, we set the generation temperature to 0.2 and the maximum generation length to 1024 tokens. For the closed-source models, we use the API endpoints provided by the respective vendors and for the open-source models, we use vLLM [[73](https://arxiv.org/html/2408.05006v1#bib.bib73)] framework for inference. For all tasks in DebugEval, we use the zero-shot setting in our experiments. To finetune the NeuDebugger model, we use DeepSeek-Coder-6.7B-Inst [[64](https://arxiv.org/html/2408.05006v1#bib.bib64)] and Llama3-8B-Inst [[68](https://arxiv.org/html/2408.05006v1#bib.bib68)] as our backbone models and leverage the same data synthesized by MASTER as the SFT data. During SFT, all models are trained with Llama-Factory [[74](https://arxiv.org/html/2408.05006v1#bib.bib74)] and we use LoRA [[75](https://arxiv.org/html/2408.05006v1#bib.bib75)] for efficient fine-tuning. In our experiments, we set the learning rate to 2e-5 and the training epoch to 1\. We optimize the models using the AdamW optimizer, with the batch size of 8 and the gradient accumulation steps of 4.

## VI Evaluation Results

In this section, we benchmark LLMs on DebugEval and evaluate the overall performance of NeuDebugger. Then we conduct ablation studies and discuss the influence of different SFT data amounts on the model performance. The next experiment explores the effectiveness of NeuDebugger in handling the problems of different code error types. Finally, case studies are presented.

TABLE V: The Effectiveness of the MASTER Framework and the Impact of CoT in the Training Data on Model Performance. SFT represents Supervised Fine-Tuning and CoT represents Chain-of-Thought. “Vanilla SFT” indicates that collected existing data was used to train the model. “MASTER (Answer)” indicates that the training data for all four tasks does not include CoT in the output. “MASTER (CoT)” means that the training data for all four tasks include CoT. “NeuDebugger” adopts mixed strategy, means that the other three tasks include CoT, excluding the Code Repair task.

 | Method |  

&#124; BUG &#124;
&#124; Loc. &#124;

 |  

&#124; BUG &#124;
&#124; Iden. &#124;

 |  

&#124; Code &#124;
&#124; Rev. &#124;

 |  

&#124; Code &#124;
&#124; Rep. &#124;

 | Avg. |
| DSCoder-6.7B-Ins |
| zero-shot | 27.5 | 26.2 | 20.3 | 40.6 | 28.7 |
| w/ Vanilla SFT | 21.8 | 23.1 | 9.4 | 40.1 | 23.6 |
| w/ MASTER (Answer) | 43.8 | 35.8 | 32.7 | 43.5 | 39.0 |
| w/ MASTER (CoT) | 60.7 | 45.0 | 34.7 | 38.7 | 44.8 |
| NeuDebugger | 58.8 | 45.8 | 71.3 | 49.5 | 56.4 |
| Llama3-8B-Ins |
| zero-shot | 57.6 | 36.6 | 74.9 | 29.6 | 49.7 |
| w/ Vanilla SFT | 53.6 | 34.0 | 26.0 | 28.7 | 35.6 |
| w/ MASTER (Answer) | 58.1 | 34.8 | 32.1 | 42.5 | 41.9 |
| w/ MASTER (CoT) | 64.4 | 34.6 | 78.1 | 32.6 | 52.4 |
| NeuDebugger | 61.1 | 33.8 | 78.5 | 41.8 | 53.8 | 

TABLE VI: The Performance of Base Model and NeuDebugger on Different Bug Types. Syntax, Ref, Logic, and Multi represent SyntaxError, ReferenceError, LogicError, and MultipleErrors respectively.

 | Task | Model | Python | C++ | Java |
| Syntax | Ref | Logic | Multi | Syntax | Ref | Logic | Multi | Syntax | Ref | Logic | Multi |
| Bug Iden. | Llama3-8B-Ins | 0.0 | 3.7 | 97.9 | 45.8 | 0.0 | 3.0 | 87.5 | 62.0 | 0.0 | 3.7 | 87.4 | 47.4 |
| NeuDebugger-Llama3-8B | 34.2 | 16.8 | 27.9 | 75.3 | 17.0 | 3.0 | 15.0 | 84.5 | 25.3 | 4.2 | 23.2 | 80.5 |
| DSCoder-6.7B-Ins | 3.2 | 3.2 | 99.5 | 0.5 | 2.0 | 0.0 | 100.0 | 2.0 | 1.6 | 1.6 | 100.0 | 0.5 |
| NeuDebugger-DS-6.7B | 54.2 | 81.1 | 32.6 | 2.6 | 45.0 | 45.0 | 77.5 | 20.0 | 50.0 | 55.8 | 72.6 | 12.6 |
| Code Rep. | Llama3-8B-Ins | 40.0 | 20.0 | 35.6 | 7.5 | 60.9 | 45.8 | 29.6 | 16.7 | 40.0 | 52.4 | 28.3 | 7.7 |
| NeuDebugger-Llama3-8B | 90.0 | 46.7 | 39.7 | 20.0 | 65.2 | 66.7 | 40.7 | 10.8 | 80.0 | 76.2 | 39.6 | 15.4 |
| DSCoder-6.7B-Ins | 40.0 | 33.3 | 38.4 | 17.5 | 69.6 | 58.3 | 40.7 | 21.6 | 60.0 | 71.4 | 47.2 | 23.1 |
| NeuDebugger-DS-6.7B | 90.0 | 46.7 | 45.2 | 27.5 | 87.0 | 62.5 | 40.7 | 27.0 | 76.0 | 85.7 | 54.7 | 30.8 | 

### VI-A Overall Performance

The evaluation results of different LLMs and our NeuDebugger on DebugEval are presented in Table [IV](https://arxiv.org/html/2408.05006v1#S5.T4 "TABLE IV ‣ V Experimental Methodology ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"). We compared LLMs of varying scales to assess their code debugging effectiveness.

Overall, larger-scale LLMs exhibit stronger code debugging ability. As indicated by the evaluation results, LLMs exceeding 70B parameters generally demonstrate consistent performance across various debugging tasks. Both the BUG Localization and BUG Identification tasks are multiple-choice questions with four options, where the accuracy of random guessing is approximately 25%. Unfortunately, most 7B-scale LLMs achieve less than 30% accuracy on both tasks. This phenomenon underscores the importance of model scale in maintaining emergent abilities and acquiring critical knowledge through supervised fine-tuning (SFT) on code data [[35](https://arxiv.org/html/2408.05006v1#bib.bib35), [20](https://arxiv.org/html/2408.05006v1#bib.bib20)].

In our experiments, we choose DSCoder-6.7B-Ins and Llama3-8B-Ins as the backbone models and then finetuning these two LLMs using the synthesized data generated by MASTER to conduct NeuDebugger-DS-6.7B and NeuDebugger-Llama3-8B models, respectively. In contrast to other 7B-scale LLMs, our NeuDebugger significantly enhances the code debugging effectiveness of foundation models and achieves competitive performance comparable to the 70B models. This demonstrates that building high-quality SFT data is essential for ensuring the code understanding and code debugging ability of these 7B models. Besides, both NeuDebugger-DS-6.7B and NeuDebugger-Llama3-8B perform better than the foundation model DSCoder-6.7B-Ins and Llama3-8B-Ins on the four tasks in DebugEval, bringing improvements of 27.7% and 4.1%, respectively. These improvements demonstrate that MASTER can refine code debugging data to significantly improve model performance on debugging tasks.

Among the four tasks defined in DebugEval, LLMs typically produce better results in both BUG Localization and Code Review tasks. For example, GPT-4o-mini-0718 achieves accuracy scores of 82.4 and 89.1 on these tasks, respectively. This indicates that these LLMs have strong code understanding capabilities by finetuning with code generation tasks, allowing them to effectively identify buggy code snippets and exhibit better code execution abilities. On the contrary, all LLMs demonstrate less effectiveness in both BUG Identification and Code Repair tasks, which focus more on assessing the code debugging ability of LLMs. For the BUG Identification task, LLMs are required to identify the cause of bugs. The reduced effectiveness of LLMs in this task illustrates the difficulty current LLMs have in deriving bug causes. The Code Repair task is even more complex, requiring LLMs to locate buggy snippets, determine the error type, and then fix the code. The suboptimal performance of these 70B LLMs further indicates the challenges they face in self-debugging [[9](https://arxiv.org/html/2408.05006v1#bib.bib9)]. This phenomenon has also been observed in previous work [[16](https://arxiv.org/html/2408.05006v1#bib.bib16)]. The researchers repair codes by incorporating additional feedback from code compilers, which aims to enhance the bug identification ability of LLMs.

The model performance across various programming languages reveals the effectiveness and robustness of different LLMs. For instance, the GPT-4o-mini-0718 and DeepSeek-Coder-V2-0724 models exhibit consistent performance across all languages, highlighting their robustness in handling diverse tasks. In contrast, some LLMs demonstrate inconsistent performance across different programming languages. For example, DSCoder-6.7B-Ins excels in Java but performs poorly in Python and C++. These findings underscore the necessity of developing a benchmark to evaluate debugging effectiveness across various programming languages, further supporting the motivation of our paper in building the DebugEval benchmark.

![Refer to caption](img/f1fbfbdd74f5acb63f2fa86ab9d087a6.png)

(a) BUG Localization.

![Refer to caption](img/1cc2d06560170d76f6fca10e473953ec.png)

(b) BUG Identification.

![Refer to caption](img/c2ef3b8ead27789c698d21db542fe4e9.png)

(c) Code Review.

![Refer to caption](img/2aef59e640c6f448d23f8c17d0d2cb27.png)

(d) Code Repair.

Figure 5: Impact of the Amount of Training Data on Model Performance. Overall, the performance gradually rises as the amount of training data increase.

![Refer to caption](img/fa050dfc5b47945bfaf20c5e7d0358b9.png)

(a) DSCoder-6.7B-Ins.

![Refer to caption](img/3288ab50938a6c94fae74d046c3efd21.png)

(b) NeuDebugger-DS-6.7B.

Figure 6: The Answer Distribution of DSCoder-6.7B-Ins and NeuDebugger-DS-6.7B in BUG Identification Task.

![Refer to caption](img/a23b7981ee743542aa2152cf47c419c5.png)

Figure 7: Case Studies. We provide two cases from BUG Localization and Code Review to show the effectiveness of NeuDebugger.

### VI-B Ablation Studies

The ablation studies are conducted to explore the effectiveness of the MASTER model in finetuning LLMs.

We compare different SFT strategies, including Vanilla SFT, MASTER (Answer), MASTER (CoT), and NeuDebugger. The Vanilla SFT strategy gathers high-quality SFT data from UltraInteract [[35](https://arxiv.org/html/2408.05006v1#bib.bib35)], InstructCoder [[60](https://arxiv.org/html/2408.05006v1#bib.bib60)], and RepairLlama [[61](https://arxiv.org/html/2408.05006v1#bib.bib61)] for fine-tuning large language models (LLMs). Then, we use the MASTER framework to construct the SFT data and explore three different SFT strategies: MASTER (Answer), MASTER (CoT), and NeuDebugger. MASTER (Answer) indicates that we remove the Code Teacher in MASTER and ask LLMs to directly give the correct choice during SFT. MASTER (CoT) asks the Code Student to mimic the thought of problem solving of the Code Teacher. The NeuDebugger method combines the SFT strategies from both MASTER (Answer) and MASTER (CoT) by mixing the datasets from both SFT strategies for finetuning backbone models, except for the code repair data of MASTER (CoT).

As shown in Table [V](https://arxiv.org/html/2408.05006v1#S6.T5 "TABLE V ‣ VI Evaluation Results ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), both DSCoder-6.7B-Ins and Llama3-8B-Ins perform worse with the Vanilla SFT method compared to the baseline, indicating that the data quality for SFT remains a challenge in fine-tuning LLMs. When the MASTER synthesized data is used for SFT, the code debugging ability of these models is significantly enhanced. This illustrates that both DSCoder-6.7B-Ins and Llama3-8B-Ins are less effective at learning debugging knowledge from human/GPT-4 annotated data [[20](https://arxiv.org/html/2408.05006v1#bib.bib20)]. Furthermore, the MASTER (CoT) method generally achieves much better performance than MASTER (Answer), except for the code repair task. This may be because the Chain-of-Thought outcomes generated by Code Teacher can better explain the reasons behind answer choices but might incorporate additional noise in code repair tasks. By combining SFT data from both MASTER (CoT) and MASTER (Answer), NeuDebugger achieves the best performance among all SFT strategies. All these experimental results demonstrate the effectiveness of the MASTER model, which employs multi-agents to synthesize and refine SFT data.

### VI-C The Impact of Data Quantity

This subsection explores the impact of data quantity when finetuning LLMs using the synthesized data from MASTER. As shown in Figure [5](https://arxiv.org/html/2408.05006v1#S6.F5 "Figure 5 ‣ VI-A Overall Performance ‣ VI Evaluation Results ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), we fine-tuned the DSCoder-6.7B-Ins and Llama3-8B-Ins models using varying amounts of SFT data points. We then assessed their debugging abilities by evaluating their performance on the DebugEval benchmark and visualizing the results.

Compared to Llama3-8B-Ins, DSCoder-6.7B-Ins shows a significant performance decrease when more SFT data points are fed. This indicates that code-oriented LLMs are better at learning from debugging data, whereas a standard language model struggles to enhance its debugging capabilities without an essential understanding of code. Among all debugging tasks, DSCoder-6.7B-Ins exhibits significant improvements in BUG Localization, BUG Identification, and Code Review, while only showing slight improvements in the code repair task. This suggests that these debugging tasks do indeed contribute to the better code repair ability of LLMs, though the task remains challenging to improve significantly.

### VI-D Effectiveness of NeuDebugger on Different Bug Types

As shown in Table [VI](https://arxiv.org/html/2408.05006v1#S6.T6 "TABLE VI ‣ VI Evaluation Results ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement"), we show the effectiveness of NeuDebugger on more difficult tasks, including Bug Identification and Code Repair. The evaluation results on different bug types are also shown.

For the BUG Identification task, the evaluation results demonstrate that existing LLMs are still suboptimal for debugging tasks. Both DSCoder-6.7B-Ins and Llama3-8B-Ins achieve over 97% accuracy on logic errors, but perform poorly on other coder error types. Furthermore, Figure [6](https://arxiv.org/html/2408.05006v1#S6.F6 "Figure 6 ‣ VI-A Overall Performance ‣ VI Evaluation Results ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement") illustrates the answer distribution of LLMs. The evaluation results indicate that DSCoder-6.7B-Ins lacks the ability to identify bugs and defaults to selecting logic errors, resulting in high accuracy for this specific error type. NeuDebugger shows its effectiveness by conducting a more balanced answer choice distribution and achieving significant improvements on the Bug Identification task.

For the Code Repair task, we observe that NeuDebugger achieves improvements across almost all types of code errors, especially for syntax errors, showing its effectiveness for code debugging. This also illustrates that syntax errors are relatively simpler and easier for the model to learn when we compare them to the other three types of code errors. Besides, NeuDebugger also struggles with repairing code containing logical or multiple errors.

### VI-E Case Studies

Finally, we show two cases in Figure [7](https://arxiv.org/html/2408.05006v1#S6.F7 "Figure 7 ‣ VI-A Overall Performance ‣ VI Evaluation Results ‣ Enhancing the Code Debugging Ability of LLMs via Communicative Agent Based Data Refinement") to demonstrate the effectiveness of NeuDebugger. NeuDebugger is trained on the code debugging data constructed by our proposed MASTER framework. And we compare the performance of the model before and after training through cases.

For the first case of the BUG Localization task, the code error is caused by the line f = min(f, Cost(s[:x]) + A(s[x:], k-1’)), which has an incorrect index k-1’. Thus, the correct answer is (C). DSCoder-6.7B-Ins considers the code fragment if s[i]!=s[j]: c+=1 as erroneous, stating “This is not the correct way to check if a string is a palindrome”. On the contrary, NeuDebugger-DS-6.7B accurately analyzes the reason of bugs “contains a reference error in the line f = min(f, Cost(s[:x]) + A(s[x:], k-1’)), the error is in the reference to k-1’, which should be k-1”, demonstrating its effectiveness in BUG Localization.

In the second case of the Code Repair task, the error involves the misuse of continue, which leads to a logical mistake. The DSCoder-6.7B-Ins model fails to identify this error and instead suggests changing the line String s = jc.next().toLowerCase() to String s = jc.nextLine().toLowerCase(). This modification introduces a new error, as it does not handle input correctly. NeuDebugger-DS-6.7B accurately recognizes that the problem lies in the use of continue and changes continue to break, successfully resolving the bug.

## VII Conclusion

This paper presents DebugEval, an innovative benchmark designed to assess the debugging capabilities of Large Language Models (LLMs) from multiple perspectives. We introduce MASTER, a method that utilizes LLMs to generate high-quality supervised fine-tuning (SFT) datasets specifically for debugging tasks, thereby improving the performance of smaller models. Our experiments indicate that LLMs with 7B parameters are less effective in these debugging tasks and MASTER effectively enhances code debugging capabilities by refining data for SFT.

## References

*   [1] B. Hailpern and P. Santhanam, “Software debugging, testing, and verification,” *IBM Systems Journal*, vol. 41, no. 1, pp. 4–12, 2002.
*   [2] L. Kirschner, E. Soremekun, and A. Zeller, “Debugging inputs,” in *Proceedings of the ACM/IEEE 42nd International Conference on Software Engineering*, 2020, pp. 75–86.
*   [3] C. Le Goues, T. Nguyen, S. Forrest, and W. Weimer, “Genprog: A generic method for automatic software repair,” *IEEE Transactions on Software Engineering*, p. 54–72, 2012.
*   [4] M. Wen, J. Chen, R. Wu, D. Hao, and S.-C. Cheung, “Context-aware patch generation for better automated program repair,” in *Proceedings of the 40th International Conference on Software Engineering*, 2018.
*   [5] J. Hua, M. Zhang, K. Wang, and S. Khurshid, “Sketchfix: a tool for automated program repair approach using lazy candidate generation,” in *Proceedings of the 2018 26th ACM Joint Meeting on European Software Engineering Conference and Symposium on the Foundations of Software Engineering*, 2018.
*   [6] K. Liu, A. Koyuncu, D. Kim, and T. F. Bissyandé, “Tbar: revisiting template-based automated program repair,” in *Proceedings of the 28th ACM SIGSOFT International Symposium on Software Testing and Analysis*, 2019.
*   [7] OpenAI. (2022) Chatgpt: Optimizing language models for dialogue.
*   [8] H. Touvron, L. Martin, K. Stone, P. Albert, A. Almahairi, Y. Babaei, N. Bashlykov, S. Batra, P. Bhargava, S. Bhosale *et al.*, “Llama 2: Open foundation and fine-tuned chat models,” *ArXiv preprint*, vol. abs/2307.09288, 2023.
*   [9] X. Chen, M. Lin, N. Schärli, and D. Zhou, “Teaching large language models to self-debug,” *ArXiv preprint*, vol. abs/2304.05128, 2023.
*   [10] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. d. O. Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman *et al.*, “Evaluating large language models trained on code,” *ArXiv preprint*, vol. abs/2107.03374, 2021.
*   [11] Z. Luo, C. Xu, P. Zhao, Q. Sun, X. Geng, W. Hu, C. Tao, J. Ma, Q. Lin, and D. Jiang, “Wizardcoder: Empowering code large language models with evol-instruct,” *ArXiv preprint*, vol. abs/2306.08568, 2023.
*   [12] Q. Zheng, X. Xia, X. Zou, Y. Dong, S. Wang, Y. Xue, L. Shen, Z. Wang, A. Wang, Y. Li *et al.*, “Codegeex: A pre-trained model for code generation with multilingual benchmarking on humaneval-x,” in *Proceedings of the 29th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*, 2023, pp. 5673–5684.
*   [13] R. Li, L. B. Allal, Y. Zi, N. Muennighoff, D. Kocetkov, C. Mou, M. Marone, C. Akiki, J. Li, J. Chim *et al.*, “Starcoder: May the source be with you!” *ArXiv preprint*, vol. abs/2305.06161, 2023.
*   [14] D. Guo, Q. Zhu, D. Yang, Z. Xie, K. Dong, W. Zhang, G. Chen, X. Bi, Y. Wu, Y. Li *et al.*, “Deepseek-coder: When the large language model meets programming–the rise of code intelligence,” *ArXiv preprint*, vol. abs/2401.14196, 2024.
*   [15] K. Zhang, Z. Li, J. Li, G. Li, and Z. Jin, “Self-edit: Fault-aware code editor for code generation,” *ArXiv preprint*, vol. abs/2305.04087, 2023.
*   [16] H. Wang, Z. Liu, S. Wang, G. Cui, N. Ding, Z. Liu, and G. Yu, “Intervenor: Prompt the coding ability of large language models with the interactive chain of repair,” in *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics*, 2024.
*   [17] R. Tian, Y. Ye, Y. Qin, X. Cong, Y. Lin, Z. Liu, and M. Sun, “Debugbench: Evaluating debugging capability of large language models,” 2024.
*   [18] J. Guo, Z. Li, X. Liu, K. Ma, T. Zheng, Z. Yu, D. Pan, Y. LI, R. Liu, Y. Wang, S. Guo, X. Qu, X. Yue, G. Zhang, W. Chen, and J. Fu, “Codeeditorbench: Evaluating code editing capability of large language models,” 2024.
*   [19] J. Achiam, S. Adler, S. Agarwal, L. Ahmad, I. Akkaya, F. L. Aleman, D. Almeida, J. Altenschmidt, S. Altman, S. Anadkat *et al.*, “Gpt-4 technical report,” *ArXiv preprint*, vol. abs/2303.08774, 2023.
*   [20] A. Gudibande, E. Wallace, C. V. Snell, X. Geng, H. Liu, P. Abbeel, S. Levine, and D. Song, “The false promise of imitating proprietary language models,” in *The Twelfth International Conference on Learning Representations*, 2024.
*   [21] J. Wei, X. Wang, D. Schuurmans, M. Bosma, B. Ichter, F. Xia, E. Chi, Q. Le, and D. Zhou, “Chain-of-thought prompting elicits reasoning in large language models,” 2022.
*   [22] Z. Feng, D. Guo, D. Tang, N. Duan, X. Feng, M. Gong, L. Shou, B. Qin, T. Liu, D. Jiang, and M. Zhou, “CodeBERT: A pre-trained model for programming and natural languages,” in *Findings of the Association for Computational Linguistics: EMNLP 2020*.   Online: Association for Computational Linguistics, 2020, pp. 1536–1547.
*   [23] W. Ahmad, S. Chakraborty, B. Ray, and K.-W. Chang, “Unified pre-training for program understanding and generation,” in *Proceedings of the 2021 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies*.   Online: Association for Computational Linguistics, 2021, pp. 2655–2668.
*   [24] D. Zan, B. Chen, D. Yang, Z. Lin, M. Kim, B. Guan, Y. Wang, W. Chen, and J.-G. Lou, “Cert: Continual pre-training on sketches for library-oriented code generation,” in *International Joint Conference on Artificial Intelligence*, 2022.
*   [25] K. Clark, M. Luong, Q. V. Le, and C. D. Manning, “ELECTRA: pre-training text encoders as discriminators rather than generators,” in *8th International Conference on Learning Representations, ICLR 2020, Addis Ababa, Ethiopia, April 26-30, 2020*.   OpenReview.net, 2020.
*   [26] S. Lu, D. Guo, S. Ren, J. Huang, A. Svyatkovskiy, A. Blanco, C. B. Clement, D. Drain, D. Jiang, D. Tang, G. Li, L. Zhou, L. Shou, L. Zhou, M. Tufano, M. Gong, M. Zhou, N. Duan, N. Sundaresan, S. K. Deng, S. Fu, and S. Liu, “Codexglue: A machine learning benchmark dataset for code understanding and generation,” in *Proceedings of NeurIPS*, 2021.
*   [27] M. Lachaux, B. Rozière, M. Szafraniec, and G. Lample, “DOBF: A deobfuscation pre-training objective for programming languages,” in *Advances in Neural Information Processing Systems 34: Annual Conference on Neural Information Processing Systems 2021, NeurIPS 2021, December 6-14, 2021, virtual*, 2021, pp. 14 967–14 979.
*   [28] Y. Wang, W. Wang, S. Joty, and S. C. Hoi, “CodeT5: Identifier-aware unified pre-trained encoder-decoder models for code understanding and generation,” in *Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing*.   Online and Punta Cana, Dominican Republic: Association for Computational Linguistics, 2021, pp. 8696–8708.
*   [29] C. Raffel, N. Shazeer, A. Roberts, K. Lee, S. Narang, M. Matena, Y. Zhou, W. Li, and P. J. Liu, “Exploring the limits of transfer learning with a unified text-to-text transformer,” *J. Mach. Learn. Res.*, vol. 21, pp. 140:1–140:67, 2020.
*   [30] X. Li, Y. Gong, Y. Shen, X. Qiu, H. Zhang, B. Yao, W. Qi, D. Jiang, W. Chen, and N. Duan, “CodeRetriever: A large scale contrastive pre-training method for code search,” in *Proceedings of the 2022 Conference on Empirical Methods in Natural Language Processing*.   Abu Dhabi, United Arab Emirates: Association for Computational Linguistics, 2022, pp. 2898–2910.
*   [31] D. Guo, S. Lu, N. Duan, Y. Wang, M. Zhou, and J. Yin, “UniXcoder: Unified cross-modal pre-training for code representation,” in *Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*.   Dublin, Ireland: Association for Computational Linguistics, 2022, pp. 7212–7225.
*   [32] Y. MA, Y. Liu, Y. Yu, Y. Zhang, Y. Jiang, C. Wang, and S. Li, “At which training stage does code data help llms reasoning?” in *The Twelfth International Conference on Learning Representations*.
*   [33] P. Liang, R. Bommasani, T. Lee, D. Tsipras, D. Soylu, M. Yasunaga, Y. Zhang, D. Narayanan, Y. Wu, A. Kumar *et al.*, “Holistic evaluation of language models,” *Transactions on Machine Learning Research*.
*   [34] Y. Fu, H. Peng, and T. Khot, “How does gpt obtain its ability? tracing emergent abilities of language models to their sources,” *Yao Fu’s Notion*, 2022.
*   [35] L. Yuan, G. Cui, H. Wang, N. Ding, X. Wang, J. Deng, B. Shan, H. Chen, R. Xie, Y. Lin *et al.*, “Advancing llm reasoning generalists with preference trees,” *ArXiv preprint*, vol. abs/2404.02078, 2024.
*   [36] J. Bai, S. Bai, Y. Chu, Z. Cui, K. Dang, X. Deng, Y. Fan, W. Ge, Y. Han, F. Huang, B. Hui, L. Ji, M. Li, J. Lin, R. Lin, D. Liu, G. Liu, C. Lu, K. Lu, J. Ma, R. Men, X. Ren, X. Ren, C. Tan, S. Tan, J. Tu, P. Wang, S. Wang, W. Wang, S. Wu, B. Xu, J. Xu, A. Yang, H. Yang, J. Yang, S. Yang, Y. Yao, B. Yu, H. Yuan, Z. Yuan, J. Zhang, X. Zhang, Y. Zhang, Z. Zhang, C. Zhou, J. Zhou, X. Zhou, and T. Zhu, “Qwen technical report,” *ArXiv preprint*, vol. abs/2309.16609, 2023.
*   [37] B. Roziere, J. Gehring, F. Gloeckle, S. Sootla, I. Gat, X. E. Tan, Y. Adi, J. Liu, T. Remez, J. Rapin *et al.*, “Code llama: Open foundation models for code,” *ArXiv preprint*, vol. abs/2308.12950, 2023.
*   [38] S. Mechtaev, J. Yi, and A. Roychoudhury, “Angelix,” in *Proceedings of the 38th International Conference on Software Engineering*, 2016.
*   [39] F. DeMarco, J. Xuan, D. Le Berre, and M. Monperrus, “Automatic repair of buggy if conditions and missing preconditions with smt,” in *Proceedings of the 6th International Workshop on Constraints in Software Testing, Verification, and Analysis*, 2014.
*   [40] C. S. Xia, Y. Wei, and L. Zhang, “Practical program repair in the era of large pre-trained language models,” *ArXiv preprint*, vol. abs/2210.14179, 2022.
*   [41] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. de Oliveira Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, A. Ray, R. Puri, G. Krueger, M. Petrov, H. Khlaaf, G. Sastry, P. Mishkin, B. Chan, S. Gray, N. Ryder, M. Pavlov, A. Power, L. Kaiser, M. Bavarian, C. Winter, P. Tillet, F. P. Such, D. Cummings, M. Plappert, F. Chantzis, E. Barnes, A. Herbert-Voss, W. H. Guss, A. Nichol, A. Paino, N. Tezak, J. Tang, I. Babuschkin, S. Balaji, S. Jain, W. Saunders, C. Hesse, A. N. Carr, J. Leike, J. Achiam, V. Misra, E. Morikawa, A. Radford, M. Knight, M. Brundage, M. Murati, K. Mayer, P. Welinder, B. McGrew, D. Amodei, S. McCandlish, I. Sutskever, and W. Zaremba, “Evaluating large language models trained on code,” 2021.
*   [42] S. D. Kolak, R. Martins, C. Le Goues, and V. J. Hellendoorn, “Patch generation with language models: Feasibility and scaling behavior,” in *Deep Learning for Code Workshop*, 2022.
*   [43] A. Radford, J. Wu, R. Child, D. Luan, D. Amodei, and I. Sutskever, “Language models are unsupervised multitask learners,” 2019.
*   [44] T. X. Olausson, J. P. Inala, C. Wang, J. Gao, and A. Solar-Lezama, “Is self-repair a silver bullet for code generation?” in *The Twelfth International Conference on Learning Representations*, 2023.
*   [45] K. Li, Q. Hu, X. Zhao, H. Chen, Y. Xie, T. Liu, Q. Xie, and J. He, “Instructcoder: Instruction tuning large language models for code editing,” 2023.
*   [46] Y. Wang, Y. Kordi, S. Mishra, A. Liu, N. A. Smith, D. Khashabi, and H. Hajishirzi, “Self-instruct: Aligning language models with self-generated instructions,” 2022.
*   [47] G. Li, C. Zhi, J. Chen, J. Han, and S. Deng, “A comprehensive evaluation of parameter-efficient fine-tuning on automated program repair,” 2024.
*   [48] E. J. Hu, Y. Shen, P. Wallis, Z. Allen-Zhu, Y. Li, S. Wang, L. Wang, and W. Chen, “Lora: Low-rank adaptation of large language models,” in *The Tenth International Conference on Learning Representations, ICLR 2022, Virtual Event, April 25-29, 2022*.   OpenReview.net, 2022.
*   [49] X. L. Li and P. Liang, “Prefix-tuning: Optimizing continuous prompts for generation,” in *Proceedings of the 59th Annual Meeting of the Association for Computational Linguistics and the 11th International Joint Conference on Natural Language Processing (Volume 1: Long Papers)*.   Online: Association for Computational Linguistics, 2021, pp. 4582–4597.
*   [50] X. Liu, K. Ji, Y. Fu, W. Tam, Z. Du, Z. Yang, and J. Tang, “P-tuning v2: Prompt tuning can be comparable to fine-tuning universally across scales and tasks,” *Cornell University - arXiv,Cornell University - arXiv*, 2021.
*   [51] H. Liu, D. Tam, M. Muqeeth, J. Mohta, T. Huang, M. Bansal, and C. Raffel, “Few-shot parameter-efficient fine-tuning is better and cheaper than in-context learning.”
*   [52] N. Jain, K. Han, A. Gu, W.-D. Li, F. Yan, T. Zhang, S. Wang, A. Solar-Lezama, K. Sen, and I. Stoica, “Livecodebench: Holistic and contamination free evaluation of large language models for code,” *ArXiv preprint*, vol. abs/2403.07974, 2024.
*   [53] M. Yasunaga and P. Liang, “Break-it-fix-it: Unsupervised learning for program repair,” in *Proceedings of the 38th International Conference on Machine Learning, ICML 2021, 18-24 July 2021, Virtual Event*, ser. Proceedings of Machine Learning Research, vol. 139.   PMLR, 2021, pp. 11 941–11 952.
*   [54] F. Huq, M. Hasan, M. M. A. Haque, S. Mahbub, A. Iqbal, and T. Ahmed, “Review4repair: Code review aided automatic program repairing,” *Information and Software Technology*, p. 106765, 2022.
*   [55] S. Lu, D. Guo, S. Ren, J. Huang, A. Svyatkovskiy, A. Blanco, C. Clement, D. Drain, D. Jiang, D. Tang *et al.*, “Codexglue: A machine learning benchmark dataset for code understanding and generation,” *ArXiv preprint*, vol. abs/2102.04664, 2021.
*   [56] M. M. A. Haque, W. U. Ahmad, I. Lourentzou, and C. Brown, “Fixeval: Execution-based evaluation of program fixes for programming problems,” in *2023 IEEE/ACM International Workshop on Automated Program Repair (APR)*.   IEEE, 2023, pp. 11–18.
*   [57] S. Yue, W. Chen, S. Wang, B. Li, C. Shen, S. Liu, Y. Zhou, Y. Xiao, S. Yun, W. Lin *et al.*, “Disc-lawllm: Fine-tuning large language models for intelligent legal services,” *ArXiv preprint*, vol. abs/2309.11325, 2023.
*   [58] J. Wei, X. Wang, D. Schuurmans, M. Bosma, F. Xia, E. Chi, Q. V. Le, D. Zhou *et al.*, “Chain-of-thought prompting elicits reasoning in large language models,” *Advances in neural information processing systems*, vol. 35, pp. 24 824–24 837, 2022.
*   [59] T. B. Brown, B. Mann, N. Ryder, M. Subbiah, J. Kaplan, P. Dhariwal, A. Neelakantan, P. Shyam, G. Sastry, A. Askell, S. Agarwal, A. Herbert-Voss, G. Krueger, T. Henighan, R. Child, A. Ramesh, D. M. Ziegler, J. Wu, C. Winter, C. Hesse, M. Chen, E. Sigler, M. Litwin, S. Gray, B. Chess, J. Clark, C. Berner, S. McCandlish, A. Radford, I. Sutskever, and D. Amodei, “Language models are few-shot learners,” in *Advances in Neural Information Processing Systems 33: Annual Conference on Neural Information Processing Systems 2020, NeurIPS 2020, December 6-12, 2020, virtual*, 2020.
*   [60] Q. Hu, K. Li, X. Zhao, Y. Xie, T. Liu, H. Chen, Q. Xie, and J. He, “Instructcoder: Empowering language models for code editing,” *ArXiv preprint*, vol. abs/2310.20329, 2023.
*   [61] A. Silva, S. Fang, and M. Monperrus, “Repairllama: Efficient representations and fine-tuned adapters for program repair,” Tech. Rep., 2023.
*   [62] M. Suzgun, N. Scales, N. Schärli, S. Gehrmann, Y. Tay, H. W. Chung, A. Chowdhery, Q. V. Le, E. H. Chi, D. Zhou, and J. Wei, “Challenging big-bench tasks and whether chain-of-thought can solve them,” 2022.
*   [63] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. D. O. Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman *et al.*, “Evaluating large language models trained on code,” *ArXiv preprint*, vol. abs/2107.03374, 2021.
*   [64] D. Guo, Q. Zhu, D. Yang, Z. Xie, K. Dong, W. Zhang, G. Chen, X. Bi, Y. Wu, Y. K. Li, F. Luo, Y. Xiong, and W. Liang, “Deepseek-coder: When the large language model meets programming – the rise of code intelligence,” 2024.
*   [65] OpenAI, “Gpt-4o mini: advancing cost-efficient intelligence,” 2024.
*   [66] DeepSeek-AI, “Deepseek-v2: A strong, economical, and efficient mixture-of-experts language model,” 2024.
*   [67] Q. Zhu, D. Guo, Z. Shao, D. Yang, P. Wang, R. Xu, Y. Wu, Y. Li, H. Gao, S. Ma *et al.*, “Deepseek-coder-v2: Breaking the barrier of closed-source models in code intelligence,” *ArXiv preprint*, vol. abs/2406.11931, 2024.
*   [68] AI@Meta, “Llama 3 model card,” 2024.
*   [69] “Qwen2 technical report,” 2024.
*   [70] DeepSeek-AI, “Deepseek llm: Scaling open-source language models with longtermism,” *ArXiv preprint*, vol. abs/2401.02954, 2024.
*   [71] Q. Team, “Introducing qwen1.5,” 2024.
*   [72] W. Cai, J. Jiang, F. Wang, J. Tang, S. Kim, and J. Huang, “A survey on mixture of experts,” 2024.
*   [73] W. Kwon, Z. Li, S. Zhuang, Y. Sheng, L. Zheng, C. H. Yu, J. E. Gonzalez, H. Zhang, and I. Stoica, “Efficient memory management for large language model serving with pagedattention,” in *Proceedings of the ACM SIGOPS 29th Symposium on Operating Systems Principles*, 2023.
*   [74] hiyouga, “Llama factory,” 2023.
*   [75] E. J. Hu, Y. Shen, P. Wallis, Z. Allen-Zhu, Y. Li, S. Wang, L. Wang, and W. Chen, “Lora: Low-rank adaptation of large language models,” in *The Tenth International Conference on Learning Representations, ICLR 2022, Virtual Event, April 25-29, 2022*.   OpenReview.net, 2022.