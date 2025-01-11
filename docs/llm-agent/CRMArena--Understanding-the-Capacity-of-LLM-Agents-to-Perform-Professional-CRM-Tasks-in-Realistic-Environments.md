<!--yml
category: 未分类
date: 2025-01-11 12:00:35
-->

# CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments

> 来源：[https://arxiv.org/html/2411.02305/](https://arxiv.org/html/2411.02305/)

Kung-Hsiang Huang   Akshara Prabhakar   Sidharth Dhawan   Yixin Mao
Huan Wang   Silvio Savarese   Caiming Xiong   Philippe Laban   Chien-Sheng Wu
Salesforce AI Research
{kh.huang, akshara.prabhakar, sidharth, y.mao,
huan.wang, ssavarese, cxiong, wu.jason}@salesforce.com 

###### Abstract

Customer Relationship Management (CRM) systems are vital for modern enterprises, providing a foundation for managing customer interactions and data. Integrating AI agents into CRM systems can automate routine processes and enhance personalized service. However, deploying and evaluating these agents is challenging due to the lack of realistic benchmarks that reflect the complexity of real-world CRM tasks. To address this issue, we introduce CRMArena, a novel benchmark designed to evaluate AI agents on realistic tasks grounded on professional work environments. We worked with CRM experts to design nine customer service tasks distributed across three personas: service agent, analyst, and manager. We synthesize a large-scale simulated organization, populating 16 commonly-used industrial objects (e.g., account, order, knowledge article, case) with high interconnectivity, and uploading it into a real Salesforce CRM organization. UI and API access to the CRM is provided to systems that attempt to complete the tasks in CRMArena. Experimental results reveal that state-of-the-art LLM agents succeed in less than 40% of the tasks with ReAct prompting, and less than 55% even when provided manually-crafted function-calling tools. Our findings highlight the need for enhanced agent capabilities in function-calling and rule-following to be deployed in real-world work environment. CRMArena is an open challenge to the community: systems that can reliably complete tasks showcase direct business value in a popular work environment.¹¹1Our code and benchmark have been released at [https://github.com/SalesforceAIResearch/CRMArena](https://github.com/SalesforceAIResearch/CRMArena).

![[Uncaptioned image]](img/710dbe9d6d6452ad36067f9d4a8dd503.png)

CRMArena: Understanding the Capacity of LLM Agents to
Perform Professional CRM Tasks in Realistic Environments

Kung-Hsiang Huang   Akshara Prabhakar   Sidharth Dhawan   Yixin Mao Huan Wang   Silvio Savarese   Caiming Xiong   Philippe Laban   Chien-Sheng Wu Salesforce AI Research {kh.huang, akshara.prabhakar, sidharth, y.mao, huan.wang, ssavarese, cxiong, wu.jason}@salesforce.com

## 1 Introduction

Customer Relationship Management (CRM) systems are pivotal in modern enterprises, serving as the backbone for managing interactions with current and potential customers Winer ([2001](https://arxiv.org/html/2411.02305v1#bib.bib17)); Payne and Frow ([2005](https://arxiv.org/html/2411.02305v1#bib.bib12)). The integration of intelligent agents based on large language models (LLMs) into CRM systems promises to automate routine tasks, enhance operational efficiency, and revolutionize customer experiences. However, evaluating LLM agents in real-world professional environments remains a challenge, due to the absence of robust benchmarks that faithfully capture the complexity of tasks encountered in real-world CRM environments, largely due to data privacy concerns within enterprises.

Prior benchmarks on evaluating LLM agents on work-related tasks, such as WorkArena Drouin et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib3)), Workbench Styles et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib16)), and Tau Yao et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib20)) tend to focus on basic functionality, and fall short in two key areas. First, the complexity of the objects (e.g., tables in databases) and dependencies (e.g., foreign keys) between these objects is often overly simple, lacking the complexity of real-world scenarios. Second, the tasks included in the benchmarks, such as navigating web pages and filtering lists, are typically too straightforward and do not represent real-world work tasks.

