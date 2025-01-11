<!--yml
category: 未分类
date: 2025-01-11 12:17:24
-->

# HoneyComb: A Flexible LLM-Based Agent System for Materials Science

> 来源：[https://arxiv.org/html/2409.00135/](https://arxiv.org/html/2409.00135/)

Huan Zhang¹ , Yu Song¹ , Ziyu Hou² , Santiago Miret³ , Bang Liu^(1,4)¹¹footnotemark: 1
¹University of Montreal / Mila - Quebec AI, ²University of Waterloo,
³Intel Labs, ⁴Canada CIFAR AI Chair
{huan.zhang, yu.song, bang.liu}@umontreal.ca
{z26hou}@uwaterloo.ca
{santiago.miret}@intel.com Equal advising.Corresponding author.

###### Abstract

The emergence of specialized large language models (LLMs) has shown promise in addressing complex tasks for materials science. Many LLMs, however, often struggle with distinct complexities of material science tasks, such as materials science computational tasks, and often rely heavily on outdated implicit knowledge, leading to inaccuracies and hallucinations. To address these challenges, we introduce HoneyComb, the first LLM-based agent system specifically designed for materials science. HoneyComb leverages a novel, high-quality materials science knowledge base (MatSciKB) and a sophisticated tool hub (ToolHub) to enhance its reasoning and computational capabilities tailored to materials science. MatSciKB is a curated, structured knowledge collection based on reliable literature, while ToolHub employs an Inductive Tool Construction method to generate, decompose, and refine API tools for materials science. Additionally, HoneyComb leverages a retriever module that adaptively selects the appropriate knowledge source or tools for specific tasks, thereby ensuring accuracy and relevance. Our results demonstrate that HoneyComb significantly outperforms baseline models across various tasks in materials science, effectively bridging the gap between current LLM capabilities and the specialized needs of this domain. Furthermore, our adaptable framework can be easily extended to other scientific domains, highlighting its potential for broad applicability in advancing scientific research and applications.

![[Uncaptioned image]](img/17a8232d423e33456855d38766572a43.png) HoneyComb: A Flexible LLM-Based Agent System for Materials Science

Huan Zhang¹ , Yu Song¹ , Ziyu Hou² , Santiago Miret³^†^†thanks: Equal advising. , Bang Liu^(1,4)¹¹footnotemark: 1^†^†thanks: Corresponding author. ¹University of Montreal / Mila - Quebec AI, ²University of Waterloo, ³Intel Labs, ⁴Canada CIFAR AI Chair {huan.zhang, yu.song, bang.liu}@umontreal.ca {z26hou}@uwaterloo.ca {santiago.miret}@intel.com

## 1 Introduction

The emergence of large language models (LLMs) (OpenAI, [2024](https://arxiv.org/html/2409.00135v1#bib.bib26); Anthropic, [2024](https://arxiv.org/html/2409.00135v1#bib.bib3); Touvron et al., [2023b](https://arxiv.org/html/2409.00135v1#bib.bib35), [a](https://arxiv.org/html/2409.00135v1#bib.bib34)) in recent years has brought about the application of LLMs across a wide range of fields related to science and engineering (AI4Science and Quantum, [2023](https://arxiv.org/html/2409.00135v1#bib.bib1)). This has resulted in a number of new benchmarks measuring the capabilities of language models to perform scientific tasks (Wang et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib40); Sun et al., [2024](https://arxiv.org/html/2409.00135v1#bib.bib31); Mirza et al., [2024](https://arxiv.org/html/2409.00135v1#bib.bib25); Song et al., [2023a](https://arxiv.org/html/2409.00135v1#bib.bib29)) along with the development of custom LLMs and LLM-based systems for scientific domains including chemistry (Bran et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib5); Boiko et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib4)), biology (Madani et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib22)) and materials science (Song et al., [2023b](https://arxiv.org/html/2409.00135v1#bib.bib30); Gupta et al., [2022](https://arxiv.org/html/2409.00135v1#bib.bib13); Walker et al., [2021](https://arxiv.org/html/2409.00135v1#bib.bib38)).

While much progress has been made in adapting LLMs to common tasks in natural language processing (Song et al., [2023a](https://arxiv.org/html/2409.00135v1#bib.bib29), [b](https://arxiv.org/html/2409.00135v1#bib.bib30)), many more challenges remain in having LLMs be effective agents for real-world materials science tasks (Miret and Krishnan, [2024](https://arxiv.org/html/2409.00135v1#bib.bib23); Miret et al., [2024](https://arxiv.org/html/2409.00135v1#bib.bib24)). As highlighted by Zaki et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib44)), LLMs often fail in performing important computational tasks for materials science. Common mistakes by most LLMs include conceptual errors where models fail to retrieve correct concepts, equations, or facts relevant to the questions, and factual hallucinations where incorrect information is generated. An analysis by Miret and Krishnan ([2024](https://arxiv.org/html/2409.00135v1#bib.bib23)) also revealed that LLMs by themselves struggle to generate relevant and correct information pertaining to specialized materials science tasks. While Song et al. ([2023b](https://arxiv.org/html/2409.00135v1#bib.bib30)) showed that instruction fine-tuning can help in improving performance, the high costs of continuous model training and fine-tuning make retraining-based approaches challenging to scale. This is further compounded by the fact that relevant knowledge is continuously updated through a diversity of knowledge sources, including pre-print servers (e.g., arXiv and ChemRxiv) , peer-reviewed literature, open encyclopedias (e.g. Wikipedia) and relevant websites. Furthermore, prior work has show that utilizing external tools may be a more promising approach to solve complex scientific tasks instead of relying entirely on an LLMs internal knowledge (Zheng et al., [2024](https://arxiv.org/html/2409.00135v1#bib.bib45); Buehler, [2024a](https://arxiv.org/html/2409.00135v1#bib.bib6)). To jointly address these challenges, we propose transforming LLMs into LLM-based agents that access external knowledge and tools to boost their performance. This approach has already shown promise in adjacent domains, such as chemistry (Bran et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib5); Boiko et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib4)) by enabling the the models to access real-time data and utilize computational as well as domain-specific tools. Altogether, the LLM-based agents showcase greater capabilities and performance compared to their native LLM counterparts.

In this paper, we present HoneyComb, the first, to the best of our knowledge, LLM-based agent system specifically designed for materials science. While there has been emerging research in LLMs for scientific domains, few studies have focused on developing comprehensive agent systems for materials science. Our work addresses two critical challenges: First, MatSciKB alleviates the challenge of obtaining reliable and relevant professional knowledge for materials. As such, MatSciKB ensures the agent has access to the most current and accurate information is essential for effective performance. Second, Tool-Hub provides materials science specific tools to augment the agent’s capabilities. These tools enable the agent to perform specialized computational tasks and enhance its overall functionality. As detailed in [Section 4](https://arxiv.org/html/2409.00135v1#S4 "4 Experiments ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science"), we observe that with the aid of MatSciKB and Tool-Hub, HoneyComb outperforms its native LLM counterparts in a more reliable manner given its ability to utilize up-to-date knowledge and tools.

## 2 Background

### 2.1 LLMs for Material Science

Advancements in text mining and information extraction from scientific publications have significantly benefited the application of LLMs for materials science (Kononova et al., [2021](https://arxiv.org/html/2409.00135v1#bib.bib18); Swain and Cole, [2016](https://arxiv.org/html/2409.00135v1#bib.bib32)). Early work include the development of specialized BERT models (Devlin et al., [2018](https://arxiv.org/html/2409.00135v1#bib.bib10)), such as MatSciBERT (Gupta et al., [2022](https://arxiv.org/html/2409.00135v1#bib.bib13)) and MatBERT (Walker et al., [2021](https://arxiv.org/html/2409.00135v1#bib.bib38)). Song et al. ([2023b](https://arxiv.org/html/2409.00135v1#bib.bib30)) and Xie et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib41)) leveraged instruction fine-tuning to develop a LlaMa-based (Touvron et al., [2023a](https://arxiv.org/html/2409.00135v1#bib.bib34)) tailored to materials science that matched the capabilities of commercial LLMs at the time of publication. The emergence of powerful commercial LLMs (OpenAI, [2024](https://arxiv.org/html/2409.00135v1#bib.bib26); Anthropic, [2024](https://arxiv.org/html/2409.00135v1#bib.bib3)) has further expanded the possibility of applying LLMs to materials science. Yet, commercial LLMs remain expensive, opaque in their methodology with consistent errors and shortcomings (Zaki et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib44); Miret and Krishnan, [2024](https://arxiv.org/html/2409.00135v1#bib.bib23)), and open-source LLMs for materials science remain sparse. This motivates the need for a practical LLM-based system that is useful for real-world materials science tasks.

Given this need, we propose HoneyComb as an open-source system to augment the capabilities of diverse LLMs. HoneyComb integrates specialized tools as well as a dynamic retrieval system to enhance the functionality any LLMs specifically for material science. By leveraging relevant knowledge source through MatSciKB and auxilliary tools through Tool-Hub, HoneyComb manages to improve the accuracy and relevance of the outputs of LLMs for materials science, while also addressing common challenges associated with static LLM applications in dynamic research fields.

### 2.2 Tool-Based LLM Agents for Scientific Applications

Prior work has shown success in expanding the capabilities of LLMs by augmenting their capabilities with diverse sets of tools (Qin et al., [2023b](https://arxiv.org/html/2409.00135v1#bib.bib28), [a](https://arxiv.org/html/2409.00135v1#bib.bib27); Chern et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib9); Wang et al., [2024](https://arxiv.org/html/2409.00135v1#bib.bib39)). Many works rely on pre-built integration frameworks, such as LangChain (Topsakal and Akinci, [2023](https://arxiv.org/html/2409.00135v1#bib.bib33)), to build the relevant interfaces between the LLMs and the desired capabilities, such as search engine APIs. Wang et al. ([2024](https://arxiv.org/html/2409.00135v1#bib.bib39)) provides a recent survey of common approaches, challenges and applications of tool-based LLMs and their applications to various technology and scientific fields.

One major application of tool-based LLMs is in query processing and optimization, where agents evaluate initial search results and iteratively refine queries to increase relevance and accuracy Buehler ([2024a](https://arxiv.org/html/2409.00135v1#bib.bib6), [b](https://arxiv.org/html/2409.00135v1#bib.bib7)). This approach addresses the limitations of isolated LLMs, which may struggle to handle ambiguous query contexts. In generating structured datasets for solar cell materials, agents gather pertinent information from a vast array of scientific papers to automate data input and synthesis Xie et al. ([2024](https://arxiv.org/html/2409.00135v1#bib.bib42)); Liu et al. ([2024b](https://arxiv.org/html/2409.00135v1#bib.bib21)). Furthermore, agents can utilize various tools to help answer specific questions by tapping into external resources Cheng et al. ([2024](https://arxiv.org/html/2409.00135v1#bib.bib8)). For example, ChemCrow by Bran et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib5)) integrates 18 expert-designed tools, such as literature search, molecule modification, and reaction execution, to autonomously execute chemical syntheses. Tool augmentation has also shown success in other research in the chemistry domain to enable real-world experiments using LLMs (Yoshikawa et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib43); Jablonka et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib16); Boiko et al., [2023](https://arxiv.org/html/2409.00135v1#bib.bib4)). Coscientist by Boiko et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib4)), for examples, relies on specialized tools to extend the capabilities of GPT4 and thereby invoke domain-specific functionalities that are not inherently present within the LLM alone. The success of agent-based approaches in adjacent domains motivates the creation of HoneyComb that extends the capabilities of LLMs specifically for materials science.

## 3 HoneyComb

In this work, we introduce HoneyComb, shown in Figure [1](https://arxiv.org/html/2409.00135v1#S3.F1 "Figure 1 ‣ 3 HoneyComb ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science"), a specialized agent system designed to advance materials science research. It integrates three key components: 1) MatSciKB, a comprehensive knowledge base; 2) ToolHub, which includes general tools for accessing up-to-date information broadly and specialized tools developed through an Inductive Tool Construction method for targeted material science queries; 3) Retriever, utilizing a hybrid approach for efficient and precise information retrieval.

![Refer to caption](img/7dc3b99ba14a3b8496f0f9e2504d6e2e.png)

Figure 1: The overall architecture of HoneyComb. The model initiates with a query input that activates the knowledge retrieval phase, where pertinent data entries and atom function are extracted from the MatSciKB and Tool-Hub respectively. The Executor iterative calls the relevant tools from the Tool-Hub, evaluating and refining these calls until a solution that adequately solves the query emerges. The preliminary solution generated by these tools is combined with relevant data entries, and then undergoes further processing by the Retriever. Finally, the Retriever consolidates and filters these input, ultimately feeding them into the LLM for final answer generation.

### 3.1 MatSciKB

Our MatSciKB knowledge base integrates a diverse array of sources, as detailed in Table [1](https://arxiv.org/html/2409.00135v1#S3.T1 "Table 1 ‣ 3.1 MatSciKB ‣ 3 HoneyComb ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science"). This collection is meticulously curated to include material science papers from ArXiv, relevant Wikipedia entries, textbooks, comprehensive datasets, pertinent mathematical formulas, and concrete GPT-generated examples tailored to material science. Each information source is thoroughly described in Appendix [A](https://arxiv.org/html/2409.00135v1#A1 "Appendix A MatSciKB Knowledge Source ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science").

The architectural framework of MatSciKB is thoughtfully structured into 16 distinct categories pertinent to material science. These are detailed in Appendix [C](https://arxiv.org/html/2409.00135v1#A3 "Appendix C Tree-Structure MatSciKB ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science") and are organized in a tree-like structure. MatSciKB supports efficient searching and CRUD (Create, Read, Update, Delete) operations Giannaros et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib11)), which are vital for both the application and ongoing maintenance of the database. Given the continuously evolving and expanding body of knowledge in the materials science domain, capabilities for efficient updates and searches based on real-time information are crucial for research and engineering applications. Additionally, our structured data approach enhances the integration of the diverse data sources commonly encountered in materials science (Miret and Krishnan, [2024](https://arxiv.org/html/2409.00135v1#bib.bib23)). This structure not only facilitates easy access and management but also allows for seamless extension to include additional data modalities.

| MatSciKB |
| # Total Number of Data Entries | 38,469 |
|         # Material Science Papers on Arxiv | 20,384 |
|         # Wikipedia for Material Science | 3,620 |
|         # Material Science Textbook | 1,930 |
|         # Material Science Dataset | 10,473 |
|         # Material Science Formula | 57 |
|         # GPT-generated Examples | 2,005 |

Table 1: Statistics of the MatSciKB knowledge base

### 3.2 Tool-Hub

The Tool-Hub in HoneyComb is bifurcated into General Tools and Material Science Tools. Both categories are organized through a unified interface that supports allow HoneyComb to make effective use of all available tools. General Tools provide researchers with access to the latest information filling gaps not covered by the static entries in MatSciKB. Material Science Tools are specifically designed to handle complex calculations and in-depth analyses. The details of the unified interface are further elucidated in Appendix [D](https://arxiv.org/html/2409.00135v1#A4 "Appendix D Tools Unified Interface Using LangChain ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science").

General Tools Construction

In materials science, one of the persistent challenges is keeping research outputs aligned with the diverse and ever-evolving data modalities that describe complex material systems (Miret and Krishnan, [2024](https://arxiv.org/html/2409.00135v1#bib.bib23)). The diversity of data sources and measurements leads to a rapid evolution of knowledge in this field, necessitating tools that can effectively access and integrate recent findings. Traditional static databases, while useful, often lag in capturing the newest research, creating gaps that can impede the currency and relevance of scientific analysis in real-time. Further, the need to efficiently process complex and dynamic computational tasks within the research workflow remains inadequately addressed, often requiring manual intervention which can introduce errors and inefficiencies. Thus, constructing tools that can handle varying data modalities and complexities, and that can adapt to the continual advancements in materials science, is essential for advancing the field.

To address these challenges, HoneyComb has been designed with innovative solutions that markedly enhance research capabilities in materials science. First, we integrated General Tools that provide direct access to current publications and facilitate dynamic discussions, as shown in Table [2](https://arxiv.org/html/2409.00135v1#S3.T2 "Table 2 ‣ 3.2 Tool-Hub ‣ 3 HoneyComb ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science"), effectively complementing the static MatSciKB. Secondly, recognizing the limitations of large language models (LLMs) in performing computational tasks, we implemented a Python REPL environment within HoneyComb. This environment is strategically utilized by the system when the agent, interacting with the Tool-Hub, identifies a need for basic numerical computations. The agent dynamically writes Python code for these tasks and executes it through the Python REPL, bypassing the LLM’s computational limitations. This automation not only streamlines data processing but also enhances the precision and reliability of numerical analyses in research activities.

| General Tools | 6 |
| Google Search |  |
| Google Scholar Search |  |
| Arxiv Search |  |
| Wikipedia Search |  |
| YouTube Search |  |
| Python REPL |  |

Table 2: ToolHub: General Tools

Algorithm 1 Inductive Tool Construction

0:Train Set $D_{train}$, LLM $M$0:Set of atom tools $A$1:  $A\leftarrow\emptyset$ {Initialize the set of atom functions}2:for each question $q_{i}$ in $D_{train}$ do3:     $f_{i}\leftarrow M(q_{i})$ {Generate specific function for $q_{i}$}4:Human verifies $f_{i}$5:Decompose $f_{i}$ into atom functions $a_{i}$6:     $A\leftarrow A\cup a_{i}$ {Add atom functions to the set}7:  end for8:  return  $A$

Inductive Tool Construction for Materials Science

Constructing domain-specific tool APIs presents significant challenges. It requires domain expert knowledge, and there are limited existing resources to draw upon. Additionally, many valuable data and tools are not open source, limiting their accessibility. Developing these tools is essential for effectively addressing the unique and complex queries inherent to materials science. The scarcity of pre-existing, specialized computational tools necessitates a methodical approach to tool construction and refinement.

We propose the Inductive Tool Construction method, delineated in Algorithm [1](https://arxiv.org/html/2409.00135v1#alg1 "Algorithm 1 ‣ 3.2 Tool-Hub ‣ 3 HoneyComb ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science") for domain-specific tool APIs construction. It adopts a systematic approach to fabricate and refine computational tools specifically designed for material science queries. The process initiates by selecting a random subset of computational questions from dataset $D$, designated as $D_{train}$ for training, with the residual questions forming $D_{test}$. For each question $q_{i}\in D_{train}$, a designated LLM, $M$ (such as GPT-4), is tasked to generate a Python function $f_{i}$ that addresses $q_{i}$. After creation, each function $f_{i}$ undergoes rigorous human verification to confirm its correctness.

However, the above procedures cannot ensure the generalizability of the constructed tool APIs. Thus, in the post-validation stage, we further use $M$ to decompose each $f_{i}$ into fundamental, reusable components known as atomic function $a_{i}$, which are crafted for extensive applicability across diverse queries, a detailed example is illustrated in Appendix [E](https://arxiv.org/html/2409.00135v1#A5 "Appendix E Examples of Inductive Tool Construction ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science")

### 3.3 Agent-Tool Hub Interactions

In HoneyComb, interactions between the agent and Tool-Hub are governed by a structured two-phase decision-making protocol. Our protocol emphasizes the critical selection and processing of data to ensure that only pertinent information influences the LLM’s decisions. This approach is vital to prevent the degradation of model performance due to irrelevant or low-quality inputs Liu et al. ([2024a](https://arxiv.org/html/2409.00135v1#bib.bib20)).

1\. Tool Assessor: During the initial phase, the Assessor evaluates both the incoming query and the extensive suite of tools within the Tool-Hub. This evaluation aims to identify a manageable subset of the most relevant tools that are best suited to address the specific requirements of the query. By filtering out irrelevant tools at this stage, we ensure that the Executor is provided only with pertinent information, thereby optimizing the model’s focus and enhancing its capacity to solve the problem accurately.

2\. Tool Executor: As illustrated in Figure [2](https://arxiv.org/html/2409.00135v1#S3.F2 "Figure 2 ‣ 3.3 Agent-Tool Hub Interactions ‣ 3 HoneyComb ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science"), the Executor receives the original query along with the subset of tools selected by the Assessor. Upon evaluating the selected tools and query, the Executor engages in a thought process to determine the most suitable tool for addressing the query. If the query’s complexity exceeds the capacity of a single tool, the Executor recognizes the challenge and decomposes the query into smaller subquestions. The strategy allows for sequential tackling of each part, starting with the selection of the optimal tool for the initial subquestion. It then initiates the action of executing the selected tool while inputting parameter values, termed action input, derived from the query or subquestion. Upon execution, the tool generates a result termed observation. Subsequently, the Executor engages in a reflective process to assess whether the observation adequately addresses the query. If the observation is adequate, it is finalized as the answer; if not, the process either reiterates with adjustments or progresses to the next subquestion if the original query was segmented into multiple parts.

![Refer to caption](img/dee0a175de1839f325b9cf14dd86bcff.png)

Figure 2: Tool Assessor and Executor interaction cycle in HoneyComb.

### 3.4 Retriever

In this section, we present the retriever in HoneyComb which returns relevant texts or tools from MatSciKB and Tool-Hub when a specific contexts is given. The retriever integrates both BM25 (Trotman et al., [2014](https://arxiv.org/html/2409.00135v1#bib.bib36)) and Contriever (Izacard et al., [2022](https://arxiv.org/html/2409.00135v1#bib.bib15)) model, leveraging their respective strengths to achieve optimal information retrieval performance.

Specifically, the retriever employs a two-step strategy. Initially, BM25 utilizes efficient calculations of term frequency and inverse document frequency to rapidly process short text queries and keyword searches within long documents. The primary advantage of BM25 lies in its computational simplicity and rapid response, allowing HoneyComb to extract the N most relevant knowledge points from an extensive materials science knowledge base, ensuring exceptional speed and efficiency. This approach enables the provision of basic relevance matching results in a minimal timeframe.

Subsequently, we employs a pre-trained deep learning models (i.e. Contriever) to generate embedding vectors and compute their similarity, facilitating the understanding of complex linguistic structures and semantic information. The strength of Contriever resides in its capability to comprehend and process intricate language structures, contextual information, and semantic relationships, thereby delivering more precise and comprehensive retrieval results. Although Contriever operates at a slower pace compared to BM25, it pulls the most relevant results from the knowledge base and memory, as well as from tools invoked through the Tool-Hub, extracting the top 3 results. Its ability to precisely handle complex queries and diverse documents ensures high accuracy and relevance.

By combining BM25 and Contriever, our model adeptly responds to simple queries with speed while offering enhanced accuracy and relevance for complex queries. This hybrid approach ensures that the model is both efficient and capable of addressing sophisticated query requirements, thereby providing comprehensive, efficient, and precise information retrieval services.

## 4 Experiments

We conduct experiments on two question answering datasets, namely MaScQA Zaki et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib44)) and SciQA Johannes Welbl ([2017](https://arxiv.org/html/2409.00135v1#bib.bib17)), to investigating the ablility of HoneyComb in materials science tasks.

MaScQA, derived from the Graduate Aptitude Test in Engineering (GATE) in India, is tailored to reflect the real-world complexity and variety of issues encountered in material science. This highly competitive examination assesses a comprehensive understanding of various undergraduate subjects Indian Institute of Technology Kanpur ([2023](https://arxiv.org/html/2409.00135v1#bib.bib14)); Zaki et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib44)). With its 650 questions covering 14 domains such as thermodynamics, atomic structure, and mechanical behavior, the dataset showcases a wide range of question types, from Multiple Choice Questions (MCQs), Numerical Answer Type (NUM), and Matching Type (MATCH) to MCQs with numerical options (MCQN). Specifically designed for advanced problem-solving, this dataset is crucial for ensuring that our ToolHub functions effectively in actual material science research and applications. It demonstrates the HoneyComb framework’s efficacy and adaptability in tackling complex material science issues within realistic scenarios. The second dataset, SciQA, comprises 11,679 multiple-choice questions that span the core disciplines of fundamental sciences from a variety of crowdsourced science exams Johannes Welbl ([2017](https://arxiv.org/html/2409.00135v1#bib.bib17)). This compilation not only underlines the dataset’s comprehensive and interdisciplinary nature but also focuses on fostering a nuanced conceptual understanding. SciQA serves as a critical testbed to ascertain whether the HoneyComb framework can augment the LLM’s capabilities beyond its initial programming. By integrating supplementary information, it aids in addressing intricate queries and unraveling complex scientific concepts that may have been overlooked during the initial training phase of the LLM. By bridging real-world complexities with rigorous academic standards, these datasets ensure that our MatSciKB and ToolHub are not only versatile but also remain at the forefront of technological and scientific application.

The choice of models for our experiments was driven by the need to evaluate the HoneyComb framework’s enhancement capabilities across a spectrum of large language models known for their robust performance in diverse applications. We selected GPT-3.5, GPT-4 OpenAI ([2024](https://arxiv.org/html/2409.00135v1#bib.bib26)), LLaMA-2 Touvron et al. ([2023b](https://arxiv.org/html/2409.00135v1#bib.bib35)), and LLaMA-3 AI@Meta ([2024](https://arxiv.org/html/2409.00135v1#bib.bib2)) due to their widespread use and proven effectiveness in handling complex language tasks. These models, with LLaMA-2 and LLaMA-3 having parameter sizes of 7 billion and 8 billion respectively, represent the current state-of-the-art in generalized language understanding and provide a solid baseline for benchmarking. Additionally, we included HoneyBeeSong et al. ([2023b](https://arxiv.org/html/2409.00135v1#bib.bib30)), a specialized model with a parameter size of 7 billion, tailored specifically for materials science. The inclusion of both general-purpose and specialized models allows us to showcase how domain-specific adaptations through HoneyComb can elevate a model’s functional scope beyond its original configuration, thus highlighting the adaptability and effectiveness of our framework.

### 4.1 HoneyComb Evaluation

 Dataset HoneyBee HoneyBee + ![[Uncaptioned image]](img/aaafd140ff0698f4000203e165c3116d.png) GPT-3.5 GPT-3.5 + ![[Uncaptioned image]](img/aaafd140ff0698f4000203e165c3116d.png) GPT-4 GPT-4 + ![[Uncaptioned image]](img/aaafd140ff0698f4000203e165c3116d.png) Llama2 Llama2 + ![[Uncaptioned image]](img/aaafd140ff0698f4000203e165c3116d.png) Llama3 Llama3 + ![[Uncaptioned image]](img/aaafd140ff0698f4000203e165c3116d.png)  MaScQA 16.62 33.38 33.54 38.46 58.46 79.07 22.15 36.31 24.62 47.23  SciQA 33.96 79.69 90.69 90.83 90.84 96.54 75.79 78.66 93.00 93.32 

Table 3: HoneyComb evaluation with diverse LLMs including open-source LLMs (HoneyBee (Song et al., [2023b](https://arxiv.org/html/2409.00135v1#bib.bib30)), LlaMa2 (Touvron et al., [2023b](https://arxiv.org/html/2409.00135v1#bib.bib35)), LlaMa3 (AI@Meta, [2024](https://arxiv.org/html/2409.00135v1#bib.bib2))) and commercial LLMs (GPT3.5, GPT4 (OpenAI, [2024](https://arxiv.org/html/2409.00135v1#bib.bib26))). The results show that HoneyComb consistently improves the performance of all LLMs for SciQA and MaScQA.

![Refer to caption](img/8f83d1785b1161f33795a6cd63149d19.png)

Figure 3: Improvements of various LLMs integrated with HoneyComb compared to relevant baseline LLMs for different materials science tasks. With few exceptions, HoneyComb improves the performance of all LLMs across all tasks showing the utility of tool augmentation.

We evaluated the performance of various models on MaScQA and SciQA, including HoneyBee, GPT-3.5, GPT-4, Llama2, and Llama3, and demonstrated the effects of using the HoneyComb. The results are illustrated in Table [3](https://arxiv.org/html/2409.00135v1#S4.T3 "Table 3 ‣ 4.1 HoneyComb Evaluation ‣ 4 Experiments ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science")

The experimental results show that all models based on HoneyComb achieved significant improvements in accuracy on both MaScQA and SciQA. Specifically, on the MaScQA dataset, models such as HoneyBee and GPT-4 experienced substantial improvements, with HoneyBee’s accuracy improving by 16.76% and GPT-4’s by 20.61%. Other models also showed notable enhancements, with improvements ranging from 4.92 to 14.16%. On the SciQA dataset, the HoneyBee model saw a dramatic increase in performance, representing a huge improvement of 45.73% . HoneyComb based on GPT-3.5 and Llama3 showed more modest enhancements of around 0.14% to 0.32% , whereas HoneyComb based on GPT-4 and Llama2 experienced considerable improvements of approximately 5.70% and 2.87% , respectively.

### 4.2 HoneyComb Evaluation on MaScQA

We assess the performance improvements when integrating the HoneyComb framework with various large language models across predefined topics within the MaScQA dataset, as shown in Figure [3](https://arxiv.org/html/2409.00135v1#S4.F3 "Figure 3 ‣ 4.1 HoneyComb Evaluation ‣ 4 Experiments ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science"). The overall trend indicates that HoneyComb substantially enhances model performance. LLaMA-3 and HoneyBee exhibit impressive gains, particularly in ’Material Testing’ where improvements of 33.34 percentage points are observed, showcasing HoneyComb’s capability to effectively augment models with its advanced Tool-Hub and extensive MatSciKB.

However, GPT-3.5 displays a unique trend with declines across multiple topics including Atomic Structure, Fluid, Magnetism, Material Processing, and Material Testing. Despite having a higher baseline accuracy than LLaMA-3, LLaMA-2, and HoneyBee, GPT-3.5’s performance dips more frequently when integrated with HoneyComb. This could be attributed to its training data’s scope and depth, which, while extensive, may not align as effectively with HoneyComb’s highly specialized material science enhancements. The sophisticated computational demands and the dynamic nature of materials science queries may expose limitations in GPT-3.5’s ability to adapt its pre-existing knowledge to the specific enhancements HoneyComb offers. This nuanced understanding highlights the importance of model and tool compatibility in achieving effective enhancements across diverse materials science domains, thereby informing further development and optimization of HoneyComb to ensure comprehensive and reliable support in all areas of materials science research.

### 4.3 Ablation Study

To study how each component of HoneyComb contributes to the overall performance, we conducted ablation studies in this section. We tested the performance of HoneyComb when retrieved only from MatSciKB or only from Tool Hub, respectively. We also report results without retriever, in such situation there is no way for MatSciKB and ToolHub results to be fed into the model. Experimental results are reported in Table [4](https://arxiv.org/html/2409.00135v1#S4.T4 "Table 4 ‣ 4.3 Ablation Study ‣ 4 Experiments ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science").

Table 4: Ablation Study Results for MaScQA and SciQA based on GPT-4

Benchmark MatSciKB ToolHub Retriever Accuracy MaScQA 61.38 ✓ ✓ 73.23 ✓ ✓ 78.31 ✓ ✓ ✓ 79.07 SciQA 90.84 ✓ ✓ 96.34 ✓ ✓ 85.57 ✓ ✓ ✓ 96.56

The experimental results show that the best performance is achieved when both MatSciKB and ToolHub are used as reliable material knowledge references. HoneyComb improved the correctness on MaScQA and SciQA by 0.76% and 10.99% when compared to retrieving only from MatSciKB, and improved the correctness on MaScQA and SciQA by 5.84% and 0.22% when compared to retrieving only from Tool Hub. Therefore, we recommend that users retrieve HoneyComb from both sources together when deploying or using it.

## 5 Conclusion

In this work, we introduced HoneyComb, a pioneering LLM-based agent system tailored for materials science. HoneyComb integrates a meticulously curated materials science knowledge base (MatSciKB) and a dual-layered ToolHub of general and specialized computational tools. It combines three critical components: MatSciKB, an inductively constructed ToolHub, and a precision-focused Retriever module. This ensures HoneyComb provides accurate, up-to-date information and performs complex computational tasks reliably.

Experimental results show that HoneyComb outperforms contemporary general-purpose models (e.g. GPT and LLaMa series) and specialized models (e.g. HoneyBee) in materials science QA tasks. HoneyComb effectively bridges the gap between advanced large language models and the specific needs of materials science research, exemplifying how specialized agent systems can advance scientific research and serve as a blueprint for future advancements in other knowledge-intensive fields.

## Limitations

While HoneyComb significantly enhances the performance of current state-of-the-art models in various materials science QA tasks, there are limitations to its generali zability and applicability beyond the specific datasets and tasks it was trained on. Materials science is a diverse and intricate field, and it remains unclear how well HoneyComb would perform on tasks outside the MaScQA and SciQA benchmarks, particularly for more complex and novel challenges in materials science. Such challenges may include designing synthesis recipes for new materials or predicting material properties.

Additionally, HoneyComb’s reliance on high-quality LLMs for the knowledge base, tool construction, and retrieval processes can be a limitation. The performance of these components is contingent on the availability and capability of the underlying LLMs, which themselves may have inherent limitations. Furthermore, our work has primarily focused on the materials science domain, and further studies are required to evaluate how applicable and effective HoneyComb would be in other scientific fields.

## Broader Impacts

By expanding the HoneyComb agent system, HoneyComb has the potential to accelerate scientific discovery and innovation, contributing to a deeper understanding of complex materials systems. This could not only lead to advancements in materials design, development, and application but also promote the discovery and optimization of new materials, benefiting a wide range of industries. Additionally, the versatility and adaptability of HoneyComb enable it to tackle challenges across various scientific domains, further broadening its scope and impact.

Our research does not raise major ethical concerns.

## References

*   AI4Science and Quantum (2023) Microsoft Research AI4Science and Microsoft Azure Quantum. 2023. [The Impact of Large Language Models on Scientific Discovery: a Preliminary Study using GPT-4](https://arxiv.org/abs/2311.07361). *arXiv preprint arXiv:2311.07361*.
*   AI@Meta (2024) AI@Meta. 2024. [Llama 3 model card](https://github.com/meta-llama/llama3/blob/main/MODEL_CARD.md).
*   Anthropic (2024) Anthropic. 2024. [Calude3](https://www.anthropic.com/news/claude-3-family).
*   Boiko et al. (2023) Daniil A Boiko, Robert MacKnight, Ben Kline, and Gabe Gomes. 2023. Autonomous chemical research with large language models. *Nature*, 624(7992):570–578.
*   Bran et al. (2023) Andres M Bran, Sam Cox, Oliver Schilter, Carlo Baldassari, Andrew D White, and Philippe Schwaller. 2023. [Chemcrow: Augmenting large-language models with chemistry tools](https://arxiv.org/abs/2304.05376). *Preprint*, arXiv:2304.05376.
*   Buehler (2024a) Markus J. Buehler. 2024a. [Generative retrieval-augmented ontologic graph and multiagent strategies for interpretive large language model-based materials design](https://doi.org/10.1021/acsengineeringau.3c00058). *ACS Engineering Au*, 4(2):241–277.
*   Buehler (2024b) Markus J Buehler. 2024b. Mechgpt, a language-based strategy for mechanics and materials modeling that connects knowledge across scales, disciplines, and modalities. *Applied Mechanics Reviews*, 76(2):021001.
*   Cheng et al. (2024) Yuheng Cheng, Ceyao Zhang, Zhengwen Zhang, Xiangrui Meng, Sirui Hong, Wenhao Li, Zihao Wang, Zekai Wang, Feng Yin, Junhua Zhao, and Xiuqiang He. 2024. [Exploring large language model based intelligent agents: Definitions, methods, and prospects](https://arxiv.org/abs/2401.03428). *Preprint*, arXiv:2401.03428.
*   Chern et al. (2023) I-Chun Chern, Steffi Chern, Shiqi Chen, Weizhe Yuan, Kehua Feng, Chunting Zhou, Junxian He, Graham Neubig, Pengfei Liu, et al. 2023. Factool: Factuality detection in generative ai–a tool augmented framework for multi-task and multi-domain scenarios. *arXiv preprint arXiv:2307.13528*.
*   Devlin et al. (2018) Jacob Devlin, Ming-Wei Chang, Kenton Lee, and Kristina Toutanova. 2018. Bert: Pre-training of deep bidirectional transformers for language understanding. *arXiv preprint arXiv:1810.04805*.
*   Giannaros et al. (2023) Anastasios Giannaros, Aristeidis Karras, Leonidas Theodorakopoulos, Christos Karras, Panagiotis Kranias, Nikolaos Schizas, Gerasimos Kalogeratos, and Dimitrios Tsolis. 2023. Autonomous vehicles: Sophisticated attacks, safety issues, challenges, open topics, blockchain, and future directions. *Journal of Cybersecurity and Privacy*, 3(3):493–543.
*   Grootendorst (2022) Maarten Grootendorst. 2022. [Bertopic: Neural topic modeling with a class-based tf-idf procedure](https://arxiv.org/abs/2203.05794). *Preprint*, arXiv:2203.05794.
*   Gupta et al. (2022) Tanishq Gupta, Mohd Zaki, NM Krishnan, et al. 2022. Matscibert: A materials domain language model for text mining and information extraction. *npj Computational Materials*, 8(1):1–11.
*   Indian Institute of Technology Kanpur (2023) Indian Institute of Technology Kanpur. 2023. [Gate 2023: Graduate aptitude test in engineering](https://gate.iitk.ac.in/GATE2023/index.html). Accessed: 2024-06-14.
*   Izacard et al. (2022) Gautier Izacard, Mathilde Caron, Lucas Hosseini, Sebastian Riedel, Piotr Bojanowski, Armand Joulin, and Edouard Grave. 2022. [Unsupervised dense information retrieval with contrastive learning](https://openreview.net/forum?id=jKN1pXi7b0). *Transactions on Machine Learning Research*.
*   Jablonka et al. (2023) Kevin Maik Jablonka, Qianxiang Ai, Alexander Al-Feghali, Shruti Badhwar, Joshua D Bocarsly, Andres M Bran, Stefan Bringuier, L Catherine Brinson, Kamal Choudhary, Defne Circi, et al. 2023. 14 examples of how llms can transform materials science and chemistry: a reflection on a large language model hackathon. *Digital Discovery*, 2(5):1233–1250.
*   Johannes Welbl (2017) Matt Gardner Johannes Welbl, Nelson F. Liu. 2017. Crowdsourcing multiple choice science questions.
*   Kononova et al. (2021) Olga Kononova, Tanjin He, Haoyan Huo, Amalie Trewartha, Elsa A Olivetti, and Gerbrand Ceder. 2021. Opportunities and challenges of text mining in materials research. *Iscience*, 24(3).
*   LangChain contributors (2023) LangChain contributors. 2023. [Langchain: Open-source library for building language-based agents](https://github.com/langchain-ai/langchain). Online; accessed 17-June-2023.
*   Liu et al. (2024a) Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, and Percy Liang. 2024a. [Lost in the Middle: How Language Models Use Long Contexts](https://doi.org/10.1162/tacl_a_00638). *Transactions of the Association for Computational Linguistics*, 12:157–173.
*   Liu et al. (2024b) Yue Liu, Sin Kit Lo, Qinghua Lu, Liming Zhu, Dehai Zhao, Xiwei Xu, Stefan Harrer, and Jon Whittle. 2024b. [Agent design pattern catalogue: A collection of architectural patterns for foundation model based agents](https://arxiv.org/abs/2405.10467). *Preprint*, arXiv:2405.10467.
*   Madani et al. (2023) Ali Madani, Ben Krause, Eric R. Greene, Subu Subramanian, Benjamin P. Mohr, James M. Holton, Jose Luis Olmos, Caiming Xiong, Zachary Z. Sun, Richard Socher, James S. Fraser, and Nikhil Naik. 2023. [Large language models generate functional protein sequences across diverse families](https://doi.org/10.1038/s41587-022-01618-2). *Nat. Biotechnol.*, 41(8):1099–1106.
*   Miret and Krishnan (2024) Santiago Miret and NM Krishnan. 2024. Are llms ready for real-world materials discovery? *arXiv preprint arXiv:2402.05200*.
*   Miret et al. (2024) Santiago Miret, NM Anoop Krishnan, Benjamin Sanchez-Lengeling, Marta Skreta, Vineeth Venugopal, and Jennifer N Wei. 2024. Perspective on ai for accelerated materials design at the ai4mat-2023 workshop at neurips 2023. *Digital Discovery*.
*   Mirza et al. (2024) Adrian Mirza, Nawaf Alampara, Sreekanth Kunchapu, Benedict Emoekabu, Aswanth Krishnan, Mara Wilhelmi, Macjonathan Okereke, Juliane Eberhardt, Amir Mohammad Elahi, Maximilian Greiner, et al. 2024. Are large language models superhuman chemists? *arXiv preprint arXiv:2404.01475*.
*   OpenAI (2024) OpenAI. 2024. [Openai](https://openai.com/). Accessed: 2024-06-14.
*   Qin et al. (2023a) Yujia Qin, Shengding Hu, Yankai Lin, Weize Chen, Ning Ding, Ganqu Cui, Zheni Zeng, Yufei Huang, Chaojun Xiao, Chi Han, Yi Ren Fung, Yusheng Su, Huadong Wang, Cheng Qian, Runchu Tian, Kunlun Zhu, Shihao Liang, Xingyu Shen, Bokai Xu, Zhen Zhang, Yining Ye, Bowen Li, Ziwei Tang, Jing Yi, Yuzhang Zhu, Zhenning Dai, Lan Yan, Xin Cong, Yaxi Lu, Weilin Zhao, Yuxiang Huang, Junxi Yan, Xu Han, Xian Sun, Dahai Li, Jason Phang, Cheng Yang, Tongshuang Wu, Heng Ji, Zhiyuan Liu, and Maosong Sun. 2023a. [Tool learning with foundation models](https://arxiv.org/abs/2304.08354). *Preprint*, arXiv:2304.08354.
*   Qin et al. (2023b) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. 2023b. Toolllm: Facilitating large language models to master 16000+ real-world apis. *arXiv preprint arXiv:2307.16789*.
*   Song et al. (2023a) Yu Song, Santiago Miret, and Bang Liu. 2023a. [MatSci-NLP: Evaluating scientific language models on materials science language tasks using text-to-schema modeling](https://doi.org/10.18653/v1/2023.acl-long.201). In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 3621–3639, Toronto, Canada. Association for Computational Linguistics.
*   Song et al. (2023b) Yu Song, Santiago Miret, Huan Zhang, and Bang Liu. 2023b. Honeybee: Progressive instruction finetuning of large language models for materials science. In *Findings of the Association for Computational Linguistics: EMNLP 2023*, pages 5724–5739.
*   Sun et al. (2024) Liangtai Sun, Yang Han, Zihan Zhao, Da Ma, Zhennan Shen, Baocai Chen, Lu Chen, and Kai Yu. 2024. Scieval: A multi-level large language model evaluation benchmark for scientific research. In *Proceedings of the AAAI Conference on Artificial Intelligence*, volume 38, pages 19053–19061.
*   Swain and Cole (2016) Matthew C Swain and Jacqueline M Cole. 2016. Chemdataextractor: a toolkit for automated extraction of chemical information from the scientific literature. *Journal of chemical information and modeling*, 56(10):1894–1904.
*   Topsakal and Akinci (2023) Oguzhan Topsakal and Tahir Cetin Akinci. 2023. Creating large language model applications utilizing langchain: A primer on developing llm apps fast. In *International Conference on Applied Engineering and Natural Sciences*, volume 1, pages 1050–1056.
*   Touvron et al. (2023a) Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, Aurelien Rodriguez, Armand Joulin, Edouard Grave, and Guillaume Lample. 2023a. [Llama: Open and efficient foundation language models](https://arxiv.org/abs/2302.13971). *Preprint*, arXiv:2302.13971.
*   Touvron et al. (2023b) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurelien Rodriguez, Robert Stojnic, Sergey Edunov, and Thomas Scialom. 2023b. [Llama 2: Open foundation and fine-tuned chat models](https://arxiv.org/abs/2307.09288). *Preprint*, arXiv:2307.09288.
*   Trotman et al. (2014) Andrew Trotman, Antti Puurula, and Blake Burgess. 2014. Improvements to bm25 and language models examined. In *Proceedings of the 19th Australasian Document Computing Symposium*, pages 58–65.
*   Vaswani et al. (2023) Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, and Illia Polosukhin. 2023. [Attention is all you need](https://arxiv.org/abs/1706.03762). *Preprint*, arXiv:1706.03762.
*   Walker et al. (2021) Nicholas Walker, Amalie Trewartha, Haoyan Huo, Sanghoon Lee, Kevin Cruse, John Dagdelen, Alexander Dunn, Kristin Persson, Gerbrand Ceder, and Anubhav Jain. 2021. The impact of domain-specific pre-training on named entity recognition tasks in materials science. *Available at SSRN 3950755*.
*   Wang et al. (2024) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. 2024. A survey on large language model based autonomous agents. *Frontiers of Computer Science*, 18(6):186345.
*   Wang et al. (2023) Xiaoxuan Wang, Ziniu Hu, Pan Lu, Yanqiao Zhu, Jieyu Zhang, Satyen Subramaniam, Arjun R Loomba, Shichang Zhang, Yizhou Sun, and Wei Wang. 2023. Scibench: Evaluating college-level scientific problem-solving abilities of large language models. *arXiv preprint arXiv:2307.10635*.
*   Xie et al. (2023) Tong Xie, Yuwei Wan, Wei Huang, Zhenyu Yin, Yixuan Liu, Shaozhou Wang, Qingyuan Linghu, Chunyu Kit, Clara Grazian, Wenjie Zhang, et al. 2023. Darwin series: Domain specific large language models for natural science. *arXiv preprint arXiv:2308.13565*.
*   Xie et al. (2024) Tong Xie, Yuwei Wan, Yufei Zhou, Wei Huang, Yixuan Liu, Qingyuan Linghu, Shaozhou Wang, Chunyu Kit, Clara Grazian, Wenjie Zhang, and Bram Hoex. 2024. [Creation of a structured solar cell material dataset and performance prediction using large language models](https://doi.org/10.1016/j.patter.2024.100955). *Patterns*, 5(5).
*   Yoshikawa et al. (2023) Naruki Yoshikawa, Marta Skreta, Kourosh Darvish, Sebastian Arellano-Rubach, Zhi Ji, Lasse Bjørn Kristensen, Andrew Zou Li, Yuchi Zhao, Haoping Xu, Artur Kuramshin, et al. 2023. Large language models for chemistry robotics. *Autonomous Robots*, 47(8):1057–1086.
*   Zaki et al. (2023) Mohd Zaki, Jayadeva, Mausam, and N. M. Anoop Krishnan. 2023. [Mascqa: A question answering dataset for investigating materials science knowledge of large language models](https://arxiv.org/abs/2308.09115). *Preprint*, arXiv:2308.09115.
*   Zheng et al. (2024) Huaixiu Steven Zheng, Swaroop Mishra, Xinyun Chen, Heng-Tze Cheng, Ed H. Chi, Quoc V Le, and Denny Zhou. 2024. [Take a step back: Evoking reasoning via abstraction in large language models](https://arxiv.org/abs/2310.06117). *Preprint*, arXiv:2310.06117.

## Appendix

## Appendix A MatSciKB Knowledge Source

*   •

    ArXiv Paper

    *   –

        Included all papers indexed under the “material science” keyword on ArXiv.

    *   –

        Data entries structured into key-value pairs: key is the paper title, and value is the abstract.

    *   –

        Data Entries Count: 20,384

*   •

    Wikipedia Material Science Concepts

    *   –

        Scraped all 438 pages categorized under "Materials Science" on Wikipedia.

    *   –

        Each section within a page was separated as a distinct data entry.

    *   –

        Content formulated into key-value pairs, with keys as section titles and values as content.

    *   –

        Data Entries Count: 3,620

*   •

    Material Science Textbook

    *   –

        Sourced 6 publicly available textbooks.

    *   –

        Converted each textbook PDF file to text documents.

    *   –

        Broke each textbook into data entries by each section in a chapter.

    *   –

        Formulated data entries into key-value pairs, with keys as section titles and values as content.

    *   –

        Data Entries Count: 1,930

*   •

    Material Science Dataset

    *   –

        Utilized the multiple-choice dataset SciQA.

    *   –

        Extracted "support" column from the dataset that provides background knowledge for each question.

    *   –

        Each extracted "support" is treated as a data entry, with keys as the knowledge piece and values as empty strings, emphasizing their concise and standalone nature.

    *   –

        Data Entries Count: 10,473

*   •

    Material Science Formula

    *   –

        Formulas collected from Wikipedia’s dedicated pages for material science formulas.

    *   –

        Each formula is stored as key-value pair in the database, where the key represents the name of the formula and the value contains the formula equation itself.

    *   –

        Data Entries Count: 57

*   •

    GPT-generated Examples

    *   –

        Used a specific prompt to generate 50 material science questions at a time, output in CSV format along with a confidence score. Please refer to Appendix [B](https://arxiv.org/html/2409.00135v1#A2 "Appendix B Prompt for GPT-Generated Examples ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science") for the detailed prompt.

    *   –

        Human reviewers then selected questions with higher confidence scores for inclusion in the dataset.

    *   –

        Inspiration for question types was drawn from an external resource offering a wide range of material science questions and answers.

    *   –

        The key-value pairs were structured with questions as the keys and answers as the values.

    *   –

        Data Entries Count: 2,005

## Appendix B Prompt for GPT-Generated Examples

Please generate 50 instances of material science questions, specifically atomic structure and interatomic bonding, in a CSV format in the following order: question, answer, accuracy, confidence_score - accuracy: for factual questions, please evaluate the answer by comparing it with known facts. this field should be a number between 0 and 1. - confidence_score: how confident are you with the answer. this field should be a number between 0 and 1. - Here are sample instances without accuracy and confidence_score: “In terms of which of the following properties, metals are better than ceramics?”,“ductility” "In the wave-mechanical model of an atom, what do degenerate energy levels have?","equal energy" "Which of the following molecules is diamagnetic?","CO" - Examples of generated instances: - "What is the valence electron configuration of carbon?","2s²2p²",0.95,0.85 - "What type of crystal defect occurs when there is a line of irregularity in the lattice structure?","dislocation defect",0.96,0.91

## Appendix C Tree-Structure MatSciKB

MatSciKB is organized as a hierarchical tree with the parent node “Material Science” branching into 16 child nodes representing specific domains within materials science. Below is a simplified representation of this structure:

{
"Material Science": {
 "Children": {
  "Thermodynamics": {"Children": {"KB_1": {}, "KB_2": {}, "KB_3": {}}},
  "Atomic Structure": {"Children": {"KB_4": {}, "KB_5": {}, "KB_6": {}}},
  ...
  "Miscellaneous": {"Children": {"KB_7": {}, "KB_n": {}, "KB_n+1": {}}}
} } }

Each child node encompasses knowledge base (KB) data entries relevant to its category. In the construction of MatSciKB, we predefined 16 topics that align with core areas in materials science. They are ’Miscellaneous’, ’Material testing’, ’Fluid’, ’Material characterization’, ’Magnetism’, ’Transport phenomena’, ’Material processing’, ’Electrical’, ’Phase transition’, ’Material Applications’, ’Material manufacturing’, ’Mechanical’,’Atomic structure’, ’Thermodynamics’, "Formula", "Fundamental_Science_Knowledge"]

To categorize the data entries within these nodes, we utilized BertTopic, a state-of-the-art topic modeling tool based on transformers and c-TF-IDF, which automatically identifies and clusters documents with high granularity and contextual relevance Vaswani et al. ([2023](https://arxiv.org/html/2409.00135v1#bib.bib37)); Grootendorst ([2022](https://arxiv.org/html/2409.00135v1#bib.bib12)). The integration of BertTopic allowed for the dynamic clustering of MatSciKB entries into 16 predetermined categories.

The process involved the following steps:

1.  1.

    Initial Clustering: BertTopic was applied to cluster all data entries into more than the target number of categories, based on the textual content of each entry.

2.  2.

    Cluster Analysis and Selection: Human reviewers analyzed each cluster, identifying those whose common keywords and themes closely aligned with one of the predefined 16 topics.

3.  3.

    Category Assignment: Entries from clusters that aligned well with a predefined topic were assigned to that category, and then removed from the dataset.

4.  4.

    Iterative Refinement: The remaining entries underwent subsequent rounds of clustering and analysis. This process was repeated until no entries were left unclassified.

## Appendix D Tools Unified Interface Using LangChain

LangChain is an advanced framework designed to enhance applications that utilize LLM by offering standardized interfaces for various modules LangChain contributors ([2023](https://arxiv.org/html/2409.00135v1#bib.bib19)). This framework facilities the seamless integration and efficient management of LLM with external tools and systems. Utilizing LangChain, HoneyComb has developed a unified interface that standardizes the integration of a wide array of tools.

In HoneyComb, the unified interface provided by LangChain ensures that all tools, regardless of their specific function, are treated as standardized LangChain objects. This standardization is achieved by defining each tool with a consistent set of attributes:

1.  1.

    Function Signature: Each tool is defined with a clear function signature that specifies input and output types,

2.  2.

    Metadata Description: Each tool is accompanied by metadata that describes its purpose, suitable use cases, parameters description.

Examples of function signatures and metadata descriptions in HoneyComb are:

*   •

    Google Search

    *   –

        Function Signature: Google_Search(query: str, timeout: Optional[int] = 30) -> str

    *   –

        Metadata Description: General web search for up-to-date information across various topics.

*   •

    Wikipedia Search

    *   –

        Function Signature: Wikipedia_Search(topic: str, summarize: bool = True) -> str

    *   –

        Metadata Description: Retrieves and optionally summarizes detailed Wikipedia articles, particularly useful for quick reference checks.

*   •

    A Sample Mass Flow Rate Tool

    *   –

        Function Signature: calculate_initial_mass_flow_rate(args: str) -> float

    *   –

        Metadata Description: See figure [4](https://arxiv.org/html/2409.00135v1#A4.F4 "Figure 4 ‣ 3rd item ‣ Appendix D Tools Unified Interface Using LangChain ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science").

    ![Refer to caption](img/c2d4286ad8a71f004aa12d0441468b40.png)

    Figure 4: Metadata Description of a Sample Mass Flow Rate Tool

## Appendix E Examples of Inductive Tool Construction

See figure [5](https://arxiv.org/html/2409.00135v1#A5.F5 "Figure 5 ‣ Appendix E Examples of Inductive Tool Construction ‣ HoneyComb: A Flexible LLM-Based Agent System for Materials Science") for a detailed example illustrating how inductive tool construction work.

![Refer to caption](img/156389dc49cae506909a007ceae03f0c.png)

Figure 5: An example of inductive tool construction