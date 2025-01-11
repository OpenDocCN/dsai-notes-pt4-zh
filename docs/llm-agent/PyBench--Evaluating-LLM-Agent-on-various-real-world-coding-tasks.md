<!--yml
category: 未分类
date: 2025-01-11 12:23:59
-->

# PyBench: Evaluating LLM Agent on various real-world coding tasks

> 来源：[https://arxiv.org/html/2407.16732/](https://arxiv.org/html/2407.16732/)

Yaolun Zhang¹  , Yinxu Pan²¹¹footnotemark: 1 , Yudong Wang²¹¹footnotemark: 1 , Jie Cai² 
¹School of Statistics, Renmin University of China
²ModelBest Inc.
{zhangyaolun5}@ruc.edu.cn
{panyinxu, wangyudong, caijie}@modelbest.cn   Equal contribution.  Work done during internship at ModelBest Inc.

###### Abstract

The LLM Agent, equipped with a code interpreter, can automatically solve real-world coding tasks such as data analysis and image editing. However, existing benchmarks primarily focus on either simplistic tasks, such as completing a few lines of code, or on extremely complex and specific tasks at the repository level, neither of which are representative of various daily coding tasks. To address this gap, we introduce PyBench, a benchmark encompassing five main categories of real-world tasks, covering more than 10 types of files. Given a high-level user query and related files, the LLM Agent needs to reason and execute Python code via a code interpreter for a few turns before making a formal response to fulfill the user’s requirements. Successfully addressing tasks in PyBench demands a robust understanding of various Python packages, superior reasoning capabilities, and the ability to incorporate feedback from executed code. Our evaluations indicate that current open-source LLMs are struggling with these tasks. Hence, we conduct analysis and experiments on four kinds of datasets, proving that comprehensive abilities are needed for PyBench. Our fine-tuned 8B size model: PyLlama3 achieves an exciting performance on PyBench which surpasses many 33B and 70B size models. Our Benchmark, Training Dataset, and Model are available at: [https://github.com/Mercury7353/PyBench](https://github.com/Mercury7353/PyBench)

## 1 Introduction

![Refer to caption](img/c9965d41857fbca684a362dc95ad83f2.png)

Figure 1: An Overview of LLMs’ performance on PyBench

The best tool is the one that gets the job done. Enormous real-world tasks like data analysis and image & audio processing can be solved by code. Among many programming languages, Python stands out for its simplicity, ease of use, extensive libraries, and high compatibility, making it a widely used tool for daily tasks. However, individuals often need to invest significant time in learning how to use extension packages, even if they are already highly proficient in Python itself. Thanks to LLM’s powerful code capabilities, it can act as an automatic agent Wang et al. ([2024a](https://arxiv.org/html/2407.16732v2#bib.bib32)); Park et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib26)); Qin et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib28)) writing and executing code to solve a wide spectrum of real-world tasks.

However, current LLM code benchmarks have not covered the real-world task area. HumanEval Chen et al. ([2021](https://arxiv.org/html/2407.16732v2#bib.bib4)) and MBPP Austin et al. ([2021](https://arxiv.org/html/2407.16732v2#bib.bib1)) focus on function complement and simple Python problems, which could not evaluate the usefulness of LLM Agent.

Although benchmarks such as DS-1000 Lai et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib18)), DevBench Li et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib19)), and SWE-Bench Jimenez et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib16)) focus on repository-level coding issues, they assess the ability of LLMs to use and manage specific codebases. These tasks are relatively narrow in scope and inherently limited, deviating from practical daily application scenarios and being overly complex for routine use.

To address the lack of benchmarks for real-world coding tasks, we introduce PyBench, a comprehensive and versatile benchmark designed to evaluate the practical coding abilities of LLMs. Specifically, we formulated real-world coding tasks into 5 main categories: Chart Analysis, Text Analysis, Image & Audio Editing, Complex Math, and Software & Website Development. Each category consists of comprehensive subclasses of tasks, reflecting real-world situations. Related files are collected for each subclass, and queries are tailored for the content of files. For the quantitative evaluation of PyBench tasks, we create a set of unit tests to verify whether the tasks are solved successfully. We also employ GPT-4 as a judge to evaluate the solutions and calculate the Average Turns as a measure of problem-solving efficiency. We evaluate 3 types of models on PyBench: Closed-source LLMs, 70B, 33B, and 7B size Open-source LLMs, and Code LLMs specifically tailored for coding tasks. The evaluation results indicate most LLMs struggle to solve PyBench tasks.

Consequently, we collect and synthesize four datasets: the homologous training dataset, multi-turn code interaction dataset, multi-turn chat dataset, and code-rich corpus for continue pre-training. We then conduct a series of analyses and experiments to figure out what are necessary abilities to solve the PyBench tasks and how to improve the LLM performance on real-world coding tasks. The result demonstrates that merely homologous data cannot help the base model adapt to real-world coding tasks. The multi-turn code interaction dataset significantly enhances comprehensive capabilities. Specifically, the multi-turn chat dataset boosts performance in Chart Analysis, while continued pre-training on a code-rich corpus improves Text Analysis. Trained on these datasets, our fine-tuned 8B size model PyLlama3 surpasses Llama3-8B-Instruct on PyBench.

In summary, our contributions are as follows:

*   •

    We construct PyBench, the first comprehensive benchmark for evaluating LLM Agents on real-world coding tasks. PyBench includes real-world files and related queries, covering a wide range of daily situations and file types.

*   •

    We present PyBench tasks that necessitate the agent’s comprehensive capabilities. Using only homologous data is inadequate for adapting base models to real-world coding tasks. The agent’s ability for multi-turn interaction is crucial, and continued pre-training on a code-rich corpus is also beneficial.

*   •

    We conduct continued pre-training of Llama3-8B-Base on a code-rich corpus and fine-tune it on homologous, multi-turn code, and multi-turn chat datasets. Our PyLlama3 model achieves outstanding performance on PyBench.

| Category | Subclass | Relative Python Packages |
| --- | --- | --- |
| Chart Analysis | Data Preprocessing, Data Visualization, Machine Learning… | pandas, numpy, sklearn, matplotlib… |
| Text Analysis | Text Based QA, Theme Analysis, Wordcloud… | jieba, wordcloud, PyPDF2… |
| Image & Audio Editing | Image Generation, Sound Feature Extraction… | opencv, PIL, pydub… |
| Complex Math | Large Number Calculation, Calculus… | numpy, scipy… |
| Software & Website Development | Game Design, Website Design… | pygame, bs4… |

Table 1: Type of tasks and related Python packages of PyBench. We only select commonly used Python packages for these tasks, though other Python packages may also be used for specific tasks.

## 2 Related Works

### 2.1 Benchmark on coding ability

Many existing benchmarks focus on LLM’s code ability. HumanEval Chen et al. ([2021](https://arxiv.org/html/2407.16732v2#bib.bib4)) and MBPP Austin et al. ([2021](https://arxiv.org/html/2407.16732v2#bib.bib1)) are two widely recognized benchmarks primarily evaluating LLM’s ability to complete functions or solve simple Python problems. HumanEval-X, HumanEval+, and MBPP+ Zheng et al. ([2024a](https://arxiv.org/html/2407.16732v2#bib.bib42)); Liu et al. ([2023a](https://arxiv.org/html/2407.16732v2#bib.bib21)) extend the benchmarks by adding multilingual and plenty of extra tests. APPS Hendrycks et al. ([2021](https://arxiv.org/html/2407.16732v2#bib.bib12)) focuses on writing code from natural language description. TACO Li et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib20)) builds a more complex benchmark evaluating LLM on algorithmic code tasks. XCodeEval Khan et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib17)) and CodeScope Yan et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib38)) provide execution-based multilingual code benchmarks but without real interaction with files. MINT Wang et al. ([2023a](https://arxiv.org/html/2407.16732v2#bib.bib34)) and $M^{3}$ToolEval Wang et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib33)) aim to evaluate models’ multi-turn interaction code ability with tools or human feedback. There are also extremely complex and hard benchmarks evaluating LLMs on software development Qian et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib27)); Hong et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib14)), code repository issues Jimenez et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib16)); Li et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib19)); Zhang et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib41)); Du et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib9)), and data science tasksLai et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib18)). However, these benchmarks are all limited to specific scenarios. To the best of our knowledge, no existing benchmark evaluates LLM Agent on real-world coding tasks interacting with files in various situations.

### 2.2 Code LLMs

Previous works enhance LLM’s coding ability through various methods. OpenCodeInterpreter Zheng et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib43)) introduces the CodeFeedback dataset and a code execution system with feedback. The fine-tuned model achieves a great performance on coding benchmarks. CodeAct Wang et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib33)) uses executable Python code to unify LLM agents’ action space, enabling sophisticated task execution through multi-turn interactions. NexT Ni et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib25)) teaches LLM reasoning the execution process of code step by step, effectively improving the code quality. WizardCoder Luo et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib24)), Magicoder Wei et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib37)), and AlchemistCoder Song et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib30)) build effective fine-tuning datasets from massive and multi-source data to train advanced code LLMs. Pre-training on code-rich data is also a good method to help LLM coding. CodeQwen Bai et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib2)), and DeepSeek-Coder Guo et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib11)); DeepSeek-AI et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib7)) develop specialized models for coding by continuing to pre-train on code data and employing supervised fine-tuning strategies.

### 2.3 LLM Agent for real-world tasks

LLM as Agent is a great Qian et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib27)); Park et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib26)); Chen et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib6)) attempt utilizing LLM in real-world tasks. ReAct Yao et al. ([2022](https://arxiv.org/html/2407.16732v2#bib.bib40)) first introduced Agent’s Reasoning and Action Format. Previous works design many frameworks that build and organize LLM Agents to complete real-world coding tasks. MetaGPT Hong et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib14)) ChatDev Qian et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib27)), DataInterpreter Hong et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib13)), and MatplotAgent Yang et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib39)) employ agents to complete software development or data science tasks. AgentCoder Huang et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib15)) focuses on simple code complement tasks. Furthermore, some general multi-agent systems try to adapt agents to various tasks Chen et al. ([2023b](https://arxiv.org/html/2407.16732v2#bib.bib5), [a](https://arxiv.org/html/2407.16732v2#bib.bib3)); Wang et al. ([2023b](https://arxiv.org/html/2407.16732v2#bib.bib35)).

## 3 PyBench

![Refer to caption](img/be65e16559a80788e4185285beb74a2d.png)

Figure 2: The construction and evaluation workflow of PyBench

Coding is a core skill for LLM Agents. When the agent needs to solve real-world coding tasks, it should not only write executable code but also utilize the results of execution to guide subsequent actions and interact with files. Python is a powerful programming language with almost 4 million packages, capable of covering almost all real-world coding tasks. Therefore, we propose building PyBench to evaluate the LLM agent’s ability in reasoning, writing executable Python code, and utilizing code results.

### 3.1 Task Formulation

We first define what kinds of tasks need to be solved as a truly helpful LLM Agent equipped with a Python code interpreter. Given a user query $q$ described in natural language and the related file $F_{in}$, the Agent need to generate a formal answer $Ans$ to user query and output file $F_{out}$, which fulfill the user’s requirement:

|  | $Ans,F_{out}=A(q,F_{in})$ |  | (1) |

where $A$ is the LLM Agent equipped with a code interpreter. To be specific, the query $q$ from users can be very high level, reflecting common occurrences in daily life where users may not have precise requirements. The related file $F_{in}$ contains various types of data such as chart, text, audio, image, etc., showcase the LLM Agent should adapt to various real-world coding tasks.

### 3.2 Task Categories

In the realm of practical coding applications, the LLM Agent is required to autonomously tackle a diverse array of real-world coding challenges. To ensure a comprehensive evaluation of its capabilities, we have meticulously curated five main categories of real-world coding tasks. Each category is designed to test the LLM Agent’s proficiency across a broad spectrum of scenarios, ranging from data analysis to software development. Table [1](https://arxiv.org/html/2407.16732v2#S1.T1 "Table 1 ‣ 1 Introduction ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") demonstrates these categories:

Chart Analysis. In the digital age, the ability to efficiently analyze and interpret data is indispensable. This category focuses on tasks that involve handling csv or xlsx files for various purposes such as data preprocessing, transformation, visualization, and the application of machine learning algorithms Hong et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib13)); Yang et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib39)). The output from the LLM Agent includes detailed analytical reports, visual representations, or even trained machine learning models in pkl or joblib formats.

Text Analysis. This category encompasses tasks related to processing txt and pdf files, including summarization, keyword extraction, word cloud generation, and thematic analysis.

Image & Audio Editing. As visual content becomes increasingly prevalent, the demand for personalized image-processing solutions rises. This category evaluates the LLM Agent’s ability to manipulate images in png and jpg formats through various techniques such as saturation adjustment, merging, cropping, and generating QR codes from scratch. Parallel to image processing, this category also focuses on the manipulation of audio files (e.g. mp3 and wav). Tasks include volume control, audio trimming, and the creation of audio visualizations, reflecting the diverse needs of users in the realm of audio content creation and modification.

Complex Math. Beyond the capabilities of basic calculators or the Chain of Thought methodology Wei et al. ([2022](https://arxiv.org/html/2407.16732v2#bib.bib36)), this category presents challenges involving large-scale computations, polynomial equation solving, and advanced calculus. It is designed to test LLM Agents’ ability to navigate and solve intricate mathematical problems via a code interpreter.

Website & Software Development. This category is dedicated to the practical application of coding skills in the development of personal websites and simple software projects, such as a Pac-Man game. It assesses the LLM Agent’s ability to translate user requirements into functional and interactive applications, showcasing its versatility and creativity in software development Hong et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib14)); Qian et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib27)).

### 3.3 Data Collection

We collect and filter the files in PyBench from two main sources.

