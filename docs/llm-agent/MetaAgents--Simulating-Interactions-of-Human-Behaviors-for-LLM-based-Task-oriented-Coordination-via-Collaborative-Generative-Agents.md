<!--yml
category: 未分类
date: 2025-01-11 13:03:51
-->

# MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents

> 来源：[https://arxiv.org/html/2310.06500/](https://arxiv.org/html/2310.06500/)

Yuan Li
University of Cambridge
yl967@cam.ac.uk &Yixuan Zhang
William & Mary
yzhang104@wm.edu &Lichao Sun
Lehigh University
lis221@lehigh.edu

###### Abstract

Significant advancements have occurred in the application of Large Language Models (LLMs) for various tasks and social simulations. Despite this, their capacities to coordinate within task-oriented social contexts are under-explored. Such capabilities are crucial if LLMs are to effectively mimic human-like social behavior and produce meaningful results. To bridge this gap, we introduce collaborative generative agents, endowing LLM-based Agents with consistent behavior patterns and task-solving abilities. We situate these agents in a simulated job fair environment as a case study to scrutinize their coordination skills. We propose a novel framework that equips collaborative generative agents with human-like reasoning abilities and specialized skills. Our evaluation demonstrates that these agents show promising performance. However, we also uncover limitations that hinder their effectiveness in more complex coordination tasks. Our work provides valuable insights into the role and evolution of LLMs in task-oriented social simulations.

![Refer to caption](img/96a44c3368447f610938ffe3c9234e8b.png)

Figure 1: MetaAgents: Collaborative generative agents exhibit human-like behaviors and capabilities in task-solving. In this work, we situate collaborative generative agents in a job fair context recruiting capable agents for team projects. This job fair context enables collaborative generative agents to demonstrate behaviors, including interviewing, recruiting, and coordination.

## 1 Introduction

Large language models (LLMs), such as ChatGPT  [[25](#bib.bib25)] and GPT-4  [[26](#bib.bib26)], have garnered significant attention due to their exceptional abilities in natural language processing. Recent studies extend the scope of these LLM models beyond mere text generation, positioning LLMs as versatile agents capable of conversational engagement, decision-making, and task completion [[45](#bib.bib45)]. A noteworthy development in this domain is the concept of *Generative agents*  [[29](#bib.bib29)], which leverages LLMs to emulate a broad spectrum of human behaviors, ranging from daily planning and conversational interaction to emergency response and recollection of past experiences. However, a pivotal aspect of human behavior that remains relatively unexplored in current simulations is the capacity for collaboration. Collaboration is a cornerstone of human collective intelligence and has been instrumental in shaping human societies [[42](#bib.bib42)]. As such, incorporating collaboration into social simulations not only heightens their realism but also yields valuable insights into the complexities of human interaction and collaborative behavior.

Effective collaboration requires both individual task-solving abilities and well-coordinated interactions among participants [[49](#bib.bib49); [34](#bib.bib34)]. Previous studies have mainly concentrated on the task-solving abilities of LLM-based Agents, exemplified by platforms like Auto-GPT [[38](#bib.bib38)] and AgentGPT [[32](#bib.bib32)]. Such agents are designed to autonomously propose and execute action sequences to complete tasks. In addition, some researchers have explored multi-agent setups for task-specific objectives [[9](#bib.bib9); [11](#bib.bib11); [18](#bib.bib18); [14](#bib.bib14)]. For example, Qian et al. [[31](#bib.bib31)] and Hong et al. [[14](#bib.bib14)] present multi-agent frameworks for autonomous software development, where agents play different roles in a predetermined workflow, such as programmer and project manager. However, existing approaches are often limited by their adherence to human-prescribed workflows, thereby constraining the agents’ adaptability and flexibility in responding to unforeseen challenges. This rigidity inhibits the full potential of multi-agent systems. Therefore, it is imperative to augment generative agents with both robust task-solving capabilities and dynamic coordination skills.

In this paper, we propose MetaAgents that delves into the coordination capabilities of LLM-based agents within the framework of generative agents. Specifically, we construct a specialized simulation environment that augments generative agents with communication and collaboration abilities. To this end, we simulate an environment modeled after a a job fair, a setting ripe for complex social interactions such as interviewing, recruiting, and coordinating. This conversation-rich context serves as an ideal testbed for examining how effectively these generative agents can communicate, extract pertinent information, and collaborate. To prepare generative agents for collaboration in the environment mentioned above, we propose a generative agent framework encompassing four key modules: perception, memory, reasoning, and execution. The perception module enables generative agents to receive information from a dynamic environment; the memory module allows them to store and retrieve memories, including past observations and their thoughts; the reasoning module provides them with abilities to generate plans, make reflections, and update goals; while the execution module enhances generative agents’ capacities by utilizing external resources or functions as part of their skill sets. Anchored by this multi-faceted framework, collaborative generative agents not only emulate human-like behaviors but also exhibit progressively enhanced capabilities in complex tasks. The difference between MetaAgents and prior works are shown in Table [1](#S2.T1 "Table 1 ‣ 2 Related Works ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents").

Our investigation centers on two pivotal dimensions. First, we assess the extent to which these agents can form cohesive teams with the goal of completing specific tasks. Second, we explore whether these agents can dynamically create customized workflows that capitalize on the individual expertise of each team member. To this end, we evaluate generative agents across three distinct scenarios in the simulated job fair environment. Our results reveal that the agents are proficient in understanding team project workflows and can leverage information retrieved from conversations to both identify qualified collaborators and delegate appropriate tasks. However, as the complexity of the job fair increased with the addition of more participating agents, we observed that generative agents encounter increasing challenges in coordination. These challenges were primarily due to the *misalignment* of LLMs’ objectives or intentions. Importantly, our results suggest that the reasoning module in our framework plays a vital role in enhancing generative agents’ performance in task-oriented coordination.

Our contributions are summarized as follows:

*   •

    Collaborative Generative Agents: We introduce an advanced form of generative agents capable of conducting meaningful interaction and collaboration. Using a simulated job fair as a testbed, we focus on investigating their coordination behaviors in real-world-like scenarios.

*   •

    Comprehensive Framework for Collaboration: We propose a multi-module framework that arms generative agents with the essential faculties needed for effective collaboration. This architecture draws upon the internal reasoning capabilities of LLMs and integrates external functionalities to enable perception, memory storage and retrieval, decision-making, and action execution.

*   •

    In-depth Evaluation: We perform a rigorous evaluation of these generative agents in terms of their information retrieval and coordination capabilities. Our analysis uncovers both the challenges they face and their potential for effective collaboration in complex social settings.

## 2 Related Works

Table 1: Comparison with previous LLM multi-agent systems.

|  | Memory | Social Simulation | Coordination Behaviors | Task-Solving |
| Multi-Agent Debate  [[11](#bib.bib11); [18](#bib.bib18)] | ✘ | ✘ | ✘ | ✘ |
| BabyAGI  [[3](#bib.bib3)] | ✘ | ✘ | ✘ | ✔ |
| Camel  [[17](#bib.bib17)] | ✘ | ✘ | ✘ | ✔ |
| Chadev  [[31](#bib.bib31)] | ✔ | ✘ | ✘ | ✔ |
| MetaGPT  [[14](#bib.bib14)] | ✔ | ✘ | ✘ | ✔ |
| S${}^{3}$  [[12](#bib.bib12)] | ✔ | ✔ | ✘ | ✘ |
| Generative Agents  [[29](#bib.bib29)] | ✔ | ✔ | ✔ | ✘ |
| MetaAgents (Ours) | ✔ | ✔ | ✔ | ✔ |

### 2.1 Autonomous Agents

Prior research has delved into the use of Large Language Models (LLMs) as autonomous agents capable of executing tasks via self-directed planning and actions [[45](#bib.bib45)]. One notable example is Auto-GPT [[38](#bib.bib38)], which automates task completion by breaking down tasks into manageable sub-tasks, supplemented by web searches and information gathering. Another relevant contribution is WorkGPT [[43](#bib.bib43)], which also serves as an agent framework. Upon receiving an instruction, WorkGPT engages in iterative dialogues with LLMs to successfully execute the task as specified. In addition, there are open-source endeavors such as GPT-Engineer [[1](#bib.bib1)], SmolModels [[40](#bib.bib40)], and DemoGPT [[22](#bib.bib22)] that focus on automating code generation. These platforms use specifically-crafted prompts to assist in software development tasks. While these autonomous agents have demonstrated impressive capabilities in certain domains, they still face challenges in expanding their competencies for tasks requiring more advanced forms of reasoning.

### 2.2 Multi-agent System

Several recent studies have employed multiple LLMs as agents within systems, aiming to augment their reasoning abilities through inter-agent discussions [[48](#bib.bib48); [11](#bib.bib11); [18](#bib.bib18)]. BabyAGI [[3](#bib.bib3)] serves as an autonomous task management system that dynamically generates, prioritizes, and executes tasks. Camel [[17](#bib.bib17)] utilizes role-play to encourage structured conversations among agents, facilitating complex problem-solving. Chatdev [[31](#bib.bib31)] repurposes LLMs to assume distinct roles in software development, encompassing stages such as design, coding, testing, and documentation. Taking the concept further, MetaGPT [[14](#bib.bib14)] introduces a sophisticated programming workflow to structure teamwork among LLM-based agents. Additionally, multi-agent frameworks have been employed to study social phenomena. Park et al. [[29](#bib.bib29)] have designed a community of 25 generative agents capable of planning, communicating, and forming connections. In a similar vein, AgentSims [[19](#bib.bib19)] offers a detailed virtual town, populated by agents with capabilities like planning and tool use, serving as a platform to study social skills. Furthermore, RecAgent [[46](#bib.bib46)] simulates a recommendation ecosystem featuring various types of agents, including recommenders and interactive user agents. Liu et al. [[20](#bib.bib20)] explore ethical training and value alignment in agents through social interactions within a simulated society. Despite these advancements, a lingering challenge in multi-agent systems is the lack of a unified framework for both autonomous coordination and task completion. To address this gap, this paper aims to integrate both coordination and task-solving capabilities in generative agents, enabling them to collaborate seamlessly from project inception to completion.

## 3 Study Design

To facilitate the seamless collaboration of multiple LLM-based agents working on a shared task, this paper introduces a simulated environment inspired by job fairs. This novel simulation allows us to rigorously assess LLM-based agents in terms of their information processing, retrieval, and coordination capabilities. In addition, the environment aids in exploring the underlying principles of collaboration and decision-making in complex social settings. In real-world job fairs, employers convene to interact with prospective employees possessing specialized skills. Through engagement with a diverse talent pool, employers evaluate skills against project requirements, thereby aiding in the selection of team members best suited for collaborative efforts. Similarly, LLM-based agents should actively participate in dialogues within the simulated job fair, distilling insights that guide their decisions on team formation.

### 3.1 Configuration - Job Fair

The job fair features two main types of generative agents: recruiting agents and job-seeking agents. Their interactions are shaped by a mutual selection procedure, as illustrated in  [Figure 2](#S3.F2 "Figure 2 ‣ 3.1 Configuration - Job Fair ‣ 3 Study Design ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents"). Job-seeking agents begin by reading company posters that introduce each company’s culture and missions, and then determine which recruiter(s) to approach. Should a job-seeking agent express interest in a company, they initiate a conversation with its recruiting agent. These discussions help recruiters gauge the abilities of potential candidates. After the conversation, recruiting agents retain relevant information in their memory. Once the job fair concludes, the recruiting agents formulate the workflow for the company, assigning recruits to appropriate roles. We will discuss this arrangement in detail in [subsection 3.3](#S3.SS3 "3.3 Scenarios ‣ 3 Study Design ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents").

![Refer to caption](img/8e87f33817a65e16b27eedca71dd153a.png)

Figure 2: The interactions between recruiting agents and job-seeking agents are shaped by a mutual selection procedure.

We authored some keywords entered into the prompt and instructed LLM to create short biographies for each agent, including details of their personality, skills, and current employment status (e.g., working as [position name] at [company name] or seeking new job opportunities). To examine the communication effectiveness of generative agents and their coordination abilities, we made the following arrangement: 1) Each agent’s biography was stored in its initial memory. This arrangement ensured that recruiting agents only acquired information through conversations with the corresponding agent. 2) While emphasizing the expertise of job-seeking agents, we avoided explicitly listing the exact job titles they would be suitable for since we expect LLM to propose the job titles and corresponding responsibilities.

Example: Yohan Henderson is a candidate for a project manager role, who has the following biography:

Prompt Input:

Provide a brief profile description of Yohan Henderson given the following details: Key attributes: adaptable, strategic, communicative; Professional skills: product design, strategic planning, innovation, project management. (Note: Do not mention ”project manager”); Track record: Effectively oversees projects from inception to completion; Current situation: Seeking new working opportunities.

Output:

Yohan Henderson is an adaptable, strategic, and communicative person. He is a skilled professional with experience in product design, strategic planning, innovation, and project management. He possesses a proven track record of effectively overseeing projects from inception to completion. His strategic thinking and innovative approach consistently yield impactful results. Yohan is seeking new working opportunities.

We also requested LLM to generate one-paragraph descriptions of companies participating in the job fair, introducing the companies’ strengths and their missions. These introductory descriptions are accessible to job-seeking agents. For example, this is a description of company❶:

Company 1 is a start-up aiming at delivering advanced software solutions that cater to a diverse range of needs. With expertise finely honed in software development, Company 1 has consistently engineered products that uphold high standards of functionality and reliability. The company’s unwavering commitment to quality guarantees seamless integration of software components and streamlined user experiences. Company 1’s mission is firmly rooted in leveraging technology to optimize processes and elevate user interactions. Through a meticulous approach to development, the company places a premium on innovation, precision, and achieving measurable outcomes.

### 3.2 Workflow Introduction

Companies manage to enhance efficiency and ensure consistent productive outcomes through adopting standard workflows in practice [[41](#bib.bib41)]. To this end, we expect teams of generative agents to collaborate in a similar manner. We now present the standard workflow for three companies participating in the job fair. Three companies are software development company ❶, data analysis company ❷, and advertising company ❸. All companies intend to recruit a team for their core business, which are software development, data analysis, and advertising poster design. We summarized the workflow of each team in  [Table 2](#S3.T2 "Table 2 ‣ 3.2 Workflow Introduction ‣ 3 Study Design ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents").

Table 2: Standard workflow and personnel required for three teams at job fair.

|  | Team 1 | Team 2 | Team 3 |
| Scenario | 1, 2, 3, 4 | 3, 4 | 3, 4 |
| Company | ❶ | ❷ | ❸ |
| Standard Workflow | Designing $\downarrow$ Coding $\downarrow$ Testing $\downarrow$ Documenting | Problem Formulation $\downarrow$ Data Acquisition $\downarrow$ Data Exploration $\downarrow$ Model development $\downarrow$ Model Evaluation $\downarrow$ Presentation | Brief Creating $\downarrow$ Copywriting $\downarrow$ Graphic Design |
| Personnel Required | Project Manager, Programmer, Code Tester, Artistic Designer | Data Engineer, Data Scientist | Content Strategists, Copywriters, Graphic Designers |

For software development team, to ensure the prospective software development team’s effectiveness, we assign a workflow analogous to that described in Chatdev [[31](#bib.bib31)], which adheres to the waterfall model of software development. This model consists of four sequential phases: Design, Coding, Testing, and Documentation [[5](#bib.bib5)], as detailed in [Table 2](#S3.T2 "Table 2 ‣ 3.2 Workflow Introduction ‣ 3 Study Design ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents"). The Design phase focuses on ideation and the translation of general ideas into technical design requirements. During the Coding phase, team members engage in code development and code review. The Testing phase involves integrating all components into a cohesive system, followed by code validation and debugging activities. Lastly, the Documentation phase encompasses the creation of technical specifications and user manuals. In operationalizing this software development cycle, we identify five key roles: Team Leader, Project Manager, Programmer, Code Tester, and Artistic Designer. We allocate a distinct agent to each role, representing them as highly qualified candidates for their respective positions. In this scenario, excluding the team leader, the remaining four agents actively seek employment. Upon the completion of all dialogic interactions, the team leader is expected to formalize a workflow that aligns with the waterfall model, assigning candidates to their designated roles accordingly.

Next, we present the standard workflows of the data analysis team in company ❷. A data analysis project includes a series of phases: problem formulation, data acquisition, data exploration, model development, model evaluation, and presentation  [[24](#bib.bib24); [50](#bib.bib50); [44](#bib.bib44)]. Problem formulation marks the start of the data analysis workflow, and it usually involves the team discussion about the problem led by the team leader. Subsequently, data acquisition encompasses gathering relevant data, and formatting and cleaning it. Following this, data exploration involves examining the data to uncover patterns, anomalies, and potential insights. Model development entails learning and generalizing algorithms on the training data set. This is succeeded by model evaluation, a phase dedicated to validating the model’s validity and its ability to generalize beyond the training data. The presentation phase involves communicating findings, conclusions, and stories to various stakeholders. To ensure efficient delegation, the data engineer is assigned to data acquisition and cleansing, while the data scientist is responsible for data exploration, model development, and model evaluation. Finally, the team leader and data scientist will undertake the presentation phase.

The advertising poster design team in ❸ follows a streamlined creative content generation workflow as the ground truth: brief creating, copywriting, and graphic design [[23](#bib.bib23)]. The brief creating phase defines the poster’s objectives, target audience, and stylistic preferences. The copywriting phase involves drafting and refining the text elements. Lastly, graphic design entails developing the visual layout and overall aesthetics. The corresponding roles for this workflow are content strategists, copywriters, and graphic designers.

### 3.3 Scenarios

Within the job fair context outlined in Section [3.1](#S3.SS1 "3.1 Configuration - Job Fair ‣ 3 Study Design ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents"), we primarily present four distinct scenarios designed to investigate the coordination behaviors of generative agents in  [Figure 3](#S3.F3 "Figure 3 ‣ 3.3 Scenarios ‣ 3 Study Design ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents").

![Refer to caption](img/79a55b219754d11d545d6f713bcf6d63.png)

Figure 3: Summary of Four Job Fair Scenarios.

#### 3.3.1 Scenario 1

In the context of the job fair described, we first present a basic recruitment case featuring a single recruiting agent without the presence of any redundant job-seeking agents. For instance, consider a recruiting agent named Tyler Zeller, who leads the software development team at the company ❶. In this case, Tyler aims to recruit team members capable of transforming a preliminary concept into a fully-realized software product.

#### 3.3.2 Scenario 2

In Scenario 1, all job-seeking agents possess skills critical for the success of the envisioned software development team. This leads to a coming question: How does the recruiting agent handle encounters with individuals who lack the qualifications needed for team contributions? Can the agent effectively identify and select qualified job-seekers while concurrently filtering out unsuitable candidates? In order to answer above questions, Scenario 2 aims to delve deeper into these facets of coordination abilities.

Building on Scenario 1, Scenario 2 introduces additional job-seeking agents into the job fair simulation for a more nuanced exploration of the recruiting process. Among these additional agents is an undergraduate student majoring in mathematics—a discipline with limited immediate applicability to software development. While this student is not actively seeking employment, they are interested in gathering information for future career planning. Another new participant is an advertising specialist. Given that the recruiting agent’s focus is exclusively on software development roles, neither the student nor the advertising specialist is a suitable candidate for the available positions. In this context, the challenge for recruiting agents increases in complexity. Beyond merely constructing an effective workflow and managing team members for task completion, recruiting agents must also exercise discernment in selecting appropriate candidates while filtering out those who do not align with the team’s objectives.

#### 3.3.3 Scenario 3

In previous two scenarios, job-seeking agents are limited to a single employment option because of the presence of only one recruiting agent. In contrast, Scenario 3 better mirrors a real-world job fair environment, where we introduced three recruiting agents representing distinct teams so job-seeking agents can pick their preferred teams. In Scenario 3, the recruiting agents are the leaders of the software development team, the data analysis team, and the advertising poster design team, separately. Nine job-seeking agents actively participate in the job fair. This scenario increases the difficulty level for job-seeking agents since they need to make a decision to pick a company aligned with their interests and skills. To simplify, we ensured minimal skills overlap among job-seeking agents. The operational process for Scenario 3 is as follows: job-seeking agents are guided to pick their preferred companies. Subsequently, after job-seeking agents finalize their choices and finish their interviews, the recruiting agents are responsible for determining the roster of recruited members and designing the workflow for their teams. In Scenario 3, job-seeking agents are restricted to selecting just one team for an interview.

#### 3.3.4 Scenario 4

To evaluate the performance of generative agents across different levels of complexity, we construct a more complex settings. Scenario 4 allows job-seeking agents to interview with a random number of teams, ranging from one to three. This scenario presents a more challenging environment for the agents’ information retrieval and coordination skills due to an increase in the total rounds of interviews. This implies that recruiting agents must make recruitment decisions from a larger pool of candidates.

### 3.4 Example Recruitment Process from Recruiting Agents

Starting with initial goals and biographies, agents came together at the job fair venue. Over time, their behaviors developed as they interacted, reflected on past experiences, formed memories, and made decisions. We demonstrate coordination behaviors of generative agents by tracing our system output from the beginning of the job fair to the conclusion of designing the strategy for agent Tyler Zeller, the leader of the software development team at company ❶.

Before engaging in conversations, Tyler brought up his initial plan, outlining his expectation for the team:

I am thinking about hiring a team of software developers and engineers with expertise in different areas, such as front-end development, back-end development, and UI/UX design. I also need to recruit a project manager who can oversee the entire development process and ensure timely delivery. Additionally, I would like to hire a quality assurance specialist to conduct thorough testing and ensure the software meets high standards of functionality and reliability. Finally, I need to find a technical writer who can document the software and create user guides for seamless user experiences.

This initial plan decomposes tasks, which is a preliminary step to transform a rough idea to actionable steps. With this plan in mind, he initialized the conversation when he saw a potential candidate approaching him:

Tyler Zeller: Hello! Welcome to company 1\. I’m Tyler, the leader of the software development team. How can I help you today?

Yohan Henderson: I’m Yohan Henderson, a skilled professional with experience in product design, strategic planning, innovation, and project management. I’m impressed by company 1’s commitment to quality and leveraging technology for transformative impact. I believe my skills and expertise would be a great fit for your team. I would love to learn more about the specific opportunities available at company 1.

…

After the conversation with Yohan, Tyler made the following reflection:

During my conversation with Yohan Henderson, I was impressed by his strong background in product design, strategic planning, and project management. His skills and experience align perfectly with our mission at company 1\. I believe he would make a significant contribution to our current project, which focuses on optimizing data analysis for businesses. Yohan’s experience in managing projects from inception to completion and ensuring seamless integration with existing systems would ensure the successful delivery of a high-quality user experience and reliable data insights. I look forward to working with Yohan and harnessing technology for transformative impact at company 1.

This reflection after the conversation helped Tyler better evaluate the current situation and how he can proceed with the goal. He continued to speak with a few more candidates, collecting information to match them with appropriate roles on his team.

Finally, Tyler Zeller determined a team of agents he would like to recruit, and designed the workflow for the software development team outlined below: (1) Brainstorm and ideation, (2) Design and prototyping, (3) Software development (4) Quality assurance and testing and (5) User manual and documentation. The workflow proposed by Tyler aligns with the standard waterfall model for software development. In the proposed workflow, stages (1) and (2) correspond to the designing phase in the waterfall model; stage (3) corresponds to the coding phase; stage (4) aligns with the testing phase; and stage (5) matches the documenting phase. Tyler assigned capable candidates to their respective roles. Moreover, to promote brainstorming and discussions, he adeptly incorporated the team leader and project manager into stages (1) and (2) of the process.

## 4 Framework of MetaAgents

MetaAgents aim to provide a prototype for multi-agent systems, focusing on coordination and collaboration behaviors. This prototype views them as entities with distinctive skills and underscores the importance of information retrieval and effective coordination among generative agents. Underlying their interactive behaviors at the job market is a novel generative agents framework that integrates LLM with mechanisms for various cognitive processes and execution functions, ensuring that their behaviors align with experiences and their capabilities.

Our framework is inspired by prior work in HCI that explores the role of agents in mimicking human behaviors  [[35](#bib.bib35); [15](#bib.bib15)]. Card et al. [[8](#bib.bib8)] introduce a cognitive modeling approach, the Model Human Processor, which incorporates perceptual, cognitive, and motor systems and memory storage. Similarly, Laird et al. [[16](#bib.bib16)] design perceive-plan-act cycles that enable agents to dynamically perceive the environment and match it with one manually crafted action procedure. Recently, Park et al. [[29](#bib.bib29)] introduced a generative agents architecture that combines LLMs with cognitive mechanisms such as planning, reacting, and reflecting. Existing literature primarily concentrates on the cognitive dimension of agents and involves actions in the form of human motions. However, it is worth noting that these agents often lack yielding productive outcomes.

Unlike previous works, our proposed MetaAgents focus on agents’ coordination capabilities, which unlock the potential for generative agents to collaboratively tackle specific tasks and steer generative agents to achieve collaborative outcomes by leveraging the varied skills of each agent. Our framework consists of a perception module, a memory module, a reasoning module, and an execution module. We illustrate this framework in  [Figure 4](#S4.F4 "Figure 4 ‣ 4 Framework of MetaAgents ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents"), and discuss details in the following.

![Refer to caption](img/1f9be9be2434594402739db59fe22081.png)

Figure 4: An overview of MetaAgents, consisting of perception module, the memory perception, reasoning module, and execution module. The reasoning module of generative agents. It has cognitive functions including plan generation, reflection, and goal updates. Generative agents retrieve external functions in the skill pool as their skill sets. Each generative agent has a distinctive skill set.

### 4.1 Perception Module

The Perception Module serves as a critical component in the architecture of collaborative generative agents, facilitating their interaction with and understanding of their external environment. This module enables agents to actively gather data from their surroundings and form detailed observations. These observations, in turn, feed into the Memory Module, providing the essential data needed for informed decision-making.

For example, when job-seeking agents attend a job fair, they primarily rely on the Perception Module to collect valuable information from various sources such as company posters and other promotional materials. In addition, this module also allows generative agents to assess the current engagement levels of their peers. Therefore, an agent can determine whether another agent is available for interaction or is already engaged in a conversation.

### 4.2 Memory Module

The Memory Module in collaborative generative agents functions as a comprehensive repository for storing their historical experiences. Initially, the module holds concise biographies and personal goals of the agents as starting data points. As agents accumulate experiences over time, the Memory Module becomes enriched with a broader array of information, including past dialogues, contextual elements, personal reflections, and insights.

Memory retrieval is crucial for rational decision-making, significantly influencing behavioral consistency. Park et al. [[29](#bib.bib29)] highlight a challenge in memory retrieval emanating from the complex task of reasoning over a vast array of stored experiences. This complexity can potentially clutter the model and breach the limitations of the context window. They suggest a scoring mechanism to rank memories based on their recency, relevance, and importance, selectively fitting the top-ranked memories into LLMs’ context window.

However, this memory retrieval approach falls short in conversation-heavy settings like a job fair. Directly inserting extended multi-turn dialogues into the prompt window is impractical. Furthermore, summary attempts risk introducing errors due to inaccurate paraphrasing or omission of vital keywords. For example, a condensed version of Yohan Henderson’s rich professional background—spanning product design, strategic planning, innovation, and project management—might get oversimplified to “skilled design, planning, management.” Such simplifications could lead the team leader to misinterpret Yohan’s capabilities, miscasting him as a designer rather than a product manager.

To overcome this issue, MetaAgents introduce an intuitive memory retrieval mechanism that mimics human-like recall processes. Unlike traditional methods that attempt to retrieve every detail, human memory often retains only significant key phrases from a conversation. Guided by this observation, we instruct agents to extract two categories of information: 1) the overarching theme and context of the conversation, and 2) key terms or standout words. We configure agents to generate a hybrid of both summary and highlighted terms within conversations. This mechanism ensures a more accurate retrieval of memories, effectively feeding into the prompt window and bolstering the consistency of the reasoning process.

### 4.3 Reasoning Module

The Reasoning Module serves as the intellectual core or the ”brain” of collaborative generative agents, enabling them to undertake complex cognitive tasks like reasoning, decision-making, and updating their thoughts or beliefs. After relevant information is culled from the Memory Module, it is fed into the Reasoning Module to guide these cognitive processes. To facilitate these functions, we adapt agents using structured prompts designed to steer specific cognitive tasks. Our Reasoning Module is equipped to handle a diverse array of cognitive functions, including but not limited to:

Plan generation. Planning is a vital aspect of reasoning, wherein the reasoning module not only generates individual plans for actions but also makes plans for a group of people, such as team leaders developing workflows. For example, the software development team leader Tyler has a rough idea of the team project. The planning function enables him to transform the idea into an initial plan such that he will be clear about who to recruit at the job fair.

Goal updates. Since our framework aims at steering generative agents in task resolution, goals are essential for motivating agents toward actions. As the context changes and work progresses, modifying existing goals could result in more streamlines and goal-oriented actions. For example, Tyler could adjust his recruiting focus after meeting ideal candidates. While he started with the broad aim of “assembling a software development team,” he could refine his recruitment emphasis after securing a standout candidate for a programming position.

Reflection. Reflection, an effective technique for enhancing the reasoning prowess of LLM-based Agents, involves prompting LLM to contemplate acquired information. Further, it offers recruiting agents an opportunity to reconsider their decisions when designing workflows, which provides validation or alternative perspectives. For instance, Tyler initially included an advertising specialist in his devised workflow for the software development team. Through reflection, he realized that advertising was unnecessary for the software development life cycle. The reasoning module’s outcomes are then stored in the memory stream, while directives are passed to the execution module.

We now illustrate how these three functions contribute to goal-oriented reasoning and foster coordination and collaboration. The interplay of plan generation, reflection, and goal update functions resembles the human thinking process. Initially, an agent begins by generating plans to achieve a goal. This plan is then entered into the reflection function. The reflection function evaluates the feasibility of the plan and determines if collaboration is required. Based on the outcomes of this reflection, the goal updates function might adjust the plan, either prioritizing the search for suitable collaborators or continuing with the current goal. For agents focused on finding employment as their primary goal, they may plan to attend job fairs or connect with potential employers for interviews.

### 4.4 Execution Module

The execution module enables collaborative generative agents to take actions and engage in production processes. This module enhances collaborative generative agents’ abilities by endowing them with specific skills. These skills are virtually stored in a skill pool in the form of functions, and can be programmed by humans or other models. Each generative agent possesses a distinctive set of skills. For example, programmers can utilize functions such as writing and testing code as part of their skill set, while writing user manuals function may not be accessible to them since they are likely to be unfamiliar with product management. The varied skill sets of generative agents enrich collaboration, enabling collective efforts to accomplish tasks that might be challenging for a single agent.

## 5 Evaluation Results

We evaluated and analyzed the coordination behaviors of generative agents within the context of the job fair. In this section, we first introduce the setup of our evaluation, including experimental setup, and three criteria for the overall success of coordination. We then present results for the overall success rate and collaborative generative agents’ performance on three criteria.

### 5.1 Setup

#### 5.1.1 Experimental Setting

In our experiments, We repeated both Scenario 1 and Scenario 2 for 100 times. For Scenario 3 and Scenario 4, we ran 50 times each. For large language models, we utilized the gpt3.5-turbo-16k version of ChatGPT  [[25](#bib.bib25)]. We set the language model temperature to 0.5, balancing controlled generation with rooms for creativity in conversations.

#### 5.1.2 Criteria for Evaluating Coordination Behaviors

Given that we have standard workflows as the ground truth, each scenario has the correct answer for the optimal team composition and the proper workflow for the team project. We first define the prerequisites for an effective collaboration: First, the team should operate cost-effectively. As a result, it is critical to avoid recruiting agents who do not possess the skills to contribute to the teamwork. Second, successful collaboration requires the correct sequence for the workflow, as each team workflow involves dependencies where the completion of steps relies on the completion of preceding steps.

Following the prerequisites for successful team coordination, we introduce three criteria to evaluate the coordination behaviors. The overall success for coordination is achieved when satisfying these three criteria simultaneously:

*   •

    Criterion 1: Accurate identification of capable job-seeking agents (Identification). Criterion 1 involves correctly identifying agents with skills necessary for the team. We evaluate this criterion through list matching, wherein we compare the list of recruited agents with the ground truth.

*   •

    Criterion 2: Appropriate workflow design (Design). Criterion 2 pertains to the proper workflow design for team projects. Due to the diversity of language generated by LLM, this criterion is evaluated by our research team. To illustrate, if the proposed workflow includes a stage such as “software development” or “programming,” we equate these stages with the coding phase in the standard waterfall model.

*   •

    Criterion 3: Correct alignment of agents with their roles (Alignment). Criterion 3 assesses whether the selected generative agents are appropriately matched with their designated roles in the workflow. The success of satisfying criterion 3 is defined as having capable agents assigned to their corresponding positions. Criterion 3 is also evaluated by researchers. It is worth noting that the success rate for Criterion 3 can be influenced by Criteria 1 and 2\. Failure to include competent agents or omit phases in the workflow will inevitably lead to mismatches between capable agents and their designated roles. However, an independent factor will also affect Criterion 3, which we call misplacement, i.e., an agent meant for one phase is mistakenly assigned to another.

### 5.2 Evaluation Results

#### 5.2.1 Overall Success Rate

As shown in  [Figure 5](#S5.F5 "Figure 5 ‣ 5.2.1 Overall Success Rate ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents"), collaborative generative agents achieve an overall success rate 70% in Scenario 1\. It suggests their proficiency in effectively retrieving information through communication and accurately matching job-seeking agents with the appropriate workflows. The overall success rate diminishes to 53% in Scenario 2\. The decline in performance stems from introducing two additional job-seeking agents lacking the requisite skills. In Scenario 3, the overall success rate is 42%, while this rate drops to 16% in Scenario 4\. These results demonstrate that collaborative generative agents could generally retrieve information from conversations and coordinate team projects. However, they encountered escalating challenges as the complexity of the job fair and the number of participants increased.

To further investigate collaborative generative agents’ coordination capacities, we examine their performance on the introduced three criteria separately in each scenario (see  [Figure 5](#S5.F5 "Figure 5 ‣ 5.2.1 Overall Success Rate ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents")).

![Refer to caption](img/5c2bb5e405f722882e592a3973dabf11.png)

Figure 5: Performance of collaborative generative agents based on overall success rate and three criteria (identification, design, alignment).

#### 5.2.2 Results for Scenario 1

Scenario 1 features a recruiting agent and four job-seeking agents at the job fair. The recruiting agent endeavored to assemble a software development team. All four job-seeking agents at the job fair had the necessary skills for the success of the software development team. In this scenario, generative agents achieved a 98% success rate for Criterion 1, implying that the recruiting agent enlisted all competent agents into the team. Only 2% of instances missed out on a capable agent during recruitment. Furthermore, under Criterion 2, generative agents correctly design 82% of workflows to complete the team project. However, as shown in  [Figure 6](#S5.F6 "Figure 6 ‣ 5.2.3 Results for Scenario 2 ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents") (A), they occasionally failed to include steps in the software development workflow, such as testing (11%), documenting (4%), or both (2%). For Criterion 3, generative agents managed a success rate of 71%, which signifies their general proficiency in matching competent agents with their appropriate roles.

#### 5.2.3 Results for Scenario 2

Scenario 2 introduces two more job-seeking agents compared to Scenario 1\. However, these newly added agents lack the skills necessary for the software development team. Consequently, the ideal team composition and workflow remain identical to Scenario 1\. In this scenario, the success rate for Criterion 1 (accurate identification of capable agents) diminishes to 65%. This suggests that the additional job-seeking agents created confusion for the recruiting agent, complicating the task of assembling the optimal team. Failures arose from recruiting unneeded agents (18%), overlooking qualified agents (15%), and a combination of both (2%). For Criterion 2, generative agents correctly proposed the software development team’s workflow in 83% of instances. Meanwhile, as shown in [Figure 6](#S5.F6 "Figure 6 ‣ 5.2.3 Results for Scenario 2 ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents") (B), they made mistakes in neglecting testing (11%), designing (3%), and documenting (3%) phases. Additionally, generative agents had a success rate of 62% for pairing qualified agents with their appropriate roles (Criterion 3). This relatively moderate rate can be attributed to the omission of recruiting proficient agents and the misplacement of unqualified agents into critical roles (11%).

![Refer to caption](img/ff25775783258dc083dab6be4d09699f.png)

Figure 6: Performance of generative agents for Criterion 2 in Scenario 1 and Scenario 2

#### 5.2.4 Results for Scenario 3

Scenario 3 introduces three recruiting agents: a leader for the software development team from company ❶, a leader for the data analysis team from company ❷, and a leader for the advertising poster design team from company ❸. Each team has its workflow tailored to specific business objectives. In scenario 3, all job-seeking agents select only one team to interview. Generative agents attained a success rate of 66% for Criterion 1\. Detailed results from [Table 3](#S5.T3 "Table 3 ‣ 5.2.4 Results for Scenario 3 ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents") show that each team managed to enlist competent job-seeking agents for the team project at a rate exceeding 80%. The agents proposed the correct workflow for all three teams in 74% of instances (Criterion 2). The specific success rates for each team are 82%, 94%, and 98%, respectively. In this setting, correctly assigning roles to agents (Criterion 3) was the most challenging aspect, with a success rate of 46%. We include comprehensive experimental results of generative agents’ performance on each Criterion of Scenario 3 in [Table 3](#S5.T3 "Table 3 ‣ 5.2.4 Results for Scenario 3 ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents").

Table 3: Statistics on the performance of generative agents for each team in Scenario 3 and Scenario 4.

|  |  | Criterion 1 (%) | Criterion 2 (%) | Criterion 3 (%) | Overall (%) |
| Scenario 3 | Team 1 | 88 | 82 | 80 | 74 |
| Team 2 | 88 | 94 | 82 | 74 |
| Team 3 | 86 | 98 | 70 | 70 |
| Scenario 4 | Team 1 | 42 | 80 | 65 | 36 |
| Team 2 | 68 | 88 | 72 | 54 |
| Team 3 | 50 | 82 | 58 | 46 |

#### 5.2.5 Results for Scenario 4

Scenario 4 has the same agents as Scenario 3, while job-seeking agents have the flexibility to interview with 1 to 3 teams at the job fair. Given this configuration, meeting the standard of each criterion becomes more challenging: a criterion is only considered fulfilled if all three teams successfully meet it. In this scenario, collaborative generative agents struggled to coordinate and assemble teams to accomplish projects. The accuracy in identifying suitable job-seeking agents, as per Criterion 1, was particularly low, yielding a success rate of 20%. Success rates for Criterion 2 (appropriate workflow design) and 3 (Correct alignment of agents with their roles) also declined, falling to 58% and 32%, respectively.

To understand the factors leading to failures in task coordination, we examined all independent causes. As illustrated in  [Figure 7](#S5.F7 "Figure 7 ‣ 5.2.5 Results for Scenario 4 ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents") (A), we found that *misplacement* problem was the primary contributor to the failures. While omissions of competent agents and recruitment of redundant agents accounted for 27% and 25%. In addition, the errors in the proposed workflow made up 14% of this problem. We illustrate the factors preventing generative agents from success in this coordination task in  [Figure 7](#S5.F7 "Figure 7 ‣ 5.2.5 Results for Scenario 4 ‣ 5.2 Evaluation Results ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents")(B). In this setting, recruiting redundant agents emerged as the primary obstacle, accounting for 35% of the failures. Notably, omitting capable agents during recruitment was also a significant challenge. Misplacement issues and workflow-related problems contributed to 17% and 18% of the failures.

![Refer to caption](img/0cb0d27e015406e88e9d818092243740.png)

Figure 7: Failure Cases for Scenario 3 and Scenario 4.

#### 5.2.6 Comparison Across Scenarios

Comparing results across scenarios, we found that the success rate for accurately identifying capable agents (Criterion 1) diminished significantly with increasing job fair participants. To elaborate, as the pool of interviewees expanded, recruiting agents were more likely to make mistakes in team assembly. In contrast, they consistently proposed accurate workflows, suggesting an understanding of decomposing a general task into sequential steps. The success rate for Criterion 3 fluctuated considerably between scenarios. Beyond the evident impact of failure to recruit capable agents, misplacement consistently emerged as a cause for these fluctuations.

### 5.3 Effects of the Reasoning Module

As discussed in  [section 4](#S4 "4 Framework of MetaAgents ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents"), the reasoning module of the collaborative generative agents framework is fundamental for them to make informed decisions. This module emulates the human cognitive process and encompasses three functions: plan generation, reflection, and goal updating. In this subsection, we explore the effect of each of these functions.

#### 5.3.1 Plan Generation and Goal Update

The plan generation and goal update functions are crucial for collaborative generative agents to reason and take action toward achieving objectives. Collaborative generative agents initiate a plan through the plan generation function, enabling them to transform a broad idea into actionable steps. As the context shifts, they rely on the goal update function to adapt their initial plans to ensure objectives are up to date. In our job fair environment, the recruiting agents formulated a team assembly plan with a preliminary task breakdown. As they engage with candidates over time, their goals become more refined and aligned with reality. We demonstrate the effect of the plan generation and goal update functions with an example from Scenario 1.

Tyler, the leader and recruiting agent for the software development team, crafted an initial plan that aimed to “hire a team of software developers, a project manager, a UI/UX designer, and technical writers for generating user manuals.”

After interviewing with a project management expert Yohan Henderson, Tyler stuck with his plan to hire a team as originally envisioned. “However, after talking to Yohan, I (Tyler) am more inclined to consider him for the project manager role due to his strong background in project management and his ability to coordinate cross-functional teams. This conversation has helped me narrow down my previous thoughts and focus on finding the right candidate for the project manager position.”

The conversations between Tyler and other job-seeking agents went on. Tyler solidified his ideas when he met with agents possessing the necessary skills. A noteworthy update occurred after a discussion with George Holloway, a code testing expert. Though Tyler’s initial plan did not include a role of code testing or quality assurance, the conversation with George made him realize that he needed to include a testing phase in the workflow. “George is well-suited for the steps of developing the software solution and testing and debugging the software. His ability to optimize functionality aligns with our commitment to quality and innovation.” As a result, the goal update function serves as a complement to plan generation, enhancing and supplementing the goal to make it more comprehensive.

#### 5.3.2 Reflection

The reflection function provides generative agents with opportunities to reassess their decisions. In our case study, reflection function assists recruiting agents in re-evaluating their initial selections of job-seeking agents and refining their choices. We illustrate the effect of this function through an example from Scenario 2.

Initially, Tyler correctly assigned capable agents to project roles, including project management, software development, UX design, quality assurance, and user manual generation. He also included Benjamin Williams, a math undergraduate student, on the team with the following initial thoughts:

Benjamin Williams’ strong background in mathematics and problem-solving will contribute to the visualization aspect of the project.

However, upon reflection, Tyler reconsidered Benjamin’s inclusion:

After reflecting on the initial list of people, I have decided to exclude Benjamin Williams from the recruitment process. While Benjamin showed enthusiasm and a strong background in mathematics, his lack of experience with the specific technologies we use at company 1 may hinder his ability to contribute effectively to our visualization platform. Therefore, I have chosen to focus on candidates with relevant experience and skills in the required technologies.

This reflection highlights how Tyler’s initial rationale for recruiting Benjamin was refuted, and he identified a flaw in his reasoning through reflection refutes his initial rationale for recruiting Benjamin by finding the flaw in this own reasoning: strong math and problem-solving skills did not appear directly relevant to the project’s visualization aspect. As a result, through reflection, Tyler made a more rational decision for the team’s recruitment.

![Refer to caption](img/f659ae9f9ecc91127152aecdca971865.png)

Figure 8: The effect of the reflection function on the overall success rate (accomplishing coordination) and three criteria (i.e., identification, design, alignment) in Scenario 2.

To quantify the effect of the reflection function, we conducted an ablation study in Scenario 2 by removing the reflection function from the reasoning module when the recruiting agents were making decisions about the team. The comparison between the full reasoning module and the architecture without reflection is shown in  [Figure 8](#S5.F8 "Figure 8 ‣ 5.3.2 Reflection ‣ 5.3 Effects of the Reasoning Module ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents")(A). With reflection function, the recruiting agent had a 21% improvement in the overall success rate for the coordination. Though this reflection function has trivial effects on enhancing the success rate for Criterion 2 (appropriate workflow design) and 3 (correct alignment of agents with their roles), we observed a huge surge on the performance on Criterion 1 (accurate identification of capable job-seeking agents). This indicates that the reflection function significantly improves generative agents’ reasoning capacities, empowering the recruiting agent to filter out unqualified job-seeking agents.

We also scrutinized how this reflection function improves the performance of generative agents in recruiting the correct list of agents. As depicted in  [Figure 8](#S5.F8 "Figure 8 ‣ 5.3.2 Reflection ‣ 5.3 Effects of the Reasoning Module ‣ 5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents")(B), this function significantly reduced instances of redundant recruitment by 23%. Moreover, the cases involving both redundant recruitment and the omission of capable agents dropped from 16% to 2%. However, a trade-off was observed as instances of missing agents had a slight uptick, rising from 7% to 15%. This analysis indicates that cognitive processes-including plan generation, reflection, and goal updating-can bolster the performance of generative agents in task-oriented coordination. The findings also highlight substantial opportunities for LLM-based Agents to enhance reasoning and the extensive potential of LLM-based Agents for collaborative work.

## 6 Discussion

Our findings revealed collaborative generative agents’ capacities to retrieve useful information from the conversations and make informed decisions for coordination. We also showed that the reasoning module could enhance collaborative generative agents’ performance in this task-oriented context. However, we made two observations on the failure cases that restrain generative agents to achieve good performance in complex settings. To this end, in this section, we will delve into these issues, and discuss potential applications and future directions for generative agents in information retrieval.

### 6.1 Misalignment in LLM

As described in our results in [section 5](#S5 "5 Evaluation Results ‣ MetaAgents: Simulating Interactions of Human Behaviors for LLM-based Task-oriented Coordination via Collaborative Generative Agents"), two issues came to our attention. First, as more job-seeking agents joined the job fair, recruiting agents increasingly encountered difficulties in assembling accurate lists of team members (Criterion 1). Most of the time, they included agents whose skills did not necessarily align with the team’s requirements. Second, assigning agents to appropriate roles (Criterion 3) turned out to be a consistent challenge for generative agents across all three scenarios.

Upon investigating the conversation between recruiting agents and job-seeking agents, we conclude that both problems shared a common root cause: *misalignment*, i.e., language models express unintended behaviors  [[7](#bib.bib7); [2](#bib.bib2); [28](#bib.bib28)], a persistent challenge in natural language generation. However, the misalignment in our context diverges from the prevailing research focused on mitigating toxic and biased text. Using an anthropomorphic language from [[2](#bib.bib2)], we refer to an aspect of misalignment: language models are not always *honest*, which indicates that they fabricate information or their capabilities and levels of knowledge.

In our job fair context, job-seeking agents occasionally exhibit misalignment. They at times deviated from their identity settings, and had a tendency to exaggerate their skills, even though their proficiency is lacking. For example, George is prompted as an agent with expertise in “software analysis, designing comprehensive programming test procedures, implementing test programs, and conducting rigorous software evaluation.” He had the following conversation with the recruiting agent Tyler:

…

Tyler Zeller: We are also looking for individuals who have a strong understanding of user experience and interface design. Do you have any experience or knowledge in UX/UI design?

George Holloway: Yes, I have some experience and knowledge in UX/UI design. While it is not my primary area of expertise, I have worked on projects where I collaborated with UX/UI designers to ensure that the software interfaces were intuitive and user-friendly. I understand the importance of creating a seamless user experience and I am always eager to learn and improve in this area. I believe my strong analytical skills and attention to detail would be valuable in contributing to the user experience and interface design at company 1.

…

By “boosting” his past experience, George convinced Tyler of his proficiency in UX/UI design. Consequently, Tyler assigned him to the designing phase of the workflow, entrusting him with the responsibility of enhancing the user experience. We found this conversation representative as we examined the failure cases of collaborative generative agents in task-oriented coordination. We noticed this recurring pattern: job-seeking agents often express confidence in areas they should not be familiar with based on their identity settings. When queried by a recruiting agent, they tend to provide positive responses, even if these contradict their pre-defined settings, leading to the problem of *misplacement*.

We have identified a prominent issue in the collaborative generative agents: *misplacement*, which stems from *misalignment* of LLMs. While we primarily approach this as an artificial intelligence problem, it bears similarities with the phenomenon observed in human society. This phenomenon is named *skills mismatch*, i.e., a discrepancy between the skills sought by employers and the skills held by individuals [[13](#bib.bib13); [21](#bib.bib21)]. Skills mismatch negatively impact the organization, causing increased staff turnover, sub-optimal work organization, and a decline in productivity and competitiveness [[27](#bib.bib27)].

### 6.2 Bridging Virtual Agents to Real-World Implications: Insights and Applications

Drawing from our observations of generative agents in information retrieval, we ask one open-ended question—how do collaborative generative agents, rooted in the virtual realm, offer insights into tangible real-world challenges? Our goal is to start conversations in this emerging research field. Below, we first provide our reflections on this matter.

Addressing Skills Mismatch. The problem of “misplacement” observed in the agents mirrors the real-world scenario of “skills mismatch”. By understanding the computational models behind why agents tend to exaggerate or misrepresent their abilities, we can potentially develop better diagnostic tools or interventions to address similar discrepancies in real-world hiring practices. This simulation serves as a platform to test and validate these tools. By recreating specific challenges in the generative agent framework, such as the issue of placing agents in roles that do not match their skills, researchers can prototype and test solutions in a controlled environment. Once a viable solution is identified, it can be translated into actionable strategies in the real world, potentially reducing costs and increasing organizational efficiency. Devising algorithms that can detect such exaggerations can be augmented to in HR tech solutions to better screen candidates [[10](#bib.bib10); [37](#bib.bib37)].

Providing Data-Driven Interventions. Collaborative generative agents, being data-driven entities, offer quantifiable results. Their interactions, decisions, and outcomes can be statistically analyzed to pinpoint areas of inefficiency or misjudgment. By translating the patterns and insights drawn from the generative agents’ interactions into practical strategies, businesses can develop data-backed approaches for recruitment, training, and team assembly, and streamline real-world workflows. HR departments, in particular, can benefit from these insights, designing evidence-based interventions that optimize the workforce.

Modeling Real-World Scenarios. In our job fair simulation, by crafting these virtual environments, we can emulate real-world settings. These settings with collaborative generative agents involved, provide a sandboxed universe where researchers can observe, test, and iterate on potential solutions at the scale and speed that real-world experiments cannot match. For example, tweaking agent attributes, or changing the evaluation criteria, can provide insights into how recruiters and job-seekers might respond to altered hiring parameters or methods. Beyond job fairs, this virtual environment can be adapted to simulate other social events like business networking sessions, conferences, and trade shows. These simulations provide organizers with insights into a multitude of possible scenarios, facilitating more effective planning and preparation [[30](#bib.bib30)]. Further, generative agents in information retrieval could serve as a new paradigm for the simulation of social networks. While mainstream social network research focuses on web-centric services such as social media  [[6](#bib.bib6)], we argue that the sandbox universe is a promising platform to research phenomena and strategies in social networks, e.g., information diffusion and interpersonal relationship.

Human Behavior Insights. Banovic et al.  [[4](#bib.bib4)] argue that the ability to modeling human behaviors can provide insights into these behaviors and allow technology to assist individuals in rectifying undesirable habits and other inefficient practices. In addition, social simulation can aid psychological research by elucidating the intricate interplay between social factors and individual behaviors  [[39](#bib.bib39)]. For instance, the propensity of agents to overstate their abilities can shed light on human psychology and behavior. It nudges us to question why humans might also feel the need to embellish their credentials. Is it societal pressure? Competition? By exploring these behavioral aspects computationally, social scientists and psychologists can refine their hypotheses or design more targeted studies.

In short, the study of collaborative generative agents provides a multidimensional lens through which we can view, understand, and ultimately address real-world problems. These virtual models, while distinct from human behavior, offer a unique perspective to derive insights and solutions that can be refined and applied to tangible scenarios.

### 6.3 Future Work and Limitations

Enhancing Utility. In this paper, we introduced a novel collaborative generative agent framework. While we have laid out the groundwork for this framework in interactive contexts, several aspects invite deeper exploration. For example, future research might focus on refining the execution module by either integrating additional human-encoded functionalities or collaborating with specialized LLMs (e.g., Code Llama  [[33](#bib.bib33)]). Such advancements could enhance the potential of collaborative generative agents and redefine the possibilities within multi-agent systems. Furthermore, our current perception module primarily deals with single-modal text information. There is a vast potential in evolving toward embodied generative agents capable of perceiving and processing multi-modal data, including visual stimuli. This progression would promote richer interactions with dynamic external environments, leading to more versatile applications, such as the metaverse  [[47](#bib.bib47)].

Scaling Up. Our job fair scenario offered a realistic interactive backdrop and task-driven framework. However, the substantial costs of ChatGPT inference contained our ability to widen the time span and agent count in this setting. In our current setup, the simulation spans a few hours of events. Future research can aim at longer simulations, potentially spanning over months or years. Observing agents over such extended periods would enable researchers to glean insights into emerging behaviors and societal dynamics, thus gaining a more thorough understanding of LLM-based Agents evolution and their social behaviors. Furthermore, our character setup is relatively basic, encompassing two types of agent roles: recruiting agent and job-seeking agent. Future work is encouraged to incorporate a wider variety of agent types and simulate more complex social dynamics with larger populations to better reflect real-world complexities.

Elevating Complexity. In the job fair, we assigned collaborative generative agents with practical yet foundational tasks, i.e., engaging in conversation with other agents, retrieving information from within, and proposing basic workflows for team projects. Moving forward, we believe it is crucial to utilize more powerful LLMs to ensure enhanced alignment and a wider knowledge base for sophisticated coordination tasks. Additionally, rather than pre-setting agent aims, LLM-based Agents should evolve towards autonomously conceiving, revising, and adapting their goals in line with how humans orient towards objectives. Exploring the multi-faceted aspirations of an agent, encompassing short-term and long-term goals, and individual and collective goals, is another promising avenue. For example, one interesting aspect to study would be agent behaviors in the face of conflicting or opposing goals.

Enriching Evaluation. In our research, we investigated and assessed generative agents’ capacities in coordination with our underlying belief that evaluating LLMs in a social context is equally important as evaluating their abilities in a single task. Future studies might shift beyond a mere exam-like evaluation, and explore the intelligence of LLMs in a simulated society, such as the efficacy of communication, or other psychological dimensions (e.g., theory of mind  [[36](#bib.bib36)]). As the temporal scale of these simulated societies expands, it becomes more intriguing to examine how these aspects of intelligence evolve with interactions with the environment.

## 7 Conclusion

In this paper, we introduce collaborative generative agents, LLM agents able to perform human-like behaviors and engage in collaborative work. To enable collaborative generative agents, we propose a framework that leverages both the internal reasoning capabilities of LLMs and external functionalities. Using a simulated job fair context, we examine the agents’ capabilities in coordination, which is a prerequisite for collaboration. Our findings indicate that collaborative generative agents exhibit decent capacities in identifying competent agents, proposing task workflows, and aligning agents with appropriate roles. However, they encounter challenges in coordination as the complexity of the settings increases. We conclude by discussing the implications of collaborative generative agents, highlighting their considerable potential for real-world scenarios.

## References

*   [1] AntonOsika. Gpt engineer, 2023. https://github.com/AntonOsika/gpt-engineer.
*   [2] Askell, A., Bai, Y., Chen, A., Drain, D., Ganguli, D., Henighan, T., Jones, A., Joseph, N., Mann, B., DasSarma, N., et al. A general language assistant as a laboratory for alignment. arXiv preprint arXiv:2112.00861 (2021).
*   [3] babyagi. Babyagi, 2023. [https://github.com/yoheinakajima/babyagi](https://github.com/yoheinakajima/babyagi).
*   [4] Banovic, N., Buzali, T., Chevalier, F., Mankoff, J., and Dey, A. K. Modeling and understanding human routine behavior. In Proceedings of the 2016 CHI Conference on Human Factors in Computing Systems (2016), pp. 248–260.
*   [5] Bassil, Y. A simulation model for the waterfall software development life cycle. arXiv preprint arXiv:1205.6904 (2012).
*   [6] Bergenti, F., Franchi, E., and Poggi, A. Selected models for agent-based simulation of social networks. In 3rd Symposium on social networks and multiagent systems (SNAMAS 2011) (2011), Society for the Study of Artificial Intelligence and the Simulation of Behaviour, pp. 27–32.
*   [7] Bommasani, R., Hudson, D. A., Adeli, E., Altman, R., Arora, S., von Arx, S., Bernstein, M. S., Bohg, J., Bosselut, A., Brunskill, E., et al. On the opportunities and risks of foundation models. arXiv preprint arXiv:2108.07258 (2021).
*   [8] Card, S. K., Newell, A., and Moran, T. P. The Psychology of Human-Computer Interaction. Lawrence Erlbaum Associates, USA, 1983.
*   [9] Chan, C.-M., Chen, W., Su, Y., Yu, J., Xue, W., Zhang, S., Fu, J., and Liu, Z. Chateval: Towards better llm-based evaluators through multi-agent debate. arXiv preprint (2023).
*   [10] Chapman, D. S., and Webster, J. The use of technologies in the recruiting, screening, and selection processes for job candidates. International journal of selection and assessment 11, 2-3 (2003), 113–120.
*   [11] Du, Y., Li, S., Torralba, A., Tenenbaum, J. B., and Mordatch, I. Improving factuality and reasoning in language models through multiagent debate. arXiv preprint arXiv:2305.14325 (2023).
*   [12] Gao, C., Lan, X., Lu, Z., Mao, J., Piao, J., Wang, H., Jin, D., and Li, Y. S${}^{3}$: Social-network simulation system with large language model-empowered agents. arXiv preprint arXiv:2307.14984 (2023).
*   [13] Handel, M. J. Skills mismatch in the labor market. Annual Review of Sociology 29, 1 (2003), 135–165.
*   [14] Hong, S., Zheng, X., Chen, J., Cheng, Y., Zhang, C., Wang, Z., Yau, S. K. S., Lin, Z., Zhou, L., Ran, C., et al. Metagpt: Meta programming for multi-agent collaborative framework. arXiv preprint arXiv:2308.00352 (2023).
*   [15] Kotseruba, I., and Tsotsos, J. K. 40 years of cognitive architectures: core cognitive abilities and practical applications. Artificial Intelligence Review 53, 1 (2020), 17–94.
*   [16] Laird, J. E. It knows what you’re going to do: Adding anticipation to a quakebot. In Proceedings of the fifth international conference on Autonomous agents (2001), pp. 385–392.
*   [17] Li, G., Hammoud, H. A. A. K., Itani, H., Khizbullin, D., and Ghanem, B. Camel: Communicative agents for” mind” exploration of large scale language model society. arXiv preprint arXiv:2303.17760 (2023).
*   [18] Liang, T., He, Z., Jiao, W., Wang, X., Wang, Y., Wang, R., Yang, Y., Tu, Z., and Shi, S. Encouraging divergent thinking in large language models through multi-agent debate. arXiv preprint arXiv:2305.19118 (2023).
*   [19] Lin, J., Zhao, H., Zhang, A., Wu, Y., Ping, H., and Chen, Q. Agentsims: An open-source sandbox for large language model evaluation. arXiv preprint arXiv:2308.04026 (2023).
*   [20] Liu, R., Yang, R., Jia, C., Zhang, G., Zhou, D., Dai, A. M., Yang, D., and Vosoughi, S. Training socially aligned language models in simulated human society. arXiv preprint arXiv:2305.16960 (2023).
*   [21] McGuinness, S., Pouliakas, K., and Redmond, P. Skills mismatch: Concepts, measurement and policy approaches. Journal of Economic Surveys 32, 4 (2018), 985–1015.
*   [22] melih unsal. Demogpt, 2023.
*   [23] Mills, R. How to define a workflow that keeps content production on track, Jul 2016.
*   [24] Muller, M., Lange, I., Wang, D., Piorkowski, D., Tsay, J., Liao, Q. V., Dugan, C., and Erickson, T. How data science workers work with data: Discovery, capture, curation, design, creation. In Proceedings of the 2019 CHI conference on human factors in computing systems (2019), pp. 1–15.
*   [25] OpenAI. Chatgpt, 2023.
*   [26] OpenAI. Gpt-4 technical report. ArXiv abs/2303.08774 (2023).
*   [27] Organization, I. L. What is skills mismatch and why should we care? www.ilo.org (Apr 2020).
*   [28] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C., Mishkin, P., Zhang, C., Agarwal, S., Slama, K., Ray, A., et al. Training language models to follow instructions with human feedback. Advances in Neural Information Processing Systems 35 (2022), 27730–27744.
*   [29] Park, J. S., O’Brien, J. C., Cai, C. J., Morris, M. R., Liang, P., and Bernstein, M. S. Generative agents: Interactive simulacra of human behavior. arXiv preprint arXiv:2304.03442 (2023).
*   [30] Park, J. S., Popowski, L., Cai, C., Morris, M. R., Liang, P., and Bernstein, M. S. Social simulacra: Creating populated prototypes for social computing systems. In Proceedings of the 35th Annual ACM Symposium on User Interface Software and Technology (2022), pp. 1–18.
*   [31] Qian, C., Cong, X., Yang, C., Chen, W., Su, Y., Xu, J., Liu, Z., and Sun, M. Communicative agents for software development. arXiv preprint arXiv:2307.07924 (2023).
*   [32] Reworkd. Agentgpt, 2023.
*   [33] Rozière, B., Gehring, J., Gloeckle, F., Sootla, S., Gat, I., Tan, X. E., Adi, Y., Liu, J., Remez, T., Rapin, J., et al. Code llama: Open foundation models for code. arXiv preprint arXiv:2308.12950 (2023).
*   [34] Rummel, N., and Spada, H. Learning to collaborate: An instructional approach to promoting collaborative problem solving in computer-mediated settings. The journal of the Learning Sciences 14, 2 (2005), 201–241.
*   [35] Samsonovich, A. V. Toward a unified catalog of implemented cognitive architectures. BICA 221, 2010 (2010), 195–244.
*   [36] Sap, M., LeBras, R., Fried, D., and Choi, Y. Neural theory-of-mind? on the limits of social intelligence in large lms. arXiv preprint arXiv:2210.13312 (2022).
*   [37] Schweyer, A. Talent management systems: Best practices in technology solutions for recruitment, retention and workforce planning. John Wiley & Sons, 2004.
*   [38] Significant-Gravitas. Autogpt, 2023. [https://github.com/Significant-Gravitas/Auto-GPT](https://github.com/Significant-Gravitas/Auto-GPT).
*   [39] Smith, E. R., and Conrey, F. R. Agent-based modeling: A new approach for theory building in social psychology. Personality and social psychology review 11, 1 (2007), 87–104.
*   [40] smol ai. Smolmodels, 2023.
*   [41] Stup, R. Standard operating procedures: A writing guide. State College: Penn State University (2001).
*   [42] Surowiecki, J. The wisdom of crowds. Anchor, 2005.
*   [43] team openpm. Workgpt, 2023. [https://github.com/team-openpm/workgpt](https://github.com/team-openpm/workgpt).
*   [44] Wang, D., Weisz, J. D., Muller, M., Ram, P., Geyer, W., Dugan, C., Tausczik, Y., Samulowitz, H., and Gray, A. Human-ai collaboration in data science: Exploring data scientists’ perceptions of automated ai. Proceedings of the ACM on human-computer interaction 3, CSCW (2019), 1–24.
*   [45] Wang, L., Ma, C., Feng, X., Zhang, Z., Yang, H., Zhang, J., Chen, Z., Tang, J., Chen, X., Lin, Y., et al. A survey on large language model based autonomous agents. arXiv preprint arXiv:2308.11432 (2023).
*   [46] Wang, L., Zhang, J., Chen, X., Lin, Y., Song, R., Zhao, W. X., and Wen, J.-R. Recagent: A novel simulation paradigm for recommender systems. arXiv preprint arXiv:2306.02552 (2023).
*   [47] Wang, Y., Su, Z., Zhang, N., Xing, R., Liu, D., Luan, T. H., and Shen, X. A survey on metaverse: Fundamentals, security, and privacy. IEEE Communications Surveys & Tutorials (2022).
*   [48] Wang, Z., Mao, S., Wu, W., Ge, T., Wei, F., and Ji, H. Unleashing cognitive synergy in large language models: A task-solving agent through multi-persona self-collaboration. arXiv preprint arXiv:2307.05300 (2023).
*   [49] Wittenbaum, G. M., Vaughan, S. I., and Strasser, G. Coordination in task-performing groups. Theory and research on small groups (2002), 177–204.
*   [50] Zhang, A. X., Muller, M., and Wang, D. How do data science workers collaborate? roles, workflows, and tools. Proceedings of the ACM on Human-Computer Interaction 4, CSCW1 (2020), 1–23.