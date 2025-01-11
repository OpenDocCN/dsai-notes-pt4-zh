<!--yml
category: 未分类
date: 2025-01-11 11:48:45
-->

# Can Modern LLMs Act as Agent Cores in Radiology Environments?

> 来源：[https://arxiv.org/html/2412.09529/](https://arxiv.org/html/2412.09529/)

Qiaoyu Zheng Shanghai Jiao Tong University Shanghai AI Laboratory Chaoyi Wu Shanghai Jiao Tong University Shanghai AI Laboratory Pengcheng Qiu Shanghai Jiao Tong University Shanghai AI Laboratory Lisong Dai Department of Radiology, Renmin Hospital of Wuhan University  Ya Zhang Shanghai Jiao Tong University Shanghai AI Laboratory Yanfeng Wang Shanghai Jiao Tong University Shanghai AI Laboratory Weidi Xie Shanghai Jiao Tong University Shanghai AI Laboratory 

###### Abstract

Advancements in large language models (LLMs) have paved the way for LLM-based agent systems that offer enhanced accuracy and interpretability across various domains. Radiology, with its complex analytical requirements, is an ideal field for the application of these agents. This paper aims to investigate the pre-requisite question for building concrete radiology agents which is, ‘Can modern LLMs act as agent cores in radiology environments?’ To investigate it, we introduce RadABench with three-fold contributions: First, we present RadABench-Data, a comprehensive synthetic evaluation dataset for LLM-based agents, generated from an extensive taxonomy encompassing 6 anatomies, 5 imaging modalities, 10 tool categories, and 11 radiology tasks. Second, we propose RadABench-EvalPlat, a novel evaluation platform for agents featuring a prompt-driven workflow and the capability to simulate a wide range of radiology toolsets. Third, we assess the performance of 7 leading LLMs on our benchmark from 5 perspectives with multiple metrics. Our findings indicate that while current LLMs demonstrate strong capabilities in many areas, they are still not sufficiently advanced to serve as the central agent core in a fully operational radiology agent system. Additionally, we identify key factors influencing the performance of LLM-based agent cores, offering insights for clinicians on how to apply agent systems in real-world radiology practices effectively. All of our code and data are open-sourced in [https://github.com/MAGIC-AI4Med/RadABench](https://github.com/MAGIC-AI4Med/RadABench).

## 1 INTRODUCTION

Recent advancements in Large Language Models (LLMs) have revolutionized numerous domains in artificial intelligence (AI), from natural language processing [[1](https://arxiv.org/html/2412.09529v2#bib.bib1), [2](https://arxiv.org/html/2412.09529v2#bib.bib2), [3](https://arxiv.org/html/2412.09529v2#bib.bib3)] to computer vision [[4](https://arxiv.org/html/2412.09529v2#bib.bib4), [5](https://arxiv.org/html/2412.09529v2#bib.bib5), [6](https://arxiv.org/html/2412.09529v2#bib.bib6)]. Models such as GPT-4, Gemini, and LLaMA have pushed the boundaries of machine understanding and generation [[3](https://arxiv.org/html/2412.09529v2#bib.bib3), [7](https://arxiv.org/html/2412.09529v2#bib.bib7), [8](https://arxiv.org/html/2412.09529v2#bib.bib8)]. Among these breakthroughs, the emergence of LLM-powered agent systems [[9](https://arxiv.org/html/2412.09529v2#bib.bib9), [10](https://arxiv.org/html/2412.09529v2#bib.bib10), [11](https://arxiv.org/html/2412.09529v2#bib.bib11), [12](https://arxiv.org/html/2412.09529v2#bib.bib12), [13](https://arxiv.org/html/2412.09529v2#bib.bib13), [14](https://arxiv.org/html/2412.09529v2#bib.bib14)] is particularly noteworthy. These systems highlight the ability of a central agent core to interact with external tools, enabling complex, multi-step tasks and demonstrating remarkable promise in domains such as customer service, business automation, and creative content generation. This success is attributed to a blend of precision, interpretability, and scalability that amplifies the utility of LLMs in real-world applications.

Despite these transformative gains, the integration of LLMs into clinical settings—specifically in radiology—remains at an early stage [[15](https://arxiv.org/html/2412.09529v2#bib.bib15), [16](https://arxiv.org/html/2412.09529v2#bib.bib16)]. Radiology is central to medical diagnostics, requiring the interpretation of detailed textual reports and complex visual data from medical images [[17](https://arxiv.org/html/2412.09529v2#bib.bib17), [18](https://arxiv.org/html/2412.09529v2#bib.bib18), [19](https://arxiv.org/html/2412.09529v2#bib.bib19), [20](https://arxiv.org/html/2412.09529v2#bib.bib20)]. Recently, radiology generalist models [[4](https://arxiv.org/html/2412.09529v2#bib.bib4), [21](https://arxiv.org/html/2412.09529v2#bib.bib21), [22](https://arxiv.org/html/2412.09529v2#bib.bib22), [23](https://arxiv.org/html/2412.09529v2#bib.bib23), [24](https://arxiv.org/html/2412.09529v2#bib.bib24), [25](https://arxiv.org/html/2412.09529v2#bib.bib25), [26](https://arxiv.org/html/2412.09529v2#bib.bib26)] have shown promise in handling diverse radiological analyses within a single framework. However, the complexity of radiology—encompassing diverse imaging modalities, varied pathologies, and intricate interpretive criteria—may exceed the capabilities of any single all-purpose model. This complexity makes radiology an ideal domain for agent-based systems, where specialized tools (or models) can work collaboratively to deliver more robust and precise analyses [[27](https://arxiv.org/html/2412.09529v2#bib.bib27)].

This study addresses a fundamental prerequisite for developing agent-based radiology analysis systems. Specifically, we ask:

> Can existing LLMs effectively interact with the radiological environment—understanding professional medical tool descriptions, accurately translating diverse clinical queries into actionable steps, and invoking tools to execute sub-tasks sequentially?

To explore this question, we introduce RadABench (Radiology Agent Benchmark), that serves as a comprehensive resource for benchmarking LLM-based agents in radiology. Specifically, it is characterized by three main contributions: (1) We present a novel dataset designed specifically for evaluating LLM-based agents in radiology, consisting synthetic patients, tools, and tasks. (2) We develop a novel evaluation platform tailored for assessing radiology-specific LLM agents. This platform establishes a general prompt-driven agent workflow and simulates a wide range of external radiology tool sets with variable and dynamic conditions. (3) We provide a comprehensive performance analysis of 7 state-of-the-art LLMs, including both closed-source models (GPT-4, GPT-4o, Gemini, Claude) and open-source counterparts (LLaMA, Mixtral, Qwen). Our systematically designed metrics evaluate five key capabilities: chain planning, optimal tool orchestration, IO (input/output) organization, response synthesis, and unsolvability parsing.

The constructed dataset, termed as RadABench-Data, follows a systematic approach to ensure extensive coverage. We first develop a thorough taxonomy, categorizing patient records across 22 common anatomy-modality combinations derived from 6 anatomical regions (e.g., chest, brain, spine) and 5 imaging modalities (e.g., X-ray, CT, MRI). For each anatomy-modality pair, we select 100 common diseases, resulting in 2,200 patient records, each representing a distinct radiology-related condition. Additionally, we define 10 commonly used radiology tool categories and incorporate 11 radiology task decomposition meta-chains, systematically generating 24,200 QA pairs. All data undergoes clinical validation by radiologists to ensure accuracy and representativeness.

RadABench-EvalPlat is a specialized evaluation platform designed to simulate complex clinical scenarios that mirror the decision-making processes of practicing radiologists. The platform incorporates a prompt-driven, three-stage workflow to systematically assess LLMs’ abilities as agent cores, focusing on their ability to interpret clinical queries, select appropriate tools, and manage task execution in these diverse and challenging scenarios. Moreover, we design a dynamic tool set simulation strategy, that captures the diverse challenges faced in real-world radiology conditions, covering 10 distinct tool categories, including organ segmentation, disease diagnosis, report generation, and other essential radiology tasks.

To rigorously assess the performance of 7 leading LLMs as agent cores, we employ an evaluation framework emphasizing five key aspects: chain planning (comparing predicted and ground-truth planning), optimal orchestrating (assessing appropriate tool selection), IO organizing (ensuring correct input/output formatting), response synthesizing (evaluating the quality of generated responses), and unsolvability parsing (identifying when tasks cannot be solved). For each perspective, we design dedicated metrics to ensure a thorough and multi-dimensional evaluation.

While state-of-the-art models like GPT-4o and Claude 3.5-Sonnet excel in certain straightforward tasks, our analysis reveals significant gaps when confronted with more complex clinical scenarios. These findings underscore a substantial gap between current LLM capabilities and the stringent requirements of real-world radiology applications. They also highlight broader challenges in developing agent-based radiology systems. To accelerate progress, we have made our datasets and evaluation codes publicly available.

## 2 RESULTS

In this section, we present the RadABench framework and benchmark in detail. We begin by introducing RadABench-Data, providing a comprehensive overview of its scope and coverage across diverse radiology cases. Next, we detail the RadABench-EvalPlat, outlining its core workflow and dynamic tool set simulation strategy. Together, these ensure a clear understanding of the evaluation conditions and criteria. Finally, we report quantitative results for 7 leading LLMs, examining their performance from multiple perspectives—decision planning, execution precision, rejection awareness, selection optimization, and response synthesis—and highlight the key strengths, limitations, and trends revealed by our benchmarks.

### 2.1 Introduction to RadABench-Data

RadABench-Data encompasses a comprehensive radiology taxonomy that includes patient records, tools, and tasks. Based on this taxonomy, we simulate 2,200 synthetic patient records, and use QA pairs to benchmark the ability of LLMs acting as agent cores. Below, we detail each data component.

Analysis on Patient Record

A “patient record” here refers to a radiology-centric medical record that includes patient demographics, medical history, annotated imaging data (with anatomical and pathological details), and extensive clinical findings. We categorize patient records by anatomy, modality, and disease. This taxonomy covers 22 common anatomy-modality combinations derived from 6 anatomical regions and 5 imaging modalities:

> Head and Neck: {X-ray, CT, MRI, Ultrasound}; Chest: {X-ray, CT, MRI, Ultrasound};
> Limb: {X-ray, CT, MRI, Ultrasound}; Abdomen and Pelvis: {X-ray, CT, MRI, Ultrasound};
> Spine: {X-ray, CT, MRI}; Breast: {Mammography, MRI, Ultrasound}.

For each combination, we select 100 common diseases detectable by the corresponding imaging modality. Using GPT-4, we generate one patient record per leaf node, resulting in 2,200 records (22 anatomy-modality combinations $\times$ 100 diseases). A radiologist with over 10 years of experience verified each record. Details on generation and verification can be found in the “METHODS” section. An example patient record is provided in the supplementary file.

![Refer to caption](img/9dfecd968f654945eccd5427feddd301.png)

Figure 1: Statistics in RadA-Bench.  a. Patient sex coverage on different anatomies. b-d. Patient records distribution on six anatomies in age/height/weight ranges. e. Features distribution on anomaly/disease/biomarker/indicator extracted by BioLORD. f. 11 task decomposition chains, including tool category coverage, tool chain length, and token/character numbers. g. Available tool number ranges in 8 simulated conditions. h. Context token length in 8 simulated conditions using GPT-4o/Llama-3.1-405B as the agent core. i. The max token length of different open-source and closed-source LLMs.

Figures [1](https://arxiv.org/html/2412.09529v2#S2.F1 "Figure 1 ‣ 2.1 Introduction to RadABench-Data ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")a-d show the distributions of these synthetic patient records across various demographic factors, including sex, age, height, and weight. Notably, while the dataset is generally balanced by sex, breast-related cases are predominantly female due to their higher clinical incidence. Age distributions center around middle-aged groups (35-70 years), reflecting typical clinical patterns. Height and weight distributions follow normal patterns, further confirming the dataset’s realism and diversity.

Figure [1](https://arxiv.org/html/2412.09529v2#S2.F1 "Figure 1 ‣ 2.1 Introduction to RadABench-Data ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")e visualizes anomalies, diseases, biomarkers, and indicators within patient records. Here, “biomarkers” refer to imaging features (e.g., dimensions, textures), while “indicators” correspond to clinical classifications (e.g., cancer staging) or scoring systems (e.g., CURB-65). We embed these attributes using BioLORD [[28](https://arxiv.org/html/2412.09529v2#bib.bib28)] and plot them via t-SNE, revealing a uniform distribution and confirming that our dataset covers a wide range of clinical scenarios.

In summary, the synthetic patient records in RadABench-Data demonstrate robust clinical diversity and closely align with real-world medical contexts. These records form a strong foundation for evaluating LLM agent cores in managing complex radiology tasks.

Analysis on Radiology Tools

In alignment with clinical practices, we categorize radiology tools into 10 high-level categories (details provided in the supplementary file): Anatomy Classifier (AC), Modality Classifier (MC), Organ Segmentor (OS), Anomaly Detector (AD), Imaging Diagnoser (ID), Synthetic Diagnoser (SD), Biomarker Quantifier (BQ), Indicator Evaluator (IE), Report Generator (RG), and Treatment Recommendation (TR).

Each tool is described using a “tool card” that outlines its category, property, ability, compulsory input, optional input, and performance. Each component is assigned values by randomly sampling from clinically reasonable ranges. For example, the category might be “anatomy classifier”, and the performance value could be a numerical score from 0 to 100\. This approach allows us to theoretically generate an infinite number of tools, covering a broad spectrum of radiological scenarios and ensuring that the evaluation remains challenging. Details on the generation process are provided in the “METHODS” section.

Analysis on Radiology Tasks

To comprehensively assess LLM-based agents, we develop a radiology task taxonomy comprising 11 distinct tasks (explained in detail in the supplementary file): (a) organ mask annotation, (b) anomaly region annotation, (c) organ-wise biomarker calculation, (d) anomaly-wise biomarker calculation, (e) organ-and-anomaly-wise image interpretation, (f) disease diagnosis without visual clues, (g) disease diagnosis with visual clues, (h) common report generation (i) report generation focusing on specific biomarkers (j) report generation focusing on specific biomarkers and indicators, (k) treatment planning. We further break down each task into a chain of tool-category steps (as shown in Figure [1](https://arxiv.org/html/2412.09529v2#S2.F1 "Figure 1 ‣ 2.1 Introduction to RadABench-Data ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")f). Each task involves 3 to 9 tool categories, reflecting varying levels of complexity. Tasks with more involved tool categories demand more sophisticated reasoning and planning.

For each of the 2,200 patient records, we create a simulation instance for each of the 11 tasks, resulting in 24,200 QA pairs (2,200 records $\times$ 11 tasks). For example, a patient record might prompt the question “What disease can be inferred?” for a diagnosis task or “Please write a radiologic report for the image.” for a report generation task. These QA pairs are uniformly distributed across all tasks, ensuring balanced representation and diversity. All QA pairs undergo manual verification. The details of the generation and verification process are provided in the “METHODS” section.

### 2.2 Introduction to RadABench-EvalPlat

In this section, we introduce RadABench-EvalPlat, starting with its main workflow formulation, and then describing our dynamic tool set simulation strategy for comprehensive evaluation.

![Refer to caption](img/e2361d5afd1c9c15dd94a18a420610f9.png)

Figure 2: An overview of our evaluation platform. We construct 4 categories and 8 specific medical conditions by dynamically combining 10 different tool categories and their corresponding attributes. Based on various tool sets, we develop a 3-stage agent core workflow to evaluate LLMs. The quality of the agent core’s responses is evaluated across 5 capability dimensions through targeted analysis.

Main Workflow Formulation

Generally speaking, a typical LLM-based agent system comprises three main components: an LLM-based agent core ($\Phi$), an external set of specialized tools ($\mathcal{T}={t_{1},t_{2},\ldots,t_{m}}$), and a memory bank ($\mathcal{B}$). The agent core orchestrates the workflow, selecting and executing a chain of appropriate tools, while the memory bank stores intermediate outputs and contextual information. Given a clinical query $\mathcal{Q}$ and the patient’s historical information, the system follows a three-step process to produce the final response:

*   •

    Task Decomposition. The agent core ($\Phi$) begins by breaking down the clinical query into a series of manageable sub-tasks aligned with the capabilities of the tools in $\mathcal{T}$. Simultaneously, the memory bank $\mathcal{B}$ is initialized with critical information—such as patient demographics, medical history, and the outputs of each tool execution—ensuring that relevant data remains accessible throughout the process.

*   •

    Tool Selection & Execution. Guided by the decomposed tasks and the context stored in $\mathcal{B}$, the agent core selects the most suitable tool for each sub-task and prepares its input. Once a tool produces its output, the result is stored in $\mathcal{B}$. After each iteration, the agent core assesses whether the overall query has been sufficiently addressed. If so, the process terminates; otherwise, it continues selecting and executing tools until the task is complete.

*   •

    Response Generation. Finally, the agent core synthesizes all relevant historical data stored in $\mathcal{B}$, structuring the information to address the clinical query. By integrating the outputs of individual tools and placing them in the context of the original query, the agent core produces a coherent and actionable final response.

A Detailed Example. Consider a clinical scenario where the agent system is tasked with providing a diagnosis based on a patient’s demographics (e.g., age, weight, chief complaints), radiology examination scans, and a free-text diagnostic query such as: “Please make a diagnosis based on all aspects of the patient’s information.”

During the Task Decomposition stage, the agent core divides this query into four sub-steps: identifying the anatomical region and imaging modality, applying organ segmentation and anomaly detection tools, and ultimately inferring disease types. Next, in the Tool Selection & Execution stage, the agent systematically selects the most appropriate tools from the set $\mathcal{T}$. Each tool’s output is recorded in the memory bank $\mathcal{B}$, allowing the agent to iteratively refine the solution until the subtasks are completed. In the final Response Generation stage, the agent integrates the patient details, segmentation results, and diagnostic inferences stored in $\mathcal{B}$ to produce a coherent and comprehensive response.

Our objective in this study is to evaluate how effectively various LLMs can serve as central agent cores within such a workflow, controlling multiple tools and tackling diverse radiology tasks. In the following sections, we describe how we assemble the tool sets necessary for this evaluation.

Dynamic Tool Set Simulation

Our goal in the dynamic tool set simulation is to ensure that, during evaluation, our considered tool sets can reflect various real-world clinical situations. For example, these may include situations where the necessary tools are missing, or where multiple tools can serve for similar tasks with different performances.

Specifically, given a certain patient record and a specific query, we adhere to the following principles to form different evaluation conditions:

*   •

    NS. (Necessary and Sufficient) Conditions. Related tool sets comprise a minimal set of tools containing exactly one comprehensive tool for each tool category, ensuring the user query can be solved by tools. This Tests the agent’s basic tool chain planning and execution abilities.

*   •

    SNN. (Sufficient but Not Necessary) Conditions. Related tool sets are extended beyond what is necessary for the query by including redundant tools. The condition can be further divided into three levels of redundancy—Regular, Medium, and Large—with increasing redundant tool numbers. This assesses the agent’s ability to select relevant tools in a noisy environment.

*   •

    NR. (Necessary but not Sufficient) Conditions. Related tool sets lack the essential tools required to complete the query. It has three difficulty levels: Denyl1, missing tool categories; Denyl2, missing modality/anatomy-specific tools; Denyl3, missing required detailed disease/abnormalities. This assesses the agent’s ability to recognize limitations and respond appropriately when critical tools are unavailable.

*   •

    OPT. (Optimal) Conditions. Related tool sets provide multiple tools with overlapping capabilities but varying performance levels and the agent core is expected to pick out the better tools for execution. This assesses the agent’s ability to select the best-performing tools from similar alternatives.

This simulation approach ensures that the dynamically generated tool sets comprehensively reflect various evaluation conditions, effectively assessing different capabilities of LLM agent cores. In the “METHODS Section”, we will introduce more implementation details for each condition.

In Figure. [1](https://arxiv.org/html/2412.09529v2#S2.F1 "Figure 1 ‣ 2.1 Introduction to RadABench-Data ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")g, we show the scale of simulated tool sets in different conditions. For the NS. Condition, as shown in the figure, regardless of different patient records and task queries, the tool set size remains constant. Specifically, this condition has a fixed set of 12 tools covering all 10 tool categories, with biomarker quantifier and indicator evaluator categories each having two tools (one for organ and one for anomaly evaluation). For the three SNN. Conditions, Regular and Medium levels vary in tool numbers based on specific patient records and queries - Regular includes necessary tools plus at most one additional unusable tool per category (12-15 tools), while Medium adds 2-3 extra tools beyond task-specific modality and anatomy per category (27-34 tools). In contrast, the Large condition maintains a fixed (though large) set size by including all possible tool combinations across modalities, anatomies, and tool optional inputs, regardless of specific patient records and tasks. For the three NR. Conditions, we construct unsolvable tool sets based on different patient records and types of QA tasks. In Denyl1 and Denyl2, the tool numbers vary (14-17 tools) depending on which essential components are missing - for example, in Denyl1, removing organ segmentation tools would make organ biomarker quantification tasks impossible to complete. In contrast, Denyl3 maintains a fixed size (18 tools) as it focuses on modifying specific tool capabilities rather than removing entire tools or categories. In the OPT Condition, we create multiple tools with different performance levels for certain categories based on specific tasks (*e.g.*, diagnosis tools with varying accuracy rates). This results in a larger tool set (17-18 tools) compared to the NS. Condition, with tool selection options varying according to different tasks. These statistics can implicitly indicate the tool set understanding complexity in different conditions.

### 2.3 Quantitative Analysis on RadABench

In this section, we present the main quantitative results on various LLMs serving as the agent cores. This analysis focuses on their ability to decompose complex radiology imaging tasks and effectively utilize diverse medical tools to generate accurate and optimal responses. The evaluation is conducted across the following key dimensions:

Analysis on Response Token Length

First, we analyze the response token lengths of different LLMs to assess whether the context window limitations of modern models may impact their use as agent cores in the radiology environment.

As shown in Figure [1](https://arxiv.org/html/2412.09529v2#S2.F1 "Figure 1 ‣ 2.1 Introduction to RadABench-Data ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")h, we present the multi-turn response token lengths generated by two LLMs, GPT-4o and Llama-3.1, representing closed-source and open-source models, respectively. In most conditions, the total token lengths for both models are comparable, ranging from 3,000 to 30,000 tokens. However, in the NR. Deny conditions, GPT-4o exhibits significantly longer context lengths than Llama, primarily due to Llama’s frequent failure to execute proper denial responses, resulting in premature terminations. Figure [1](https://arxiv.org/html/2412.09529v2#S2.F1 "Figure 1 ‣ 2.1 Introduction to RadABench-Data ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")k further compares the maximum supported token lengths of different LLMs with the highest token demands observed in our conditions. While GPT-3.5, a previous-generation model, struggles to accommodate the full context length, all other state-of-the-art models handle it effectively.

These findings demonstrate that the context windows of the modern LLMs are sufficiently large to support agent-based applications. This ensures that our subsequent evaluation results are not influenced by excessively long contexts and confirms the reliability of our analysis with the capabilities of current LLMs.

![Refer to caption](img/913b99f01479653d57a4a30d1d37e0ac.png)

Figure 3: Evaluation results under NS. Condition and SNN. Conditions.  a. 3 Levenshtein Distance between pairs of decision, execution, and ground truth tool chains under NS. Condition among 7 LLMs b-c. Levenshtein distance, False Discovery Rate, Tool Match Rate, Exectution Completion Rate, Pre-Failure Success Percentage values under SNN. Regular Condition among LLMs. d-e. Same metrics under SNN. Medium Condition. f-g. Same metrics under SNN. Large Condition.

Analysis on Chain Planning Ability

Here, to analyze agent core’s ability to decompose complex tasks into appropriate tool chains and maintain planning consistency through multiple iterations, we adopt Levenshtein Distance (LD), that measures the minimal edit steps between generated and ground-truth chains. False Discovery Rate (FDR) denotes the proportion of incorrectly included tools, and Tool Matching Accuracy (TMA) evaluates the position-wise matching ratio between generated and ground-truth chains. This evaluation is under NS. tool set condition and SNN. tool set conditions.

As shown in Figure. [3](https://arxiv.org/html/2412.09529v2#S2.F3 "Figure 3 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?"), we can make the following observations: 1) closed-source models generally outperform open-source models (except Llama-3.1-405B) across both NS. and SNN. Conditions, with closed-source models achieving LD between planning and ground truth less than 1.5 (average 1.5 edits needed to match ground truth chains) in NS. Condition as shown in Figure. [3](https://arxiv.org/html/2412.09529v2#S2.F3 "Figure 3 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")a - this represents relatively good performance given the average chain length of 5 step; 2) as shown in Figure. [3](https://arxiv.org/html/2412.09529v2#S2.F3 "Figure 3 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")a, differences emerge between planned and executed chains during multi-iteration execution, where models can adjust tool selections - Claude-3.5 and LLama-3.1 show convergence toward ground truth chains, while others maintain or increase deviations; 3) as shown in Figure. [3](https://arxiv.org/html/2412.09529v2#S2.F3 "Figure 3 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")c, e and g, GPT-4o demonstrates lower FDR across SNN. Regular, Medium and Large Conditions compared to other LLMs, indicating lower tool selection redundancy; 4) GPT-4o and Claude-3.5 achieve higher TMA compared to other LLMs, indicating superior position-wise alignment with ground truth tool chains at each step of the planning process.; 5) when comparing Figure. [3](https://arxiv.org/html/2412.09529v2#S2.F3 "Figure 3 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")b, d, and f, as the complexity of tool set condition increases, all LLMs show significant performance decline, particularly open-source models.

![Refer to caption](img/34e55ff6f5ec27560c65126e1b648c5e.png)

Figure 4: Evaluation results under SNN. Conditions and OPT. Condition.  a. Target Hit Rate under SNN. Regular, SNN. Medium and SNN. Large Conditions among 7 LLMs. b. Tool chain-level results including Optimal Tool Socre and Milestone Hit Rate under OPT. Condition among 7 LLMs. c. Free text Final response-level results including F1, BLEU and ROUGE scores under OPT. Condition among 7 LLMs.

Analysis on Optimal Orchestrating Ability

To evaluate the agent core’s ability on selecting optimal tools from multiple candidates, we adopt Optimal Tool Score (OTS), that compares the performance rank of chosen tools with available alternatives. Evaluation is under OPT. tool set condition.

From Figure. [4](https://arxiv.org/html/2412.09529v2#S2.F4 "Figure 4 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")b, we can observe that most LLMs exhibit competence in selecting optimal tools and successfully executing core task steps, under OPT. Condition. In particular, Gemini-1.5-Pro achieves the best performance with 0.720 on OTS, while the open-source Qwen-2.5 also delivers a comparable result of 0.701, indicating their superior ability to select higher-performing tools when multiple options are available.

Analysis on IO Organizing Ability

To analyze the agent core’s ability for managing data flow between sequential tools during execution, we adopt Execution Completion Rate (ECR) which measures chain completion without IO errors, and Pre-Failure Success Percentage (PFSP) which calculates the progress ratio before failure in unsuccessful cases. The evaluation here is under SNN. tool set condition.

We make the following observation on data flow management: 1) as shown in Figure. [3](https://arxiv.org/html/2412.09529v2#S2.F3 "Figure 3 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")c and d, Claude-3.5-sonnet achieves superior ECR across all three SNN. Conditions (0,795 and 0.737 for Regular and Medium conditions respectively), particularly, it demonstrates notably better execution reliability compared to other LLMs; 2) in SNN. Regular and SNN. Medium Conditions, half of the LLMs achieve PFSP scores above 0.5, indicating that even when they fail to complete the entire task, they can successfully execute a significant portion of the tool chain before encountering failures; 3) as shown in Figure. [3](https://arxiv.org/html/2412.09529v2#S2.F3 "Figure 3 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")e, as tool set context increases, all LLMs show a sharp performance drop on both ECR and PFSP metrics. particularly in the Large condition, suggesting significant challenges in managing data flows in complex tool set conditions; 3) Mixtral demonstrates consistently poor performance across all three conditions, indicating limited capability in precisely organizing and managing input-output relationships between tools.

Analysis on Response Synthesis

To analyze agent core’s ability to follow recommended solution paths and generate appropriate responses, we adopt Target Hit Rate (THR) to measure whether the final tool aligns with the required answer format, Milestone Hit Rate (MHR) to evaluate valuable intermediate outputs, and multiple text similarity metrics including BLEU, ROUGE and F1 to assess the quality match between generated and reference outputs from different aspects. This evaluation is under SNN. tool set conditions and OPT. tool set condition.

From our analysis, we can draw the following observations: 1) from Figure. [4](https://arxiv.org/html/2412.09529v2#S2.F4 "Figure 4 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")a, we find that Claude-3.5-sonnet achieves significantly better THR than other open-source models across three SNN.  Conditions, with scores of 0.519, 0.450, and 0.231 respectively, demonstrating its superior ability to maintain adherence to target objectives compared to other LLMs; 2) Even Claude-3.5, the top-performing model, only succeeds in 51.9% of tasks under the simple SNN. Regular Condition. This low target hit rate may stems from two issues: agents often deviate from ground truth tool sequences, failing to reach the correct final tool selection, and even when the planning is correct, input/output errors during tool execution can prevent task completion; 3) for the MHR metric shown in Figure. [4](https://arxiv.org/html/2412.09529v2#S2.F4 "Figure 4 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")b, Claude-3.5, GPT-4o, and Qwen-2.5 show similar and superior performance in OPT Condition compared to other LLMs, reaching values close to 0.8, demonstrating their strong ability in identifying and executing critical steps during the task-solving process; 4) as shown in Figure. [4](https://arxiv.org/html/2412.09529v2#S2.F4 "Figure 4 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")c, all LLMs demonstrate relatively low BLEU, F1 and ROUGE performance (with scores rarely exceeding 0.35) in the final generated responses, attributing to both missing or redundant steps in tool planning and execution chains (as evident by relatively low THR and MHR), while also indicating the room for improvement in synthesizing intermediate outputs into coherent and accurate conclusions.

![Refer to caption](img/d16d281a1b1369cc25f9e663a0bedbdf.png)

Figure 5: Evaluation results under NR. Conditions.  a. Unsolvable Awareness Rate under NR. Denyl1, NR. Denyl2 and NR. Denyl3 Conditions among 7 LLMs. b. Unsolvable Grounding Rate under NR. Denyl1, NR. Denyl2 and NR. Denyl3 Conditions among 7 LLMs.

Analysis on Unsolvability Parsing

To analyze agent core’s ability to recognize and respond to unsolvable cases, we adopt Unsolvability Awareness Rate (UAR), to measure the ability to identify impossible tasks, and Unsolvability Grounding Rate (UGR), to evaluate the ability to pinpoint specific missing tools or capabilities. This evaluation is under NR. conditions. The test conditions are designed with increasing difficulty: “Denyl1” involves the absence of an entire tool category, “Denyl2” denotes the absence of specific tools for certain anatomical modalities, and “Denyl3” involves insufficient tool ability rather than complete tool absence. This progressive difficulty increase is also reflected in the performance of these LLMs. As shown in Figure. [5](https://arxiv.org/html/2412.09529v2#S2.F5 "Figure 5 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")a, Claude-3.5-Sonnet and GPT-4o show best performances on UARs under thre NR. Conditions, which are around 0.6, this is an impressive value indicating their ability to aware unsolvabilities.

Success in these conditions is defined by the ability to not only recognize unsolvable tasks, but also to propose supplemental tools or strategies for continuation. For instance, models are expected to identify missing tool categories in “Denyl1”, specify the required modalities and anatomical regions in “Denyl2”, and define the minimum tool capability requirements in “Denyl3”. Under these strict criteria, as shown in Figure. [5](https://arxiv.org/html/2412.09529v2#S2.F5 "Figure 5 ‣ 2.3 Quantitative Analysis on RadABench ‣ 2 RESULTS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?")b, UGR obviously declines for some models, such as Gemini-1.5-Pro, while others like Claude-3.5-Sonnet and GPT-4o demonstrate relatively strong performance, reflecting superior comprehension capabilities. Conversely, GPT-4-Turbo, Llama-3.1-405B, Mixtral, and Qwen-2.5 consistently attempt to solve tasks with inappropriate tools, highlighting their challenges in discerning the feasibility of given tools for solving certain task.

## 3 DISCUSSION

Recent studies [[29](https://arxiv.org/html/2412.09529v2#bib.bib29), [12](https://arxiv.org/html/2412.09529v2#bib.bib12), [30](https://arxiv.org/html/2412.09529v2#bib.bib30), [11](https://arxiv.org/html/2412.09529v2#bib.bib11), [31](https://arxiv.org/html/2412.09529v2#bib.bib31), [20](https://arxiv.org/html/2412.09529v2#bib.bib20)] highlight the promising capabilities of LLMs to build agent system for solving general problems through effective tool utilization. In the medical community, research [[32](https://arxiv.org/html/2412.09529v2#bib.bib32), [33](https://arxiv.org/html/2412.09529v2#bib.bib33)] underscores the potential of LLMs in clinical settings, demonstrating their ability to support decision-making, facilitate interactions, and invoke tools for applications, ranging from clinical workflow automation to multi-agent-aided diagnosis. However, for radiology-specific tasks, prior studies [[34](https://arxiv.org/html/2412.09529v2#bib.bib34), [35](https://arxiv.org/html/2412.09529v2#bib.bib35)] reveal significant challenges, with even state-of-the-art vision-language models (VLMs) such as GPT-4V [[36](https://arxiv.org/html/2412.09529v2#bib.bib36)] struggling to perform reliable medical image analysis.

Existing LLMs agent in medical field has been limited by targeting specific clinical tasks, for example, diagnosis [[37](https://arxiv.org/html/2412.09529v2#bib.bib37)], risk prediction [[11](https://arxiv.org/html/2412.09529v2#bib.bib11)], or particular imaging modalities and anatomical regions, for example, brain-focused studies [[31](https://arxiv.org/html/2412.09529v2#bib.bib31), [20](https://arxiv.org/html/2412.09529v2#bib.bib20)]. These limitations restrict their applicability to the broader and more complex landscape of radiology. In this paper, we aim to bridge this gap by comprehensively evaluating the abilities of existing large language models (LLMs) to act as core agents in managing complex radiology tasks. This involves leveraging and integrating specialized radiology imaging interpretation and analysis tools, mimicking the workflow of real clinicians for handling diverse and intricate clinical queries.

### 3.1 Research Impacts

Our research contributions can be summarized in three aspects:

First, we introduce RadABench-Data, a comprehensive dataset for evaluating radiology analysis. It includes 2200 patient records of different anatomy-modality-disease combinations, spanning 11 clinical tasks and 10 high-level tool categories, capturing a wide spectrum of radiological scenarios. This dataset provides a robust benchmark for testing agent systems across a diverse set of radiological tasks.

Second, we develop RadABench-EvalPlat, an automated evaluation platform designed to simulate real-world clinical settings, incorporating an interactive prompt system to facilitate iterative task execution. This platform dynamically generates radiology tool sets tailored to specific evaluation requirements. It supports a range of clinical conditions, from simple, well-defined tasks to complex, resource-constrained ones. It offers a flexible and realistic environment for testing the adaptability and robustness of LLM-based agent cores.

Third, on the final RadABench, we benchmark the performance of seven state-of-the-art LLMs as agent cores in radiology. We evaluate their abilities across three crucial dimensions: task decomposition and tool selection, tool execution and input-output coordination, response synthesis and integration. To address the challenges for evaluating free-text outputs, we propose a comprehensive set of metrics that assess both task-specific and overall system performance. Our findings reveal the strengths and limitations of existing LLMs, providing insights into their readiness for deployment in clinical workflows.

All data and code in this study are openly available, allowing researchers to benchmark new models and advance the field. By addressing the challenges of tool integration, task decomposition, and real-world scenario simulation, this work establishes a foundation for improving LLM-based systems in radiology and beyond.

### 3.2 Clinical Impacts

Our experimental findings reveal a critical insight: existing LLMs remain far from being reliable as core agents for radiology applications. While these models perform well in relatively simple tasks—particularly when tool availability is complete and task complexity is limited—they struggle significantly in more complex scenarios, such as those involving incomplete or inconsistent tool sets. This shortfall underscores a pressing concern: despite the claims in prior studies [[38](https://arxiv.org/html/2412.09529v2#bib.bib38), [9](https://arxiv.org/html/2412.09529v2#bib.bib9), [10](https://arxiv.org/html/2412.09529v2#bib.bib10), [11](https://arxiv.org/html/2412.09529v2#bib.bib11), [12](https://arxiv.org/html/2412.09529v2#bib.bib12), [13](https://arxiv.org/html/2412.09529v2#bib.bib13)] that agent systems can enhance the robustness and reliability of LLM outputs, their trustworthiness in medical, and particularly radiology, remains questionable and necessitates careful clinician oversight.

We identify several key factors that limit the performance and real-world adoption of LLM-based agent systems in radiology:

Challenges in understanding complex external tools: LLMs struggle to interpret and apply instructions involving long and detailed contextual descriptions. This limitation is particularly problematic in radiology, where tool instructions and diagnostic criteria often require sustained coherence and nuanced understanding.

Inefficiencies in synthesizing multi-round information: Performance degrades markedly as response rounds increase. This limits the models’ ability for iterative diagnostic processes and longitudinal patient monitoring, both of which demand consistent tracking and integration of information.

LLMs are prone to significant “tool incomplete hallucinations”: When working with external tools, LLMs often generate erroneous or incomplete outputs—referred to as “hallucinations”—especially when these tools are not fully integrated or accessible. These hallucinations can mislead clinicians or undermine the trustworthiness of the AI system in clinical decision-making, a crucial concern in high-stakes medical environments.

LLMs struggle with organizing strict IO formats for successive tools: LLMs often fail to precisely follow complex instructions, especially those that require systematic organization of IO to link different tools. In radiology, where diagnostic workflows may involve multiple stages, for example, imaging analysis, report generation, and treatment recommendation, LLMs are unable to strictly organize tasks and link tools in a coherent manner.

LLMs often fail to select the most appropriate tools based on their performance: A key aspect of effective agent systems in radiology is the ability to evaluate and select the best tools based on objective performance metrics. Existing LLMs often make suboptimal choice, that can compromise diagnostic accuracy and overall system performance.

Closed-source LLMs remain superior than open-source alternatives: In our evaluations, closed-source LLMs consistently demonstrated better performance than their open-source counterparts. This may be due to proprietary optimizations, access to higher-quality training data, or more advanced model architectures, all of which contribute to a higher level of reliability and clinical utility.

Our findings aim to offer insight to clinicians and radiologists, who are seeking to integrate LLM-based agent systems into practice. They highlight scenarios where human supervision is critical, areas requiring additional system support, and potential risks to patient safety. By addressing these limitations, the development of LLM-based systems can evolve toward greater reliability and clinical utility, ultimately minimizing the risk of diagnostic errors.

### 3.3 Limitations and Future Directions

This study advances the evaluation of LLMs as agent manager for radiology applications but also has several limitations that point to valuable future directions.

First, while we assess seven state-of-the-art (SOTA) LLMs, these models are general-purpose and not specifically designed for the medical domain. Existing medical-specialized LLMs [[39](https://arxiv.org/html/2412.09529v2#bib.bib39), [40](https://arxiv.org/html/2412.09529v2#bib.bib40), [41](https://arxiv.org/html/2412.09529v2#bib.bib41), [42](https://arxiv.org/html/2412.09529v2#bib.bib42), [43](https://arxiv.org/html/2412.09529v2#bib.bib43)] are tailored for integrating basic medical knowledge and directly answering medical questions. However, they lack proficiency in tool-based task decomposition, which limits their effectiveness within our workflow. This presents an opportunity to enhance the performance of LLM agents by developing specialized training approaches and datasets that emphasize tool interaction and task-solving within medical contexts.

Second, replacing large language models (LLMs) with vision-language models (VLMs) as the agent core could substantially improve capabilities. Currently, LLM-based systems rely on external image processing tools to convert visual information into textual signals. In scenarios where such tools are unavailable, the system fails to function effectively. VLM-based cores could mitigate this limitation by directly processing raw image inputs, offering more detailed analysis and enabling precise responses to image-related queries.

Third, our benchmark evaluates the performance of LLM-based agents by treating tool outputs as oracle results, rather than actual implementations. While this approach allows for controlled testing of LLM decision-making and coordination, it only denotes an upper-bound performance of the agent system, and does not account for tool error propagation in real-world workflows. Future work will involve implementing these tools, using their real outputs for evaluation, and studying how cumulative errors impact overall system performance.

Finally, for scalable evaluation, our study relies on synthetic data and automatic metrics. However, involving real-world data and manual evaluation would provide a more nuanced understanding of the performance on the agent system. With additional resources, it is still essential to involve more human effort in the evaluation.

## 4 METHODS

In this section, we start by introducing the procedure for constructing RadABench-Data, including patient records, tools, and tasks. Then we present the implementation details for our RadABench-EvalPlat. Lastly, we describe the different LLMs for our evaluation, along with a series of metrics to monitor their performance.

### 4.1 Building RadABench-Data

We create RadABench-Data, an LLM-agent evaluation dataset tailored for radiology, comprising 2,200 synthetic patient records, 24,200 associated QA pairs, and infinite radiology commonly used tools based on our proposed radiology taxonomy system. In this part, we will introduce our implementation details in data generation and manual verification.

![Refer to caption](img/2d8f88305f78ecc76480404d55b11d59.png)

Figure 6: RadABench-Data Generation. The GPT-4 related dataset construction pipeline for patient records and QA pairs. Patient records are generated based on related taxonomy, where GPT-4 is prompted with anatomy-modality-disease triplets. QA pairs are derived from task taxonomy based on corresponding patient record information, where GPT-4 generates 11 QA tasks for each patient record covering all listed tasks through task-specific prompts.

#### 4.1.1 Data Generation.

We generate three types of data based on our radiology taxonomy system, *i.e.*, patient records generation, tool cards generation, and task simulation. In the following, we will describe the generation details.

Patient Record Generation. The patient record denotes the profile for the patient, with the following information:

*   •

    Patient Information: demographics(age, sex, height, weight), medical history, and chief complaints.

*   •

    Imaging Information: anatomical location, modality specifications, organ and anomaly masks with corresponding classifications.

*   •

    Imaging-related clinical Information: diagnosed conditions, organ and abnormal biomarker metrics with their respective values, specific patient indicators, integrated patient and radiological reports, and treatment recommendations.

As shown in Figure [6](https://arxiv.org/html/2412.09529v2#S4.F6 "Figure 6 ‣ 4.1 Building RadABench-Data ‣ 4 METHODS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?"), for each patient record construction, we first specify a <Disease> associated with a particular <Anatomy> site and imaging <Modality> from the case generation taxonomy. Second, we define a Patient Record Template to guide the generation process. The component of this template consists of three aspects: 1) Patient information in six aspects, including <Age>, <Sex>, <Height>, <Weight>, <History>, and <Complaint>. 2) Imaging information, including <Anatomy> site, imaging <Modality>, and segmentation masks for organs and anomalies (marked with special symbols). 3) Imaging-related clinical information in six aspects, including <Anomaly> description, <Disease>, <Biomarker> measurements (including dimensions, quantity, intensity, texture and other characteristics of an object observed in the image), <Indicator> values (such as scoring or grading systems, e.g., CURB-65, tumor grading), imaging <Report> (aligning MIMIC-CXR and PadChest format with Findings and Impressions), and <Treatment> plans (covering diagnostic procedures, medication guidelines, and follow-up protocols). Third, we create prompts by combining disease-anatomy-modality combinations with the template. Each prompt instructs GPT-4 to fill appropriate values into the template’s entities (<>) based on the given combination. For example, when a chest-xray-pneumonia combination is selected, GPT-4 should generate suitable values for patient information (like age and symptoms typical for pneumonia), imaging information (focusing on chest area in x-ray), and clinical information (pneumonia-related findings and treatments) to create a complete patient record. We generate multiple patient records by selecting different combinations using this template structure. For detailed prompts and a generated patient record example, please refer to the supplementary file.

Tool Card Generation. We generate a “Tool Card” to represent a tool by filling 8 components: name, category, property, ability, compulsory input, optional input, output, performance. Each component has its specific definition, as follows:

*   •

    Name refers to an index of the tool, which does not have specific meaning, for example, Tool1, Tool2, *etc.*;

*   •

    Category denotes one instance from our pre-defined 10 tool categories from the radiology tool taxonomy system;

*   •

    Property describes if a tool is universal or targets a specific anatomy-modality combination (e.g., universal “anomaly detector” or “imaging diagnoser” only suitable for “head and neck X-ray” images);

*   •

    Ability defines the tool’s application scenarios, encompassing relevant modalities, anatomies, and, when necessary, related diseases;

*   •

    Compulsory input gives the minimum inputs required for the tool to operate;

*   •

    Optional input describes additional inputs that may improve performance;

*   •

    Output specifies the output format;

*   •

    Performance shows a score number between 0-1, indicating the tool’s average performance on certain tasks, for example, dice score for segmenting certain anatomy.

During the generation process, we begin by selecting the category component. Next, we obtain the predefined candidate value sets for the other components within that tool category. Using random sampling, we then generate a conceptual tool.

Task Simulation. To simulate the 11 tasks in our radiology taxonomy system, we generate free-text QA pairs focusing on a specific radiology task. Commonly, their formats are as follows:

*   •

    Question.. The question must be specifically directed at the medical imaging aspects, addressing either explicit information within the case or content that can be synthesized from known case information, like “What is the size of the tumor?”.

*   •

    Answer. The answer must be derived from critical information within the related patient records that support the question. When visual information such as masks is involved in the answer, it is presented as free text with special tokens (e.g., “The organ segmentation result is shown as [Organ Mask]”).

As shown in Figure [6](https://arxiv.org/html/2412.09529v2#S4.F6 "Figure 6 ‣ 4.1 Building RadABench-Data ‣ 4 METHODS ‣ Can Modern LLMs Act as Agent Cores in Radiology Environments?"), to generate QA pairs that comprehensively cover the 11 tasks described in our radiology taxonomy system, we develop detailed prompt descriptions encompassing target scenarios, procedural steps, and specific requirements. Specifically, we feed the GPT-4 with the definition of each task and the corresponding planning chains listed in our taxonomy system, along with a specific patient case, and ask it to generate both questions and answers simultaneously. The questions must align with the corresponding task theme, and their solution decomposition must follow the predefined chains. The answers should directly address the questions based on the available case information. To ensure clinical relevance and coherence, we set several other requirements in the generation prompt. 1) The all generated task simulation tasks for a patient record must be independent and not affect each other, 2) responses should be brief, 3) visual element placeholder must follow standard notation format, e.g., [organ mask] and [anomaly mask], and 4) each response must follow a structured format with special question-answer tokens (e.g., <Q>…</Q>, <A>…</A>). Finally, for each patient record, we generate 11 simulation instances corresponding to different tasks respectively.

#### 4.1.2 Manual Verification

To ensure the quality and reliability of the synthetic dataset, each patient record is rigorously reviewed by a radiologist with over 10 years of clinical experience. The evaluation focused on four critical aspects:

*   •

    Patient diversity. Most anatomical sites demonstrated sufficient diversity across clinical scenarios, with one exception, namely breast-related patients, that have 99% female patients and an age concentration between 45–65 years. In contrast, other anatomical sites showed adequate variability in patient demographics and record complexity.

*   •

    Disease representativeness. The selected diseases were confirmed to be clinically common and to exhibit distinct radiological features. This ensures that tasks are both clinically relevant and solvable using radiological analysis.

*   •

    Information validity. Medical entities, including organs, abnormalities, biomarkers, indicators, reports, and treatments, were verified for their objective existence. Approximately 97.32% of records were deemed valid, with the remaining records undergoing manual correction to ensure accuracy.

*   •

    Logical consistency. Coherence between patient information, medical history, chief complaints, and radiological findings was emphasized. A consistency rate of 96.37% was observed, with all inconsistencies manually resolved to maintain logical integrity across records.

For the synthetic tools, all related predefined value sets are designed by a radiologist with 10 years of experience, ensuring that our generated tool cards are clinically meaningful and highly relevant.

For the QA pairs, considering the answer correctness can be directly checked by reading patient records, the verification difficulty is low. Thus, the related quality control is conducted by two medical PhD students focusing on two aspects:

*   •

    Query-answer rationality. Each question is examined to determine whether it represents common clinical tasks or concerns of radiologists and whether the corresponding answer has objectively addressed the question while aligning with the record information. The review reveals that 94.23% of questions reflected typical imaging-related tasks, with only a small fraction focusing on irrelevant details. Similarly, 98.36% of answers demonstrate sufficient quality, with minor deviations from record information observed in a small subset.

*   •

    Task chain decomposition accuracy. The designated step chains for each QA task are manually assessed to ensure they are necessary, sufficient, correct, and logically structured. Results indicate that 90.64% of task chain labels are appropriate. The remaining chains could partially resolve the queries but do not represent the most suitable ones.

Finally, the unexpected patient records or QA pairs identified are directly revised by clinicians. Thus, the final data scale in RadABench-Data remains to be of high-quality.

### 4.2 Establishing RadABench-EvalPlat

The objective of RadA-EvalPlat is to create a free-text agent system that enables LLM cores to interact with diverse external tool sets. It contains two main approaches, first, a prompt-driven agent workflow enabling assessment of any LLMs, and second, a dynamic tool set simulation strategy to cover the possible radiology analysis tools and simulate varying evaluation conditions.

Prompt-driven Agent Workflow

A typical agent workflow consists of three main components: an LLM-based central core, an external expertise tool set, and a memory bank to maintain the intermediate results. Given a clinical query with the patient’s information, the system follows three high-level steps to obtain a final answer: Task Decomposition, Tool Selection & Execution, and Response Generation. Notably, in the supplementary file, we provide detailed prompt examples of the three stages.

![Refer to caption](img/cf11172f8e1435d814be8a58a030ff8f.png)

Figure 7: Agent core workflow. It consists of a three-phase architecture. The first phase is the Initialization phase, which generates the tool chain and organizes known parameters by considering toolkit descriptions, patient information, and the initial prompt. The second phase is the Circulation phase, which involves multiple rounds of interactive execution through a select-call-update mechanism to accomplish tasks incrementally. Each iteration produces one of three possible output flags: Call (continue invocation), EndCall (terminate invocation), or NoCall (tool deficiency detected). In the third phase, the agent core performs Conclusion processing: when the flag is NoCall, it outputs a Denial message; when the flag is EndCall, it synthesizes all intermediate knowledge and questions obtained from previous phases to generate the final answer.

Task Decomposition. In this first stage, the agent core receives an initial prompt that describes the complete agent workflow guidelines to initialize its role of ‘radiology agent cores’, along with the “Patient Information” of the patient records and queries. First, LLMs need to initialize the memory bank by storing all available initial information, including the patient’s radiology images and demographics, medical history and chief complaint. Second, LLMs will perform task decomposition by breaking down the clinical query into a series of sub-steps that can be addressed using various specialized tools. This prompt establishes the agent core’s role in solving medical imaging tasks and sets the clinical context while providing specific query descriptions and response format examples. The agent must follow strict response rules, such as using special markers “$$” to highlight key information and adhering to predefined scopes of information types and tool categories. While the agent is made aware of the subsequent stages in the workflow, these will be addressed through separate prompts later in the process.

Tool Selection & Execution. In this stage, the agent core performs medical image analysis by iteratively calling different external tools. Following the tool chain identified in the first stage, the agent core begins by making its first tool selection and invocation, providing necessary inputs, and obtaining results that are then pushed into the memory bank. The process continues iteratively, with the agent refining subsequent decisions by analyzing the overall planned steps, previous execution results, and all stored information.

The execution follows three distinct XML-style template formats: <Call>, <EndCall>, and <NoCall>. The <Call> template represents intermediate steps in the execution plan where a specific tool is needed but the final goal has not yet been achieved. The <EndCall> template indicates a terminal step where the tool’s execution will produce the desired final result. When the agent encounters situations where no suitable tools exist for a particular task, it employs the <NoCall> template to document the capability gap, specifying the purpose, relevant tool category, anatomical context, imaging modality, and the required but currently insufficient capabilities. Each iteration follows these strict XML-style templates and comprehensive guidelines, that cover task descriptions, input requirements, and execution rules. After each tool is executed, the memory bank is updated to record tool’s output. The agent core then assesses whether the desired results have been achieved. If they have, it exits this step; otherwise, it loops back and repeats this step.

Response Generation. In the final stage, once the agent core determines that sufficient information has been gathered in the memory bank, it proceeds to generate a comprehensive response. Guided by a conclusion prompt, the agent synthesizes all findings from the previous stages into a conclusive answer. This process integrates multiple elements: the initial analysis plan, the sequence of tool executions and their results, and all relevant information stored in the memory bank. The prompt directs the agent to construct a concise answer to the original clinical query, ensuring that key evidence from the results supports the conclusions, while maintaining alignment with the planned analysis strategy. The direct and brief prompts result in a focused and efficient response that effectively integrates all relevant findings from the diagnostic process.

In such an agent workflow, LLMs serve as agent cores, requiring five crucial abilities. These encompass analysis (comprehensively evaluating clinical medical scenarios, clinical information, and task/problem descriptions), decision-making (strategically decomposing tasks based on their type and content), tool invocation (understanding available tools and selecting appropriate ones for specific tasks), integration (coordinating and harmonizing the input-output relationships across tool invocations), and synthesis (consolidating intermediate results from all interactions to formulate comprehensive conclusions). Through these interconnected abilities, the agent core can establish a systematic workflow for medical tasks with a well-defined prompting strategy.

Dynamic Tool Set Simulation

To implement the above agent workflow, it is essential to simulate external tool sets. Thus, we propose a dynamic tool set simulation strategy, that can generate varying conceptual tool sets simulating different evaluation conditions. Specifically, we combine different tools following a heuristic algorithm, forming tool sets to meet different evaluation conditions, as follows:

NS. (Necessary and Sufficient) Conditions. In this fundamental testing environment, we configure one to three tools for each of the ten tool categories, with each tool equipped with maximum capabilities within its specified input-output constraints. For example, we implement a single comprehensive disease diagnoser capable of analyzing all disease types across different modalities and anatomical locations. This simplified setup is specifically designed to evaluate the agent core’s foundational capabilities in task decomposition, high-level tool chain planning and basic tool execution, focusing on its ability to understand tool functionality and construct appropriate sequences rather than dealing with complex tool selection within categories.

SNN. (Sufficient but Not Necessary) Conditions. In this expanding test enviroment, we build upon the NS. tool scenario by introducing a larger number of “unusable” tools that don’t match the current case’s modality and anatomical requirements. This deliberate expansion of incompatible tools better simulates real-world scenarios where only a small subset of available tools is actually applicable to any given case, while the majority are unsuiTable We establish three distinct scenarios with varying numbers of unusable tools:

*   •

    Regular Conditions. A basic scenario where each tool category contains necessary tools plus at most one additional unusable tool.

*   •

    Medium Conditions. A moderately complex scenario where each tool category includes 2-3 additional tools beyond those specific to the QA task’s modality and anatomy.

*   •

    Large Conditions. A highly complex scenario where each tool category contains tools for all common anatomy-modality combinations, resulting in substantial redundancy of unusable tools.

This setup specifically targets the evaluation of the agent core’s capability to comprehend and filter through extensive tool descriptions, testing its ability to efficiently identify and select relevant tools from a larger pool of options while maintaining accurate task execution despite the increased cognitive load of processing longer tool card text descriptions.

NR. (Necessary but not Sufficient) Conditions. In this testing environment, we deliberately create unsolvable task scenarios by omitting essential tools based on case information and QA task. The scenario encompasses three distinct categories of tool omission scenarios.

*   •

    Denyl1 Conditions. This scenario considers the complete absence of a tool category, such as when no organ segmentation tool can be found, making it impossible for a organ segmentation task to obtain organ masks and labels. Ideally, this represents the most straightforward scenario for the agent core to recognize and decline a response.

*   •

    Denyl2 Conditions. This scenario examines the absence of tools for specific modalities and anatomies. While disease diagnosis tools might be available in general, the scenario lacks tools that specifically match the current case’s modality and anatomy. For example, in a chest X-ray analysis task, disease diagnosis tools might exist for brain MRI or abdominal CT, but none are available for chest radiographs, making it impossible to complete the required analysis.

*   •

    Denyl3 Conditions. This scenario addresses situations where specific tools have insufficient capabilities. Even when tools of the appropriate category, anatomical region, and modality exist, their capabilities might be inadequate for the current task. For instance, if a disease diagnosis tool can only detect pneumothorax and atelectasis, but the query requires lung cancer assessment, the tool’s capabilities are insufficient to answer the question, necessitating a decline response.

This setup is specifically designed to evaluate the agent core’s performance when it should decline to provide answers, as all three types of omissions strictly ensure that reaching the ground truth becomes impossible, testing the it’s ability to recognize and appropriately respond to unsolvable scenarios.

OPT. (Optimal) Conditions. In this testing environment, multiple tools are configured within each category with varying performance metrics, reflecting real-world scenarios where more specialized models typically achieve higher performance. For example, within the Anomaly Detection category, a universal tool might score $0.5$, while a tool specialized for the current case’s modality and anatomical location scores $0.6$, and an even more specialized tool focused on specific anomaly types for the same modality and location achieves $0.7$. This setup mimics the common pattern where increased specialization correlates with improved performance, and is designed to evaluate the agent core’s decision-making capabilities in selecting optimal tools by balancing between tool universality and performance metrics when constructing solution chains.

Overall, each tool combination condition is designed to provide comprehensive insights into specific aspects of agent core performance while maintaining practical feasibility in testing environments. They collectively enable systematic assessment of agent core capabilities across various operational dimensions, from basic tool manipulation to sophisticated chain planning and optimization.

### 4.3 Evaluation

In this part, we introduce the considered LLMs as the agent core in our proposed RadABench and the metrics we adopt to quantize their performance.

Agent Core Selection

The selection of the agent core significantly influences the overall performance of the Agent system. Large Language Models serving as agent cores must possess several critical capabilities to function effectively. First, they must demonstrate robust multi-turn interactive dialogue capabilities to support real-time decision integration and refinement. Second, they require substantial domain-specific knowledge, necessitating models with sufficient parameter scale to encompass comprehensive medical domain understanding for informed decision-making. Third, models must support a context window of at least 40,000 tokens to accommodate the extensive tool descriptions within our toolkit, as models with limited context windows (such as GPT-3.5 [[44](https://arxiv.org/html/2412.09529v2#bib.bib44)]’s 16,000 tokens) prove insufficient for this purpose.

Based on these requirements, we have selected specific models for evaluation as potential agent cores, with particular attention to their capacity to handle extensive context, maintain coherent multi-turn interactions, and demonstrate robust medical domain knowledge. These selection criteria ensure that the chosen models can effectively orchestrate the complex interactions required in medical imaging interpretation tasks. In the following, we will introduce the LLMs considered in our work for comparison:

*   •

    GPT-4 [[3](https://arxiv.org/html/2412.09529v2#bib.bib3)] is a groundbreaking large language model developed by OpenAI. It has demonstrated superior ability across different domains, including healthcare, and is regarded as the most representative close-source LLM. Due to the confidentiality of data and model details, the detailed model scale is uncertain. In this work, we evaluate it in the zero-shot through its official API, termed as version “gpt-4-turbo-2024-04-09”.

*   •

    GPT-4o [[45](https://arxiv.org/html/2412.09529v2#bib.bib45)] is the most advanced GPT model. It is multimodal (accepting text or image inputs and outputting text), and it has the same high intelligence as GPT-4 Turbo but is much more efficient. Compared to GPT-4, it further enhances LLMs’ reasoning abilities, with a focus on solving complex rationale tasks through more thoughtful inference, which aligns closely with the skills assessed in RadA-Bench. Similar to GPT-4, its data and model details are unreleased and we evaluate it with the official API, termed as “gpt-4o-2024-08-06”.

*   •

    Gemini [[7](https://arxiv.org/html/2412.09529v2#bib.bib7)], formerly known as Bard, is a general multimodal foundation model developed by Google. Though it is targeted at multimodal cases, It is also widely regarded as Google’s most representative closed-source language model and its language ability even surpasses Google’s other LLMs, like PaLM 2 [[46](https://arxiv.org/html/2412.09529v2#bib.bib46)]. Its detailed scale and training details are not released. We evaluate it through official APIs with the version named “gemini-1.5-pro-latest” with the latest update in September 2024.

*   •

    Claude [[47](https://arxiv.org/html/2412.09529v2#bib.bib47)] is a family of large language models developed by Anthropic. In size, it contains three types of model scales, *i.e.*, “Haiku”, “Sonnet”, and “Opus” from small to large. on the other hand, it also has two mainly used generations, *i.e.*, Claude-3, and Claude-3.5\. Based on the official introduction, we choose “Claude-3.5-Sonnet” to represent the series, considering it has proved to be the most powerful on the widely used benchmarks. Similar to former close-source LLMs, its details are confidential and we access it with official API “claude-3-5-sonnet-20241022”.

*   •

    Llama [[8](https://arxiv.org/html/2412.09529v2#bib.bib8)] series is the best known open source LLMs developed by Meta. The latest version is Llama-3.1, whose largest version (Llama-3.1-405B) demonstrates comparable performance to GPT-4 on most benchmarks. We employ “Llama-3.1-405B” with the latest update in September 26th, 2024 in our evaluation process.

*   •

    Mixtral [[48](https://arxiv.org/html/2412.09529v2#bib.bib48)] was first released in October 2023 and continued updating. In architecture design, Mixtral adopts the mixture of experts [[49](https://arxiv.org/html/2412.09529v2#bib.bib49)] to reduce the computational cost and enhance the performance. In this paper, we evaluate the “Mixtral-8x7B” proposed in December 11th, 2023 by default.

*   •

    Qwen [[50](https://arxiv.org/html/2412.09529v2#bib.bib50)] is a bilingual LLM family developed by the Qwen Team. Although it incorporates both Chinese and English, its ability in English alone is still impressive compared with other English-centric LLMs, based on its evaluations. We choose it as a representative of the bilingual LLMs. We evalaute its latest model version released in September 19, 2024, “Qwen-2.5-72B” on our RadABench.

Evaluation Metrics

Inspired by former agent-wise evaluation works [[13](https://arxiv.org/html/2412.09529v2#bib.bib13), [12](https://arxiv.org/html/2412.09529v2#bib.bib12)], our evaluation metrics comprehensively assess agent core performance, from 5 key capabilities: first, chain planning, which evaluates both planning quality and consistency between planned and actual execution chains; second, optimal orchestrating, which assesses the agent’s ability to optimize tool usage during the execution phase; third, IO organizing, which measures how well the agent organize inputs and outputs between multiple tools; fourth, response synthesizing, which examines the quality of generated responses based on tool outputs; fifth, unsolvability parsing, which evaluates the capability to identify and handle unsolvable cases appropriately.

Metrics on Chain Planning. The chain planning assessment primarily focuses on measuring the differences between LLM-generated tool-category-wise chains against the ground truth decomposition. The following metrics are adopted:

*   •

    Levenshtein Distance [[51](https://arxiv.org/html/2412.09529v2#bib.bib51)]. This metric calculates the minimum number of edit operation steps (insertion, deletion, substitution) required to transform a generated tool chain into the ground-truth chain. A lower score indicates higher similarity between the two sequences, with 0 representing identical sequences.

*   •

    False Discovery Rate [[52](https://arxiv.org/html/2412.09529v2#bib.bib52)]. This metric calculates the proportion of incorrectly included tools (tools present in the generated chain but absent in the ground-truth chain) relative to all tools in the generated chain. It reflects the agent’s precision in planning, avoiding unnecessary tool usage.

*   •

    Tool Matching Accuracy [[53](https://arxiv.org/html/2412.09529v2#bib.bib53)]. This metric directly checks how many positions in the ground-truth tool sequence can be hit (the same tool appears in the same position) by the prediction chain, and then calculates the ratio of matched tools to the total length of the ground-truth length. It reflects the agent’s recall in planning.

Metrics on Optimal Orchestrating. The optimal orchestrating evaluates whether the agent can execute a planned tool-category-wise chain with optimal tools by comparing their performance scores. The following metrics are adopted:

*   •

    Optimal Tool Score [[54](https://arxiv.org/html/2412.09529v2#bib.bib54)]. This metric evaluates the agent’s decision quality when it needs to select a tool to finish a certain tool category demand. The score is defined as $(N-R+1)/N$, where N is the total number of suitable tools for the tool category in the provided tool set, and R is the performance rank of the selected tool (1 for the best). For example, if there are 4 suitable tools and the agent selects the second-best one, the score would be $(4-2+1)/4=0.75$. The final score is determined by averaging all relevant tool selection steps. For example, a QA pair may involve multiple tool selection steps during execution, each generating its own Optimal Tool Score.

Metrics on IO Organizing. The IO organizing assesses how well the agent organizes and manages IO flows between sequential tools in execution. The following metrics are adopted:

*   •

    Execution Completion Rate. This metric measures whether the agent can complete the entire planned chain without any input/output errors. A pass is awarded when all tools in the chain execute successfully regardless of the final output’s correctness, indicating the agent’s basic ability to handle tool IO requirements properly.

*   •

    Pre-Failure Success Percentage. This metric further clarifies the percentage of successfully executed steps before a failure happens. The calculation is performed exclusively on failed cases by comparing the number of successfully executed tools before a failure occurs and then dividing the total length of the ground-truth chain. This helps to further compare the IO organizing ability between different LLMs among the fail cases.

Metrics on Response Synthesizing. The response synthesizing evaluates how well the agent produces final outputs and how closely these outputs match expert-provided references. The following metrics are adopted:

*   •

    Target Hit Rate [[54](https://arxiv.org/html/2412.09529v2#bib.bib54)]. This metric checks whether the execution chain ends with an appropriate tool that provides the required response format. A pass means the final output matches the expected response type and format for the given task.

*   •

    Milestone Hit Rate [[54](https://arxiv.org/html/2412.09529v2#bib.bib54)]. This metric measures the agent’s ability to complete valuable intermediate steps even when the final task is not fully successfully addressed. In medical imaging tasks, it evaluates whether the agent can generate useful intermediate outputs (like organ segmentation or preliminary diagnoses) before encountering any failures. This helps assess the agent’s effectiveness in providing partial but clinically valuable results. Specifically, we manually set a milestone for each radiology task, they calculate the hitting rate across all QA pairs.

*   •

    BLEU [[55](https://arxiv.org/html/2412.09529v2#bib.bib55)]. This metric measures word overlap between generated and reference texts, focusing on precision. In medical imaging tasks, it evaluates how well the generated diagnostic reports match reference reports by comparing the overlap of word sequences.

*   •

    ROUGE [[56](https://arxiv.org/html/2412.09529v2#bib.bib56)]. This metric assesses text quality by measuring word overlap between generated and reference texts, focusing on recall. It helps evaluate whether the generated medical reports include all key information found in reference reports.

*   •

    F1 [[57](https://arxiv.org/html/2412.09529v2#bib.bib57)]. This metric combines precision and recall measures based on word overlap between generated and reference texts. It provides a balanced assessment of whether the generated medical reports are both accurate and complete.

*   •

    RaTEScore [[58](https://arxiv.org/html/2412.09529v2#bib.bib58)]. This medical-specific metric evaluates text similarity by focusing on key medical terms and entities. It provides a more clinically relevant assessment by measuring how well the generated reports capture essential medical information compared to reference reports.

Metrics on Unsolvability Parsing. All the above metrics evaluate the LLM performance in solvable cases while the unsolvability parsing targets to measure whether the agent core can figure out the unsolvability when a certain query is unsolvable with an external tool set. We design two metrics to monitor the parsing ability from two levels:

*   •

    Unsolvability Awareness Rate. Measures if the agent can identify when a task cannot be solved with the current tools. A pass means the agent correctly declines to provide an invalid answer, while a fail means it attempts to solve an impossible task.

*   •

    Unsolvability Grounding Rate. Evaluates if the agent can identify specific deficiencies, whether in missing tools or lacking capabilities. A pass requires the agent to correctly point out what is missing - either specific tools or capabilities needed to complete the task.

## References

*   [1] Karan Singhal, Shekoofeh Azizi, Tao Tu, S Sara Mahdavi, Jason Wei, Hyung Won Chung, Nathan Scales, Ajay Tanwani, Heather Cole-Lewis, Stephen Pfohl, et al. Large language models encode clinical knowledge. Nature, 620(7972):172–180, 2023.
*   [2] Humza Naveed, Asad Ullah Khan, Shi Qiu, Muhammad Saqib, Saeed Anwar, Muhammad Usman, Naveed Akhtar, Nick Barnes, and Ajmal Mian. A comprehensive overview of large language models. arXiv preprint arXiv:2307.06435, 2023.
*   [3] Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, et al. Gpt-4 technical report. arXiv preprint arXiv:2303.08774, 2023.
*   [4] Chaoyi Wu, Xiaoman Zhang, Ya Zhang, Yanfeng Wang, and Weidi Xie. Towards generalist foundation model for radiology. arXiv preprint arXiv:2308.02463, 2023.
*   [5] Hejie Cui, Lingjun Mao, Xin Liang, Jieyu Zhang, Hui Ren, Quanzheng Li, Xiang Li, and Carl Yang. Biomedical visual instruction tuning with clinician preference alignment. arXiv preprint arXiv:2406.13173, 2024.
*   [6] Haotian Liu, Chunyuan Li, Qingyang Wu, and Yong Jae Lee. Visual instruction tuning. Advances in neural information processing systems, 36, 2024.
*   [7] Gemini Team, Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, et al. Gemini: a family of highly capable multimodal models. arXiv preprint arXiv:2312.11805, 2023.
*   [8] Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, et al. Llama: Open and efficient foundation language models. arXiv preprint arXiv:2302.13971, 2023.
*   [9] Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, and Thomas Scialom. Toolformer: Language models can teach themselves to use tools. Advances in Neural Information Processing Systems, 36, 2024.
*   [10] Xiangru Tang, Anni Zou, Zhuosheng Zhang, Ziming Li, Yilun Zhao, Xingyao Zhang, Arman Cohan, and Mark Gerstein. Medagents: Large language models as collaborators for zero-shot medical reasoning. In ICLR 2024 Workshop on Large Language Model (LLM) Agents.
*   [11] Qiao Jin, Zhizheng Wang, Yifan Yang, Qingqing Zhu, Donald Wright, Thomas Huang, W John Wilbur, Zhe He, Andrew Taylor, Qingyu Chen, et al. Agentmd: Empowering language agents for risk prediction with large-scale clinical tool learning. arXiv preprint arXiv:2402.13225, 2024.
*   [12] Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. Toolllm: Facilitating large language models to master 16000+ real-world apis. arXiv preprint arXiv:2307.16789, 2023.
*   [13] Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu, and Yueting Zhuang. Hugginggpt: Solving ai tasks with chatgpt and its friends in hugging face. Advances in Neural Information Processing Systems, 36, 2024.
*   [14] Hejie Cui, Zhuocheng Shen, Jieyu Zhang, Hui Shao, Lianhui Qin, Joyce C Ho, and Carl Yang. Llms-based few-shot disease predictions using ehr: A novel approach combining predictive agent reasoning and critical agent instruction. arXiv preprint arXiv:2403.15464, 2024.
*   [15] Michael Moor, Oishi Banerjee, Zahra Shakeri Hossein Abad, Harlan M Krumholz, Jure Leskovec, Eric J Topol, and Pranav Rajpurkar. Foundation models for generalist medical artificial intelligence. Nature, 616(7956):259–265, 2023.
*   [16] Peilun Shi, Jianing Qiu, Sai Mu Dalike Abaxi, Hao Wei, Frank P-W Lo, and Wu Yuan. Generalist vision foundation models for medical imaging: A case study of segment anything model on zero-shot medical segmentation. Diagnostics, 13(11):1947, 2023.
*   [17] Qiaoyu Zheng, Weike Zhao, Chaoyi Wu, Xiaoman Zhang, Lisong Dai, Hengyu Guan, Yuehua Li, Ya Zhang, Yanfeng Wang, and Weidi Xie. Large-scale long-tailed disease diagnosis on radiology images. Nature Communications, 15(1):10147, 2024.
*   [18] Ziheng Zhao, Yao Zhang, Chaoyi Wu, Xiaoman Zhang, Ya Zhang, Yanfeng Wang, and Weidi Xie. One model to rule them all: Towards universal segmentation for medical images with text prompts. arXiv preprint arXiv:2312.17183, 2023.
*   [19] Xiao Zhou, Xiaoman Zhang, Chaoyi Wu, Ya Zhang, Weidi Xie, and Yanfeng Wang. Knowledge-enhanced visual-language pretraining for computational pathology. In Aleš Leonardis, Elisa Ricci, Stefan Roth, Olga Russakovsky, Torsten Sattler, and Gül Varol, editors, Computer Vision – ECCV 2024, pages 345–362, Cham, 2025\. Springer Nature Switzerland.
*   [20] Jiayu Lei, Xiaoman Zhang, Chaoyi Wu, Lisong Dai, Ya Zhang, Yanyong Zhang, Yanfeng Wang, Weidi Xie, and Yuehua Li. Autorg-brain: Grounded report generation for brain mri. arXiv preprint arXiv:2407.16684, 2024.
*   [21] Tao Tu, Shekoofeh Azizi, Danny Driess, Mike Schaekermann, Mohamed Amin, Pi-Chuan Chang, Andrew Carroll, Charles Lau, Ryutaro Tanno, Ira Ktena, et al. Towards generalist biomedical ai. NEJM AI, 1(3):AIoa2300138, 2024.
*   [22] Chunyuan Li, Cliff Wong, Sheng Zhang, Naoto Usuyama, Haotian Liu, Jianwei Yang, Tristan Naumann, Hoifung Poon, and Jianfeng Gao. Llava-med: Training a large language-and-vision assistant for biomedicine in one day. arXiv preprint arXiv:2306.00890, 2023.
*   [23] Zhengliang Liu, Yiwei Li, Peng Shu, Aoxiao Zhong, Longtao Yang, Chao Ju, Zihao Wu, Chong Ma, Jie Luo, Cheng Chen, et al. Radiology-llama2: Best-in-class large language model for radiology. arXiv e-prints, pages arXiv–2309, 2023.
*   [24] Zhihong Chen, Maya Varma, Jean-Benoit Delbrouck, Magdalini Paschali, Louis Blankemeier, Dave Van Veen, Jeya Maria Jose Valanarasu, Alaa Youssef, Joseph Paul Cohen, Eduardo Pontes Reis, et al. Chexagent: Towards a foundation model for chest x-ray interpretation. arXiv preprint arXiv:2401.12208, 2024.
*   [25] Kai Zhang, Rong Zhou, Eashan Adhikarla, Zhiling Yan, Yixin Liu, Jun Yu, Zhengliang Liu, Xun Chen, Brian D Davison, Hui Ren, et al. A generalist vision–language foundation model for diverse biomedical tasks. Nature Medicine, pages 1–13, 2024.
*   [26] Xiaoman Zhang, Chaoyi Wu, Ziheng Zhao, Weixiong Lin, Ya Zhang, Yanfeng Wang, and Weidi Xie. Pmc-vqa: Visual instruction tuning for medical visual question answering. arXiv preprint arXiv:2305.10415, 2023.
*   [27] Nikita Mehandru, Brenda Y Miao, Eduardo Rodriguez Almaraz, Madhumita Sushil, Atul J Butte, and Ahmed Alaa. Evaluating large language models as agents in the clinic. NPJ digital medicine, 7(1):84, 2024.
*   [28] François Remy, Kris Demuynck, and Thomas Demeester. Biolord: Learning ontological representations from definitions (for biomedical concepts and their textual descriptions). arXiv preprint arXiv:2210.11892, 2022.
*   [29] Yaobo Liang, Chenfei Wu, Ting Song, Wenshan Wu, Yan Xia, Yu Liu, Yang Ou, Shuai Lu, Lei Ji, Shaoguang Mao, et al. Taskmatrix. ai: Completing tasks by connecting foundation models with millions of apis. Intelligent Computing, 3:0063, 2024.
*   [30] Chenyu Wang, Weixin Luo, Qianyu Chen, Haonan Mai, Jindi Guo, Sixun Dong, Zhengxin Li, Lin Ma, Shenghua Gao, et al. Tool-lmm: A large multi-modal model for tool agent learning. arXiv preprint arXiv:2401.10727, 2024.
*   [31] Andrew Hoopes, Victor Ion Butoi, John V Guttag, and Adrian V Dalca. Voxelprompt: A vision-language agent for grounded medical image analysis. arXiv preprint arXiv:2410.08397, 2024.
*   [32] Jianing Qiu, Kyle Lam, Guohao Li, Amish Acharya, Tien Yin Wong, Ara Darzi, Wu Yuan, and Eric J Topol. Llm-based agentic systems in medicine and healthcare. Nature Machine Intelligence, 12 2024.
*   [33] Junkai Li, Siyu Wang, Meng Zhang, Weitao Li, Yunghwei Lai, Xinhui Kang, Weizhi Ma, and Yang Liu. Agent hospital: A simulacrum of hospital with evolvable medical agents. arXiv preprint arXiv:2405.02957, 2024.
*   [34] Chaoyi Wu, Jiayu Lei, Qiaoyu Zheng, Weike Zhao, Weixiong Lin, Xiaoman Zhang, Xiao Zhou, Ziheng Zhao, Ya Zhang, Yanfeng Wang, et al. Can gpt-4v (ision) serve medical applications? case studies on gpt-4v for multimodal medical diagnosis. arXiv preprint arXiv:2310.09909, 2023.
*   [35] Dana Brin, Vera Sorin, Yiftach Barash, Eli Konen, Benjamin S Glicksberg, Girish N Nadkarni, and Eyal Klang. Assessing gpt-4 multimodal performance in radiological image analysis. European Radiology, pages 1–7, 2024.
*   [36] Gpt-4v(ision) system card. 2023.
*   [37] Samuel Schmidgall, Rojin Ziaei, Carl Harris, Eduardo Reis, Jeffrey Jopling, and Michael Moor. Agentclinic: a multimodal agent benchmark to evaluate ai in simulated clinical environments. arXiv preprint arXiv:2405.07960, 2024.
*   [38] Vasilios Mavroudis. Langchain. 2024.
*   [39] Chaoyi Wu, Weixiong Lin, Xiaoman Zhang, Ya Zhang, Weidi Xie, and Yanfeng Wang. Pmc-llama: toward building open-source language models for medicine. Journal of the American Medical Informatics Association, page ocae045, 2024.
*   [40] Pengcheng Qiu, Chaoyi Wu, Xiaoman Zhang, Weixiong Lin, Haicheng Wang, Ya Zhang, Yanfeng Wang, and Weidi Xie. Towards building multilingual language model for medicine. Nature Communications, 15(1):8384, 2024.
*   [41] Zeming Chen, Alejandro Hernández Cano, Angelika Romanou, Antoine Bonnet, Kyle Matoba, Francesco Salvi, Matteo Pagliardini, Simin Fan, Andreas Köpf, Amirkeivan Mohtashami, et al. Meditron-70b: Scaling medical pretraining for large language models. arXiv preprint arXiv:2311.16079, 2023.
*   [42] Chaoyi Wu, Pengcheng Qiu, Jinxin Liu, Hongfei Gu, Na Li, Ya Zhang, Yanfeng Wang, and Weidi Xie. Towards evaluating and building versatile large language models for medicine. arXiv preprint arXiv:2408.12547, 2024.
*   [43] Arun James Thirunavukarasu, Darren Shu Jeng Ting, Kabilan Elangovan, Laura Gutierrez, Ting Fang Tan, and Daniel Shu Wei Ting. Large language models in medicine. Nature medicine, 29(8):1930–1940, 2023.
*   [44] Openai platform, 2024.
*   [45] OpenAI, Accessed by October 12, 2024.
*   [46] Rohan Anil, Andrew M Dai, Orhan Firat, Melvin Johnson, Dmitry Lepikhin, Alexandre Passos, Siamak Shakeri, Emanuel Taropa, Paige Bailey, Zhifeng Chen, et al. Palm 2 technical report. arXiv preprint arXiv:2305.10403, 2023.
*   [47] Anthropic, Accessed by October 12, 2024.
*   [48] Albert Q Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, et al. Mixtral of experts. arXiv preprint arXiv:2401.04088, 2024.
*   [49] Saeed Masoudnia and Reza Ebrahimpour. Mixture of experts: a literature survey. Artificial Intelligence Review, 42:275–293, 2014.
*   [50] Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang, et al. Qwen technical report. arXiv preprint arXiv:2309.16609, 2023.
*   [51] Li Yujian and Liu Bo. A normalized levenshtein distance metric. IEEE transactions on pattern analysis and machine intelligence, 29(6):1091–1095, 2007.
*   [52] John D Storey. False discovery rate. International encyclopedia of statistical science, 1:504–508, 2011.
*   [53] Awatif Alqahtani, Hosam Alhakami, Tahani Alsubait, and Abdullah Baz. A survey of text matching techniques. Engineering, Technology & Applied Science Research, 11(1):6656–6661, 2021.
*   [54] Aryan Jadon and Avinash Patil. A comprehensive survey of evaluation techniques for recommendation systems. In International Conference on Computation of Artificial Intelligence & Machine Learning, pages 281–304\. Springer, 2024.
*   [55] Kishore Papineni, Salim Roukos, Todd Ward, and Wei-Jing Zhu. Bleu: a method for automatic evaluation of machine translation. In Proceedings of the 40th annual meeting of the Association for Computational Linguistics, pages 311–318, 2002.
*   [56] Chin-Yew Lin. Rouge: A package for automatic evaluation of summaries. In Text summarization branches out, pages 74–81, 2004.
*   [57] David MW Powers. Evaluation: from precision, recall and f-measure to roc, informedness, markedness and correlation. arXiv preprint arXiv:2010.16061, 2020.
*   [58] Weike Zhao, Chaoyi Wu, Xiaoman Zhang, Ya Zhang, Yanfeng Wang, and Weidi Xie. Ratescore: A metric for radiology report generation. arXiv preprint arXiv:2406.16845, 2024.

## SUPPLEMENTARY FILE

### Diseases/abnormalities Names for Patient Records

We consider 100 common conditions for each anatomy-modality combination in our synthetic patient records. The considered diseases/abnormalities names are listed in the [CSV file](https://github.com/MAGIC-AI4Med/RadABench/Patient_Record_Disease_Abnormality_Names.csv).

### Detailed Explanations for Tool Categories

We explain the 10 tool categories here:

*   •

    Anatomy Classifier (AC) is a classifier to predict what anatomy region the input image is shot on.

*   •

    Modality Classifier (MC) is a classifier to predict what imaging modality the input image is based on.

*   •

    Organ Segmentor (OS) is a tool to predict the dense segmentation masks for a certain organ on the input image.

*   •

    Anomaly Detector (AD) is a detection tool to predict the anomaly region masks or box coordinates for a certain anomaly type based on the input image.

*   •

    Imaging Diagnoser (ID) is a diagnosis tool to predict related diseases solely based on the input image.

*   •

    Synthetic Dignoser (SD) is a diagnosis tool to predict related diseases synthetically based on the input image and text information.

*   •

    Biomarker Quantifier (BQ) is a biomarker calculation tool to exactly calculate the radiology biomarker based on the input image and organ-or-anomaly-wise dense masks.

*   •

    Indicator Evaluator (IE) is an indicator (like tumor grading) calculation tool to exactly calculate or predict some medical indicators based on the input image and organ-or-anomaly-wise dense masks.

*   •

    Report Generator (RG) is a text generation tool to predict radiology reports.

*   •

    Treatment Recommendation (TR) is a treatment recommendation tool (or system) to provide treatment recommendation based on current clinical findings.

### Detailed Explanations for Tool Categories

We explain the 11 radiology tasks here with their detailed tool category chains:

*   •

    Organ mask annotation is a task targeting predicting a dense mask for a certain organ. Chain: anatomy classification → modality classification → organ segmentation

*   •

    Anomaly region annotation is a task targeting predicting a regional mask for a certain anomaly type. Chain: anatomy classification → modality classification → anomaly detection

*   •

    Organ-wise biomarker calculation is a task to calculate or measure some organ-wise biomarkers. Chain: anatomy classification → modality classification → organ segmentation → biomarker quantification

*   •

    Anomaly-wise biomarker calculation is a task to calculate or measure some anomaly-wise biomarkers. Chain: anatomy classification → modality classification → anomaly detection → anomaly quantification

*   •

    Organ-and-anomaly-wise image interpretation is a task to interpret the image and answer the user query on organs or anomaly regions or both. Chain: anatomy classification → modality classification → [organ segmentation, anomaly detection]

*   •

    Disease diagnosis without visual clues is a task to end-to-end make a diagnosis based on a radiology-central patient record. Chain: anatomy classification → modality classification → disease diagnosis

*   •

    Disease diagnosis with visual clues is a task to make a diagnosis based on not only original images but also relevant segmentation annotations (organs or anomalies). Chain: anatomy classification → modality classification → [organ segmentation, anomaly detection] → disease inference

*   •

    Common report generation is a basic radiology routine task to interpret a certain radiology image into free-text descriptions. Chain: anatomy classification → modality classification → anomaly detection → disease diagnosis → report generation

*   •

    Report generation focusing on specific biomarkers is a report generation task variant emphasizing reflecting certain biomarker conditions in the final report. Chain: anatomy classification → modality classification → [organ segmentation, anomaly detection] → [organ biomarker quantification, anomaly biomarker quantification] → report generation

*   •

    Report generation focusing on specific biomarkers and indicators is a report generation task variant emphasizing reflecting certain biomarkers and indicator conditions in the final report. Chain: anatomy classification → modality classification → [organ segmentation, anomaly detection] → disease diagnosis → [organ biomarker quantification, anomaly quantification] → indicator evaluation → report generation

*   •

    Treatment planning is a task to provide a treatment plan for a patient. Chain: anatomy classification → modality classification → [organ segmentation, anomaly detection] → disease diagnosis → [organ biomarker quantification, anomaly quantification] → indicator evaluation → report generation → treatment recommendation

### A Patient Record Example

We, here, demonstrate a concrete synthetic patient record in our RadABench-Data: {mdframed}[backgroundcolor=lightgray!20] Patient Record Example  

```
"Information": {
    "Age": "42",
    "Sex": "Female",
    "Height": "165",
    "Weight": "68",
    "History": "Patient has a history of seasonal allergies and recurrent upper
    respiratory infections",
    "Complaint": "Persistent facial pain, nasal congestion, and headache for the
    past 2 weeks"
},
"Anatomy": "Head and Neck",
"Modality": "X-ray",
"Anomaly": {
    "Part": "Maxillary sinuses",
    "Symptom": "Opacification"
},
"Disease": "Sinusitis",
"OrganBiomarker": {
    "OrganObject": "Maxillary sinus",
    "OrganDim": "density",
    "OrganQuant": "+40 Hounsfield Units"
},
"AnomalyBiomarker": {
    "AnomalyObject": "Opacification",
    "AnomalyDim": "intensity",
    "AnomalyQuant": "80% increase compared to normal airspace"
},
"Indicator": {
    "Name": "Lund-Mackay Score",
    "Value": "8 (Moderate sinusitis)"
},
"Report": {
    "Finding": "X-ray of the paranasal sinuses demonstrates bilateral maxillary
    sinus opacification. The right maxillary sinus shows complete opacification,
    while the left maxillary sinus demonstrates air-fluid levels. Frontal and ethmoid
    sinuses appear clear. No evidence of bone erosion or destruction. Nasal septum
    appears midline. Soft tissues of the face and neck are unremarkable.",
    "Impression": "Findings consistent with bilateral maxillary sinusitis, more
    pronounced on the right side. No evidence of complications such as orbital or
    intracranial involvement."
},
"Treatment": "Given the patient’s symptoms and radiographic findings, a diagnosis
of acute bacterial sinusitis is likely. Initial treatment should include a 10-14 day
course of broad-spectrum antibiotics such as amoxicillin-clavulanate or, in case of
penicillin allergy, a respiratory fluoroquinolone. Adjunctive treatments include nasal
saline irrigation, intranasal corticosteroids, and oral decongestants for symptom
relief. The patient should be advised to stay well-hydrated and use over-the-counter
pain relievers as needed. If symptoms persist or worsen after 72 hours of antibiotic
therapy, reassessment is warranted. A follow-up appointment should be scheduled in 2-3
weeks to ensure resolution of symptoms. If recurrent episodes occur, further evaluation
with CT imaging and potential referral to an ENT specialist for consideration of
endoscopic sinus surgery may be necessary."

```

which is generated by prompting GPT-4 leveraging the following prompts: {mdframed}[backgroundcolor=lightgray!20] Patient Record Generation Prompt  

```

You are an experienced clinical radiologist. Your task is to generate a detailed
medical case based on a hypothetical 256x256 medical image. I will provide you with
the Anatomy and Modality of the image, as well as an overall Disease. Using this
information, you should create a comprehensive case report including patient
information, specific anomalies, biomarkers, indicators, a radiology report, and
treatment recommendations.

Please structure your response using the following template:

<Case>
    <Information>
        <Age> [Number without units] </Age>
        <Sex> [Male / Female] </Sex>
        <Height> [Number in cm] </Height>
        <Weight> [Number in kg] </Weight>
        <History> [Brief descriptive text] </History>
        <Complaint> [Brief descriptive text] </Complaint>
    </Information>
    <Anatomy> [Head and Neck / Chest / Breast / Abdomen and Pelvis / Limb / Spine]
    </Anatomy>
    <Modality> [CT / MRI / X-ray / Ultrasound / Mammography] </Modality>
    <Anomaly>
        <Part> [Specific location of anomaly (e.g., right upper lobe of lung)] </Part>
        <Symptom> [Type of anomaly (e.g., nodule)] </Symptom>
    </Anomaly>
    <Disease> [Corresponding disease name] </Disease>
    <OrganBiomarker>
        <OrganObject> [A specific organ serving as biomarker] </OrganObject>
        <OrganDim> [number / length / size / volume / angle / density / intensity /
        texture] </OrganDim>
        <OrganQuant> [Specific quantitative value] </OrganQuant>
    </OrganBiomarker>
    <AnomalyBiomarker>
        <AnomalyObject> [The same Anomaly described in the Anomaly Symptom serving
        as biomarker] </AnomalyObject>
        <AnomalyDim> [number / length / size / volume / angle / density / intensity /
        texture] </AnomalyDim>
        <AnomalyQuant> [Specific quantitative value] </AnomalyQuant>
    </AnomalyBiomarker>
    <Indicator>
        <Name> [Name of the indicator (e.g., Lung Cancer TNM Staging Score)] </Name>
        <Value> [Specific value or grade (e.g., cT2aN0M0 (Stage IB))] </Value>
    </Indicator>
    <Report>
        <Finding> [Findings section in the style of a MIMIC-CXR report] </Finding>
        <Impression> [Impression section in the style of a MIMIC-CXR report]
        </Impression>
    </Report>
    <Treatment> [A paragraph including diagnostic procedures, medication
    recommendations, and follow-up suggestions] </Treatment>
</Case>

Guidelines for generating the case report:

Information: Provide realistic patient demographics, medical history, and chief
complaint.
Anomaly: Describe a specific anomaly consistent with the given anatomy, modality,
and disease.
Biomarkers: Choose a relevant organ biomarker and the anomaly biomarker mentioned
before that can be observed in the image and provide plausible quantitative value.
Indicator: Calculate an appropriate indicator based on the patient information and
biomarker value. Provide a specific, medically accurate score or grade.
Report: Generate a concise radiological report in the style of MIMIC-CXR, with
separate Findings and Impression sections.
Treatment: Offer a comprehensive treatment plan including diagnostic procedures,
medication recommendations, and follow-up care.
Ensure that all parts of the case are medically accurate and consistent with each
other. Use your expertise as a radiologist to provide realistic and detailed
information throughout the case.

Given the following parameters:
- Anatomy: {ANATOMY}
- Imaging Modality: {MODALITY}
- Disease: {DISEASE}
Please generate this patient record.

```

### A Tool Card Example

Here is an example of a tool card of a report generation tool focused on Head and Neck X-ray images:

{mdframed}

[backgroundcolor=lightgray!20] Tool Card Example  

```
=== Tool Description for TOOL17 ===
Name: TOOL17
Category: Report Generator
Ability: Given the Head and Neck X-ray Image, any other text information and
organ/anomaly masks and labels, generate a radiology report.
Property: Report Generator only suitable for Head and Neck X-ray image with
Text and Mask
Compulsory Input: [‘$Image$’]
Optional Input: [‘$Information$’, ‘$OrganObject$’, ‘$AnomalyObject$’, ‘$Disease$’,
‘$OrganDim$’, ‘$OrganQuant$’, ‘$AnomalyDim$’, ‘$AnomalyQuant$’, ‘$IndicatorName$’,
‘$ValueName$’, ‘$OrganMask$’, ‘$AnomalyMask$’]
Output: [‘$Report$’]
Performance: Score from 0.4 to 0.88, increases with optional inputs

```

### A Task Simulation Example

Here is an example of QA pair generation from a chest X-ray record of a pneumonia patient:

{mdframed}

[backgroundcolor=lightgray!20] QA Pair Example  

```
    <Q1> Identify and segment the lung fields in this chest X-ray. </Q1>
    <A1> The lung fields are segmented [{\em organ mask}: left lung, right lung].
    Both lungs show clear boundaries. </A1>

    ...

    <Q11> Based on the imaging findings and clinical indicators, what treatment plan
    would you recommend? </Q11>
    <A11> Given the moderate pneumonia severity (CURB-65 score: 2) and [anomaly mask],
    recommend oral antibiotics and follow-up chest X-ray in 2 weeks. </A11>

```

which is generated by the following prompt, leveraging GPT-4:

{mdframed}

[backgroundcolor=lightgray!20] QA Pair Generation Prompt  

```
Assume a clinical medical imaging scenario where you, as the Agent core, play the role
of a doctor. Given a patient’s radiological image, you want to complete different tasks
by calling various tools. There are ten tools in total (numbered 0 to 9):

TOOLKIT
0\. |Modality Classifier|
    Property: A classification model
    Ability: Determine the modality of the Image.
    Input: [Image]
    Output: [Modality]

1\. |Anatomy Classifier|
    Property: A classification model,
    Ability: Determine the anatomy of the Image.
    Input: [Image]
    Output: [Anatomy]

2\. |Organ Segmentation Model|
    Property: A segmentation model
    Ability: Given the modality and anatomy, segment all organs in the Image
    (can not segment any lesion or abnormality).
    Input: [Image] & [Modality] & [Anatomy]
    Output: [Organ Mask] & [Organ Label]

3\. |Anomaly Detection Model|
    Property: A detection model
    Ability: Given the modality and anatomy, determine the location and type of
    abnormality.
    Input: [Image] & [Modality] & [Anatomy]
    Output: [Anomaly Mask] & [Anomaly Label]

4\. |Disease Diagnosis Model|
    Property: A classification model
    Ability: Given the modality and anatomy, diagnose diseases directly from the input
    image.
    Input: [Image] & [Modality] & [Anatomy]
    Output: [Diseases]

5\. |Disease Inference Model|
    Property: A Inference model
    Ability: Infer disease based on organ segmentation and anomaly detection results.
    Input: [Image] & [Organ Mask] & [Organ Label] & [Anomaly Mask] & [Anomaly Label]
    Output: [Diseases]

6\. |Biomarker Quantification Model|
    Property: A quantification model
    Ability: Given the organ region or anomaly region and the biomarker of interest,
    estimate its property. The biomarker can be either organ or anomaly. The dimension
    can be one of the number, length, size, volume, angle, density, intensity or
    texture of the organ or anomaly.
    Input: [Image] & [Object] & [Dim] & [Biomarker Mask] & [Biomarker Label]
    Output: [Quant]

7\. |Indicator Evaluation Model|
    Property: A calculation model
    Ability: Use prior patient information and several biomarkers values to calculate
    the indicator, the indicator can be a score or a grading.
    Input: [Priors] & [Indicator] & [Biomarkers] & [Quants]
    Output: [Value]

8\. |Report Generation Model|
    Property: A multimodal model
    Ability: Generate a medical report by integrating results processed by former tools.
    Input: [Image] & [Modality] & [Anatomy]
    Optional Input: ([Organ Mask] & [Organ Label]), ([Anomaly Mask] & [Anomaly Label]),
    [Diseases], ([Object] & [Dim] & [Quant]), ([Indicators] & [Values])
    Output: [Report]

9\. |Treatment Recommendation Model|
    Property: a language model
    Ability: Recommend personalized treatment plans based on all results processed by
    former tools and the patient’s information.
    Input: [Image] & [Priors] & [Modality] & [Anatomy] & [Diseases]
    Optional Input: ([Organ Mask] & [Organ Label]),([Anomaly Mask] & [Anomaly Label]),
    ([Object] & [Dim] & [Quant]), ([Indicators] & [Values])
    Output: [Treatment]

There are 11 different tasks and the chain of tools each task hopes to break down into:

(1) Basic Image Analysis and Organ Segmentation
    ToolUse: 012
    Description: Perform basic image analysis and segment organs within the medical
    image.

(2) Basic Image Analysis and Anomaly Detection
    ToolUse: 013
    Description: Perform basic image analysis and detect anomalies within the medical
    image.

(3) Image-based Direct Disease Diagnosis
    ToolUse: 014
    Description: Diagnose disease directly from the medical image without intermediate
    steps.

(4) Organ segmentation and anomaly localization
    ToolUse: 0123
    Description: Segment organs and locate anomalies within the medical image.

(5) Anomaly-based Disease Diagnosis
    ToolUse: 01235
    Description: Diagnose disease based on disease inference by finding abnormalities
    and the organ in which they occur.

(6) Organ Biomarker Quantification
    ToolUse: 0126
    Description: Quantify specific biomarkers related to organs in the medical image.

(7) Anomaly Biomarker Quantification
    ToolUse: 0136
    Description: Quantify specific biomarkers related to anomalies in the medical image.

(8) Disease and Anomaly Based Report Generation
    ToolUse: 01348
    Description: Generate a medical report based on detected diseases and anomalies.

(9) Disease and Biomarker Based Report Generation
    ToolUse: 0123568
    Description: Generate a comprehensive report incorporating anomaly detection,
    disease diagnosis and biomarker quantification.

(10) Comprehensive Evaluation Report Generation
    ToolUse: 01235678
    Description: Generate a detailed evaluation report including all aspects of the
    medical image analysis.

(11) Comprehensive Report Generation and Treatment Recommendations
    ToolUse: 012356789
    Description: Generate a comprehensive report including all analysis results and
    treatment recommendations.

Please generate 11 mutually independent question-answer pairs corresponding to tasks
above, based on the different task natures and the content of the case. Specifically,
strictly avoid including information
in the questions that should be determined by the tools (such as imaging modality,
specific anatomy, or precise abnormality types).
Pay attention: create task-specific question-answer pairs that highlight the unique
tool usage patterns for different tasks. The questions should be:

1\. Naturally aligned with the task description
2\. Representative of real-world scenarios for that task type
3\. Questions must not reveal information about modality or anatomy that should be
determined by Tools 0 and 1 or other tools. The questions should be phrased in a way
that necessitates the use of these basic identification tools.

Provide answers in the most concise free-text form possible. If visual results such as
masks are involved, please embed them in the text in the form of [Organ Mask] or
[Anomaly Mask]. (eg. The organ segmentation result is shown as [Organ Mask].) The 11
generated question-answer pairs should follow this template:

<Q1> ... </Q1>
<A1> ... </A1>
<Q2> ... </Q2>
<A2> ... </A2>
...
<Q11> ... </Q11>
<A11> ... </A11>

The answer only needs to provide a simple final result based on the information already
available in the Case, without showing the thought process. I will now provide you with
a Case containing all the information.

The patient record is {Patient Record}, please generate the corresponding QA pairs.

```

### Detailed Workflow Prompts in RadABench-EvalPlat

In this part we detail the prompt template used in the main workflow of our RadABench-EvalPlat.

Task Decomposition.

{mdframed}

[backgroundcolor=lightgray!20] Task Decomposition Prompts  

```
# Medical Image Analysis Assistant

## Task Overview
You are a radiological agent analyzing medical images.
For each query, you will receive:
    1\. A medical imaging examination (Image) of a patient (assume already provided)
    2\. Known patient Information including demographics, medical history,
    and main complaints.

Your task involves three sequential parts:

1\. Problem Decomposition (Part 1)
- Identify available information
- Break down the question into sequential steps

2\. Sequential Tool Application (Part 2)
- Execute one tool at a time
- Record each tool’s output
- Continue until sufficient information is gathered

3\. Solution Synthesis (Part 3)
- Integrate all results
- Generate final answer

## Available Information Categories
The following categories must be used exactly as written:

[’$Information$’, ’$Anatomy$’, ’$Modality$’, ’$Disease$’, ’$OrganObject$’,
’$OrganDim$’, ’$OrganQuant$’, ’$AnomalyObject$’, ’$AnomalyDim$’,
’$AnomalyQuant$’, ’$IndicatorName$’, ’$IndicatorValue$’, ’$Report$’, ’$Treatment$’]

Where:
- $Information$: Patient demographics
(e.g., "45-year-old male", "BMI: 24", "history of diabetes")
- $Anatomy$: Body part (e.g., "chest", "abdomen", "brain")
- $Modality$: Imaging technique (e.g., "X-ray", "CT", "MRI")
- $Disease$: Medical condition (e.g., "pneumonia", "cancer", "fracture")
- $OrganObject$: Organ to measure (e.g., "liver", "heart")
- $OrganDim$: Organ measurement type (e.g., "number", "length", "size",
  "volume", "angle", "density", "intensity", "texture")
- $OrganQuant$: Organ measurement value (e.g., "5cm", "120ml")
- $AnomalyObject$: Abnormality to measure (e.g., "tumor", "fracture")
- $AnomalyDim$: Abnormality measurement type (e.g., "number", "length", "size",
  "volume", "angle", "density", "intensity", "texture")
- $AnomalyQuant$: Abnormality measurement value (e.g., "2cm", "5ml")
- $IndicatorName$: Clinical indicator name
- $IndicatorValue$: Clinical indicator value
- $Report$: Medical report content
- $Treatment$: Treatment recommendations

## Available Tool Categories
Tool categories must be used exactly as written:

[*Anatomy Classification Tool*, *Modality Classification Tool*,
*Organ Segmentation Tool*, *Anomaly Detection Tool*,
*Disease Diagnosis Tool*, *Disease Inference Tool*,
*Organ Biomarker Quantification Tool*,
*Anomaly Biomarker Quantification Tool*, *Indicator Evaluation Tool*,
*Report Generation Tool*, *Treatment Recommendation Tool*]

## Response Format for Part 1
For each query, respond ONLY with:

Known Info: [list any categories explicitly mentioned in the query]
Tool Chain: [list required tools connected by ->]

## Examples

Query 1: "For a straightforward approach to diagnosing the patient’s condition
based on her symptoms and the image, what diseases can be directly identified?"
Response:
Known Info: []
Tool Chain: [*Anatomy Classification Tool* -> *Modality Classification Tool*
             -> *Disease Diagnosis Tool*]

Query 2: "This 45-year-old male’s chest CT shows a 2cm nodule in the right lung.
Can you give a report?"
Response:
Known Info: [’$Information$’, ’$Anatomy$’, ’$Modality$’, ’$AnomalyObject$’,
            ’$AnomalyDim$’, ’$AnomalyQuant$’]
Tool Chain: [*Organ Segmentation Tool* -> *Anomaly Detection Tool*
             -> *Disease Inference Tool* -> *Report Generation Tool*]
             (because some information is provided, so
             *Anatomy Classification Tool*, *Modality Classification Tool*,
             *Anomaly Biomarker Quantification Tool* are optimized.)

## Important Rules
1\. Assume the medical image is already provided
2\. Use exact item category names with $$ as listed (e.g., ’$Anatomy$’)
3\. Use exact tool category names with ** as listed
   (e.g., ’*Anatomy Classification Tool*’)
4\. Only respond with Part 1 analysis - Parts 2 & 3 will be addressed
   in subsequent interactions
5\. Include only the categories explicitly mentioned in the query
6\. Connect tools using -> symbol

Please wait for my query.
When provided, analyze it following the format shown in the examples above.

{Patient Record}

{Query}

```

Tool Selection & Execution.

{mdframed}

[backgroundcolor=lightgray!20] Tool Selection & Execution Prompts  

```
# Next Step Planning

## Current Status
Current results dictionary: {value_dict}

## Planning Guidelines
1\. Reference your high-level tool chain from Part 1 decomposition
2\. Consider current results to refine specific tool selection
3\. Maintain sequential progression according to planned workflow
4\. Adjust tool selection if needed based on intermediate results
5\. Check if the tool category is missing when this category of tools is required
6\. Check if the tool is suitable for the detected Anatomy and Modality in reserved
value dictionary based on the Tool description Ability and Property
7\. Check if the result in reserved value dictionary can be derived from each tool used
in former steps based on the limited label list described in Tool Ability
8\. If no suitable tool exists, identify which type of denial applies:
   - Missing tool category
   - Missing specific modality-anatomy tool
   - Insufficient tool capability

## Input Requirements
1\. Required inputs: Must include all mandatory inputs specified in tool description
2\. Optional inputs: Include if available and beneficial to tool performance
3\. Do not include variables that are not relevant to the tool’s function
4\. All variables must exist in the current results dictionary
5\. Use proper $$ notation for all variables

## Response Format
For ongoing analysis (if not final step):
<Call>
    <Purpose>Brief, clear statement of this step’s goal in context of overall
    analysis</Purpose>
    <Tool>TOOL[number] - must match available specific tools</Tool>
    <Input>[’$variable1$’, ’$variable2$’, ...] - use only existing variables from
    results</Input>
</Call>

For final step only:
<EndCall>
    <Purpose>Brief, clear statement of this final step’s goal</Purpose>
    <Tool>TOOL[number] - must match available specific tools</Tool>
    <Input>[’$variable1$’, ’$variable2$’, ...] - use only existing variables from
    results</Input>
</EndCall>

When a tool category is completely missing:
<NoCall>
    <Purpose>The purpose requiring a missing tool category</Purpose>
    <Category>The missing category from [’Anatomy Classifier’, ’Modality Classifier’,
    ’Organ Segmentor’, ’Anomaly Detector’, ’Disease Diagnoser’, ’Disease Inferencer’,
    ’Biomarker Quantifier’, ’Indicator Evaluator’, ’Report Generator’, ’Treatment
    Recommender’]</Category>
    <Anatomy>Universal</Anatomy>
    <Modality>Universal</Modality>
    <Ability>CategoryMissing</Ability>
</NoCall>

When specific tools for a modality-anatomy combination are missing:
<NoCall>
    <Purpose>The purpose requiring a specific modality-anatomy tool</Purpose>
    <Category>The required tool category</Category>
    <Anatomy>The specific anatomy from [’Universal’, ’Head and Neck’, ’Chest’,
    ’Breast’, ’Abdomen and Pelvis’, ’Limb’, ’Spine’]</Anatomy>
    <Modality>The specific modality from [’Universal’, ’X-ray’, ’CT’, ’MRI’,
    ’Ultrasound’]</Modality>
    <Ability>SpecificToolMissing</Ability>
</NoCall>

When existing tools lack required capabilities:
<NoCall>
    <Purpose>The purpose requiring advanced capabilities</Purpose>
    <Category>The category of existing but insufficient tools</Category>
    <Anatomy>The relevant anatomy</Anatomy>
    <Modality>The relevant modality</Modality>
    <Ability>InsufficientCapability</Ability>
</NoCall>

## Format Requirements
1\. Maintain proper XML structure
2\. Use exact tool numbers as specified in tool descriptions
3\. Mark all variables with $$ notation
4\. Include only existing variables from results dictionary
5\. Keep purpose statements clear and concise
6\. Brief response only includes one Call, EndCall, or NoCall XML part without
additional explanations
7\. For NoCall responses, use the appropriate format based on denial type

## Decision Making Process
1\. Review planned tool chain from Part 1
2\. Check current results in value dictionary
3\. Check if tool category is missing when this category of tools is required
4\. Check tool Ability and Property in detail to judge its suitability for detected
Anatomy and Modality in the value dictionary
5\. Check if the result in reserved value dictionary can be derived from each tool used
in former steps based on the limited label list described in Tool Ability
6\. Evaluate tool availability and capability:
   - Is the required tool category available?
   - Are specific tools available for the needed modality-anatomy combination?
   - Do available tools have sufficient capabilities?
7\. If tools are insufficient, use appropriate NoCall format
8\. If tools are available, select and format appropriate Call/EndCall
9\. Use <EndCall> only if this will be the final step

Please provide your next step based on:
- Original tool chain plan
- Current results
- Available specific tools
- Remaining analysis needs
- Tool availability and capability assessment

```

Response Generation. {mdframed}[backgroundcolor=lightgray!20] Response Generation Prompt  

```
Based on your Part 1 analysis plan, Part 2 tool execution sequence, and the final
results dictionary {value_dict}, provide:

    1\. A concise answer to the initial question
    2\. Key supporting evidence from your results
    3\. How your findings align with the planned analysis

Keep your response brief and focused on directly answer the initial question.

```

### An In-depth Case Study

Here we select a representative case from the agent core evaluation workflow to demonstrate the process, where we test the agent cores’ performance using a QA pair based on the task type "Anomaly-wise Biomarker Calculation" and a patient record (a 62-year-old female with hypertension, osteoarthritis of the knees, and chronic neck pain with stiffness radiating to right shoulder) under the NR. Denyl2 tool set condition, using Claude-3.5-Sonnet as the agent core, documenting each decision it makes. We can observe both the strengths and limitations reflected by this example.

Initial Input. The agent core receives the patient information along with a virtual image. For our evaluation task "Anomaly-wise Biomarker Calculation", we generate a corresponding QA pair based on the patient’s complete medical record. The initial input is provided below:

{mdframed}

[backgroundcolor=lightgray!20] Initial Input  

```
$Information$: {
    "Age": "62",
    "Sex": "Female",
    "Height": "165",
    "Weight": "72",
    "History": "Hypertension, osteoarthritis of the knees",
    "Complaint": "Chronic neck pain and stiffness, radiating pain to right shoulder"
},

$Query$: From an anomaly perspective in a specific medical image, after identifying
the type and area, could you quantify specific biomarker characteristics?

$Image$: ‘PLACEHOLDER_IMAGE’

```

Task Decomposition. The agent core processes these inputs by analyzing the patient information and identifying key data for extraction. It determines the appropriate high-level tool chain required for task completion. The agent core then stores all relevant information in a memory bank and establishes a tool category chain to guide subsequent execution. The process is illustrated below:

{mdframed}

[backgroundcolor=lightgray!20] Stage 1: Task Decomposition  

```
Initial Output: Known Info: []
Tool Chain: [*Anatomy Classification Tool* -> *Modality Classification Tool* ->
*Anomaly Detection Tool* -> *Anomaly Biomarker Quantification Tool*]
Initial Value Dict: {’$Image$’: ’PLACEHOLDER_IMAGE’, ’$Information$’: ’PLACEHOLDER_
INFORMATION’}
Initial Score Dict: {’$Image$’: 1.0, ’$Information$’: 1.0}
Initial Fixed Dict: frozendict.frozendict({’$Image$’: 1.0, ’$Information$’: 1.0})
High-level Tool chain: Anatomy Classification Tool -> Modality Classification Tool ->
Anomaly Detection Tool -> Anomaly Biomarker Quantification Tool

Memory bank: {’$Image$’: ’PLACEHOLDER_IMAGE’, ’$Information$’: ’PLACEHOLDER_
INFORMATION’}
Score bank: {’$Image$’: 1.0, ’$Information$’: 1.0}
Fixed bank: frozendict.frozendict({’$Image$’: 1.0, ’$Information$’: 1.0})

```

Tool Selection & Execution: The first phase of tool selection and execution commences. At each step, the agent core integrates the planned high-level tool chain with stored information from the memory bank to generate a tool API call, specifying the purpose, tool name, and input parameters. The Platform then executes the API call by activating the selected tool and processing the input. Upon successful execution, the tool’s output is stored in the memory bank. We configure the tool set according to the NR. Denyl2 condition, where there is a mismatch between the anomaly detection tool and the provided $Image$ characteristics (Head and Neck X-ray) in terms of anatomy and modality. The established tool set is as follows:

{mdframed}

[backgroundcolor=lightgray!20] Tool Set Description  

```

"TOOL1": {
    "Name": "TOOL1",
    "Category": "Anatomy Classifier",
    "Ability": "Determine the anatomy of the Image.",
    "Property": "Universal Anatomy Classifier",
    "Compulsory Input": [
        "$Image$"
    ],
    "Optional Input": [],
    "Output": [
        "$Anatomy$"
    ],
    "Performance": "Score from 0.95 to 0.95, increases with optional inputs"
},
"TOOL2": {
    "Name": "TOOL2",
    "Category": "Modality Classifier",
    "Ability": "Determine the modality of the Image.",
    "Property": "Universal Modality Classifier",
    "Compulsory Input": [
        "$Image$"
    ],
    "Optional Input": [],
    "Output": [
        "$Modality$"
    ],
    "Performance": "Score from 0.95 to 0.95, increases with optional inputs"
},

...

"TOOL5": {
    "Name": "TOOL5",
    "Category": "Anomaly Detector",
    "Ability": "Given the Limb Ultrasound Image, determine the location and
    type of abnormality.",
    "Property": "Anomaly Detector only suitable for Limb Ultrasound image",
    "Compulsory Input": [
        "$Image$"
    ],
    "Optional Input": [],
    "Output": [
        "$AnomalyMask$",
        "$AnomalyObject$"
    ],
    "Performance": "Score from 0.8 to 0.8, increases with optional inputs"
},
"TOOL6": {
    "Name": "TOOL6",
    "Category": "Anomaly Detector",
    "Ability": "Given the Breast MRI Image, determine the location and type
    of abnormality.",
    "Property": "Anomaly Detector only suitable for Breast MRI image",
    "Compulsory Input": [
        "$Image$"
    ],
    "Optional Input": [],
    "Output": [
        "$AnomalyMask$",
        "$AnomalyObject$"
    ],
    "Performance": "Score from 0.8 to 0.8, increases with optional inputs"
},

...

"TOOL8": {
    "Name": "TOOL8",
    "Category": "Disease Diagnoser",
    "Ability": "Given the Head and Neck X-ray Image, diagnose the disease.",
    "Property": "Disease Diagnoser only suitable for Head and Neck X-ray image",
    "Compulsory Input": [
        "$Image$"
    ],
    "Optional Input": [],
    "Output": [
        "$Disease$"
    ],
    "Performance": "Score from 0.75 to 0.75, increases with optional inputs"
},

...

"TOOL10": {
    "Name": "TOOL10",
    "Category": "Biomarker Quantifier",
    "Ability": "Measure the anomaly biomarker of the Image.",
    "Property": "Universal Anomaly Biomarker Quantifier",
    "Compulsory Input": [
        "$Image$",
        "$AnomalyObject$",
        "$AnomalyMask$"
    ],
    "Optional Input": [
        "$AnomalyDim$"
    ],
    "Output": [
        "$AnomalyDim$",
        "$AnomalyQuant$"
    ],
    "Performance": "Score from 0.75 to 0.8, increases with optional inputs"
},

...

```

In this initial step, the agent core selects the anatomy classification tool. We observe that the anatomical information is successfully stored in the memory bank after the tool execution:

{mdframed}

[backgroundcolor=lightgray!20] Tool Selection & Execution Step 1  

```
Step 1 starts!
Stepoutput: Based on the tool chain from Part 1 (Anatomy Classification Tool ->
Modality Classification Tool -> Anomaly Detection Tool -> Anomaly Biomarker
Quantification Tool) and current results dictionary, I’ll proceed with the first step:

<Call>
    <Purpose>Determine the anatomy region of the medical image</Purpose>
    <Tool>TOOL1</Tool>
    <Input>[’$Image$’]</Input>
</Call>
Call Dict: {’Purpose’: ’Determine the anatomy region of the medical image’, ’Tool’:
’TOOL1’, ’Input’: [’$Image$’]}
Memory bank: {’$Image$’: ’PLACEHOLDER_IMAGE’, ’$Information$’: ’PLACEHOLDER_
INFORMATION’, ’$Anatomy$’: ’Head and Neck’}
Score bank: {’$Image$’: 1.0, ’$Information$’: 1.0, ’$Anatomy$’: 0.95}
Fixed bank: frozendict.frozendict({’$Image$’: 1.0, ’$Information$’: 1.0})
Step 1 completes!

```

In the second step, the modality classification tool is selected and executed. The successful execution is demonstrated below. As we can observe that the modality information is added to memory bank after tool execution.

{mdframed}

[backgroundcolor=lightgray!20] Tool Selection & Execution Step 2  

```
Step 2 starts!
Stepoutput: Based on the tool chain and current results showing Head and Neck anatomy,
the next step is to determine the image modality:

<Call>
    <Purpose>Determine the imaging modality of the medical image</Purpose>
    <Tool>TOOL2</Tool>
    <Input>[’$Image$’]</Input>
</Call>
Call Dict: {’Purpose’: ’Determine the imaging modality of the medical image’, ’Tool’:
’TOOL2’, ’Input’: [’$Image$’]}
Memory bank: {’$Image$’: ’PLACEHOLDER_IMAGE’, ’$Information$’: ’PLACEHOLDER_
INFORMATION’, ’$Anatomy$’: ’Head and Neck’, ’$Modality$’: ’X-ray’}
Score bank: {’$Image$’: 1.0, ’$Information$’: 1.0, ’$Anatomy$’: 0.95, ’$Modality$’:
0.95}
Fixed bank: frozendict.frozendict({’$Image$’: 1.0, ’$Information$’: 1.0})
Step 2 completes!

```

In the third step, contrary to the expected utilization of the organ segmentation tool, the disease diagnosis tool is invoked instead. While this tool selection is irrelevant to the current task, the execution completes without errors. The tool call is detailed below:

{mdframed}

[backgroundcolor=lightgray!20] Tool Selection & Execution Step 3  

```
Step 3 starts!
Stepoutput: Given the current results showing Head and Neck anatomy and X-ray modality,
the next step is to detect any anomalies in the medical image:

<Call>
    <Purpose>Detect abnormalities in the Head and Neck X-ray image</Purpose>
    <Tool>TOOL8</Tool>
    <Input>[’$Image$’]</Input>
</Call>
Call Dict: {’Purpose’: ’Detect abnormalities in the Head and Neck X-ray image’, ’Tool’:
’TOOL8’, ’Input’: [’$Image$’]}
Memory bank: {’$Image$’: ’PLACEHOLDER_IMAGE’, ’$Information$’: ’PLACEHOLDER_
INFORMATION’, ’$Anatomy$’: ’Head and Neck’, ’$Modality$’: ’X-ray’, ’$Disease$’:
’Cervical spine degenerative changes’}
Score bank: {’$Image$’: 1.0, ’$Information$’: 1.0, ’$Anatomy$’: 0.95, ’$Modality$’:
0.95, ’$Disease$’: 0.75}
Fixed bank: frozendict.frozendict({’$Image$’: 1.0, ’$Information$’: 1.0})
Step 3 completes!

```

During the fourth step, when selecting the anomaly detection tool, the agent core correctly identifies that the available tools in the set lack the capability to detect and localize anomalies in Head and Neck X-ray images. As demonstrated below, the agent core appropriately rejects this invalid operation and successfully recognizes the tools’ capability gap.

{mdframed}

[backgroundcolor=lightgray!20] Tool Selection & Execution Step 4  

```
Step 4 starts!
Stepoutput: Before we can quantify biomarkers (per the original tool chain), we need
the anomaly detection results first. Looking at the available tools for Head and Neck
X-ray anomaly detection:

<NoCall>
    <Purpose>Detect specific anomalies in Head and Neck X-ray for biomarker
    quantification</Purpose>
    <Category>Anomaly Detector</Category>
    <Anatomy>Head and Neck</Anatomy>
    <Modality>X-ray</Modality>
    <Ability>SpecificToolMissing</Ability>
</NoCall>
Deny Call Dict: {’Purpose’: ’Detect specific anomalies in Head and Neck X-ray for
biomarker quantification’, ’Category’: ’Anomaly Detector’, ’Anatomy’: ’Head and Neck’,
’Modality’: ’X-ray’, ’Ability’: ’SpecificToolMissing’}Agent denies the call because no
suitable tool is found.

```

Overall, while the agent core demonstrates correct initialization and tool chain planning, it exhibits a deviation during the tool selection & execution stage. Despite having access to tool chain guidance in each round, the agent core fails to follow these directives, resulting in an unnecessary disease diagnosis tool call. Nevertheless, the execution proceeds without any IO errors, and the agent core successfully identifies the tools’ capability limitations, appropriately concluding with a denial of task execution due to the missing functionality.