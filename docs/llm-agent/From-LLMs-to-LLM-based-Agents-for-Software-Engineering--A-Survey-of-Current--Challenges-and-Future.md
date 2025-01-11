<!--yml
category: 未分类
date: 2025-01-11 12:20:45
-->

# From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future

> 来源：[https://arxiv.org/html/2408.02479/](https://arxiv.org/html/2408.02479/)

Haolin Jin, Linghan Huang, Haipeng Cai, Jun Yan, Bo Li, Huaming Chen Haolin Jin, Linghan Huang and Huaming Chen are with the School of Electrical and Computer Engineering, The University of Sydney, Sydney, 2006, Australia. (email: huaming.chen@sydney.edu.au)Haipeng Cai is with the School of Electrical Engineering and Computer Science at Washington State University, USJun Yan is with the School of Computing and Information Technology at University of Wollongong, AustraliaBo Li is with the Computer Science Department at the University of Chicago, US

###### Abstract

With the rise of large language models (LLMs), researchers are increasingly exploring their applications in various vertical domains, such as software engineering. LLMs have achieved remarkable success in areas including code generation and vulnerability detection. However, they also exhibit numerous limitations and shortcomings. LLM-based agents, a novel technology with the potential for Artificial General Intelligence (AGI), combine LLMs as the core for decision-making and action-taking, addressing some of the inherent limitations of LLMs such as lack of autonomy and self-improvement. Despite numerous studies and surveys exploring the possibility of using LLMs in software engineering, it lacks a clear distinction between LLMs and LLM-based agents. It is still in its early stage for a unified standard and benchmarking to qualify an LLM solution as an LLM-based agent in its domain. In this survey, we broadly investigate the current practice and solutions for LLMs and LLM-based agents for software engineering. In particular we summarise six key topics: requirement engineering, code generation, autonomous decision-making, software design, test generation, and software maintenance. We review and differentiate the work of LLMs and LLM-based agents from these six topics, examining their differences and similarities in tasks, benchmarks, and evaluation metrics. Finally, we discuss the models and benchmarks used, providing a comprehensive analysis of their applications and effectiveness in software engineering. We anticipate this work will shed some lights on pushing the boundaries of LLM-based agents in software engineering for future research.

###### Index Terms:

Large Language Models, LLM-based Agents, Software Engineering, Benchmark, Software Security, AI System Development

## I Introduction

Software engineering (SE) has seen its booming research and development with the aid of artificial intelligence techniques. Traditional approaches leveraging neural networks and machine learning have facilitated various SE topics such as bug detection, code synthesis, and requirements analysis [[1](https://arxiv.org/html/2408.02479v1#bib.bib1)][[2](https://arxiv.org/html/2408.02479v1#bib.bib2)]. However, they often present limitations, including the need for exclusive feature engineering, scalability issues, and the adaptability across diverse codebases. The rise of Large Language Models (LLMs) has embarked on new solutions and findings in this landscape. LLMs, such as GPT [[3](https://arxiv.org/html/2408.02479v1#bib.bib3)] and Codex [[4](https://arxiv.org/html/2408.02479v1#bib.bib4)], have demonstrated remarkable capabilities in handling downstream tasks in SE, including code generation, debugging, and documentation. These models leverage vast amounts of training data to generate human-like text, offering unprecedented levels of fluency and coherence. Studies have shown that LLMs can enhance productivity in software projects by providing intelligent code suggestions, automating repetitive tasks, even generating entire code snippets from natural language descriptions [[5](https://arxiv.org/html/2408.02479v1#bib.bib5)].

Despite their potential, there are significant challenges in applying LLMs to SE. One major issue is their limited context length [[6](https://arxiv.org/html/2408.02479v1#bib.bib6)], which restricts the model’s ability to comprehend and manage extensive codebases, making it challenging to maintain coherence over prolonged interactions. Hallucinations is another main concern, where the model generates code that appears plausible but is actually incorrect or nonsensical [[7](https://arxiv.org/html/2408.02479v1#bib.bib7)], potentially introducing bugs or vulnerabilities if not carefully reviewed by experienced developers. Additionally, the inability of LLMs to use external tools restricts their access to real-time data and prevent them from performing tasks outside their training scope. It diminishes their effectiveness in dynamic environments. These limitations significantly impact the application of LLMs in SE, and also highlight the need for expert developeers to critically refine and validate LLM-generated code for accuracy and security [[8](https://arxiv.org/html/2408.02479v1#bib.bib8)]. In complex projects, the static nature of LLMs can hinder their ability to adapt to changing requirements or efficiently incorporate new information. Moreover, LLMs typically cannot interact with external tools or databases, further limits their utility in dynamic and evolving SE contexts.

To address these challenges, LLM-based agents have emerged [[9](https://arxiv.org/html/2408.02479v1#bib.bib9)][[10](https://arxiv.org/html/2408.02479v1#bib.bib10)], combining the strengths of LLMs with external tools and resources to enable more dynamic and autonomous operations. These agents leverage recent advancements in AI, such as Retrieval-Augmented Generation (RAG) and tool utilization, to perform more complex and contextually aware tasks [[11](https://arxiv.org/html/2408.02479v1#bib.bib11)]. For instance, OpenAI’s Codex has been integrated into GitHub Copilot [[12](https://arxiv.org/html/2408.02479v1#bib.bib12)], enabling real-time code suggestions and completion within development environments. Unlike static LLMs, LLM-based agents can perform a wide range of tasks, such as autonomously debugging code by identifying and fixing errors, proactively refactoring code to enhance efficiency or readability, and generating adaptive test cases that evolve alongside the codebase. These features make LLM-based agents a powerful tool for SE, capable of handling more complex and dynamic workflows than traditional LLMs.

Historically, AI agents focused on autonomous actions based on predefined rules or learning from interactions  [[13](https://arxiv.org/html/2408.02479v1#bib.bib13)][[14](https://arxiv.org/html/2408.02479v1#bib.bib14)]. The integration of LLMs has presented new opportunities in this area, providing the language understanding and generative capabilities needed for more sophisticated agent behaviors. [[10](https://arxiv.org/html/2408.02479v1#bib.bib10)] shows that LLM-based agents are capable of autonomous reasoning and decision-making, achieving the third and fourth levels of WS (World Scope) [[15](https://arxiv.org/html/2408.02479v1#bib.bib15)], which outlines the progression from natural language processing (NLP) to general AI. In software engineering, LLM-based agents show promise in areas such as autonomous debugging, code refactoring, and adaptive test generation, demonstrating capabilities that approach artificial general intelligence(AGI).

![Refer to caption](img/32b1feadd2d4edcb37d14dd42fb9ddd8.png)

Figure 1: Paper Number for LLMs and LLM-based Agent between 2020-2024

In this work, we present, to the best of our knowledge, a first survey outlining the integration and transformation of LLMs to LLM-based agents in the domain of SE. Our survey covers six key themes in SE:

1.  1.

    Requirement Engineering and Documentation: Capturing, analyzing, and documenting software requirements, as well as generating user manuals and technical documentation.

2.  2.

    Code Generation and Software Development: Automating code generation, assisting in the development lifecycle, refactoring code, and providing intelligent code recommendations.

3.  3.

    Autonomous Learning and Decision Making: Highlighting the capabilities of LLM-based agents in autonomous learning, decision-making, and adaptive planning within SE contexts.

4.  4.

    Software Design and Evaluation: Contributing to design processes, architecture validation, performance evaluation, and code quality assessment.

5.  5.

    Software Test Generation: Generating, optimizing, and maintaining software tests, including unit tests, integration tests, and system tests.

6.  6.

    Software Security & Maintenance: Enhancing security protocols, facilitating maintenance tasks, and aiding in vulnerability detection and patching.

In detail, we aim to address following research questions:

*   •

    RQ1: What are the state-of-the-art techniques and practices in LLMs and LLM-based agents for SE? (Section. [IV](https://arxiv.org/html/2408.02479v1#S4 "IV Requirement Engineering and and Documentation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future")- [IX](https://arxiv.org/html/2408.02479v1#S9 "IX Software Security and Maintenance ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"))

*   •

    RQ1: What are the key differences in task performance between LLMs and LLM-based agents in SE applications? (Section. [IV](https://arxiv.org/html/2408.02479v1#S4 "IV Requirement Engineering and and Documentation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future")- [IX](https://arxiv.org/html/2408.02479v1#S9 "IX Software Security and Maintenance ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"))

*   •

    RQ2: Which benchmark datasets and evaluation metrics are most commonly used for assessing the performance of LLMs and LLM-based agents in SE tasks? (Section. [IV](https://arxiv.org/html/2408.02479v1#S4 "IV Requirement Engineering and and Documentation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future")- [IX](https://arxiv.org/html/2408.02479v1#S9 "IX Software Security and Maintenance ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") and Section. [X](https://arxiv.org/html/2408.02479v1#S10 "X Discussion ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"))

*   •

    RQ3: What are the predominant experimental models and methodologies employed when utilizing LLMs in SE? (Section. [X](https://arxiv.org/html/2408.02479v1#S10 "X Discussion ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"))

## II EXISTING WORKS AND THE SURVEY STRUCTURE

### II-A Existing works

In recent years, large language models have been primarily applied to help programmers generate code and fix bugs. These models understand and complete code or text based on the user’s input, leveraging their training data and reasoning capabilities. In previous survey papers, such as Angela Fan’s research [[8](https://arxiv.org/html/2408.02479v1#bib.bib8)], there has not been much elaboration on requirement engineering. As mentioned in the paper, software engineers are generally reluctant to rely on LLMs for higher-level design goals. However, with LLMs achieving remarkable improvements in contextual analysis and reasoning abilities through various methods like prompt engineering and Chain-of-Thought (COT) [[16](https://arxiv.org/html/2408.02479v1#bib.bib16)], their applications in requirement engineering are gradually increasing. Table [I](https://arxiv.org/html/2408.02479v1#S2.T1 "TABLE I ‣ II-A Existing works ‣ II EXISTING WORKS AND THE SURVEY STRUCTURE ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") summarizes and categorizes the tasks in requirement engineering. Many studies utilize models for requirement classification and generation. Since the collection primarily focuses on the latter half of 2023 and before April 2024, and some papers address multiple tasks, the table does not reflect the exact number of papers we have collected.

TABLE I: Distribution of SE tasks

| Category | LLMs | LLM-based agents | Total |
| --- | --- | --- | --- |
|  

&#124; Requirement &#124;
&#124; Engineering and &#124;
&#124; Documentation &#124;

 |  

&#124; Requirement Classification and Extraction (3) &#124;
&#124; Requirement Generation and Description (4) &#124;
&#124; Requirements Satisfaction Assessment (1) &#124;
&#124; Specification Generation (3) &#124;
&#124; Quality Evaluation (2) &#124;
&#124; Ambiguity Detection (2) &#124;

 |  

&#124; Generation of Semi-structured Documents (1) &#124;
&#124; Generate safety requirements (1) &#124;
&#124; Automatically generating use cases based on &#124;
&#124; high-level requirements (1) &#124;
&#124; Automated User Story Quality Enhancement (2) &#124;

 | 19 |
|  

&#124; Code Generation &#124;
&#124; and &#124;
&#124; software &#124;
&#124; development &#124;

 |  

&#124; Code Generation Debugging (3) &#124;
&#124; Code Evaluation (2) &#124;
&#124; Implement HTTP server (1) &#124;
&#124; Enhancing Code Generation Capabilities (3) &#124;
&#124; Specialized Code Generation (2) &#124;
&#124; Human Feedback Preference Simulation (1) &#124;

 |  

&#124; Automating the Software Development Process (5) &#124;
&#124; Large-Scale Code and Document Generation (1) &#124;
&#124; Tool and External API Usage (2) &#124;
&#124; Multi-Agent Collaboration and Code Refine (2) &#124;
&#124; Improving Code Generation Quality (2) &#124;

 | 23 |
|  

&#124; Autonomous &#124;
&#124; Learning &#124;
&#124; and Decision &#124;
&#124; Making &#124;

 |  

&#124; Multi-LLMs Decision-Making (1) &#124;
&#124; Creativity Evaluation (1) &#124;
&#124; Self-Identify and Correct Code (1) &#124;
&#124; Judge Chatbot Response (1) &#124;
&#124; Mimics Human Scientific Debugging (1) &#124;
&#124; Deliberate Problem Solving(1) &#124;

 |  

&#124; Collaborative Decision-Making and Multi-Agent &#124;
&#124; Systems (5) &#124;
&#124; Autonomous Reasoning and Decision-Making (7) &#124;
&#124; Learning and Adaptation Through Feedback (4) &#124;
&#124; Simulation and Evaluation of Human-like &#124;
&#124; Behaviors (2) &#124;

 | 24 |
|  

&#124; Software Design &#124;
&#124; and Evaluation &#124;

 |  

&#124; Creative Capabilities Evaluation (1) &#124;
&#124; Performance in SE Tasks (1) &#124;
&#124; Educational Utility and Assessment (1) &#124;
&#124; Efficiency Optimization (2) &#124;

 |  

&#124; Automation of Software Engineering Processes (3) &#124;
&#124; Enhancing Problem Solving and Reasoning (4) &#124;
&#124; Integration and Management of AI Models and &#124;
&#124; Tools (3) &#124;
&#124; Optimization and Efficiency Improvement (2) &#124;
&#124; Performance Assessment in Dynamic Environments &#124;
&#124; (2) &#124;

 | 19 |
|  

&#124; Software Test &#124;
&#124; Generation &#124;

 |  

&#124; Bug Reproduction and Debugging (2) &#124;
&#124; Security Test (2) &#124;
&#124; Test Coverage (2) &#124;
&#124; Universal Fuzzing (1) &#124;

 |  

&#124; Multi-agent Collaborative Test Generation (2) &#124;
&#124; Autonomous Testing and Conversational Interfaces &#124;
&#124; (3) &#124;

 | 11 |
|  

&#124; Software Security &#124;
&#124; & &#124;
&#124; Maintainance &#124;

 |  

&#124; Vulnerability Detection (6) &#124;
&#124; Vulnerability Repair (2) &#124;
&#124; Program Repair (4) &#124;
&#124; Robustness Testing (1) &#124;
&#124; Requirements Analysis (1) &#124;
&#124; Fuzzing (1) &#124;
&#124; Duplicate Entry (1) &#124;
&#124; Code Generation and Debugging (4) &#124;
&#124; Penetration Testing and Security Assessment (2) &#124;
&#124; Program Analysis and Debugging (1) &#124;

 |  

&#124; Autonomous Software Development and &#124;
&#124; Maintenance (4) &#124;
&#124; Debugging and Fault Localization (4) &#124;
&#124; Vulnerability Detection and Penetration &#124;
&#124; Testing (3) &#124;
&#124; Smart Contract Auditing and Repair (2) &#124;
&#124; Safety and Risk Analysis (2) &#124;
&#124; Adaptive and Communicative Agents (1) &#124;

 | 39 |

![Refer to caption](img/d7a2decb1afa938d38108725102e552a.png)

Figure 2: Paper Distribution

While other works have surveyed LLMs applications in some SE tasks [[17](https://arxiv.org/html/2408.02479v1#bib.bib17)] [[8](https://arxiv.org/html/2408.02479v1#bib.bib8)] [[18](https://arxiv.org/html/2408.02479v1#bib.bib18)], they lack a wider coverage of the general SE area to incorporate recent research developments. More importantly, a focus of LLMs is the main contributions of these works, but there is no distinguish the capabilities between LLMs and LLM-based agents. We summarize the difference between our work and others in Table [II](https://arxiv.org/html/2408.02479v1#S2.T2 "TABLE II ‣ II-A Existing works ‣ II EXISTING WORKS AND THE SURVEY STRUCTURE ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), this survey addresses these limitations by distinctly analyzing LLMs and LLM-based agents applications across six SE domains, providing a thorough and up-to-date review. From previous research, it is evident that the performance of LLMs in various applications and tasks heavily depends on the model’s inherent capabilities  [[10](https://arxiv.org/html/2408.02479v1#bib.bib10)]. More importantly, earlier surveys often present findings from papers spanning in a wide range of publication dates, leading to significant content disparities for LLMs in different SE tasks. For instance, research in requirement engineering was relatively nascent, resulting in sparse content in this area in previous surveys. The recent rise of LLM-based agents, with their enhanced capabilities and autonomy, fills these gaps. By focusing on the latest researches and clearly differentiating between LLMs and LLM-based agents, our survey provides a thorough and in-depth overview of how these technologies are applied and new opportunities they bring to SE.

In summary, we have collected a total of 117 papers directly relevant to this topic, covering the six SE domains mentioned earlier as shown in Figure.[1](https://arxiv.org/html/2408.02479v1#S1.F1 "Figure 1 ‣ I Introduction ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). Our analysis distinguishes between LLM and LLM-based agent contributions, offering a comparative overview and addressing the limitations of previous surveys. Considering the novelty nature of the LLM-based agents field and the lack of standardized benchmarks, this work seeks to offer a detailed review that can guide future research and provide a clearer view of the potential of these technologies in SE.

TABLE II: Comparison between Our Work and the Existing Work for LLM in SE

| Paper | Year | Domain | Benchmarks | Metrics |  

&#124; Agent &#124;
&#124; in SE &#124;

 |  

&#124; Agent &#124;
&#124; Distinction &#124;

 |
| --- | --- | --- | --- | --- | --- | --- |
|  [[19](https://arxiv.org/html/2408.02479v1#bib.bib19)] | 2023 | GenAI in SE | ✓ | ✓ | ✓ | ✗ |
|  [[8](https://arxiv.org/html/2408.02479v1#bib.bib8)] | 2023 | LLM in SE | ✓ | ✓ | ✓ | ✗ |
|  [[18](https://arxiv.org/html/2408.02479v1#bib.bib18)] | 2023 | Generation task by LLM in SE | ✓ | ✓ | ✗ | ✗ |
|  [[20](https://arxiv.org/html/2408.02479v1#bib.bib20)] | 2023 | LLM in syntax comprehension | ✓ | ✓ | ✗ | ✗ |
|  [[21](https://arxiv.org/html/2408.02479v1#bib.bib21)] | 2024 | LLM4Code in SE | ✓ | ✓ | ✗ | ✗ |
|  [[17](https://arxiv.org/html/2408.02479v1#bib.bib17)] | 2024 | LLM for process optimization in SE | ✓ | ✓ | ✗ | ✗ |
|  [[22](https://arxiv.org/html/2408.02479v1#bib.bib22)] | 2024 | Generation task by LLM in SE | ✓ | ✓ | ✗ | ✗ |
| Ours | 2024 | LLM & LLM-based Agent in SE | ✓ | ✓ | ✓ | ✓ |

### II-B Methodology

The paper collection process primarily involved searching DBLP and arXiv databases, focusing on recent studies from the latter half of 2023 to May 2024\. This approach ensured the inclusion of the latest research. We filtered out non-LLM-related papers and those with fewer than seven pages. To further refine our selection, we used keywords in Table [III](https://arxiv.org/html/2408.02479v1#S2.T3 "TABLE III ‣ II-C Overall Structure of the Work ‣ II EXISTING WORKS AND THE SURVEY STRUCTURE ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") to search SE-related works. We then manually screened the remaining papers to remove any with formatting errors or student projects. Additionally, we employed a snowballing search technique to capture significant works that might have been missed initially. Overall, we identified 117 relevant papers. Figure. [2](https://arxiv.org/html/2408.02479v1#S2.F2 "Figure 2 ‣ II-A Existing works ‣ II EXISTING WORKS AND THE SURVEY STRUCTURE ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") presents the distribution of these papers across the six SE domains and the proportion of LLM-based agents studies. However, some papers can be counted as multi-class fields so the literature review in the figure is more than 117 in total.

### II-C Overall Structure of the Work

The remainder of this paper is organized as follows Section 2 Introduces the architectures and background of LLMs and LLM-based agents, including an overview of RAG, tool utilization, and their implications for SE. Section 3-8 is the comparative analysis which Summarizes and compares the datasets, tasks, benchmarks, and metrics used in LLM and LLM-based agents studies across the six SE domains. Section 9 is the general discussion, section 10 is the final conclusion.

TABLE III: Keywords for Software Engineering Topics

| Topic | Keywords |
| --- | --- |
| Software Security & Maintenance | Software security, Vulnerability detection, Automated Program repair, Self-debugging, Vulnerability reproduction |
| Code Generation and Software Development | Code generation, Automatic code synthesis, Code refactoring, Programming language translation, Software development automation, Code completion, AI-assisted coding, Development lifecycle automation |
| Requirement Engineering and Documentation | Requirement engineering, Software requirements analysis, Automated requirement documentation, Technical documentation generation, User manual generation, Documentation maintenance, Requirements modeling, Requirements elicitation |
| Software Design and Evaluation | Software design automation, Architectural validation, Design optimization, Performance evaluation, Code quality assessment, Software metrics, Design pattern recognition, Architectural analysis, Code structure analysis |
| Software Test Generation | Test case generation, Automated testing, Unit test generation, Integration test generation, System test generation, Test suite optimization, Fault localization, Test maintenance, Regression testing, Adaptive testing |
| Autonomous Learning and Decision Making | Autonomous learning systems, Decision making, Adaptive planning, Project management automation, Self-improving software, Autonomous software agents |

## III Preliminaries

In this section, we introduce the foundational concepts of large language models, including the evolution of their frameworks and an overview of their architectures. Subsequent to this, we will discuss LLM-based agents, exploring both single-agent and multi-agent systems. We will also covers the background of these systems and their applications and distinctions in the field of software engineering.

### III-A Large Language Model

There is an inherent connection between large language models and natural language processing (NLP), with the historical development of natural language technologies tracing back to the 1950s. The earliest attempts to generate language dialogues through machines using specific rules can be traced to the period between 1950 and 1970\. The advent of machine learning technologies in the 1980s and the groundbreaking introduction of neural networks in the 1990s indicated a new era for NLP  [[23](https://arxiv.org/html/2408.02479v1#bib.bib23)]. These advancements facilitated significant progress in the NLP field, especially in the development of technologies for text translation and generation. The development of Long Short-Term Memory (LSTM) and Recurrent Neural Networks (RNN) during this period enabled more effective handling of the sequential nature of language data [[24](https://arxiv.org/html/2408.02479v1#bib.bib24)] [[25](https://arxiv.org/html/2408.02479v1#bib.bib25)]. These models addressed challenges associated with the lack of dependency in context, thereby enhancing the application of NLP in various domains.

In 2017 the new framework called ”Transformer” introduced by Google’s research team [[26](https://arxiv.org/html/2408.02479v1#bib.bib26)]. The transformer model based on the self-attention mechanism which significantly improved the effectiveness of language models. The inclusion of positional encoding not only solved the long-sequence dependency issue but also enabled parallel computation, which was a considerable improvement over previous models. In 2018, OpenAI developed the Generative Pre-trained Transformer (GPT) [[3](https://arxiv.org/html/2408.02479v1#bib.bib3)], a model based on the transformer architecture. The core idea behind GPT-1 was to utilize a large corpus of unlabelled text for pre-training to learn the patterns and structures of language, followed by fine-tuning for specific tasks. Over the next two years, OpenAI released GPT-2 and GPT-3 which increased the parameter count to 175 billions and also demonstrated strong capabilities in context understanding and text generation [[27](https://arxiv.org/html/2408.02479v1#bib.bib27)]. GPT-4 launched by OpenAI in 2023, represents a milestone following GPT-3.5\. Although GPT-4 maintains a similar parameter count of approximately 175 billion, its performance and diversity have seen considerable improvements. Through more refined training techniques and algorithm optimizations, GPT-4 enhanced the capability of language understanding and generation, particularly outperformed in handling complex texts and special contexts. Compared to other contemporary models like Google’s PaLM or Meta’s OPT, GPT-4 continues to stand out in multi-task learning and logical consistency in the text generation. While Google’s PaLM model boasts up to 54 billion parameters, GPT-4 shows superior generalization abilities across a broader range of NLP tasks [[28](https://arxiv.org/html/2408.02479v1#bib.bib28)]. On the open-source large models, Meta’s OPT model with a parameter size similar to the GPT-4 offers direct competition. Despite OPT’s advantages in openness and accessibility, GPT-4 still maintains a lead in specific application areas such as creative writing and complex problem solving [[29](https://arxiv.org/html/2408.02479v1#bib.bib29)].

### III-B Model Architecture

There are three common LLM architectures, the Encoder-Decoder architecture, exemplified by the traditional transformer model. This architecture comprises six encoders and six decoders, data input into the system will first passes through the encoder, where it undergoes sequential feature extraction via the model’s self-attention mechanism. Subsequently, the decoders utilize the word vectors produced by the encoders to generate outputs, this technique is common to see in machine translation tasks, where the encoder processes word vectors from one language through several attention layers and feed-forward networks, thereby creating representations of the context. The decoder then uses this information to incrementally construct the correct translated text. A recent example of this architecture is the CodeT5+ model, launched by Salesforce AI Research in 2023 [[30](https://arxiv.org/html/2408.02479v1#bib.bib30)]. This model is an enhancement of the original T5 architecture, which designed to improve performance in code understanding and generation tasks. It incorporates a flexible architecture and diversified pre-training objectives to optimize its effectiveness in these specialized areas. This development highlights the competency of Encoder-Decoder architectures in tackling increasingly complex NLP challenges.

The Encoder-only architecture, as the name suggests it eliminates the decoder from the entire structure making the data more compact. Unlike RNNs, this architecture is stateless and uses a masking mechanism that allows input processing without relying on hidden states, and also accelerating parallel processing speeds and providing excellent contextual awareness. BERT (Bidirectional Encoder Representations from Transformers) is a representative model of this architecture, this model is a large language model built solely on the encoder architecture. BERT leverages the encoder’s powerful feature extraction capabilities and pre-training techniques to learn bidirectional representations of text, achieving outstanding results in sentiment analysis and contextual analysis  [[31](https://arxiv.org/html/2408.02479v1#bib.bib31)].

The Decoder-only archiecture, in the transformer framework primarily involves the decoder receiving processed word vectors and generating output. Utilizing the decoder to directly generate text accelerates tasks such as text generation and sequence prediction. This characteristic with high scalability is known as auto-regressiveness, which is why popular models like GPT use this architecture. In 2020, the exceptional performance of GPT-3 and its remarkable few-shot learning capabilities demonstrated the vast potential of the decoder-only architecture [[32](https://arxiv.org/html/2408.02479v1#bib.bib32)]. Given the enormous computational cost and time required to train a model from scratch, and the exponential increase in the number of parameters, many researchers now prefer to leverage pre-trained models for further research. The most popular open-source pre-trained language model LLaMA, developed by Meta AI also employs the decoder-only architecture [[33](https://arxiv.org/html/2408.02479v1#bib.bib33)], as mentioned earlier, the autoregressiveness and simplicity of this structure make the model easier to train and fine-tune.

### III-C Large Language Model Based Agent

The concept of agents even trace back to the 19th century and is often referred to as intelligent agents, envisioned to possess intelligence comparable to humans. Over the past few decades, as AI technology has evolved, the capabilities of AI agents have significantly advanced, particularly with the reinforcement learning. This development has enabled AI agents to autonomously handle tasks and learn and improve based on specified reward/punishment rules. Notable milestones include AlphaGo [[34](https://arxiv.org/html/2408.02479v1#bib.bib34)], which leveraged reinforcement learning to defeat the world champion in Go competition.

The success of GPT has further propelled the field, with researchers exploring the use of large language models as the ”brain” of AI agents, thanks to GPT’s powerful text understanding and reasoning capabilities. In 2023, a research team from Fudan University [[10](https://arxiv.org/html/2408.02479v1#bib.bib10)] conducted a comprehensive survey on LLM-based agents, examining their perception, behavior, and cognition. Traditional LLMs typically generate responses based solely on given natural language descriptions, lacking the ability for independent thinking and judgment. LLM-based agents able to employ multiple rounds of interaction and customized prompts to gather more information, which enable the model to think and make decisions autonomously. In 2023, Andrew Zhao proposed the ExpeL framework [[35](https://arxiv.org/html/2408.02479v1#bib.bib35)], which utilizes ReAct as the planning framework combined with an experience pool [[36](https://arxiv.org/html/2408.02479v1#bib.bib36)]. This allows the LLM to extract insights from past records to aid in subsequent related queries, by letting the LLM analyze why previous answers were incorrect, it learns from experience to identify the problems.

At the same time, the application of LLM-based embodied agents has also become a hot research area in recent years. LLM-based Embodied Agents are intelligent systems that integrate LLMs with embodied agents [[37](https://arxiv.org/html/2408.02479v1#bib.bib37)]. These systems can not only process natural language but also complete tasks through perception and actions in physical or virtual environments. By combining language understanding with actual actions, these agents can perform tasks in more complex environments. This integration often involves using visual domain technologies to process and understand visual data and reinforcement learning algorithms to train agents to take optimal actions in the environment. These algorithms guide the agent through reward mechanisms to learn how to make optimal decisions in different situations, while the LLM acts as the brain to understand user instructions and generate appropriate feedback. In 2023, Guanzhi Wang introduced VOYAGER, an open-ended embodied agent with large language models [[38](https://arxiv.org/html/2408.02479v1#bib.bib38)]. It uses GPT-4 combined with input prompts, an iterative prompting mechanism, and a skill library enabling the LLM-based agents to autonomously learn and play the game Minecraft, becoming the first lifelong learning agent in the game.

![Refer to caption](img/d207174ea5cf3ee0bd0b2539937328ab.png)

Figure 3: Illustration of Common Data Augmentation Methods

Nowadays, various agent systems are emerging and they relies on large language models to make judgments, combined with techniques such as few-shot learning and multi-turn dialogue for model fine-tuning. However, due to the lack of datasets and the novelty of LLM-based agents, many researchers employ different methods for data augmentation. Common approaches include synonym replacement, where words in the text are replaced with synonyms from the same domain to increase textual diversity; back-translation, where the text is translated into another language and then back to the original language to generate new texts with slightly different grammatical structures and word choices; Paraphrasing refers to the new dialogue which similar in context but slightly different in expression created through manual or automated means; Synthetic Data Generation refers to use pretrained model to make a synthetic data generation as shown in Figure.[3](https://arxiv.org/html/2408.02479v1#S3.F3 "Figure 3 ‣ III-C Large Language Model Based Agent ‣ III Preliminaries ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). In 2023, Chenxi Whitehouse explored using LLMs for data augmentation to enhance the performance of multilingual commonsense reasoning datasets, especially under conditions of extremely limited training data [[39](https://arxiv.org/html/2408.02479v1#bib.bib39)]. The study employed various LLMs (such as Dolly-v2, StableVicuna, ChatGPT, and GPT-4) to generate new data. These models were prompted to create new examples similar to the original data, thereby increasing the diversity and quantity of training data. Prompt engineering was mentioned as an essential skill for effectively interacting with LLMs. By applying these prompt patterns, users can efficiently customize their dialogues with LLMs ensuring the generation of high-quality outputs and achieving complex automated tasks. In 2023, Jules White introduced a set of methods and patterns to enhance prompt engineering, optimizing interactions with LLMs such as ChatGPT [[40](https://arxiv.org/html/2408.02479v1#bib.bib40)]. This study categorizes prompt engineering into five main areas: Input Semantics, Output Customization, Error Identification, Prompt Improvement, and Interaction, to address a wide range of problems and adapt to different fields.

One notable technique is Retrieval-Augmented Generation (RAG), the input question undergoes similarity matching with documents in an index library, attempting to find relevant results. If similar documents are retrieved, they are organized in conjunction with the input question to generate a new prompt, which is then fed into the large language model. Currently, large language models possess long-text memory capabilities, numerous studies have tested Gemini v1.5 against the Needle In A Haystack (NIAH) evaluation to explore whether RAG has become obsolete [[41](https://arxiv.org/html/2408.02479v1#bib.bib41)]. However, from various perspectives such as cost and expense, RAG still holds significant advantages (RAG could be 99 percent cheaper than utilizing all tokens). Additionally, long texts can negatively impact response performance, causing LLM to respond more slowly when the input text is too long. Therefore, the advancements in context length for LLM will not fully replace the role of RAG but treated as complements between each other.

### III-D Single and Multi-Agents

Single agent systems leverage the capabilities of a LLM to perform various tasks, these agents typically use a single LLM to understand and respond to user queries, generate content, or execute automated tasks based on predefined instructions. Single agents are commonly used in scenarios where tasks accept a general answer and do not require complex decision-making. Examples include customer service chatbots, virtual assistants for scheduling, and automated content generation tools. However, single agents may struggle with dealing long context inputs, leading to inconsistent or irrelevant responses. The scalability of these systems is also limited when dealing with tasks that require extensive knowledge or context, this issue is often exacerbated by long texts as large language models cannot fully comprehend and analyze overly lengthy information in one turn. One of the primary issues with large language models is hallucination [[7](https://arxiv.org/html/2408.02479v1#bib.bib7)]. Hallucination refers to the generation of fabricated information or definitions by LLMs, presented in seemingly logical and reasonable language to the user. Most research papers on LLMs have stated this problem, while prompt engineering or tool interventions can mitigate the affect caused by hallucination, but it cannot be entirely eliminated. In 2023, Ziwei Ji conducted an in-depth study on hallucination in natural language generation [[42](https://arxiv.org/html/2408.02479v1#bib.bib42)]. This survey reviewed the progress and challenges in addressing hallucinations in NLG, providing a comprehensive analysis of hallucination phenomena across different tasks, including their definitions and classifications, causes, evaluation metrics, and mitigation methods.

Multi-agent systems involve the collaboration of multiple LLMs or agents to tackle complex tasks effectively. These systems fully utilize the advantages of multiple models, with each model specializing in specific aspects of the task to reduce overhead caused by multi-processes in single agents, the collaboration among agents allows for more sophisticated and robust problem-solving capabilities. Due to their exceptional capabilities, more researchers are beginning to explore the field of Multi-LLM based agents and start applying into software engineering domains. In 2024, a lot of researchers adopt the multi-agent system into the practical experiments [[43](https://arxiv.org/html/2408.02479v1#bib.bib43)] [[44](https://arxiv.org/html/2408.02479v1#bib.bib44)].

Multi-agent systems address the limitations of single-agent systems in the following ways:

*   •

    Enhanced Context Management: Multiple agents can maintain and share context, generating more coherent and relevant responses over long interactions.

*   •

    Specialization and Division of Labor: Different agents can focus on specific tasks or aspects of a problem, improving efficiency and effectiveness.

*   •

    Robustness and Error Correction: Collaborative agents can cross-check and validate each other’s outputs, reducing the likelihood of errors and improving overall reliability.

*   •

    Contextual Consistency: Multi-agent systems can better manage context over long dialogues. The collaboration of multiple agents improves the efficiency of incident mitigation.

*   •

    Scalability and Flexibility: These systems can integrate specialized agents to scale and handle more complex tasks. Through the division of labor among multiple agents, the quality of code generation is improved.

*   •

    Dynamic Problem Solving: By integrating agents with different expertise, multi-agent systems can adapt to a wider range of problems and provide more accurate solutions.

### III-E LLM in Software Engineering

Recently, there has been a shift towards applying general AI models to specific vertical domains such as medical and finance. In software engineering, new AI agents are emerging that are more flexible and intelligent compared to previous applications of LLMs, although they utilize different data and experiments. This continuous innovation underscores the transformative potential of AI agents across various fields, these models excel in text understanding and generation, promoting innovative applications in software development and maintenance.

LLMs profoundly impact software engineering by facilitating tasks such as code generation, defect prediction, and automated documentation. Integrating these models into development workflows not only simplifies the coding process but also reduces human errors. LLM-based agents enhance basic LLM capabilities by integrating decision-making and interactive problem-solving functions. These agents can understand and generate result by interacting with other software tools which optimize workflows, and make autonomous decisions to improve software development practices. In 2023, Yann Dubois introduced the AlpacaFarm framework [[45](https://arxiv.org/html/2408.02479v1#bib.bib45)], where LLMs are used to simulate the behavior of software agents in complex environments. Moreover, significant research has been conducted in the field of automated program repair (APR). In 2024, Islem Bouzenia introduced RepairAgent [[46](https://arxiv.org/html/2408.02479v1#bib.bib46)], another LLM-based tool designed for automatic software repair, this tool reduced the time developers spent on fixing issues. Additionally, in 2023, Emanuele Musumeci demonstrated a multi-agent LLM system [[47](https://arxiv.org/html/2408.02479v1#bib.bib47)], which involved a multi-agent architecture where each agent had a specific role in the generation of documents. This system significantly improved handling complex document structures without extensive human supervision. Besides these, LLMs have made outstanding contributions in software testing, software design, and emerging fields such as software security and maintenance.

Currently, there is no comprehensive and accurate definition of the capabilities an LLM must exhibit to be considered an llm-based agent. Since the application of LLMs in software engineering is relatively broad and some frameworks already behave certain levels of autonomy and intelligent, this study defines the distinction between LLM and LLM-based agents based on mainstream definitions and literature from the first half of 2024\. In this survey, an LLM architecture can be called an agent when it satisfy the Table [IV](https://arxiv.org/html/2408.02479v1#S3.T4 "TABLE IV ‣ III-E LLM in Software Engineering ‣ III Preliminaries ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future").

TABLE IV: Criteria for LLM-based agent

| Criteria |
| 1)  The LLM serves as the brain (the center of information processing and generation of thought). 2)  The framework not only relies on the language understanding and generation capabilities of LLMs but also possesses decision-making and planning abilities. 3)  If tools are available, the model can autonomously decide when and which tools to use and integrate the results into its predictions to enhance task completion efficiency and accuracy. 4)  The model can select the optimal solution from multiple homogeneous results (the ability to evaluate and choose among various possible solutions). 5)  The model can handle multiple interactions and maintain contextual understanding. 6)  The model has autonomous learning capabilities and adaptability. |

## IV Requirement Engineering and and Documentation

Requirement Engineering is a critical field within software engineering and plays an essential role in the software development process, its primary task is to ensure that the software system meets the needs of all relevant stakeholders. Typically, requirement engineering in project development involves many steps, where developers need to fully understand the users’ needs and expectations to ensure that the development direction of the software system aligns with actual requirements. The collected requirements are then organized and evaluated by the development group. Requirements Specification is the process of formally documenting the analyzed requirements, the specification must be accurate and concise, and the requirement verification must be conducted to ensure that developers are building what users need and that it aligns with the specifications. Requirement engineering also includes requirement management, a task that spans the entire software development life-cycle, developers need to continuously track, control, and respond to any changes occurring during development, ensuring that these changes do not negatively impact the project’s progress and overall quality.

### IV-A LLMs Tasks

In the field of requirement engineering, LLMs have demonstrated significant potential in automating and enhancing tasks such as requirement elicitation, classification, generation, specification generation, and quality assessment. Requirement classification and extraction is a crucial task in requirement engineering during the development process. It is common to encounter situations where clients present multiple requirements at once, necessitating manual classification by developers. By categorizing requirements into functional and non-functional requirements, developer can better understand and manage them, thanks to the strong performance of LLMs in classification tasks, many relevant frameworks have been developed. The PRCBERT framework, utilizing the BERT pre-trained language model, transforms classification problems into a series of binary classification tasks through flexible prompt templates, significantly improving classification performance [[48](https://arxiv.org/html/2408.02479v1#bib.bib48)]. Studies have shown that the PRCBERT achieved an F1 score of 96.13% on the PROMISE dataset which outperform the previous state-of-arts NoRBERT [[49](https://arxiv.org/html/2408.02479v1#bib.bib49)] and BERT-MLM models [[31](https://arxiv.org/html/2408.02479v1#bib.bib31)]. Additionally, the application of ChatGPT in requirement information retrieval has shown promising results, by classifying and extracting information from requirement documents, ChatGPT achieved comparable or even better $F\beta$ scores under zero-shot settings, particularly in feature extraction tasks, where its performance surpassed baseline models [[50](https://arxiv.org/html/2408.02479v1#bib.bib50)]. As seen in Table [I](https://arxiv.org/html/2408.02479v1#S2.T1 "TABLE I ‣ II-A Existing works ‣ II EXISTING WORKS AND THE SURVEY STRUCTURE ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), there is also substantial literature and research on using LLMs to automatically generate requirements and descriptions in requirement engineering.

By automating the generation and description of requirements, the efficiency and accuracy of requirement elicitation can be improved. Research indicates that LLMs hold significant potential in requirements generation task. For example, using ChatGPT to generate and gather user requirements, studies found that participants with professional knowledge could use ChatGPT more effectively, indicating the influence of domain expertise on the effectiveness of LLM-assisted requirement elicitation [[51](https://arxiv.org/html/2408.02479v1#bib.bib51)]. The study employed qualitative assessments of the LLMs’ output against predefined criteria for requirements matches, including full matches, partial matches, and the relevancy of the elicited requirements, although their success varied depending on the complexity of the task and the experience of the users, the result showing that LLMs could effectively assist in eliciting requirements, and its particularly useful in identifying, and suggesting requirements based on the large corpus of training data they provided. The SRS (Software Requirement Specification) generation is an important task which the developer normally spent a lot of time to refine and verified. In [[52](https://arxiv.org/html/2408.02479v1#bib.bib52)], researchers use both iterative prompting and a single comprehensive prompt to assess the performance of LLMs to generate SRS. The experiment conducted on GPT-4 and CodeLlama-34b one close-source LLM and one open-source LLM for comprehensive evaluation, the generated SRS will compare with human-crafted SRS and finally scored by the likert scale. The result indicate that, the human-generated SRS was overall superior, but CodeLlama often came close, sometimes outperforming in specific categories. The CodeLlama scored higher in completeness and internal consistency than GPT-4 but less concise, so this stuy demonstrated the potential of using fine-tuned LLMs to generate SRS and increase the overall project productivity. Another paper also explores using LLMs for generating specifications. In [[53](https://arxiv.org/html/2408.02479v1#bib.bib53)], the authors introduce a framework called SpecGen for generating program specifications. The framework primarily uses GPT-3.5-turbo as the base model and employs prompt engineering combined with multi-turn dialogues to generate the specifications. SpecGen applies four mutation operators to modify these specifications and finally uses a heuristic selection strategy to choose the optimal variant. The results show that SpecGen can generate 70% of the program specifications, outperforming traditional tools like Houdini [[54](https://arxiv.org/html/2408.02479v1#bib.bib54)] and Daikon¹¹1[https://github.com/codespecs/daikon](https://github.com/codespecs/daikon).

Furthermore, designing prompt patterns can significantly enhance LLMs’ capabilities in tasks such as requirement elicitation and system design. The paper provides a catalog of 13 prompt patterns, each aimed at addressing specific challenges in software development [[55](https://arxiv.org/html/2408.02479v1#bib.bib55)]. The experiments test the efficacy of these patterns in real world scenarios to validate their usefulness. By applying different prompt patterns, the study found that these patterns could help generate more structured and modular results and reduce common errors. Automated requirement completeness enhancement is another important benefit brought by the LLMs in requirement generation. The study [[56](https://arxiv.org/html/2408.02479v1#bib.bib56)] use BERT’s Masked Language Model (MLM) can detect and fill in missing parts in natural language requirements, significantly improving the completeness of requirements. BERT’s MLM achieved a precision of 82%, indicating that 82% of the predicted missing terms were correct.

There is also the application of LLMs in ambiguity detection tasks, aimed at detecting ambiguities in natural language requirement documents to improve clarity and reduce misunderstandings. This study primarily aims to address the issue of detecting term ambiguities within the same application domain (where the same term has different meanings in different domains). Although current models generally possess excellent contextual understanding capabilities, this was a common problem in machine learning at that time. This study provides an excellent paradigm for the subsequent application of LLMs in requirements engineering, study demonstrated the transformer-based machine learning models can effectively detect and identify ambiguities in requirement documents, thereby enhancing document clarity and consistency. The framework utilizes BERT and K-means clustering to identify terms used in different contexts within the same application domain or interdisciplinary project requirements documents [[57](https://arxiv.org/html/2408.02479v1#bib.bib57)]. In recent two years, more and more researchers use LLMs to help them to evaluate the requirement documentations, quality assessment tasks ensure that the generated requirements and code meet expected quality standards. The application of ChatGPT in user story quality evaluation has shown potential in identifying quality issues, but it requires further optimization and improvement [[58](https://arxiv.org/html/2408.02479v1#bib.bib58)]. A similar study use LLM to automatically process the requirement satisfaction assessment, and evaluate whether design elements fully covered by the given requirements, but the the researcher indicated the necessity of further verification and optimization in practical applications [[59](https://arxiv.org/html/2408.02479v1#bib.bib59)].

### IV-B LLM-based Agents Tasks

Currently the application of LLM-based agents in the requirement engineering is till quite nascent, but there are some useful researches to help us to see the potential possibility. LLM-based agents bring both efficiency and accuracy for tasks like requirement elicitation, classification, generation, and verification. Compared to traditional LLMs, these systems exhibit higher levels of automation and precision through task division and collaboration. The application of multi-agent systems in semi-structured document generation has shown significant effectiveness. In [[60](https://arxiv.org/html/2408.02479v1#bib.bib60)], a multi-agent framework is introduced that combines semantic recognition, information retrieval, and content generation tasks to streamline the creation and management of semi-structured documents in the public administration domain. The proposed framework involves three main types of agents: Semantics Identification Agent, Information Retrieval Agent, and Content Generation Agent. By avoiding the overhead of a single model, each agent is assigned a specific task with minimal user intervention, following the designed framework and workflow.

Additionally, the AI-assisted software development framework (AISD) also showcases the autonomy brought by the LLM-based agents in requirement engineering. [[61](https://arxiv.org/html/2408.02479v1#bib.bib61)] proposes the AISD framework, which continuously improves and optimizes generated use cases and code through ongoing user feedback and interaction. In the process of the experiment, humans need to first give a fuzzy requirement definition, and then LLM-based agent will improve the requirement case according to this information, and then design the model and generate the system according to the case, and then the generated results will let humans judge whether the requirements are met or not. The study results indicate that AISD significantly increased use case pass rates to 75.2%, compared to only 24.1% without human involvement. AISD demonstrates the agents’ autonomous learning ability by allowing LLMs to generate all code files in a single session, continually refining and modifying based on user feedback. This also ensures code dependency and consistency, further proving the importance of human involvement in the requirement analysis and system testing stages.

Furthermore, in generating safety requirements for autonomous driving, LLM-based agents have shown unique advantages by introducing multimodal capabilities. The system employs LLMs as automated agents to generate and refine safety requirements with minimal human intervention until the verification stage, which is unattainable with only LLMs. [[62](https://arxiv.org/html/2408.02479v1#bib.bib62)] describes an LLM prototype integrated into the existing Hazard Analysis and Risk Assessment (HARA) process, significantly enhancing efficiency by automatically generating specific safety-related requirements. The study through three design iterations progressively improved the LLM prototype’s efficiency by completing within a day compared to months manually. In agile software development, the quality of user stories directly impacts the development cycle and the realization of customer expectations. [[63](https://arxiv.org/html/2408.02479v1#bib.bib63)] demonstrates the successful application of the ALAS system in six agile teams at the Austrian Post Group IT. The ALAS system significantly improved the clarity, comprehensibility, and alignment with business objectives of user stories through automated analysis and enhancement. The entire agent framework allows the model to perform specific roles in the Agile development process, the study results indicated that the ALAS-improved user stories received high satisfaction ratings from team members.

### IV-C Analysis

The application of LLM-based agents in requirement engineering has demonstrated significant efficiency improvements and quality assurance. Through multi-agent collaboration and automated processing, these systems not only reduce manual intervention but also enhance the accuracy and consistency of requirement generation and verification. We can see that the tasks of LLM-based agents are no longer limited to simply generating requirements or filling in the gaps in descriptions. Instead, they involve the implementation of an automated process, with the generation of requirement documents being just one part of it, integrating LLM into agents enhances the overall system’s natural language processing and reasoning capabilities. In the real-world application, many tasks can no longer be accomplished by simple LLMs alone, especially for high-level software design. The emergence of LLM-based agents addresses this issue through a multi-agent collaborative system centered around LLMs, these agents continuously analyze and refine the deficiencies in the requirement documents, this is might be the main application trend of LLM-based agents in requirements engineering in the future.

![Refer to caption](img/735299caccec37c8a3adecbeec6157fb.png)

Figure 4: Illustration of Comparison Framework Between LLM-based Agent and LLM in User Story Refinement

The application of LLM-based agents in requirements engineering is still relatively limited, with most efforts focusing on leveraging the collaborative advantage of multi-agent systems to generate and refine requirements engineering documents. As illustrated in Figure.[4](https://arxiv.org/html/2408.02479v1#S4.F4 "Figure 4 ‣ IV-C Analysis ‣ IV Requirement Engineering and and Documentation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), which roughly simulates the architectures presented in [[58](https://arxiv.org/html/2408.02479v1#bib.bib58)] and [[63](https://arxiv.org/html/2408.02479v1#bib.bib63)], both applied to the generation and refinement of user stories, we can clearly compare the differences between the two architectures. On the left is the architecture of the LLM-based agent, while on the right is the approach of using prompt engineering and LLMs alone to refine user stories. The figure omits more detailed and complex aspects of the architecture to highlight the core differences between the two approaches. LLM-based agents can continuously improve from different professional perspectives by utilizing a shared database. Although there are not many papers on LLM-based agents, we can observe the trend and benefits of transitioning from LLMs to LLM-based agents.

### IV-D Benchmarks

Requirement engineering, unlike tasks such as bug fixing and code generation, does not have an abundance of public datasets available, such as HumanEval which commonly used for code generation assessment. Most training datasets for models in requirement engineering are self-collected by the authors and not all of them are open-sourced on Huggingface, resulting in a limited amount of dataset in requirement engineering. For instance, some papers do not mention a specific benchmark dataset but instead focus on practical examples and case studies to demonstrate the effectiveness of proposed prompt patterns [[55](https://arxiv.org/html/2408.02479v1#bib.bib55)]. The researcher Let actual developers and requirements engineers use the generated requirements documents and code to evaluate its accuracy, usability, and completeness. User feedback will be collected to further improve and optimize the prompt mode.

In [[50](https://arxiv.org/html/2408.02479v1#bib.bib50)], four datasets are primarily used, characterized by average length, type-token ratio (TTR), and lexical density (LD). The NFR Multi-class Classification dataset includes 249 non-functional requirements (NFRs) across 15 projects from the PROMISE NFR dataset. The App Review NFR Multi-label Classification dataset comprises 1800 app reviews from Google Play and Apple App Store, labeled with various NFRs. The Term Extraction dataset contains 100 smart home user stories with 250 manually extracted domain terms. Lastly, the Feature Extraction dataset consists of 50 app descriptions across 10 application categories with manually identified feature phrases. In [[56](https://arxiv.org/html/2408.02479v1#bib.bib56)], the PURE dataset consisting of 40 requirements specifications totaling over 23,000 sentences, is used to test BERT’s ability to complete requirements. In [[64](https://arxiv.org/html/2408.02479v1#bib.bib64)], the benchmark dataset comprised 36 responses to six questions: 6 responses generated by ChatGPT and 30 responses from five human RE experts (each expert provided 6 responses). These datasets serve as evaluation metrics for the models. Combining these papers, we can see that benchmark datasets for LLMs in requirement engineering mainly include various classifications of software requirements and functional and non-functional requirements to aid and assist models in learning this domain, the dataset utilization are quite flexible and diversifies.

In LLM-based agents’ research in requirement engineering, the selection and construction of datasets are also important. In [[47](https://arxiv.org/html/2408.02479v1#bib.bib47)], the dataset mainly consists of semantic templates from the public administration domain. These templates cover various semi-structured forms of administrative documents, such as official certificates and public service forms. Although the detailed composition of the dataset is not specified, it can be inferred that these templates include a large number of practical cases and contextual information to ensure that the documents generated by the multi-agent system meet actual needs.

Additionally, in [[61](https://arxiv.org/html/2408.02479v1#bib.bib61)], the CAASD (Capability Assessment of Automatic Software Development) dataset is introduced. This specially constructed benchmark dataset is used to evaluate the capabilities of AI-assisted software development systems. The CAASD dataset contains 72 tasks from various domains, such as small games and personal websites, each with reference use cases to define system requirements. The purpose of constructing this dataset is to provide a comprehensive evaluation benchmark that covers different types of development tasks, testing the performance of LLM-based agents in diverse tasks. In [[62](https://arxiv.org/html/2408.02479v1#bib.bib62)], the study mainly uses Design Science Methodology to design and evaluate the LLM prototype but does not mention a specific dataset, focusing on validating the model’s effectiveness through practical application and case studies. Despite the lack of detailed dataset descriptions, this approach emphasizes iterative improvement and practical application to ensure that the safety requirements generated by LLM-based agents meet high safety standards. Finally, in [[63](https://arxiv.org/html/2408.02479v1#bib.bib63)], 25 synthetic user stories are used, derived from a mobile delivery application project. The study evaluates the ALAS system’s effectiveness by testing it in six agile teams at the Austrian Post Group IT. Although these user stories are synthetic data designed for the experiment, they realistically reflect the requirements in actual projects, providing a valuable testing benchmark.

From these papers, it can be seen that the selection and construction of datasets in LLM-based agents’ research in requirement engineering often rely on practical projects and case studies, lacking standardization and large-scale datasets. Compared to LLM literature, the datasets used are broader and in a higher level like an actual system’s files, not limited to the classification of non-functional requirements and pure software requirement specifications. Researchers focus more on validating the model’s effectiveness through practical application and iterative improvement to enhance model performance. While this approach is flexible and targeted, it also highlights the field’s shortcomings in dataset standardization and scaling. In the future, with more public datasets being constructed and shared, the application of LLM-based agents in requirement engineering is expected to achieve broader and deeper development.

TABLE V: Evaluation Metrics in Requirement Engineering and Documentation

{tblr}

row1 = c, cell24 = c, cell34 = c, cell44 = c, cell54 = c, cell64 = c, cell74 = c, cell84 = c, cell94 = c, cell104 = c, cell114 = c, cell124 = c, cell134 = c, cell144 = c, cell154 = c, cell164 = c, cell174 = c, cell184 = c, cell194 = c, hlines, vlines, Reference Paper & Benchmarks Evaluation Metrics Agent
 [[51](https://arxiv.org/html/2408.02479v1#bib.bib51)] Evaluation on ActApp Precision and Recall for Elicitation
Clarity, Consistency, and Compliance
Completeness and accuracy of acceptance
criteria No
 [[50](https://arxiv.org/html/2408.02479v1#bib.bib50)] NFR, Smarthome user stories Precision, recall, and F\beta (F1 or F2) No
 [[56](https://arxiv.org/html/2408.02479v1#bib.bib56)] PURE Precision, F1 Score, Recall No
 [[52](https://arxiv.org/html/2408.02479v1#bib.bib52)] Not Specified Likert Scale No
 [[55](https://arxiv.org/html/2408.02479v1#bib.bib55)] Case studies Accuracy in identifying missing requirements.
Quality and Modularity of generated code.
Correctness of refactoring suggestions.
Efficiency in automating software engineering
tasks No
 [[64](https://arxiv.org/html/2408.02479v1#bib.bib64)] 36 responses to the six questions Abstraction, Atomicity, Consistency, Correctness
Unambiguity, Understandability, Feasibility No
 [[48](https://arxiv.org/html/2408.02479v1#bib.bib48)] PROMISE NFR-Review, NFR-SO F1 Score, Weighted F1 Score (w-F)) No
 [[53](https://arxiv.org/html/2408.02479v1#bib.bib53)] SV-COMP, SpecGenBench Number of Passes, Success Probability
Number of Verifier Calls, User Rating No
 [[65](https://arxiv.org/html/2408.02479v1#bib.bib65)] Jdoctor-data, DocTer-data, 
SpecGenBench, SV-COMP Accuracy, Precision, Recall, F1 Score No
 [[58](https://arxiv.org/html/2408.02479v1#bib.bib58)] Benchmark evaluations of user
stories using the AQUSA tool Agreement Rate, Precision, Recall, Specificity,
F1 Score No
 [[57](https://arxiv.org/html/2408.02479v1#bib.bib57)] Crawled Documents from Wikipedia Manual validation No
 [[59](https://arxiv.org/html/2408.02479v1#bib.bib59)] CM1, CCHIT, Dronology, PTC-A, 
PTC-B F\beta score, Mean Average Precision (MAP) No
 [[49](https://arxiv.org/html/2408.02479v1#bib.bib49)]] PROMISE NFR Precision (P), Recall (R), F1-score (F1),
Weighted average F1-score (A) No
 [[66](https://arxiv.org/html/2408.02479v1#bib.bib66)] CS-specific corpora, PURE Contextual Clarity, User Feedback No
 [[60](https://arxiv.org/html/2408.02479v1#bib.bib60)] Semantic Templates from Public
Administration Accuracy, Prompt Conformity, User Intervention
Frequency, Hallucination Rate Yes
 [[61](https://arxiv.org/html/2408.02479v1#bib.bib61)] CAASD Pass Rate, Token Consumption Yes
 [[62](https://arxiv.org/html/2408.02479v1#bib.bib62)] AEB, CAEM Performance Accuracy and Relevance, Efficiency,
Feedback from industry Yes
 [[63](https://arxiv.org/html/2408.02479v1#bib.bib63)] 25 Synthetic User Stories for a
Mobile Delivery Application Independence, Negotiability, Value, Estimability,
Smallness, Testability.
Survey among professionals  Yes

### IV-E Evaluation Metrics

In the field of requirement engineering, LLMs and LLM-based agents are evaluated using various metrics. These metrics not only include traditional indicators such as precision, recall, and F1 score but also more specific indicators tailored to the unique nature of requirement engineering. Through these evaluations, we can see how these models are assessed and how they are changing the practice of requirement engineering. The specific evaluation metrics are detailed in Table [V](https://arxiv.org/html/2408.02479v1#S4.T5 "TABLE V ‣ IV-D Benchmarks ‣ IV Requirement Engineering and and Documentation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). In [[51](https://arxiv.org/html/2408.02479v1#bib.bib51)], while precision and recall are fundamental for evaluating the effectiveness of information retrieval, additional evaluations of clarity, consistency, and compliance are included, which are crucial quality indicators in requirement engineering. This multidimensional evaluation method not only measures the operational performance of LLMs but also examines their ability to maintain the quality of requirement specifications. Through this approach, LLMs have demonstrated their value in automating and optimizing the requirement elicitation process, enhancing both efficiency and the reliability of results. The paper  [[52](https://arxiv.org/html/2408.02479v1#bib.bib52)] use the Likert Scale to measure the quality of generated specifications, the specification will be scored by its Unambiguous, Understandable, Conciseness, etc. The Likert Scale will be scored from 1 to 5 of agreements.

For agent-based LLMs, as demonstrated in [[63](https://arxiv.org/html/2408.02479v1#bib.bib63)], the evaluation extends to assessing the independence and negotiability of the agents, elevating their functionality to a new level. These agents provide technical solutions and also interact with users, autonomously adjusting to meet specific project needs, thus resembling collaborative partners. This capability makes LLM-based agents valuable in requirement engineering in the requirement management and decision optimization, also highlight that LLMs typically focus on improving the accuracy and efficiency of specific tasks, while LLM-based agents exhibit higher capabilities in autonomy and adaptability.

In Table [V](https://arxiv.org/html/2408.02479v1#S4.T5 "TABLE V ‣ IV-D Benchmarks ‣ IV Requirement Engineering and and Documentation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), we can see that the application of LLMs in requirements engineering typically requires common metrics such as F1 Score to evaluate model’s performance. However, for LLM-based agents, the evaluation focus shifts from the performance of requirement document generation to the quality of the final product. Therefore, evaluation metrics emphasize user satisfaction, such as pass rate, feedback, etc. Essentially, LLM-based agents still leverage LLMs themselves to achieve higher-level development, and depends pretty much on the nature of the task. In summary we can conclude that, the characteristics of agent models both reflect their complex decision-making and learning abilities also reveal their potential advantages in collaboration with human or other tools to provide higher scalability and flexibility design. This phenomenon implies the potential opportunities that the methods of requirement elicitation and processing in future software development will become more efficient, precise, and continuous refine to better align with stakeholder’s needs by using LLM-based agents.

## V Code Generation and Software Development

Code generation and software development are core areas within software engineering which plays a crucial role in the software development process. The primary objective of using LLMs in code generation is to enhance development efficiency and code quality through automation processes, thereby meeting the needs of both developers and users.

In recent years, the application of LLMs in code generation and software development has made significant progress, this has changed the way developers work and revealed a shift in automated development processes. Compared to requirement engineering, research on the application of LLMs and LLM-based agents in code generation and software development is more extensive and in-depth. Using natural language processing and generation technologies, LLMs can understand and generate complex code snippets, assisting developers in automating various stages from code writing and debugging to software optimization. The decoder-based large language models such as GPT-4 have shown significant potential in code generation by providing accurate code suggestions and automated debugging, greatly improving development efficiency. Recently, the application of LLM-based agents in software development is also gaining attention, these intelligent agents can not only perform complex code generation tasks but also engage in autonomous learning and continuous refinement, thereby offering flexible assist in dynamic development environments. Tools like GitHub Copilot [[12](https://arxiv.org/html/2408.02479v1#bib.bib12)], which integrate LLMs, have already demonstrated their advantages in enhancing programming efficiency and code quality.

### V-A LLMs Tasks

Large language models have optimized various tasks in code generation and software development through automation and reasoning, covering areas such as code generation, debugging, code comprehension, code completion, code transformation, and multi-turn interactive code generation. The primary method is generating executable code from natural language descriptions, where models utilize previously learnt code snippets or apply few-shot learning to better understand user requirements. Nowadays the AI tools integrates deeply with IDEs like Visual Studio Code²²2[https://code.visualstudio.com/](https://code.visualstudio.com/) and JetBrains³³3[https://www.jetbrains.com/](https://www.jetbrains.com/) to enhance code writing and translation tasks such as OpenAI’s Codex model [[67](https://arxiv.org/html/2408.02479v1#bib.bib67)]. Codex fine-tuned on public code from GitHub, demonstrate the capability to generate Python functions from doc-strings also outperformed other similar models on the HumanEval benchmark.

In [[68](https://arxiv.org/html/2408.02479v1#bib.bib68)], researchers comprehensively evaluated the performance of multiple LLMs on L2C(language to code) tasks. The results showed that GPT-4 demonstrates strong capability in tasks such as semantic parsing, mathematical reasoning, and Python programming. With instruction tuning and support from large-scale training data, the model can understand and generate code that aligns with user intent, achieving high-precision code generation. Applying LLMs to text-to-database management and query optimization is also a novel research direction in natural language to code generation task. By converting natural language queries into SQL statements, LLMs help developers quickly generate efficient database query code. In [[69](https://arxiv.org/html/2408.02479v1#bib.bib69)], proposed the SQL-PaLM framework which significantly enhances the execution accuracy and exact match rate for text-to-SQL tasks through a few-shot prompt and instruction fine-tuning, providing an effective solution for complex cross-domain SQL generation tasks. The improvements in accuracy and exact match achieved in the SQL-PaLM model are considered state-of-the-art (SOTA) in tested benchmarks, the SQL-PaLM performed promise results comparing with existing methods such as T5-3B + PICARD, RASAT + PICARD, and even GPT-4, achieving the highest test accuracy of 77.3% and an execution accuracy of 82.7%. Multilingual code generation is another important application of LLMs, particularly suited to the transformer architecture. In [[70](https://arxiv.org/html/2408.02479v1#bib.bib70)], researchers introduced the CodeGeeX model, which was pre-trained on multiple programming languages and performed well in multilingual code generation and translation tasks. Experimental results showed that CodeGeeX outperformed other multilingual models on the HumanEval-X benchmark.

Although current LLMs possess excellent code generation capabilities, with accuracy and compile rates reaching usable levels, the quality of generated code often depends on the user’s prompts. If the prompts are too vague or general, the LLM typically struggles to understand the user’s true requirements, making it difficult to generate the desired code in a single attempt. In [[71](https://arxiv.org/html/2408.02479v1#bib.bib71)], researchers introduced ”print debugging” technique, using GPT-4 to track variable values and execution flows, which enhancing the efficiency and accuracy by using in-context learning techniques. This method is particularly suitable for medium-difficulty problems on Leetcode, compared to the rubber duck debugging method, print debugging improved performance by 1.5% on simple Leetcode problems and by 17.9% on medium-difficulty problems.

Additionally, the application of LLMs in improving programming efficiency has garnered widespread attention, the tools like GitHub Copilot which integrating OpenAI’s Codex model, provide real-time code completion and suggestions during coding. According to [[72](https://arxiv.org/html/2408.02479v1#bib.bib72)], researchers present a controlled experiment with the Github Copilot, the result demonstrated that with developers completing HTTP server tasks 55.8% faster when using Copilot. Another similar study also using LLM to be the programmer tools, in [[73](https://arxiv.org/html/2408.02479v1#bib.bib73)], researchers introduced the INCODER model which capable of both program synthesis and editing. By leveraging bidirectional context, the model performs well in both single-line and multi-line code filling tasks, providing developers with smarter code editing tools. This real-time code generation and completion functionality not only improves programming efficiency but also reduce the burden on developers, allowing them to focus on higher-level design which is a common problem in software development where substantial workforce and time are wasted on tedious coding tasks.

The multi-turn program synthesis tasks represent a significant breakthrough for LLMs in handling complex programming tasks, in  [[74](https://arxiv.org/html/2408.02479v1#bib.bib74)], researchers introduced the CODEGEN model, which iteratively generates programs through multiple interactions, significantly improving program synthesis quality and making the development process more efficient and accurate. By gradually generating and continuously optimizing code at each interaction, LLMs can better understand user intent and generate more precise and optimized code. In the experiments, comparisons were made with the Codex model, which was considered state-of-the-art in code generation at the time. CODEGEN-MONO 2.7B outperformed the Codex model of equivalent outcome in pass@k metrics for both k=1 and k=10\. Furthermore, CODEGEN-MONO 16.1B exhibited performance that was comparable to or better than the best Codex model on certain metrics, further demonstrating its SOTA performance in the code generation. By iteratively generating and optimizing code, LLMs continuously improve their output quality. In [[75](https://arxiv.org/html/2408.02479v1#bib.bib75)], researchers proposed the Cycle framework, which enhances the self-improvement capability of code language models by learning from execution feedback, improving code generation performance by 63.5% on multiple benchmark datasets. Although Cycle has a certain degree of autonomy, its decision-making and planning capabilities are mainly limited to code generation and improvement tasks without overall planning, and the execution sequence is completely followed a fixed pattern, so it’s better to classified as an advanced LLM application.

### V-B LLM-based Agents Tasks

LLM-based agents have shown significant potential and advantages by substantially improving task efficiency and effectiveness through multi-agent collaboration. Unlike traditional LLMs, LLM-based agents adopt a division of labor approach, breaking down complex tasks into multiple subtasks handled by specialized agents, this method can enhance task efficiency and improves the quality and accuracy of generated code to mitigate the hallucination from the single LLM.

In [[76](https://arxiv.org/html/2408.02479v1#bib.bib76)], researchers proposed a self-collaboration framework where multiple ChatGPT (GPT-3.5-turbo) agents act as different roles to collaboratively handle complex code generation tasks. Specifically, the introduction of Software Development Methodology (SDM) divides the development process into three stages: analysis, coding, and testing. Each stage is managed by specific roles, and after completing their tasks, each role provides feedback and collaborates with others to improve the quality of the generated code. Experiment shows that this self-collaboration framework significantly improves performance on both the HumanEval and MBPP benchmarks, with the highest improvement reaching 29.9% in HummanEval compared to the SOTA model GPT-4\. This result demonstrating the potential of collaborative teams in complex code generation tasks. Although it lacks external tool integration and dynamic adjustment capabilities, this framework exhibits common characteristics of LLM-based agents, such as role distribution, self-improvement ability, and excellent autonomous decision-making, these combined capabilities qualify it to be considered an LLM-based agent. Similarly, In [[77](https://arxiv.org/html/2408.02479v1#bib.bib77)], the LCG framework improved code generation quality also through multi-agent collaboration and chain-of-thought techniques, once again demonstrating the effectiveness of multi-agent collaboration in the software development process.

The limitations of context windows was not discussed in previous studies, this has been thoroughly explored in a 2024 by University of Cambridge team. In [[78](https://arxiv.org/html/2408.02479v1#bib.bib78)], researchers introduced the L2MAC framework, which dynamically manages memory and execution context through a multi-agent system to generate large codebases, and achieved SOTA performance in generating large codebases for system design tasks. The framework is primarily divided into the following components: the processor, which is responsible for the actual generation of task outputs; the Instruction Registry, which stores program prompts to solve user tasks; and the File Storage, which contains both final and intermediate outputs. The Control Unit periodically checks the outputs to ensure that the generated content is both syntactically and functionally correct. The researchers conducted multiple experiments and compared with many novel methods like GPT-4, Reflexion, and AutoGPT, achieving a Pass@1 score of 90.2% on the HumanEval benchmark, showcasing its superior performance in generating large-scale codebases.

Recently, many studies have begun to use LLM-based agents to simulate real software development processes, the paper  [[79](https://arxiv.org/html/2408.02479v1#bib.bib79)] introduced the MetaGPT framework, which enhanced problem-solving capabilities through standard operating procedures (SOPs) encoded in multi-agent collaboration. The entire process of the multi-collaboration framework simulates the waterfall life-cycle of software development, with each agent playing different roles and collaborating to achieve the goal of automating software development. LLM-based agents have also shown strong ability in automated software development, [[80](https://arxiv.org/html/2408.02479v1#bib.bib80)] proposed a multi-GPT agent framework that automates tasks such as project planning, requirement engineering, software design, and debugging, illustrating the potential for automated software development. Similarly  [[81](https://arxiv.org/html/2408.02479v1#bib.bib81)] introduced the model called CodePori, which is a novel model designed to automate code generation for extensive and complex software projects based on natural language prompts. In [[82](https://arxiv.org/html/2408.02479v1#bib.bib82)] the AgentCoder framework collaborates with programmer agents, test design agents, and test execution agents to generate and optimize code, outperforming existing methods, achieved SOTA performance on the HumanEval-ET benchmark with pass@1 of 77.4% compared to the previous state-of-the-art result of 69.5%, this result showcasing the advantages of multi-agent systems in code generation and testing.

The purpose of integrating LLMs into agents from many framework is to enhance the self-feedback and reflection capabilities of the entire agent system. Because the current open-source LLMsgenerally have much lower capabilities in this aspect compared to proprietary models, the emergence of LLM-based agents can help bridge the gap between open-source models and the advanced capabilities of proprietary systems like GPT-4. [[83](https://arxiv.org/html/2408.02479v1#bib.bib83)] introduced the OpenCodeInterpreter framework, which improved the accuracy of code generation models by integrating code generation, execution, and human feedback. Based on CodeLlama and DeepSeekCoder, this framework performed close to the GPT-4 Code Interpreter on the HumanEval and MBPP benchmarks. The abbility of using external tools or APIs is another significant advantage of LLM-based agents, [[84](https://arxiv.org/html/2408.02479v1#bib.bib84)] proposed the Toolformer model, which significantly enhanced task performance by learning to call APIs through self-supervision. The framework Based on GPT-J (6.7B parameters) achieved significant performance improvements across multiple benchmark tasks, demonstrating the possibility of LLM-based agent brought by the external tool, the diverse choice of tools and architectures, allowing LLMs to continuously learn new things and improve themselves. Similarly, [[85](https://arxiv.org/html/2408.02479v1#bib.bib85)] enhanced LLMs’ interaction with external APIs through the ToolLLM framework, outperforming Text-Davinci-003 and Claude-2 on the ToolBench and APIBench benchmarks and excelling in multi-tool instruction processing.

### V-C Analysis

The main differences between LLM-based agents and traditional LLMs in software development applications mainly focus on the efficiency and autonomy, particularly in task division and collaboration. Traditional LLMs typically use a single model to handle specific tasks, such as generating code from text and code completion. However, this approach has limitations when dealing with complex tasks, especially regarding context window restrictions and the need for continuous feedback. LLM-based agents handle different subtasks through collaboration with clear division of labor, thereby enhancing task efficiency and quality. For example, in a code generation task, one agent generates the initial code, another designs test cases, and a third executes tests and provides feedback, thus achieving iterative optimization. Through task division, multi-agent systems, and tool integration, LLM-based agents can tackle more complex and broader tasks, improving the quality and efficiency of code generation. This approach overcomes the limitations of traditional LLMs also provides new directions and ideas for future software development research and applications, to frees programmers from the boring test suite generation.

![Refer to caption](img/ac9f7a42234523cf8d52df0dacd16271.png)

Figure 5: Illustration of Comparison Framework Between LLM-based Agent and LLM in Code Generation and Software Development

In software engineering task handling, there are subtle differences between LLMs and LLM-based agents in terms of task focus, approach, complexity and scalability, automation level, and task management. LLMs primarily focus on enhancing the code generation capabilities of a single LLM, including debugging, precision, evaluation. These methods typically improve specific aspects of code generation or evaluation through a single model, concentrating on performance enhancement within existing constraints, such as context windows and single-task execution. In contrast, LLM-based agents emphasize handling more complex and broader tasks through the collaboration of multiple specialized LLMs or frameworks, integrating tool usage, iterative testing, and multi-agent coordination to optimize the whole development process and easily surpass the state-of-art model in common benchmarks. The emeergence of multi-agent systems also brings more possibilities, this system can imitate the real software developer to perform the scrum development. Figure. [5](https://arxiv.org/html/2408.02479v1#S5.F5 "Figure 5 ‣ V-C Analysis ‣ V Code Generation and Software Development ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") utilize studies [[77](https://arxiv.org/html/2408.02479v1#bib.bib77)] and [[75](https://arxiv.org/html/2408.02479v1#bib.bib75)] showcase the differences between LLM-based agents and LLMs on the same code generation task. The LLM-based agents system are able to perform multi-agent collaboration and simulate the real scrum development team in the industry. In contrast the LLMs on the right are normally use multi-LLMs to analysis mistakes from the test cases, and refine the initial generated code, but they lack autonomy and efficiency, as the test cases are manually generated by humans.

### V-D Benchmarks

In the field of code generation and software development, there are notable differences and commonalities in the dataset used for research on LLMs and LLM-based agents. These datasets provide important benchmarks for evaluating model performance, the HumanEval dataset, widely used for assessing code generation models, is handcrafted by OpenAI and contains 164 programming problems, each including a function signature, problem description, function body, and unit tests. This dataset is primarily used to evaluate a model’s ability to generate correct code, particularly in tasks that involve converting natural language descriptions into executable code. Many studies have utilized HumanEval to test the performance of code generation models [[76](https://arxiv.org/html/2408.02479v1#bib.bib76)]. The MBPP (Mostly Basic Python Programming) dataset is another common benchmark, comprising 427 Python programming problems that cover basic concepts and standard library functions, this dataset is used to evaluate model performance across various programming scenarios. In [[82](https://arxiv.org/html/2408.02479v1#bib.bib82)], researchers used the MBPP dataset to test the performance of multi-agent systems in code generation and optimization, improving the accuracy and robustness of generated code through agent collaboration. The HumanEval-ET and MBPP-ET datasets are extensions of the original HumanEval and MBPP datasets, adding more test cases and more complex problems for a comprehensive evaluation of model performance [[86](https://arxiv.org/html/2408.02479v1#bib.bib86)]. The Spider and BIRD datasets focus on converting natural language to SQL queries, evaluating the model’s ability to handle complex query generation tasks. In [[69](https://arxiv.org/html/2408.02479v1#bib.bib69)], researchers used these datasets to test the SQL-PaLM framework, which evaluating the execution accuracy and exact match rate for SQL generation tasks through few-shot prompt and instruction fine-tuning. ToolBench and APIBench datasets are used to evaluate a model’s capability in using tools and APIs, ToolBench contains 16,464 real-world RESTful API instructions, and APIBench normally tests a model’s generalization ability to unseen API instructions [[85](https://arxiv.org/html/2408.02479v1#bib.bib85)]. The CAASD (Capability Assessment of Automatic Software Development) dataset is a newly developed benchmark comprising 72 software development tasks from various domains, each with a set of reference use cases to evaluate AI-assisted software development systems [[61](https://arxiv.org/html/2408.02479v1#bib.bib61)].

There are some obvious commonalities in dataset selection for LLMs and LLM-based agents, the HumanEval and MBPP datasets are widely used to assess code generation capabilities, covering a variety of programming tasks and languages. Moreover, many studies have adopted multilingual and cross-domain datasets, such as HumanEval-X and CodeSearchNet, to evaluate model performance across different languages and tasks. For the differences, LLM-based agents tend to use multi-agent collaboration frameworks to handle complex tasks, thus favoring benchmark datasets that emphasize multi-turn interactions and iterative optimization, also focus on tool usage and API integration capabilities, the framework TOOLLLM used ToolBench and APIBench to assess its tool usage capabilities, while Toolformer demonstrated its ability to autonomously learn to use tools. These differences primarily from the different approaches to task handling between LLMs and LLM-based agents, LLMs typically optimize a single model’s performance by fine-tuning on relevant datasets.

### V-E Evaluation Metrics

Various evaluation metrics are used to assess the performance of LLMs and LLM-based agents in code generation and software development. These metrics measures the models’ performance in specific tasks and how they improve the code generation and software development process. Table [VI](https://arxiv.org/html/2408.02479v1#S5.T6 "TABLE VI ‣ V-E Evaluation Metrics ‣ V Code Generation and Software Development ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") includes the distribution of evaluation metrics cited in this paper, encompassing both LLMs and LLM-based agents.

In research on LLMs and LLM-based agents, Pass@k is a common evaluation metric used to measure the proportion of generated code that passes all test cases within the first k attempts, this metric is widely applied across various datasets. In [[86](https://arxiv.org/html/2408.02479v1#bib.bib86)], Pass@k was used to evaluate the quality of code generation in multi-turn interactions, showing that the model’s Pass@k significantly improved by introducing a planning phase. Besides Pass@k, BLEU score is another common evaluation metric, mainly used to measure the syntactic similarity and correctness between generated code and reference code. In [[73](https://arxiv.org/html/2408.02479v1#bib.bib73)], BLEU score was used to evaluate the quality of generated code. Complete Time and Success Rate are other important evaluation metrics, particularly when assessing the productivity impact of AI-assisted development tools, these metrics are crucial as we expect LLMs to generate accurate code while maintaining expected speed. Confidence Calibration and Execution Rate are metrics used to evaluate the confidence level and execution success rate of the model when generating code. Researchers often use these metrics to assess various LLMs’ performance in understanding user intent and generating correct code with high precision.

Compared to the evaluation metrics for LLMs in software development, LLM-based agents also use Pass@k but more diverse to reflect their multi-agent collaboration characteristics. Win Rate and Agreement Rate are important metrics for evaluating the effectiveness of multi-agent collaboration. Additionally, LLM-based agents often use metrics like Execution Effectiveness and Cost Efficiency to evaluate their performance in real-world applications. For instance, in MetaGPT [[79](https://arxiv.org/html/2408.02479v1#bib.bib79)], researchers evaluated not only the correctness of code generation but also analyzed the execution effectiveness, development costs, and productivity. Results indicated that MetaGPT significantly improved development efficiency and reduced development costs while generating high-quality code. Overall both are using traditional metrics such as Pass@k, Win Rate, and task completion time to evaluate their code generation capabilities, these metrics directly reflect the accuracy and efficiency of the model in generating code. But LLM-based agents normally requiring more comprehensive and diverse metrics for evaluation to help assess the performance of multiple agents and the whole development process, that’s why we can see the human revision cost, qualitative feedback in the evaluation metrics. Researchers consider user or developer satisfaction metrics, as agent applications often involve extensive projects rather than isolated small-scale development, these metrics focus on the correctness of code generation and also resource utilization efficiency of the agent system.

TABLE VI: Evaluation Metrics in Code Generation and Software Development

{tblr}

column4 = c, cell11 = c, cell13 = c, cell12 = c, hlines, vlines, Reference Paper & Benchmarks Evaluation Metrics Agent
 [[71](https://arxiv.org/html/2408.02479v1#bib.bib71)] Leetcode problem Accuracy No
 [[72](https://arxiv.org/html/2408.02479v1#bib.bib72)] HTTP server in JavaScript by
95 programmer Task Completion Time, 
Task Success No
 [[68](https://arxiv.org/html/2408.02479v1#bib.bib68)] Spider, WikiTQ, GSM8k, 
SVAMP, MBPP, MBPP, DS-1000 Execution Accuracy,
Confidence Calibration
Execution Rate No
 [[45](https://arxiv.org/html/2408.02479v1#bib.bib45)] Alpaca Data Win-Rate, Agreement
Rate No
 [[86](https://arxiv.org/html/2408.02479v1#bib.bib86)] HumanEval/-X/-ET, 
MBPP-sanitized/-ET Pass@k, AvgPassRatio,
CodeBLEU No
 [[73](https://arxiv.org/html/2408.02479v1#bib.bib73)] HumanEval, CodeXGLUE Pass rate, Exact Match
BLEU Score No
 [[69](https://arxiv.org/html/2408.02479v1#bib.bib69)] Spider Accuracy, 
Exact Match No
 [[74](https://arxiv.org/html/2408.02479v1#bib.bib74)] HumanEval, MTPB Pass@k, Pass rate No
 [[30](https://arxiv.org/html/2408.02479v1#bib.bib30)] HumanEval, MathQA-Python,
GSM8K-Python, CodeSearchNet,
CosQA, AdvTest Pass@k, BLEU-4,
Exact Matcha
Edit Similarity,
Mean Reciprocal Rank (MRR) No
 [[70](https://arxiv.org/html/2408.02479v1#bib.bib70)] HumanEval/-X Pass@k No
 [[67](https://arxiv.org/html/2408.02479v1#bib.bib67)] HumanEval Pass@k, BLEU Score No
 [[75](https://arxiv.org/html/2408.02479v1#bib.bib75)] HumanEval, MBPP-S, APPS Pass Rate, Token Edit Distance,
 Exact Copy Rate No
 [[76](https://arxiv.org/html/2408.02479v1#bib.bib76)] MBPP/-ET, HumanEval/-ET Pass@k Yes
 [[78](https://arxiv.org/html/2408.02479v1#bib.bib78)] HumanEval Pass@1 Yes
 [[87](https://arxiv.org/html/2408.02479v1#bib.bib87)] CAASD Pass Rate, Token Consumption Yes
 [[84](https://arxiv.org/html/2408.02479v1#bib.bib84)] CCNet, SQuAD, Google-RE, T-REx,
ASDiv, SVAMP, MAWPS,
Web Questions, Natural Questions,
TriviaQA, MLQA, TEMPLAMA Zero-shot performance, 
Perplexity, Tool usage effectiveness Yes
 [[82](https://arxiv.org/html/2408.02479v1#bib.bib82)] MBPP/-ET, HumanEval/-ET Pass@1 Yes
 [[79](https://arxiv.org/html/2408.02479v1#bib.bib79)] HumanEval, HumanEval,
SoftwareDev Pass@k, Executability, Cost, 
Code Statistics, Productivity, 
Human Revision Cost Yes
 [[81](https://arxiv.org/html/2408.02479v1#bib.bib81)] HumanEval, MBPP Pass@k, Practitioner-Based
Assessment Yes
 [[85](https://arxiv.org/html/2408.02479v1#bib.bib85)] ToolBench, APIBench Pass Rate, Win Rate Yes
 [[80](https://arxiv.org/html/2408.02479v1#bib.bib80)] No Specificed Pass Rate, Win Rate Yes
 [[77](https://arxiv.org/html/2408.02479v1#bib.bib77)] MBPP/-ET, HumanEval/-ET Pass@1 Yes
 [[83](https://arxiv.org/html/2408.02479v1#bib.bib83)] HumanEval, MBPP, EvalPlus Pass@1 Yes
 [[88](https://arxiv.org/html/2408.02479v1#bib.bib88)] First-party data from Meta’s
code repositories and notebooks Acceptance Rate, P
ercentage of Code Typed, 
Qualitative Feedback Yes

## VI Autonomous Learning and Decision Making

Autonomous Learning and Decision Making is a critical and evolving field in modern software engineering, especially under the influence of artificial intelligence and big data. The core task of autonomous learning and decision making is to achieve automated data analysis, model building, and decision optimization through machine learning algorithms and intelligent systems, thereby enhancing the autonomy and intelligence of systems.

In this process, LLMs and LLM-based agents bring numerous possibilities, following the development of NLP technology, a lot of achievements have been made in the application of LLMs in this field. These models can handle complex language tasks and also demonstrate powerful reasoning and decision-making abilities, the research on voting inference using multiple LLMs calls has revealed new methods for optimizing performance, with the frequently used method called majority vote [[89](https://arxiv.org/html/2408.02479v1#bib.bib89)], this improves the accuracy of inference systems and ensures the selection of the optimal possibility. Additionally, the performance of LLMs in tasks such as automated debugging and self-correction has enhanced the system’s autonomous learning capabilities, achieving efficient error identification and correction. At the same time, the application of LLM-based agents in autonomous learning and decision-making is also a novel but popular topic, these agents can perform complex reasoning and decision-making tasks with the help from the LLM, and also improve their adaptability in dynamic environments through continuous learning and optimization. In this context, we have collected nineteen research papers on LLM-based agents in this field. This survey will provides a general review of these studies, analyzing the specific applications and technical implementations in autonomous learning and decision making.

### VI-A LLMs Tasks

The API call for the LLMs is one common applications, often requiring continuous calls to enable the model to make judgments and inferences, but does continuously increasing the number of calls always improve performance? In [[90](https://arxiv.org/html/2408.02479v1#bib.bib90)], researchers explored the impact of increasing LLM calls on the performance of composite reasoning systems. Paper analyze the voting inference design systems, the result showed that there is a non-linear relationship between the number of LLM calls and system performance; performance improves initially with more calls but declines after reaching a certain threshold. This research provides a theoretical basis for optimizing LLM calls, helping to allocate resources reasonably in practical applications to achieve optimal performance. However, the performance of Voting Inference Systems shows a non-monotonic trend due to the diversity of query difficulties, and the continuously increasing cost also needs to be considered.

Autonomous learning is also applied in bug fixing, where researchers hope LLMs can continuously learn to fix bugs and eventually identify human oversights or common errors. In [[91](https://arxiv.org/html/2408.02479v1#bib.bib91)], the SELF-DEBUGGING method was proposed, enabling LLMs to debug code by analyzing execution results and natural language explanations. This method significantly improved the accuracy and sample efficiency of code generation tasks especially for complex problems. Experimental results on the Spider and TransCoder benchmarks showed that the SELF-DEBUGGING method increase the model’s accuracy by 2-12% which demonstrates the potential of LLMs in autonomous learning to debug and correct any erros. Another similar study introduced the AutoSD (Automated Scientific Debugging) technique [[92](https://arxiv.org/html/2408.02479v1#bib.bib92)], which simulates the scientific debugging process through LLMs, generating explainable patched code. Researchers evaluated AutoSD’s capabilities from six aspects: feasibility, debugger ablation, language model change, developer benefit, developer acceptance, and qualitative analysis. Result have shown that AutoSD can generate effective patches and also improve developers’ accuracy in evaluating patched code by providing explanations, its explainability function makes it easier for developers to understand and accept automatically generated patches. Although the above two studies primarily focus on automated debugging techniques, the frameworks designed in these studies automatically determine the optimal repair solution based on the debugging results after collecting sufficient information, and provide specific code implementations, which demonstrated the capability of autonomous decision-making and learning.

Since the rise of LLMs applied to various fields, one research direction has been the rational analysis of their creativity and the exploration of their potential for continuous learning, this creativity also highly determined by the decision making capability of the models. [[93](https://arxiv.org/html/2408.02479v1#bib.bib93)] analyzed the outputs of LLMs from the perspective of creativity theory, exploring their ability to generate creative content, the study used metrics such as value, novelty, and surprise, finding that current LLMs have limitations in generating combinatorial, exploratory, and transformative creativity. Although LLMs can generate high-quality creative content, further research and improvement are needed to achieve true creative breakthroughs. Additionally, innovative responses generated by LLMs may come with the possibility of hallucination, a long-standing issue for large language models. Despite many techniques to mitigate its downsides, it still cannot be entirely prevented. There are many interesting experiments in decision making, such as having LLMs act as judges to determine whether a person has committed a crime [[94](https://arxiv.org/html/2408.02479v1#bib.bib94)]. A familiar attempt is to have a primary LLM interact with other LLMs. [[95](https://arxiv.org/html/2408.02479v1#bib.bib95)] explored the effectiveness of using LLMs as judges to evaluate other LLM-driven chat assistants. The study validated the consistency of LLM judgements with human preferences through the MT-Bench and Chatbot Arena benchmarks, with results showing that GPT-4’s judgments were highly consistent with human judgments across various tasks. This research demonstrates the potential of LLMs in simulating human evaluation, providing new ideas for automated evaluation and optimization.

### VI-B LLM-based Agents Tasks

Multi-agent collaboration and dialogue frameworks also demonstrated strong capabilities in both decision making and autonomous learning. [[96](https://arxiv.org/html/2408.02479v1#bib.bib96)] explores whether multi-agent discussions can enhance the reasoning abilities of LLMs. The proposed CMD framework simulates human group discussion processes, showing that multi-agent discussions can improve performance in commonsense knowledge and mathematical reasoning tasks without task-specific examples. Additionally, the study found that multi-agent discussions also correct common errors in single agents, such as judgment errors and the propagation of incorrect answers, thereby enhancing overall reasoning accuracy.  [[97](https://arxiv.org/html/2408.02479v1#bib.bib97)] researchers explored the potential of multi-modal large language models (MLLMs) like GPT4-Vision in enhancing agents’ autonomous decision-making processes. The paper introduce the PCA-EVAL benchmark, and evaluated multi-modal decision-making capabilities in areas such as autonomous driving, home assistants, and gaming. The results showed that GPT4-Vision exhibiting outstanding performance across the dimensions of perception, cognition, and action.

[[98](https://arxiv.org/html/2408.02479v1#bib.bib98)] proposes the Reflexion framework, a novel approach that strengthens learning through language feedback rather than traditional weight updates to avoid expensive re-train costs. The framework uses self-reflection and language feedback to help language agents learn from mistakes, significantly improving performance in decision-making, reasoning, and programming tasks. The Reflexion’s first-pass success rate on the HumanEval Python programming task increased from 80.1% to 91.0%, success rates in the ALFWorld decision-making task improved by 22%, and performance in the HotPotQA reasoning task increased by 14%. These results indicate that the Reflexion framework demonstrate the state-of-art performance in various tasks through self-reflection and language feedback.

Another agent framework [[35](https://arxiv.org/html/2408.02479v1#bib.bib35)] introduces the ExpeL agent framework which enhances decision-making capabilities by autonomously collecting experiences and extracting knowledge from a series of training tasks using natural language, this experience collection process is similar to how humans gain insights through practice and apply them in exams. By accessing internal databases, ExpeL also reduces hallucinations, employing the RAG technique discussed in [III](https://arxiv.org/html/2408.02479v1#S3 "III Preliminaries ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). The ExpeL framework doesn’t require parameter updates it enhances decision-making capabilities by recalling past successes and failures which fully leveraging the advantages of ReAct framework [[36](https://arxiv.org/html/2408.02479v1#bib.bib36)]. Experimental showed that ExpeL can continuous improvement across tasks in multiple domains and exhibited cross-task transfer learning capabilities. The combination of ExpeL and Reflexion even further enhance the performance in iterative task attempts, highlighting the importance of autonomous learning and experiential accumulation in developing intelligent agents. The ExpeL framework demonstrates its potential as a state-of-the-art (SOTA) LLM-based agents in several aspects, particularly in cross-task learning, self-improvement, and memory mechanisms. By comparing ExpeL with existing SOTA agents like Reflexion [[98](https://arxiv.org/html/2408.02479v1#bib.bib98)], ExpeL outperforms baseline methods in various task environments. These studies collectively indicate the importance of autonomous learning and improvement in LLM-based agents, agent systems continuously optimize and improve decision-making processes through self-feedback, self-reflection, and experiential accumulation which shows higher autonomy and flexibility in handling dynamic and complex tasks compared to traditional LLMs. Unlike traditional LLMs, which mainly rely on pre-training data and parameter updates, LLM-based agents adapt and improve their performance in real-time through continuous self-learning and feedback mechanisms, thus demonstrating outstanding performance in various tasks.

[[99](https://arxiv.org/html/2408.02479v1#bib.bib99)] proposes the AGENTVERSE multi-agent framework, designed to improve task completion efficiency and effectiveness through collaboration. The framework draws on human group dynamics by designing a collaborative system of expert agents that exhibit outstanding performance in tasks such as text understanding, reasoning, coding, and tool usage. Experiments showed that the AGENTVERSE framework performed well not only in independent task completion but also significantly improved performance through group collaboration especially in coding tasks where the framework use GPT-4 to be the brain of the agent groups. The framework also observed emergent behaviors in agents during collaboration, such as voluntary actions, conformity, and destructive behaviors, providing valuable insights for understanding and optimizing multi-agent systems.

Another multi-agents study  [[100](https://arxiv.org/html/2408.02479v1#bib.bib100)] Introducing the CAMEL framework, this is a well known agent framework, which explores building scalable techniques to facilitate autonomous collaborative agent frameworks. The study proposes a role-playing collaborative agent framework that guides dialogue agents to complete tasks through embedded prompts while maintaining alignment with human intentions. The CAMEL framework generates dialogue data to study behaviors and capabilities within the agent society, the study further enhanced agent performance by fine-tuning the LLaMA-7B model, validating the effectiveness of generated datasets in enhancing LLM capabilities.  [[101](https://arxiv.org/html/2408.02479v1#bib.bib101)] investigates the comprehensive comparison of LLM-augmented autonomous agents and proposes a new multi-agent coordination strategy for solving complex tasks through efficient communication and coordination called BOLAA. The experiment showed that the BOLAA outperforms other agent architectures in the WebShop environment especially in high-performance LLMs The above three studies focus on achieving a multi-agent collaboration architecture by increasing the number of agents. This trend indicates that more frameworks are beginning to explore the potential of multi-agent systems. [[44](https://arxiv.org/html/2408.02479v1#bib.bib44)] explores methods to enhance LLMs performance by increasing the number of agents. Using sampling and voting methods, the study showed that as the number of agents increased, LLM performance in arithmetic reasoning, general reasoning, and code generation tasks improved significantly. This method proves the effectiveness of multi-agent collaboration in enhancing model performance. These studies collectively indicate the importance of multi-agent collaboration and dialogue frameworks in autonomous learning and decision-making tasks. Compared to traditional LLMs, these multi-agent frameworks enhance reasoning accuracy under zero-shot learning and demonstrate higher autonomy and flexibility which reduce the burden on developers.

LLM-based agents not only perform complex data analysis tasks but also demonstrate potential in simulating and understanding human trust behaviors. [[102](https://arxiv.org/html/2408.02479v1#bib.bib102)] introduces a framework named SELF, designed to achieve self-evolution of LLMs through language feedback, use RLHF to train agent behavior to meet the human alignment. The framework enhances model capabilities through iterative processes of self-feedback and self-improvement without human intervention. In experiments, the test accuracy on GSM8K and SVAMP datasets increasing by 6.82% and 4.9%, respectively and the overall task win rates on the Vicuna test set and Evol-Instruct test set also increased by 10% and 6.9%. Another similar study exploring the potential of LLM-based agents to simulate the human trust behaviors.  [[103](https://arxiv.org/html/2408.02479v1#bib.bib103)] also examines whether LLM-based agents can simulate human trust behaviors. The study aims to determine if LLM-based agents exhibit trust behaviors similar to humans and explore whether these behaviors can align with human trust. Through a series of trust game variants such as initial fund allocation and return trust games, the research analyzes LLM-based agents’ trust decisions and behaviors in different contexts. Results show that particularly for GPT-4, LLM-based agents exhibit trust behaviors consistent with human expectations in these trust games, validating the potential of LLM-based agents in simulating human trust behaviors. The efficient and accurate handling of diverse datasets highlights the broad application prospects in fields such as software engineering. In terms of simulating trust behaviors, LLM-based agents demonstrate human-like behavior patterns through complex trust decisions and behavior analysis providing an important theoretical foundation for future human-machine collaboration and human behavior simulation.

Integrating LLMs into agents allows for more complex task processing. [[104](https://arxiv.org/html/2408.02479v1#bib.bib104)] proposes a lightweight user-friendly library named AgentLite whic designed to simplify the development, prototyping and evaluation of task-oriented LLM-based agents systems. The main goal of the study is to enhance the capabilities and flexibility of LLM-based agents in various applications by introducing a flexible framework. This framework enhances task-solving capabilities through task decomposition and multi-agent coordination, using a hierarchical multi-agent coordination approach where a managing agent supervises the task execution of each agent. [[105](https://arxiv.org/html/2408.02479v1#bib.bib105)] introduces a framework, GPTSwarm, that represents LLM-based agents as computational graphs to unify existing prompt engineering techniques and introduces methods for optimizing these graphs to enhance agent performance. The study verifies the effectiveness of the framework through various benchmarks such as MMLU, Mini Crosswords, and HumanEval. The framework demonstrated significant performance improvements on the GAIA benchmark with an improvement margin of up to 90.2% compared to the best existing methods. Additionally, agents have shown strong capabilities in autonomous learning and decision-making in software engineering and security, which will be introduced in the subsequent Software Security section [[106](https://arxiv.org/html/2408.02479v1#bib.bib106)] [[107](https://arxiv.org/html/2408.02479v1#bib.bib107)] [[108](https://arxiv.org/html/2408.02479v1#bib.bib108)] [[109](https://arxiv.org/html/2408.02479v1#bib.bib109)].

### VI-C Analysis

Overall, LLMs and LLM-based agents exhibit strong capability on the autonomous learning and decision making but slightly different view. These differences are reflected in the focus of task execution and also in autonomy, interactivity, learning and adaptation mechanisms, and the integration with other systems and modalities. From the perspective of task execution focus, LLMs primarily concentrate on enhancing specific functions in software engineering, such as debugging, problem-solving and automated reasoning. The tasks they perform are usually static and well-defined, such as automatic debugging, enhancing debugging capabilities to autonomously identify and correct errors, evaluating creativity and judging responses from other chatbots. In contrast, LLM-based agents not only focus on specific tasks but also manage multiple tasks simultaneously, often involving dynamic decision-making and interaction with other agents or systems. Examples of these agents’ tasks include enhancing reasoning through multi-agent discussions, continuous learning from experiences, requiring real-time dynamic decision-making, and also LLM-based agents can get in touch with the multimodal task in the visual environment.

We can conclude that, the application of LLM-based agents in the topic of autonomous learning and decision-making primarily involves exploring their performance in specific tasks through various framework designs. These studies evaluate the agents’ autonomy and decision-making capabilities to determine whether they align with human behavior and decision-making processes. If we dive into the specific task deigns, in terms of autonomy and interactivity LLMs are usually designed to perform highly specific tasks without needing to adapt to external input or environmental changes, they mainly operate as single models focusing on processing and responding within predefined boundaries, this also applied to all LLM applications. On the other hand, LLM-based agents exhibit higher autonomy which are typically designed to interact with or adapt to the environment in real-time, they are often part of multi-agent systems where collaboration and communication are key components, for example use extra model or tools to further help with the planning phases. In terms of integration with other systems and modalities, LLMs typically operate in text input-output scenarios and even in multi-modal settings, their role is usually limited to processing and generating text-based content. Also, LLM-based agents are more likely to integrate with other systems and modalities such as visual input or real-world perception data, enabling them to perform more complex and context-based decision-making tasks.

![Refer to caption](img/b9aaec30593af4910f3bd0e409b989c1.png)

Figure 6: Expel[[35](https://arxiv.org/html/2408.02479v1#bib.bib35)] Framework with Reflexion[[98](https://arxiv.org/html/2408.02479v1#bib.bib98)] in Experience Gathering

Regarding learning and adaptation mechanisms, LLMs’ adaptation and learning are usually confined to the model’s training data and parameter range, although they can adapt through new data updates, they lack the ability to continuously learn from real-time feedback, they are more focused on using existing knowledge to solve problems and generate responses. In contrast, LLM-based agents are often equipped with experiential learning and real-time feedback adaptation mechanisms, allowing them to optimize strategies and responses based on continuous interactions. One good example of LLm-based agents framework is Expel [[35](https://arxiv.org/html/2408.02479v1#bib.bib35)], which utilize the previous researches ReAct [[36](https://arxiv.org/html/2408.02479v1#bib.bib36)] and Reflexion [[98](https://arxiv.org/html/2408.02479v1#bib.bib98)] as shown in Figure. [6](https://arxiv.org/html/2408.02479v1#S6.F6 "Figure 6 ‣ VI-C Analysis ‣ VI Autonomous Learning and Decision Making ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). This framework utilizes a memory pool and insights pool to enable the LLM to learn from past knowledge, thereby aiding subsequent decision-making. This autonomous decision-making capability is something that traditional LLM frameworks cannot achieve.

### VI-D Benchmarks

In the field of autonomous learning and decision-making, the benchmark datasets used by LLMs and LLM-based agents are quite similarly in the task handling and application requirements. We can gain a deeper understanding of the strengths and weaknesses of both approaches in different tasks and their application contexts. The specific dataset references, please see Table [VII](https://arxiv.org/html/2408.02479v1#S6.T7 "TABLE VII ‣ VI-E Evaluation Metrics ‣ VI Autonomous Learning and Decision Making ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future").

In the research on LLMs, the main datasets include Defects4J, MMLU, TransCoder, and MBPP. These datasets are primarily used to evaluate model performance in specific domains and tasks. Defects4J is a widely used in the software engineering, this software defect dataset containing 525 real defects from 17 Java projects. It’s designed to test the effectiveness of automated program repair and defect detection tools by providing a standardized benchmark that allows researchers to compare the performance of different methods. MMLU (Massive Multitask Language Understanding) is a large-scale benchmark dataset covering 57 subjects, testing models on a broad spectrum of knowledge and reasoning abilities in multitask language understanding. It includes questions ranging from elementary education to professional level such as College Mathematics, Business Ethics, and College Chemistry, challenging the models’ diverse knowledge base and reasoning capabilities. The TransCoder dataset focuses on code translation across programming languages which evaluate the model’s ability to automatically translate code from one programming language to another. This is crucial for multilingual software development and maintenance, as it can greatly enhance development efficiency. MBPP (Mostly Basic Python Programming) has been introduced in previous section, it’s a dataset containing 427 Python programming problems, covering basic concepts and standard library functions, it’s widely used to test the model’s performance in different programming scenarios, evaluating its ability to generate correct and efficient code.

In contrast, LLM-based agents use datasets that emphasize multitasking and decision-making capabilities in complex scenarios. The main datasets include HotpotQA, ALFWorld, FEVER, WebShop, and MGSM. HotpotQA is a multi-hop question-answering dataset that requires models to reference content from multiple documents when answering questions, evaluating their information synthesis and reasoning abilities, this dataset challenges the model’s performance in complex reasoning tasks. ALFWorld is a text-based environment simulation dataset requiring multi-step decision-making where the model completes tasks in a virtual home environment. The dataset combines natural language processing and decision-making, evaluating the model’s performance in dynamic and interactive tasks. The FEVER (Fact Extraction and VERification) dataset is used for fact verification tasks, where the model needs to verify the truthfulness of given statements and provide evidence, it assesses the model’s capabilities in information retrieval and logical reasoning. WebShop is an online shopping environment simulation dataset containing 1.18 million real-world products and human instructions, it used to test the model’s performance in complex decision-making tasks such as completing shopping tasks and attribute matching. MGSM (Multimodal Generalized Sequence Modeling) is a multimodal dataset containing tasks related to dialogue, creative writing, mathematical reasoning, and logical reasoning, evaluating the model’s comprehensive abilities in multimodal tasks.

Comparatively, LLM datasets typically focus on single, static tasks such as code generation, mathematical reasoning and creative writing, which suitable for models working within predefined task scopes. Datasets like Defects4J, MMLU, and MBPP help evaluate model capabilities in specific domains. LLM-based agents are more suited for complex, multitasking, and dynamic environments where datasets require models to handle multimodal inputs and real-time decision-making, it can showcase their advantages in handling complex interactions and multitasking scenarios. Datasets like HotpotQA, ALFWorld, FEVER, and WebShop challenge the models’ performance in information synthesis, dynamic decision-making/interaction and multimodal tasks. This difference arises from the distinct design goals of the two: LLMs aim to optimize performance on single tasks, while LLM-based agents are designed to handle complex or multi-modal task, this require higher autonomy and adaptability. It’s also reflects modern applications’ demand for highly interactive, adaptive, and multifunctional AI systems, driving the development from single LLM models to multi-agent systems. Through these analyses, we can identify the different application of LLMs and LLM-based agents in autonomous learning and decision-making, it’s important to choose the appropriate framework to meet different task requirements in the real world applications.

### VI-E Evaluation Metrics

various evaluation metrics are used in the research on LLMs and LLM-based agents, these metrics used to evaluate the models’ performance in specific tasks and analyze their application effectiveness in this domain. Below, we discuss several representative studies analyzing the evaluation metrics they employed and exploring the differences between LLMs and LLM-based agents in this field.

In research on LLMs, evaluation metrics primarily focus on model accuracy and task completion. In [[90](https://arxiv.org/html/2408.02479v1#bib.bib90)], researchers used the accuracy of a voting inference system which measured by the expected 0/1 loss (the proportion of correct responses) to assess model performance. This metric evaluates the accuracy of models through multiple calls, reflecting the ability of LLMs to improve result accuracy via iterative reasoning. Common evaluation metrics in the literature include accuracy and sample efficiency, accuracy refers to the proportion of correct predictions made by the model, while sample efficiency measures the number of samples required to achieve a certain accuracy level. These metrics assess both the predictive and decision making ability of the model and its data utilization efficiency during training. In [[92](https://arxiv.org/html/2408.02479v1#bib.bib92)], evaluation metrics include possible patches, correct patches, precision, and developer accuracy. Possible patches refer to patches that pass all tests, while correct patches are semantically equivalent to the original developer patches. Precision measures the proportion of correct patches among the possible patches, and developer accuracy assesses the correctness of patches with and without explanations through human evaluation. These metrics emphasize the model’s explanatory capability and practical effectiveness in automated code repair, increasing reliance on human evaluation. To assess model creativity, value, novelty and surprise are used as creativity dimensions. Quality, social acceptability, and similarity of generated works, as well as the ability to generate creative product, are also included in the evaluation. [[110](https://arxiv.org/html/2408.02479v1#bib.bib110)] used the success rate in the Game of 24 and the coherence of generated paragraphs in creative writing as evaluation metrics. These metrics assess the model’s performance in problem-solving and text generation, showcasing LLMs’ potential in solving complex problems and generating coherent text. In [[95](https://arxiv.org/html/2408.02479v1#bib.bib95)], consistency and success rate were used as evaluation metrics, the consistency calculates the probability of agreement between two judges on randomly selected questions which measures the alignment of LLM judges with human preferences. Success rate is used for specific tasks (such as the Game of 24) to measure the correct response rate.

In contrast, LLM-based agents use more diverse evaluation metrics to reflect their multi-agent collaboration characteristics. In [[97](https://arxiv.org/html/2408.02479v1#bib.bib97)], evaluation metrics include Perception Score (P-Score), Cognition Score (C-Score), and Action Score (A-Score). These metrics comprehensively assess the model’s perception, cognition and action capabilities, demonstrating the comprehensive performance of LLM-based agents in handling multimodal tasks. In multimodal applications, success rate (SR) is often used as a primary metric, evaluated through tasks such as HotpotQA and FEVER to assess precise matching success. These metrics focus on task completion success and accuracy, showcasing the practical execution capabilities of LLM-based agents in different task environments. In [[111](https://arxiv.org/html/2408.02479v1#bib.bib111)], evaluation metrics include practitioner feedback, efficiency, and accuracy. Practitioner feedback uses the Likert scale to collect satisfaction and performance feedback, the Likert scale is a commonly used psychometric tool designed to measure an individual’s attitude or opinion toward a particular statement. The scale typically consists of the following five options: Strongly Disagree, Disagree, Neutral, Agree, Strongly Agree. While efficiency and accuracy are measured through the effectiveness of model-executed qualitative data analysis validated by practitioners. These metrics assess the agents’ performance in qualitative data analysis, demonstrating their utility and accuracy in practical applications.

By comparing these metrics, we find that LLMs using traditional metrics such as accuracy and sample efficiency to assess their capabilities. In contrast, LLM-based agents handle more complex algorithm through multi-agents, which requires more comprehensive and diverse metrics to evaluate their performance from multiple directions. LLM-based agents in multimodal tasks and self-evolution tasks emphasize the integrated performance of perception, cognition, and action capabilities. This difference reflects LLMs’ strengths in single-task optimization and LLM-based agents’ potential in collaborative handling of complex tasks with higher capability of autonomous learning. Additionally, practical application evaluation metrics for LLM-based agents, such as practitioner feedback, efficiency, and accuracy, demonstrate their utility and user satisfaction in real-world scenarios. This evaluation approach assesses task completion but also consider a comprehensive evaluation of user experience, which can also evaluate the human alignment of their decision making capabilities.

TABLE VII: Evaluation Metrics in Autonomous Learning and Decision Making

{tblr}

cell11 = c, cell12 = c, cell13 = c, cell24 = c, cell34 = c, cell44 = c, cell54 = c, cell64 = c, cell74 = c, cell84 = c, cell94 = c, cell104 = c, cell114 = c, cell124 = c, cell134 = c, cell144 = c, cell154 = c, cell164 = c, cell174 = c, cell184 = c, cell194 = c, cell204 = c, cell214 = c, cell224 = c, cell234 = c, cell244 = c, cell254 = c, hlines, vlines, Reference Paper & Benchmarks Evaluation Metrics Agent
 [[90](https://arxiv.org/html/2408.02479v1#bib.bib90)] MMLU Accuracy No
 [[91](https://arxiv.org/html/2408.02479v1#bib.bib91)] Spider, TransCoder, MBPP Accuracy, Sample Efficiency No
 [[92](https://arxiv.org/html/2408.02479v1#bib.bib92)] Defects4J v1.2, Defects4J v2.0,
Almost-Right HumanEval Plausible Patches,
Correct Patches,
Precision, Accuracy No
 [[93](https://arxiv.org/html/2408.02479v1#bib.bib93)] No Specific Quality, Acceptance Rate No
 [[110](https://arxiv.org/html/2408.02479v1#bib.bib110)] Game of 24, Creative Writing,
5x5 Crosswords Success rate, Coherency No
 [[95](https://arxiv.org/html/2408.02479v1#bib.bib95)] MT-Bench, Chatbot Arena Agreement Rate, Success Rate
Human Judgement No
 [[96](https://arxiv.org/html/2408.02479v1#bib.bib96)] ECQA, GSM8K, FOLIO-wiki Accuracy Yes
 [[97](https://arxiv.org/html/2408.02479v1#bib.bib97)] PCA-EVAL Accuracy, P/C/A-Score Yes
 [[35](https://arxiv.org/html/2408.02479v1#bib.bib35)] HotpotQA, ALFWorld, WebShop, FEVER Success Rate Yes
 [[106](https://arxiv.org/html/2408.02479v1#bib.bib106)] Not specified Success Rate, Autonomy Leve Yes
 [[44](https://arxiv.org/html/2408.02479v1#bib.bib44)] GSM8K, MATH, MMLU, Chess, HumanEval Accuracy Yes
 [[107](https://arxiv.org/html/2408.02479v1#bib.bib107)] MITRE ATTCK framework Ability Identify Vulnerabilities Yes
 [[102](https://arxiv.org/html/2408.02479v1#bib.bib102)] GSM8K, SVAMP, Vicuna testset,
Evol-Instruct testset Accuracy, Feedback Accuracy Yes
 [[98](https://arxiv.org/html/2408.02479v1#bib.bib98)] HotPotQA, ALFWorld, HumanEval,MBPP,
LeetcodeHardGym Pass@1, Success Rate Yes
 [[111](https://arxiv.org/html/2408.02479v1#bib.bib111)] Github Developer Discussions,BBC News,
Social MediaConversations,
In-depth Interviews Practitioner Feedback,
Efficiency and Accuracy Yes
 [[100](https://arxiv.org/html/2408.02479v1#bib.bib100)] AI Society, Code, Math,Science,
Misalignment Human Evaluation,
GPT-4 Evaluation Yes
 [[99](https://arxiv.org/html/2408.02479v1#bib.bib99)] FED, Commongen Challenge,
MGSM, Logic Grid Puzzles,
HumanEval Pass@1, Task completion rate Yes
 [[36](https://arxiv.org/html/2408.02479v1#bib.bib36)] HotpotQA, FEVER, ALFWorld, WebShop Exact Match, Accuracy,
Success rate, Average Score Yes
 [[103](https://arxiv.org/html/2408.02479v1#bib.bib103)] Trust Game, Dictator Game,
MAP Trust Game,
Risky Dictator Game,
Lottery Game, Repeated Trust Game Valid Response Rate,
Alignment Yes
 [[104](https://arxiv.org/html/2408.02479v1#bib.bib104)] HotPotQA, WebShop F1-Score, Average Reward Yes
 [[108](https://arxiv.org/html/2408.02479v1#bib.bib108)] 263 real smart contract vulnerabilities F1 Score, Accuracy
Precision, Recall
Consistency Rate. Yes
 [[109](https://arxiv.org/html/2408.02479v1#bib.bib109)] 15 real-world one-day vulnerabilities
from CVE database Success Rate, Cost Yes
 [[101](https://arxiv.org/html/2408.02479v1#bib.bib101)] WebShop, HotPotQA with Wikipedia AP Reward Score, Recall Yes
 [[105](https://arxiv.org/html/2408.02479v1#bib.bib105)] MMLU, Mini Crosswords, HumanEval,
GAIA Accuracy, Pass@1 Yes

## VII Software Design and Evaluation

The application of LLMs to software design and evaluation has very similar overlaps with previous topics, software design is an early phase of software development, and the quality of the design directly impacts the quality of furture development. Modern software engineering methodologies emphasize the integration of design and development to ensure that decisions made during the design phase seamlessly translate into high-quality code. Consequently, the research on software design often explores aspects related to code generation and development by utilizing LLMs for software development with a certain framework and special architecture design. Software design frameworks often involve multiple stages of continuous refinement to achieve optimal results, which can be considered part of LLM applications in software development  [[83](https://arxiv.org/html/2408.02479v1#bib.bib83)]. Similarly, [[85](https://arxiv.org/html/2408.02479v1#bib.bib85)] and [[84](https://arxiv.org/html/2408.02479v1#bib.bib84)] highlight the frequent use of tools or API interfaces when using LLMs to assist in development and design, demonstrating an overlap with the topic of code generation and software development.

LLMs in software design and evaluation also intersect extensively with autonomous learning and decision making, these two topics are interrelated fields. Software design needs to consider system adaptability and learning capabilities to handle dynamic environments, therefore design evaluations involving autonomous learning and decision making naturally become a focal point of intersection for these two topics. Many LLM techniques and methods find similar applications in both fields, for example LLMs based on reinforcement learning can be used for automated design decisions and evaluations, as well as for self-learning and optimization. Common applications of LLMs in software engineering involve fine-tuning models with prompt engineering techniques to continuously enhance performance particularly in software design and evaluation, more sample learning is often required to ensure that the model outputs align with user expectations [[93](https://arxiv.org/html/2408.02479v1#bib.bib93)] [[102](https://arxiv.org/html/2408.02479v1#bib.bib102)] [[44](https://arxiv.org/html/2408.02479v1#bib.bib44)] [[111](https://arxiv.org/html/2408.02479v1#bib.bib111)] [[105](https://arxiv.org/html/2408.02479v1#bib.bib105)] [[96](https://arxiv.org/html/2408.02479v1#bib.bib96)]. Additionally, requirement elicitation and specification in requirement engineering can also be considered part of software design and evaluation [[51](https://arxiv.org/html/2408.02479v1#bib.bib51)] [[112](https://arxiv.org/html/2408.02479v1#bib.bib112)]. This section reviews the main research achievements of LLMs in software design and evaluation in recent years, discussing their application scenarios and practical effects.

### VII-A LLMs Tasks

In recent years, there has been extensive research on the use of LLMs in tasks such as automation, optimization, and code understanding. ChatGPT has been widely utilized for various software engineering tasks and demonstrated excellent performance in tasks like log summarization, pronoun resolution, and code summarization, achieving a 100% success rate in both log summarization and pronoun resolution tasks [[113](https://arxiv.org/html/2408.02479v1#bib.bib113)]. However, its performance on tasks such as code review and vulnerability detection is relatively poor, which shows that it needs further improvement for more complex tasks. Another framework EvaluLLM addresses the limitations of traditional reference-based evaluation metrics (such as BLEU and ROUGE) by using LLMs to assess the quality of natural language generation (NLG) outputs [[114](https://arxiv.org/html/2408.02479v1#bib.bib114)]. The EvaluLLM introduces a new evaluation method that compares generative outputs in pairs and uses win rate metrics to measure model performance, this approach can simplifies the evaluation process also ensures consistency with human assessments, showcasing the broad application prospects of LLMs in generative tasks. Similarly, in the LLMs evaluation domain, LLM-based NLG Evaluation provides a review and classification of current LLMs used for NLG evaluation, the paper summarizes four main evaluation methods: LLM-derived metrics, prompt-based LLMs, fine-tuned LLMs, and human-LLM collaborative evaluations [[115](https://arxiv.org/html/2408.02479v1#bib.bib115)]. These methods demonstrate the potential of LLMs in evaluating generative outputs which also mention challenges such as the need for improved evaluation metrics and further exploration of human-LLM collaboration.

There are also many novel application design with the LLMs which applied in the engineering design, one study explores strategies for software/hardware co-design to optimize LLMs and applies these strategies to design verification [[116](https://arxiv.org/html/2408.02479v1#bib.bib116)]. Through quantization, pruning, and operation-level optimization, this research demonstrates applications in high-level synthesis (HLS) design functionality verification, GPT-4 was used to generate high-level synthesis (HLS) designs containing predefined errors to create a dataset called Chrysalis, this dataset provides a valuable resource for evaluating and optimizing LLM-based HLS debugging assistants. The optimized LLM significantly improves inference performance, providing new possibilities for error detection and correction in the electronic design automation (EDA) field. In [[117](https://arxiv.org/html/2408.02479v1#bib.bib117)], the researchers introduces RaWi, a data-driven GUI prototyping approach. The framework allows users to retrieve GUIs from this repository, edit them, and create new high-fidelity prototypes quickly. The experiment conducted by comparing RaWi with a traditional GUI prototyping tool (Mockplus) to measure how quickly and effectively users can create prototypes. The result demonstrated that RaWi outperformed on multiple benchmarks, with 40% improvement on precision@k metric. This study proves the possibility of LLMs to improve the efficiency during prototyping phase of software design, which allows designers to quickly iterate on GUI designs, facilitating early detection of design flaws. With the new possibility brought by the LLMs, there has been much discussion in the education field, with researchers exploring the implications of the prevalence of large language models for education [[118](https://arxiv.org/html/2408.02479v1#bib.bib118)]. Study indicates that ChatGPT shows significant potential but some limitations in answering questions from software testing courses [[119](https://arxiv.org/html/2408.02479v1#bib.bib119)]. ChatGPT was able to answer about 77.5% of the questions and provided correct or partially correct answers 55.6% of the time. However, the correctness of its explanations was only 53.0%, indicating the need for further improvement in educational applications.

### VII-B LLM-based Agents Tasks

The application of LLM-based agents in software design and evaluation enhance the development efficiency and code quality, as well as showcase the broad applicability and immense potential of LLM-based agents in practical software engineering tasks. [[120](https://arxiv.org/html/2408.02479v1#bib.bib120)] explores the current capabilities, challenges, and opportunities of autonomous agents in software engineering. Study evaluate Auto-GPT’s performance across different stages of the software development lifecycle (SDLC), including software design, testing, and integration with GitHub, the paper finds that detailed contextual prompts significantly enhance agent performance in complex software engineering tasks which mentions the importance of context-rich prompts in reducing errors and improving efficiency, underscoring the potential of LLM-based agents to automate and optimize various SDLC tasks, thereby enhancing development efficiency. This paper also evaluate the limitation of the Auto-GPT, includes task or goal skipping, generating unnecessary code or files (hallucinations), repetitive or looping responses, lack of task completion verification mechanisms. These limitations can lead to incomplete workflows, inaccurate outputs, and unstable performance in practical applications.

[[121](https://arxiv.org/html/2408.02479v1#bib.bib121)] introduces ChatDev, the first virtual chat-driven software development company, a concept of using LLMs not just for specific tasks but as central coordinators in a chat-based, multi-agent framework. this approach allows for more structured, efficient, and collaborative software development processes, exploring how chat-driven multi-agent systems can achieve efficient software design and evaluation, reduce code vulnerabilities, and enhance development efficiency and quality. Experiments show that ChatDev can design and generate software in an average of 409.84 seconds at a cost of only $0.2967 while significantly reducing code vulnerabilities. This indicates that chat-based multi-agent frameworks capable to improve software development efficiency and quality. Another similar collaboration framework introduced by Microsoft research team,  [[122](https://arxiv.org/html/2408.02479v1#bib.bib122)] demonstrates the effectiveness of using LLMs, particularly ChatGPT as agent’s controllers to manage and execute various AI tasks. The HuggingGPT system that uses ChatGPT to orchestrate the execution of tasks by various AI models available in Hugging Face, the purpose is to test how effectively the system can handle complex AI tasks, including language, vision, and speech tasks, by executing appropriate models based on user requests. The innovation lies in using LLMs not just as tools for direct task execution but as central orchestrators that leverage existing AI models to fulfill complex tasks, This approach expands the practical applicability of LLMs beyond typical language tasks.  [[123](https://arxiv.org/html/2408.02479v1#bib.bib123)] proposes the LLMARENA benchmark framework to evaluate LLMs’ capabilities in dynamic multi-agent environments, the idea is similar to the ChatDev but innovates by shifting the focus from single-agent static tasks to dynamic and interactive multi-agent environments, providing a more realistic and challenging setting to assess the practical utility of LLMs, this approach mirrors real-world conditions where multiple agents (either AI or human) interact and collaborate. Experiments show that this framework can test LLMs’ spatial designing, strategic planning, and teamwork abilities in gaming environments, offering new possibilities and tools for designing and evaluating LLMs in multi-agent systems.

[[124](https://arxiv.org/html/2408.02479v1#bib.bib124)] introduces the ”Flows” conceptual framework for structuring interactions between AI models and humans to improve reasoning and collaboration capabilities. The study present the idea of conceptualizing processes as independent, goal-driven entities that interact through standardized message-based interfaces, enabling a modular and extensible design. This approach is inherently concurrency-friendly and supports the development of complex nested AI interactions without having to manage complex dependencies. Experiments in competitive coding tasks show that the ”Flows” framework increases the AI model’s problem-solving rate by 21 percentage points and the human-AI collaboration rate by 54 percentage points. This demonstrates how modular design can enhance AI and human collaboration, thereby improving the software design and evaluation process.

[[125](https://arxiv.org/html/2408.02479v1#bib.bib125)] presents a new taxonomy to structurally understand and analyze LLM-integrated applications, providing new theories and methods for software design and evaluation. This taxonomy helps in understanding the integration of LLM components in software systems, laying a theoretical foundation for developing more effective and efficient LLM-integrated applications. Similarly, [[126](https://arxiv.org/html/2408.02479v1#bib.bib126)] explores the application of LLM-based agents in software maintenance tasks, improving code quality and reliability through a collaborative framework. This study should origin be categorized under the software maintenance domain but exhibit the iterative manner of the design structure. The framework utilize the task decomposition and multi-agent strategies to tackle complex engineering tasks that traditional one-shot methods cannot handle effectively, multiple agents can learn from each other, leading to improved software maintenance outcomes. Experiments show that multi-agent systems outperform single-agent systems in complex debugging tasks, indicating that this new framework can be applied in software design to provide safer architectures.

### VII-C Analysis

Overall, LLM applications in software design and evaluation typically focus on the automation of specific tasks, such as code generation and log summarization, with a tendency towards evaluation the capability rather than implementation during the design phases. The process of software design is largely intertwined with software development and requirements engineering. As previously mentioned, the use of LLMs to assist in software development often includes aspects of the software design process, particularly in generating related design documentation. Therefore, there is relatively limited research focused on using LLMs for higher-level software design tasks.

LLM-based agents expand the capabilities of LLMs by handling more complex workflows through intelligent decision-making and task execution, these agents can collaborate, dynamically adjust tasks and gather and utilize external information. In software design and evaluation, a single model often cannot comprehensively consider both design and evaluation aspects, which is why more software developers are reluctant to entrust high-level tasks to AI. LLM-based agents, through collaborative work and more refined role division, can efficiently complete design tasks and adapt to various application scenarios. However, the application of LLM-based agents in software design is commonly included in the software development, like previously discussed, the self-reflection and reasoning before action occurs during the software design phases. The Chatdev[[121](https://arxiv.org/html/2408.02479v1#bib.bib121)] framework uses role distribution to create a separate software design phase which significantly increases the flexibility and accuracy in the later development phases. In terms of efficiency and cost, LLMs are still slightly superior to LLM-based agents in text generation and vulnerability detection. However, handling tasks similar to software maintenance and root cause analysis requires more complex architectures, such as multi-turn dialogues, knowledge graphs, and RAG techniques, which can further benefit the design and evaluation phases.

### VII-D Benchmarks

The benchmarks include public datasets and datasets self-crafted by the authors themselves, and the application scenarios are also quite differently as shown in the Table [VIII](https://arxiv.org/html/2408.02479v1#S7.T8 "TABLE VIII ‣ VII-E Evaluation Metrics ‣ VII Software Design and Evaluation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). BigCloneBench is a benchmark dataset for code clone detection, containing a large number of Java function pairs. These pairs are classified as clones and non-clones, used for training and evaluating clone detection models, with the main evaluation metric being the correct identification rate. The Chrysalis dataset created by [[116](https://arxiv.org/html/2408.02479v1#bib.bib116)], it contains over 1000 function-level designs from 11 open-source synthesizable HLS datasets, primarily used to evaluate the effectiveness of LLM debugging tools in detecting and correcting injected errors in HLS designs, with the main evaluation metric being the effectiveness of error detection and correction. The CodexGLUE dataset is a comprehensive benchmark dataset covering various code generation and understanding tasks such as code completion, code repair, and code translation, used to evaluate the performance of code generation models in practical programming tasks. In addition to these public datasets, some artificially simulated datasets are used, such as a simulated job fair environment dataset. This dataset simulates a virtual job fair environment containing multiple task scenarios such as interviews, recruitment, and team project coordination. The dataset used to evaluate the coordination capabilities of generative agents in complex social tasks, with the main evaluation metrics being task coordination success rate and role matching accuracy.

Comparatively, LLMs research tends to use specific and publicly available datasets, such as BigCloneBench. These datasets provide standardized evaluation benchmarks, aiding in the reproducibility and comparability of results. Researches on LLM-based agents tends to use customized experimental settings or unspecified datasets, such as requirement documentations, without specifying particular datasets but emphasizing that the experiments involve 70 user requirements. This choice is usually because the research needs to evaluate the performance from multiple angles, and it is difficult to perfectly adapt to the vertical application scenarios if some general datasets are used. Both LLM and LLM-based agents use a variety of datasets to evaluate the performance of the model, these datasets cover tasks ranging from code generation, code understanding, to natural language generation and task management, due to the topic of software design and evaluation is relatively inter-related with others. However, because the LLM-based agents can be expanded to application scenarios such as videos and pictures, the agents like Auto-GPT and HuggingGPT also use multimodal datasets. These datasets not only contain code and text, but also involve multiple data types such as images and speech. Moreover, compared with a single LLM framework, LLM-based agents need to evaluate more areas, so benchmarks also need to be considered separately. For example, LLMARENA is specially designed to test the performance of LLM in dynamic, multi-agent environments, covering complex tasks such as spatial reasoning, strategic planning, and risk assessment.

### VII-E Evaluation Metrics

In Software Design and Evaluation, various studies have employed different evaluation metrics to measure the performance of LLMs and LLM-based agents across a range of tasks. Both LLM and LLM-based agent research use more than one metrics to comprehensively assess model performance, LLMs research tends to focus on traditional metrics such as accuracy, win rate, and consistency, while LLM-based agent research still consider those fundamental metrics but further introduces complex evaluation methods, such as task coordination success rate and role matching accuracy. However, it cannot be definitively stated that future LLM-based agent research will always use more flexible evaluation metrics considering multiple dimensions, but more dependent on the specific task and dataset being used. The reason for this phenomenon, as observed in this survey, is primarily that tasks in LLMs research are relatively single-tasked, mainly focusing on static tasks such as log summarization with traditional evaluation methods. On the other hand, LLM-based agent research involves more general multi-agent tasks, and its evaluation methods emphasize interactivity and dynamics. LLM-based agent research focuses more on the model’s collaboration and decision-making capabilities by using multi-dimensional evaluation metrics to comprehensively assess their potential in practical applications consider not only the accuracy. This explains why, despite the similarity in evaluation metrics such as accuracy and completion time, LLM-based agents use flexible evaluation metrics, including metrics like mutual exclusiveness and appropriateness.

TABLE VIII: Evaluation Metrics in Software Design and Evaluation

| Reference Paper | Benchmarks | Evaluation Metrics | Agent |
| --- | --- | --- | --- |
|  

&#124; [[113](https://arxiv.org/html/2408.02479v1#bib.bib113)] &#124;

 |  

&#124; BigCloneBench, &#124;
&#124; Python functions, &#124;
&#124; Java methods, &#124;
&#124; Random logs, &#124;
&#124; Bug reports, &#124;
&#124; Requirement specifications &#124;

 | Accuracy | No |
|  

&#124; [[114](https://arxiv.org/html/2408.02479v1#bib.bib114)] &#124;

 | Not Specified | Win rate, Agreement score | No |
|  

&#124; [[115](https://arxiv.org/html/2408.02479v1#bib.bib115)] &#124;

 | Not Specified |  

&#124; Embedding-based metrics, &#124;
&#124; probability-based metrics, &#124;
&#124; Comparison, Ranking &#124;

 | No |
|  

&#124; [[116](https://arxiv.org/html/2408.02479v1#bib.bib116)] &#124;

 | Chrysalis | Effectiveness | No |
|  

&#124; [[127](https://arxiv.org/html/2408.02479v1#bib.bib127)] &#124;

 |  

&#124; CommonsenseQA, &#124;
&#124; StrategyQA, GSM8K &#124;

 |  

&#124; Accuracy, &#124;
&#124; Token, Time costs &#124;

 | No |
|  

&#124; [[119](https://arxiv.org/html/2408.02479v1#bib.bib119)] &#124;

 |  

&#124; 31 Questions from &#124;
&#124; software testing textbook. &#124;

 | Correctness, Effectiveness | No |
|  

&#124; [[128](https://arxiv.org/html/2408.02479v1#bib.bib128)] &#124;

 |  

&#124; Medical transcripts, &#124;
&#124; Amazon Product &#124;
&#124; Descriptions &#124;

 |  

&#124; Coverage, &#124;
&#124; False Failure Rate &#124;
&#124; Alignment. &#124;

 | No |
|  

&#124; [[117](https://arxiv.org/html/2408.02479v1#bib.bib117)] &#124;

 | Rico |  

&#124; Precision@k, &#124;
&#124; NDCG@k, &#124;
&#124; Mean Reciprocal Rank, &#124;
&#124; Average Precision, HITS@k &#124;

 | No |
|  

&#124; [[120](https://arxiv.org/html/2408.02479v1#bib.bib120)] &#124;

 | Not Specified |  

&#124; Accuracy, Success rate, &#124;
&#124; Consistency, Effectiveness &#124;

 | Yes |
|  

&#124; [[122](https://arxiv.org/html/2408.02479v1#bib.bib122)] &#124;

 |  

&#124; Hugging Face’s &#124;
&#124; Model Repository. &#124;

 |  

&#124; Accuracy, &#124;
&#124; Precision, &#124;
&#124; Recall, &#124;
&#124; F1-Score, &#124;
&#124; Edit Distance, &#124;
&#124; GPT-4 Score, &#124;
&#124; Passing Rate, &#124;
&#124; Rationality, &#124;
&#124; Success Rate. &#124;

 | Yes |
|  

&#124; [[124](https://arxiv.org/html/2408.02479v1#bib.bib124)] &#124;

 | Codeforces, LeetCode | Pass@1 | Yes |
|  

&#124; [[121](https://arxiv.org/html/2408.02479v1#bib.bib121)] &#124;

 | 70 User Requirements. |  

&#124; Number of files generated, &#124;
&#124; Time taken, Cost &#124;

 | Yes |
|  

&#124; [[121](https://arxiv.org/html/2408.02479v1#bib.bib121)] &#124;

 | Codeforces |  

&#124; Comprehensiveness, &#124;
&#124; Robustness, Conciseness, &#124;
&#124; Mutual exclusiveness, &#124;
&#124; Explanatory power, &#124;
&#124; Extensibility. &#124;

 | Yes |
|  

&#124; [[125](https://arxiv.org/html/2408.02479v1#bib.bib125)] &#124;

 | Sample Applications. | BERTScore, BLEU | Yes |
|  

&#124; [[126](https://arxiv.org/html/2408.02479v1#bib.bib126)] &#124;

 | CodexGLUE |  

&#124; BLEU, METEOR, &#124;
&#124; ROUGE-L, BERTScore &#124;

 | Yes |
|  

&#124; [[129](https://arxiv.org/html/2408.02479v1#bib.bib129)] &#124;

 | Production Incidents |  

&#124; Success rate, &#124;
&#124; Accuracy, Alignment, &#124;
&#124; Appropriateness &#124;

 | Yes |
|  

&#124; [[130](https://arxiv.org/html/2408.02479v1#bib.bib130)] &#124;

 |  

&#124; Simulated Job Fair &#124;
&#124; Environment &#124;

 |  

&#124; Completion time, &#124;
&#124; Task Progress, &#124;
&#124; Understanding Level &#124;

 | Yes |

## VIII Software Test Generation

In software development, a crucial component is software testing, which need to continuously been conducted from the initial system development to the final deployment. In industry, agile development is commonly used which test system continuously at every stage to ensure the robustness of the entire system, whenever new code is committed to the GitHub, tests are conducted to ensure the usability of the updated version. A common approach is to use Jenkins⁴⁴4[https://www.jenkins.io/](https://www.jenkins.io/) to achieve continuous integration and continuous deployment. Jenkins automatically hooks into the developer’s action of pushing code to GitHub and runs a test suite against the new version. Although the entire process leans towards automated development, creating and refining test cases still requires large human effort.

Typical roles in development involve software testing, such as writing unit tests, integration tests, and fuzz tests. Researchers have been attempting to use AI to help generate test cases since before the 2000\. Initial implementations typically involved simpler forms of AI and machine learning to automate parts of the test case generation process. Over time, more sophisticated methods such as natural language processing and machine learning models have been applied to improve the precision and scope of test case generation. Online tools like Sofy⁵⁵5[https://sofy.ai/](https://sofy.ai/), which use machine learning to generate context-based paths in applications, also exist to aid in generating test suites. Using large language models to generate test cases is a relatively new attempt but has been developing rapidly. In 2020, researchers utilized pre-trained language models fine-tuned on labeled data to generate test cases. They developed a sequence-to-sequence transformer-based model called ”ATHENATEST” and compared its generated results with EvoSuite and GPT-3, demonstrating better test coverage [[131](https://arxiv.org/html/2408.02479v1#bib.bib131)]. More research and models are being dedicated to test suite generation experiments, for instance, the Codex model [[67](https://arxiv.org/html/2408.02479v1#bib.bib67)], mentioned earlier in the code generation section, combined with chain-of-thought prompting, achieved high-quality test suite generation with CodeCoT, even in zero-shot scenarios. The introduction of LLMs aims to automate and streamline the testing process, making it more rigorous and capable of addressing aspects that humans might easily overlook.

### VIII-A LLMs Tasks

The application of LLMs in software test generation is extensive and encompasses more than just test suite generation. The reviewed paper included in this survey covers several aspects, including security test generation, bug reproduction, general bug reproduction, fuzz testing, and coverage-driven test generation. These tasks are achieved through various models and techniques, significantly improving software quality and reducing developers’ workload.  [[132](https://arxiv.org/html/2408.02479v1#bib.bib132)] aims to evaluate the effectiveness of using GPT-4 to generate security tests, demonstrating how to conduct supply chain attacks by exploiting dependency vulnerabilities. The study experimented with different prompt styles and templates to explore the effectiveness of varying information inputs on test generation quality, the results showed that tests generated by ChatGPT successfully discovered 24 proof-of-concept vulnerabilities in 55 applications, outperforming existing tools TRANSFER[[133](https://arxiv.org/html/2408.02479v1#bib.bib133)] and SIEGE⁶⁶6[https://siegecyber.com.au/services/penetration-testing/](https://siegecyber.com.au/services/penetration-testing/). This research introduces a new method for generating security tests using LLMs and provides empirical evidence of LLM’s potential in the security testing domain, offering developers a novel approach to handling library vulnerabilities in applications.

Another application is bug reproduction, which allows testers to locate and fix bugs more quickly and efficiently. [[134](https://arxiv.org/html/2408.02479v1#bib.bib134)] addresses the limitations of current bug reproduction methods, which are constrained by the quality and clarity of handcrafted patterns and predefined vocabularies. The paper proposes and evaluates a new method framework called AdbGPT, which uses a large language model to automatically reproduce errors from Android bug reports. AdbGPT is described as outperforming current SOTA approaches in the context of automated bug replay for only Android system. The experimental results show that AdbGPT achieved accuracies of 90.4% and 90.8% in S2R entity extraction and a success rate of 81.3% in error reproduction, significantly outperforming the baseline ReCDroid and ablation study versions. By introducing prompt engineering, few-shot learning, and chain-of-thought reasoning, AdbGPT demonstrates the powerful capabilities of LLMs in automated error reproduction. It also uses GUI encoding to convert the GUI view hierarchy into HTML-like syntax, providing LLMs with a clear understanding of the current GUI state. While AdbGPT is specialized for Android systems, [[135](https://arxiv.org/html/2408.02479v1#bib.bib135)] proposes the LIBRO framework, which uses LLMs to generate bug reproduction tests from bug reports. The experimental results show that LIBRO successfully reproduced 33.5% of bugs in the Defects4J dataset and 32.2% in the GHRB dataset. By combining advanced prompt engineering and post-processing techniques, LIBRO demonstrates the effectiveness and efficiency of LLMs in generating bug reproduction tests. Although LIBRO has a lower absolute effectiveness compared to AdbGPT, it was tested across a more diverse set of Java applications and not limited to Android. Therefore, while AdbGPT excels in specialized bug replay for Android, LIBRO provides a wider range of bug reproduction for Java applications. The extensive application of LLMs in test generation tasks such as security test generation, bug reproduction, fuzz testing, program repair, and coverage-driven test generation highlights their significant potential in improving software quality and reducing the burden on developers. Through various models and techniques, these tasks demonstrate how LLMs can automate and enhance the software testing process, addressing aspects that are often overlooked by humans.

Similarly, in fuzz testing, LLMs have shown promise potential. [[136](https://arxiv.org/html/2408.02479v1#bib.bib136)] developed a universal fuzzing tool, Fuzz4All, which uses LLMs to generate and mutate inputs for various software systems. This tool addresses the issues of traditional fuzzers being tightly coupled with specific languages or systems and lacking support for evolving language features. The study conducted various experiments to test the tool’s capabilities, including coverage comparison, bug finding, and targeted fuzzing. The results showed that Fuzz4All achieved the highest code coverage in all tested languages, with an average increase of 36.8%, and discovered 98 bugs across nine systems, which considered as state-of-art technique in universal fuzzing with LLMs at that time. Through self-prompting and LLM-driven fuzzing loops, Fuzz4All demonstrated the effectiveness of LLMs in fuzz testing and showcased their capability across multiple languages and systems under test (SUTs) through comprehensive evaluations.

[[137](https://arxiv.org/html/2408.02479v1#bib.bib137)] introduced SymPrompt, a new code-aware prompting strategy aimed at addressing the limitations of existing Search-Based Software Testing (SBST) methods and traditional LLM prompting strategies in generating high-coverage test cases. By decomposing the original test generation process into a multi-stage sequence aligned with the execution paths of the method under test, SymPrompt generated high-coverage test cases. Experimental results indicated that SymPrompt increased coverage on CodeGen2 and GPT-4 by 26% and 105% respectively. Through path constraint prompting and context construction techniques, SymPrompt demonstrated the potential of LLMs in generating high-coverage test cases. [[138](https://arxiv.org/html/2408.02479v1#bib.bib138)] also focused on test suite coverage, this study introduced the COVERUP system which generates high-coverage Python regression tests through coverage analysis and interaction with LLMs. The experimental results showed that COVERUP increased code coverage from 62% to 81% and branch coverage from 35% to 53% through iterative prompting and coverage-driven methods. [[139](https://arxiv.org/html/2408.02479v1#bib.bib139)] proposed the AID method, which combines LLMs with differential testing to improve fault detection in ”plausibly correct” software. By comparing the effectiveness of AID in generating fault-revealing test inputs and oracles, the experiments showed that AID improved recall and precision by 1.80 times and 2.65 times respectively, and increased the F1 score by 1.66 times. By integrating LLMs with differential testing, AID showcased the powerful capability of LLMs in detecting complex bugs.

### VIII-B LLM-based Agents Tasks

In the field of software test generation, the application of LLM-based agents demonstrates their potential in automated test generation. While relying on LLM-based agents for software test generation might seem excessive, more research is directed towards vulnerability detection and system maintenance. LLM-based agents can enhance test reliability and quality by distributing tasks such as test generation, execution, and optimization through a multi-agent collaborative system. These multi-agent systems offer obvious improvements in error detection and repair, and coverage testing. An example of such a system is AgentCoder’s multi-agent framework, as discussed in the code generation and software development section [[82](https://arxiv.org/html/2408.02479v1#bib.bib82)]. The primary goal of this system is to leverage multiple specialized agents to iteratively optimize code generation, overcoming the limitations of a single agent model in generating effective code and test cases. The paper introduce the test design agent, which creates diverse and comprehensive test cases; and the test execution agent, which executes the tests and provides feedback, it reached an 89.9% pass rate on the MBPP dataset. Similarly, the SocraTest framework falls under the Autonomous Learning and Decision Making topic [[106](https://arxiv.org/html/2408.02479v1#bib.bib106)]. This framework automates the testing process through conversational interactions, the paper presents detailed examples of generating and optimizing test cases using GPT-4, emphasizing how multi-step interactions enhance testing methods and generate test code. Experimental results show that through conversational LLMs, SocraTest can effectively generate and optimize test cases and utilize middleware to facilitate interactions between the LLM and various testing tools, achieving more advanced automated testing capabilities.

The paper collected for the software test generation topic are mostly multiple agents based system. The study [[140](https://arxiv.org/html/2408.02479v1#bib.bib140)] evaluates the effectiveness of LLMs in generating high-quality test cases and identifies their limitations. It proposes a novel multi-agent framework called TestChain. The paper evaluates StarChat, CodeLlama, GPT-3.5, and GPT-4 on the HumanEval and LeetCode-hard datasets. Experimental results show that the TestChain framework, using GPT-4, achieved 71.79% accuracy on the LeetCode-hard dataset, an improvement of 13.84% over baseline methods. On the HumanEval dataset, TestChain with GPT-4 achieved 90.24% accuracy. The TestChain framework designs agents to generate diverse test inputs, maps inputs to outputs using ReAct format dialogue chains, and interacts with the Python interpreter to obtain accurate test outputs.

LLM-based agents can also be applied in user acceptance testing (UAT), [[141](https://arxiv.org/html/2408.02479v1#bib.bib141)] aims to enhance the automation of the WeChat Pay UAT process by proposing a multi-agent collaborative system named XUAT-Copilot, which uses LLMs to automatically generate test scripts. The study evaluates XUAT-Copilot’s performance on 450 test cases from the WeChat Pay UAT system, comparing it to a single-agent system and a variant without the reflection component. Experimental results show that XUAT-Copilot achieved a Pass@1 rate of 88.55%, compared to 22.65% for the single-agent system and 81.96% for the variant without the reflection component, with a Complete@1 rate of 93.03%. XUAT-Copilot employs a multi-agent collaborative framework, including action planning, state checking, and parameter selection agents, and uses advanced prompting techniques. XUAT-Copilot demonstrates the potential and feasibility of LLMs in automating UAT test script generation.

### VIII-C Analysis

![Refer to caption](img/5ec0ec72ae62caed6c2d25e54bb660a1.png)

Figure 7: Illustration of Comparison Framework Between LLM-based Agent[[141](https://arxiv.org/html/2408.02479v1#bib.bib141)] and LLM[[136](https://arxiv.org/html/2408.02479v1#bib.bib136)] in Software Test Generation

In comparison, LLMs perform well in single-task implementations, generating high-quality test cases through techniques like prompt engineering and few-shot learning. The number of related studies is increasing as the capabilities of LLMs improve. On the other hand, LLM-Based Agents, through multi-agent collaborative systems, decompose tasks for specialized processing, significantly enhancing the effectiveness and efficiency of test generation and execution through iterative optimization and feedback. Considering the cost, using LLMs for test generation only is enough and more cost saving than using LLM-based agents. However, if a specific model performs poorly, it can affect the entire system’s performance.

A single LLM may struggle with complex, multi-step tasks. For example, in high-coverage test generation, LLMs may require more complex prompts and post-processing steps to achieve the desired results. Additionally, the quality of the generated results depends heavily on the prompt design and quality. For tasks requiring fine control and continuous optimization, a single LLM may find it challenging to deal with. As shown in Figure.LABEL:testGen, the LLM framework uses [[136](https://arxiv.org/html/2408.02479v1#bib.bib136)] as an example to demonstrate the usage of LLMs in fuzz testing, the prompt will be optimized by given code snippets (fuzz inputs), and re-select by the LLM again to choose the best prompt for the future generation. The overall framework lacks autonomy, the LLM-based agent [[141](https://arxiv.org/html/2408.02479v1#bib.bib141)] framework on the left fills this gap, as well as able to perceive the UI and interact with the skill library for the operations. The operation agent will receive any error reported by the inspection agent and do the self-reflection to refine the process autonomously. However, as previously discussed, build a LLM-based agents framework only for the software test generation task are ”overkill”, so the collected paper for LLM-based agents system generally focused on program repair by generated test cases or bug replay system, like in the Figure.LABEL:testGen, the LLM-based agent framework is actually used for automatically test the Wechat Pay system.

### VIII-D Benchmarks

In the tasks of LLMs in software test generation, the dataset Defects4J used to evaluate bug reproduction and program repair techniques. Other public datasets such as ReCDroid, ANDROR2+, and Themis are primarily used to evaluate mobile application bug reproduction and security test generation, particularly for the Android platform. GCC, Clang, Go toolchain, Java compiler (javac), and Qiskit involve fuzz testing datasets for various programming languages and toolchains, aimed at assessing the effectiveness of fuzz testing in multi-language environments. TrickyBugs and EvalPlus are datasets containing complex bug scenarios, used to evaluate the precision and recall of generated test cases, the benchmark applications evaluated by CODAMOSA are used to assess the effectiveness of coverage-based test generation tools.

The datasets used in LLM-Based Agents research are also quite common, HumanEval, MBPP, and LeetCode-hard are mainly used to evaluate the accuracy and coverage of code generation and test generation, involving various programming problems and challenges which frequently appeared in previous sections. Datasets like Codeflaws, QuixBugs, and ConDefects are collected to familiarize LLMs with erroneous code and programs, containing multiple program errors and defects, and are used to evaluate the effectiveness of automated debugging and bug repair. A unique dataset is the WeChat Pay UAT system, which includes user acceptance test cases from actual applications and is used to evaluate the performance of multi-agent systems in user acceptance testing, focusing specifically on WeChat’s security system.

Overall, the datasets used in LLM-based agents’ research are broader covering a wide range of programming problems and challenges, LLM research is more focused on the actual generation tasks, such as bug reproduction on the Android platform and fuzz testing in multi-language environments. This because the LLM-Based agents not only focus on the quality of generated test cases and code but also evaluate the collaborative effects and iterative optimization capabilities of multi-agent systems, so the benchmarks also include the dataset used to evaluate performance of the framework. For instance, AgentCoder [[82](https://arxiv.org/html/2408.02479v1#bib.bib82)] improves the efficiency and accuracy of test generation and execution through multi-agent collaboration consider qualitative and quantitative evaluations and using MBPP,HummanEval to do the evaluations, researches on LLM-Based agents places more emphasis on verifying the effectiveness of the system through qualitative evaluation and user feedback.

### VIII-E Evaluation Metrics

As seen in Table [IX](https://arxiv.org/html/2408.02479v1#S8.T9 "TABLE IX ‣ VIII-E Evaluation Metrics ‣ VIII Software Test Generation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), LLMs research predominantly utilizes traditional quantitative metrics such as bug reproduction rate, code coverage, precision, and recall, these metrics directly reflect the effectiveness and quality of test generation. In contrast, LLM-Based agents research not only focuses on quantitative metrics but also introduces qualitative evaluations, such as improvements through conversational interactions and the collaborative effects of multi-agent systems. This diversified evaluation approach provides a more comprehensive reflection of the system’s practical application effects. From the task perspectives, LLMs are more inclined to single task processing, such as generating test sets and considering the coverage of generated test sets. However, because of the expansion of agents framework, LLM-based agents often tend to use the generated test sets to evaluate whether vulnerabilities can be found to achieve a more ideal practicality. From a design perspective, LLM systems are relying on prompt engineering and the generative capabilities of the models themselves, their evaluation metrics are also mainly focused on the quality and effectiveness of the model outputs, also their evaluation metrics include the collaborative effects and efficiency within the system, such as improving Pass@1 and Complete@1 rates through multi-agent collaboration. Overall, LLMs are more suited for rapid test generation and evaluation for specific tasks, with evaluation metrics directly reflecting the generation’s effectiveness and quality. LLM-Based Agents excel in handling complex and diversified tasks, achieving higher system efficiency and effectiveness through multi-agent collaboration and iterative optimization.

TABLE IX: Evaluation Metrics in Software Test Generation

| Reference Paper | Benchmarks | Evaluation Metrics | Agent |
| --- | --- | --- | --- |
|  

&#124; [[132](https://arxiv.org/html/2408.02479v1#bib.bib132)] &#124;

 |  

&#124; 26 libraries and 55 &#124;
&#124; applications with &#124;
&#124; known vulnerabilities &#124;

 |  

&#124; Number of applications for &#124;
&#124; which security tests were successfully &#124;
&#124; generated.Number of tests that could &#124;
&#124; demonstrate exploits. &#124;

 | No |
| [[134](https://arxiv.org/html/2408.02479v1#bib.bib134)] |  

&#124; ReCDroid, ANDROR2+, &#124;
&#124; Themis Empirical Study Dataset &#124;

 |  

&#124; Accuracy of S2R Entity Extraction. &#124;
&#124; Reproducibility of Bugs. &#124;
&#124; Runtime Efficiency. &#124;
&#124; User Satisfaction. &#124;

 | No |
|  

&#124; [[135](https://arxiv.org/html/2408.02479v1#bib.bib135)] &#124;

 | Defects4J, GHRB |  

&#124; Bug Reproduction Rate. &#124;
&#124; Precision and Recall. &#124;
&#124; Execution Time. &#124;
&#124; Developer Effort. &#124;

 | No |
|  

&#124; [[136](https://arxiv.org/html/2408.02479v1#bib.bib136)] &#124;

 |  

&#124; GCC and Clang. &#124;
&#124; CVC5 and Z3. &#124;
&#124; Go Toolchain. &#124;
&#124; Java Compiler (javac). &#124;
&#124; Qiskit. &#124;

 |  

&#124; Code Coverage. &#124;
&#124; Validity Rate. &#124;
&#124; Hit Rate. &#124;
&#124; Bugs Detected. &#124;

 | No |
|  

&#124; [[137](https://arxiv.org/html/2408.02479v1#bib.bib137)] &#124;

 |  

&#124; 897 focal methods from 26 &#124;
&#124; widely used open-source &#124;
&#124; Python projects. &#124;

 |  

&#124; Pass@1. &#124;
&#124; FM Call@1. &#124;
&#124; Correct@1. &#124;
&#124; Line & Branch Coverage. &#124;

 | No |
|  

&#124; [[139](https://arxiv.org/html/2408.02479v1#bib.bib139)] &#124;

 |  

&#124; TrickyBugs &#124;
&#124; EvalPlus datasets. &#124;

 |  

&#124; Recall. &#124;
&#124; Precision. &#124;
&#124; F1 Score. &#124;

 | No |
|  

&#124; [[138](https://arxiv.org/html/2408.02479v1#bib.bib138)] &#124;

 |  

&#124; Benchmark applications originally &#124;
&#124; used to evaluate CODAMOSA. &#124;

 |  

&#124; Line Coverage. &#124;
&#124; Branch Coverage. &#124;
&#124; Line + Branch Coverage. &#124;

 | No |
|  

&#124; [[82](https://arxiv.org/html/2408.02479v1#bib.bib82)] &#124;

 |  

&#124; HumanEval. &#124;
&#124; MBPP. &#124;
&#124; HumanEval-ET. &#124;
&#124; MBPP-ET. &#124;

 | Pass@1 | Yes |
|  

&#124; [[106](https://arxiv.org/html/2408.02479v1#bib.bib106)] &#124;

 | Not Specified |  

&#124; Qualitative improvement through &#124;
&#124; conversational interactions. &#124;

 | Yes |
|  

&#124; [[140](https://arxiv.org/html/2408.02479v1#bib.bib140)] &#124;

 |  

&#124; HumanEval. &#124;
&#124; LeetCode-hard. &#124;

 |  

&#124; Accuracy. &#124;
&#124; Line Coverage (Line Cov). &#124;
&#124; Code-with-Bugs (CwB). &#124;

 | Yes |
|  

&#124; [[142](https://arxiv.org/html/2408.02479v1#bib.bib142)] &#124;

 |  

&#124; Codeflaws. &#124;
&#124; QuixBugs. &#124;
&#124; ConDefects. &#124;

 |  

&#124; Number of Correct Patches. &#124;
&#124; Number of Plausible Patches. &#124;
&#124; Correctness Rate. &#124;

 | Yes |
|  

&#124; [[141](https://arxiv.org/html/2408.02479v1#bib.bib141)] &#124;

 |  

&#124; 450 test cases from the &#124;
&#124; WeChat Pay UAT system &#124;

 |  

&#124; Pass@1. &#124;
&#124; Complete@1. &#124;

 | Yes |

## IX Software Security and Maintenance

In the software engineering, software security and maintenance is a popular area for the application of LLMs, primarily aimed at enhancing the security and stability of software systems through existing technologies to meet the needs of users and developers. These models provide promise methods of vulnerability detection and repair, while also enabling automated security testing and innovative maintenance processes. The application of LLMs in software security and maintenance encompasses several aspects, including vulnerability detection, automatic repair, penetration testing, and system robustness evaluation. Compared to traditional methods, LLMs leverage natural language processing and generation technologies to understand and generate complex code and security policies, thereby automating detection and repair tasks. For example, LLMs can accurately identify potential vulnerabilities by analyzing code structures and contextual information and generate corresponding repair suggestions which improves the efficiency and accuracy of vulnerability recovery.

Moreover, LLMs not only exhibit strong capabilities in vulnerability detection but also play a role in tasks like penetration testing and security evaluations. Automated penetration testing tools, such as PENTESTGPT [[143](https://arxiv.org/html/2408.02479v1#bib.bib143)]. LLMs also demonstrate significant advantages in evaluating system robustness by simulating various attack scenarios to assess system performance under different conditions, helping developers better identify and address potential security issues. Research on LLM-based agents in software security and maintenance is also keep growing, these intelligent agents can execute complex code generation and vulnerability repair tasks and possess self-learning and optimization capabilities to handle issues encountered in dynamic development environments. Tools like RITFIS [[144](https://arxiv.org/html/2408.02479v1#bib.bib144)] and NAVRepair [[145](https://arxiv.org/html/2408.02479v1#bib.bib145)] have shown potential in improving the precision and efficiency of program repairs by using LLM-based agents.

### IX-A LLMs Tasks

In the field of software security and maintenance, research on LLMs can be categorized into three main areas: vulnerability detection, automatic repair, and penetration testing, along with some evaluation studies. The collected papers reviewed on LLMs in these domain illustrate their diverse applications and potential.

#### IX-A1 Program Vulnerability

In the domain of vulnerability detection, researchers have fine-tuned LLMs to enhance the accuracy of source code vulnerability detection. [[146](https://arxiv.org/html/2408.02479v1#bib.bib146)] aims to investigate the potential of applying LLMs to the task of vulnerability detection in source code and to determine if the performance limits of CodeBERT-like models are due to their limited capacity and code understanding ability. The study fine-tuned the WizardCoder model (an improved version of StarCoder) and compared its performance with the ContraBERT model on balanced and unbalanced datasets. The experimental results showed that WizardCoder outperformed ContraBERT in both ROC AUC and F1 scores, significantly improving Java function vulnerability detection performance, which achieved the state-of-art performance at that time by improving ROC AUC from 0.66 in CodeBERT to 0.69.

There are study mainly explored the applications of pure LLMs without any framework architecture in vulnerability detection, uncovering current challenges. [[147](https://arxiv.org/html/2408.02479v1#bib.bib147)] evaluated only the performance of ChatGPT and GPT-3 models in detecting vulnerabilities in Java code, the study compared text-davinci-003 (GPT-3) and gpt-3.5-turbo against a baseline virtual classifier in binary and multi-label classification tasks. The experimental results showed that while text-davinci-003 and gpt-3.5-turbo had high accuracy and recall rates in binary classification tasks, their AUC (Area Under Curve) scores were only 0.51, indicating performance equivalent to random guessing. In multi-label classification tasks, GPT-3.5-turbo and text-davinci-003 did not significantly outperform the baseline virtual classifier in overall accuracy and F1 scores. These findings indicate that the earlier model like GPT-3 has limited capabilities in practical vulnerability detection tasks, suggesting the need for further research and model optimization to improve their performance in real-world applications, fine-tuning and optimizing LLMs can significantly enhance their performance in source code vulnerability detection, However, these models still face many challenges in practical applications, requiring further research and technological improvements to enhance their real-world effectiveness and reliability.

In the later years, [[148](https://arxiv.org/html/2408.02479v1#bib.bib148)] introduced a method to incorporate complex code structures directly into the model learning process, the GRACE framework combines graph structure information and in-context learning, using Code Property Graphs (CPGs) to represent code structure information. By integrating the semantic, syntactic, and lexical similarities of code, the framework GRACE addresses the limitations of text-based LLM analysis, improves the precision and recall rates of vulnerability detection tasks. The study utilized three vulnerability datasets, showing a 28.65% improvement in F1 scores over baseline models, an important aspect of vulnerability detection is enhancing LLM performance in code security tasks. [[149](https://arxiv.org/html/2408.02479v1#bib.bib149)] fine-tuned LLMs for specific tasks and evaluated their performance against existing models such as ContraBERT. The researchers conducted numerous experiments to determine the optimal model architecture, training hyperparameters, and loss functions to optimize performance in vulnerability detection tasks. The study primarily focused on WizardCoder and ContraBERT, validating their performance through comparisons on balanced and unbalanced datasets and developing an efficient batch packing strategy that improved training speed. Results indicated that with appropriate fine-tuning and optimization, LLMs could surpass state-of-the-art models, contributing to more robust and secure software development practices.

Despite the development of numerous models, it is still necessary to investigate their practical effectiveness. [[150](https://arxiv.org/html/2408.02479v1#bib.bib150)] explored the effectiveness of code language models (code LMs) in detecting software vulnerabilities and identified significant flaws in existing vulnerability datasets and benchmarks. The researchers developed a new dataset called PRIMEVUL, and conducted experiments using it, they compared PRIMEVUL with existing benchmarks such as BigVul to evaluate several code LMs, including state-of-the-art base models like GPT-3.5 and GPT-4, using various training techniques and evaluation metrics. The results revealed that existing benchmarks significantly overestimated the performance of code LMs. For example, a state-of-the-art 7B model scored an F1 of 68.26% on BigVul but only 3.09% on PRIMEVUL, highlighting the gap between current code language models’ performance and actual requirements for vulnerability detection.

#### IX-A2 Automating Program Repair

In the domain of software security and maintenance, LLMs have not only been applied to vulnerability detection but also extensively used for automating program repair. One study proposed using Round-Trip Translation (RTT) for automated program repair, researchers translated defective code into another language and then back to the original language to generate potential patches. The study used various language models and benchmarks to evaluate RTT’s performance in APR. The experiments explored how RTT performs when using programming languages as an intermediate representation, how RTT performs when using natural language (English) as an intermediate representation, and what qualitative trends can be observed in the patches generated by RTT. Three measurement standards and eight models were used in the experiments, the results showed that the RTT method achieved significant repair effects on multiple benchmarks, particularly excelling in terms of compilation and feasibility [[151](https://arxiv.org/html/2408.02479v1#bib.bib151)]. Similarly, in automated program repair, [[145](https://arxiv.org/html/2408.02479v1#bib.bib145)] introduced several innovative methods. For example, NAVRepair specifically targets C/C++ code vulnerabilities by combining node type information and error types. Due to the unique pointer operations and memory management issues in C/C++, this language poses complexities. The framework uses Abstract Syntax Trees (ASTs) to extract node type information and combines it with CWE-derived vulnerability templates to generate targeted repair suggestions, the study evaluated NAVRepair on several popular LLMs (ChatGPT, DeepSeek Coder, and Magicoder) to demonstrate its effectiveness in improving code vulnerability repair performance. The results showed that NAVRepair achieved state-of-art performance in C/C++ program repair task, which improved repair accuracy by 26% compared to existing methods.

In order to address the two main limitations of existing fine-tuning methods for LLM-based program repair, which is the lack of reasoning about the logic behind code changes and high computational costs associated with fine-tuning large datasets. [[152](https://arxiv.org/html/2408.02479v1#bib.bib152)] introduced the MOREPAIR framework, this framework improve the performance of LLMs in automated program repair (APR) by simultaneously optimizing syntactic code transformations and the logical reasoning behind code changes, the study used techniques to enhance fine-tuning efficiency, such as QLoRA (Quantized Low-Rank Adaptation) [[153](https://arxiv.org/html/2408.02479v1#bib.bib153)] to reduce memory requirements and NEFTune (Noisy Embedding Fine-Tuning) [[154](https://arxiv.org/html/2408.02479v1#bib.bib154)] to prevent overfitting during the fine-tuning process. The experiments evaluated MOREPAIR on four open-source LLMs of different sizes and architectures (CodeLlama-13B, CodeLlama-7B, StarChat-alpha, and Mistral-7B) using two benchmarks, evalrepair-C++ and EvalRepair-Java. The results indicated that CodeLlama improved by 11% and 8% on the first 10 repair suggestions for evalrepair-C++ and EvalRepair-Java respectively. Another study introduced the PyDex system, which automatically repairs syntax and semantic errors in introductory Python programming assignments using LLMs, the system combines multimodal prompts and iterative querying methods to generate repair candidates and uses few-shot learning to improve repair accuracy. PyDex was evaluated on 286 real student programs from an introductory Python programming course and compared against three baselines. The results showed that PyDex significantly improved the repair rate and effectiveness compared to existing baselines [[155](https://arxiv.org/html/2408.02479v1#bib.bib155)].

[[156](https://arxiv.org/html/2408.02479v1#bib.bib156)] introduced a new system named RING that leverages large language models (LLMCs) to perform multilingual program repair across six programming languages. RING employs a prompt strategy that minimizes customization efforts, including three stages: fault localization, code transformation, and candidate ranking. The results showed that RING was particularly effective in Python, successfully repairing 94% of errors on the first attempt. The study also introduced a new PowerShell command repair dataset, providing valuable resources for the research community, this research demonstrated that AI-driven automation makes program repair more efficient and scalable. Another study, [[157](https://arxiv.org/html/2408.02479v1#bib.bib157)] conducted a comprehensive investigation into function-level automated program repair, introducing a new LLM-based APR technique called SRepair. SRepair utilizes a dual-LLM framework to enhance repair performance, the SRepair framework combines a repair suggestion model and a patch generation model. It uses chain-of-thought to generate natural language repair suggestions based on auxiliary repair-related information and then utilizes these suggestions to generate the repaired function. The results showed that SRepair outperformed existing APR techniques on the Defects4J dataset, repairing 300 single-function errors, with an improvement of at least 85% over previous techniques. This study demonstrated the effectiveness of the dual-LLM framework in function-level repair and, for the first time achieved multi-function error repair, highlighting the significant potential of LLMs in program repair. By extending the scope of APR, SRepair paves the way for applying LLMs in practical software development and evaluation.

#### IX-A3 Penetration Testing

LLMs can also be applied in the field of penetration testing, where they are used to enhance the efficiency and effectiveness of automated penetration testing. Although not as frequently studied as vulnerability detection and automated repair, this review includes two relevant papers. [[143](https://arxiv.org/html/2408.02479v1#bib.bib143)] investigates the development and evaluation of an LLM-driven automatic penetration testing tool PENTESTGPT. The main purpose of this study is to evaluate the performance of LLMs in practical penetration testing tasks and address the issue of context loss during the penetration testing process, the paper introduces three self-interaction modules of PENTESTGPT (reasoning, generation, and parsing) and provides empirical research based on benchmarks involving 13 targets and 182 sub-tasks. It compares the penetration testing performance of GPT-3.5, GPT-4, and Bard. The experimental results show that PENTESTGPT’s task completion rate is 228.6% higher than GPT-3.5 and 58.6% higher than GPT-4, this study demonstrates the potential of LLMs in automated penetration testing, helping to identify and resolve security vulnerabilities, thereby enhancing the security and robustness of software systems.

A similar research paper explores the application of generative AI in penetration testing. [[158](https://arxiv.org/html/2408.02479v1#bib.bib158)] evaluates the effectiveness, challenges, and potential consequences of using generative AI tools (specifically ChatGPT 3.5) in penetration testing. Through practical application experiments. The research conducts a five-stage penetration test (reconnaissance, scanning, vulnerability assessment, exploitation, and reporting) on a vulnerable machine from VulnHub, integrating Shell_GPT (sgpt) with ChatGPT’s API to automate guidance in the penetration testing process. The experimental results demonstrate that generative AI tools can significantly speed up the penetration testing process and provide accurate and useful commands, enhancing testing efficiency and effectiveness. This study indicates that the need to consider potential risks and unintended consequences, emphasizing the importance of responsible use and human oversight. Assessing the robustness of systems is also a crucial part of development, LLMs are used to develop and evaluate new testing frameworks to detect and improve the robustness of intelligent software systems. [[144](https://arxiv.org/html/2408.02479v1#bib.bib144)] introduces a robust input testing framework named RITFIS, designed to evaluate the robustness of LLM-based intelligent software against natural language inputs. The study adapts 17 existing DNN testing methods to LLM scenarios and empirically validates them on multiple datasets to highlight the current robustness deficiencies and limitations of LLM software. The study indicate that RITFIS effectively assesses the robustness of LLM software and reveals its vulnerabilities in handling complex natural language inputs. This research underscores the importance of robustness testing for LLM-based intelligent software and provides directions for improving testing methods to enhance reliability and security in practical applications.

### IX-B LLM-based Agents Task

LLM-based Agents primarily appied in areas such as autonomous decision-making, task-specific optimization, and multi-agent collaboration, these frameworks showcasing their strong potential in proactive defense.  [[159](https://arxiv.org/html/2408.02479v1#bib.bib159)] aims to address the limitations of existing debugging methods that treat the generated program as an indivisible entity. By segmenting the program into basic blocks and verifying the correctness of each block based on task descriptions, the proposed method LDB (Large Language Model Debugger) provide a more detailed and effective debugging tool that closely reflects human debugging practices. The study’s experiments covered testing LDB on several benchmarks and compared with baseline models without a debugger and those using traditional debugging methods (self-debugging with explanations and traces). LDB’s accuracy increased from a baseline of 73.8% to 82.9% on the HumanEval benchmark, an improvement of 9.1%. In the domain of vulnerability detection, researchers have enhanced detection accuracy by combining Role-Based Access Control (RBAC) practices with deep learning of complex code structures.

[[160](https://arxiv.org/html/2408.02479v1#bib.bib160)] addresses the challenge of automatically and appropriately repairing access control (AC) vulnerabilities in smart contracts. The innovation of this paper lies in combining mined RBAC practices with LLMs to create a context-aware repair framework for AC vulnerabilities. The model primarily uses GPT-4, enhanced by a new method called ACFIX, which mines common RBAC practices from existing smart contracts and employs a Multi-Agent Debate (MAD) mechanism to verify the generated patches through debates between generator and verifier agents to ensure correctness. Experimental results show that ACFIX successfully repaired 94.92% of access control vulnerabilities, significantly outperforming the baseline GPT-4’s 52.54%. Another application in smart contracts  [[161](https://arxiv.org/html/2408.02479v1#bib.bib161)], this paper introduces a two-stage adversarial framework, GPTLENS, which improves vulnerability detection accuracy through generation and discrimination phases. GPTLENS achieved a 76.9% success rate in detecting smart contract vulnerabilities, better than the 38.5% success rate of traditional methods. Another study, [[109](https://arxiv.org/html/2408.02479v1#bib.bib109)] investigates the use of GPT-4 to automatically exploit disclosed but unpatched vulnerabilities, the experiments showed that the LLM-based agent achieved an 87% success rate in exploiting vulnerabilities when provided with CVE descriptions. Finally another LLM-based agent application in the penetration test,  [[107](https://arxiv.org/html/2408.02479v1#bib.bib107)] employs GPT-3.5 to assist penetration testers by automating high-level task planning and low-level vulnerability discovery, thereby enhancing penetration testing capabilities. The experiments demonstrated successful automation of multiple stages of penetration testing, including high-level strategy formulation and low-level vulnerability discovery, showcasing the effectiveness of LLMs in penetration testing.

In the field of software repair by multi-agents collaborations, [[162](https://arxiv.org/html/2408.02479v1#bib.bib162)] proposes a dual-agent framework that enhances the automation and accuracy of repairing declarative specifications through iterative prompt optimization and multi-agent collaboration. The researcher compare the effectiveness of the LLM-based repair pipeline with several state-of-the-art Alloy APR techniques (ARepair, ICEBAR, BeAFix, and ATR). In the result, framework repaired 231 defects in the Alloy4Fun benchmark which surpassing the 278 defects repaired by traditional tools. In  [[142](https://arxiv.org/html/2408.02479v1#bib.bib142)], developed and evaluated an automated debugging framework named FixAgent, which improves fault localization, repair generation, and error analysis through an LLM-based multi-agent system. Although this research primarily focuses on automated debugging, incorporating elements like fault localization and automated program repair (APR), it intersects with test generation, particularly in the validation phase for testing bug fixes. The study evaluates FixAgent’s performance on the Codeflaws, QuixBugs, and ConDefects datasets, comparing it to 16 baseline methods, including state-of-the-art APR tools and LLMs. Experimental results show that FixAgent fixed 78 out of 79 bugs in the QuixBugs dataset, including 9 never-before-fixed bugs. In the Codeflaws dataset, FixAgent fixed 3982 out of 2780 defects, with a correctness rate of 96.5%. The framework includes specialized agents responsible for localization, repair, and analysis tasks and uses the rubber duck debugging principle. FixAgent demonstrates the powerful capabilities of LLMs in automated debugging, improving the performance of existing APR tools and LLMs which can be considered as state-of-art framework in APR by LLM-based agent.

[[46](https://arxiv.org/html/2408.02479v1#bib.bib46)] introduces an automated program repair agent named RepairAgent, this agent can dynamically generates prompts and integrates tools to automatically fix software bugs. This researcher also address the limitations of current LLM-based repair techniques, which typically involve fixed prompts or feedback loops that do not allow the model to gather comprehensive information about the bug or code. RepairAgent is a LLM-based agent designed to alternately collect information about the bug, gather repair ingredients, and validate the repairs, similar to how human developers fix bugs. RepairAgent achieved impressive result by overall repaired 186 bugs in the Defects4J benchmark, with 164 being correctly repaired outperforming existing repair techniques achieved the state-of-art performances.

In the realm of software security, researchers have combined LLM and security engineering models to improve security analysis and design processes. [[163](https://arxiv.org/html/2408.02479v1#bib.bib163)] aims to propose a complex hybrid strategy to ensure the reliability and security of software systems, this involves a concept-guided approach where LLM-based agents interact with system model diagrams to perform tasks related to safety analysis. [[108](https://arxiv.org/html/2408.02479v1#bib.bib108)] introduces the TrustLLM framework which increase the accuracy and interpretability of smart contract auditing by customizing LLM capabilities to the specific requirements of smart contract code. This paper conducts experiments on a balanced dataset comprising 1,734 positive samples and 1,810 negative samples, comparing TrustLLM with other models such as CodeBERT, GraphCodeBERT, and several versions of GPT and CodeLlama. TrustLLM achieves an F1 score of 91.21% and an accuracy of 91.11% which outperforming other models. Beyond software-level security design, LLMs can also be integrated into autonomous driving systems. [[164](https://arxiv.org/html/2408.02479v1#bib.bib164)] which has already been discussed in the [IV](https://arxiv.org/html/2408.02479v1#S4 "IV Requirement Engineering and and Documentation ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future").

### IX-C Analysis

Overall, the direction of LLM-based Agents represents significant innovative advancements in software security and maintenance, demonstrating improvements across all areas. LLM-based Agents, through multi-agent collaboration and runtime information tracking to help with debugging tasks, compared to traditional LLMs approaches are often rely on fixed prompts or feedback loops to debug a given code snippet or program. In vulnerability detection, LLM-based Agents combine RBAC practices and in-depth learning of complex code structures to improve the accuracy and efficiency of detecting vulnerabilities, traditional LLMs methods normally depend on extensive manual intervention and detailed guidance when handling tasks. LLM-based Agents also demonstrate effectiveness in penetration testing by automating high-level task planning and low-level vulnerability exploration, thereby enhancing penetration testing capabilities. In contrast, traditional LLM methods are more suited for passive detection and analysis, lacking proactive testing and defense capabilities.

![Refer to caption](img/d5d82e03ddbd51ca0dfb1a2279473121.png)

Figure 8: Illustration of Comparison Framework Between LLM-based Agent [[46](https://arxiv.org/html/2408.02479v1#bib.bib46)] and LLM [[152](https://arxiv.org/html/2408.02479v1#bib.bib152)] in Software Security and Maintenance

From the perspective of automation, LLM-based agents automate the detection and repair of software errors through multi-agent frameworks and dynamic analysis tools, improving the automation and accuracy of the repair process. Traditional LLMs methods also have a good performance on various maintenance or debug tasks, but often lack autonomous decision-making and dynamic adjustment capabilities during the repair process. In terms of software security, intelligent agent become more flexibly by combining LLM and security engineering models to improve security analysis and design processes, thereby enhancing the reliability and security of software systems. when dealing with security tasks by LLMs only, often rely on static analysis lacking adaptability and optimization capabilities. As shown in the Figure.[8](https://arxiv.org/html/2408.02479v1#S9.F8 "Figure 8 ‣ IX-C Analysis ‣ IX Software Security and Maintenance ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), the comparison using the MOREPAIR [[152](https://arxiv.org/html/2408.02479v1#bib.bib152)] for LLMs and RepairAgent [[46](https://arxiv.org/html/2408.02479v1#bib.bib46)] for LLM-based agents. The LLM framework utilize the optimization techniques (QLoRA, NEFTune) to generate repair advices, the RepairAgent utilize multiple tools during the inspection which facilitate the precision and accuracy of the analysis before the repair process, the idea is quite similar with ”reasoning before action”. Then the agent framework utilize state machine and LLM to refine continuously, and if failed during the repair process, the RepairAgent will enter the self-reflection phase to understand the reason autonomously.

Thus, from the review, we can say that LLM-based agents brings more autonomy and flexibility in the field of software security and maintenance. These improvements can enhance task execution efficiency and accuracy, also extend the application scope of LLMs in complex software engineering tasks, demonstrating their strong potential in proactive defense, complex task handling, and meeting high reliability requirements.

### IX-D Benchmarks

When analyzing the benchmarks used in LLM literature, several public datasets stand out due to their frequent use and presence across different application scenarios. Datasets such as Defects4J, Codeflaws, QuixBugs, and the Common Vulnerability and Exposure (CVE) database are commonly employed in the domains of vulnerability detection and software security. For instance, Defects4J is widely used in papers like [[46](https://arxiv.org/html/2408.02479v1#bib.bib46)] and [[159](https://arxiv.org/html/2408.02479v1#bib.bib159)] to evaluate automated program repair tools. Similarly, Codeflaws and QuixBugs are used in papers like [[142](https://arxiv.org/html/2408.02479v1#bib.bib142)] to test debugging capabilities, focusing on smaller algorithmic problems typically found in competitive programming and educational settings. These datasets effectively measure the ability of LLMs to detect vulnerabilities and modify code in specific code blocks.

CVE is a critical benchmark for evaluating the security capabilities of LLMs, offering a repository of known vulnerabilities that allow LLMs to assess their ability to autonomously detect and exploit security flaws, bridging the gap between theoretical research and practical cybersecurity applications. Another notable dataset is ARepair, used in [[162](https://arxiv.org/html/2408.02479v1#bib.bib162)]. This dataset consists of defective specifications and tests the ability of LLMs to understand and repair formal specifications. More common datasets like HumanEval and MBPP are also frequently used to evaluate the functional correctness of code generated by LLMs. Similarly, Alloy4Fun is used to test the repair of declarative specifications in Alloy framework [[162](https://arxiv.org/html/2408.02479v1#bib.bib162)], reflecting the LLM’s performance in understanding and fixing logical errors in formal languages.

Specialized datasets such as VulnHub and HackTheBox are used to evaluate the penetration testing capabilities of LLMs. Papers like [[107](https://arxiv.org/html/2408.02479v1#bib.bib107)] utilize these environments to simulate real-world hacking scenarios, thereby assessing the practical applications of LLMs in cybersecurity. These benchmarks are crucial for evaluating the real-world efficacy of LLM-based agents in cyber-security environments, bridging the gap between theoretical capabilities and practical applications. In the context of smart contract security, datasets extracted from Etherscan and those compiled for tools like SmartFix provide benchmarks for evaluating LLMs’ ability to identify and fix vulnerabilities in blockchain-based applications, emphasizing the reliability and security of decentralized applications.

When comparing the benchmarks used in LLM and LLM-based agent research, several key similarities and differences emerge. Both approaches frequently use datasets like Defects4J, CVE, and HumanEval, highlighting their foundational role in evaluating software engineering tasks. However, LLM-based agent research often combines these datasets with specialized benchmarks like VulnHub and HackTheBox to test more dynamic and interactive capabilities, especially in the context of cybersecurity. LLM-based agent research typically focuses more on real-time autonomous decision-making and action, reflected in their choice of benchmarks. These datasets test not only the agents’ knowledge but also their ability to autonomously apply this knowledge in real-world scenarios. This contrasts with traditional LLMs research, which typically focuses on static tasks like vulnerability repair and code generation without requiring real-time interaction and further changes or decision-making. Moreover, the use of specialized benchmarks like the smart contract datasets from Etherscan in LLM-based agent research underscores the importance of blockchain technology and the need for robust security measures in decentralized applications, this trend highlights the adaptability and diversity of LLM-based agents in addressing emerging challenges in software security and maintenance. This distinction reflects the broader and more interactive application scenarios of LLM-based agents, also the public dataset may not suitable for LLM-based agent in particular structure designed, so there are a lot of self-collected benchmark emerged which provide more flexibility.

### IX-E Evaluation Metrics

The evaluation metrics for the llm in the software security and maintenance are quite diverse. Researchers need to consider various factors such as coverage, efficiency, and reliability of the model or framework. Evaluation Metrics like success rate and pass rate are directly related to the performance of LLMs in different scenarios. In Table [X](https://arxiv.org/html/2408.02479v1#S9.T10 "TABLE X ‣ IX-E Evaluation Metrics ‣ IX Software Security and Maintenance ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), common standards such as success rate and change rate are frequently used to evaluate the robustness of models when faced with diverse inputs. Time overhead and query number are used to assess the efficiency and resource consumption of models when performing specific tasks. Additionally, ROC AUC, F1 score, and accuracy are important for evaluating the model’s ability to identify vulnerabilities, especially in binary classification tasks. In code repair tasks, metrics such as compilability and plausibility are very common, these metrics ensure that the generated solutions are correct and deployable. Common standards like BLEU and CodeBLEU are used to evaluate the quality and human-likeness of generated code, which helps determine if the model’s capabilities and performance are comparable to human performance. Furthermore, domain-specific metrics like tree edit distance and test pass rate are used to evaluate the effectiveness of LLM applications in specialized fields of software engineering, these metrics are used to address the limitations posed by software security and maintenance. In contrast, while LLM-based agents use evaluation metrics similar to those used for LLMs, such as success rate, they also incorporate more subjective metrics for evaluation. These include appropriateness, relevance, and adequacy, which are human-judged standards. Overall, the evaluation metrics used by agents tend to be simpler and easier to understand than those used for LLMs. This is likely because agents handle high-level tasks, such as the success rate of generating potential vulnerabilities and the frequency of agents calling external tools, so they also need to consider computational and time overheads of the overall architecture.

By comparing these metrics, we can see that LLMs emphasising the success rate of individual testing methods, LLM-based agents focus more on the overall task completion time/cost/effectiveness. LLMs typically use binary classification metrics like ROC, AUC, and F1 score, while agents tend to emphasize the success rate and accuracy during both the generation and validation phases, providing a comprehensive evaluation. For the time cost and performance, LLMs mainly focus on the execution time of testing methods and the number of queries to assess their efficiency. In contrast, LLM-based agents focus more on the completion time of repair tasks and the number of API calls, it will make sure the efficiency and practicality of overall architecture.

TABLE X: Evaluation Metrics in Software Security and Maintenance

| Reference Paper | Benchmarks | Evaluation Metrics | Agent |
| --- | --- | --- | --- |
|  

&#124; [[144](https://arxiv.org/html/2408.02479v1#bib.bib144)] &#124;

 |  

&#124; Financial Sentiment Analysis &#124;
&#124; Movie Review Analysis &#124;
&#124; News Classification &#124;

 |  

&#124; Success Rate, Change Rate, Perplexity, Time Overhead, Query Number &#124;

 | No |
|  

&#124; [[146](https://arxiv.org/html/2408.02479v1#bib.bib146)] &#124;

 |  

&#124; CVEfixes &#124;
&#124; Manually-Curated Dataset &#124;
&#124; (624 vulnerabilities across &#124;
&#124; 205 Java projects) &#124;
&#124; VCMatch &#124;
&#124; (10 popular repositories) &#124;

 |  

&#124; ROC AUC, F1 Score, Accuracy, Optimal Classification, Threshold &#124;

 | No |
|  

&#124; [[149](https://arxiv.org/html/2408.02479v1#bib.bib149)] &#124;

 |  

&#124; CVEfixes &#124;
&#124; Manually-Curated Dataset &#124;
&#124; VCMatch &#124;

 |  

&#124; Precision, Recall &#124;

 | No |
| [[143](https://arxiv.org/html/2408.02479v1#bib.bib143)] |  

&#124; HackTheBox &#124;
&#124; VulnHub &#124;

 |  

&#124; Overall Task Completion, Sub-task Completion, Task Variety, Challenge &#124;
&#124; Levels, Progress Tracking &#124;

 | No |
|  

&#124; [[151](https://arxiv.org/html/2408.02479v1#bib.bib151)] &#124;

 |  

&#124; Defects4J v1.2 &#124;
&#124; Defects4J v2.0 &#124;
&#124; QuixBugs &#124;
&#124; HumanEval-Java &#124;

 |  

&#124; Compilability, Plausibility, Test pass rate, Exact Match, BLEU &#124;

 | No |
| [[148](https://arxiv.org/html/2408.02479v1#bib.bib148)] |  

&#124; Devign &#124;
&#124; Reveal &#124;
&#124; Big-Vul &#124;

 |  

&#124; F1 score, Accuracy, Precision,Recall. &#124;

 | No |
|  

&#124; [[150](https://arxiv.org/html/2408.02479v1#bib.bib150)] &#124;

 |  

&#124; PRIMEVUL &#124;
&#124; BigVul &#124;

 |  

&#124; F1 score, Accuracy, Precision, Recall, VD-S, Pair-wise, evaluation metrics &#124;

 | No |
|  

&#124; [[147](https://arxiv.org/html/2408.02479v1#bib.bib147)] &#124;

 |  

&#124; Customized GitHub dataset &#124;
&#124; (308 binary classification and &#124;
&#124; 120 multi-label classification) &#124;

 |  

&#124; Precision, Recall, F1-Score, AUC, Accuracy &#124;

 | No |
|  

&#124; [[158](https://arxiv.org/html/2408.02479v1#bib.bib158)] &#124;

 |  

&#124; VulnHub &#124;

 |  

&#124; Output’s Description &#124;

 | No |
|  

&#124; [[165](https://arxiv.org/html/2408.02479v1#bib.bib165)] &#124;

 |  

&#124; VulDeePecker &#124;
&#124; SeVC &#124;

 |  

&#124; False Positive Rate, False Negative Rate, Precision, Recall, F1-score &#124;

 | No |
| [[145](https://arxiv.org/html/2408.02479v1#bib.bib145)] | CVEFixes |  

&#124; CodeBLEU, Tree Edit Distance, Pass@k &#124;

 | No |
|  

&#124; [[152](https://arxiv.org/html/2408.02479v1#bib.bib152)] &#124;

 |  

&#124; EvalRepair-C++ &#124;
&#124; EvalRepair-Java &#124;

 |  

&#124; TOP-5 and TOP-10, Repair &#124;

 | No |
| [[155](https://arxiv.org/html/2408.02479v1#bib.bib155)] |  

&#124; Introductory Python &#124;
&#124; Assignments Dataset &#124;

 |  

&#124; Repair Rate, Token Edit Distance &#124;

 | No |
| [[156](https://arxiv.org/html/2408.02479v1#bib.bib156)] |  

&#124; Multi-languages dataset &#124;
&#124; (Excel,Power Fx,Python, &#124;
&#124; JavaScript,C andPowerShell) &#124;

 | Exact Matches | No |
|  

&#124; [[157](https://arxiv.org/html/2408.02479v1#bib.bib157)] &#124;

 | Defects4J 1.2 and 2.0 |  

&#124; Plausible Patches, Correct Fix &#124;

 | No |
|  

&#124; [[107](https://arxiv.org/html/2408.02479v1#bib.bib107)] &#124;

 |  

&#124; Vulnerable Virtual Machine &#124;
&#124; (lin.security Linux VM) &#124;

 | Success Rate | Yes |
|  

&#124; [[160](https://arxiv.org/html/2408.02479v1#bib.bib160)] &#124;

 | 118 AC Vulnerabilities |  

&#124; Success Rate, Exploitation based evaluation, Manual inspection of the &#124;
&#124; patches. &#124;

 | Yes |
|  

&#124; [[161](https://arxiv.org/html/2408.02479v1#bib.bib161)] &#124;

 | 13 Smart Contract Bugs |  

&#124; Success Rate, Contract level, Trial level &#124;

 | Yes |
|  

&#124; [[142](https://arxiv.org/html/2408.02479v1#bib.bib142)] &#124;

 |  

&#124; Codeflaws,QuixBugs, &#124;
&#124; ConDefects &#124;

 |  

&#124; Number of correctly fixed bugs, Number of plausibly patched bugs, &#124;
&#124; Correctness rate of generated patches &#124;

 | Yes |
|  

&#124; [[162](https://arxiv.org/html/2408.02479v1#bib.bib162)] &#124;

 | ARepair,Alloy4Fun |  

&#124; Correct@6, Runtime and Token Usage &#124;

 | Yes |
|  

&#124; [[163](https://arxiv.org/html/2408.02479v1#bib.bib163)] &#124;

 | System Model Graph |  

&#124; Accuracy, Effectiveness, Appropriateness &#124;

 | Yes |
|  

&#124; [[108](https://arxiv.org/html/2408.02479v1#bib.bib108)] &#124;

 |  

&#124; 1734 Positive Samples, &#124;
&#124; 1810 Negative Samples &#124;

 |  

&#124; F1 score, Accuracy, Consistency &#124;

 | Yes |
|  

&#124; [[166](https://arxiv.org/html/2408.02479v1#bib.bib166)] &#124;

 |  

&#124; HumanEval,MBPP, &#124;
&#124; TransCoder &#124;

 |  

&#124; Accuracy, Pass@1 &#124;

 | Yes |
|  

&#124; [[139](https://arxiv.org/html/2408.02479v1#bib.bib139)] &#124;

 |  

&#124; 13 Android Applications &#124;
&#124; from GitHub &#124;

 |  

&#124; Recall, Precision, Correct, Over-fitting, Correct@k &#124;

 | Yes |
|  

&#124; [[164](https://arxiv.org/html/2408.02479v1#bib.bib164)] &#124;

 | System Model Graph |  

&#124; Relevance, Adequacy &#124;

 | Yes |
|  

&#124; [[109](https://arxiv.org/html/2408.02479v1#bib.bib109)] &#124;

 | 15 Vulnerabilities from CVE Lib | Success Rate | Yes |
|  

&#124; [[46](https://arxiv.org/html/2408.02479v1#bib.bib46)] &#124;

 | Defects4J |  

&#124; Plausible Fixes, Correct Fixes &#124;

 | Yes |

## X Discussion

### X-A Experiment Models

In section 3-8, we reviewed and introduced the research on LLMs and LLM-based agent applications in software engineering in recent years. These studies have different research directions and we divided them into six subtopics for classification and discussion. With the advancement of large language models, thousands of models have appeared in the public eye, in order to more intuitively understand the application of large language models in various fields and the use of large language models as the core of intelligent agents, we summarized a total of 117 papers, mainly to discuss the frequency of use of LLMs in the field of software engineering.

Based on the review of 117 papers, our primary focus is on the models or frameworks utilized by the authors in their experiments. This is due to the fact that these papers often include tests of model performance in specific domains, such as evaluating LLaMA’s performance in code generation. Therefore, during our data collection process, we also included models used for comparison purposes, as these models often represent the state-of-the-art capabilities in their respective fields at the time of the study. In summary, across the 117 papers, we identified a total of 79 unique large language models. We visualized the frequency of these model names in a word cloud for a more intuitive representation, as shown in Figure.[9](https://arxiv.org/html/2408.02479v1#S10.F9 "Figure 9 ‣ X-A Experiment Models ‣ X Discussion ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). From the figure, we can observe that models such as GPT-3.5, GPT-4, LLaMA2, and Codex are frequently used. Although close source LLMs cannot be locally deployed or further trained, their exceptional capabilities make them a popular choice for comparison in experiments or for data augmentation, where GPT-4 is used to generate additional data to support the research model frameworks. For instance, researchers might use OpenAI’s API to generate initial text and then employ locally deployed models for further processing and optimization [[76](https://arxiv.org/html/2408.02479v1#bib.bib76)] [[122](https://arxiv.org/html/2408.02479v1#bib.bib122)] [[119](https://arxiv.org/html/2408.02479v1#bib.bib119)] [[113](https://arxiv.org/html/2408.02479v1#bib.bib113)].

Therefore, it is not difficult to see that the use of general large models with superior performance to assist development or as a measurement standard has been increasingly used in the vertical field of software engineering in the past two years. In addition, for some fields that have never been touched by LLMs before, many researchers first refer to the model ChatGPT and conduct various performance experiments on the newer GPT-4 [[55](https://arxiv.org/html/2408.02479v1#bib.bib55)] [[58](https://arxiv.org/html/2408.02479v1#bib.bib58)] [[64](https://arxiv.org/html/2408.02479v1#bib.bib64)]. Those models can be integrated into larger systems and combining with other machine learning models and tools, these models can be used to generate natural language responses, while another model handles intent recognition and dialogue management.

![Refer to caption](img/04dd103536e8134d42984228c05cb9de.png)

Figure 9: Experiment Models Usage WordCloud

Although the word cloud provides a rough overview of model usage frequency, it lacks detailed information. To gain deeper insights, we combined a grouped bar chart and a stacked bar chart to further analyze the usage of models in studies across different subtopics. The corresponding bar charts are presented in Figure.[10](https://arxiv.org/html/2408.02479v1#S10.F10 "Figure 10 ‣ X-A Experiment Models ‣ X Discussion ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"). During the analysis, we found that a large number of models appeared only once. Including these in the bar chart would have made the overall representation cluttered. Therefore, we excluded models that appeared only once and focused on the versatility of the remaining models. On the left side of each subtopic, we depict the models used in LLM-related studies, with the models used in LLM-based agent-related studies highlighted in red-bordered bars. From the figure, it is evident that in the Autonomous Learning and Decision Making subtopic, the number of models used in LLM-based agent-related studies is quite high. Specifically, GPT-4 and GPT-3.5 were used in 10 out of 18 papers and 15 out of 18 papers, respectively. In this subtopic, studies commonly utilized GPT-3.5/4 and LLaMA-2 for research and evaluation. During our analysis, we found that many studies on LLM-based agents evaluated the agents’ ability to mimic human behavior and decision-making or perform some reasoning tasks [[103](https://arxiv.org/html/2408.02479v1#bib.bib103)] [[111](https://arxiv.org/html/2408.02479v1#bib.bib111)] [[108](https://arxiv.org/html/2408.02479v1#bib.bib108)]. Since these studies do not require local deployment, they mainly assess the performance of state-of-the-art models in specific directions, leading to the frequent use of the GPT-family models. Frameworks like [[98](https://arxiv.org/html/2408.02479v1#bib.bib98)] [[36](https://arxiv.org/html/2408.02479v1#bib.bib36)] constructed LLM-based agents by calling the GPT-4 API, using verbal reinforcement to help language agents learn from their mistakes. Due to the limitations of GPT models, many studies also used LLaMA as the LLM for agents, fine-tuning it on the generated datasets to evaluate the emergence of knowledge and capabilities. Overall, we found that in the Autonomous Learning and Decision Making subtopic, LLM-based agents often use multiple models for testing and performance evaluation in a single task, this results in a significantly higher model usage frequency in this topic compared to others.

![Refer to caption](img/31cd56e4ec30636b36d7b2cb758d98e4.png)

Figure 10: Experiment Models Usage in Different Subtopics (REQ DENOTES ”Requirement Engineering and Documentation”, CODE DENOTES ”Code Generation and Software Development”, AUTO DENOTES ”Autonomous Learning and Decision Making”, DES DENOTES ”Software Design and Decision Making”, SEC DENOTES ”Software Security and Maintenance”)

Not only in the Autonomous Learning and Decision Making subtopic, but also across other themes, we observe that the variety of models (represented by the number of colors) used by LLM-based agents is relatively limited. For instance, in the requirement engineering and documentation subtopic, only GPT-3.5 and GPT-4 models were involved in the experiments. To analyze the reasons behind this phenomenon, we need to exclude the factors that models appearing only once were not considered and that there are inherently fewer studies on intelligent agents. We believe this primarily reflects the integration relationship between the agents and the large language models. The combination of these two technologies aims to address the limitations of large language models in specific tasks or aspects. Intelligent agents allow researchers to design a more flexible framework and incorporate the large language model into it. These models, having been trained on vast amounts of data, possess strong generalizability, making them suitable for a wide range of tasks and domains.

Therefore, researchers and developers can use the same model to address multiple issues, reducing the need for various models. In code generation [[83](https://arxiv.org/html/2408.02479v1#bib.bib83)] [[79](https://arxiv.org/html/2408.02479v1#bib.bib79)], test case generation [[140](https://arxiv.org/html/2408.02479v1#bib.bib140)] [[142](https://arxiv.org/html/2408.02479v1#bib.bib142)], and software security [[167](https://arxiv.org/html/2408.02479v1#bib.bib167)] [[159](https://arxiv.org/html/2408.02479v1#bib.bib159)], there are instances of using CodeLlama. This model is fine-tuned and optimized based on the LLaMA architecture. At its release, it was considered one of the state-of-the-art models for code generation and understanding tasks, showing strong performance and potential compared to other models like Codex. Another potential reason is the previous successful applications and research outcomes that have proven these models’ effectiveness, further enhancing researchers’ trust and reliance on them. Compared to models that perform well in specific domains, in intelligent agent development, there is a preference for using general-purpose large models to ensure that the core of the agent possesses excellent text comprehension abilities, allowing for further reasoning, planning, and task execution. From the Figure.[10](https://arxiv.org/html/2408.02479v1#S10.F10 "Figure 10 ‣ X-A Experiment Models ‣ X Discussion ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), we can also observe that research in the code generation and software development fields adopts a wide variety of models, further indicating the extensive attention this area receives and the excellent performance of models in code generation task.

![Refer to caption](img/ebd92f5a20ecb18d5e95973150d7a0f0.png)

Figure 11: Distribution of LLMs and Agents across six topics

|  | CODE | REQ | AUTO | DESIGN | SEC | TEST |
| --- | --- | --- | --- | --- | --- | --- |
| CODE | X | 1 | 0 | 2 | 3 | 1 |
| REQ | 1 | X | 1 | 0 | 2 | 0 |
| AUTO | 0 | 1 | X | 6 | 5 | 1 |
| DESIGN | 2 | 0 | 6 | X | 1 | 0 |
| SEC | 3 | 2 | 5 | 1 | X | 2 |
| TEST | 1 | 0 | 1 | 0 | 2 | X |

TABLE XI: Overlap of Papers Among Different Topics (REQ DENOTES ”Requirement Engineering and Documentation”, CODE DENOTES ”Code Generation and Software Development”, AUTO DENOTES ”Autonomous Learning and Decision Making”, DES DENOTES ”Software Design and Decision Making”, SEC DENOTES ”Software Security and Maintenance”)

### X-B Topics Overlapping

Figure.[11](https://arxiv.org/html/2408.02479v1#S10.F11 "Figure 11 ‣ X-A Experiment Models ‣ X Discussion ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") shows the distribution of all collected literature across six themes. For LLM-type literature, the theme of software security and maintenance accounts for nearly 30%, whereas test case generation accounts for less than 10%. This trend is similarly reflected in the LLM-based agent literature. Research on using LLM-based agents to address requirements engineering and test case generation is relatively sparse. Requirements engineering is a new endeavor for LLM-based agents, and using the entire agent framework to generate test cases might be considered excessive. Therefore, more research tends to evaluate and explore the changes LLMs bring within the agent framework, such as autonomous decision-making abilities and capabilities in software maintenance and repair.

Table‘[XI](https://arxiv.org/html/2408.02479v1#S10.T11 "TABLE XI ‣ X-A Experiment Models ‣ X Discussion ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future") presents the number of papers spanning multiple themes. For instance, five papers can be classified under both software security and maintenance and autonomous learning and decision making. These two themes also overlap the most with other themes, indicating that LLMs and LLM-based agent research is broad and these tasks often require integrating knowledge and techniques from various fields such as code generation, design, and testing. The significant overlap reflects the close interrelation between these themes and other areas. For example, autonomous learning and decision-making often involve the model’s ability to autonomously learn and optimize decision trees, techniques that are applied in many specific software engineering tasks. Similarly, software security and maintenance typically require a combination of multiple techniques to enhance security, such as automatic code generation tools and automated testing frameworks [[71](https://arxiv.org/html/2408.02479v1#bib.bib71)] [[80](https://arxiv.org/html/2408.02479v1#bib.bib80)] [[83](https://arxiv.org/html/2408.02479v1#bib.bib83)] [[102](https://arxiv.org/html/2408.02479v1#bib.bib102)]. The overlap in literature highlights the increasing need for integrating methods and techniques from different research areas within software engineering. For instance, ensuring software security relies not only on security measures but also on leveraging code generation, automated testing, and design optimization technologies. Similarly, autonomous learning and decision-making require a comprehensive consideration of requirements engineering, code generation, and system design. Moreover, it suggests that certain technologies and methods possess strong commonality. For instance, LLM-based agents enhance capabilities in code generation, test automation, and security analysis through autonomous learning and decision-making. This sharing of technologies promotes knowledge exchange and technological dissemination across various fields within software engineering.

### X-C Benchmarks and Metrics

As shown in Figure.[12](https://arxiv.org/html/2408.02479v1#A0.F12 "Figure 12 ‣ -A Benchmarks ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), it include the distribution of common benchmarks across six topics. In reality, the number of benchmark datasets used is far greater than what is shown in the figure. Different software engineering tasks use various benchmark datasets for evaluation and testing. For instance, in requirements engineering, researchers often collect user stories or requirement specifications as datasets [[55](https://arxiv.org/html/2408.02479v1#bib.bib55)] [[63](https://arxiv.org/html/2408.02479v1#bib.bib63)], and these datasets are not well-known public datasets, so they were not included in the statistics. Alternatively, some studies specify their datasets as “Customized GitHub datasets” [[168](https://arxiv.org/html/2408.02479v1#bib.bib168)]. Therefore, the benchmark datasets shown in the figure represent commonly used public datasets. For example, MBPP and HumanEval, which have been introduced in previous sections, are frequently used. We can also observe that the datasets used in LLM and LLM-based agents tasks, apart from common public datasets, are different.

For instance, the FEVER⁷⁷7[https://fever.ai/dataset/fever.html](https://fever.ai/dataset/fever.html) dataset is often used in agent-related research. In [[35](https://arxiv.org/html/2408.02479v1#bib.bib35)], the FEVER dataset is used to test the ExpeL agent’s performance in fact verification tasks. Similarly, the HotpotQA⁸⁸8[https://hotpotqa.github.io/](https://hotpotqa.github.io/) dataset is frequently used in agent-related research for knowledge-intensive reasoning and question-answering tasks. When handling vulnerability repair tasks, LLMs often use the Defects4J⁹⁹9[https://github.com/rjust/defects4j](https://github.com/rjust/defects4j) benchmark dataset. This dataset contains 835 real-world defects from multiple open-source Java projects, categorized into buggy versions and repaired versions, typically used to evaluate the effectiveness of automated program repair techniques. Despite its extensive use in LLM research, Defects4J is relatively less used in LLM-based agents research. We speculate that this may be because Defects4J primarily evaluates single code repair tasks, which do not fully align with the multi-task and real-time requirements of LLM-based agents. Additionally, new datasets like ConDefects have been introduced [[142](https://arxiv.org/html/2408.02479v1#bib.bib142)], focusing on addressing data leakage issues and providing more comprehensive defect localization and repair evaluations.

As shown in Figure.[13](https://arxiv.org/html/2408.02479v1#A0.F13 "Figure 13 ‣ -B Evaluation Metrics ‣ From LLMs to LLM-based Agents for Software Engineering: A Survey of Current, Challenges and Future"), it includes the top ten evaluation metrics for LLMs and LLM-based agents. The analysis reveals that the evaluation methods used by both are almost identical. In previous sections, we also discussed that for agents, it is necessary to consider time and computational resource consumption, which is evident from the pie chart. Meanwhile, many studies focus on the code generation capabilities of LLMs, so more evaluation metrics pertain to the correctness and Exact Match of the generated code [[73](https://arxiv.org/html/2408.02479v1#bib.bib73)] [[69](https://arxiv.org/html/2408.02479v1#bib.bib69)] [[30](https://arxiv.org/html/2408.02479v1#bib.bib30)], but overall, the evaluation metrics for LLMs and LLM-based agents in software engineering applications are quite similar.

## XI Conclusion

In this paper, we conducted a comprehensive literature review on the application of LLM and LLM-based agents in software engineering. We categorized software engineering into six topics: requirement engineering and documentation, code generation and software development, autonomous learning and decision making, software design and evaluation, software test generation, and software security and maintenance. For each topic, we analyzed the tasks, benchmarks, and evaluation metrics, distinguishing between LLM and LLM-based agents and discussing the differences and impacts they bring. We further analyzed and discussed the models used in the experiments of the 117 collected papers. Additionally, we provided statistics and distinctions between LLM and LLM-based agents regarding datasets and evaluation metrics. The analysis revealed that the emergence of LLM-based agents has led to extensive research and applications across various software engineering topics, demonstrating different emphases compared to traditional LLMs in terms of tasks, benchmarks, and evaluation metrics.

## References

*   [1] S. Wang, D. Chollak, D. Movshovitz-Attias, and L. Tan, “Bugram: bug detection with n-gram language models,” in Proceedings of the 31st IEEE/ACM International Conference on Automated Software Engineering, pp. 724–735, 2016.
*   [2] A. Vogelsang and M. Borg, “Requirements engineering for machine learning: Perspectives from data scientists,” in 2019 IEEE 27th International Requirements Engineering Conference Workshops (REW), (Jeju, Korea (South)), pp. 245–251, 2019.
*   [3] “Chatgpt: Optimizing language models for dialogue,” 11 2022. [Online; accessed 17-July-2024].
*   [4] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. de Oliveira Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, A. Ray, R. Puri, G. Krueger, M. Petrov, H. Khlaaf, G. Sastry, P. Mishkin, B. Chan, S. Gray, N. Ryder, M. Pavlov, A. Power, L. Kaiser, M. Bavarian, C. Winter, P. Tillet, F. P. Such, D. Cummings, M. Plappert, F. Chantzis, E. Barnes, A. Herbert-Voss, W. H. Guss, A. Nichol, A. Paino, N. Tezak, J. Tang, I. Babuschkin, S. Balaji, S. Jain, W. Saunders, C. Hesse, A. N. Carr, J. Leike, J. Achiam, V. Misra, E. Morikawa, A. Radford, M. Knight, M. Brundage, M. Murati, K. Mayer, P. Welinder, B. McGrew, D. Amodei, S. McCandlish, I. Sutskever, and W. Zaremba, “Evaluating large language models trained on code,” arXiv preprint arXiv:2107.03374, 2021. arXiv:2107.03374 [cs.LG].
*   [5] N. Jain, S. Vaidyanath, A. Iyer, N. Natarajan, S. Parthasarathy, S. Rajamani, and R. Sharma, “Jigsaw: large language models meet program synthesis,” in Proceedings of the 44th International Conference on Software Engineering, ICSE ’22, (New York, NY, USA), p. 1219–1231, Association for Computing Machinery, 2022.
*   [6] T. Li, G. Zhang, Q. D. Do, X. Yue, and W. Chen, “Long-context llms struggle with long in-context learning,” 2024.
*   [7] J. Yang, H. Jin, R. Tang, X. Han, Q. Feng, H. Jiang, S. Zhong, B. Yin, and X. Hu, “Harnessing the power of llms in practice: A survey on chatgpt and beyond,” ACM Trans. Knowl. Discov. Data, vol. 18, apr 2024.
*   [8] A. Fan, B. Gokkaya, M. Harman, M. Lyubarskiy, S. Sengupta, S. Yoo, and J. M. Zhang, “Large language models for software engineering: Survey and open problems,” in 2023 IEEE/ACM International Conference on Software Engineering: Future of Software Engineering (ICSE-FoSE), pp. 31–53, 2023.
*   [9] L. Wang, C. Ma, X. Feng, Z. Zhang, H. Yang, J. Zhang, Z. Chen, J. Tang, X. Chen, Y. Lin, W. X. Zhao, Z. Wei, and J. Wen, “A survey on large language model based autonomous agents,” Frontiers of Computer Science, vol. 18, no. 6, pp. 186345–, 2024.
*   [10] Z. Xi, W. Chen, X. Guo, W. He, Y. Ding, B. Hong, M. Zhang, J. Wang, S. Jin, E. Zhou, R. Zheng, X. Fan, X. Wang, L. Xiong, Y. Zhou, W. Wang, C. Jiang, Y. Zou, X. Liu, Z. Yin, S. Dou, R. Weng, W. Cheng, Q. Zhang, W. Qin, Y. Zheng, X. Qiu, X. Huang, and T. Gui, “The rise and potential of large language model based agents: A survey,” 2023.
*   [11] P. Lewis, E. Perez, A. Piktus, F. Petroni, V. Karpukhin, N. Goyal, H. Küttler, M. Lewis, W.-t. Yih, T. Rocktäschel, S. Riedel, and D. Kiela, “Retrieval-augmented generation for knowledge-intensive nlp tasks,” in Advances in Neural Information Processing Systems (H. Larochelle, M. Ranzato, R. Hadsell, M. Balcan, and H. Lin, eds.), vol. 33, pp. 9459–9474, Curran Associates, Inc., 2020.
*   [12] GitHub, Inc., “Github copilot: Your ai pair programmer.” [https://github.com/features/copilot](https://github.com/features/copilot), 2024. [Online; accessed 17-July-2024].
*   [13] S. Russell and P. Norvig, Artificial Intelligence: A Modern Approach. Pearson Education Limited, 2016.
*   [14] N. R. Jennings, “A survey of agent-oriented software engineering,” Knowledge Engineering Review, vol. 15, no. 4, pp. 215–249, 2000.
*   [15] Y. Bisk, A. Holtzman, J. Thomason, J. Andreas, Y. Bengio, J. Chai, M. Lapata, A. Lazaridou, J. May, A. Nisnevich, N. Pinto, and J. Turian, “Experience grounds language,” 2020.
*   [16] J. Wei, X. Wang, D. Schuurmans, M. Bosma, F. Xia, E. Chi, Q. V. Le, D. Zhou, et al., “Chain-of-thought prompting elicits reasoning in large language models,” Advances in neural information processing systems, vol. 35, pp. 24824–24837, 2022.
*   [17] X. Hou, Y. Zhao, Y. Liu, Z. Yang, K. Wang, L. Li, X. Luo, D. Lo, J. Grundy, and H. Wang, “Large language models for software engineering: A systematic literature review,” 2024.
*   [18] Z. Zheng, K. Ning, J. Chen, Y. Wang, W. Chen, L. Guo, and W. Wang, “Towards an understanding of large language models in software engineering tasks,” 2023.
*   [19] A. Nguyen-Duc, B. Cabrero-Daniel, A. Przybylek, C. Arora, D. Khanna, T. Herda, U. Rafiq, J. Melegati, E. Guerra, K.-K. Kemell, M. Saari, Z. Zhang, H. Le, T. Quan, and P. Abrahamsson, “Generative artificial intelligence for software engineering – a research agenda,” 2023.
*   [20] W. Ma, S. Liu, Z. Lin, W. Wang, Q. Hu, Y. Liu, C. Zhang, L. Nie, L. Li, and Y. Liu, “Lms: Understanding code syntax and semantics for code analysis,” 2024.
*   [21] Z. Yang, Z. Sun, T. Z. Yue, P. Devanbu, and D. Lo, “Robustness, security, privacy, explainability, efficiency, and usability of large language models for code,” 2024.
*   [22] Y. Huang, Y. Chen, X. Chen, J. Chen, R. Peng, Z. Tang, J. Huang, F. Xu, and Z. Zheng, “Generative software engineering,” 2024.
*   [23] C. Manning and H. Schutze, Foundations of statistical natural language processing. MIT press, 1999.
*   [24] S. Hochreiter and J. Schmidhuber, “Long short-term memory,” Neural Computation, vol. 9, no. 8, pp. 1735–1780, 1997.
*   [25] S. Hochreiter and J. Schmidhuber, “Long short-term memory,” Neural computation, vol. 9, no. 8, pp. 1735–1780, 1997.
*   [26] A. Vaswani, N. Shazeer, N. Parmar, J. Uszkoreit, L. Jones, A. N. Gomez, Ł. Kaiser, and I. Polosukhin, “Attention is all you need,” Advances in neural information processing systems, vol. 30, 2017.
*   [27] L. Floridi and M. Chiriatti, “Gpt-3: Its nature, scope, limits, and consequences,” Minds and Machines, vol. 30, pp. 681–694, 2020.
*   [28] A. Chowdhery, S. Narang, J. Devlin, M. Bosma, G. Mishra, A. Roberts, P. Barham, H. W. Chung, C. Sutton, S. Gehrmann, et al., “Palm: Scaling language modeling with pathways,” Journal of Machine Learning Research, vol. 24, no. 240, pp. 1–113, 2023.
*   [29] S. Zhang, S. Roller, N. Goyal, M. Artetxe, M. Chen, S. Chen, C. Dewan, M. Diab, X. Li, X. V. Lin, T. Mihaylov, M. Ott, S. Shleifer, K. Shuster, D. Simig, P. S. Koura, A. Sridhar, T. Wang, and L. Zettlemoyer, “Opt: Open pre-trained transformer language models,” 2022.
*   [30] Y. Wang, H. Le, A. D. Gotmare, N. D. Bui, J. Li, and S. C. Hoi, “Codet5+: Open code large language models for code understanding and generation,” arXiv preprint arXiv:2305.07922, 2023.
*   [31] J. Devlin, M.-W. Chang, K. Lee, and K. Toutanova, “Bert: Pre-training of deep bidirectional transformers for language understanding,” arXiv preprint arXiv:1810.04805, 2018.
*   [32] T. Brown, B. Mann, N. Ryder, M. Subbiah, J. D. Kaplan, P. Dhariwal, A. Neelakantan, P. Shyam, G. Sastry, A. Askell, et al., “Language models are few-shot learners,” Advances in neural information processing systems, vol. 33, pp. 1877–1901, 2020.
*   [33] H. Touvron, T. Lavril, G. Izacard, X. Martinet, M.-A. Lachaux, T. Lacroix, B. Rozière, N. Goyal, E. Hambro, F. Azhar, et al., “Llama: Open and efficient foundation language models,” arXiv preprint arXiv:2302.13971, 2023.
*   [34] J. X. Chen, “The evolution of computing: Alphago,” Computing in Science & Engineering, vol. 18, no. 4, pp. 4–7, 2016.
*   [35] A. Zhao, D. Huang, Q. Xu, M. Lin, Y.-J. Liu, and G. Huang, “Expel: Llm agents are experiential learners,” in Proceedings of the AAAI Conference on Artificial Intelligence, vol. 38, pp. 19632–19642, 2024.
*   [36] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. Narasimhan, and Y. Cao, “React: Synergizing reasoning and acting in language models,” arXiv preprint arXiv:2210.03629, 2022.
*   [37] W. Huang, P. Abbeel, D. Pathak, and I. Mordatch, “Language models as zero-shot planners: Extracting actionable knowledge for embodied agents,” in International conference on machine learning, pp. 9118–9147, PMLR, 2022.
*   [38] G. Wang, Y. Xie, Y. Jiang, A. Mandlekar, C. Xiao, Y. Zhu, L. Fan, and A. Anandkumar, “Voyager: An open-ended embodied agent with large language models,” arXiv preprint arXiv:2305.16291, 2023.
*   [39] C. Whitehouse, M. Choudhury, and A. F. Aji, “Llm-powered data augmentation for enhanced cross-lingual performance,” 2023.
*   [40] J. White, Q. Fu, S. Hays, M. Sandborn, C. Olea, H. Gilbert, A. Elnashar, J. Spencer-Smith, and D. C. Schmidt, “A prompt pattern catalog to enhance prompt engineering with chatgpt,” arXiv preprint arXiv:2302.11382, 2023.
*   [41] M. Reid, N. Savinov, D. Teplyashin, D. Lepikhin, T. Lillicrap, J.-b. Alayrac, R. Soricut, A. Lazaridou, O. Firat, J. Schrittwieser, et al., “Gemini 1.5: Unlocking multimodal understanding across millions of tokens of context,” arXiv preprint arXiv:2403.05530, 2024.
*   [42] Z. Ji, N. Lee, R. Frieske, T. Yu, D. Su, Y. Xu, E. Ishii, Y. J. Bang, A. Madotto, and P. Fung, “Survey of hallucination in natural language generation,” ACM Computing Surveys, vol. 55, no. 12, pp. 1–38, 2023.
*   [43] K. An, F. Yang, L. Li, Z. Ren, H. Huang, L. Wang, P. Zhao, Y. Kang, H. Ding, Q. Lin, et al., “Nissist: An incident mitigation copilot based on troubleshooting guides,” arXiv preprint arXiv:2402.17531, 2024.
*   [44] J. Li, Q. Zhang, Y. Yu, Q. Fu, and D. Ye, “More agents is all you need,” 2024.
*   [45] Y. Dubois, C. X. Li, R. Taori, T. Zhang, I. Gulrajani, J. Ba, C. Guestrin, P. S. Liang, and T. B. Hashimoto, “Alpacafarm: A simulation framework for methods that learn from human feedback,” Advances in Neural Information Processing Systems, vol. 36, 2024.
*   [46] I. Bouzenia, P. Devanbu, and M. Pradel, “Repairagent: An autonomous, llm-based agent for program repair,” arXiv preprint arXiv:2403.17134, 2024.
*   [47] E. Musumeci, M. Brienza, V. Suriani, D. Nardi, and D. D. Bloisi, “Llm based multi-agent generation of semi-structured documents from semantic templates in the public administration domain,” in International Conference on Human-Computer Interaction, pp. 98–117, Springer, 2024.
*   [48] X. Luo, Y. Xue, Z. Xing, and J. Sun, “Prcbert: Prompt learning for requirement classification using bert-based pretrained language models,” in Proceedings of the 37th IEEE/ACM International Conference on Automated Software Engineering, pp. 1–13, 2022.
*   [49] T. Hey, J. Keim, A. Koziolek, and W. F. Tichy, “Norbert: Transfer learning for requirements classification,” in 2020 IEEE 28th International Requirements Engineering Conference (RE), pp. 169–179, 2020.
*   [50] J. Zhang, Y. Chen, N. Niu, and C. Liu, “Evaluation of chatgpt on requirements information retrieval under zero-shot setting,” Available at SSRN 4450322, 2023.
*   [51] C. Arora, J. Grundy, and M. Abdelrazek, “Advancing requirements engineering through generative ai: Assessing the role of llms,” in Generative AI for Effective Software Development, pp. 129–148, Springer, 2024.
*   [52] M. Krishna, B. Gaur, A. Verma, and P. Jalote, “Using llms in software requirements specifications: An empirical evaluation,” 2024.
*   [53] L. Ma, S. Liu, Y. Li, X. Xie, and L. Bu, “Specgen: Automated generation of formal program specifications via large language models,” 2024.
*   [54] C. Flanagan and K. R. M. Leino, “Houdini, an annotation assistant for esc/java,” in FME 2001: Formal Methods for Increasing Software Productivity (J. N. Oliveira and P. Zave, eds.), (Berlin, Heidelberg), pp. 500–517, Springer Berlin Heidelberg, 2001.
*   [55] J. White, S. Hays, Q. Fu, J. Spencer-Smith, and D. C. Schmidt, ChatGPT Prompt Patterns for Improving Code Quality, Refactoring, Requirements Elicitation, and Software Design, pp. 71–108. Cham: Springer Nature Switzerland, 2024.
*   [56] D. Luitel, S. Hassani, and M. Sabetzadeh, “Improving requirements completeness: Automated assistance through large language models,” Requirements Engineering, vol. 29, no. 1, pp. 73–95, 2024.
*   [57] A. Moharil and A. Sharma, “Identification of intra-domain ambiguity using transformer-based machine learning,” in Proceedings of the 1st International Workshop on Natural Language-Based Software Engineering, NLBSE ’22, (New York, NY, USA), p. 51–58, Association for Computing Machinery, 2023.
*   [58] K. Ronanki, B. Cabrero-Daniel, and C. Berger, “Chatgpt as a tool for user story quality evaluation: Trustworthy out of the box?,” in Agile Processes in Software Engineering and Extreme Programming – Workshops (P. Kruchten and P. Gregory, eds.), (Cham), pp. 173–181, Springer Nature Switzerland, 2024.
*   [59] A. Poudel, J. Lin, and J. Cleland-Huang, “Leveraging transformer-based language models to automate requirements satisfaction assessment,” 2023.
*   [60] E. Musumeci, M. Brienza, V. Suriani, D. Nardi, and D. D. Bloisi, “Llm based multi-agent generation of semi-structured documents from semantic templates in the public administration domain,” in Artificial Intelligence in HCI (H. Degen and S. Ntoa, eds.), (Cham), pp. 98–117, Springer Nature Switzerland, 2024.
*   [61] S. Zhang, J. Wang, G. Dong, J. Sun, Y. Zhang, and G. Pu, “Experimenting a new programming practice with llms,” 2024.
*   [62] A. Nouri, B. Cabrero-Daniel, F. Törner, H. Sivencrona, and C. Berger, “Engineering safety requirements for autonomous driving with large language models,” 2024.
*   [63] Z. Zhang, M. Rayhan, T. Herda, M. Goisauf, and P. Abrahamsson, “Llm-based agents for automating the enhancement of user story quality: An early report,” in Agile Processes in Software Engineering and Extreme Programming (D. Šmite, E. Guerra, X. Wang, M. Marchesi, and P. Gregory, eds.), (Cham), pp. 117–126, Springer Nature Switzerland, 2024.
*   [64] K. Ronanki, C. Berger, and J. Horkoff, “Investigating chatgpt’s potential to assist in requirements elicitation processes,” in 2023 49th Euromicro Conference on Software Engineering and Advanced Applications (SEAA), pp. 354–361, 2023.
*   [65] D. Xie, B. Yoo, N. Jiang, M. Kim, L. Tan, X. Zhang, and J. S. Lee, “Impact of large language models on generating software specifications,” 2023.
*   [66] A. Moharil and A. Sharma, “Tabasco: A transformer based contextualization toolkit,” Science of Computer Programming, vol. 230, p. 102994, 2023.
*   [67] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. de Oliveira Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, A. Ray, R. Puri, G. Krueger, M. Petrov, H. Khlaaf, G. Sastry, P. Mishkin, B. Chan, S. Gray, N. Ryder, M. Pavlov, A. Power, L. Kaiser, M. Bavarian, C. Winter, P. Tillet, F. P. Such, D. Cummings, M. Plappert, F. Chantzis, E. Barnes, A. Herbert-Voss, W. H. Guss, A. Nichol, A. Paino, N. Tezak, J. Tang, I. Babuschkin, S. Balaji, S. Jain, W. Saunders, C. Hesse, A. N. Carr, J. Leike, J. Achiam, V. Misra, E. Morikawa, A. Radford, M. Knight, M. Brundage, M. Murati, K. Mayer, P. Welinder, B. McGrew, D. Amodei, S. McCandlish, I. Sutskever, and W. Zaremba, “Evaluating large language models trained on code,” 2021.
*   [68] A. Ni, P. Yin, Y. Zhao, M. Riddell, T. Feng, R. Shen, S. Yin, Y. Liu, S. Yavuz, C. Xiong, S. Joty, Y. Zhou, D. Radev, and A. Cohan, “L2ceval: Evaluating language-to-code generation capabilities of large language models,” 2023.
*   [69] R. Sun, S. Ö. Arik, A. Muzio, L. Miculicich, S. Gundabathula, P. Yin, H. Dai, H. Nakhost, R. Sinha, Z. Wang, et al., “Sql-palm: Improved large language model adaptation for text-to-sql (extended),” arXiv preprint arXiv:2306.00739, 2023.
*   [70] Q. Zheng, X. Xia, X. Zou, Y. Dong, S. Wang, Y. Xue, Z. Wang, L. Shen, A. Wang, Y. Li, T. Su, Z. Yang, and J. Tang, “Codegeex: A pre-trained model for code generation with multilingual benchmarking on humaneval-x,” 2024.
*   [71] X. Hu, K. Kuang, J. Sun, H. Yang, and F. Wu, “Leveraging print debugging to improve code generation in large language models,” 2024.
*   [72] S. Peng, E. Kalliamvakou, P. Cihon, and M. Demirer, “The impact of ai on developer productivity: Evidence from github copilot,” 2023.
*   [73] D. Fried, A. Aghajanyan, J. Lin, S. Wang, E. Wallace, F. Shi, R. Zhong, W. tau Yih, L. Zettlemoyer, and M. Lewis, “Incoder: A generative model for code infilling and synthesis,” 2023.
*   [74] E. Nijkamp, B. Pang, H. Hayashi, L. Tu, H. Wang, Y. Zhou, S. Savarese, and C. Xiong, “Codegen: An open large language model for code with multi-turn program synthesis,” 2023.
*   [75] Y. Ding, M. J. Min, G. Kaiser, and B. Ray, “Cycle: Learning to self-refine the code generation,” Proc. ACM Program. Lang., vol. 8, apr 2024.
*   [76] Y. Dong, X. Jiang, Z. Jin, and G. Li, “Self-collaboration code generation via chatgpt,” 2024.
*   [77] F. Lin, D. J. Kim, et al., “When llm-based code generation meets the software development process,” arXiv preprint arXiv:2403.15852, 2024.
*   [78] S. Holt, M. R. Luyten, and M. van der Schaar, “L2MAC: Large language model automatic computer for extensive code generation,” in The Twelfth International Conference on Learning Representations, 2024.
*   [79] S. Hong, M. Zhuge, J. Chen, X. Zheng, Y. Cheng, C. Zhang, J. Wang, Z. Wang, S. K. S. Yau, Z. Lin, L. Zhou, C. Ran, L. Xiao, C. Wu, and J. Schmidhuber, “Metagpt: Meta programming for a multi-agent collaborative framework,” 2023.
*   [80] Z. Rasheed, M. Waseem, K.-K. Kemell, W. Xiaofeng, A. N. Duc, K. Systä, and P. Abrahamsson, “Autonomous agents in software development: A vision paper,” arXiv preprint arXiv:2311.18440, 2023.
*   [81] Z. Rasheed, M. Waseem, M. Saari, K. Systä, and P. Abrahamsson, “Codepori: Large scale model for autonomous software development by using multi-agents,” arXiv preprint arXiv:2402.01411, 2024.
*   [82] D. Huang, Q. Bu, J. M. Zhang, M. Luck, and H. Cui, “Agentcoder: Multi-agent-based code generation with iterative testing and optimisation,” arXiv preprint arXiv:2312.13010, 2023.
*   [83] T. Zheng, G. Zhang, T. Shen, X. Liu, B. Y. Lin, J. Fu, W. Chen, and X. Yue, “Opencodeinterpreter: Integrating code generation with execution and refinement,” arXiv preprint arXiv:2402.14658, 2024.
*   [84] T. Schick, J. Dwivedi-Yu, R. Dessì, R. Raileanu, M. Lomeli, E. Hambro, L. Zettlemoyer, N. Cancedda, and T. Scialom, “Toolformer: Language models can teach themselves to use tools,” Advances in Neural Information Processing Systems, vol. 36, 2024.
*   [85] Y. Qin, S. Liang, Y. Ye, K. Zhu, L. Yan, Y. Lu, Y. Lin, X. Cong, X. Tang, B. Qian, et al., “Toolllm: Facilitating large language models to master 16000+ real-world apis,” arXiv preprint arXiv:2307.16789, 2023.
*   [86] X. Jiang, Y. Dong, L. Wang, F. Zheng, Q. Shang, G. Li, Z. Jin, and W. Jiao, “Self-planning code generation with large language models,” ACM Trans. Softw. Eng. Methodol., jun 2024. Just Accepted.
*   [87] S. Zhang, J. Wang, G. Dong, J. Sun, Y. Zhang, and G. Pu, “Experimenting a new programming practice with llms,” arXiv preprint arXiv:2401.01062, 2024.
*   [88] V. Murali, C. Maddila, I. Ahmad, M. Bolin, D. Cheng, N. Ghorbani, R. Fernandez, and N. Nagappan, “Codecompose: A large-scale industrial deployment of ai-assisted code authoring,” arXiv preprint arXiv:2305.12050, 2023.
*   [89] J. Huang, S. S. Gu, L. Hou, Y. Wu, X. Wang, H. Yu, and J. Han, “Large language models can self-improve,” arXiv preprint arXiv:2210.11610, 2022.
*   [90] L. Chen, J. Q. Davis, B. Hanin, P. Bailis, I. Stoica, M. Zaharia, and J. Zou, “Are more llm calls all you need? towards scaling laws of compound inference systems,” arXiv preprint arXiv:2403.02419, 2024.
*   [91] X. Chen, M. Lin, N. Schärli, and D. Zhou, “Teaching large language models to self-debug,” arXiv preprint arXiv:2304.05128, 2023.
*   [92] S. Kang, B. Chen, S. Yoo, and J.-G. Lou, “Explainable automated debugging via large language model-driven scientific debugging,” arXiv preprint arXiv:2304.02195, 2023.
*   [93] G. Franceschelli and M. Musolesi, “On the creativity of large language models,” arXiv preprint arXiv:2304.00008, 2023.
*   [94] J. Lai, W. Gan, J. Wu, Z. Qi, and P. S. Yu, “Large language models in law: A survey,” 2023.
*   [95] L. Zheng, W.-L. Chiang, Y. Sheng, S. Zhuang, Z. Wu, Y. Zhuang, Z. Lin, Z. Li, D. Li, E. Xing, et al., “Judging llm-as-a-judge with mt-bench and chatbot arena,” Advances in Neural Information Processing Systems, vol. 36, 2024.
*   [96] Q. Wang, Z. Wang, Y. Su, H. Tong, and Y. Song, “Rethinking the bounds of llm reasoning: Are multi-agent discussions the key?,” arXiv preprint arXiv:2402.18272, 2024.
*   [97] L. Chen, Y. Zhang, S. Ren, H. Zhao, Z. Cai, Y. Wang, P. Wang, T. Liu, and B. Chang, “Towards end-to-end embodied decision making via multi-modal large language model: Explorations with gpt4-vision and beyond,” arXiv preprint arXiv:2310.02071, 2023.
*   [98] N. Shinn, F. Cassano, A. Gopinath, K. Narasimhan, and S. Yao, “Reflexion: Language agents with verbal reinforcement learning,” Advances in Neural Information Processing Systems, vol. 36, 2024.
*   [99] W. Chen, Y. Su, J. Zuo, C. Yang, C. Yuan, C. Qian, C.-M. Chan, Y. Qin, Y. Lu, R. Xie, et al., “Agentverse: Facilitating multi-agent collaboration and exploring emergent behaviors in agents,” arXiv preprint arXiv:2308.10848, 2023.
*   [100] G. Li, H. Hammoud, H. Itani, D. Khizbullin, and B. Ghanem, “Camel: Communicative agents for” mind” exploration of large language model society,” Advances in Neural Information Processing Systems, vol. 36, pp. 51991–52008, 2023.
*   [101] Z. Liu, W. Yao, J. Zhang, L. Xue, S. Heinecke, R. Murthy, Y. Feng, Z. Chen, J. C. Niebles, D. Arpit, et al., “Bolaa: Benchmarking and orchestrating llm-augmented autonomous agents,” arXiv preprint arXiv:2308.05960, 2023.
*   [102] J. Lu, W. Zhong, W. Huang, Y. Wang, Q. Zhu, F. Mi, B. Wang, W. Wang, X. Zeng, L. Shang, X. Jiang, and Q. Liu, “Self: Self-evolution with language feedback,” 2024.
*   [103] C. Xie, C. Chen, F. Jia, Z. Ye, K. Shu, A. Bibi, Z. Hu, P. Torr, B. Ghanem, and G. Li, “Can large language model agents simulate human trust behaviors?,” arXiv preprint arXiv:2402.04559, 2024.
*   [104] Z. Liu, W. Yao, J. Zhang, L. Yang, Z. Liu, J. Tan, P. K. Choubey, T. Lan, J. Wu, H. Wang, et al., “Agentlite: A lightweight library for building and advancing task-oriented llm agent system,” arXiv preprint arXiv:2402.15538, 2024.
*   [105] M. Zhuge, W. Wang, L. Kirsch, F. Faccio, D. Khizbullin, and J. Schmidhuber, “Language agents as optimizable graphs,” arXiv preprint arXiv:2402.16823, 2024.
*   [106] R. Feldt, S. Kang, J. Yoon, and S. Yoo, “Towards autonomous testing agents via conversational large language models,” in 2023 38th IEEE/ACM International Conference on Automated Software Engineering (ASE), pp. 1688–1693, IEEE, 2023.
*   [107] A. Happe and J. Cito, “Getting pwn’d by ai: Penetration testing with large language models,” in Proceedings of the 31st ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering, pp. 2082–2086, 2023.
*   [108] W. Ma, D. Wu, Y. Sun, T. Wang, S. Liu, J. Zhang, Y. Xue, and Y. Liu, “Combining fine-tuning and llm-based agents for intuitive smart contract auditing with justifications,” arXiv preprint arXiv:2403.16073, 2024.
*   [109] R. Fang, R. Bindu, A. Gupta, and D. Kang, “Llm agents can autonomously exploit one-day vulnerabilities,” arXiv preprint arXiv:2404.08144, 2024.
*   [110] S. Yao, D. Yu, J. Zhao, I. Shafran, T. Griffiths, Y. Cao, and K. Narasimhan, “Tree of thoughts: Deliberate problem solving with large language models,” Advances in Neural Information Processing Systems, vol. 36, 2024.
*   [111] Z. Rasheed, M. Waseem, A. Ahmad, K.-K. Kemell, W. Xiaofeng, A. N. Duc, and P. Abrahamsson, “Can large language models serve as data analysts? a multi-agent assisted approach for qualitative data analysis,” arXiv preprint arXiv:2402.01386, 2024.
*   [112] M. Ataei, H. Cheong, D. Grandi, Y. Wang, N. Morris, and A. Tessier, “Elicitron: An llm agent-based simulation framework for design requirements elicitation,” arXiv preprint arXiv:2404.16045, 2024.
*   [113] G. Sridhara, S. Mazumdar, et al., “Chatgpt: A study on its utility for ubiquitous software engineering tasks,” arXiv preprint arXiv:2305.16837, 2023.
*   [114] M. Desmond, Z. Ashktorab, Q. Pan, C. Dugan, and J. M. Johnson, “Evalullm: Llm assisted evaluation of generative outputs,” in Companion Proceedings of the 29th International Conference on Intelligent User Interfaces, pp. 30–32, 2024.
*   [115] M. Gao, X. Hu, J. Ruan, X. Pu, and X. Wan, “Llm-based nlg evaluation: Current status and challenges,” arXiv preprint arXiv:2402.01383, 2024.
*   [116] L. J. Wan, Y. Huang, Y. Li, H. Ye, J. Wang, X. Zhang, and D. Chen, “Software/hardware co-design for llm and its application for design verification,” in 2024 29th Asia and South Pacific Design Automation Conference (ASP-DAC), pp. 435–441, IEEE, 2024.
*   [117] K. Kolthoff, C. Bartelt, and S. P. Ponzetto, “Data-driven prototyping via natural-language-based gui retrieval,” Automated software engineering, vol. 30, no. 1, p. 13, 2023.
*   [118] V. D. Kirova, C. S. Ku, J. R. Laracy, and T. J. Marlowe, “Software engineering education must adapt and evolve for an llm environment,” in Proceedings of the 55th ACM Technical Symposium on Computer Science Education V. 1, pp. 666–672, 2024.
*   [119] S. Jalil, S. Rafi, T. D. LaToza, K. Moran, and W. Lam, “Chatgpt and software testing education: promises & perils (2023),” arXiv preprint arXiv:2302.03287, 2023.
*   [120] S. Suri, S. N. Das, K. Singi, K. Dey, V. S. Sharma, and V. Kaulgud, “Software engineering using autonomous agents: Are we there yet?,” in 2023 38th IEEE/ACM International Conference on Automated Software Engineering (ASE), pp. 1855–1857, IEEE, 2023.
*   [121] C. Qian, X. Cong, C. Yang, W. Chen, Y. Su, J. Xu, Z. Liu, and M. Sun, “Communicative agents for software development,” arXiv preprint arXiv:2307.07924, 2023.
*   [122] Y. Shen, K. Song, X. Tan, D. Li, W. Lu, and Y. Zhuang, “Hugginggpt: Solving ai tasks with chatgpt and its friends in hugging face,” Advances in Neural Information Processing Systems, vol. 36, 2024.
*   [123] J. Chen, X. Hu, S. Liu, S. Huang, W.-W. Tu, Z. He, and L. Wen, “Llmarena: Assessing capabilities of large language models in dynamic multi-agent environments,” arXiv preprint arXiv:2402.16499, 2024.
*   [124] M. Josifoski, L. Klein, M. Peyrard, Y. Li, S. Geng, J. P. Schnitzler, Y. Yao, J. Wei, D. Paul, and R. West, “Flows: Building blocks of reasoning and collaborating ai,” arXiv preprint arXiv:2308.01285, 2023.
*   [125] I. Weber, “Large language models as software components: A taxonomy for llm-integrated applications,” arXiv preprint arXiv:2406.10300, 2024.
*   [126] F. Vallecillos Ruiz, “Agent-driven automatic software improvement,” in Proceedings of the 28th International Conference on Evaluation and Assessment in Software Engineering, pp. 470–475, 2024.
*   [127] Z. Cheng, J. Kasai, and T. Yu, “Batch prompting: Efficient inference with large language model apis,” arXiv preprint arXiv:2301.08721, 2023.
*   [128] S. Shankar, J. Zamfirescu-Pereira, B. Hartmann, A. G. Parameswaran, and I. Arawjo, “Who validates the validators? aligning llm-assisted evaluation of llm outputs with human preferences,” arXiv preprint arXiv:2404.12272, 2024.
*   [129] D. Roy, X. Zhang, R. Bhave, C. Bansal, P. Las-Casas, R. Fonseca, and S. Rajmohan, “Exploring llm-based agents for root cause analysis,” in Companion Proceedings of the 32nd ACM International Conference on the Foundations of Software Engineering, pp. 208–219, 2024.
*   [130] Y. Li, Y. Zhang, and L. Sun, “Metaagents: Simulating interactions of human behaviors for llm-based task-oriented coordination via collaborative generative agents,” arXiv preprint arXiv:2310.06500, 2023.
*   [131] M. Tufano, D. Drain, A. Svyatkovskiy, S. K. Deng, and N. Sundaresan, “Unit test case generation with transformers and focal context,” arXiv preprint arXiv:2009.05617, 2020.
*   [132] Y. Zhang, W. Song, Z. Ji, N. Meng, et al., “How well does llm generate security tests?,” arXiv preprint arXiv:2310.00710, 2023.
*   [133] H. J. Kang, T. G. Nguyen, B. Le, C. S. Păsăreanu, and D. Lo, “Test mimicry to assess the exploitability of library vulnerabilities,” in Proceedings of the 31st ACM SIGSOFT International Symposium on Software Testing and Analysis, pp. 276–288, 2022.
*   [134] S. Feng and C. Chen, “Prompting is all you need: Automated android bug replay with large language models,” in Proceedings of the 46th IEEE/ACM International Conference on Software Engineering, pp. 1–13, 2024.
*   [135] S. Kang, J. Yoon, and S. Yoo, “Large language models are few-shot testers: Exploring llm-based general bug reproduction,” in 2023 IEEE/ACM 45th International Conference on Software Engineering (ICSE), pp. 2312–2323, IEEE, 2023.
*   [136] C. S. Xia, M. Paltenghi, J. Le Tian, M. Pradel, and L. Zhang, “Fuzz4all: Universal fuzzing with large language models,” in Proceedings of the IEEE/ACM 46th International Conference on Software Engineering, pp. 1–13, 2024.
*   [137] G. Ryan, S. Jain, M. Shang, S. Wang, X. Ma, M. K. Ramanathan, and B. Ray, “Code-aware prompting: A study of coverage guided test generation in regression setting using llm,” arXiv preprint arXiv:2402.00097, 2024.
*   [138] J. A. Pizzorno and E. D. Berger, “Coverup: Coverage-guided llm-based test generation,” arXiv preprint arXiv:2403.16218, 2024.
*   [139] K. Liu, Y. Liu, Z. Chen, J. M. Zhang, Y. Han, Y. Ma, G. Li, and G. Huang, “Llm-powered test case generation for detecting tricky bugs,” arXiv preprint arXiv:2404.10304, 2024.
*   [140] K. Li and Y. Yuan, “Large language models as test case generators: Performance evaluation and enhancement,” arXiv preprint arXiv:2404.13340, 2024.
*   [141] Z. Wang, W. Wang, Z. Li, L. Wang, C. Yi, X. Xu, L. Cao, H. Su, S. Chen, and J. Zhou, “Xuat-copilot: Multi-agent collaborative system for automated user acceptance testing with large language model,” arXiv preprint arXiv:2401.02705, 2024.
*   [142] C. Lee, C. S. Xia, J.-t. Huang, Z. Zhu, L. Zhang, and M. R. Lyu, “A unified debugging approach via llm-based multi-agent synergy,” arXiv preprint arXiv:2404.17153, 2024.
*   [143] G. Deng, Y. Liu, V. Mayoral-Vilches, P. Liu, Y. Li, Y. Xu, T. Zhang, Y. Liu, M. Pinzger, and S. Rass, “Pentestgpt: An llm-empowered automatic penetration testing tool,” arXiv preprint arXiv:2308.06782, 2023.
*   [144] M. Xiao, Y. Xiao, H. Dong, S. Ji, and P. Zhang, “Ritfis: Robust input testing framework for llms-based intelligent software,” arXiv preprint arXiv:2402.13518, 2024.
*   [145] R. Wang, Z. Li, C. Wang, Y. Xiao, and C. Gao, “Navrepair: Node-type aware c/c++ code vulnerability repair,” arXiv preprint arXiv:2405.04994, 2024.
*   [146] A. Shestov, A. Cheshkov, R. Levichev, R. Mussabayev, P. Zadorozhny, E. Maslov, C. Vadim, and E. Bulychev, “Finetuning large language models for vulnerability detection,” arXiv preprint arXiv:2401.17010, 2024.
*   [147] A. Cheshkov, P. Zadorozhny, and R. Levichev, “Evaluation of chatgpt model for vulnerability detection,” arXiv preprint arXiv:2304.07232, 2023.
*   [148] G. Lu, X. Ju, X. Chen, W. Pei, and Z. Cai, “Grace: Empowering llm-based software vulnerability detection with graph structure and in-context learning,” Journal of Systems and Software, vol. 212, p. 112031, 2024.
*   [149] H. Li, Y. Hao, Y. Zhai, and Z. Qian, “The hitchhiker’s guide to program analysis: A journey with large language models,” arXiv preprint arXiv:2308.00245, 2023.
*   [150] Y. Ding, Y. Fu, O. Ibrahim, C. Sitawarin, X. Chen, B. Alomair, D. Wagner, B. Ray, and Y. Chen, “Vulnerability detection with code language models: How far are we?,” arXiv preprint arXiv:2403.18624, 2024.
*   [151] F. V. Ruiz, A. Grishina, M. Hort, and L. Moonen, “A novel approach for automatic program repair using round-trip translation with large language models,” arXiv preprint arXiv:2401.07994, 2024.
*   [152] B. Yang, H. Tian, J. Ren, H. Zhang, J. Klein, T. F. Bissyandé, C. L. Goues, and S. Jin, “Multi-objective fine-tuning for enhanced program repair with llms,” arXiv preprint arXiv:2404.12636, 2024.
*   [153] T. Dettmers, A. Pagnoni, A. Holtzman, and L. Zettlemoyer, “Qlora: Efficient finetuning of quantized llms,” 2023.
*   [154] N. Jain, P. yeh Chiang, Y. Wen, J. Kirchenbauer, H.-M. Chu, G. Somepalli, B. R. Bartoldson, B. Kailkhura, A. Schwarzschild, A. Saha, M. Goldblum, J. Geiping, and T. Goldstein, “Neftune: Noisy embeddings improve instruction finetuning,” 2023.
*   [155] J. Zhang, J. P. Cambronero, S. Gulwani, V. Le, R. Piskac, G. Soares, and G. Verbruggen, “Pydex: Repairing bugs in introductory python assignments using llms,” Proceedings of the ACM on Programming Languages, vol. 8, no. OOPSLA1, pp. 1100–1124, 2024.
*   [156] H. Joshi, J. C. Sanchez, S. Gulwani, V. Le, G. Verbruggen, and I. Radiček, “Repair is nearly generation: Multilingual program repair with llms,” in Proceedings of the AAAI Conference on Artificial Intelligence, vol. 37, pp. 5131–5140, 2023.
*   [157] J. Xiang, X. Xu, F. Kong, M. Wu, H. Zhang, and Y. Zhang, “How far can we go with practical function-level program repair?,” arXiv preprint arXiv:2404.12833, 2024.
*   [158] E. Hilario, S. Azam, J. Sundaram, K. Imran Mohammed, and B. Shanmugam, “Generative ai for pentesting: the good, the bad, the ugly,” International Journal of Information Security, vol. 23, no. 3, pp. 2075–2097, 2024.
*   [159] L. Zhong, Z. Wang, and J. Shang, “Ldb: A large language model debugger via verifying runtime execution step-by-step,” arXiv preprint arXiv:2402.16906, 2024.
*   [160] L. Zhang, K. Li, K. Sun, D. Wu, Y. Liu, H. Tian, and Y. Liu, “Acfix: Guiding llms with mined common rbac practices for context-aware repair of access control vulnerabilities in smart contracts,” 2024.
*   [161] S. Hu, T. Huang, F. İlhan, S. F. Tekin, and L. Liu, “Large language model-powered smart contract vulnerability detection: New perspectives,” 2023.
*   [162] M. Alhanahnah, M. R. Hasan, and H. Bagheri, “An empirical evaluation of pre-trained large language models for repairing declarative formal specifications,” arXiv preprint arXiv:2404.11050, 2024.
*   [163] F. Geissler, K. Roscher, and M. Trapp, “Concept-guided llm agents for human-ai safety codesign,” in Proceedings of the AAAI Symposium Series, vol. 3, pp. 100–104, 2024.
*   [164] A. Nouri, B. Cabrero-Daniel, F. Törner, H. Sivencrona, and C. Berger, “Engineering safety requirements for autonomous driving with large language models,” arXiv preprint arXiv:2403.16289, 2024.
*   [165] C. Thapa, S. I. Jang, M. E. Ahmed, S. Camtepe, J. Pieprzyk, and S. Nepal, “Transformer-based language models for software vulnerability detection,” in Proceedings of the 38th Annual Computer Security Applications Conference, pp. 481–496, 2022.
*   [166] L. Zhong, Z. Wang, and J. Shang, “Debug like a human: A large language model debugger via verifying runtime execution step-by-step,” 2024.
*   [167] N. Alshahwan, M. Harman, I. Harper, A. Marginean, S. Sengupta, and E. Wang, “Assured llm-based software engineering,” 2024.
*   [168] A. Cheshkov, P. Zadorozhny, and R. Levichev, “Evaluation of chatgpt model for vulnerability detection,” 2023.

### -A Benchmarks

![Refer to caption](img/e5e29e79c82a8fb820a83f1f9eaf8e06.png)

Figure 12: Distribution of Benchmarks

### -B Evaluation Metrics

![Refer to caption](img/960d4a77b5ed67a17ae5bf7acb71b58c.png)

Figure 13: Top 10 Evaluation Metrics