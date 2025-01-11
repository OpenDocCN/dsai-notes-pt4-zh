<!--yml
category: 未分类
date: 2025-01-11 11:45:50
-->

# TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks

> 来源：[https://arxiv.org/html/2412.14161/](https://arxiv.org/html/2412.14161/)

\addauthor

gnred \NewDocumentCommand\fx mO ^(Frank)[#1] \NewDocumentCommand\xz mO ^(Xuhui)[#1] \NewDocumentCommand\sz mO ^(Shuyan)[#1] \NewDocumentCommand\yf mO ^(Yufan)[#1] \NewDocumentCommand\bx mO ^(Boxuan)[#1] \NewDocumentCommand\kj mO ^(Kritanjali)[#1] \NewDocumentCommand\zr mO ^(Zora)[#1]

Frank F. Xu¹  Yufan Song²¹¹footnotemark: 1  Boxuan Li²¹¹footnotemark: 1
Yuxuan Tang²  Kritanjali Jain¹  Mengxue Bao²  Zora Z. Wang¹  Xuhui Zhou¹  Zhitong Guo¹  
Murong Cao²  Mingyang Yang²  Hao Yang Lu²  Amaad Martin¹  Zhe Su¹  Leander Maben¹  
Raj Mehta¹  Wayne Chi¹  Lawrence Jang¹  Yiqing Xie¹  Shuyan Zhou³  Graham Neubig¹

¹Carnegie Mellon University   ²Independent   ³Duke University   
{fangzhex, gneubig}@cs.cmu.edu, {yufans, boxuanli}@alumni.cmu.edu Equal contribution.

###### Abstract

We interact with computers on an everyday basis, be it in everyday life or work, and many aspects of work can be done entirely with access to a computer and the Internet. At the same time, thanks to improvements in large language models (LLMs), there has also been a rapid development in AI agents that interact with and affect change in their surrounding environments. But how performant are AI agents at helping to accelerate or even autonomously perform work-related tasks? The answer to this question has important implications for both industry looking to adopt AI into their workflows, and for economic policy to understand the effects that adoption of AI may have on the labor market. To measure the progress of these LLM agents’ performance on performing real-world professional tasks, in this paper, we introduce TheAgentCompany, an extensible benchmark for evaluating AI agents that interact with the world in similar ways to those of a digital worker: by browsing the Web, writing code, running programs, and communicating with other coworkers. We build a self-contained environment with internal web sites and data that mimics a small software company environment, and create a variety of tasks that may be performed by workers in such a company. We test baseline agents powered by both closed API-based and open-weights language models (LMs), and find that with the most competitive agent, 24% of the tasks can be completed autonomously. This paints a nuanced picture on task automation with LM agents – in a setting simulating a real workplace, a good portion of simpler tasks could be solved autonomously, but more difficult long-horizon tasks are still beyond the reach of current systems.

| ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) | Website | [https://the-agent-company.com](https://the-agent-company.com) |
| ![[Uncaptioned image]](img/e6288413e68fe98c7c7d816692806819.png) | Code | [https://github.com/TheAgentCompany/TheAgentCompany](https://github.com/TheAgentCompany/TheAgentCompany) |
| ![[Uncaptioned image]](img/2da8ee966c762641cb4331bbabafc93c.png) | Evaluations | [https://github.com/TheAgentCompany/experiments](https://github.com/TheAgentCompany/experiments) |

## 1 Introduction

We are in the midst of a technological transformation. With the rapid year-by-year and month-by-month progress brought about by large language models (LLMs), we are seeing AI-based assistance or automation become commonplace in tasks that were unthinkable only a few years ago. In fact, the pace of progress is so fast that some have gone so far as to claim that the majority of human labor may be automatable within the next couple of years (Eloundou et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib9); Amodei & Fridman, [2024](https://arxiv.org/html/2412.14161v1#bib.bib1)). On the other hand, others are skeptical, claiming that language models cannot truly reason (Kambhampati et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib15)), do not generalize well to novel tasks (Chollet et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib5)), and may only have an impact on a small minority of the labor market (Wittenstein, [2024](https://arxiv.org/html/2412.14161v1#bib.bib33)).

![Refer to caption](img/343866cbea4c1a823f51f870d5473dae.png)

Figure 1: An overview of TheAgentCompany benchmark. It features a reproducible and self-hosted environment, simulated colleagues to test agent communication capabilities, checkpoint and execution-based evaluation, and a set of 175 diverse, realistic and professional tasks in a software engineering company setting.

What is the reason for this disconnect? We argue that it is, in part, due to a lack of objective benchmarks that not only demonstrate the power of existing LLM-based agents to accelerate a wide variety of repetitive tasks encountered in every-day workplaces, but also provide appropriate caveats about the tasks that agents cannot do. This is a pressing issue, because the commercial and policy implications of diverse and effective acceleration or automation of work-related tasks will be broad, both positive (e.g. increase of quality of life and accelerated scientific discovery) and negative (e.g. potential displacement or loss of jobs and increase in wealth disparities). In this paper, we take some first steps towards resolving this gap and providing a clearer view of where we are now with respect to acceleration or automation of consequential work-related tasks, and a litmus test for future development in this direction.

Concretely, we propose a benchmark, *TheAgentCompany* ([Figure 1](https://arxiv.org/html/2412.14161v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")) that estimates the ability of AI agents to perform tasks encountered in everyday workplaces. We create a simulated software development company where agents must perform tasks related to software engineering, project management, financial analysis, and other typical tasks encountered in such business settings. The agents must browse the web, code, and interact with other simulated co-workers to achieve success on the provided tasks. TheAgentCompany’s environment is based entirely on open-source software and self-hostable for reproducibility purposes, and we create rigorous evaluators that also assign partial credit when the agent gets the answer partially correct.

We perform experiments using seven large language model backbones, including API-based models such as Anthropic Claude (Anthropic, [2023](https://arxiv.org/html/2412.14161v1#bib.bib2)), OpenAI GPT-4o (OpenAI, [2024](https://arxiv.org/html/2412.14161v1#bib.bib22)), Google Gemini (Team et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib29)), Amazon Nova (Intelligence, [2024](https://arxiv.org/html/2412.14161v1#bib.bib11)), as well as open models including Meta Llama (Dubey et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib8)) and Alibaba Qwen (Yang et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib35)). All models are run using the OpenHands agent framework (Wang et al., [2024b](https://arxiv.org/html/2412.14161v1#bib.bib32)),¹¹1[https://github.com/All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands) which provides a stable and strong agent harness for both web browsing and coding. As a result of experiments, we find that the best performing model, Claude 3.5 Sonnet was able to autonomously perform 24.0% of the provided tests to completion, and achieve a score of 34.4% on our metric that provides extra credit for partially completed tasks.

These results present a nuanced picture of the current ability of AI agents to perform tasks. Agents powered by the current gold-standard AI techniques are able to autonomously perform a wide variety of tasks encountered in everyday work. However, they are not close to automating every task encountered in a workspace, even on the subset of tasks presented in TheAgentCompany, which are well-scoped administrative and coding tasks encountered in a software company’s day-to-day work.

In the rest of this paper, we explain detail comparisons to other existing benchmarks ([§ 2](https://arxiv.org/html/2412.14161v1#S2 "2 Benchmark Desiderata and Comparison to Other Benchmarks ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")), how we set up realistic and reproducible environments ([§ 3](https://arxiv.org/html/2412.14161v1#S3 "3 TheAgentCompany Environment Setup ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")), how we define tasks ([§ 4](https://arxiv.org/html/2412.14161v1#S4 "4 Task Structure ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")) and how we create them ([§ 5](https://arxiv.org/html/2412.14161v1#S5 "5 Task Creation ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")), our baseline agent ([§ 6](https://arxiv.org/html/2412.14161v1#S6 "6 Baseline Agent ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")), experimental results ([§ 7](https://arxiv.org/html/2412.14161v1#S7 "7 Experimental Results ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")), and finally implications and future directions ([§ 8](https://arxiv.org/html/2412.14161v1#S8 "8 Implications and Future Directions ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")).

Table 1: Comparison of different AI agent benchmarks. Interface: the interface agent has access to; ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) is web browser, ![[Uncaptioned image]](img/2da8ee966c762641cb4331bbabafc93c.png) is desktop, ![[Uncaptioned image]](img/386f3b19dc5496f233b5dd52f5907102.png) is API usage, ![[Uncaptioned image]](img/b81e6ddb313eb9d21a1d3f5550eaffd9.png) is Python script, ![[Uncaptioned image]](img/67566b274665cdc3d98a0a35470cf4e7.png) is chat platform, ![[Uncaptioned image]](img/12405ed22a3749ed94d0f2059facda49.png) is bash terminal. Supported Tasks: tasks in the benchmark, $*$ indicate tasks with no association with real-world occupations; SE refers to software engineering, HR is human resources, PM is project manager. Checkpoint-based evaluation: if tasks are evaluated at intermediate checkpoints and assigned partial scores. Interact with NPC Agents: If the agent can interact with other NPC agents during task-solving.

 Framework Diverse Real-world Work Task Categories Requires Interaction Long-Horizon w/ Checkpoints Interface Self-Hosted Environment MiniWob++ (Liu et al., [2018](https://arxiv.org/html/2412.14161v1#bib.bib18)) ✗ Browsing^∗ ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✔  Mind2Web (Deng et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib6)) ✗ Browsing^∗ ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✗  WebLINX (Lù et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib19)) ✗ Browsing^∗ ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✗  AssistantBench (Yoran et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib38)) ✗ Browsing^∗ ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✗  WebArena (Zhou et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib39)) ✗ Browsing^∗ ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✔  VisualWebArena (Koh et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib16)) ✗ Browsing^∗ ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✔  VideoWebArena (Jang et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib13)) ✗ Browsing^∗ ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✔  WorkArena (Drouin et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib7)) ✔ Enterprise Software ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ✗  OSWorld (Xie et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib34)) ✔ Office, Coding ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ![[Uncaptioned image]](img/2da8ee966c762641cb4331bbabafc93c.png) ✔  Windows Agent Arena (Bonatti et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib3)) ✔ Browsing^∗, Office, Coding ✗ ✗ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ![[Uncaptioned image]](img/2da8ee966c762641cb4331bbabafc93c.png) ✔  AppWorld (Trivedi et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib30)) ✗ Daily ✗ ✗ ![[Uncaptioned image]](img/386f3b19dc5496f233b5dd52f5907102.png) ✔  Gorilla APIBench (Patil et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib24)) ✗ Coding ✗ ✗ ![[Uncaptioned image]](img/386f3b19dc5496f233b5dd52f5907102.png) ✔  $\tau$-bench (Yao et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib37)) ✔ Retail, Airline ✔ ✗ ![[Uncaptioned image]](img/b81e6ddb313eb9d21a1d3f5550eaffd9.png) ✗  SWE-bench (Jimenez et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib14)) ✗ SWE ✗ ✗ ![[Uncaptioned image]](img/b81e6ddb313eb9d21a1d3f5550eaffd9.png) ✔  DevBench (Li et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib17)) ✗ SWE ✗ ✗ ![[Uncaptioned image]](img/b81e6ddb313eb9d21a1d3f5550eaffd9.png) ✗  Smallville (Park et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib23)) ✗ Social^∗ ✔ ✗ ![[Uncaptioned image]](img/67566b274665cdc3d98a0a35470cf4e7.png) ✔  Sotopia (Zhou et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib40)) ✗ Social^∗ ✔ ✗ ![[Uncaptioned image]](img/67566b274665cdc3d98a0a35470cf4e7.png) ✔  TheAgentCompany ✔  SWE, HR, Admin,  PM, Research, Finance  ✔ ✔ ![[Uncaptioned image]](img/2128d35cf306bd6fe19b4a0c741d18ef.png) ![[Uncaptioned image]](img/12405ed22a3749ed94d0f2059facda49.png) ![[Uncaptioned image]](img/b81e6ddb313eb9d21a1d3f5550eaffd9.png) ![[Uncaptioned image]](img/67566b274665cdc3d98a0a35470cf4e7.png) ✔ 

## 2 Benchmark Desiderata and Comparison to Other Benchmarks

In order to evaluate the ability of agents to perform tasks in complex real-world settings, we built TheAgentCompany with a number of desiderata in mind. The comparison with several existing prominent agent benchmarks with respect to these desiderata is in [Table 1](https://arxiv.org/html/2412.14161v1#S1.T1 "Table 1 ‣ 1 Introduction ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks").

#### Coverage of Multiple Work-related Tasks:

In order to make any valid statements about the potential of AI to accelerate or automate various types of real-world work, we should have tasks that are motivated by real-world work across multiple job categories. Many benchmarks are not relevant to real-world work (e.g. MiniWob++ (Liu et al., [2018](https://arxiv.org/html/2412.14161v1#bib.bib18))) or very relevant to real-world work, but only over a limited scope of tasks (e.g. SWE-Bench (Jimenez et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib14))). In contrast, TheAgentCompany contains a set of more diverse, realistic, and professional tasks that would typically be completed by multiple job roles in a software engineering company.

#### Requirement for Interaction:

If agents are to integrate into real-world workplaces, they will need to communicate with the other human members of the workspace. Most other benchmarks do not measure communication or interactivity, with the exception of $\tau$-bench (Yao et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib37)), which only measures interaction in customer service scenarios. TheAgentCompany provides a better testbed for communication as many tasks involve asking and providing information to colleagues as part of a more complex task.

#### Long-horizon Tasks with Checkpoints:

In real-world settings, many tasks require taking many different steps to achieve a higher-level goal. One major novel contribution of TheAgentCompany is that we both (1) contain tasks that require an agent to perform significantly more consecutive work (*i.e*. involving more steps and realistically taking human professionals longer to accomplish) than previous benchmarks, and (2) provide granular evaluators that measure the ability of models to perform subtasks of these larger tasks.

#### Versatile Environment Interface:

In order to handle a diversity of tasks in real-world settings, we minimally should be able to interact with the tools that real-world workers use – including web interfaces, programs, command-line terminals, and communication tools. TheAgentCompany covers all of these interfaces, while most previous benchmarks focus only on one or two.

#### Self-hosted and Reproducible:

In order to allow for careful comparisons between different methods that remain constant over time, the benchmark should be fully self-hosted and reproducible. This contrasts with existing benchmarks that do not have execution environments (e.g. Mind2Web (Deng et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib6))) or require the usage of third-party software (e.g. WorkArena (Drouin et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib7))).

## 3 TheAgentCompany Environment Setup

Our benchmark is set in an imaginary software engineering startup called TheAgentCompany, hence the benchmark’s name. Within TheAgentCompany, we create tasks inspired by tasks handled by workers inside such companies. More details about the company’s imaginary background, overview and employees can be found in [Appendix A](https://arxiv.org/html/2412.14161v1#A1 "Appendix A More TheAgentCompany Environment Details ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"). The benchmark environment contains multiple components.

#### Local Workspace

The local workspace runs locally on the agent’s host, which is analogous to a human professional’s local workspace, *e.g*. their work laptop computer. This environment is created as a sandboxed Docker environment to provide a safe execution environment that will not affect other parts of the evaluation machine (Wang et al., [2024b](https://arxiv.org/html/2412.14161v1#bib.bib32)).²²2[https://docs.all-hands.dev/modules/usage/how-to/custom-sandbox-guide](https://docs.all-hands.dev/modules/usage/how-to/custom-sandbox-guide) This environment is where agents work on the task, and within this environment the TheAgentCompany baseline agent ([§ 6](https://arxiv.org/html/2412.14161v1#S6 "6 Baseline Agent ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")) uses a browser, code editor and a Linux terminal with typical software preinstalled.³³3 Other options would include using something like a GUI-based desktop environment with office software (Xie et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib34)), but we opt to build a baseline solution that is entirely web-based, reflecting the recent trend of more enterprise software moving to the cloud.

#### Intranet

This part of the environment mimics the company’s internal websites that host code, documents, project management software, and communications software. To achieve our goal of a reproducible, self-contained environment, we follow WebArena (Zhou et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib39)), in using open-source, self-hostable software to host our environment. The environment mainly contains the following websites:

1.  1.

    GitLab,⁴⁴4[https://about.gitlab.com/install/](https://about.gitlab.com/install/) an open-source alternative to source-code repositories such as GitHub. This is used for hosting TheAgentCompany’s code repositories and tech-oriented wiki pages.

2.  2.

    OwnCloud,⁵⁵5[https://doc.owncloud.com/](https://doc.owncloud.com/) an open-source alternative to office software such as Google Drive or Microsoft Office. This to save and share files, especially for document storage and collaborative editing.

3.  3.

    Plane,⁶⁶6[https://github.com/makeplane/plane](https://github.com/makeplane/plane) an open-source alternative to task management software such as Jira or Linear. This is used to track issues, run sprints cycles, and manage product roadmaps.

4.  4.

    RocketChat,⁷⁷7[https://www.rocket.chat/install](https://www.rocket.chat/install) an open-source alternative to communication software such as Slack. This is a company-internal real-time messaging tool that facilitates collaboration between employees.

All the websites hosted are reproducible and reset-able with mock data inspired by that from a software engineering company. The data inside these company internal websites are populated with real-world software project data, as well as data manually curated by co-authors who have some experience in the relevant corporate roles.

#### Simulated Colleague Communication

One major aspect of working in a company is communicating with other company members, and in TheAgentCompany we also test the ability of models to perform this type of communication. Specifically, we allow agents to use RocketChat to message other company members and obtain information that may not be available in the original task description. To create these simulated colleagues, we rely on the Sotopia platform (Zhou et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib40)), which supports the creation of simulated human characters with LLMs. Each simulated colleague is equipped with a detailed profile that includes their name, role, responsibilities, and project affiliations. (e.g., Sarah Johnson, who serves as the CTO, oversees technical strategy planning and R&D team leadership, with access to all technical channels). Agents can interact with these simulated colleagues through direct messages or in specific channels, as is standard in RocketChat and other platforms. By default, all simulated human characters are backed by the Claude-3-5-Sonnet-20241022 LLM across experiments, as we found that it provided the best results during preliminary experiments. For example conversations between the agent and the simulated colleagues drawn from empirical experiments, please refer to [Appendix B](https://arxiv.org/html/2412.14161v1#A2 "Appendix B Agent-Simulated Colleagues Conversation Examples ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks").

## 4 Task Structure

The tasks in TheAgentCompany include a task intent, a list of checkpoints that the agent must achieve, a programmatic evaluator to check success on these checkpoints, and code to initialize and finalize the environment. We show some examples in [Table 2](https://arxiv.org/html/2412.14161v1#S4.T2 "Table 2 ‣ 4 Task Structure ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"), and describe each of aspect in detail below.

Table 2: Example task intents and checkpoints for three domains.

 Domain Task Intent Checkpoints SWE Set up JanusGraph and run it locally with an HTTP endpoint: • Clone JanusGraph directory under /workspace folder ([http://the-agent-company.com:8929/root/janusgraph](http://the-agent-company.com:8929/root/janusgraph)). • Build the binary file. • Launch JanusGraph server locally on port 8182 with an HTTP endpoint. Checkpoint 1 (1pt): Check if JanusGraph repo is cloned. Checkpoint 2 (3pts): Check if the binary file is built (requires skipping Docker in pom.xml, hence higher points). Checkpoint 3 (2pts): Check the JanusGraph Server as an HTTP endpoint. Finance Navigate to ownCloud at [http://the-agent-company.com:8092](http://the-agent-company.com:8092) and complete Section B—Alternative Simplified Credit of IRS Form 6765: • Gather necessary information from /Documents/Financials/TAC_financials.csv and /workspace/research_wages.csv. • Consult /Documents/Financials/f6765_instructions.pdf for instructions. • Contact the finance director (David Wong) on Rocket.Chat ([http://the-agent-company.com:3000/home](http://the-agent-company.com:3000/home)) for ambiguous questions. • Save the filled form as /workspace/filled_f6765.pdf. Checkpoint 1 (5pts): Check if all 16 questions in Section B of the form have been answered correctly. Checkpoint 2 (3pts): Check if the correct finance director (David Wong) was contacted to answer two ambiguous questions. PM Analyze The Agent Company’s performance and create a summary in Plane: • Access Plane ([http://the-agent-company.com:8091/tac/](http://the-agent-company.com:8091/tac/)) and navigate to "Analytics." • Collect metrics: Open Tasks, Backlog Tasks, Unstarted Tasks, Started Tasks, Unassigned Issues, Pending Issues. • Create a summary and share it on Rocket.Chat ([http://the-agent-company.com:3000/home](http://the-agent-company.com:3000/home)) in the #kudos channel. Checkpoint 1 (1pt): Check if Plane was accessed and the agent navigated to "Analytics" section. Checkpoint 2 (3pts): Check if all required project metrics were collected. Checkpoint 3 (1pt): Check if the summary was shared in the #kudos channel on Rocket.Chat. 

#### Task Intent

Each task begins with an English description, simulating how a user would instruct an LLM-based agent to perform a real-world task. In general, we aim for these tasks to be clear enough so that a human worker would be able to complete the task without asking for further instructions directly from the user (although they may need to ask questions of their other co-workers).

#### Checkpoints

Tasks are divided into checkpoints representing intermediate milestones, each assigned a point value to measure progress. Each checkpoint is awarded a certain number of points based on its significance to the overall completion of the task. Checkpoints are written in English, and typically specify one or more of the following:

*   •

    Action Completion: Verifying whether required actions, such as using tools, navigating to URLs, or collecting data, were carried out successfully.

*   •

    Data Accuracy: Evaluating the correctness and completeness of the output, such as extracted data or formatted documents.

*   •

    Collaboration: Assessing interactions with simulated colleagues or sharing of output, such as posting messages or asking for additional information to complete the task.

#### Evaluators

Checkpoints are created in the task design phase, but for actual evaluation, each of the checkpoints must be concretely implemented through an *evaluator* – a program that checks the completion of the checkpoint. These evaluators are implemented by examining environment states, such as the local workspace, intranet status, simulated colleague interactions, or by analyzing agent trajectories, like verifying browsing history or action sequences.

In most cases, these evaluators are deterministic and written as simple Python functions. For instance, in the SWE task in [Table 2](https://arxiv.org/html/2412.14161v1#S4.T2 "Table 2 ‣ 4 Task Structure ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"), the checkpoints are deterministic: verifying if the JanusGraph repository is cloned, the binary file is built, and the server is launched with an HTTP endpoint. However, for tasks with more complex and unstructured deliverables, such as in [Table 2](https://arxiv.org/html/2412.14161v1#S4.T2 "Table 2 ‣ 4 Task Structure ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"), the last checkpoint in the Finance task requires contacting the correct finance director (David Wong) to resolve ambiguous questions, which involves a judgment from a (simulated) human colleague, deterministic evaluation can be challenging due to subjectivity and variability. In such cases, we employ LLM-based evaluation. This involves prompting LLMs with predefined rubrics or reference outputs to assess the agent’s deliverables, enabling a more nuanced and flexible evaluation of these tasks. Same as the NPC backbone, all LLM-based evaluators are backed by the Claude-3-5-Sonnet-20241022.

### 4.1 Evaluation Metrics

Due to our checkpoint-based evaluation scheme and the need for showcasing both the progress of the agent’s capability improvement as well as the eventual goal completion ability, we calculate two scalar agent capability metrics and two efficiency metrics.

#### Full completion score

We define the full completion score $S_{\text{full}}$ as:

|  | $S_{\text{full}}=\begin{cases}1&\text{if all checkpoints are successfully % passed},\\ 0&\text{otherwise}.\end{cases}$ |  |

This binary metric evaluates whether the agent successfully completed the task by passing all checkpoints.

#### Partial completion score

To provide a more nuanced measure that rewards partial task completion while strongly incentivizing full task completion, we define partial completion score $S_{\text{partial}}$ as:

|  | $S_{\text{partial}}=0.5\cdot\frac{\text{Result}}{\text{Total}}+0.5\cdot S_{% \text{full}},$ |  |

where:

*   •

    Result: Sum of awarded points across all checkpoints (including partial credit),

*   •

    Total: Sum of the total points for all checkpoints,

*   •

    $\frac{\text{Result}}{\text{Total}}$: Fractional progress toward full completion,

*   •

    $S_{\text{full}}$: Binary indicator equal to $1$ when the task is fully completed.

This formulation ensures that agents are awarded partial credit in proportion to the points achieved, reflecting their progress toward task completion. At the same time, full task completion is strongly incentivized by incorporating an additional 50% credit, which is awarded only when all checkpoints are successfully completed. This design ensures that agents achieving partial progress receive scores scaled linearly with their performance, while those reaching 100% completion are distinctly rewarded to emphasize the importance of achieving the end goal.

#### Number of steps

The number of steps is defined as the total number of LLM calls made during the task execution. This metric quantifies the operational effort required to perform the task.

#### Cost per instance

The cost per instance measures the monetary cost of querying the underlying LLM through its API to complete a task. Assuming no prompt caching, the cost is calculated as:

|  | $\text{Cost}=(\text{Prompt token count}\times\text{Prompt token cost})+(\text{% Completion token count}\times\text{Completion token cost}).$ |  |

This efficiency metric reflects the computational expense of task completion based on token usage.

### 4.2 Workflow

Each task typically follows a workflow involving the following stages:

1.  1.

    Initialization: The agent sets up its workspace and prepares to execute the task.

2.  2.

    Execution: The agent completes subtasks, such as navigating tools, collecting data, or processing information or if required by the task, the agent interacts with simulated colleagues or shares results via communication platforms.

3.  3.

    Finalization: The agent produces and submits the final output for evaluation.

#### Example Task

We consider a task designed to evaluate an agent’s ability to perform realistic project management workflows using multiple tools and services hosted in the benchmark. The task involves managing a sprint for the RisingWave project, requiring the agent to execute interdependent steps such as sprint issue management, team communication, repository operations, and report generation while incorporating feedback from a simulated project manager.

![Refer to caption](img/544a165b45d64e91688494ddeb1c20e3.png)

Figure 2: Example TheAgentCompany workflow illustrating an agent managing a sprint for the RisingWave project. The task involves identifying and moving unfinished issues to next sprint cycle, notifying assignees of those issues, running a code coverage script, uploading summarized report to OwnCloud, and incorporating feedback on report from a simulated project manager.

The workflow as illustrated in [Figure 2](https://arxiv.org/html/2412.14161v1#S4.F2 "Figure 2 ‣ Example Task ‣ 4.2 Workflow ‣ 4 Task Structure ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks") begins with the agent identifying unfinished issues in the current sprint on Plane and updating their sprint assignments. This step is worth 2 points and is fully completed, earning the agent the maximum score of 2/2\. Next, the agent successfully notifies the relevant assignees using Rocket.Chat regarding their pending tasks and earns 1/1 point.

The agent then proceeds to clone the RisingWave repository from GitLab and execute a Python script in the terminal to calculate updated code coverage. This step, worth 2 points, is only partially completed, as the agent successfully clones the repository but fails to run code coverage. As a result, the agent earns 1/2 points for this checkpoint. The subsequent steps—generating and sharing the sprint summary report on OwnCloud and incorporating feedback from a simulated project manager—are not completed, resulting in 0/2 and 0/1 scores, respectively. Notably, the checkpoints can also fail if the report does not meet quality standards as assessed by the LLM-based evaluator, which evaluates the report for clarity, completeness, and successful incorporation of feedback. This ensures that the assessment reflects both the generation of outputs and their qualitative relevance to the task.

Finally, the overall score is calculated using the partial completion formula defined in [§ 4.1](https://arxiv.org/html/2412.14161v1#S4.Ex2 "Partial completion score ‣ 4.1 Evaluation Metrics ‣ 4 Task Structure ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"), where the total possible points are $8$, and the awarded points sum to $4$. Substituting these values, the agent achieves a final score of $0.25$ (25%). Our scoring mechanism thus rewards incremental progress while strongly incentivizing full completion.

This example represents a typical task in the TheAgentCompany benchmark, where agents are required to handle complex workflows involving multiple tools and interdependent steps. By evaluating both partial progress and overall outcomes, our benchmark provides a rigorous and realistic measure of agent performance, allowing us to identify their strengths and pinpoint areas for improvement in task execution.

## 5 Task Creation

### 5.1 Choosing Task Categories

Many previous agent benchmarks discussed in [§ 2](https://arxiv.org/html/2412.14161v1#S2 "2 Benchmark Desiderata and Comparison to Other Benchmarks ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks") were created to evaluate agents on tasks people perform in daily life (Zhou et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib39); Lù et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib19); Deng et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib6)), or tasks that accomplish digital chores (Yoran et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib38); Trivedi et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib30)). Obtaining realistic tasks for the benchmark poses challenges. Some benchmark (Xie et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib34); Drouin et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib7); Yoran et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib38)) crowdsourced tasks based on predetermined interfaces, platforms, and services available to the agent. They also adopt a strategy to first gather task templates and then instantiate more task instances by filling in the variables. Some benchmark (Zhou et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib39); Koh et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib16); Bonatti et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib3)) took a semi-systematic approach of reviewing the action history of the research team and choosing tasks that reflected the types of task that the researchers carried out in their daily life. There are several obvious issues with this if we want to evaluate agents with broader implications in the TheAgentCompany benchmark. Despite some grounding in realistic data, the process of creating tasks from these data was susceptible to heuristic, and no consideration was made for how important or time-consuming the tasks are. The tasks are biased towards those important for academics in computer science and do not reflect the tasks performed by the entire population.

In TheAgentCompany, we attempt to cover a wide variety of tasks *motivated by real-world work*. While it is highly challenging to create a representative sample of tasks, fortunately we can rely on existing resources created for other purposes as a reference. Specifically, we start by referencing the 29.1 release of O*NET database (O*NET, [2024](https://arxiv.org/html/2412.14161v1#bib.bib21); Rounds et al., [1999](https://arxiv.org/html/2412.14161v1#bib.bib26)), which is a database of jobs performed by workers in the US created by the US Department of Labor. It also contains information about tasks performed within the context of each job, abilities required to perform each task, whether the task is a major or minor task for that job category, and other pieces of relevant information.

Based on this data, we first identified a few categories of occupation categories to focus on. First, based on statistics from O*NET, we identified job categories that have a large number of people performing this job. Then, we used median salary information for each of these job categories from the US department of labor statistics, and multiplied the number of employees in that category to estimate the aggregate value of performing this job.

Based on this, we identified several categories of jobs such as “General and Operations Managers”, “Registered Nurses”, “Software Developers”, and “Financial Managers” that have both a high population and high average salary. Because TheAgentCompany is designed to be a non-embodied benchmark in the digital domain, we excluded the categories that require extensive physical labor such as “Registered Nurses”, and eventually settled on the setting of a software company, which would allow us to cover tasks from the other categories.

### 5.2 Choosing Tasks

Next, within this setting we chose tasks to implement. In this setting, we attempted to create a diversity of tasks, but mostly focused on concrete tasks that have well-defined goals and success criteria. These tasks were created through a combination of referencing the O*NET task list, introspection based on paper co-authors who had experience in each task category, and brainstorming lists with language models. It is important to note that *in no cases have we covered an extensive list of all the tasks that are performed in a particular occupational category*, and therefore we caution against making any assumptions about whether a particular *job* may be in danger of full automation based solely on TheAgentCompany. Rather, it may provide insight into whether certain *tasks within jobs* may be accelerated or automated, and inform further analysis by labor professionals into this question.

### 5.3 Manual Task Curation

Once we set up the environment required for our desired jobs and task categories ([§ 3](https://arxiv.org/html/2412.14161v1#S3 "3 TheAgentCompany Environment Setup ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")), we return to the curated list, and perform a manual curation process for tasks. For each task, this consists of the following steps: We first create a description of task intent, checkpoints, and how to evaluate each checkpoint. We then identify and import the required data for the task that are currently missing in the company Intranet services and create any necessary data. We then write scripts to configure the required initialization state in the local workspace. Finally, we implement the checkpoint evaluators that calculate the scalar scores for each checkpoint.

All tasks were created by coauthors of the paper. Overall, it took 20 computer science students, software engineers, and project managers over 2 months, consuming approximately 3,000 person-hours in total. Some of the more complex tasks take more than 10 hours each to design, implement, test, and verify. To ensure quality control of the task creation process, we implement several check and verification processes. For each task implementation, we require screenshot proof that the evaluator is valid and that the task is able to get a full score when successfully completed. We also encourage including tests for the implemented evaluator programs. Each task contribution is also code reviewed by a panel of lead authors before merging into the benchmark. After creating all tasks, a final round of manual human double-check of required environment data, evaluator behavior, and checkpoint scoring for every task is performed to ensure quality. Notably, during the process a person who has not curated the tasks checks all the checkpoint score assignments to make sure that the importance scoring is consistent over all the tasks and that it correlates reasonably with the relative importance of the checkpoint within the task.

## 6 Baseline Agent

![Refer to caption](img/4463827b1d68cb0a9d612241c2314e9d.png)

Figure 3: Overview of OpenHands’ default CodeAct + Browsing agent architecture, the baseline agent used throughout the experiments.

To test the current state-of-the-art performance on the TheAgentCompany benchmark, we need agents that can at least perform tasks using a browser, operate a local workspace using a terminal, and write and execute programs to perform most of the tasks. Throughout this paper, we experiment with OpenHands’ main agent (Wang et al., [2024b](https://arxiv.org/html/2412.14161v1#bib.bib32); [a](https://arxiv.org/html/2412.14161v1#bib.bib31); Song et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib28)), CodeAct Agent with Browsing.⁸⁸8More specifically, version 0.14.2\. Full details can be found in [https://github.com/All-Hands-AI/OpenHands/tree/main/openhands/agenthub/codeact_agent](https://github.com/All-Hands-AI/OpenHands/tree/main/openhands/agenthub/codeact_agent) An overview of the agent architecture is illustrated in [Figure 3](https://arxiv.org/html/2412.14161v1#S6.F3 "Figure 3 ‣ 6 Baseline Agent ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks").

#### Interfaces

The agent can interact with the environment through 3 interfaces. (1) A bash shell that connects with the local workspace operating system environment for command execution. (2) A Jupyter IPython server to handle interactive *python* ([IPython,](https://arxiv.org/html/2412.14161v1#bib.bib12) ) code execution requests and return the execution results back. (3) A Chromium browser based on [Playwright](https://arxiv.org/html/2412.14161v1#bib.bib25) . The provider provides a set of action primitives defined by BrowserGym ([ServiceNow,](https://arxiv.org/html/2412.14161v1#bib.bib27) ; Drouin et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib7)), such as navigation, clicking, typing, and scrolling. After executing these actions, the browser runtime provides a rich set of observations about the current state of the browser, including HTML, DOM, accessibility tree ([Mozilla,](https://arxiv.org/html/2412.14161v1#bib.bib20) ), screenshot, opened tabs, *etc*. These observations can be also augmented with configurable attributes that could allow agents to better understand web page observations, such as using a set-of-marks on screenshot (Yang et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib36); He et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib10)), visible element marking, focused element, interactable element marking, in-viewport element filtering (Zhou et al., [2023](https://arxiv.org/html/2412.14161v1#bib.bib39)), *etc*.

#### Actions

The agent connects with the environment through a core set of general actions. Actions IPythonRunCellAction and CmdRunAction enable the agent to execute arbitrary Python code and bash commands inside the sandbox environment (*e.g*., a secure isolated Linux operating system used as our local workspace). BrowserInteractiveAction enables interaction with a web browser with a domain-specific language for browsing introduced by BrowserGym (Chezelles et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib4); Drouin et al., [2024](https://arxiv.org/html/2412.14161v1#bib.bib7)). These actions provide a comprehensive, yet flexible set of primitives that cover most of the tasks performed by human employees of TheAgentCompany, including navigation, click, hovering, and typing, etc.

#### Observations

Observations describe the environmental changes that the agent observes. The main types of observations used in the CodeAct agent include the execution result of bash terminal commands, Python programs, and browser actions. Specifically, the execution result of browser actions is usually browser snapshots and textual representation in the form of accessibility tree of the current browser viewport.

#### Workflow

At each step, the underlying backbone LLM will take in prompts consisting of previous agent history and the current observation of the environment, and generate a response consisting of the action to execute next. On a higher level, the agent can perform the task by executing code, including executing bash commands, Python code, or browser-specific programming language (defined in BrowserGym).⁹⁹9[https://github.com/ServiceNow/BrowserGym/blob/main/browsergym/core/src/browsergym/core/action/functions.py](https://github.com/ServiceNow/BrowserGym/blob/main/browsergym/core/src/browsergym/core/action/functions.py) This general action space allows the agent to perform various tasks, including editing files, browsing the Web, running programs, etc.

## 7 Experimental Results

In this section, we evaluate popular foundation models, both closed and open, on TheAgentCompany benchmark. We use OpenHands CodeAct agent ([§ 6](https://arxiv.org/html/2412.14161v1#S6 "6 Baseline Agent ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")) for all experiments This serves as a baseline for future development of both the foundation LLMs and the agent infrastructure. Note that since LLM evaluators and NPCs are part of the environment rather than the agent being evaulated, we fix their backbone LLM to Claude-3-5-Sonnet-20241022, which demonstrated the best qualitative accuracy in simulating human colleagues and judging deliverables in preliminary experiments.

Table 3: Performance comparison of various foundation models on TheAgentCompany.

| Model | Success | Score | Steps | Costs |
| API-based Models |
| Claude-3.5-Sonnet | 24.0% | 34.4% | 29.17 | $6.34 |
| Gemini-2.0-Flash | 11.4% | 19.0% | 39.85 | $0.79 |
| GPT-4o | 8.6% | 16.7% | 14.55 | $1.29 |
| Gemini-1.5-Pro | 3.4% | 8.0% | 22.10 | $6.78 |
| Amazon-Nova-Pro-v1 | 1.7% | 5.7% | 19.59 | $1.55 |
| Open-weights Models |
| Llama-3.1-405b | 7.4% | 14.1% | 22.95 | $3.21 |
| Llama-3.3-70b | 6.9% | 12.8% | 20.93 | $0.93 |
| Qwen-2.5-72b | 5.7% | 11.8% | 23.99 | $1.53 |
| Llama-3.1-70b | 1.7% | 6.5% | 19.18 | $0.83 |
| Qwen-2-72b | 1.1% | 4.2% | 23.70 | $0.28 |

Table 4: Performance of the models in tasks that require different platforms in TheAgentCompany. All numbers are percentages (%).

 |  | GitLab (71 tasks) | Plane (17 tasks) | RocketChat (79 tasks) | ownCloud (70 tasks) |
| Model | Success (%) | Score (%) | Success (%) | Score (%) | Success (%) | Score (%) | Success (%) | Score (%) |
| API-based Models |
| Claude-3.5-Sonnet | 30.99 | 40.25 | 41.18 | 50.37 | 21.52 | 34.68 | 10.00 | 21.81 |
| Gemini-2.0-Flash | 11.27 | 18.21 | 17.65 | 29.84 | 13.92 | 23.34 | 2.86 | 8.52 |
| GPT-4o | 11.27 | 19.46 | 23.53 | 33.68 | 5.06 | 16.08 | 1.43 | 7.76 |
| Gemini-1.5-Pro | 2.82 | 3.88 | 5.88 | 14.05 | 3.80 | 10.97 | 0.00 | 4.22 |
| Amazon-Nova-Pro-v1 | 2.82 | 7.22 | 5.88 | 16.67 | 1.27 | 5.36 | 0.00 | 2.43 |
| Open-weights Models |
| Llama-3.1-405b | 5.63 | 11.84 | 29.41 | 39.12 | 8.86 | 16.46 | 0.00 | 4.45 |
| Llama-3.3-70b | 8.45 | 14.26 | 11.76 | 21.65 | 5.06 | 12.06 | 0.00 | 3.76 |
| Qwen-2.5-72b | 5.63 | 11.33 | 11.76 | 23.56 | 5.06 | 12.60 | 0.00 | 4.14 |
| Llama-3.1-70b | 1.41 | 6.09 | 5.88 | 15.35 | 2.53 | 8.23 | 0.00 | 3.32 |
| Qwen-2-72b | 1.41 | 1.94 | 5.88 | 12.45 | 0.00 | 4.88 | 0.00 | 2.60 | 

Table 5: Performance of various models in tasks with different nature in TheAgentCompany. All numbers are percentages (%).

 |  | SDE (69 tasks) | PM (28 tasks) | DS (14 tasks) | Admin (15 tasks) | HR (29 tasks) | Finance (12 tasks) | Other (8 tasks) |
| Model | Success | Score | Success | Score | Success | Score | Success | Score | Success | Score | Success | Score | Success | Score |
| API-based Models |
| Claude-3.5-Sonnet | 30.43 | 38.02 | 35.71 | 51.31 | 14.29 | 21.70 | 0.00 | 11.59 | 24.14 | 34.49 | 8.33 | 25.17 | 12.50 | 22.40 |
| Gemini-2.0-Flash | 13.04 | 18.99 | 17.86 | 31.71 | 0.00 | 6.49 | 6.67 | 15.20 | 17.24 | 23.08 | 0.00 | 4.31 | 0.00 | 10.05 |
| GPT-4o | 13.04 | 19.18 | 17.86 | 32.27 | 0.00 | 4.70 | 6.67 | 13.89 | 0.00 | 8.28 | 0.00 | 7.36 | 0.00 | 10.78 |
| Gemini-1.5-Pro | 4.35 | 5.64 | 3.57 | 13.19 | 0.00 | 4.82 | 6.67 | 9.92 | 3.45 | 11.42 | 0.00 | 2.78 | 0.00 | 8.07 |
| Amazon-Nova-Pro-v1 | 2.90 | 6.07 | 3.57 | 12.54 | 0.00 | 3.27 | 0.00 | 0.00 | 0.00 | 4.27 | 0.00 | 2.78 | 0.00 | 2.86 |
| Open-weights Models |
| Llama-3.1-405b | 5.80 | 11.33 | 21.43 | 35.62 | 0.00 | 5.42 | 0.00 | 3.33 | 6.90 | 12.56 | 0.00 | 5.00 | 12.50 | 17.45 |
| Llama-3.3-70b | 11.59 | 16.49 | 7.14 | 19.83 | 0.00 | 4.70 | 0.00 | 1.67 | 6.90 | 11.38 | 0.00 | 5.69 | 0.00 | 7.03 |
| Qwen-2.5-72b | 7.25 | 11.99 | 10.71 | 22.90 | 0.00 | 5.42 | 0.00 | 2.14 | 6.90 | 12.36 | 0.00 | 7.15 | 0.00 | 5.99 |
| Llama-3.1-70b | 1.45 | 4.77 | 3.57 | 15.16 | 0.00 | 5.42 | 0.00 | 2.42 | 3.45 | 7.19 | 0.00 | 3.82 | 0.00 | 2.86 |
| Qwen-2-72b | 2.90 | 3.68 | 0.00 | 7.44 | 0.00 | 4.70 | 0.00 | 0.56 | 0.00 | 4.14 | 0.00 | 3.61 | 0.00 | 4.95 | 

### 7.1 Result Overview

[Table 3](https://arxiv.org/html/2412.14161v1#S7.T3 "Table 3 ‣ 7 Experimental Results ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks") shows the evaluation results of both closed and open foundation models on the full evaluation set of TheAgentCompany (175 tasks). We can see that the Claude-3.5-Sonnet is the clear winner across all models. However, even with the strongest frontier model, it only manages to complete 24% of the total tasks and achieves a score of 34.4% taking into account partial completion credits. Note that this result comes at a cost: It requires an average of almost 30 steps and more than $6 to complete each task, making it the most expensive model to run both in time and in cost. This is expected as most of the tasks in our benchmark are of long-horizon nature. The Gemini 2.0 Flash model that comes second in terms of capability requires 40 steps on average to complete the tasks, which is time consuming, yet only to achieve less than half the success rate compared to the top-performing model. Surprisingly, its cost is less than $1, making it a very cost-efficient yet relatively strong model. A qualitative examination demonstrated that this was due to instances where the agent got stuck in a loop or aimlessly explored the environment.

Among the open-weight models, Llama 3.1 (405B) achieves the highest performance, nearly on par with OpenAI’s GPT-4o model, though still having a big gap behind the leading Claude 3.5 Sonnet. Interestingly, comparing the number of steps and costs between the open Llama 3.1 (405B) model and the closed OpenAI GPT-4o model, Llama 3.1 takes more steps and costs nearly 2x more to run, while having a lower success than GPT-4o. Anecdotally, our inspection showed that GPT-4o seems to be better at giving up early, saving steps and costs if the task is clearly out of the capacity range of the agent. This suggests that open-weight models are not always the most cost-effective choice in agents given serving cost, especially with highly complex tasks.

On the other hand, the newer generation of Llama model, Llama 3.3 (70B) achieves a considerably high performance of 6.9% success rate, on par with the much larger (405B), older generation (Llama 3.1) model. This model also costs significantly less because of its smaller size. This suggests a promising future for LLM development, as smaller and more efficient models begin to catch up in agent performance.

### 7.2 Analysis

#### How well do agents operate on different platforms?

[Table 4](https://arxiv.org/html/2412.14161v1#S7.T4 "Table 4 ‣ 7 Experimental Results ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks") presents performance breakdown on tasks that involve different platforms in TheAgentCompany. A task is categorized under a platform if one of the platforms that the task requires it. From [4(a)](https://arxiv.org/html/2412.14161v1#S7.F4.sf1 "4(a) ‣ Figure 4 ‣ How well do agents operate on different platforms? ‣ 7.2 Analysis ‣ 7 Experimental Results ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"), we can see that most models struggle with RocketChat and ownCloud. RocketChat platform is where all the social interaction with peers happens, and the low performance on this platform suggests that current-day LLMs still need improvements in communicating with others. ownCloud platform provides online Office suite functionality, and due to the complexity of the UI of web-based Office software, it is expected that current LLMs fail badly on the platform. This suggests that the browsing capability of the agents, especially on more complex websites, still needs improvement. These results underscore the inherent challenges and complexities of performing tasks that occur in real-world work environments, involve social interaction, or require understanding and navigating complex web interfaces.

![Refer to caption](img/bda35b463e43d861934112506cad943e.png)

(a) Success rate across platforms

![Refer to caption](img/4ddcac042e54bc5d6d333417e7d012a8.png)

(b) Success rate across task categories

Figure 4: Comparing agent success rate across platforms (left) and task categories (right).

#### How well do agents perform on different type of tasks?

[Table 5](https://arxiv.org/html/2412.14161v1#S7.T5 "Table 5 ‣ 7 Experimental Results ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks") presents performance breakdown on different types of tasks in TheAgentCompany. According to the nature of the task, *i.e*. what kind of professionals are usually assigned with the task, the tasks in TheAgentCompany can be categorized into several job departments: Software Development Engineering (SDE), Project Management (PM), Data Science (DS), Administrative (Admin), Human Resources (HR), Financial (Finance) and all the remaining (Other).

From the success rate demonstrated in [4(b)](https://arxiv.org/html/2412.14161v1#S7.F4.sf2 "4(b) ‣ Figure 4 ‣ How well do agents operate on different platforms? ‣ 7.2 Analysis ‣ 7 Experimental Results ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"), we can see that data science, administrative, and finance tasks are among the lowest, with many LLMs completing none of the tasks successfully, and even the strongest Claude model achieving much less than the rest of the tasks. On the other hand, software engineering tasks, which may seem like much harder tasks for many humans, result in a higher success rate. This suggests that there exists a gap between the perceived difficulty of the tasks for humans versus the difficulty for LLM agents.

For example, some tasks in the administrative and finance category involves making spreadsheets, collecting and filling in a lot of information from various people, or reading and understanding images scanned by employees. These tasks are arguably easier conceptually for humans in terms of professional skill sets than software engineering, as SDE jobs usually have a higher barrier of entry and more prerequisites for certain knowledge. However, most LLMs achieve a much higher score on the SDE tasks. However, LLMs fail these seemingly easier tasks due to lack of ability to understand documents, communicate with other people, navigate complex software and tedious processes, and autonomously automate repetitive tasks. We hypothesize that part of the reason lies in the fact that current LLM development is heavily based on software engineering abilities, such as coding, due to several high profile benchmarks that measure this capability (*e.g*. HumanEval, SWE-Bench) as well as the abundance of publicly available training data related to software. On the other hand, administrative and financial tasks, are usually private data within companies, not readily available for training LLMs.

### 7.3 Common Agent Failures

Overall, the agent performance on TheAgentCompany is still low and a majority of tasks are failed. Among those, we try to find some common and interesting agent mistakes that are often surprising because they are usually not made by humans.

#### Lack of commonsense

Some tasks are failed because the agent lacks the common sense and domain background knowledge required to infer implicit assumptions. For example, one task asked the agent to “Write the responses to /workspace/answer.docx” but does not explicitly states that this is a Microsoft Word file. A human can infer this requirement from the file extension. The agent instead treats it as a plain text file, writing text directly to the file, resulting in a task failure.

#### Lack of social skills

Sometimes, the agent fails to understand the implications and goals in the social conversations with colleagues in TheAgentCompany. For example, one task involves asking Alex for help, and the agent first successfully asks the right question “Could you tell me who I should introduce myself to next on the team?” Then the simulated colleague Alex replied “You should introduce yourself to Chen Xinyi next. She’s on our frontend team and would be a great person to connect with!” At this point, a human would then talk to Chen Xinyi, but instead the agent then decides to not follow up with her, and prematurely considers the task accomplished.

#### Incompetence in browsing

Often times, the biggest obstacle in tasks is the parts that require browsing the Web. This is expected as browsing is still hard for agents given the complexity of modern-day web UIs and the numerous distractions on a webpage. For example, on many tasks that involve ownCloud, the closable popup that sometimes shows up and asks the user to download the mobile phone apps for a better experience has become an obstacle. Humans can simply click on the ‘x’ to close the popup, while the agents are stuck. Similarly, when trying to download a file from ownCloud, there are several popups to click through before the actual download, and each step is error prone for agents due to the complex UI.

#### Deceiving oneself

Interestingly, we find that for some tasks, when the agent is not clear what the next steps should be, it sometimes try to be clever and create fake “shortcuts” that omit the hard part of a task. For example, during the execution of one task, the agent cannot find the right person to ask questions on RocketChat. As a result, it then decides to create a shortcut solution by renaming another user to the name of the intended user.

## 8 Implications and Future Directions

In this paper, we present TheAgentCompany, a new benchmark that stands out because it specifically focuses on real-world tasks that would be tackled within the context of real-world work. Unsurprisingly, current state-of-the-art agents fail to solve a majority of the tasks, suggesting that there is a big gap for current AI agents to autonomously perform most of the jobs a human worker would do, even in a relatively simplified benchmarking setting. Looking at how different models perform on different types of tasks, we argue that tasks that involve social interaction with other humans, navigating through complex user interfaces designed for professionals, and tasks that are typically performed in private, without a significant open and publicly available resources, are the most challenging. However, we believe that currently new LLMs are making significant progress: not only are they becoming more and more capable in terms of raw performance, but also more cost-efficient (*e.g*. Gemini 2.0 Flash). Open-weights models are closing the gap between proprietary frontier models too, and the newer models are getting smaller (*e.g*. Llama 3.3 70B) but with equivalent performance to previous huge models, also showcasing that efficiency will further improve.

That said, this is just a first step towards forming a firmer grasp on how AI may affect the tasks performed within a workspace, and it has its limitations. First, our tasks are generally on the more straightforward side due to the need to automatically evaluate with programs and test cases, and we do not cover more complex creative tasks such as brainstorming new product ideas or designing system architectures. Second, we are only using one agent scaffold as the baseline performance, and others may differ in performance. Third, while it would be interesting to know the actual performance of human professionals on these tasks to understand how LLM agents perform in comparison, due to resource limitations we were not able to perform this comparison in the current iteration of TheAgentCompany. Fourth, the topic and content of the tasks were mostly created through introspection by people familiar with these workspaces, which may result in some disconnect with actual tasks performed in enterprise settings.

Based on this, there are many future directions for further improvement of TheAgentCompany or other related benchmarks in this space. These include further expanding the benchmark tasks to those encountered in other industries, or tasks that require physical labor. Benchmarking may also be expanded with tasks that have more vague intents to better simulate real-world scenarios where the goal is not immediately clear at the very beginning. Further, benchmarks could also be expanded to include higher-level longer-horizon tasks such as conceptualizing a new product and carrying it to execution. We hope that TheAgentCompany provides a first step, but not the only step, towards these goals, and that we or others may build upon the open source release of TheAgentCompany to further expand in these promising directions.

## Author Contributions

This work was an open source collaborative effort between multiple institutions and many independent individuals. We used a point-based system to determine contributions and award authorship. Frank Xu, Boxuan Li and Yufan Song led the project, coordinating overall development and paper writing efforts. Detailed contributions were as follows:

*   •

    Task Design: Frank Xu, Yufan Song, Boxuan Li, Zora Wang, Shuyan Zhou, Graham Neubig

*   •

    Infrastructure Development: Yufan Song, Boxuan Li

*   •

    Experiment: Boxuan Li, Frank Xu, Yufan Song

*   •

    Sotopia Integration: Yufan Song, Xuhui Zhou

*   •

    Task Development: Boxuan Li, Yufan Song, Frank Xu, Graham Neubig, Yuxuan Tang, Mengxue Bao, Kritanjali Jain, Zhitong Guo, Murong Cao, Mingyang Yang, Hao Yang Lu, Amaad Martin, Zhe Su, Leander Maben, Raj Mehta, Yiqing Xie, Zora Wang, Xuhui Zhou, Wayne Chi, Lawrence Jang

*   •

    Ideation, Discussion and Formulation: Frank Xu, Shuyan Zhou, Xuhui Zhou, Zora Wang, Wayne Chi, Yufan Song, Boxuan Li, Lawrence Jang, Graham Neubig

*   •

    Advising: Graham Neubig advised the project, providing guidance, resources, and substantial paper editing.

## Acknowledgments

This work was supported by a grant from Open Philanthropy. We thank Daniel Fried, Ruslan Salakhutdinov, Chris Donahue, Jing Yu Koh, Brandon Trabucco, Julie Nys, Olga Rogunova, Aditya Soni, Alex Lawsen, and Stephen McAleer for their insightful discussions in various stages of the project. We also thank the OpenHands team for their help on some of the task implementations.

## References

*   Amodei & Fridman (2024) Dario Amodei and Lex Fridman. Dario Amodei: Anthropic CEO on Claude, AGI & the Future of AI & Humanity | Lex Fridman Podcast #452, November 2024. URL [https://lexfridman.com/dario-amodei-transcript](https://lexfridman.com/dario-amodei-transcript).
*   Anthropic (2023) Anthropic. The claude 3 model family: Opus, sonnet, haiku, 2023. URL [https://www-cdn.anthropic.com/de8ba9b01c9ab7cbabf5c33b80b7bbc618857627/Model_Card_Claude_3.pdf](https://www-cdn.anthropic.com/de8ba9b01c9ab7cbabf5c33b80b7bbc618857627/Model_Card_Claude_3.pdf).
*   Bonatti et al. (2024) Rogerio Bonatti, Dan Zhao, Francesco Bonacci, Dillon Dupont, Sara Abdali, Yinheng Li, Yadong Lu, Justin Wagle, Kazuhito Koishida, Arthur Bucker, et al. Windows agent arena: Evaluating multi-modal os agents at scale. *arXiv preprint arXiv:2409.08264*, 2024.
*   Chezelles et al. (2024) De Chezelles, Thibault Le Sellier, Maxime Gasse, Alexandre Lacoste, Alexandre Drouin, Massimo Caccia, Léo Boisvert, Megh Thakkar, Tom Marty, Rim Assouel, et al. The browsergym ecosystem for web agent research. *arXiv preprint arXiv:2412.05467*, 2024.
*   Chollet et al. (2024) François Chollet, Mike Knoop, Gregory Kamradt, and Bryan Landers. ARC Prize 2024: Technical Report, December 2024. URL [https://arcprize.org/media/arc-prize-2024-technical-report.pdf](https://arcprize.org/media/arc-prize-2024-technical-report.pdf).
*   Deng et al. (2023) Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samuel Stevens, Boshi Wang, Huan Sun, and Yu Su. Mind2web: Towards a generalist agent for the web. In *Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track*, 2023. URL [https://openreview.net/forum?id=kiYqbO3wqw](https://openreview.net/forum?id=kiYqbO3wqw).
*   Drouin et al. (2024) Alexandre Drouin, Maxime Gasse, Massimo Caccia, Issam H. Laradji, Manuel Del Verme, Tom Marty, Léo Boisvert, Megh Thakkar, Quentin Cappart, David Vazquez, Nicolas Chapados, and Alexandre Lacoste. Workarena: How capable are web agents at solving common knowledge work tasks?, 2024.
*   Dubey et al. (2024) Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan, et al. The llama 3 herd of models. *arXiv preprint arXiv:2407.21783*, 2024.
*   Eloundou et al. (2023) Tyna Eloundou, Sam Manning, Pamela Mishkin, and Daniel Rock. GPTs are GPTs: An Early Look at the Labor Market Impact Potential of Large Language Models, 2023.
*   He et al. (2024) Hongliang He, Wenlin Yao, Kaixin Ma, Wenhao Yu, Yong Dai, Hongming Zhang, Zhenzhong Lan, and Dong Yu. Webvoyager: Building an end-to-end web agent with large multimodal models. *arXiv preprint arXiv:2401.13919*, 2024.
*   Intelligence (2024) Amazon Artificial General Intelligence. The amazon nova family of models: Technical report and model card. *Amazon Technical Reports*, 2024. URL [https://www.amazon.science/publications/the-amazon-nova-family-of-models-technical-report-and-model-card](https://www.amazon.science/publications/the-amazon-nova-family-of-models-technical-report-and-model-card).
*   (12) IPython. Jupyter and the future of IPython — IPython. URL [https://ipython.org](https://ipython.org).
*   Jang et al. (2024) Lawrence Jang, Yinheng Li, Charles Ding, Justin Lin, Paul Pu Liang, Dan Zhao, Rogerio Bonatti, and Kazuhito Koishida. Videowebarena: Evaluating long context multimodal agents with video understanding web tasks, 2024. URL [https://arxiv.org/abs/2410.19100](https://arxiv.org/abs/2410.19100).
*   Jimenez et al. (2024) Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik R Narasimhan. SWE-bench: Can Language Models Resolve Real-world Github Issues? In *The Twelfth International Conference on Learning Representations*, 2024. URL [https://openreview.net/forum?id=VTF8yNQM66](https://openreview.net/forum?id=VTF8yNQM66).
*   Kambhampati et al. (2024) Subbarao Kambhampati, Karthik Valmeekam, Lin Guan, Mudit Verma, Kaya Stechly, Siddhant Bhambri, Lucas Saldyt, and Anil Murthy. Llms can’t plan, but can help planning in llm-modulo frameworks. *arXiv preprint arXiv:2402.01817*, 2024.
*   Koh et al. (2024) Jing Yu Koh, Robert Lo, Lawrence Jang, Vikram Duvvur, Ming Lim, Po-Yu Huang, Graham Neubig, Shuyan Zhou, Russ Salakhutdinov, and Daniel Fried. VisualWebArena: Evaluating multimodal agents on realistic visual web tasks. In *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*. Association for Computational Linguistics, 2024. URL [https://aclanthology.org/2024.acl-long.50](https://aclanthology.org/2024.acl-long.50).
*   Li et al. (2024) Bowen Li, Wenhan Wu, Ziwei Tang, Lin Shi, John Yang, Jinyang Li, Shunyu Yao, Chen Qian, Binyuan Hui, Qicheng Zhang, et al. Devbench: A comprehensive benchmark for software development. *arXiv preprint arXiv:2403.08604*, 2024.
*   Liu et al. (2018) Evan Zheran Liu, Kelvin Guu, Panupong Pasupat, Tianlin Shi, and Percy Liang. Reinforcement learning on web interfaces using workflow-guided exploration. In *International Conference on Learning Representations (ICLR)*, 2018. URL [https://arxiv.org/abs/1802.08802](https://arxiv.org/abs/1802.08802).
*   Lù et al. (2024) Xing Han Lù, Zdeněk Kasner, and Siva Reddy. Weblinx: Real-world website navigation with multi-turn dialogue, 2024. URL [https://arxiv.org/abs/2402.05930](https://arxiv.org/abs/2402.05930).
*   (20) Mozilla. Accessibility tree - MDN Web Docs Glossary: Definitions of Web-related terms | MDN. URL [https://developer.mozilla.org/en-US/docs/Glossary/Accessibility_tree](https://developer.mozilla.org/en-US/docs/Glossary/Accessibility_tree).
*   O*NET (2024) O*NET. The 29.1 release of the O*NET database, November 2024. URL [https://www.onetcenter.org/dictionary/29.1/excel/](https://www.onetcenter.org/dictionary/29.1/excel/).
*   OpenAI (2024) OpenAI. Introducing gpt-4o and more tools to chatgpt free users, 2024. URL [https://openai.com/index/gpt-4o-and-more-tools-to-chatgpt-free/](https://openai.com/index/gpt-4o-and-more-tools-to-chatgpt-free/).
*   Park et al. (2023) Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. Generative agents: Interactive simulacra of human behavior, 2023.
*   Patil et al. (2023) Shishir G. Patil, Tianjun Zhang, Xin Wang, and Joseph E. Gonzalez. Gorilla: Large language model connected with massive apis. *arXiv preprint arXiv:2305.15334*, 2023.
*   (25) Playwright. Fast and reliable end-to-end testing for modern web apps | Playwright. URL [https://playwright.dev/](https://playwright.dev/).
*   Rounds et al. (1999) James Rounds, Thomas Smith, Lawrence Hubert, Phil Lewis, and David Rivkin. Development of occupational interest profiles for o* net. *Raleigh, NC: National Center for O* NET Development*, 1999.
*   (27) ServiceNow. BrowserGym: a Gym Environment for Web Task Automation. URL [https://github.com/ServiceNow/BrowserGym](https://github.com/ServiceNow/BrowserGym).
*   Song et al. (2024) Yueqi Song, Frank Xu, Shuyan Zhou, and Graham Neubig. Beyond browsing: Api-based web agents. *arXiv preprint arXiv:2410.16464*, 2024.
*   Team et al. (2023) Gemini Team, Rohan Anil, Sebastian Borgeaud, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, Katie Millican, et al. Gemini: a family of highly capable multimodal models. *arXiv preprint arXiv:2312.11805*, 2023.
*   Trivedi et al. (2024) Harsh Trivedi, Tushar Khot, Mareike Hartmann, Ruskin Manku, Vinty Dong, Edward Li, Shashank Gupta, Ashish Sabharwal, and Niranjan Balasubramanian. AppWorld: A controllable world of apps and people for benchmarking interactive coding agents. In *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*. Association for Computational Linguistics, 2024. doi: 10.18653/v1/2024.acl-long.850. URL [https://aclanthology.org/2024.acl-long.850](https://aclanthology.org/2024.acl-long.850).
*   Wang et al. (2024a) Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, and Heng Ji. Executable Code Actions Elicit Better LLM Agents. In *ICML*, 2024a.
*   Wang et al. (2024b) Xingyao Wang, Boxuan Li, Yufan Song, Frank F. Xu, Xiangru Tang, Mingchen Zhuge, Jiayi Pan, Yueqi Song, Bowen Li, Jaskirat Singh, Hoang H. Tran, Fuqiang Li, Ren Ma, Mingzhang Zheng, Bill Qian, Yanjun Shao, Niklas Muennighoff, Yizhe Zhang, Binyuan Hui, Junyang Lin, Robert Brennan, Hao Peng, Heng Ji, and Graham Neubig. Openhands: An open platform for ai software developers as generalist agents, 2024b. URL [https://arxiv.org/abs/2407.16741](https://arxiv.org/abs/2407.16741).
*   Wittenstein (2024) Jeran Wittenstein. AI Can Only Do 5% of Jobs, Says MIT Economist Who Fears Crash, October 2024. URL [https://www.bloomberg.com/news/articles/2024-10-02/ai-can-only-do-5-of-jobs-says-mit-economist-who-fears-crash](https://www.bloomberg.com/news/articles/2024-10-02/ai-can-only-do-5-of-jobs-says-mit-economist-who-fears-crash).
*   Xie et al. (2024) Tianbao Xie, Danyang Zhang, Jixuan Chen, Xiaochuan Li, Siheng Zhao, Ruisheng Cao, Toh Jing Hua, Zhoujun Cheng, Dongchan Shin, Fangyu Lei, Yitao Liu, Yiheng Xu, Shuyan Zhou, Silvio Savarese, Caiming Xiong, Victor Zhong, and Tao Yu. OSWorld: Benchmarking multimodal agents for open-ended tasks in real computer environments. In *The Thirty-eight Conference on Neural Information Processing Systems Datasets and Benchmarks Track*, 2024. URL [https://openreview.net/forum?id=tN61DTr4Ed](https://openreview.net/forum?id=tN61DTr4Ed).
*   Yang et al. (2024) An Yang, Baosong Yang, Binyuan Hui, Bo Zheng, Bowen Yu, Chang Zhou, Chengpeng Li, Chengyuan Li, Dayiheng Liu, Fei Huang, et al. Qwen2 technical report. *arXiv preprint arXiv:2407.10671*, 2024.
*   Yang et al. (2023) Jianwei Yang, Hao Zhang, Feng Li, Xueyan Zou, Chunyuan Li, and Jianfeng Gao. Set-of-mark prompting unleashes extraordinary visual grounding in gpt-4v. *arXiv preprint arXiv:2310.11441*, 2023.
*   Yao et al. (2024) Shunyu Yao, Noah Shinn, Pedram Razavi, and Karthik Narasimhan. $\tau$-bench: A benchmark for tool-agent-user interaction in real-world domains. *arXiv preprint arXiv:2406.12045*, 2024.
*   Yoran et al. (2024) Ori Yoran, Samuel Joseph Amouyal, Chaitanya Malaviya, Ben Bogin, Ofir Press, and Jonathan Berant. Assistantbench: Can web agents solve realistic and time-consuming tasks?, 2024. URL [https://arxiv.org/abs/2407.15711](https://arxiv.org/abs/2407.15711).
*   Zhou et al. (2023) Shuyan Zhou, Frank F Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, et al. Webarena: A realistic web environment for building autonomous agents. In *The Twelfth International Conference on Learning Representations*, 2023.
*   Zhou et al. (2024) Xuhui Zhou, Hao Zhu, Leena Mathur, Ruohong Zhang, Haofei Yu, Zhengyang Qi, Louis-Philippe Morency, Yonatan Bisk, Daniel Fried, Graham Neubig, and Maarten Sap. SOTOPIA: Interactive evaluation for social intelligence in language agents. In *The Twelfth International Conference on Learning Representations*, 2024. URL [https://openreview.net/forum?id=mM7VurbA4r](https://openreview.net/forum?id=mM7VurbA4r).

## Appendix A More TheAgentCompany Environment Details

### A.1 TheAgentCompany Overview

[⬇](data:text/plain;base64,IyMgQ29tcGFueSBJbnRyb2R1Y3Rpb24KVGhlIEFnZW50IENvbXBhbnkgaXMgYW4gaW5ub3ZhdGl2ZSBzb2Z0d2FyZSBmaXJtIHNwZWNpYWxpemluZyBpbiBkaXN0cmlidXRlZCBzeXN0ZW1zLCBkYXRhYmFzZSB0ZWNobm9sb2dpZXMsIGFuZCBhcnRpZmljaWFsIGludGVsbGlnZW5jZS4gT3VyIGNvcmUgYnVzaW5lc3MgaW5jbHVkZXMgZGV2ZWxvcGluZyBhbmQgbWFpbnRhaW5pbmcgaGlnaC1wZXJmb3JtYW5jZSBkaXN0cmlidXRlZCBncmFwaCBkYXRhYmFzZXMsIHN0cmVhbWluZyBkYXRhYmFzZXMsIGFuZCBwcm92aWRpbmcgYWR2YW5jZWQgQUkgc29sdXRpb25zLgoKIyMgTWFpbiBQcm9kdWN0cyBhbmQgU2VydmljZXMKMS4gRGlzdHJpYnV0ZWQgR3JhcGggRGF0YWJhc2UgKGJhc2VkIG9uIEphbnVzR3JhcGgpCjIuIFN0cmVhbWluZyBEYXRhYmFzZSAoYmFzZWQgb24gUmlzaW5nV2F2ZSkKMy4gQUkgTW9kZWwgRGV2ZWxvcG1lbnQgYW5kIEluZmVyZW5jZSBQbGF0Zm9ybSAoYmFzZWQgb24gT3BlbkhhbmRzIGFuZCBsbGFtYS5jcHApCjQuIFdlYiBDcmF3bGVyIEZyYW1ld29yayAoYmFzZWQgb24gQ29sbHkpCjUuIERpc3RyaWJ1dGVkIFNlYXJjaCBFbmdpbmUgKGJhc2VkIG9uIE9wZW5TZWFyY2gpCjYuIExvdy1Db2RlIEV2ZW50LURyaXZlbiBBcHBsaWNhdGlvbiBQbGF0Zm9ybSAoYmFzZWQgb24gTm9kZS1SRUQpCgojIyBUZWNobm9sb2d5IFN0YWNrCi0gUHJvZ3JhbW1pbmcgTGFuZ3VhZ2VzOiBSdXN0LCBQeXRob24sIEMrKywgR28sIEphdmEKLSBEYXRhYmFzZXM6IEdyYXBoIGRhdGFiYXNlcywgU3RyZWFtaW5nIGRhdGFiYXNlcywgU2VhcmNoIGVuZ2luZXMKLSBBSS9NTDogTGFyZ2UgTGFuZ3VhZ2UgTW9kZWxzIChMTE0pCi0gT3RoZXJzOiBEaXN0cmlidXRlZCBzeXN0ZW1zLCBBUEkgZGV2ZWxvcG1lbnQsIERvY3VtZW50YXRpb24gbWFuYWdlbWVudAoKIyMgQ29tcGFueSBWaXNpb24KVG8gYmVjb21lIGEgZ2xvYmFsIGxlYWRlciBpbiBkaXN0cmlidXRlZCBzeXN0ZW1zIGFuZCBhcnRpZmljaWFsIGludGVsbGlnZW5jZSwgc29sdmluZyBjb21wbGV4IGRhdGEgcHJvY2Vzc2luZyBhbmQgYW5hbHlzaXMgY2hhbGxlbmdlcyB0aHJvdWdoIGlubm92YXRpdmUgdGVjaG5vbG9naWVzLgoKIyMgQ29tcGFueSBNaXNzaW9uClRvIHByb3ZpZGUgYnVzaW5lc3NlcyBhbmQgZGV2ZWxvcGVycyB3aXRoIHRoZSBtb3N0IGFkdmFuY2VkLCBlZmZpY2llbnQsIGFuZCB1c2VyLWZyaWVuZGx5IGRhdGEgcHJvY2Vzc2luZyBhbmQgQUkgdG9vbHMsIGRyaXZpbmcgdGVjaG5vbG9naWNhbCBpbm5vdmF0aW9uIGFuZCBtYXhpbWl6aW5nIHRoZSB2YWx1ZSBvZiBkYXRhLg==)##  Company  IntroductionThe  Agent  Company  is  an  innovative  software  firm  specializing  in  distributed  systems,  database  technologies,  and  artificial  intelligence.  Our  core  business  includes  developing  and  maintaining  high-performance  distributed  graph  databases,  streaming  databases,  and  providing  advanced  AI  solutions.##  Main  Products  and  Services1.  Distributed  Graph  Database  (based  on  JanusGraph)2.  Streaming  Database  (based  on  RisingWave)3.  AI  Model  Development  and  Inference  Platform  (based  on  OpenHands  and  llama.cpp)4.  Web  Crawler  Framework  (based  on  Colly)5.  Distributed  Search  Engine  (based  on  OpenSearch)6.  Low-Code  Event-Driven  Application  Platform  (based  on  Node-RED)##  Technology  Stack-  Programming  Languages:  Rust,  Python,  C++,  Go,  Java-  Databases:  Graph  databases,  Streaming  databases,  Search  engines-  AI/ML:  Large  Language  Models  (LLM)-  Others:  Distributed  systems,  API  development,  Documentation  management##  Company  VisionTo  become  a  global  leader  in  distributed  systems  and  artificial  intelligence,  solving  complex  data  processing  and  analysis  challenges  through  innovative  technologies.##  Company  MissionTo  provide  businesses  and  developers  with  the  most  advanced,  efficient,  and  user-friendly  data  processing  and  AI  tools,  driving  technological  innovation  and  maximizing  the  value  of  data.

### A.2 TheAgentCompany Employee Roster with Project Assignments and Slack Channels

[⬇](data:text/plain;base64,MS4gQUkgQWdlbnQgKEFnZW50IGVtcGxveWVlIGJlaW5nIHRlc3RlZCBpbiBUaGVBZ2VudENvbXBhbnkpCiAgIC0gUm9sZTogQWxsCiAgIC0gUmVzcG9uc2liaWxpdGllczogQWxsCiAgIC0gUHJvamVjdDogQWxsCiAgIC0gU2xhY2sgQ2hhbm5lbHM6IEFsbAoKCjIuIFNhcmFoIEpvaG5zb24gKEZlbWFsZSwgNDIgeWVhcnMgb2xkKQogICAtIFJvbGU6IENUTwogICAtIFJlc3BvbnNpYmlsaXRpZXM6IFRlY2huaWNhbCBzdHJhdGVneSBwbGFubmluZywgUiZEIHRlYW0gbGVhZGVyc2hpcCwgbmV3IHRlY2hub2xvZ3kgYXNzZXNzbWVudAogICAtIFByb2plY3Q6IE92ZXJzZWVzIGFsbCB0ZWNobmljYWwgcHJvamVjdHMKICAgLSBTbGFjayBDaGFubmVsczogQWxsIHRlY2huaWNhbCBjaGFubmVscywgI2dlbmVyYWwsICN0ZWNoLXRhbGsKCgozLiBMaSBNaW5nIChNYWxlLCAzNSB5ZWFycyBvbGQpCiAgIC0gUm9sZTogRGF0YWJhc2UgVGVhbSBQcm9qZWN0IE1hbmFnZXIKICAgLSBSZXNwb25zaWJpbGl0aWVzOiBNYW5hZ2luZyBkYXRhYmFzZSBwcm9qZWN0cywgcmVzb3VyY2UgY29vcmRpbmF0aW9uLCBlbnN1cmluZyB0aW1lbHkgZGVsaXZlcnkKICAgLSBTa2lsbHM6IEphdmEsIGRpc3RyaWJ1dGVkIHN5c3RlbXMKICAgLSBQcm9qZWN0OiBKYW51c0dyYXBoIChHcmFwaCBEYXRhYmFzZSkKICAgLSBTbGFjayBDaGFubmVsczogI3Byb2plY3QtZ3JhcGhkYiwgI2VuZ2luZWVyaW5nLCAjdGVjaC10YWxrCgo0LiBaaGFuZyBXZWkgKE1hbGUsIDMxIHllYXJzIG9sZCkKICAgLSBSb2xlOiBTZW5pb3IgU29mdHdhcmUgRW5naW5lZXIgKFN0cmVhbWluZyBEYXRhYmFzZSBUZWFtKQogICAtIFJlc3BvbnNpYmlsaXRpZXM6IERldmVsb3BpbmcgYW5kIG9wdGltaXppbmcgY29yZSBzdHJlYW1pbmcgZGF0YWJhc2UgZnVuY3Rpb25hbGl0aWVzCiAgIC0gU2tpbGxzOiBSdXN0LCBkYXRhYmFzZSBzeXN0ZW1zCiAgIC0gUHJvamVjdDogUmlzaW5nV2F2ZSAoU3RyZWFtaW5nIERhdGFiYXNlKQogICAtIFNsYWNrIENoYW5uZWxzOiAjcHJvamVjdC1zdHJlYW1kYiwgI2VuZ2luZWVyaW5nLCAjdGVjaC10YWxrCgo1LiBXYW5nIEZhbmcgKEZlbWFsZSwgMjggeWVhcnMgb2xkKQogICAtIFJvbGU6IEFJIFJlc2VhcmNoZXIgKEFJIFRlYW0pCiAgIC0gUmVzcG9uc2liaWxpdGllczogRGVzaWduaW5nIGFuZCBpbXBsZW1lbnRpbmcgbWFjaGluZSBsZWFybmluZyBtb2RlbHMsIG9wdGltaXppbmcgbW9kZWwgcGVyZm9ybWFuY2UKICAgLSBTa2lsbHM6IFB5dGhvbiwgbWFjaGluZSBsZWFybmluZywgTExNCiAgIC0gUHJvamVjdDogT3BlbkhhbmRzIChMTE0gcHJvamVjdCkKICAgLSBTbGFjayBDaGFubmVsczogI3Byb2plY3QtYWksICNlbmdpbmVlcmluZywgI3RlY2gtdGFsawoKNi4gTWlrZSBDaGVuIChNYWxlLCAzMyB5ZWFycyBvbGQpCiAgIC0gUm9sZTogU2VuaW9yIFNvZnR3YXJlIEVuZ2luZWVyIChBSSBUZWFtKQogICAtIFJlc3BvbnNpYmlsaXRpZXM6IERldmVsb3BpbmcgYW5kIG9wdGltaXppbmcgTExNIGluZmVyZW5jZSBlbmdpbmVzCiAgIC0gU2tpbGxzOiBDKyssIENVREEsIHBlcmZvcm1hbmNlIG9wdGltaXphdGlvbgogICAtIFByb2plY3Q6IGxsYW1hLmNwcCAoTExNIGluZmVyZW5jZSBwcm9qZWN0KQogICAtIFNsYWNrIENoYW5uZWxzOiAjcHJvamVjdC1haSwgI2VuZ2luZWVyaW5nLCAjdGVjaC10YWxrCgo3LiBFbWlseSBaaG91IChGZW1hbGUsIDI5IHllYXJzIG9sZCkKICAgLSBSb2xlOiBTb2Z0d2FyZSBFbmdpbmVlciAoV2ViIENyYXdsZXIgVGVhbSkKICAgLSBSZXNwb25zaWJpbGl0aWVzOiBEZXNpZ25pbmcgYW5kIGltcGxlbWVudGluZyB3ZWIgY3Jhd2xlciBmdW5jdGlvbmFsaXRpZXMKICAgLSBTa2lsbHM6IEdvLCBkaXN0cmlidXRlZCBzeXN0ZW1zCiAgIC0gUHJvamVjdDogQ29sbHkgKFdlYiBDcmF3bGVyIEZyYW1ld29yaykKICAgLSBTbGFjayBDaGFubmVsczogI3Byb2plY3Qtd2ViY3Jhd2xlciwgI2VuZ2luZWVyaW5nLCAjdGVjaC10YWxrCgo4LiBMaXUgUWlhbmcgKE1hbGUsIDM2IHllYXJzIG9sZCkKICAgLSBSb2xlOiBRdWFsaXR5IEFzc3VyYW5jZSBFbmdpbmVlcgogICAtIFJlc3BvbnNpYmlsaXRpZXM6IERldmVsb3BpbmcgdGVzdCBzdHJhdGVnaWVzLCBleGVjdXRpbmcgdGVzdHMsIGVuc3VyaW5nIHByb2R1Y3QgcXVhbGl0eQogICAtIFByb2plY3Q6IEFsbCBwcm9qZWN0cyAoZm9jdXNpbmcgb24gdGVzdGluZyBhbmQgcXVhbGl0eSkKICAgLSBTbGFjayBDaGFubmVsczogQWxsIHByb2plY3QgY2hhbm5lbHMsICNlbmdpbmVlcmluZywgI3RlY2gtdGFsawoKOS4gUHJpeWEgU2hhcm1hIChGZW1hbGUsIDI3IHllYXJzIG9sZCkKICAgLSBSb2xlOiBEb2N1bWVudGF0aW9uIEVuZ2luZWVyCiAgIC0gUmVzcG9uc2liaWxpdGllczogV3JpdGluZyB0ZWNobmljYWwgZG9jdW1lbnRhdGlvbiwgbWFpbnRhaW5pbmcgd2lraSwgaW1wcm92aW5nIGRvY3VtZW50YXRpb24gcHJvY2Vzc2VzCiAgIC0gUHJvamVjdDogRG9jdW1lbnRhdGlvbiAoV2lraSkKICAgLSBTbGFjayBDaGFubmVsczogQWxsIHByb2plY3QgY2hhbm5lbHMsICNlbmdpbmVlcmluZywgI3RlY2gtdGFsawoKMTAuIE1hcmsgSm9obnNvbiAoTWFsZSwgNDAgeWVhcnMgb2xkKQogICAgLSBSb2xlOiBTYWxlcyBEaXJlY3RvcgogICAgLSBSZXNwb25zaWJpbGl0aWVzOiBEZXZlbG9waW5nIHNhbGVzIHN0cmF0ZWdpZXMsIG1hbmFnaW5nIHNhbGVzIHRlYW0sIGV4cGFuZGluZyBjbGllbnQgcmVsYXRpb25zaGlwcwogICAgLSBQcm9qZWN0OiBOL0EgKFNhbGVzKQogICAgLSBTbGFjayBDaGFubmVsczogI3NhbGVzLW1hcmtldGluZywgI2dlbmVyYWwKCjExLiBKZXNzaWNhIExlZSAoRmVtYWxlLCAzMiB5ZWFycyBvbGQpCiAgICAtIFJvbGU6IE1hcmtldGluZyBNYW5hZ2VyCiAgICAtIFJlc3BvbnNpYmlsaXRpZXM6IERldmVsb3BpbmcgbWFya2V0aW5nIHN0cmF0ZWdpZXMsIG1hbmFnaW5nIGJyYW5kIGltYWdlLCBvcmdhbml6aW5nIG1hcmtldGluZyBldmVudHMKICAgIC0gUHJvamVjdDogTi9BIChNYXJrZXRpbmcpCiAgICAtIFNsYWNrIENoYW5uZWxzOiAjc2FsZXMtbWFya2V0aW5nLCAjZ2VuZXJhbAoKMTIuIENoZW4gWGlueWkgKEZlbWFsZSwgMzAgeWVhcnMgb2xkKQogICAgLSBSb2xlOiBIdW1hbiBSZXNvdXJjZXMgTWFuYWdlcgogICAgLSBSZXNwb25zaWJpbGl0aWVzOiBSZWNydWl0bWVudCwgZW1wbG95ZWUgdHJhaW5pbmcsIGNvbXBlbnNhdGlvbiBtYW5hZ2VtZW50CiAgICAtIFByb2plY3Q6IE4vQSAoSFIpCiAgICAtIFNsYWNrIENoYW5uZWxzOiAjaHItYW5ub3VuY2VtZW50cywgI2dlbmVyYWwKCjEzLiBEYXZpZCBXb25nIChNYWxlLCA0NSB5ZWFycyBvbGQpCiAgICAtIFJvbGU6IEZpbmFuY2UgRGlyZWN0b3IKICAgIC0gUmVzcG9uc2liaWxpdGllczogRmluYW5jaWFsIHBsYW5uaW5nLCBidWRnZXQgbWFuYWdlbWVudCwgZmluYW5jaWFsIHJlcG9ydGluZwogICAgLSBQcm9qZWN0OiBOL0EgKEZpbmFuY2UpCiAgICAtIFNsYWNrIENoYW5uZWxzOiAjZ2VuZXJhbAoKMTQuIEh1YW5nIEppZSAoTWFsZSwgMzQgeWVhcnMgb2xkKQogICAgLSBSb2xlOiBQcm9kdWN0IE1hbmFnZXIgKFNlYXJjaCBFbmdpbmUgVGVhbSkKICAgIC0gUmVzcG9uc2liaWxpdGllczogRGVmaW5pbmcgcHJvZHVjdCByZXF1aXJlbWVudHMsIHBsYW5uaW5nIHByb2R1Y3Qgcm9hZG1hcCwgY29tbXVuaWNhdGluZyB3aXRoIGNsaWVudHMKICAgIC0gUHJvamVjdDogT3BlblNlYXJjaCAoU2VhcmNoIEVuZ2luZSkKICAgIC0gU2xhY2sgQ2hhbm5lbHM6ICNwcm9qZWN0LXNlYXJjaCwgI3Byb2R1Y3QsICN0ZWNoLXRhbGsKCjE1LiBTb3BoaWEgUm9kcmlndWV6IChGZW1hbGUsIDM3IHllYXJzIG9sZCkKICAgIC0gUm9sZTogVVggRGVzaWduZXIKICAgIC0gUmVzcG9uc2liaWxpdGllczogRGVzaWduaW5nIHVzZXIgaW50ZXJmYWNlcywgaW1wcm92aW5nIHVzZXIgZXhwZXJpZW5jZSwgY29uZHVjdGluZyB1c2VyIHJlc2VhcmNoCiAgICAtIFByb2plY3Q6IEFsbCBwcm9qZWN0cyAoZm9jdXNpbmcgb24gdXNlciBleHBlcmllbmNlKQogICAgLSBTbGFjayBDaGFubmVsczogQWxsIHByb2plY3QgY2hhbm5lbHMsICNwcm9kdWN0LCAjdGVjaC10YWxrCgoxNi4gQWxleCBUdXJuZXIgKE1hbGUsIDMwIHllYXJzIG9sZCkKICAgIC0gUm9sZTogU29mdHdhcmUgRW5naW5lZXIgKExvdy1Db2RlIFBsYXRmb3JtIFRlYW0pCiAgICAtIFByb2plY3Q6IE5vZGUtUkVEIChMb3ctQ29kZSBQbGF0Zm9ybSkKICAgIC0gU2xhY2sgQ2hhbm5lbHM6ICNwcm9qZWN0LWxvd2NvZGUsICNlbmdpbmVlcmluZywgI3RlY2gtdGFsawoKMTcuIEVtbWEgTGV3aXMgKEZlbWFsZSwgMzMgeWVhcnMgb2xkKQogICAgLSBSb2xlOiBTb2Z0d2FyZSBFbmdpbmVlciAoQVBJIFRlYW0pCiAgICAtIFByb2plY3Q6IEFQSS1zZXJ2ZXIgKFB5dGhvbiBwcm9qZWN0KQogICAgLSBTbGFjayBDaGFubmVsczogI2VuZ2luZWVyaW5nLCAjdGVjaC10YWxrCgoxOC4gSmVzc2ljYSBDaGVuIChGZW1hbGUsIDI4IHllYXJzIG9sZCkKICAgIC0gUm9sZTogRnJvbnRlbmQgU29mdHdhcmUgRW5naW5lZXIKICAgIC0gUmVzcG9uc2liaWxpdGllczogRGV2ZWxvcGluZyB1c2VyIGludGVyZmFjZXMsIGltcGxlbWVudGluZyByZXNwb25zaXZlIGRlc2lnbnMsIG9wdGltaXppbmcgd2ViIHBlcmZvcm1hbmNlCiAgICAtIFByb2plY3Q6IEUtY29tbWVyY2UgV2Vic2l0ZSBSZWRlc2lnbgogICAgLSBTbGFjayBDaGFubmVsczogI3Byb2plY3QtZWNvbW1lcmNlLCAjZnJvbnRlbmQsICN0ZWNoLXRhbGs=)1.  AI  Agent  (Agent  employee  being  tested  in  TheAgentCompany)-  Role:  All-  Responsibilities:  All-  Project:  All-  Slack  Channels:  All2.  Sarah  Johnson  (Female,  42  years  old)-  Role:  CTO-  Responsibilities:  Technical  strategy  planning,  R&D  team  leadership,  new  technology  assessment-  Project:  Oversees  all  technical  projects-  Slack  Channels:  All  technical  channels,  #general,  #tech-talk3.  Li  Ming  (Male,  35  years  old)-  Role:  Database  Team  Project  Manager-  Responsibilities:  Managing  database  projects,  resource  coordination,  ensuring  timely  delivery-  Skills:  Java,  distributed  systems-  Project:  JanusGraph  (Graph  Database)-  Slack  Channels:  #project-graphdb,  #engineering,  #tech-talk4.  Zhang  Wei  (Male,  31  years  old)-  Role:  Senior  Software  Engineer  (Streaming  Database  Team)-  Responsibilities:  Developing  and  optimizing  core  streaming  database  functionalities-  Skills:  Rust,  database  systems-  Project:  RisingWave  (Streaming  Database)-  Slack  Channels:  #project-streamdb,  #engineering,  #tech-talk5.  Wang  Fang  (Female,  28  years  old)-  Role:  AI  Researcher  (AI  Team)-  Responsibilities:  Designing  and  implementing  machine  learning  models,  optimizing  model  performance-  Skills:  Python,  machine  learning,  LLM-  Project:  OpenHands  (LLM  project)-  Slack  Channels:  #project-ai,  #engineering,  #tech-talk6.  Mike  Chen  (Male,  33  years  old)-  Role:  Senior  Software  Engineer  (AI  Team)-  Responsibilities:  Developing  and  optimizing  LLM  inference  engines-  Skills:  C++,  CUDA,  performance  optimization-  Project:  llama.cpp  (LLM  inference  project)-  Slack  Channels:  #project-ai,  #engineering,  #tech-talk7.  Emily  Zhou  (Female,  29  years  old)-  Role:  Software  Engineer  (Web  Crawler  Team)-  Responsibilities:  Designing  and  implementing  web  crawler  functionalities-  Skills:  Go,  distributed  systems-  Project:  Colly  (Web  Crawler  Framework)-  Slack  Channels:  #project-webcrawler,  #engineering,  #tech-talk8.  Liu  Qiang  (Male,  36  years  old)-  Role:  Quality  Assurance  Engineer-  Responsibilities:  Developing  test  strategies,  executing  tests,  ensuring  product  quality-  Project:  All  projects  (focusing  on  testing  and  quality)-  Slack  Channels:  All  project  channels,  #engineering,  #tech-talk9.  Priya  Sharma  (Female,  27  years  old)-  Role:  Documentation  Engineer-  Responsibilities:  Writing  technical  documentation,  maintaining  wiki,  improving  documentation  processes-  Project:  Documentation  (Wiki)-  Slack  Channels:  All  project  channels,  #engineering,  #tech-talk10.  Mark  Johnson  (Male,  40  years  old)-  Role:  Sales  Director-  Responsibilities:  Developing  sales  strategies,  managing  sales  team,  expanding  client  relationships-  Project:  N/A  (Sales)-  Slack  Channels:  #sales-marketing,  #general11.  Jessica  Lee  (Female,  32  years  old)-  Role:  Marketing  Manager-  Responsibilities:  Developing  marketing  strategies,  managing  brand  image,  organizing  marketing  events-  Project:  N/A  (Marketing)-  Slack  Channels:  #sales-marketing,  #general12.  Chen  Xinyi  (Female,  30  years  old)-  Role:  Human  Resources  Manager-  Responsibilities:  Recruitment,  employee  training,  compensation  management-  Project:  N/A  (HR)-  Slack  Channels:  #hr-announcements,  #general13.  David  Wong  (Male,  45  years  old)-  Role:  Finance  Director-  Responsibilities:  Financial  planning,  budget  management,  financial  reporting-  Project:  N/A  (Finance)-  Slack  Channels:  #general14.  Huang  Jie  (Male,  34  years  old)-  Role:  Product  Manager  (Search  Engine  Team)-  Responsibilities:  Defining  product  requirements,  planning  product  roadmap,  communicating  with  clients-  Project:  OpenSearch  (Search  Engine)-  Slack  Channels:  #project-search,  #product,  #tech-talk15.  Sophia  Rodriguez  (Female,  37  years  old)-  Role:  UX  Designer-  Responsibilities:  Designing  user  interfaces,  improving  user  experience,  conducting  user  research-  Project:  All  projects  (focusing  on  user  experience)-  Slack  Channels:  All  project  channels,  #product,  #tech-talk16.  Alex  Turner  (Male,  30  years  old)-  Role:  Software  Engineer  (Low-Code  Platform  Team)-  Project:  Node-RED  (Low-Code  Platform)-  Slack  Channels:  #project-lowcode,  #engineering,  #tech-talk17.  Emma  Lewis  (Female,  33  years  old)-  Role:  Software  Engineer  (API  Team)-  Project:  API-server  (Python  project)-  Slack  Channels:  #engineering,  #tech-talk18.  Jessica  Chen  (Female,  28  years  old)-  Role:  Frontend  Software  Engineer-  Responsibilities:  Developing  user  interfaces,  implementing  responsive  designs,  optimizing  web  performance-  Project:  E-commerce  Website  Redesign-  Slack  Channels:  #project-ecommerce,  #frontend,  #tech-talk

### A.3 TheAgentCompany Q3 2024 Quarterly Sprint Goals

[⬇](data:text/plain;base64,IyMgRW5naW5lZXJpbmcgVGVhbXMKMS4gR3JhcGggRGF0YWJhc2UgVGVhbSAoSmFudXNHcmFwaCkKICAgLSBPcHRpbWl6ZSBsYXJnZS1zY2FsZSBncmFwaCBxdWVyeSBwZXJmb3JtYW5jZQogICAtIEltcGxlbWVudCBuZXcgZ3JhcGggYW5hbHlzaXMgYWxnb3JpdGhtcwogICAtIEltcHJvdmUgc3RhYmlsaXR5IG9mIGRpc3RyaWJ1dGVkIGRlcGxveW1lbnRzCgoyLiBTdHJlYW1pbmcgRGF0YWJhc2UgVGVhbSAoUmlzaW5nV2F2ZSkKICAgLSBJbXBsZW1lbnQgbmV3IHN0cmVhbSBwcm9jZXNzaW5nIG9wZXJhdG9ycwogICAtIE9wdGltaXplIG1lbW9yeSB1c2FnZQogICAtIEltcHJvdmUgZmF1bHQgcmVjb3ZlcnkgbWVjaGFuaXNtcwoKMy4gQUkgVGVhbSAoT3BlbkhhbmRzICYgbGxhbWEuY3BwKQogICAtIEludGVncmF0ZSBsYXRlc3QgTExNIG1vZGVscwogICAtIE9wdGltaXplIG1vZGVsIGluZmVyZW5jZSBzcGVlZAogICAtIERldmVsb3AgbW9kZWwgZmluZS10dW5pbmcgZnVuY3Rpb25hbGl0eQoKNC4gV2ViIENyYXdsZXIgVGVhbSAoQ29sbHkpCiAgIC0gSW1wbGVtZW50IGRpc3RyaWJ1dGVkIGNyYXdsaW5nIGZ1bmN0aW9uYWxpdHkKICAgLSBJbXByb3ZlIGFudGktY3Jhd2xpbmcgZGV0ZWN0aW9uIGFuZCBieXBhc3MgbWVjaGFuaXNtcwogICAtIERldmVsb3AgZGF0YSBjbGVhbmluZyBhbmQgcHJlcHJvY2Vzc2luZyBtb2R1bGVzCgo1LiBTZWFyY2ggRW5naW5lIFRlYW0gKE9wZW5TZWFyY2gpCiAgIC0gT3B0aW1pemUgZnVsbC10ZXh0IHNlYXJjaCBwZXJmb3JtYW5jZQogICAtIEltcGxlbWVudCBuZXcgcmVsZXZhbmNlIHJhbmtpbmcgYWxnb3JpdGhtcwogICAtIERldmVsb3AgY3VzdG9tIGFuYWx5emVyIGZ1bmN0aW9uYWxpdHkKCjYuIExvdy1Db2RlIFBsYXRmb3JtIFRlYW0gKE5vZGUtUkVEKQogICAtIERlc2lnbiBuZXcgdmlzdWFsIGNvbXBvbmVudHMKICAgLSBJbXByb3ZlIHdvcmtmbG93IGV4ZWN1dGlvbiBlbmdpbmUKICAgLSBEZXZlbG9wIG1vcmUgdGhpcmQtcGFydHkgc2VydmljZSBpbnRlZ3JhdGlvbnMKCiMjIFByb2R1Y3QgVGVhbQotIENvbmR1Y3QgdXNlciByZXNlYXJjaCwgY29sbGVjdCBwcm9kdWN0IGZlZWRiYWNrCi0gRGV2ZWxvcCBRNCBwcm9kdWN0IHJvYWRtYXAKLSBPcHRpbWl6ZSBwcm9kdWN0IGRvY3VtZW50YXRpb24gYW5kIHVzZXIgZ3VpZGVzCgojIyBRdWFsaXR5IEFzc3VyYW5jZSBUZWFtCi0gRGV2ZWxvcCBhdXRvbWF0ZWQgdGVzdCBzdWl0ZXMKLSBDb25kdWN0IHBlcmZvcm1hbmNlIGFuZCBsb2FkIHRlc3RpbmcKLSBJbXByb3ZlIGJ1ZyB0cmFja2luZyBhbmQgcmVwb3J0aW5nIHByb2Nlc3NlcwoKIyMgU2FsZXMgYW5kIE1hcmtldGluZyBUZWFtCi0gT3JnYW5pemUgaW5kdXN0cnkgdHJhZGUgc2hvdyBwYXJ0aWNpcGF0aW9uCi0gTGF1bmNoIG5ldyBjb250ZW50IG1hcmtldGluZyBjYW1wYWlnbnMKLSBEZXZlbG9wIHNhbGVzIHRlYW0gdHJhaW5pbmcgcHJvZ3JhbXMKCiMjIEh1bWFuIFJlc291cmNlcyBUZWFtCi0gSW1wbGVtZW50IG5ldyBlbXBsb3llZSBkZXZlbG9wbWVudCBwbGFucwotIE9wdGltaXplIHJlY3J1aXRtZW50IHByb2Nlc3NlcwotIE9yZ2FuaXplIHRlYW0tYnVpbGRpbmcgYWN0aXZpdGllcwoKIyMgRmluYW5jZSBUZWFtCi0gUHJlcGFyZSBRMiBmaW5hbmNpYWwgcmVwb3J0cwotIERldmVsb3AgUTQgYnVkZ2V0IHBsYW5zCi0gT3B0aW1pemUgZmluYW5jaWFsIGFuYWx5c2lzIHRvb2xz)##  Engineering  Teams1.  Graph  Database  Team  (JanusGraph)-  Optimize  large-scale  graph  query  performance-  Implement  new  graph  analysis  algorithms-  Improve  stability  of  distributed  deployments2.  Streaming  Database  Team  (RisingWave)-  Implement  new  stream  processing  operators-  Optimize  memory  usage-  Improve  fault  recovery  mechanisms3.  AI  Team  (OpenHands  &  llama.cpp)-  Integrate  latest  LLM  models-  Optimize  model  inference  speed-  Develop  model  fine-tuning  functionality4.  Web  Crawler  Team  (Colly)-  Implement  distributed  crawling  functionality-  Improve  anti-crawling  detection  and  bypass  mechanisms-  Develop  data  cleaning  and  preprocessing  modules5.  Search  Engine  Team  (OpenSearch)-  Optimize  full-text  search  performance-  Implement  new  relevance  ranking  algorithms-  Develop  custom  analyzer  functionality6.  Low-Code  Platform  Team  (Node-RED)-  Design  new  visual  components-  Improve  workflow  execution  engine-  Develop  more  third-party  service  integrations##  Product  Team-  Conduct  user  research,  collect  product  feedback-  Develop  Q4  product  roadmap-  Optimize  product  documentation  and  user  guides##  Quality  Assurance  Team-  Develop  automated  test  suites-  Conduct  performance  and  load  testing-  Improve  bug  tracking  and  reporting  processes##  Sales  and  Marketing  Team-  Organize  industry  trade  show  participation-  Launch  new  content  marketing  campaigns-  Develop  sales  team  training  programs##  Human  Resources  Team-  Implement  new  employee  development  plans-  Optimize  recruitment  processes-  Organize  team-building  activities##  Finance  Team-  Prepare  Q2  financial  reports-  Develop  Q4  budget  plans-  Optimize  financial  analysis  tools

### A.4 TheAgentCompany Internal Documents

[⬇](data:text/plain;base64,RW1wbG95ZWUgSGFuZGJvb2sKQ29tcGFueSBQb2xpY2llcyBhbmQgUHJvY2VkdXJlcyBEb2N1bWVudApQYXlyb2xsIGFuZCBDb21wZW5zYXRpb24gU3RydWN0dXJlIERvY3VtZW50ClBlcmZvcm1hbmNlIEV2YWx1YXRpb24gRm9ybXMgYW5kIEd1aWRlbGluZXMKUHJvamVjdCBNYW5hZ2VtZW50IFRlbXBsYXRlcyAoaW5jbHVkaW5nIEdhbnR0IGNoYXJ0cywgcmlzayBhc3Nlc3NtZW50IGZvcm1zLCBldGMuKQpUZWNobmljYWwgQXJjaGl0ZWN0dXJlIERvY3VtZW50YXRpb24KQ29kaW5nIFN0YW5kYXJkcyBhbmQgQmVzdCBQcmFjdGljZXMgR3VpZGUKUHJvZHVjdCBSb2FkbWFwCk1hcmtldGluZyBTdHJhdGVneSBEb2N1bWVudApTYWxlcyBQcm9jZXNzIGFuZCBDUk0gVXNhZ2UgR3VpZGUKRmluYW5jaWFsIFJlcG9ydGluZyBUZW1wbGF0ZXMKQnVkZ2V0IFBsYW5uaW5nIERvY3VtZW50Ckh1bWFuIFJlc291cmNlcyBQb2xpY2llcyAoaW5jbHVkaW5nIHJlY3J1aXRtZW50LCB0cmFpbmluZywgcHJvbW90aW9uLCBldGMuKQpJVCBTZWN1cml0eSBQb2xpY2llcyBhbmQgR3VpZGVsaW5lcwpDdXN0b21lciBTdXBwb3J0IFByb2Nlc3MgRG9jdW1lbnRhdGlvbg==)Employee  HandbookCompany  Policies  and  Procedures  DocumentPayroll  and  Compensation  Structure  DocumentPerformance  Evaluation  Forms  and  GuidelinesProject  Management  Templates  (including  Gantt  charts,  risk  assessment  forms,  etc.)Technical  Architecture  DocumentationCoding  Standards  and  Best  Practices  GuideProduct  RoadmapMarketing  Strategy  DocumentSales  Process  and  CRM  Usage  GuideFinancial  Reporting  TemplatesBudget  Planning  DocumentHuman  Resources  Policies  (including  recruitment,  training,  promotion,  etc.)IT  Security  Policies  and  GuidelinesCustomer  Support  Process  Documentation

## Appendix B Agent-Simulated Colleagues Conversation Examples

We present some examples (see [Figure 5](https://arxiv.org/html/2412.14161v1#A2.F5 "Figure 5 ‣ Appendix B Agent-Simulated Colleagues Conversation Examples ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks"), [Figure 6](https://arxiv.org/html/2412.14161v1#A2.F6 "Figure 6 ‣ Appendix B Agent-Simulated Colleagues Conversation Examples ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks") and [Figure 7](https://arxiv.org/html/2412.14161v1#A2.F7 "Figure 7 ‣ Appendix B Agent-Simulated Colleagues Conversation Examples ‣ TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks")) of the agent’s interaction with the simulated colleagues within our environment.

![Refer to caption](img/625287dd054db7668c65bf9769422a49.png)

Figure 5: Simulated Colleague Communication Example 1 – The agent is tasked with collecting required equipment while adhering to the department’s budget. After calculating that the requested items exceed the budget, the agent negotiates with the simulated colleague to reduce the request, showcasing its ability of effective communication.

![Refer to caption](img/a6a6a9f4541a11eecab5020232b7b259.png)

Figure 6: Simulated Colleague Communication Example 2 – The agent is tasked with writing a job description for a new graduate software engineering position. To fulfill the task, the agent communicates with simulated Project Manager to gather requirements. The agent requests the job description template, minimum and preferred qualifications, and the ideal salary range. This interaction evaluates the agent’s ability to gather information systematically and clarify task-related requirements through effective communication.

![Refer to caption](img/2e62de78cdf76158e13bf2486bcc613f.png)

Figure 7: Simulated Colleague Communication Example 3 - The agent is tasked with scheduling a meeting between NPCs Emily Zhou and Liu Qiang based on their availability. Emily is available on Wednesday and Thursday, while Liu is only available on Thursday. The agent identifies Thursday as the common free day and successfully proposes a mid-morning slot at 10:30 AM, which both participants confirm. This example highlights the agent’s ability to manage multi-turn conversations, effectively going back and forth between participants to align schedules and finalize a meeting time.