Kaggle Data. [Kaggle](https://www.kaggle.com/) is a great platform for machine learning, which contains massive datasets. We obtained csv and xlsx data on Kaggle through web crawlers. There are two principles of filtering the files. Firstly, the files should not be too large, considering the limited memory in the test environment. Secondly, the data tables must contain multiple columns with clear meaning, simulating commonly used files.

arXiv Data. We collect pdf and txt from [arXiv](https://arxiv.org/). The papers on arXiv are high-quality text with a clear theme and structure, which is suitable for text-based QA, Theme Analysis, and drawing word clouds.

Other Sources Data. For other file types, we responsibly collect files, including png, jpeg, gif, mp3, and wav from the internet, ensuring that all content respects copyright laws, protects user privacy, and is free from harmful elements.

### 3.4 Task Generation

The queries in PyBench must be precisely related to files and diverse to ensure comprehensive coverage. To generate these queries, we designed a multi-agent cooperation mechanism. Figure [2](https://arxiv.org/html/2407.16732v2#S3.F2 "Figure 2 ‣ 3 PyBench ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") illustrates the PyBench construction workflow.

First, we prepare lists of keywords for each subclass in a category, forming a keywords dictionary for each category Eldan and Li ([2023](https://arxiv.org/html/2407.16732v2#bib.bib10)). The type of file determines which category the task belongs to. Given the initial lines of a file(for chart and text files) and randomly selected keywords from the related category’s dictionary, the Query Generator selects appropriate keywords that align with the file content and compose a query. Detailed keyword list examples can be found in Appendix [B](https://arxiv.org/html/2407.16732v2#A2 "Appendix B Key words example for each category ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks").

Next, a Keywords Checker and a Content Checker verify if the query aligns with the file content and the selected keywords. Feedback from these checkers is used to refine the query. Once both checkers approve, the query undergoes manual editing to clarify the output format and file path for the convenience of verifying. If the Query Generator fails to produce a proper query within 5 rounds, these files will be skipped.

Queries that pass all checks are paired with their related files to form a query-file pair, which constitutes a task in PyBench. Appendix [A.2](https://arxiv.org/html/2407.16732v2#A1.SS2 "A.2 Query Generation Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") and [A.3](https://arxiv.org/html/2407.16732v2#A1.SS3 "A.3 Checker Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") detailed the prompt of each agent.

### 3.5 Evaluation

#### 3.5.1 Trajectory Generation

Equip LLM Agent with Code Interpreter. We equipped each LLM with a code interpreter to execute Python code written by the LLM and provide feedback on the execution results. Previous works on LLM tool usage Qin et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib28)) typically used tools in a function-calling format. Inspired by Wang et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib33)), which uses code as an action and outperforms alternatives, we designed two special tokens: <|execute_start|> and <|execute_end|> to help the LLM use the code interpreter more effectively. Appendix [D](https://arxiv.org/html/2407.16732v2#A4 "Appendix D Code as Action VS Function Calling ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows the different performance of the two formats.

![Refer to caption](img/9aec8155a3e0bb2b27ad50cdb8e07651.png)

Figure 3: Function Call vs. Our Code Interpreter Format

Reasoning and Action. Given the task description and the uploaded file path, the LLM Agent is prompted to analyze the current situation and plan its actions before writing executable code Yao et al. ([2022](https://arxiv.org/html/2407.16732v2#bib.bib40)). The code will be executed in a pre-defined Python environment (with commonly used packages). The result of the code, whether is an output or an error message, well serve as feedback to the LLM. The LLM will follow the loop until it fulfills the task or reaches a maximum step limit (default set to 10). Figure [4](https://arxiv.org/html/2407.16732v2#S3.F4 "Figure 4 ‣ 3.5.1 Trajectory Generation ‣ 3.5 Evaluation ‣ 3 PyBench ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") detailed this process.

![Refer to caption](img/8366b17acdf17f7937f2e53e7118414f.png)

Figure 4: Generating Trajectory Data by ReAct

#### 3.5.2 Unit Test

To objectively and effectively test whether the LLM Agent has completed a task, we have implemented a unit test for each task in PyBench. For tasks with a fixed answer, we verify whether the Agent provides a final response that contains the correct answer. For tasks requiring file output, such as cleaning datasets or editing images and audio, we check whether the output files meet the specified requirements. For tasks without a fixed answer, such as generating a word cloud or a website, we verify the existence of the output files. The design of the unit Test is detailed in Appendix [C](https://arxiv.org/html/2407.16732v2#A3 "Appendix C Example Unit Test ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks")

#### 3.5.3 LLM as Evaluator

Although unit tests are convenient and objective, they may fail to comprehensively evaluate open-ended tasks, such as assessing the coherence and fluency of text output by large models. Therefore, we also employ an LLM (GPT-4o) as an evaluator to provide pass-or-fail decisions for each trajectory, serving as an alternative to unit tests. Appendix [A.4](https://arxiv.org/html/2407.16732v2#A1.SS4 "A.4 Evaluation Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") provides the detailed prompt used for the LLM evaluator.

| Datasets | Instructions | Turns per Traj. | Tokens per Traj. |
| --- | --- | --- | --- |
|  |  | <=3 | 4to6 | 7to9 | >=10 |  |
| CodeFeedback | 66383 | 41696 | 21583 | 2923 | 181 | 1503.93 |
| CodeActInstruct | 7139 | 3482 | 3567 | 0 | 0 | 1165.03 |
| PyInstruct | 3091 | 1234 | 1644 | 164 | 49 | 2017.15 |

Table 2: Statistics of CodeFeedback, CodaActInstruct, and PyInstruct. Token statistics are computed by using Llama-2 tokenizer.

#### 3.5.4 Evaluation Metrics

There are three evaluation metrics in PyBench.

Pass Rate (UT). The percentage of passed tasks evaluated by unit tests.

Pass Rate (LLM). The percentage of passed tasks evaluated by the LLM Evaluator.

Average Turns. The number of steps taken to complete each task. If a task fails, the number of turns is set to the maximum turn limit (10 turns).

## 4 Fine-tuning LLM Agent for Real World Coding Task

In order to figure out what kind of capabilities are required and what training data could help enhance the LLM Agent’s performance on PyBench, we collect 4 datasets enhancing the Agent’s abilities in planning, coding, and multi-turn interaction.

Homologous dataset. Intuitively, homologous datasets can enhance performance on similar tasks. Therefore, we employ GPT-3.5-turbo to synthesize a homologous dataset: PyInstruct. We generate tasks using the same method as PyBench, but with different files and without manual modifications. Subsequently, using the method described in Section [3.5.1](https://arxiv.org/html/2407.16732v2#S3.SS5.SSS1 "3.5.1 Trajectory Generation ‣ 3.5 Evaluation ‣ 3 PyBench ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks"), we synthesized 3,091 trajectories based on these tasks. These trajectories cover every task category in PyBench, resulting in the construction of PyInstruct as a homologous dataset to PyBench. Multi-turn of code interaction dataset. Most of the tasks in PyBench need multi-turn interaction with the Python code interpreter. It will provide feedback on code execution results or error traceback messages, which should be fully leveraged by the LLM Agent. There are several existing datasets aiming to enhance LLM’s ability. CodeActInstruct Wang et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib33)) focuses on improving LLM’s abilities in various multi-turn tasks such as information seeking, software tool usage, external memory access, and robot planning, all executed in the format of Python code. CodeFeedback Zheng et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib43)) filters open-source code instruction data and converts them into multi-turn code with execution results. We repurpose the data by “equipping” our special token for the Python code interpreter. These datasets may help enhance LLM’s ability to utilize code feedback. Table [2](https://arxiv.org/html/2407.16732v2#S3.T2 "Table 2 ‣ 3.5.3 LLM as Evaluator ‣ 3.5 Evaluation ‣ 3 PyBench ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows the statistics information of CodeFeedback, CodeActInstruct, and PyInstruct.

Multi-turn chat dataset. Additionally, in PyBench, the LLM Agent is expected to comprehend the user’s instructions and provide a formal response upon completing the task. High-quality multi-turn chat is also crucial for a Code Agent. UltraChat Ding et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib8)) is a large-scale dataset comprising 1.5 million high-quality multi-turn instructional dialogues specifically designed to improve the performance of open-source conversational models, which is perfectly aligned with our requirements.

Code-Rich corpus. The foundation ability of a code agent is the quality and correctness of its code. We assume that continue pre-train on code-rich corpus could contribute to solving PyBench tasks. The-stack-v2 Lozhkov et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib23)) introduces a code-rich corpus of Jupyter notebooks, which contains 11 million lines.

After collecting the four types of datasets, we conducted a series of experiments to figure out what are necessary abilities to solve the PyBench tasks and how to improve the LLM performance on real-world coding tasks.

## 5 Experiment

| Models | Params. | PyBench | HumanEval(+) | MBPP(+) |
|  |  | Pass Rate(LLM) | Pass Rate(UT) | Avg Turns |  |  |
|  |  |  | Chart. | Text. | Image. | Math. | Software. | Overall |  |  |  |
| Closed-source Models |  |  |  |  |  |  |  |  |  |  |  |
| GPT-3.5 Turbo | - | 58.3 | 72.6 | 60.9 | 70.8 | 100.0 | 0.0 | 62.9 | 5.2 | 78.6(70.7) | 82.5(69.7) |
| GPT-4o | - | 81.8 | 80.6 | 82.6 | 77.1 | 100.0 | 100.0 | 79.7 | 4.3 | 91.5(85.4) | 91.2(76.6) |
| GPT-4o-mini | - | 67.1 | 77.4 | 69.6 | 87.5 | 100.0 | 100.0 | 81.1 | 4.5 | 87.2(85.4) | 90.5(76.5) |
| 70B size |  |  |  |  |  |  |  |  |  |  |  |
| Llama3-Instruct | 70B | 80.2 | 38.7 | 34.8 | 72.9 | 33.3 | 50.0 | 78.3 | 4.0 | 77.4(72.0) | 82.3(69.0) |
| CodeLlama-Instruct | 70B | 26.6 | 24.2 | 17.4 | 41.7 | 16.7 | 0.0 | 37.8 | 7.5 | 72.0(65.9) | 62.2(51.2) |
| DeepSeek-llm-chat | 67B | 39.4 | 21.0 | 17.4 | 37.5 | 33.3 | 25.0 | 26.6 | 9.2 | 67.7(58.5) | 62.2(50.5) |
| 33B size |  |  |  |  |  |  |  |  |  |  |  |
| DeepSeek-coder-Instruct | 33B | 35.3 | 22.6 | 0.0 | 4.2 | 16.7 | 0.0 | 37.0 | 6.7 | 81.1(75.0) | 80.4(70.1) |
| CodeLlama-Instruct | 34B | 9.6 | 8.1 | 13.0 | 0.0 | 16.7 | 0.0 | 6.3 | 8.9 | 51.8(43.9) | 69.3(56.3) |
| Yi-1.5-Chat-16K | 34B | 44.2 | 33.9 | 34.8 | 56.3 | 50.0 | 0.0 | 41.3 | 9.0 | 58.8(53.4) | 77.5(63.6) |
| Qwen-1.5-Chat | 32B | 43.0 | 12.9 | 39.1 | 10.4 | 50.0 | 75.0 | 19.6 | 9.3 | 58.5(55.5) | 66.1(56.1) |
| Gemma-2-Instruct | 27B | 66.1 | 58.1 | 56.5 | 87.5 | 83.3 | 75.0 | 69.2 | 5.0 | 49.4(45.7) | 73.3(61.1) |
| 7B size |  |  |  |  |  |  |  |  |  |  |  |
| Llama3-Instruct | 8B | 49.7 | 43.5 | 26.1 | 72.9 | 66.7 | 50.0 | 49.7 | 6.7 | 61.6(56.7) | 70.1(59.3) |
| Mistral-Instruct-v0.2 | 7B | 17.5 | 17.7 | 17.4 | 18.8 | 16.7 | 0.0 | 17.5 | 8.9 | 35.4(30.5) | 31.0(24.1) |
| Gemma-2-Instruct | 9B | 41.7 | 37.1 | 30.4 | 70.1 | 83.3 | 50.0 | 49.7 | 7.2 | 55.5(48.8) | 66.1(56.1) |
| Qwen2-Instruct | 7B | 57.8 | 59.7 | 47.8 | 58.3 | 83.3 | 75.0 | 58.7 | 5.9 | 64.6(61.0) | 62.3(52.3) |
| CodeQwen1.5-chat | 7B | 49.2 | 48.4 | 43.5 | 66.7 | 83.3 | 25.0 | 54.5 | 7.9 | 83.5(78.7) | 79.4(69.0) |
| CodeActAgent-Llama2 | 7B | 12.4 | 16.1 | 21.7 | 20.8 | 33.3 | 0.0 | 18.9 | 8.1 | 18.9(15.2) | 22.8(18.3) |
| CodeActAgent-Mistral | 7B | 18.8 | 17.7 | 4.3 | 20.8 | 16.7 | 0.0 | 16.1 | 8.8 | 28.7(26.8) | 43.9(35.4) |
| OpenCodeInterpreter-DS | 6.7B | 25.0 | 11.3 | 21.7 | 75.0 | 66.7 | 25.0 | 51.7 | 6.8 | 77.4(73.8) | 80.2(66.4) |
| InternLM2_5-chat | 7B | 17.5 | 32.3 | 13.0 | 4.17 | 50.0 | 25.0 | 20.27 | 9.84 | 57.3(51.2) | 61.6(51.5) |
| PyLlama3(w/o cpt) | 8B | 56.7 | 58.0 | 52.2 | 73.0 | 50.0 | 50.0 | 58.7 | 6.3 | 54.3(48.2) | 59.5(50.3) |
| PyLlama3 | 8B | 73.4 | 62.9 | 65.2 | 56.3 | 66.7 | 50.0 | 60.8 | 6.1 | 57.3(48.2) | 62.2(51.1) |

Table 3: The main result table. We test closed-source models, 70B size models, 33B size models, and 7B size models. We bold the best results for each size model’s results.

| PyInst. | Code. | UltraCh. | Jupyter. | Pass Rate(UT) |
| --- | --- | --- | --- | --- |
|  |  |  |  | Chart. | Text. | Image. | Math. | Software. | Over All |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ✓ | ✗ | ✗ | ✗ | 3.2 | 13.0 | 6.3 | 50.0 | 25.0 | 8.4 |
| ✗ | ✓ | ✗ | ✗ | 40.3 | 26.1 | 58.3 | 83.3 | 25.0 | 45.5 |
| ✓ | ✓ | ✗ | ✗ | 51.6 | 56.5 | 60.4 | 100.0 | 25.0 | 56.6 |
| ✓ | ✓ | ✓ | ✗ | 62.9 | 52.2 | 56.3 | 83.3 | 25.0 | 58.7 |
| ✓ | ✓ | ✗ | ✓ | 58.0 | 52.2 | 73.0 | 50.0 | 50.0 | 59.4 |
| ✓ | ✓ | ✓ | ✓ | 62.9 | 65.2 | 56.3 | 66.7 | 50.0 | 60.8 |

Table 4: Pass Rate (UT) with different training datasets. PyInst. stands for PyInstruct. Code. refers to Codefeedback & CodeActInstruct. UltraCh. means UltraChat, and Jupyter. indicates continuing pre-training on the Jupyter notebook corpus.

### 5.1 Main Result on PyBench

Testing Evaluation Setup. Firstly, we prepare a conda environment equipped with 182 commonly used Python packages (Detailed in Appendix [G](https://arxiv.org/html/2407.16732v2#A7 "Appendix G Python Packages in our environment ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks")). The max turn is set to $k=10$. We prompt the LLM to use code interpreter and follow ReAct Yao et al. ([2022](https://arxiv.org/html/2407.16732v2#bib.bib40)) format. (Appendix [A.4](https://arxiv.org/html/2407.16732v2#A1.SS4 "A.4 Evaluation Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks")) The code in LLM’s response will be extracted, and the execution result will be returned to LLM. After getting the trajectory and output files, we calculate the pass rate through the unit test set (UT) and LLM Evaluator set (LLM). We also adopt other benchmarks to test the model’s basic code ability, including HumanEval Chen et al. ([2021](https://arxiv.org/html/2407.16732v2#bib.bib4)) and HumanEval+ Liu et al. ([2023a](https://arxiv.org/html/2407.16732v2#bib.bib21)) for single-turn code generation, and MBPP Austin et al. ([2021](https://arxiv.org/html/2407.16732v2#bib.bib1)) and MBPP+ Liu et al. ([2023a](https://arxiv.org/html/2407.16732v2#bib.bib21)) for simple Python programming problems.

Results on PyBench. Table [3](https://arxiv.org/html/2407.16732v2#S5.T3 "Table 3 ‣ 5 Experiment ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows our main experiment result. GPT-4o scored highly in all five tasks, showing its great ability. However, models including CodeLlama-Instruct-70B, DeepSeek-coder-Instruct-33B, and CodeLlama-Instruct-33B, although trained on code corpus and perform well on HumanEval and MBPP, struggle to solve PyBench tasks, proving that code ability does not always lead to a good score on PyBench. Most of the advanced 7B size models can only solve about 50% of tasks in PyBench. Our PyLlama3, with continued pre-train and fine-tuning on related datasets, surpasses Llama3-8B-Instruct on Chart Analysis, Text Analysis, and Complex Math.

Coding ability is the foundation. CodeActAgent-Llama-2-7B and CodeActAgent-Mistral-7B Wang et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib33)), although trained on CodeActInstruct, who teach LLM Agent use tools in code format, have the weakest performance on PyBench. The failure might be caused by low performance on basic coding ability benchmarks: HumanEval(+) and MBPP(+). The result illustrates basic code ability and the foundation to solve real-world coding tasks in PyBench.

PyBench evaluates comprehensive abilities of LLM Agent. Great performance on basic coding benchmarks does not directly lead to equal performance on PyBench. Although models like DeepSeek-coder-Instruct-33B and CodeLlama-Instruct-34B scored well on HumanEval(+) and MBPP(+), they got a weak score on PyBench. This indicates that mere coding ability cannot lead to success in real-world coding tasks. PyBench also evaluates the LLM agent’s abilities, including planning, multi-turn interaction, leveraging code feedback, and generating formal responses to the user.

Compare LLM Evaluator and Unit Test. Table [3](https://arxiv.org/html/2407.16732v2#S5.T3 "Table 3 ‣ 5 Experiment ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows that the evaluation results from the LLM and the unit tests are generally consistent, with only minor fluctuations. However, there are significant discrepancies for several models. To investigate this, we randomly selected 60 trajectories from the entire dataset for manual assessment. We found that in over 80% of the cases, the LLM evaluation results were consistent with the unit test results. Nevertheless, in some instances, the LLM evaluator might only determine whether the code executes successfully without verifying its correctness. In other cases, the tested LLM agent might write code or provide answers that coincidentally pass the unit tests. A detailed analysis of these scenarios is provided in Appendix [E](https://arxiv.org/html/2407.16732v2#A5 "Appendix E Compare LLM Evaluator and Unit Test ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks").

### 5.2 Analysis on the training dataset

In this section, we conduct ablation studies on training datasets to explore the necessary capabilities for solving PyBench.

#### 5.2.1 Train Setup

We conduct full-parameter supervised fine-tuning on Llama3-8B with a sequence length of 32768 tokens Liu et al. ([2023b](https://arxiv.org/html/2407.16732v2#bib.bib22)); Roziere et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib29)); Touvron et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib31)), ensuring it could handle the content of chart and text files. For each version of the supervised fine-tuning model, we use 32 A100 GPU and train 4000 steps. The learning rate is set as 1e-5 with a 0.05 warm-up ratio and a cosine scheduler. As for the continue pre-trained model, we added an extra 3000 steps to train the corpus.

#### 5.2.2 Ablation study

LLMs’ performance on specific tasks can often be enhanced through supervised fine-tuning on homologous datasets, where the model learns exactly what to do in particular situations. Initially, we hypothesized that training the Llama3-8B-base model on PyInstruct, 3k homologous trajectories data, would teach the model to handle real-world coding tasks. However, as shown in Table [4](https://arxiv.org/html/2407.16732v2#S5.T4 "Table 4 ‣ 5 Experiment ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks"), this model struggles with PyBench tasks. From the generated trajectories, we observed that the model fails to follow human instructions and even struggles to locate the correct file paths.

To address the drawbacks, we add two multi-turn code interaction datasets: CodeActInstruct Wang et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib33)) and CodeFeedback Zheng et al. ([2024b](https://arxiv.org/html/2407.16732v2#bib.bib43)). We train the model on these datasets and PyInstruct. The result indicated a significant improvement. But when we remove PyInstruct and train the model only on CodeActInstruct and CodeFeedback, the model’s performance suffers a sharp decline in Chart Analysis and Text Analysis, demonstrating PyInstruct can enhance the model’s capability.

Additionally, to further enhance the model’s capabilities, we added UltraChat Ding et al. ([2023](https://arxiv.org/html/2407.16732v2#bib.bib8)) in the training datasets and trained the model with the same training settings. However, the result shows adding UltraChat only brings improvement to Chart Analysis tasks.

We found that the model still struggles with using some packages and specific techniques for real-world coding tasks. Consequently, we conducted continue pre-train on a Jupyter Notebook corpus Lozhkov et al. ([2024](https://arxiv.org/html/2407.16732v2#bib.bib23)) before fine-tuning on PyInstruct, CodeFeedback, CodeActInstruct, and UltraChat. As shown in Table [4](https://arxiv.org/html/2407.16732v2#S5.T4 "Table 4 ‣ 5 Experiment ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks"), continue pre-train can also bring great improvement to Text Analysis tasks. The continued pretraining model achieves the best performance after fine-tuning on PyInstruct, CodeFeedback, CodeActInstruct, and UltraChat, which we named to PyLlama3. And the version without continue pre-train is named PyLlama3(w/o cpt).

In conclusion, we figured out that PyBench requires the model’s basic coding abilities, which instruct them to write Python properly, as well as multi-turn interaction and reasoning abilities to solve real-world tasks.

## 6 Conclusion

In this paper, we propose PyBench, a comprehensive benchmark encompassing five main categories that reflect real-world coding situations. Through collecting files and generating related queries, PyBench could evaluate LLM’s usability and efficiency in real-world tasks. After evaluating plenty of LLMs, we find many of them are struggling with real-world coding tasks. We collected and synthesized four datasets and trained PyLlama3 whose performance surpass many 7B, 33B, and 70B size models. Our ablation studies prove the effectiveness of the datasets, figuring out a way to train a model that could adapt to real-world coding tasks. Solving PyBench tasks means the LLM could interact with the file system with Python code, which symbolizes a great milestone in developing a really usable LLM Agent who can serve as a helpful life assistant for humankind.

## 7 limitations

Our work introduces a comprehensive benchmark: PyBench to evaluate LLM Agent on real-world coding tasks. Although five main categories are included in PyBench, there are many cases in the real world that have not been covered. The coding problem that uses other coding languages instead of Python is not covered either.

## References

*   Austin et al. (2021) Jacob Austin, Augustus Odena, Maxwell Nye, Maarten Bosma, Henryk Michalewski, David Dohan, Ellen Jiang, Carrie Cai, Michael Terry, Quoc Le, et al. 2021. Program synthesis with large language models. *arXiv preprint arXiv:2108.07732*.
*   Bai et al. (2023) Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang, et al. 2023. Qwen technical report. *arXiv preprint arXiv:2309.16609*.
*   Chen et al. (2023a) Guangyao Chen, Siwei Dong, Yu Shu, Ge Zhang, Jaward Sesay, Börje F Karlsson, Jie Fu, and Yemin Shi. 2023a. Autoagents: A framework for automatic agent generation. *arXiv preprint arXiv:2309.17288*.
*   Chen et al. (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, et al. 2021. Evaluating large language models trained on code. *arXiv preprint arXiv:2107.03374*.
*   Chen et al. (2023b) Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chen Qian, Chi-Min Chan, Yujia Qin, Yaxi Lu, Ruobing Xie, et al. 2023b. Agentverse: Facilitating multi-agent collaboration and exploring emergent behaviors in agents. *arXiv preprint arXiv:2308.10848*.
*   Chen et al. (2024) Zehui Chen, Kuikun Liu, Qiuchen Wang, Wenwei Zhang, Jiangning Liu, Dahua Lin, Kai Chen, and Feng Zhao. 2024. Agent-flan: Designing data and methods of effective agent tuning for large language models. *arXiv preprint arXiv:2403.12881*.
*   DeepSeek-AI et al. (2024) DeepSeek-AI, Qihao Zhu, Daya Guo, Zhihong Shao, Dejian Yang, Peiyi Wang, Runxin Xu, Y. Wu, Yukun Li, Huazuo Gao, Shirong Ma, Wangding Zeng, Xiao Bi, Zihui Gu, Hanwei Xu, Damai Dai, Kai Dong, Liyue Zhang, Yishi Piao, Zhibin Gou, Zhenda Xie, Zhewen Hao, Bingxuan Wang, Junxiao Song, Deli Chen, Xin Xie, Kang Guan, Yuxiang You, Aixin Liu, Qiushi Du, Wenjun Gao, Xuan Lu, Qinyu Chen, Yaohui Wang, Chengqi Deng, Jiashi Li, Chenggang Zhao, Chong Ruan, Fuli Luo, and Wenfeng Liang. 2024. [Deepseek-coder-v2: Breaking the barrier of closed-source models in code intelligence](http://arxiv.org/abs/2406.11931).
*   Ding et al. (2023) Ning Ding, Yulin Chen, Bokai Xu, Yujia Qin, Zhi Zheng, Shengding Hu, Zhiyuan Liu, Maosong Sun, and Bowen Zhou. 2023. Enhancing chat language models by scaling high-quality instructional conversations. *arXiv preprint arXiv:2305.14233*.
*   Du et al. (2023) Xueying Du, Mingwei Liu, Kaixin Wang, Hanlin Wang, Junwei Liu, Yixuan Chen, Jiayi Feng, Chaofeng Sha, Xin Peng, and Yiling Lou. 2023. [Classeval: A manually-crafted benchmark for evaluating llms on class-level code generation](http://arxiv.org/abs/2308.01861).
*   Eldan and Li (2023) Ronen Eldan and Yuanzhi Li. 2023. Tinystories: How small can language models be and still speak coherent english? *arXiv preprint arXiv:2305.07759*.
*   Guo et al. (2024) Daya Guo, Qihao Zhu, Dejian Yang, Zhenda Xie, Kai Dong, Wentao Zhang, Guanting Chen, Xiao Bi, Y Wu, YK Li, et al. 2024. Deepseek-coder: When the large language model meets programming–the rise of code intelligence. *arXiv preprint arXiv:2401.14196*.
*   Hendrycks et al. (2021) Dan Hendrycks, Steven Basart, Saurav Kadavath, Mantas Mazeika, Akul Arora, Ethan Guo, Collin Burns, Samir Puranik, Horace He, Dawn Song, and Jacob Steinhardt. 2021. Measuring coding challenge competence with apps. *NeurIPS*.
*   Hong et al. (2024) Sirui Hong, Yizhang Lin, Bangbang Liu, Binhao Wu, Danyang Li, Jiaqi Chen, Jiayi Zhang, Jinlin Wang, Lingyao Zhang, Mingchen Zhuge, et al. 2024. Data interpreter: An llm agent for data science. *arXiv preprint arXiv:2402.18679*.
*   Hong et al. (2023) Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, et al. 2023. Metagpt: Meta programming for multi-agent collaborative framework. *arXiv preprint arXiv:2308.00352*.
*   Huang et al. (2023) Dong Huang, Qingwen Bu, Jie M Zhang, Michael Luck, and Heming Cui. 2023. Agentcoder: Multi-agent-based code generation with iterative testing and optimisation. *arXiv preprint arXiv:2312.13010*.
*   Jimenez et al. (2023) Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik Narasimhan. 2023. Swe-bench: Can language models resolve real-world github issues? *arXiv preprint arXiv:2310.06770*.
*   Khan et al. (2023) Mohammad Abdullah Matin Khan, M Saiful Bari, Xuan Long Do, Weishi Wang, Md Rizwan Parvez, and Shafiq Joty. 2023. [xcodeeval: A large scale multilingual multitask benchmark for code understanding, generation, translation and retrieval](http://arxiv.org/abs/2303.03004).
*   Lai et al. (2023) Yuhang Lai, Chengxi Li, Yiming Wang, Tianyi Zhang, Ruiqi Zhong, Luke Zettlemoyer, Wen-tau Yih, Daniel Fried, Sida Wang, and Tao Yu. 2023. Ds-1000: A natural and reliable benchmark for data science code generation. In *International Conference on Machine Learning*, pages 18319–18345\. PMLR.
*   Li et al. (2024) Bowen Li, Wenhan Wu, Ziwei Tang, Lin Shi, John Yang, Jinyang Li, Shunyu Yao, Chen Qian, Binyuan Hui, Qicheng Zhang, et al. 2024. Devbench: A comprehensive benchmark for software development. *arXiv preprint arXiv:2403.08604*.
*   Li et al. (2023) Rongao Li, Jie Fu, Bo-Wen Zhang, Tao Huang, Zhihong Sun, Chen Lyu, Guang Liu, Zhi Jin, and Ge Li. 2023. Taco: Topics in algorithmic code generation dataset. *arXiv preprint arXiv:2312.14852*.
*   Liu et al. (2023a) Jiawei Liu, Chunqiu Steven Xia, Yuyao Wang, and Lingming Zhang. 2023a. [Is your code generated by chatgpt really correct? rigorous evaluation of large language models for code generation](http://arxiv.org/abs/2305.01210).
*   Liu et al. (2023b) Xiaoran Liu, Hang Yan, Shuo Zhang, Chenxin An, Xipeng Qiu, and Dahua Lin. 2023b. Scaling laws of rope-based extrapolation. *arXiv preprint arXiv:2310.05209*.
*   Lozhkov et al. (2024) Anton Lozhkov, Raymond Li, Loubna Ben Allal, Federico Cassano, Joel Lamy-Poirier, Nouamane Tazi, Ao Tang, Dmytro Pykhtar, Jiawei Liu, Yuxiang Wei, Tianyang Liu, Max Tian, Denis Kocetkov, Arthur Zucker, Younes Belkada, Zijian Wang, Qian Liu, Dmitry Abulkhanov, Indraneil Paul, Zhuang Li, Wen-Ding Li, Megan Risdal, Jia Li, Jian Zhu, Terry Yue Zhuo, Evgenii Zheltonozhskii, Nii Osae Osae Dade, Wenhao Yu, Lucas Krauß, Naman Jain, Yixuan Su, Xuanli He, Manan Dey, Edoardo Abati, Yekun Chai, Niklas Muennighoff, Xiangru Tang, Muhtasham Oblokulov, Christopher Akiki, Marc Marone, Chenghao Mou, Mayank Mishra, Alex Gu, Binyuan Hui, Tri Dao, Armel Zebaze, Olivier Dehaene, Nicolas Patry, Canwen Xu, Julian McAuley, Han Hu, Torsten Scholak, Sebastien Paquet, Jennifer Robinson, Carolyn Jane Anderson, Nicolas Chapados, Mostofa Patwary, Nima Tajbakhsh, Yacine Jernite, Carlos Muñoz Ferrandis, Lingming Zhang, Sean Hughes, Thomas Wolf, Arjun Guha, Leandro von Werra, and Harm de Vries. 2024. [Starcoder 2 and the stack v2: The next generation](http://arxiv.org/abs/2402.19173).
*   Luo et al. (2023) Ziyang Luo, Can Xu, Pu Zhao, Qingfeng Sun, Xiubo Geng, Wenxiang Hu, Chongyang Tao, Jing Ma, Qingwei Lin, and Daxin Jiang. 2023. [Wizardcoder: Empowering code large language models with evol-instruct](http://arxiv.org/abs/2306.08568).
*   Ni et al. (2024) Ansong Ni, Miltiadis Allamanis, Arman Cohan, Yinlin Deng, Kensen Shi, Charles Sutton, and Pengcheng Yin. 2024. Next: Teaching large language models to reason about code execution. *arXiv preprint arXiv:2404.14662*.
*   Park et al. (2023) Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S Bernstein. 2023. Generative agents: Interactive simulacra of human behavior. In *Proceedings of the 36th Annual ACM Symposium on User Interface Software and Technology*, pages 1–22.
*   Qian et al. (2023) Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, and Maosong Sun. 2023. Communicative agents for software development. *arXiv preprint arXiv:2307.07924*.
*   Qin et al. (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. 2023. Toolllm: Facilitating large language models to master 16000+ real-world apis. *arXiv preprint arXiv:2307.16789*.
*   Roziere et al. (2023) Baptiste Roziere, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Tal Remez, Jérémy Rapin, et al. 2023. Code llama: Open foundation models for code. *arXiv preprint arXiv:2308.12950*.
*   Song et al. (2024) Zifan Song, Yudong Wang, Wenwei Zhang, Kuikun Liu, Chengqi Lyu, Demin Song, Qipeng Guo, Hang Yan, Dahua Lin, Kai Chen, et al. 2024. Alchemistcoder: Harmonizing and eliciting code capability by hindsight tuning on multi-source data. *arXiv preprint arXiv:2405.19265*.
*   Touvron et al. (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, et al. 2023. Llama 2: Open foundation and fine-tuned chat models. *arXiv preprint arXiv:2307.09288*.
*   Wang et al. (2024a) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. 2024a. A survey on large language model based autonomous agents. *Frontiers of Computer Science*, 18(6):186345.
*   Wang et al. (2024b) Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, and Heng Ji. 2024b. Executable code actions elicit better llm agents. *arXiv preprint arXiv:2402.01030*.
*   Wang et al. (2023a) Xingyao Wang, Zihan Wang, Jiateng Liu, Yangyi Chen, Lifan Yuan, Hao Peng, and Heng Ji. 2023a. Mint: Evaluating llms in multi-turn interaction with tools and language feedback. *arXiv preprint arXiv:2309.10691*.
*   Wang et al. (2023b) Zhenhailong Wang, Shaoguang Mao, Wenshan Wu, Tao Ge, Furu Wei, and Heng Ji. 2023b. Unleashing cognitive synergy in large language models: A task-solving agent through multi-persona self-collaboration. *arXiv preprint arXiv:2307.05300*, 1(2):3.
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting elicits reasoning in large language models. *Advances in neural information processing systems*, 35:24824–24837.
*   Wei et al. (2024) Yuxiang Wei, Zhe Wang, Jiawei Liu, Yifeng Ding, and Lingming Zhang. 2024. [Magicoder: Empowering code generation with oss-instruct](http://arxiv.org/abs/2312.02120).
*   Yan et al. (2024) Weixiang Yan, Haitian Liu, Yunkun Wang, Yunzhe Li, Qian Chen, Wen Wang, Tingyu Lin, Weishan Zhao, Li Zhu, Hari Sundaram, and Shuiguang Deng. 2024. [Codescope: An execution-based multilingual multitask multidimensional benchmark for evaluating llms on code understanding and generation](http://arxiv.org/abs/2311.08588).
*   Yang et al. (2024) Zhiyu Yang, Zihan Zhou, Shuo Wang, Xin Cong, Xu Han, Yukun Yan, Zhenghao Liu, Zhixing Tan, Pengyuan Liu, Dong Yu, et al. 2024. Matplotagent: Method and evaluation for llm-based agentic scientific data visualization. *arXiv preprint arXiv:2402.11453*.
*   Yao et al. (2022) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2022. React: Synergizing reasoning and acting in language models. *arXiv preprint arXiv:2210.03629*.
*   Zhang et al. (2023) Fengji Zhang, Bei Chen, Yue Zhang, Jacky Keung, Jin Liu, Daoguang Zan, Yi Mao, Jian-Guang Lou, and Weizhu Chen. 2023. [Repocoder: Repository-level code completion through iterative retrieval and generation](http://arxiv.org/abs/2303.12570).
*   Zheng et al. (2024a) Qinkai Zheng, Xiao Xia, Xu Zou, Yuxiao Dong, Shan Wang, Yufei Xue, Zihan Wang, Lei Shen, Andi Wang, Yang Li, Teng Su, Zhilin Yang, and Jie Tang. 2024a. [Codegeex: A pre-trained model for code generation with multilingual benchmarking on humaneval-x](http://arxiv.org/abs/2303.17568).
*   Zheng et al. (2024b) Tianyu Zheng, Ge Zhang, Tianhao Shen, Xueling Liu, Bill Yuchen Lin, Jie Fu, Wenhu Chen, and Xiang Yue. 2024b. Opencodeinterpreter: Integrating code generation with execution and refinement. *arXiv preprint arXiv:2402.14658*.

## Appendix A Prompts

### A.1 Equip LLM with a code interpreter

Fig [5](https://arxiv.org/html/2407.16732v2#A1.F5 "Figure 5 ‣ A.1 Equip LLM with a code interpreter ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows how we prompt the LLM to use code interpreter.

![Refer to caption](img/e5d11eba23b8d73c1d6d924ea3fab9f7.png)

Figure 5: Prompt equipping LLM Agent with a code interpreter

### A.2 Query Generation Prompt

Fig [6](https://arxiv.org/html/2407.16732v2#A1.F6 "Figure 6 ‣ A.2 Query Generation Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") is the prompt to generate a related and diverse query from given file content.

![Refer to caption](img/8ecb1be5fa00476f92012e7a02e9ba33.png)

Figure 6: Prompt guide the LLM to generate queries

### A.3 Checker Prompt

Fig [7](https://arxiv.org/html/2407.16732v2#A1.F7 "Figure 7 ‣ A.3 Checker Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") and Fig [8](https://arxiv.org/html/2407.16732v2#A1.F8 "Figure 8 ‣ A.3 Checker Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") are the prompts for the content checker and keywords checker.

![Refer to caption](img/df586c270b1c8e8c240ff2f2326f8049.png)

Figure 7: Prompt of The Content Checker

![Refer to caption](img/2ce987a7a8ba551f1bc74fcc12d4fe93.png)

Figure 8: Prompt of the Keyword Checker

### A.4 Evaluation Prompt

Figure [9](https://arxiv.org/html/2407.16732v2#A1.F9 "Figure 9 ‣ A.4 Evaluation Prompt ‣ Appendix A Prompts ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows the LLM Evaluator’s prompt.

![Refer to caption](img/3fc5ad79bc0b991c3221cbaba413381b.png)

Figure 9: Evaluation Prompt

## Appendix B Key words example for each category

Fig [10](https://arxiv.org/html/2407.16732v2#A2.F10 "Figure 10 ‣ Appendix B Key words example for each category ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") is a keywords list example for Chart Analysis tasks

![Refer to caption](img/f484954387f37168da962b172afb1f17.png)

Figure 10: Keywords list on Chart Data Analysis

## Appendix C Example Unit Test

We show three types of Unit Tests in this appendix.

### C.1 Directly Verify the Answer

For the task with a fixed answer, we check whether the final response contains the answer

[⬇](data:text/plain;base64,ZGVmIHRlc3RfdGFza18xOSh0cmFqZWN0b3J5KToKICAgIGZpbmFsX2Fuc3dlcj10cmFqZWN0b3J5Wy0xXVsnY29udGVudCddCiAgICBhbnN3ZXJfbGlzdD1bIi0zNDYiLCIyNTA0Il0KICAgIGZvciBhbnMgaW4gYW5zd2VyX2xpc3Q6CiAgICAgICAgYXNzZXJ0IGFucyBpbiBmaW5hbF9hbnN3ZXI=)1def  test_task_19(trajectory):2  final_answer=trajectory[-1][’content’]3  answer_list=["-346","2504"]4  for  ans  in  answer_list:5  assert  ans  in  final_answer

### C.2 Verify the output file

For task clarify the output file, we check whether the output file fulfills the requirements.

[⬇](data:text/plain;base64,ZGVmIHRlc3RfdGFza184OCh0cmFqZWN0b3J5KToKICAgIGltYWdlX3BhdGggPSAiLi9vdXRwdXQvODgucG5nIgogICAgcmVmX3BhdGggPSAiLi9kYXRhLzg4LmpwZWciCiAgICBpbWFnZSA9IEltYWdlLm9wZW4oaW1hZ2VfcGF0aCkKICAgIGdyYXlfaW1hZ2UgPSBpbWFnZS5jb252ZXJ0KCdMJykKICAgIGdyYXlfYXJyYXkgPSBucC5hcnJheShncmF5X2ltYWdlKQogICAgYmxhY2tfdGhyZXNob2xkID0gNTAKICAgIGJsYWNrX3BpeGVscyA9IG5wLnN1bShncmF5X2FycmF5IDw9IGJsYWNrX3RocmVzaG9sZCkKICAgIHRvdGFsX3BpeGVscyA9IGdyYXlfYXJyYXkuc2l6ZQogICAgYmxhY2tfcmF0aW8gPSBibGFja19waXhlbHMgLyB0b3RhbF9waXhlbHMKICAgIGFzc2VydCBibGFja19yYXRpbyA+IDAuMjUsICJUaGUgYmxhY2sgcGl4ZWwgcGVyY2VudGFnZSBkb2VzIG5vdCBleGNlZWQgMzAlLiI=)1def  test_task_88(trajectory):2  image_path  =  "./output/88.png"3  ref_path  =  "./data/88.jpeg"4  image  =  Image.open(image_path)5  gray_image  =  image.convert(’L’)6  gray_array  =  np.array(gray_image)7  black_threshold  =  508  black_pixels  =  np.sum(gray_array  <=  black_threshold)9  total_pixels  =  gray_array.size10  black_ratio  =  black_pixels  /  total_pixels11  assert  black_ratio  >  0.25,  "The  black  pixel  percentage  does  not  exceed  30%."

### C.3 Verify the output file path

For Open-end tasks like website development, we simply detect whether the output file exists.

[⬇](data:text/plain;base64,ZGVmIHRlc3RfdGFza18xNDEodHJhamVjdG9yeSk6CiAgICBvdXRwdXRfZm9sZGVyID0gIi4vb3V0cHV0LzE0MSIKICAgIGh0bWxfZmlsZXNfZXhpc3QgPSBhbnkoZmlsZS5lbmRzd2l0aCgnLmh0bWwnKSBmb3IgZmlsZSBpbiBvcy5saXN0ZGlyKG91dHB1dF9mb2xkZXIpKQogICAgYXNzZXJ0IGh0bWxfZmlsZXNfZXhpc3Q=)1def  test_task_141(trajectory):2  output_folder  =  "./output/141"3  html_files_exist  =  any(file.endswith(’.html’)  for  file  in  os.listdir(output_folder))4  assert  html_files_exist

## Appendix D Code as Action VS Function Calling

We conducted an ablation study on the format of calling code interpreter. For function calling, we defined an execute_python function where the LLM passes a code string as the parameter and uses a special token <|tool_call|> before the function.

We repurposed all code snippets in PyInstruct, CodeFeedback, and CodeActInstruct to follow the function calling format. We trained the model without continue pre-train on code-rich corpus for it exactly aligns with our format. Compare it to PyLlama3(w/o cpt), Table [5](https://arxiv.org/html/2407.16732v2#A4.T5 "Table 5 ‣ Appendix D Code as Action VS Function Calling ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows that the function calling format struggles with real-world code tasks. The model often fails to follow the format and frequently neglects to call the execute_python tool, leading to failures.

| Format | Pass Rate(UT) | Average Turns |
| --- | --- | --- |
| Function call | 19.6 | 8.3 |
| Ours | 58.7 | 6.3 |

Table 5: Function Call vs Our format

## Appendix E Compare LLM Evaluator and Unit Test

| Good | Same | Bad |
| 12 | 43 | 5 |

Table 6: Compare the LLM Evaluator with the Unit Test. "Good" indicates that the LLM Evaluator is correct while the Unit Test is incorrect. "Same" means both the LLM Evaluator and the Unit Test provide the same pass-or-fail decision. "Bad" signifies that the LLM Evaluator is incorrect while the Unit Test is correct.

We randomly select 60 samples from all the generated trajectories and judge them manually. Table [6](https://arxiv.org/html/2407.16732v2#A5.T6 "Table 6 ‣ Appendix E Compare LLM Evaluator and Unit Test ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows in most situations, LLM Evaluator and Unit Test reach a consensus. In other situations, LLM Evaluator and Unit Test have their own advantages and drawbacks. Figure [11](https://arxiv.org/html/2407.16732v2#A5.F11 "Figure 11 ‣ Appendix E Compare LLM Evaluator and Unit Test ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows two examples.

![Refer to caption](img/c31889f6453545f714f66b9948013fdf.png)

Figure 11: Compare Unit Test and LLM Evaluator

## Appendix F Task Distribution

Table [7](https://arxiv.org/html/2407.16732v2#A6.T7 "Table 7 ‣ Appendix F Task Distribution ‣ PyBench: Evaluating LLM Agent on various real-world coding tasks") shows the number of each type of task in PyBench.

| Chart. | Text. | Image. | Math. | Software. |
| --- | --- | --- | --- | --- |
| 62 | 23 | 48 | 6 | 4 |

Table 7: The distribution of each type of task in PyBench.

## Appendix G Python Packages in our environment

> absl-pyanalytics-pythonattrsaudioreadbeautifulsoup4bokehcachetoolsCairoSVGcairosvgchardetclickclick-pluginscompressed-rtfdebugpydecoratordefusedxmldeprecatdocx2txtebooklibemail-validatorfastapifastjsonschemafbprophetffmpeg-pythonffmpyfirefitzFlaskflaskFlask-CacheBusterflask-cachebusterFlask-Corsflask-corsFlask-Loginflask-loginfonttoolsfrontendfpdffuturefuzzywuzzygensim==3.8.2gradiographvizh5pyhtml5libhttpxIMAPClientimageioimageio-ffmpegimgkitipythonJinja2jinja2jiebajson5jsonpicklejsonschemajupyter-clientjupyter-corejupyter-serverjupyterlabjupyterlab-pygmentsjupyterlab-serverkeraslangchainlangchain-experimentallibrosalogurulxmlmarkdown2markdownifyMarkupSafematplotlibmatplotlib-inlinematplotlib-vennmoviepymurmurhashnbclientnbconvertnbformatnetworkxnltknotebooknumbanumexprnumpynumpy-financialopenaiopencv-pythonopenpyxlorjsonpandaspdf2imagepdfkitpdfminer.sixpdfplumberPillowpillowplotlypsutilPyAudioPyMuPDFpydanticpydubPygmentspygmentspylogPyMuPDFpymupdfpypandocpyparsingPyPDF2pypdf2==2.12.0pytesseractpytestpython-dateutilpython-docxpython-multipartpython-pptxpytzPyWaveletspywaveletspygamePyYAMLpyyamlqrcoderarfilerequestsscikit-imagescikit-learnscipyseabornsentencepieceShapelysoundfileSoundFileSpeechRecognitionspeechrecognitionstarlettestatsmodelssvglibsvgwritesympysumytabulatabulatetextblobtifffiletomltorchtorchaudiotorchtexttorchvisiontornadotqdmtyping-extensionstzdatatzlocalujsonurllib3uvicornWandwandwebsocket-clientwebsocketsWerkzeugwerkzeugwordcloudxgboostxlrdXlsxWriterxlsxwriterxml-pythonzipp