To address these limitations, we introduce CRMArena, a comprehensive benchmark tailored to evaluate LLM agents on performing realistic CRM tasks in real-world work environments. CRMArena features a realistic sandbox environment modeled after Salesforce’s schema, developed using an extensible data generation pipeline powered by LLMs (top left of [Figure 1](https://arxiv.org/html/2411.02305v1#S1.F1 "In 1 Introduction ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments")). Specifically, the pipeline tackles two key challenges: (1) Object connectivity: reflecting the complex relationships between data objects (e.g., Account associated with Case and Order) by mirroring Salesforce’s Service Cloud schema²²2[https://architect.salesforce.com/diagrams/data-models/service-cloud/service-cloud-overview](https://architect.salesforce.com/diagrams/data-models/service-cloud/service-cloud-overview). (2) Introducing latent variables to better simulate realistic data dynamics, such as influencing case-filing behavior and modeling deviations from company guidelines.

Moreover, CRMArena defines tasks based on actual customer service personas. By consulting CRM experts experienced with Salesforce, we identified nine tasks representative of CRM use cases ([Section 2.1](https://arxiv.org/html/2411.02305v1#S2.SS1 "2.1 Tasks ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments")). These tasks span three personas: Service Manager, Service Agent, and Service Analyst. For example, Service Managers focus on agent performance and strategic resource allocation. [Table 1](https://arxiv.org/html/2411.02305v1#S2.T1 "In 2.1 Tasks ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments") compares CRMArena with previous datasets.

![Refer to caption](img/37291f416a0acdb6932efd297cfdf477.png)

Figure 1: An overview of the contribution of this work. We begin by generating realistic CRM data based on the Salesforce Service Cloud schema, ensuring both quality and diversity through rigorous verification processes. This verified data is then stored locally and uploaded to a Salesforce organization (Org). An expert study, conducted with domain experts, validated the data’s realism. Using this Org as a sandbox environment, we create query instances and benchmark various LLMs across different agentic frameworks.

CRMArena seamlessly integrates with Salesforce,³³3[https://www.salesforce.com/crm/](https://www.salesforce.com/crm/) enabling interaction via both the user interface and API access (see bottom of [Figure 1](https://arxiv.org/html/2411.02305v1#S1.F1 "In 1 Introduction ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments")). This integration facilitated an expert study with CRM professionals to assess the quality of our synthesized organization ([Section 2.5](https://arxiv.org/html/2411.02305v1#S2.SS5 "2.5 Expert Study ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments")). Study findings revealed that 90% of domain experts found the test environment to be Realistic or better, underscoring the benchmark’s fidelity to real-world CRM scenarios. Upon verifying the realism of CRMArena, we then assess various agentic systems through API access. We develop two sets of tools general-purpose vs. task-specific tools, combine them with three agentic frameworks and various LLMs. Findings indicate that all LLM agents struggle to reliably complete tasks when using general-purpose tools, with top performing systems completing less than 40% of the tasks. Incorporating manually designed tools can enhance performance, with top LLM agents solving up to 55% of the tasks. However, we discover that weaker LLMs often do not benefit from manually-crafted tools due to their weaker function calling capabilities.

In summary, our main contributions are:

*   •

    Introducing CRMArena, a realistic CRM agent benchmark with tasks validated by domain experts to evaluate LLM agents in real-world business scenarios.

*   •

    Developing a data generation strategy anchored in a real-world CRM schema, incorporating latent variables, deduplication, and rigorous data validation to ensure diversity and quality.

*   •

    Demonstrating through experiments that even state-of-the-art LLM agents do not reliably complete CRMArena tasks, emphasizing the benchmark’s value and challenges.

## 2 CRMArena

Motivated by tasks commonly addressed by CRM personas: service manager, service agent, and service analyst, CRMArena comprises nine tasks that reflect real-world CRM scenarios. Verified by domain experts as common occurrences in CRM, an overview of these tasks is presented in [Figure 2](https://arxiv.org/html/2411.02305v1#S2.F2 "In Best Region Identification (BRI) ‣ 2.1 Tasks ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"). Below, we provide detailed illustrations of each task.

### 2.1 Tasks

The tasks in CRMArena are designed to accurately reflect the variety of challenges encountered in real-world CRM environments. They span the responsibilities of three key personas: the Service Manager, who focuses on strategic resource allocation; the Service Agent, who addresses customer inquiries; and the Service Analyst, who analyzes data trends and performance metrics to improve service operations.

 Datasets # Objects # Dependencies/ Object Real-world Environment Realistic Work Tasks Expert Validation WorkBench Styles et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib16)) 5 0 ✗ ✗ ✗ Tau-Bench Yao et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib20)) 3 0.67 ✗ ✗ ✗ WorkArena Drouin et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib3)) 7 0.86 ✓ ✗ ✗ CRMArena (Ours) 16 1.31 ✓ ✓ ✓ 

Table 1: A comparison between our benchmark with prior datasets. CRMArena is the most complex benchmark given the highest number of objects and object dependencies involved. Furthermore, CRMArena is the only expert-validated benchmark that encompasses both a real-world environment and realistic work tasks.

#### New Case Routing (NCR)

The goal of this task is to assign the best human agent to an incoming case, aiming to optimize various performance metrics. The input consists of a case subject and description, and the output is the ID of the recommended human agent. This task assesses LLM agent’s ability to match cases to the most suitable human agent based on case histories and the skills and availability of these agents.

#### Handle Time Understanding (HTU)

This task involves identifying the agent with the shortest/longest average handle time. Given the case history data, the objective is to determine the human agent who handled the previous cases the fastest/slowest.

#### Transfer Count Understanding (TCU)

In this task, the LLM agent must find out which human agent transferred cases to others the least/most given a period of case history. Both HTU and TCU evaluate LLM agent’s capacity to analyze performance based on predefined metrics accurately.

#### Name Entity Disambiguation (NED)

The LLM agent must disambiguate named entities related to customer transactions. Here, we focus on disambiguating product names. Given the query shown in [Figure 2](https://arxiv.org/html/2411.02305v1#S2.F2 "In Best Region Identification (BRI) ‣ 2.1 Tasks ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), the agent needs to identify the specific order corresponding to running shoes bought by the mentioned customer within the given time frame. This tests the understanding of product names and customer order histories.

#### Policy Violation Identification (PVI)

In this task, the LLM agent is given a case with interaction between a customer and an agent and must determine if any company policies have been breached. This involves analyzing the case details and comparing them against policy rules outlined in knowledge articles to identify violations.

#### Knowledge Question Answering (KQA)

The goal here is for the LLM agent to answer a specific question based on knowledge articles. This evaluates the agent’s capacity to look for accurate and relevant information from the CRM knowledge repository.

#### Top Issue Identification (TII)

This task requires the LLM agent to identify the most reported issue for a particular product. Given case history, the agent must determine which issue has the highest frequency. This tests the ability to analyze issue reports for trend analysis.

#### Monthly Trend Analysis (MTA)

The LLM agent must determine which months have the highest number of cases for a given product and a given time period. By analyzing the case history in a given period of time, the LLM agent identifies the month with the most cases, demonstrating its ability to recognize trends and patterns over time.

#### Best Region Identification (BRI)

In this task, the LLM agent’s objective is to identify the regions where cases are closed the fastest. The agent must analyze case closure times across various regions and indicate which regions perform best.

![Refer to caption](img/b14e204838d4d8f446b9fa3675c424e1.png)

Figure 2: An overview of the nine distinct tasks introduced in CRMArena.

### 2.2 Sandbox Environment

Creating a sandbox environment for CRMArena poses unique challenges, particularly related to privacy concerns and the need for realistic data without using real enterprise data. To this end, we develop a scalable data generation pipeline capable of producing diverse and realistic CRM data across various domains. In our setup, we model 16 business objects. The complete list of objects and their descriptions can be found in [Appendix D](https://arxiv.org/html/2411.02305v1#A4 "Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"). There are two major challenges for building such a pipeline: object connectivity and hidden casual relationship. In the following subsections, we illustrate how we address these challenges.

#### Object Connectivity

Real-world company data is characterized by complex interconnections between objects like Case and Account. Our data generation approach, based on Salesforce’s Service Cloud schema, ensures high connectivity. For instance, the Case object is connected to objects like Account, Contact, and User. [Figure 7](https://arxiv.org/html/2411.02305v1#A4.F7 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments") displays these interdependencies, creating meaningful data environments. [Table 1](https://arxiv.org/html/2411.02305v1#S2.T1 "In 2.1 Tasks ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments") highlights our benchmark’s much higher object connectivity compared to existing work.

#### Hidden Causal Relationship

Replicating the implicit causal relationships found in real-world data presents a significant challenge. To address this, we introduce latent variables that simulate various underlying factors, creating data that mirrors the subtle dependencies and patterns in authentic CRM databases. These latent variables are crucial for facilitating certain tasks and generating desired scenarios. As shown in the example in [Figure 3](https://arxiv.org/html/2411.02305v1#S2.F3 "In Quality and Diversity Assurance ‣ 2.2 Sandbox Environment ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), the ShoppingHabit variable allows us to more realistically simulate a customer’s purchasing patterns based on time periods or holiday seasons. Similarly, the Skill latent variable for the User (Agent) object enables our simulations of EmailMessage and LiveChatTranscript to include situations where an agent is unable to resolve an issue and must transfer it to another agent. Without this latent variable, we would lack scenarios essential for our Transfer Count Understanding task. The full data generation flow is shown in [Figure 8](https://arxiv.org/html/2411.02305v1#A4.F8 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

#### Quality and Diversity Assurance

We generate data in JSON format, with each JSON object representing one entry of an object, to ensure higher controllability Huang et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib5)); Laban et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib7)). Due to the large volume of objects (e.g., 500 Product entries paired with 40+ Pricebook entries resulting in over 20,000 PricebookEntry items) and the limited maximum output tokens of LLMs, directly prompting LLMs to generate all entries of an object is infeasible. To address this, we employ mini-batch prompting with a batch size of 10\. However, this approach can lead to duplicated or highly similar content across batches. To mitigate this issue, we implement a two-phase de-duplication strategy. First, for all objects, we include all previously generated entries in the prompt during mini-batch prompting and instruct the LLM not to generate the same content. After data generation, we use string exact matching to remove duplicate entries for fields and objects crucial to certain tasks (e.g., the email of User).

Additionally, we subject the data to a rigorous quality assurance process involving a dual-layer verification. The format verifier ensures all data entries conform to predefined schemas by checking whether each entry in the generated mini-batch contains all required fields for the object. Mini-batches that fail this verification are discarded and regenerated. The content verifier checks for feasibility for tasks, focusing on objects crucial for specific tasks. For example, in the Named Entity Disambiguation task, we verify that the paraphrased ambiguous product name (1) does not deviate too much from its original name and (2) is not too similar to other products the customer has purchased. In this scenario, the content verifier provides an LLM with a list of products the customer has purchased and the paraphrased product name. If the LLM correctly identifies the product, we retain the entry; otherwise, it is discarded.⁴⁴4We utilize gpt-4o as the LLM for data synthesis and content verification for its cost efficiency.

![Refer to caption](img/b90cbf66118c8adaf7af0e3bf0db8062.png)

(a)

![Refer to caption](img/7c9be7e9c5903325638ca557b92fe798.png)

(b)

Figure 3: Examples of latent variables (gray) influencing object (blue) generation. (a) The ShoppingHabit variable affects when and what type of products a customer buys. (b) The Skill variable determines if an agent can handle a case or needs to transfer it.

#### Upload to Org

Once the data is generated, we populate it into a clean Simple Demo Org (SDO)⁵⁵5[https://partners.salesforce.com/s/education/general/Salesforce_Orgs](https://partners.salesforce.com/s/education/general/Salesforce_Orgs) on Salesforce without latent variables. This exclusion serves two purposes: it mirrors the typical scenario where companies do not have access to such information, thus providing a more realistic testing environment, and it adds an extra layer of challenge compared to testing on the generated databases. Moreover, utilizing Salesforce’s SDO as the sandbox eliminates the necessity and complexity for local environment setup, which is required in many related work Styles et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib16)); Drouin et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib3)); Yao et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib20)); Zhou et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib24)). This approach not only facilitates testing but also encourages scientific rigor and future research on our benchmark. The details of the sandbox environment can be found in [Appendix D](https://arxiv.org/html/2411.02305v1#A4 "Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

#### Environment Specification

The input to our data generation pipeline are company name, company description, database schema, and the scale of the objects (e.g., number of cases and products). We choose to create an Org for a fictional shoe company due to the diverse product range and customer service scenarios typical in the footwear industry. The scale of our generated data is designed to reflect a mid-sized enterprise, with thousands of orders and hundreds of products and support cases spanning a 4-year period. The total number of entries per object is shown in [Appendix D](https://arxiv.org/html/2411.02305v1#A4 "Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

#### Extensibility

Our data generation pipeline is designed for flexibility and can be easily adapted to other industries through changes in user-specified input parameters. For instance, by specifying the industry in the company description, our pipeline can automatically generate realistic CRM data tailored to that specific industry, such as finance. Furthermore, to accommodate other use cases beyond customer service, such as sales, users would only need to provide the corresponding schema (e.g., Salesforce Sales Cloud schema for sales). This flexibility ensures that the pipeline can be extended to meet a wide range of business needs and LLM agent benchmarking purposes.

Note that our current setup reflects a simplified version of CRM scenarios, where each Case is linked to both an Issue and a Product. This simplification helps manage the complexity of tasks like Top Issue Identification, which would otherwise require LLM agents to individually analyze every case, making the tasks too infeasible for the current state of LLMs. Our benchmark can be adjusted to create more complex settings by removing such dependencies as LLM capabilities advance.

### 2.3 Query Instance Generation

Following the creation of the sandbox environment, we generate natural language query instances and their ground-truth answers to benchmark our tasks. For the Knowledge QA tasks, queries can be naively constructed by prompting an LLM each knowledge article to generate question answer pairs (Laban et al., [2022](https://arxiv.org/html/2411.02305v1#bib.bib8); Huang et al., [2024](https://arxiv.org/html/2411.02305v1#bib.bib5)). For the remaining tasks, we construct query instance through a four-step process: (1) seed query construction, (2) ground-truth computation, (3) ID mapping, and (4) query paraphrasing.

We manually create 14 seed queries in total with placeholders for corresponding variables, such as time period or product name. This facilitates the development of functions that compute the ground truth answers on the generated database by leveraging the latent variables that are only visible there. For example, an agent’s policy violation during a live chat is verifiable only within the generated database. Upon obtaining the answers, we map the IDs in the generated database to their counterparts in Salesforce Org, thereby establishing the ground truths for our queries in the sandbox environment. Finally, to ensure diversity in the test queries, we employ an LLM to paraphrase the seed queries, enhancing the robustness and variety of our benchmarking process. An example of this process is shown in the top right of [Figure 1](https://arxiv.org/html/2411.02305v1#S1.F1 "In 1 Introduction ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

Additionally, to simulate real-world scenarios where some questions may be unanswerable, we construct non-answerable queries. Inspired by the non-answerable question schema outlined in Brahman et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib1)), we focus on False Presuppositions queries, which are most relevant in CRM settings. For example, a query may request the identification of an agent who transfers the most cases during a given period, despite no agents transferring cases in that period. We include non-answerable queries in five tasks: Transfer Time Understanding, Handle Time Understanding, Top Issue Identification, Named Entity Disambiguation, and Policy Violation Identification. For these instances, we expect models to produce “None” as outputs. In summary, non-answerable queries account for 30% of the total queries per corresponding task. Overall, we produce 130 query instances per task, totaling 1,170 queries for CRMArena. Details and seed queries are provided in [Appendix B](https://arxiv.org/html/2411.02305v1#A2 "Appendix B Query Generation Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

### 2.4 Tools: APIs and Functions

Salesforce Orgs naturally support a variety of general-purpose APIs, such as the Apex API, REST API, and Tooling API, which are designed to cover a broad set of functionalities within the Salesforce ecosystem. For the scope of our tasks and their integration with a Python environment, we choose to utilize SOQL and SOSL queries⁶⁶6[https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql_sosl_intro.htm](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql_sosl_intro.htm). SOQL queries are intended for obtaining a specific subset of objects using exact matches or filtering criteria, typically formatted as "SELECT Id ...", while SOSL queries enable fuzzy searching in objects like knowledge articles and product names, formatted as "FIND ...". These two types of queries can theoretically support a wide range of query instances, eliminating the necessity to manually design actions for function calls.

In addition to general-purpose APIs, we also develop task-specific tools in the form of Python wrapper functions to facilitate the evaluation of function-calling agents. These functions optimize task performance by providing structured and logical operations directly mapped to typical CRM tasks. We manually define 27 such Python wrapper functions on top of SOQL and SOSL (complete list in [Appendix C](https://arxiv.org/html/2411.02305v1#A3 "Appendix C Action Space Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments")) to streamline function calls and target the key components needed for each task. These task-specific functions are designed to maximize reusability across various tasks, minimizing the need for task-specific customizations.

![Refer to caption](img/55183801e75f3102d7419d02797f710c.png)

Figure 4: Expert study results. The plots illustrate domain experts’ realism ratings for the overall Org structure (top) and individual objects we generated (bottom).

### 2.5 Expert Study

To ensure the realism and practicality of the sandbox environment we developed, we conducted a user study involving ten experts with diverse professional backgrounds who have experience working on Salesforce Orgs daily. These experts were recruited via the User Interviews platform⁷⁷7[https://www.userinterviews.com/](https://www.userinterviews.com/). Details of the expert study can be found in [Appendix F](https://arxiv.org/html/2411.02305v1#A6 "Appendix F Expert Study Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

Each session of the expert study was structured into three parts. First, we provided the experts with an overview of our sandbox, highlighting key objects such as Case and Contact, and allowing them access through relevant URLs. This initial orientation was designed to familiarize them with the organization. Second, we assigned them five query instances sampled from CRMArena, each representing a different task, to complete. This task completion phase was aimed at evaluating the practical application and operational coherence of the sandbox in executing real-world CRM tasks. Finally, the experts rated the realism of our Org environment compared to the real-world systems they are accustomed to. They also provided detailed rationales for their ratings, giving insights into how our environment aligns with actual CRM scenarios.

The results of our expert study are presented in [Figure 4](https://arxiv.org/html/2411.02305v1#S2.F4 "In 2.4 Tools: APIs and Functions ‣ 2 CRMArena ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"). The findings are highly encouraging: 90% of the experts rated our populated Org as either Realistic or Very Realistic. This positive assessment extended to the individual objects within the Org, with more than 77% of experts giving them similarly high ratings for realism. These results strongly suggest that our sandbox environment closely mirrors real-world CRM systems. We provide the qualitative feedback and rationale from the experts we interviewed in [Table 14](https://arxiv.org/html/2411.02305v1#A6.T14 "In F.3 Qualitative Feedback ‣ Appendix F Expert Study Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

## 3 Benchmarking Experiments

 | Model | NCR | HTU | TCU | NED | PVI | KQA | TII | MTA | BRI | ALL |
| Act |
| gpt-4o | 43.1 | 10.0 | 17.7 | 30.8 | 28.5 | 29.3 | 68.5 | 29.2 | 7.7 | 29.4 |
| gpt-4o-mini | 0.8 | 38.5 | 23.8 | 9.2 | 0.0 | 43.1 | 26.9 | 3.8 | 3.8 | 16.7 |
| claude-3.5-sonnet | 78.5 | 24.6 | 15.4 | 51.5 | 28.5 | 44.7 | 45.4 | 20.8 | 26.9 | 37.4 |
| claude-3-sonnet | 9.2 | 26.9 | 24.6 | 30.8 | 23.8 | 16.6 | 16.2 | 1.5 | 0.0 | 16.6 |
| llama3.1-405b | 46.2 | 17.7 | 17.7 | 13.9 | 30.0 | 47.0 | 15.4 | 5.4 | 6.9 | 22.2 |
| llama3.1-70b | 28.5 | 20.0 | 24.6 | 6.2 | 30.0 | 47.9 | 8.5 | 0.0 | 1.5 | 18.6 |
| ReAct |
| gpt-4o | 70.0 | 39.2 | 22.3 | 30.8 | 35.4 | 50.2 | 64.6 | 20.9 | 10.8 | 38.2 |
| gpt-4o-mini | 40.8 | 36.9 | 25.4 | 31.5 | 24.6 | 52.8 | 30.0 | 6.2 | 6.2 | 28.3 |
| claude-3.5-sonnet | 62.9 | 20.0 | 11.5 | 52.3 | 30.0 | 45.0 | 43.9 | 20.8 | 21.5 | 34.3 |
| claude-3-sonnet | 7.7 | 24.6 | 26.9 | 29.2 | 28.5 | 16.0 | 22.3 | 0.8 | 0.0 | 17.3 |
| llama3.1-405b | 81.5 | 22.3 | 15.4 | 33.9 | 34.6 | 55.3 | 34.6 | 13.9 | 13.1 | 33.8 |
| llama3.1-70b | 48.5 | 20.0 | 13.9 | 33.1 | 37.7 | 48.7 | 23.9 | 13.9 | 10.8 | 27.8 |
| Function Calling |
| gpt-4o | 60.0 | 47.7 | 81.5 | 46.2 | 39.2 | 30.4 | 97.7 | 27.7 | 59.2 | 54.4 |
| gpt-4o-mini | 0.8 | 10.8 | 10.8 | 17.7 | 13.8 | 39.7 | 60.0 | 0.0 | 21.5 | 19.5 |
| claude-3.5-sonnet | 4.6 | 33.1 | 82.3 | 52.3 | 30.0 | 40.5 | 69.2 | 26.9 | 36.9 | 41.8 |
| claude-3-sonnet | 0.8 | 1.5 | 30.0 | 25.4 | 41.5 | 23.2 | 12.3 | 1.5 | 0.0 | 15.1 |
| llama3.1-405b (prompt) | 16.2 | 31.5 | 64.6 | 50.0 | 26.9 | 47.6 | 95.4 | 86.9 | 42.3 | 51.3 |
| llama3.1-70b (prompt) | 1.5 | 23.1 | 44.6 | 53.8 | 37.4 | 42.4 | 93.8 | 43.8 | 29.2 | 41.1 | 

Table 2: Overall performance (%) of various LLMs under different agentic frameworks on CRMArena. The evaluation metric is F1 score for the knowledge question answering (KQA) task and exact match for all other tasks. ALL represents the average performance across all tasks.

### 3.1 Experimental Settings

#### Models

We evaluate state-of-the-art proprietary and open-source LLMs, including gpt models (gpt-4o and gpt-3.5-turbo); claude models (claude-3.5-sonnet and claude-3-sonnet), and the llama models (llama-3.1-405b and llama-3.1-70B). ⁸⁸8We observed the native function-calling mode to perform poorly and hence report the prompt mode performance for Llama 3.1 models. With these models, we tested three common agentic frameworks: Act, ReAct Yao et al. ([2023](https://arxiv.org/html/2411.02305v1#bib.bib21)), and Function Calling (FC). ReAct is a prompt-based method, with each step characterized by a thought and action process, while Act is ReAct without the thought component. The details of these settings are described in the following paragraphs and [Appendix G](https://arxiv.org/html/2411.02305v1#A7 "Appendix G Implementation Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

#### Action Space

Every task can be formulated as a Partially Observable Markov Decision Process (POMDP) $(\mathcal{U},\mathcal{S},\mathcal{A},\mathcal{O},\mathcal{T},\mathcal{R})$ with instruction space $\mathcal{U}$, state space $\mathcal{S}$, action space $\mathcal{A}$, observation space $\mathcal{O}$, transition function $\mathcal{T}:\mathcal{S}\times\mathcal{A}\to\mathcal{S}$, and reward function $\mathcal{R}:\mathcal{S}\times\mathcal{A}\to\{0,1\}$. In the Act and ReAct settings, the action space is rich, i.e. $\mathcal{A}=\{\texttt{execute <query>},\texttt{submit <result>}\}$. Given a user query $u\in\mathcal{U}$ in natural language, an agent can execute <query> to issue a SOQL or SOSL query to interact with the instance to receive the observation $o_{t}\in\mathcal{O}$ of executing the query in the environment. This continues until the agent chooses to submit and receives a binary reward $r=\mathcal{R}(s_{T},\texttt{submit})\in\{0,1\}$. In the Function Calling setting, the agent interacts with the environment via API tools implemented as Python functions. In this case the agent is not directly exposed to the Salesforce environment and the object dependencies are kept hidden. Internally the APIs interact with the environment in a controlled manner defined by us. An action $a$ is of the form tool_call{**kwargs}. The system prompts for these three setups are described in [Appendix E](https://arxiv.org/html/2411.02305v1#A5 "Appendix E Prompts ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

#### Observation Space

Actions are executed on the sandbox environment through the Simple Salesforce package⁹⁹9[https://github.com/simple-salesforce/simple-salesforce](https://github.com/simple-salesforce/simple-salesforce). If an action succeeds, the environment will return the queried data in the CRM system; otherwise, an error message, such as incorrect function calling parameters, is returned.

#### Evaluation Metrics

For the knowledge QA task, since it is an open-ended text generation task, we use F1 scores. For the remaining tasks, we only need to compare the predicted and ground truth object IDs; therefore, an exact match is used.

### 3.2 Results

The main results are summarized in [Table 2](https://arxiv.org/html/2411.02305v1#S3.T2 "In 3 Benchmarking Experiments ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"). We made the following observations. First, real-world CRM tasks remain challenging for top LLM agents. Using the ReAct framework, the best model (gpt-4o) only achieves an overall score of 38.2%. Even when equipped with human-crafted functions, the overall performance is still only 54.4%. These findings highlight the challenges of our CRMArena. Second, stronger and weaker LLMs show opposite trend on different agentic frameworks. In particular, models like gpt-4o and claude-3.5-sonnet score higher in the FC setting, while their weaker counterparts performs worse when equipped with function calling capabilities. This indicate that human-defined functions may not always help LLM agents, as weaker models may not be able to properly utilize the functions, resulting in lower performance. Finally, open-source models are catching up the proprietary LLMs. Across three settings, we see the llama models score similar, and sometimes higher, than the gpt and claude models. This indicate a closing gap between the open and closed-source models. From [Figure 6](https://arxiv.org/html/2411.02305v1#A4.F6 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), we observe how llama models tend to show higher scope for error recovery based on execution feedback than the closed-source models.

 Model # Completion Tokens # Turns Cost ($) ReAct gpt-4o 48568.73 5.4 0.182 claude-3.5-sonnet 70814.75 6.9 0.228 llama3.1-405b 35647.29 7.3 0.125 FC gpt-4o 78305.38 6.8 0.305 claude-3.5-sonnet 105248.43 8.1 0.371 

Table 3: The cost of top-performing agents averaged across queries and tasks.

### 3.3 Discussions

#### What is the most cost-effective solution?

In two-thrid of the agentic frameworks, gpt-4o performs the best. The efficiency of gpt-4o is also reflected in [Table 3](https://arxiv.org/html/2411.02305v1#S3.T3 "In 3.2 Results ‣ 3 Benchmarking Experiments ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), which shows that gpt-4o has the lowest cost per instance and requires the least number of turns to complete a query. Therefore, the most cost-effective solution is using gpt-4o under the function calling setting.

#### How does the type of function affect model performance?

In [Table 2](https://arxiv.org/html/2411.02305v1#S3.T2 "In 3 Benchmarking Experiments ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), we observe that equipping LLM agents with function calling capabilities does not necessary results in increased performance. To better understand this phenomenon, we categorizes the functions based on two dimensions: functionality and functional dependency. Functionality refers to whether the function solely queries the CRM system via SOSL or SOQL (Query) or if it includes additional operations such as mathematical calculations or aggregations (Calculation). Functional dependency, on the other hand, classifies functions into those that rely on the outputs of other functions (Dependent)and those that are independent (Independent). This is crucial because our benchmark requires LLM agents to perform a sequence of calls, with each call dependent on the output of the previous ones Qin et al. ([2023](https://arxiv.org/html/2411.02305v1#bib.bib13)); Lu et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib10)). [Table 15](https://arxiv.org/html/2411.02305v1#A7.T15 "In Appendix G Implementation Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments") shows the list of functions and tasks we tested.

We sampled four function-task pairs from each category to evaluate the performance of gpt-4o, gpt-4o-mini, and claude-3-sonnet when specific functions were removed from the toolset, substituting two generic functions, execute_soql and execute_sosl, to execute arbitrary queries. The findings, summarized in [Table 4](https://arxiv.org/html/2411.02305v1#S3.T4 "In How does the type of function affect model performance? ‣ 3.3 Discussions ‣ 3 Benchmarking Experiments ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), indicate that while all function types enhance gpt-4o’s performance, they do not have the same effect on gpt-4o-mini or claude-3-sonnet. This suggests that stronger models are better at utilizing human-crafted functions effectively, whereas weaker models might struggle. Interestingly, Calculation functions, hypothesized to benefit LLMs weak in mathematical operations, may actually decrease performance in weaker models due to their limited function calling capabilities.

 Functionality Dependency gpt-4o gpt-4o-mini claude-3-sonnet Query Independent -6.6 -6.9 2.3 Query Dependent -2.9 3.0 7.5 Calculation Independent -9.4 4.6 -3.3 Calculation Dependent -26.7 4.0 3.3 

Table 4: Performance difference (%) when removing each category of functions. A lower number indicates more useful functions to the LLM agents.

![Refer to caption](img/6895e7fd173fc4b275e044e59882d68a.png)

Figure 5: The consistency of gpt-4o using different agentic frameworks.

#### How consistent are the agents across multiple trials?

Consistency is important for LLM agents, especially when deployed in work environments. We evaluate the consistency of LLM agents through multiple trials of prompting. Here, we adapt the pass^k metric proposed by Yao et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib20)). pass^k computes the probability that all k independent and identically distributed task attempts are successful, averaged over all tasks. We run ten trials across all tasks in CRMArena except for KQA, as the reward for KQA is not binary. The results are shown in [Figure 5](https://arxiv.org/html/2411.02305v1#S3.F5 "In How does the type of function affect model performance? ‣ 3.3 Discussions ‣ 3 Benchmarking Experiments ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), we found that, surprisingly, pass^k for all three agentic frameworks we tested drop at the nearly same rate as k increases. This indicates that the consistency for these three frameworks are similar and that the top-performing LLM cannot reliably solve the tasks with any of the three agentic frameworks we evaluated.

## 4 Related Work

#### Agent Benchmark

Several benchmarks have been developed to evaluate LLM-based agents Yao et al. ([2022](https://arxiv.org/html/2411.02305v1#bib.bib19)); Liu et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib9)); Jimenez et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib6)). Recently, major efforts have focused specifically on web agents, which challenges LLMs to navigate and perform actions on websites. These websites are often about everyday scenarios, such as e-commerce, and social discussion form Deng et al. ([2023](https://arxiv.org/html/2411.02305v1#bib.bib2)); He et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib4)); Zhou et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib24)); Lù et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib11)); Yoran et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib22)). Another line of work focus on evaluating the safety of deploying agents Ruan et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib14)); Yuan et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib23)).

#### Work-oriented Datasets

A few studies have developed datasets specifically for work-oriented tasks. The CRM Benchmark Salesforce ([2024](https://arxiv.org/html/2411.02305v1#bib.bib15)) aims to assess LLMs’ text generation and summarization abilities in business applications. WorkBench Styles et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib16)) consists of five databases designed to evaluate LLM agents’ performance in simple work tasks, such as sending emails, creating calendar invites, and counting traffic sources for a website. $\tau$-Bench Yao et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib20)) creates tasks that require interactions with users to obtain relevant information and authorization, achieved by using LLMs to simulate users. WorkArena Drouin et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib3)) builds a web-based work environment that allows for testing agents with visual capabilities.

## 5 Conclusion

This work introduces CRMArena, a novel benchmark for evaluating LLM agents in performing realistic CRM tasks within professional work environments. By incorporating expert-validated tasks and modeling intricate data interconnections typical of CRM systems, CRMArena offers a comprehensive and realistic challenge for LLM agents. Our experiments demonstrate that even state-of-the-art LLMs struggle with these realistic tasks, achieving limited success rates even with function-calling capabilities. These findings highlight the gap between current LLM capabilities and the requirements of real-world CRM scenarios. CRMArena serves as a foundational step towards more sophisticated evaluations of LLM agents in realistic work environments.

## 6 Ethical Considerations

This work introduces a benchmark for evaluating LLM agents within the context of CRM systems. While the data used is synthetically generated, it is modeled after real-world CRM data structures and tasks. Thus, it is important to consider the ethical implications of this work, particularly regarding data biases and privacy concerns.

#### Data Bias

Although synthetic, the data is generated by models trained on real-world data, which may contain inherent biases. These biases, related to customer demographics, purchase behavior, or case resolution, could be inadvertently reflected in the generated data, potentially perpetuating stereotypes or discriminatory practices. Thankfully, after conducting a thorough manual inspection of the generated data to identify potential biases, we did not observe such patterns.

#### Privacy Concerns

While our benchmark does not use any real customer data and therefore does not have access to personal information, the structure and nature of CRM data itself can raise privacy concerns. The tasks in our benchmark involve accessing sensitive customer information, albeit synthetic. To ensure responsible handling of this data, even though synthetic, we performed a thorough manual inspection to verify the absence of any personally identifiable information and to confirm that the data cannot be used to infer private information about individuals. This meticulous review process reinforces our commitment to ethical data practices and mitigates potential privacy risks.

## 7 Limitations

The CRMArena comprises nine tasks that thoroughly assess the ability of LLM agents to perform duties typically associated with three primary roles within a realistic environment: Service Manager, Service Agent, and Service Analyst. Nonetheless, this study does not encompass other common personas in CRM, such as sales representatives. We aim to incorporate these additional roles in our future studies.

## References

*   Brahman et al. (2024) Faeze Brahman, Sachin Kumar, Vidhisha Balachandran, Pradeep Dasigi, Valentina Pyatkin, Abhilasha Ravichander, Sarah Wiegreffe, Nouha Dziri, Khyathi Chandu, Jack Hessel, et al. 2024. The art of saying no: Contextual noncompliance in language models. *arXiv preprint arXiv:2407.12043*.
*   Deng et al. (2023) Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samuel Stevens, Boshi Wang, Huan Sun, and Yu Su. 2023. [Mind2web: Towards a generalist agent for the web](https://openreview.net/forum?id=kiYqbO3wqw). In *Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track*.
*   Drouin et al. (2024) Alexandre Drouin, Maxime Gasse, Massimo Caccia, Issam H. Laradji, Manuel Del Verme, Tom Marty, David Vazquez, Nicolas Chapados, and Alexandre Lacoste. 2024. [Workarena: How capable are web agents at solving common knowledge work tasks?](https://openreview.net/forum?id=BRfqYrikdo) In *Forty-first International Conference on Machine Learning*.
*   He et al. (2024) Hongliang He, Wenlin Yao, Kaixin Ma, Wenhao Yu, Yong Dai, Hongming Zhang, Zhenzhong Lan, and Dong Yu. 2024. [WebVoyager: Building an end-to-end web agent with large multimodal models](https://doi.org/10.18653/v1/2024.acl-long.371). In *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 6864–6890, Bangkok, Thailand. Association for Computational Linguistics.
*   Huang et al. (2024) Kung-Hsiang Huang, Philippe Laban, Alexander Fabbri, Prafulla Kumar Choubey, Shafiq Joty, Caiming Xiong, and Chien-Sheng Wu. 2024. [Embrace divergence for richer insights: A multi-document summarization benchmark and a case study on summarizing diverse information from news articles](https://doi.org/10.18653/v1/2024.naacl-long.32). In *Proceedings of the 2024 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies (Volume 1: Long Papers)*, pages 570–593, Mexico City, Mexico. Association for Computational Linguistics.
*   Jimenez et al. (2024) Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik R Narasimhan. 2024. [SWE-bench: Can language models resolve real-world github issues?](https://openreview.net/forum?id=VTF8yNQM66) In *The Twelfth International Conference on Learning Representations*.
*   Laban et al. (2024) Philippe Laban, Alexander R Fabbri, Caiming Xiong, and Chien-Sheng Wu. 2024. Summary of a haystack: A challenge to long-context llms and rag systems. In *Proceedings of the 2024 Conference on Empirical Methods in Natural Language Processing*. Association for Computational Linguistics.
*   Laban et al. (2022) Philippe Laban, Chien-Sheng Wu, Lidiya Murakhovs’ ka, Xiang’Anthony’ Chen, and Caiming Xiong. 2022. Discord questions: A computational approach to diversity analysis in news coverage. *arXiv preprint arXiv:2211.05007*.
*   Liu et al. (2024) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, and Jie Tang. 2024. [Agentbench: Evaluating LLMs as agents](https://openreview.net/forum?id=zAdUB0aCTQ). In *The Twelfth International Conference on Learning Representations*.
*   Lu et al. (2024) Jiarui Lu, Thomas Holleis, Yizhe Zhang, Bernhard Aumayer, Feng Nan, Felix Bai, Shuang Ma, Shen Ma, Mengyu Li, Guoli Yin, Zirui Wang, and Ruoming Pang. 2024. [Toolsandbox: A stateful, conversational, interactive evaluation benchmark for llm tool use capabilities](https://arxiv.org/abs/2408.04682). *Preprint*, arXiv:2408.04682.
*   Lù et al. (2024) Xing Han Lù, Zdeněk Kasner, and Siva Reddy. 2024. Weblinx: Real-world website navigation with multi-turn dialogue. *arXiv preprint arXiv:2402.05930*.
*   Payne and Frow (2005) Adrian Payne and Pennie Frow. 2005. A strategic framework for customer relationship management. *Journal of marketing*, 69(4):167–176.
*   Qin et al. (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, Sihan Zhao, Lauren Hong, Runchu Tian, Ruobing Xie, Jie Zhou, Mark Gerstein, Dahai Li, Zhiyuan Liu, and Maosong Sun. 2023. [Toolllm: Facilitating large language models to master 16000+ real-world apis](https://arxiv.org/abs/2307.16789). *Preprint*, arXiv:2307.16789.
*   Ruan et al. (2024) Yangjun Ruan, Honghua Dong, Andrew Wang, Silviu Pitis, Yongchao Zhou, Jimmy Ba, Yann Dubois, Chris J Maddison, and Tatsunori Hashimoto. 2024. Identifying the risks of lm agents with an lm-emulated sandbox. In *The Twelfth International Conference on Learning Representations*.
*   Salesforce (2024) Salesforce. 2024. [Salesforce announces the world’s first llm benchmark for crm](https://www.salesforce.com/uk/news/stories/crm-benchmark/).
*   Styles et al. (2024) Olly Styles, Sam Miller, Patricio Cerda-Mardini, Tanaya Guha, Victor Sanchez, and Bertie Vidgen. 2024. [Workbench: a benchmark dataset for agents in a realistic workplace setting](https://openreview.net/forum?id=4HNAwZFDcH). In *First Conference on Language Modeling*.
*   Winer (2001) Russell S Winer. 2001. A framework for customer relationship management. *California management review*, 43(4):89–105.
*   Yang et al. (2024) John Yang, Akshara Prabhakar, Karthik Narasimhan, and Shunyu Yao. 2024. Intercode: Standardizing and benchmarking interactive coding with execution feedback. *Advances in Neural Information Processing Systems*, 36.
*   Yao et al. (2022) Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. 2022. [Webshop: Towards scalable real-world web interaction with grounded language agents](https://proceedings.neurips.cc/paper_files/paper/2022/file/82ad13ec01f9fe44c01cb91814fd7b8c-Paper-Conference.pdf). In *Advances in Neural Information Processing Systems*, volume 35, pages 20744–20757\. Curran Associates, Inc.
*   Yao et al. (2024) Shunyu Yao, Noah Shinn, Pedram Razavi, and Karthik Narasimhan. 2024. Tau-bench: A benchmark for tool-agent-user interaction in real-world domains. *arXiv preprint arXiv:2406.12045*.
*   Yao et al. (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, and Yuan Cao. 2023. [React: Synergizing reasoning and acting in language models](https://openreview.net/forum?id=WE_vluYUL-X). In *The Eleventh International Conference on Learning Representations*.
*   Yoran et al. (2024) Ori Yoran, Samuel Joseph Amouyal, Chaitanya Malaviya, Ben Bogin, Ofir Press, and Jonathan Berant. 2024. [Assistantbench: Can web agents solve realistic and time-consuming tasks?](https://arxiv.org/abs/2407.15711) *Preprint*, arXiv:2407.15711.
*   Yuan et al. (2024) Tongxin Yuan, Zhiwei He, Lingzhong Dong, Yiming Wang, Ruijie Zhao, Tian Xia, Lizhen Xu, Binglin Zhou, Fangqi Li, Zhuosheng Zhang, et al. 2024. R-judge: Benchmarking safety risk awareness for llm agents. *arXiv preprint arXiv:2401.10019*.
*   Zhou et al. (2024) Shuyan Zhou, Frank F. Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, Uri Alon, and Graham Neubig. 2024. [Webarena: A realistic web environment for building autonomous agents](https://openreview.net/forum?id=oKn9c6ytLx). In *The Twelfth International Conference on Learning Representations*.

## Appendix A Further Discussions

#### Reward vs number of turns

In [Figure 6](https://arxiv.org/html/2411.02305v1#A4.F6 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), we show the distribution of the number of turns it takes for agents to successfully complete an user query.

#### Non-answerable query analysis

In [Table 5](https://arxiv.org/html/2411.02305v1#A3.T5 "In Appendix C Action Space Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), we present the performance of each LLM agents. Overall, LLM agents are good at handling such queries, compared to standard queries. Interestingly, a trend shown in [Table 2](https://arxiv.org/html/2411.02305v1#S3.T2 "In 3 Benchmarking Experiments ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments") is observed in this experiment as well: function calling only benefit stronger LLMs, while weaker LLMs like claude-3-sonnet and gpt-4o performs worse when equipped with function calling capabilities.

## Appendix B Query Generation Details

[Table 6](https://arxiv.org/html/2411.02305v1#A3.T6 "In Appendix C Action Space Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments") show the complete list of seed queries used in our experiments. More examples of how the final queries are constructed can be found in [Table 9](https://arxiv.org/html/2411.02305v1#A4.T9 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

## Appendix C Action Space Details

In the text-based agent settings (i.e. ReAct and Act), the actions include (1) executing SOSL queries, (2) executing SOQL queries, and (3) submitting the answer. In the function-calling settings, the actions are a list of carefully designed functions, shown in [Table 7](https://arxiv.org/html/2411.02305v1#A3.T7 "In Appendix C Action Space Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

 | Model | HTU | TCU | NED | TII | PVI |
| Act |
| gpt-4o | 15.4 | 48.7 | 94.9 | 87.2 | 92.3 |
| gpt-4o-mini | 94.9 | 79.5 | 30.8 | 79.5 | 74.4 |
| claude-3.5-sonnet | 25.6 | 28.2 | 82.1 | 33.3 | 84.6 |
| claude-3-sonnet | 84.6 | 79.5 | 100.0 | 51.3 | 74.4 |
| llama3.1-405b | 56.4 | 51.3 | 46.2 | 38.5 | 0.0 |
| llama3.1-70b | 46.2 | 76.9 | 20.5 | 20.5 | 100.0 |
| ReAct |
| gpt-4o | 64.1 | 48.7 | 100.0 | 84.6 | 74.4 |
| gpt-4o-mini | 97.4 | 82.1 | 97.4 | 61.5 | 71.8 |
| claude-3.5-sonnet | 12.8 | 7.7 | 87.2 | 30.8 | 82.1 |
| claude-3-sonnet | 79.5 | 84.6 | 94.9 | 69.2 | 94.9 |
| llama3.1-405b | 53.8 | 38.5 | 97.4 | 41.0 | 64.1 |
| llama3.1-70b | 64.1 | 41.0 | 97.4 | 17.9 | 17.9 |
| Function Calling |
| gpt-4o | 59.0 | 84.6 | 74.4 | 100.0 | 35.9 |
| gpt-4o-mini | 15.4 | 7.7 | 0.0 | 0.0 | 0.0 |
| claude-3.5-sonnet | 52.6 | 74.4 | 100.0 | 100.0 | 100.0 |
| claude-3-sonnet | 2.6 | 15.4 | 59.0 | 38.5 | 56.4 | 

Table 5: Performance (%) of various LLMs under different agentic frameworks on CRMArena for the non-answerable queries.

 | 1\. In [YEAR] [MONTH/QUARTER/SEASON], identify the agent who managed more than [NCASES] cases and had the [EXTREMA] handle time. |
| 2\. In the past [TIMEPERIOD], find the agent with the [EXTREMA] handle time among those who managed more than [NCASES] cases. |
| 3\. During the last [TIMEPERIOD], which agent had the [EXTREMA] average handle time for those handling over [NCASES] cases? |
| 4\. In [YEAR] [MONTH/QUARTER/SEASON], identify the agent who managed more than [NCASES] cases and had the [EXTREMA] transfer counts. |
| 5\. In the past [TIMEPERIOD], find the agent with the [EXTREMA] transfer counts among those who managed more than [NCASES] cases. |
| 6\. During the last [TIMEPERIOD], which agent had the [EXTREMA] average transfer counts for those handling over [NCASES] cases? |
| 7\. Which knowledge article did the agent violate policy? |
| 8\. Today is [TODAY]. Is there any month in which the cases we received for [PRODUCT] is much more than other months over the past [TIMEPERIOD]? |
| 9\. Today is [TODAY]. For [PRODUCT], what is the most common issue in the last [TIMEPERIOD]. |
| 10\. Today is [TODAY]. In [YEAR] [MONTH/QUARTER/SEASON], what is the most common issue for [PRODUCT]. |
| 11\. Today is [TODAY]. In which states do we close cases the fastest in the last [TIMEPERIOD]? |
| 12\. Today is [TODAY]. In [YEAR] [MONTH/QUARTER/SEASON], which states do we close cases the fastest. |
| 13\. What is the best agent to assign to for this case? |
| 14\. Today is [TODAY]. Show me the [PRODUCT] that I ordered [PERIOD] ago. | 

Table 6: The full set of seed queries used for query generation.

 Functions Description get_agents_with_max_cases(subset_cases) Returns a list of agent IDs with the maximum number of cases from the given subset of cases. get_agents_with_min_cases(subset_cases) Returns a list of agent IDs with the minimum number of cases from the given subset of cases. calculate_average_handle_time(cases) Calculates the average handle time for each agent based on a list of cases. get_start_date(end_date, period, interval_count) Calculates the start date based on the end date, period, and interval count. get_period(period_name, year) Calculates the start and end date based on the period name and year. get_agent_handled_cases_by_period(start_date, end_date) Retrieves the number of cases handled by each agent within a specified time period. get_qualified_agent_ids_by_case_count(agent_handled_cases, n_cases) Filters agent IDs based on the number of cases they have handled. get_cases(start_date, end_date, agent_ids, case_ids, Retrieves cases based on various filtering criteria. order_item_ids, issue_ids, statuses) get_non_transferred_case_ids(start_date, end_date) Retrieves the IDs of cases that were not transferred between agents within a specified date range. get_agent_transferred_cases_by_period(start_date, end_date, Retrieves the number of cases transferred between agents within a specified date range. qualified_agent_ids) get_shipping_state(cases) Adds shipping state information to the given cases. calculate_region_average_closure_times(cases) Calculates the average closure times for cases grouped by region (shipping state). get_order_item_ids_by_product(product_id) Retrieves the order item IDs associated with a given product. get_issue_counts(start_date, end_date, order_item_ids) Retrieves the issue counts for a product within a given time period. find_id_with_max_value(values_by_id) Identifies the ID with the maximum value from a dictionary. find_id_with_min_value(values_by_id) Identifies the ID with the minimum value from a dictionary. get_account_id_by_contact_id(contact_id) Retrieves the Account ID associated with a given Contact ID. get_purchase_history(account_id, purchase_date, related_product_ids) Retrieves the purchase history for a specific account, date, and set of products. get_month_to_case_count(cases) Counts the number of cases for each month from a list of cases. search_knowledge_articles(search_term) Searches for knowledge articles based on a given search term. search_products(search_term) Searches for products based on a given search term. get_issues() Retrieves a list of issue records. get_email_messages_by_case_id(case_id) Retrieves the email exchanges for a given case. get_livechat_transcript_by_case_id(case_id) Retrieves the live chat transcript for a given case. submit(content) Returns the response content. 

Table 7: The complete list of functions for the function calling settings.

## Appendix D Sandbox Environment Details

We show the objects and dependencies in [Figure 7](https://arxiv.org/html/2411.02305v1#A4.F7 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"). These objects, except for Knowledge__kav are densely connected, reflecting the complexity of real-world work environment. The total number of entry per objects is shown in [Table 8](https://arxiv.org/html/2411.02305v1#A4.T8 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"). Our data generation flow is shown in [Figure 8](https://arxiv.org/html/2411.02305v1#A4.F8 "In Appendix D Sandbox Environment Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

Object Number of Entries User 100 Contact 196 ProductCategory 12 Product 500 OrderItem 71,00 Pricebook 44 PricebookEntry 22,000 Case 977 Order 2,071 EmailMessage 3,234 LiveChatTranscript 387 Knowledge 45

Table 8: The number of entries per object.

![Refer to caption](img/09daffdeae98842ecda2578e410f5638.png)

Figure 6: Distribution of the number of turns it takes for agents to reach the goal under ReAct.

![Refer to caption](img/f791714bf22011a07ea63a3934d19ffb.png)

Figure 7: The objects and their dependencies in our sandbox environment.

![Refer to caption](img/b9c2f56e9a80a553dae0436ac6ce273d.png)

Figure 8: Data generation overview. The generation of each object is conditioned on the previously generated objects with arrows pointing to it. Blue boxes represent standard object, while gray boxes denote latent variables that are not uploaded to the Salesforce Org.

 <svg class="ltx_picture" height="249" id="A4.T9.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,249) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="221.44" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| Handle Time Understand Seed query: In [YEAR] [MONTH/QUARTER/SEASON], identify the agent who managed more than [NCASES] cases and had the [EXTREMA] handle time. Filled-in query: In 2021 February, identify the agent who managed more than 2 cases and had the highest handle time. Paraphrased query: In February 2021, determine the agent with the longest handle time who managed more than 2 cases. |
| Top Issue Identification Seed query: In [YEAR] [MONTH/QUARTER/SEASON], what is the most common issue for [PRODUCT]? Filled-in Query: In 2023 Q2, what is the most common issue for Flex Yoga Mat? Paraphrased query: What was the most frequent issue with Flex Yoga Mat in the second quarter of 2021? |</foreignobject></g></g></svg> 

Table 9: Examples of the query generation process.

### D.1 Object Details

Below, we describe the details of each object.

*   •

    ProductCategory: Represents the category that products are organized in.

*   •

    Product2: Represents a product that your company sells.

*   •

    ProductCategoryProduct: Holds the relation between product and product category to assign products to a category.

*   •

    Pricebook2: Represents a price book that contains the list of products.

*   •

    Pricebook Entry: Represents a product entry (an association between a Pricebook2 and Product2) in a price book.

*   •

    Order: Represents an order associated with a contract or an account.

*   •

    Order Item: Represents an order product that the company sells.

*   •

    Knowledge: Documentation or information articles that are accessible to users or customers.

*   •

    Contact: Refers to an individual or party related to an account.

*   •

    Issue: Represents a type of problem raised by a customer.

*   •

    Account: An entity, company, or individual your company does business with. In B2C setting, an account represents a customer.

*   •

    User (agent): System user, often representing customer support agents.

*   •

    Case: A record that describes a customer inquiry or issue.

*   •

    CaseHistory: A log of the changes and updates made to a case over time.

*   •

    EmailMessage: Email communication related to cases or customer inquiries between an agent and a customer.

*   •

    LiveChatTranscript: A conversation from a live chat session between an agent and a customer.

## Appendix E Prompts

In this section, we display the prompts used in our experiments. [Table 10](https://arxiv.org/html/2411.02305v1#A5.T10 "In Appendix E Prompts ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), [Table 11](https://arxiv.org/html/2411.02305v1#A5.T11 "In Appendix E Prompts ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), [Table 12](https://arxiv.org/html/2411.02305v1#A5.T12 "In Appendix E Prompts ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments") show the system prompt for the Act, ReAct, and Function Calling settings, respectively.

 <svg class="ltx_picture" height="429.24" id="A5.T10.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,429.24) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="401.68" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| You are an expert in Salesforce and you have access to a Salesforce instance. |
| Instructions - You will be provided a question, the system description, and relevant task context. - Interact with the Salesforce instance to build Salesforce Object Query Language (SOQL) or Salesforce Object Search Language (SOSL) queries as appropriate, to help you answer the question. - Salesforce Object Search Language (SOSL) can be used to construct text-based search queries against the search index. - Your generation should always be an Action command and NOTHING ELSE. Generate only one Action command. - DO NOT generate ANY system observation, you will receive this based on your Action command. - If no record is found matching the requirements mentioned, just return ’None’. Here is a description of how to use these commands: Action - Can be ’execute’ or ’submit’. - execute, to execute SOQL/SOSL that will return the observation from running the query on the Salesforce instance. - submit, to return the final answer of the task to the user. - Format: <execute> a valid SOQL/SOSL query </execute> or <submit> response to user </submit> Guidelines - Execute SOQL/SOSL queries to understand the Salesforce instance that will help you find the answer to the question. - When you are confident about the answer, submit it. - Always end with a submit action containing ONLY the answer, NO full sentences or any explanation. Example 1 Question: What is the total number of opportunities? Output: <execute> SELECT COUNT() FROM Opportunity </execute> (If the observation from the Salesforce instance 100, your next step can be) <submit> 100 </submit> NOT <submit> The total number of opportunities is 100 </submit> Example 2 [… Hide details for space… ] Salesforce instance description The objects available in the Salesforce instance are: User, Account, Contact, Case, Knowledge__kav, ProductCategory, Product2, … The fields available for the objects along with their descriptions and dependencies are: User - FirstName: First name of the agent - LastName: Last name of the agent - Email: Email address of the agent [… Hide details for space… ] Additional task context Handle/Transfer Times Policies [… Hide details for space… ] |</foreignobject></g></g></svg> 

Table 10: The system prompt used in the Act setting.

 <svg class="ltx_picture" height="516.53" id="A5.T11.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,516.53) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="488.97" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| You are an expert in Salesforce and you have access to a Salesforce instance. |
| Instructions - You will be provided a question, the system description, and relevant task context. - Think step by step and interact with the Salesforce instance to build Salesforce Object Query Language (SOQL) or Salesforce Object Search Language (SOSL) queries as appropriate, to help you answer the question. - Salesforce Object Search Language (SOSL) can be used to construct text-based search queries against the search index. - Your generation should always be a Thought followed by an Action command and NOTHING ELSE. Generate only one Thought and one Action command. - DO NOT generate ANY system observation, you will receive this based on your Action command. - If no record is found matching the requirements mentioned, just return ’None’. Here is a description of how to use these commands: Thought - A single line of reasoning to process the context and inform the decision making. Do not include any extra lines. - Format: <thought> your thought </thought> Action - Can be ’execute’ or ’submit’. - execute, to execute SOQL/SOSL that will return the observation from running the query on the Salesforce instance. - submit, to return the final answer of the task to the user. - Format: <execute> a valid SOQL/SOSL query </execute> or <submit> response to user </submit> Guidelines - Always start with a Thought and then proceed with an Action. - Generate only one Thought and one Action command at a time. - Execute SOQL/SOSL queries to understand the Salesforce instance that will help you find the answer to the question. - When you are confident about the answer, submit it. - Always end with a submit action containing ONLY the answer, NO full sentences or any explanation. Example 1 Question: What is the total number of opportunities? Output: <thought> I need to find the total number of opportunities in the system. </thought> <execute> SELECT COUNT() FROM Opportunity </execute> (If the observation from the Salesforce instance 100, your next step can be) <thought> I have found the total number of opportunities. </thought> <submit> 100 </submit> NOT <submit> The total number of opportunities is 100 </submit> Example 2 [… Hide details for space… ] Salesforce instance description The objects available in the Salesforce instance are: User, Account, Contact, Case, Knowledge__kav, ProductCategory, Product2, … The fields available for the objects along with their descriptions and dependencies are: User - FirstName: First name of the agent - LastName: Last name of the agent - Email: Email address of the agent [… Hide details for space… ] Additional task context Handle/Transfer Times Policies [… Hide details for space… ] |</foreignobject></g></g></svg> 

Table 11: The system prompt used in the ReAct setting.

 <svg class="ltx_picture" height="361.3" id="A5.T12.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,361.3) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="333.74" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| Instructions - You are an expert in Salesforce and you have access to a Salesforce instance. - You will be provided a question, the system description, and relevant task context. - Interact with the Salesforce instance using the tools provided to help you answer the question. - You should ALWAYS make ONLY ONE tool call at a time. If you want to submit your final answer, use the ’submit’ tool. If not, you should call some other tool. But ALWAYS make a tool call. - Always end by calling ’submit’ tool containing ONLY the answer, NO full sentence or any explanation. - If your answer is empty that is there are no records found matching the requirements mentioned, just return ’None’ to the ’submit’ tool. Additional task context Case Routing Policy The case routing policy determines the best agent to assign the given new case based on the following criteria - Issue Expertise: The agent who has closed the most cases associated with the issue most similar to the given case. - Product Expertise: If there is a tie in issue expertise, the best agent is the one who has solved the most cases associated with the product most relevant to the given case. - Workload: If there is still a tie, the best agent is the one that has least cases with Status not ’Closed’. Domain Details Quarters of the Year - Q1: January 1 to March 31 (both inclusive). - Q2: April 1 to June 30 (both inclusive). - Q3: July 1 to September 30 (both inclusive). - Q4: October 1 to December 31 (both inclusive). Seasons - Winter: December 1 to February 28/29 (both inclusive). - Spring: March 1 to May 31 (both inclusive). - Summer: June 1 to August 31 (both inclusive). - Autumn/Fall: September 1 to November 30 (both inclusive). Time Periods - Past 2 quarters: This refers to any timeframe spanning two quarters back from a specified ‘end_date‘. That translates to a six-month period retrospectively from the ‘end_date‘. - Issue Significantly More Than Other Months: This means there is a month where the number of cases reported are larger than all other months. |</foreignobject></g></g></svg> 

Table 12: The system prompt used in the Function Calling setting.

## Appendix F Expert Study Details

 Profession Gender Age Customer Service Associate Female 23 Customer Service Associate Female 25 Customer Service Agent Male 39 Customer Service Associate Male 29 Customer Service Advisor Male 49 Customer Service Manager Male 39 Account Executive Female 25 Technical Support Female 38 Customer Service Advisor Female 25 Customer Service Agent Female 35 

Table 13: The background of the participants in our expert study.

As detailed in [Table 13](https://arxiv.org/html/2411.02305v1#A6.T13 "In Appendix F Expert Study Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), we recruited a diverse range of domain experts for our study. The participants varied in age, gender, and professional backgrounds.

### F.1 Recruitment Criteria

Using the User Interviews platform, we set the job filter such that the participants of our survey must have a job title of one of the following:

*   •

    Account Manager

*   •

    Technical Support Engineer

*   •

    Support Engineer

*   •

    Technical Support Specialist

*   •

    Technical Support Manager

*   •

    Technical Support Technician

*   •

    Technical Support Agent

*   •

    Technical Support Expert

*   •

    Account Manager/Agent

*   •

    Account Manager/Analyst

*   •

    Customer Service Advisor/Customer Service Associate

*   •

    Customer Service Associate

*   •

    Customer Service Representative

In addition, we have created a screener survey. The most important question in the survey is “How often do you use Salesforce CRM?”. The valid candidate must select the option “Several times a day” to be able to participate in our study.

### F.2 The study

We use Google Form to conduct expert studies due to its ease to use. The study is broken down into three parts:

*   •

    Part 1: Familiarizing the Org [5 minutes]. This is for a broad overview of some of the objects in this Org.

*   •

    Part 2: Task Completion [45 minutes]. At this stage, they are be given tasks regarding customer service. They should try to accomplish as many as possible within the 45-minute time frame.

*   •

    Part 3: Quality Rating [10 minutes]. Based on their experience with the first two parts of this study, rate the quality of the Org and objects.

Below, we illustrate how each part is executed.

#### Part 1

In this part, we provide interviewee the log in credentials to our created Org (sandbox environment). Once they log in, they are instructed to spend 5 minutes to read the objects in the Org that are relevant to the tasks they will be completing later. The instructions and interface for this part are shown in [Figure 9](https://arxiv.org/html/2411.02305v1#A6.F9 "In Part 1 ‣ F.2 The study ‣ Appendix F Expert Study Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

![Refer to caption](img/8c795b44ab2b22a074a901cf579d90a8.png)

Figure 9: The instructions and interface of Part 1 of our expert study.

#### Part 2

After familiarizing with our created Org, participants are then asked to complete the tasks. They are required to complete 5 query instances from CRMArena. An example of the query is shown in [Figure 10](https://arxiv.org/html/2411.02305v1#A6.F10 "In Part 2 ‣ F.2 The study ‣ Appendix F Expert Study Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

![Refer to caption](img/a3cc157243a9f69abccad783a245ef68.png)

Figure 10: An example query instance for the part 2 of expert study.

#### Part 3

Upon completing the first two parts of the expert study, in the final stage, participants are asked to rate the realism of our Orgs and data. In addition to providing ratings, they also need to provide rationales for their ratings. An example question is shown in [Figure 11](https://arxiv.org/html/2411.02305v1#A6.F11 "In Part 3 ‣ F.2 The study ‣ Appendix F Expert Study Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").

Below, we provide the rating and descriptions for participants to choose from.

Org ratings:

*   •

    Very Unrealistic: The organization structure and setup felt highly artificial, with no resemblance to typical Salesforce setups.

*   •

    Unrealistic: The organization had some familiar elements, but significant parts were not convincingly structured.

*   •

    Neutral: The organization felt somewhat realistic, with a mix of plausible and implausible elements.

*   •

    Realistic: The organization largely mirrored a real-world Salesforce setup, with minor inconsistencies.

*   •

    Very Realistic: The organization felt entirely authentic, closely resembling a real-world Salesforce configuration.

Object ratings:

*   •

    I don’t know/I’m not familiar with the object.

*   •

    Very Unrealistic: The objects seemed fundamentally flawed or invented with little regard for typical Salesforce objects.

*   •

    Unrealistic: The objects had recognizable features but were generally not representative of actual Salesforce objects.

*   •

    Neutral: The objects were moderately realistic, combining elements of both realistic and unrealistic features.

*   •

    Realistic: The objects were mostly realistic and aligned well with typical objects used in Salesforce, with minor issues.

*   •

    Very Realistic: The objects felt entirely authentic and perfectly matched real-world Salesforce objects.

![Refer to caption](img/757b98b0f3e939b30c5788071e902d02.png)

Figure 11: An example question for the part 3 of our expert study.

### F.3 Qualitative Feedback

In [Table 14](https://arxiv.org/html/2411.02305v1#A6.T14 "In F.3 Qualitative Feedback ‣ Appendix F Expert Study Details ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments"), we present qualitative feedback and rationale from the experts we interviewed, as they determine whether our Organization and Object are perceived as Realistic or Unrealistic.

| Rated Instance | Rating | Rationale |
| Org | Realistic | 1\. This is really similar to what a normal Salesforce instance looks like (i.e. the one we use at our company). However, there are a few missing details in some of the pages like when you click into a contact or account. |
| 2\. It feels like my usual Salesforce Dashboard for my current job, I could more or less get a feel for the general navigation of the simulation. |
| 3\. This is what salesforce looks like for me to find case numbers and information about each of the cases that were indentified by customers. |
| 4\. Knowing nothing about the org I was able to fumble my way around and find what I needed to. |
| Unrealistic | 1\. The lack of customer data/information filling out the rest of the fields. There is no semblance of a system that’s been “worked in” and everything feels very empty and confusing with nothing to fill the interface. |
| Object | Realistic | 1\. Case management, customer interactions, knowledge base, and the transcripts were what made it realistic. |
| 2\. I think the email correspondence wasn’t perfect, but it did feel rather authentic. |
| 3\. I feel like the cases and customers issue are real life issue so I feel like they are realistic. |
| 4\. They have similar details and structures as a typical salesforce deployment (at least in my company). A lot of those elements have the same fields that are in their expected places (like Details, additional context on the right side) |
| Unrealistic | 1\. The unrealistic ones are finding the agent information. This is unrealistic because I should be able to filter and find each of the agent transfers and handle time with the customers. |

Table 14: Example rationales provided by domain experts for their ratings of our sandbox environment’s realism.

## Appendix G Implementation Details

We use the OpenAI API for the gpt models; Amazon Bedrock API for the claude models; and the Together API for the llama3.1 models. Below we provide the version of the model we tested:

*   •

    gpt-4o: gpt-4o-2024-08-06

*   •

    gpt-3.5-turbo: gpt-3.5-turbo-0125

*   •

    claude-3.5-sonnet: anthropic.claude-3-5-sonnet-20240620-v1:0

*   •

    claude-3-sonnet: anthropic.claude-3-sonnet-20240229-v1:0

*   •

    llama3.1-405b: meta-llama/Meta-Llama-3.1-405B-Instruct-Turbo

*   •

    llama3.1-70b: meta-llama/Meta-Llama-3.1-70B-Instruct-Turbo

We choose the ReAct setting over Plan based approaches that decompose the task into more manageable steps as prior works showed that in SQL based database querying tasks, planning strategy is less flexible to altering its plan by adjusting to execution feedback Yang et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib18)). We set the max actions for each instance to 20, temperature to 0, and top_p to 1 for all experiments.

 Functionality Dependency Function Task Query Independent get_order_item_ids_by_product(product_id) MTA get_order_item_ids_by_product(product_id) NCR search_products(search_term) NED get_account_id_by_contact_id(contact_id) NED Query Dependent get_non_transferred_case_ids(start_date, end_date) HTU get_cases(start_date, end_date, agent_ids, case_ids, NCR get_cases(start_date, end_date, agent_ids, case_ids, BRI get_cases(start_date, end_date, agent_ids, case_ids, HTU Calculation Independent get_start_date(end_date, period, interval_count) TCU get_start_date(end_date, period, interval_count) BRI get_start_date(end_date, period, interval_count) TII get_period(period_name, year) TCU Calculation Dependent calculate_region_average_closure_times(cases) BRI get_qualified_agent_ids_by_case_count(agent_handled_cases, n_cases) TCU calculate_average_handle_time(cases) HTU get_agents_with_max_cases(subset_cases) NCR 

Table 15: The list of functions and tasks tested in [Table 4](https://arxiv.org/html/2411.02305v1#S3.T4 "In How does the type of function affect model performance? ‣ 3.3 Discussions ‣ 3 Benchmarking Experiments ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments").