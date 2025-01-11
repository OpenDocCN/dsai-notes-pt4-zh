<!--yml
category: 未分类
date: 2025-01-11 12:43:09
-->

# AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications

> 来源：[https://arxiv.org/html/2404.04902/](https://arxiv.org/html/2404.04902/)

Xin Pang^(1†), Zhucong Li^(2,4†), Jiaxiang Chen^(2,4), Yuan Cheng^(2,3), Yinghui Xu², Yuan Qi^(2,3)
${}^{1}$ ContinuAI ${}^{2}$ Artificial Intelligence Innovation and Incubation Institute,
Fudan University, Shanghai, China
${}^{3}$ Shanghai Academy of Artificial Intelligence for Science, Shanghai, China
${}^{4}$ School of Computer Science, Fudan University, Shanghai, China
pxavdpro@gmail.com, {zcli22, jiaxiangchen23}@m.fudan.edu.cn
{cheng_yuan, xuyinghui, qiyuan}@fudan.edu.cn

###### Abstract

We introduce AI2Apps, a Visual Integrated Development Environment (Visual IDE) with full-cycle capabilities that accelerates developers to build deployable LLM-based AI agent Applications. This Visual IDE prioritizes both the Integrity of its development tools and the Visuality of its components, ensuring a smooth and efficient building experience. On one hand, AI2Apps integrates a comprehensive development toolkit—ranging from a prototyping canvas and AI-assisted code editor to agent debugger, management system, and deployment tools—all within a web-based graphical user interface. On the other hand, AI2Apps visualizes reusable front-end and back-end code as intuitive drag-and-drop components. Furthermore, a plugin system named AI2Apps Extension (AAE) is designed for Extensibility, showcasing how a new plugin with 20 components enables web agent to mimic human-like browsing behavior. Our case study demonstrates substantial efficiency improvements, with AI2Apps reducing token consumption and API calls when debugging a specific sophisticated multimodal agent by approximately 90% and 80%, respectively. The AI2Apps, including an online demo¹¹1[https://www.ai2apps.com](https://www.ai2apps.com), open-source code²²2[https://github.com/Avdpro/ai2apps](https://github.com/Avdpro/ai2apps), and a screencast video³³3[https://youtu.be/tQTqxk1LzzU](https://youtu.be/tQTqxk1LzzU), is now publicly accessible.

²²footnotetext: Corresponding author.

## 1 Introduction

The advent of Large Language Models (LLMs) has significantly advanced the technology of AI agents, paving the way for next-generation applications. Nonetheless, developers are in urgent need of a comprehensive full-stack solution to alleviate the repetitive tasks and escalating costs they face. However, despite achieving remarkable success, existing efforts to develop LLM-based AI agent applications still face their respective limitations. LLM Operations (LLMOps) platforms often lack integration with engineering-level tools designed for professional developers, thereby limiting the flexibility in programming and debugging (Openai, [2023](https://arxiv.org/html/2404.04902v1#bib.bib27); LangChain, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib15); Microsoft, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib19); Baidubce, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib2); ByteDance, [2023](https://arxiv.org/html/2404.04902v1#bib.bib7); Gao et al., [2024](https://arxiv.org/html/2404.04902v1#bib.bib11); Xie et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib34); LangGenius, [2023](https://arxiv.org/html/2404.04902v1#bib.bib16); Logspace, [2023](https://arxiv.org/html/2404.04902v1#bib.bib18); FlowiseAI, [2023](https://arxiv.org/html/2404.04902v1#bib.bib10); Dataelement, [2023](https://arxiv.org/html/2404.04902v1#bib.bib9)). Integrated Development Environments (IDEs) fall short in providing sufficient reusable visual components and remain cumbersome and time-consuming throughout the development process. (Microsoft, [2023c](https://arxiv.org/html/2404.04902v1#bib.bib21), [e](https://arxiv.org/html/2404.04902v1#bib.bib23)). Software Development Kits (SDKs), serving as the foundation of agent frameworks, are typically integrated into LLMOps or utilized through IDEs (Microsoft, [2023d](https://arxiv.org/html/2404.04902v1#bib.bib22); AutoGPT, [2023](https://arxiv.org/html/2404.04902v1#bib.bib1); LangChain, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib14); Wu et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib32); Li et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib17); Chen et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib8); Baidubce, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib3); Nakajima, [2023](https://arxiv.org/html/2404.04902v1#bib.bib25); Microsoft, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib20); Hong et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib12)).

![Refer to caption](img/781258f598c811d77131829ac8ed97b6.png)

Figure 1: The comparison between AI2Apps and existing works on building LLM-based AI agent application is outlined, with the integrity of development tools represented on the vertical axis and the visuality of components indicated on the horizontal axis.

In response to the aforementioned shortcomings, we introduce AI2Apps, a Visual Integrated Development Environment (Visual IDE) with full-cycle capabilities that empowers developers to efficiently build deployable LLM-based AI agent Applications. To the best of our knowledge, AI2Apps is the first LLM-based AI agent application development environment that achieves the engineering-level integrity and the full-stack visuality of a Visual IDE, as shown in Figure [1](https://arxiv.org/html/2404.04902v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications"). Its advantages are reflected in:

1.  1.

    Integrity. AI2Apps achieves engineering-level integrity by offering a seamlessly integrated development toolkit within a web-based Graphical User Interface (GUI), featuring a range of tools from a prototyping canvas and AI-assisted code editor to an agent debugger, management system, and deployment tool. In design mode, developers can quickly design agents by dragging and dropping components in the prototyping canvas. In code mode, AI2Apps features the AI-assisted code editor designed to help developers write agent application code faster and more consistently. The management system automatically maintains two-way synchronization between the prototyping canvas and the code editor, greatly improving programming efficiency. Then the agent debugger offers topology-based debugging features, including the breakpoint, step run, trace, and GPT mimic. These features allow developers to quickly pinpoint issues and optimize agent performance. Completed AI agents can be packaged as deployable web/mobile applications by deployment tool with one click. They can also be integrated as AI extensions into existing websites/applications with just a few lines of code.

2.  2.

    Visuality. AI2Apps achieves full-stack visuality by representing multi-dimensional reusable front-end and back-end code as drag-and-drop visual components. AI2Apps provides reusable components oriented toward three dimensions: user interaction, chain, and flow control. The user interaction represents the front-end GUI widgets that facilitate user-agent interaction modes. Relying solely on chat is often not efficient enough to engage with users. Therefore, AI2Apps offers over 50 GUI widgets to support agent development, including menus, buttons, charts, and more. These widgets enable agents to interact with users in a manner akin to real-world applications. The chain represents the backend sequential flow that includes LLMs, prompts, code, agents, and other tools as components. The flow control enables developers to express the logic of applications, incorporating components such as connectors, branches, array loops, summaries, and error handlers. It offers a visual representation of what is traditionally expressed as textual program code.

3.  3.

    Extensibility. AI2Apps Extension (AAE) is a plugin extension system specifically designed for AI2Apps. AAE offers developers extensive opportunities to enhance applications by leveraging open technologies as plugins and draggable components. In our screencast video⁴⁴4[https://youtu.be/tQTqxk1LzzU](https://youtu.be/tQTqxk1LzzU) we showcase how a new plugin with 20 components enables web agent applications to mimic human-like browsing behavior.

We conduct a case study and find substantial efficiency improvements. With the help of the agent debugger, AI2Apps can reduce token consumption and API calls when debugging a story writing multimodal agent application by approximately 90% and 80%, respectively.

Our Contributions. (1) We design a Visual IDE with full-cycle capabilities that accelerates developers to build deployable LLM-based AI agent applications. (2) We implement a plugin extension system for this Visual IDE, offering developers extensive opportunities to enhance applications by freely leveraging open technologies.

![Refer to caption](img/65afb33c5a49242b72b1f1ca367409e6.png)

Figure 2: Architecture of AI2Apps. The left and right sides display screenshots. (a) Prototyping Canvas utilizes built-in components for designing topology diagrams. (b) Code Editor utilizes AI assistance to continue programming the code generated in real-time by the Prototyping Canvas. (c) Agent Debugger pinpoints issues and optimizes agent performance. (d) Deployment Tool releases deployable apps. (e) Plugin Extension System introduces new components. (f) Management System supports the operating environment and resource scheduling.

## 2 Preliminary

### 2.1 LLM-based AI Agent

AI agents are artificial entities that are capable of perceiving their surroundings using sensors, making decisions, and then taking actions in response using actuators. LLM-based AI Agent means employing LLMs as the primary component of brain or controller of these agents and expanding their perceptual and action space through strategies such as multimodal perception and tool utilization (Xi et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib33); Nakano et al., [2021](https://arxiv.org/html/2404.04902v1#bib.bib26); Yao et al., [2022](https://arxiv.org/html/2404.04902v1#bib.bib35); Schick et al., [2024](https://arxiv.org/html/2404.04902v1#bib.bib29); Wei et al., [2022](https://arxiv.org/html/2404.04902v1#bib.bib31); Kojima et al., [2022](https://arxiv.org/html/2404.04902v1#bib.bib13); Wang et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib30)). They have been applied to various real-world scenarios, such as software development (Li et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib17); Qian et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib28); Hong et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib12)) and scientific research (Boiko et al., [2023a](https://arxiv.org/html/2404.04902v1#bib.bib4); Bran et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib6); Boiko et al., [2023b](https://arxiv.org/html/2404.04902v1#bib.bib5)).

### 2.2 Visual IDE

IDEs are software suites that enhance software development through features like source-code editors, build automation tools, and debuggers. The VScode (Microsoft, [2023f](https://arxiv.org/html/2404.04902v1#bib.bib24)) mentioned in Figure [1](https://arxiv.org/html/2404.04902v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications") is indeed a widely used IDE software.

Visual IDEs surpass traditional text-based IDEs by enhancing efficiency, improving code understanding, and facilitating collaboration through user-friendly GUIs, rapid prototyping, and integrated development resources. The other two concepts mentioned earlier that are easily confused are: SDKs often manifest as Application Programming Interfaces (APIs) or software frameworks and comprise on-device libraries with reusable functions for interfacing with specific programming languages. LLMOps platforms are specialized tools designed to streamline the deployment, management, and scaling of LLMs. These platforms are crucial for businesses and developers that LLMs for applications such as chatbots, content generation, data analysis, and more.

## 3 The AI2Apps Framework

AI2Apps is designed for both the integrity of its development tools and the visuality of its components, ensuring a smooth and efficient building experience. It can be organized into six blocks, as shown in Figure [2](https://arxiv.org/html/2404.04902v1#S1.F2 "Figure 2 ‣ 1 Introduction ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications").

### 3.1 Prototyping Canvas

Prototype design plays a pivotal role in application development by creating interactive models to validate functionality and user experience, ultimately reducing costs and ensuring the delivery of a successful application that meets user needs. In design mode, developers can quickly design agent logic by dragging and dropping components in the prototyping canvas. Its features include:

Topology-based. The prototyping canvas represents application logic in the form of a topology diagram, thereby breaking down all coupled code into clear and reliable units. Compared to reading complex code line by line, the advantages of AI agent code containing topology diagrams in later maintenance are enormous. It not only provides a clear understanding of the original code’s design rationale but also allows for quicker identification of code issues.

Visual Components. The prototyping canvas visualizes components oriented toward three dimensions: user interaction, chain, and flow control. The user interaction represents the front-end GUI widgets that facilitate user-agent interaction modes. Relying solely on chat is often not efficient enough to engage with users. Therefore, AI2Apps offers over 50 GUI widgets to support agent development, including menus, buttons, charts, and more. These widgets enable agents to interact with users in a manner akin to real-world applications. The chain represents the backend sequential flow that includes LLMs, prompts, code, agents, and other tools as visual components. The flow control enables developers to express the logic of applications, incorporating components such as connectors, branches, array loops, summaries, and error handlers. As a result, developers can easily add and configure them without having to manually write extensive code. This not only enhances development efficiency, but also reduces the likelihood of errors.

Code Sync. Traditional development approaches struggle to maintain real-time synchronization between the initial design and code implementation, rendering design documents less effective as references during maintenance phases. AI2Apps’ innovative “Design and Coding at Same Time” mode guarantees constant alignment between design and implementation, effectively eliminating any discrepancies between the envisioned design and the actual code.

 Prototyping Canvas Code Editor Agent Debugger Deployment Tool Name Topology Components Code Sync AI Copilot Multi-lang Breakpoint Step Run Trace Code App Visual IDE: AI2Apps (Ours)${}^{*}$ ✓ ✓ Two-way ✓ ✓ ✓ ✓ ✓ ✓ Open IDE: Prompt flow w/ VScode (Microsoft, [2023c](https://arxiv.org/html/2404.04902v1#bib.bib21)) ✓ ✗ Two-way ✗ ✗ ✓ ✓ ✓ ✓ Open Semantic Kernel w/ VScode (Microsoft, [2023e](https://arxiv.org/html/2404.04902v1#bib.bib23)) ✗ ✗ ✗ ✗ ✓ ✓ ✓ ✗ ✓ Open LLMOps Platform: GPTs (Openai, [2023](https://arxiv.org/html/2404.04902v1#bib.bib27)) ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✗ Closed LangSmith (LangChain, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib15)) ✗ ✓ ✗ ✗ ✓ ✓ ✓ ✓ ✓ Closed Autogen Studio (Microsoft, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib19)) ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✓ Closed Appbuilder (Baidubce, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib2)) ✗ ✓ ✗ ✗ ✓ ✗ ✗ ✗ ✓ Closed Coze (ByteDance, [2023](https://arxiv.org/html/2404.04902v1#bib.bib7)) ✓ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✗ Closed AgentScope${}^{*}$  (Gao et al., [2024](https://arxiv.org/html/2404.04902v1#bib.bib11)) ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✓ Closed OpenAgent${}^{*}$  (Xie et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib34)) ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✗ Closed Dify${}^{*}$  (LangGenius, [2023](https://arxiv.org/html/2404.04902v1#bib.bib16)) ✗ ✓ One-way ✗ ✓ ✗ ✗ ✗ ✓ Closed Langflow${}^{*}$  (Logspace, [2023](https://arxiv.org/html/2404.04902v1#bib.bib18)) ✓ ✓ One-way ✗ ✗ ✗ ✗ ✗ ✓ Closed Flowise${}^{*}$  (FlowiseAI, [2023](https://arxiv.org/html/2404.04902v1#bib.bib10)) ✓ ✓ One-way ✗ ✗ ✗ ✗ ✗ ✓ Closed Bisheng${}^{*}$  (Dataelement, [2023](https://arxiv.org/html/2404.04902v1#bib.bib9)) ✓ ✓ One-way ✗ ✗ ✗ ✗ ✗ ✓ Closed SDK: Prompt flow${}^{*}$  (Microsoft, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib20)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ Semantic Kernel${}^{*}$  (Microsoft, [2023d](https://arxiv.org/html/2404.04902v1#bib.bib22)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ AutoGPT${}^{*}$  (AutoGPT, [2023](https://arxiv.org/html/2404.04902v1#bib.bib1)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ BabyAGI${}^{*}$  (Nakajima, [2023](https://arxiv.org/html/2404.04902v1#bib.bib25)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ LangChain${}^{*}$  (LangChain, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib14)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ Autogen${}^{*}$  (Wu et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib32)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ MetaGPT${}^{*}$  (Hong et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib12)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ AppBuilder-SDK${}^{*}$  (Baidubce, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib3)) ✗ ✓ ✗ ✗ ✓ ✗ ✗ ✗ ✓ ✗ Camel${}^{*}$  (Li et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib17)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ Closed AgentVerse${}^{*}$  (Chen et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib8)) ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ Closed 

Table 1: Comparison between AI2Apps and current existing works via the integrity of development tools. The “${}^{*}$” indicates that the project has been open-sourced. The “Two-way” means that the two-way synchronization in real-time between the prototyping canvas and the code editor. The “One-way” means only one-way synchronization. The “Open” indicates that the generated application belongs to the user’s personal copyright and can be freely used. The “Closed” indicates that the generated application can only be deployed within the platform itself or encapsulated as an API. All statistics in the table are collected by March 2024.

 | Name | User Interaction | Chain | Flow Control |
| Visual IDE: |  |  |  |
| AI2Apps (Ours)${}^{*}$ | ✓ | ✓ | ✓ |
| IDE: |  |  |  |
| PromptFlow w/ VScode (Microsoft, [2023c](https://arxiv.org/html/2404.04902v1#bib.bib21)) | ✗ | ✓ | ✗ |
| Semantic Kernel w/ VScode (Microsoft, [2023e](https://arxiv.org/html/2404.04902v1#bib.bib23)) | ✗ | ✗ | ✗ |
| LLMOps Platform: |  |  |  |
| GPTs (Openai, [2023](https://arxiv.org/html/2404.04902v1#bib.bib27)) | ✗ | Limited | ✗ |
| LangSmith (LangChain, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib15)) | ✗ | Limited | ✗ |
| Autogen Studio (Microsoft, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib19)) | ✗ | Limited | ✗ |
| Appbuilder (Baidubce, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib2)) | ✗ | Limited | ✗ |
| Coze (ByteDance, [2023](https://arxiv.org/html/2404.04902v1#bib.bib7)) | ✗ | ✓ | ✗ |
| AgentScope${}^{*}$ (Gao et al., [2024](https://arxiv.org/html/2404.04902v1#bib.bib11)) | ✗ | Limited | ✗ |
| OpenAgent${}^{*}$ (Xie et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib34)) | ✗ | Limited | ✗ |
| Dify${}^{*}$ (LangGenius, [2023](https://arxiv.org/html/2404.04902v1#bib.bib16)) | ✗ | ✓ | ✗ |
| Langflow${}^{*}$ (Logspace, [2023](https://arxiv.org/html/2404.04902v1#bib.bib18)) | ✗ | ✓ | ✗ |
| Flowise${}^{*}$ (FlowiseAI, [2023](https://arxiv.org/html/2404.04902v1#bib.bib10)) | ✗ | ✓ | ✗ |
| Bisheng${}^{*}$ (Dataelement, [2023](https://arxiv.org/html/2404.04902v1#bib.bib9)) | ✗ | ✓ | ✗ |
| SDK: |  |  |  |
| Prompt flow${}^{*}$ (Microsoft, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib20)) | ✗ | ✗ | ✗ |
| Semantic Kernel${}^{*}$ (Microsoft, [2023d](https://arxiv.org/html/2404.04902v1#bib.bib22)) | ✗ | ✗ | ✗ |
| AutoGPT${}^{*}$ (AutoGPT, [2023](https://arxiv.org/html/2404.04902v1#bib.bib1)) | ✗ | ✗ | ✗ |
| BabyAGI${}^{*}$ (Nakajima, [2023](https://arxiv.org/html/2404.04902v1#bib.bib25)) | ✗ | ✗ | ✗ |
| LangChain${}^{*}$ (LangChain, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib14)) | ✗ | ✗ | ✗ |
| Autogen${}^{*}$ (Wu et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib32)) | ✗ | ✗ | ✗ |
| MetaGPT${}^{*}$ (Hong et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib12)) | ✗ | ✗ | ✗ |
| AppBuilder-SDK${}^{*}$ (Baidubce, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib3)) | ✗ | ✗ | ✗ |
| Camel${}^{*}$ (Li et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib17)) | ✗ | ✗ | ✗ |
| AgentVerse${}^{*}$ (Chen et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib8)) | ✗ | ✗ | ✗ | 

Table 2: Comparison between AI2Apps and current existing works via the visuality of components. The “${}^{*}$” indicates that the project has been open-sourced. The “Limited” means that the chain cannot be visualized in an intuitive topology diagram. All statistics in the table are collected by March 2024.

### 3.2 Code Editor

A code editor is a software tool used by developers to write and edit code for software development projects. In code mode, AI2Apps features the AI-assisted code editor designed to help developers write high-quality application code. Its features include:

AI Copilot. Our code editor comes with an AI copilot that assists users in generating subsequent code at the cursor position based on context or rewriting the entire document directly. We allow users to freely modify the topology of the AI copilot to enhance its functionality, or users can integrate an external AI copilot through APIs.

AI UI Creator. The AI UI creator can generate standardized UI code that users need through a conversational interface, utilizing over 50 GUI widgets, and display it in the prototyping canvas.

Multi-language. The code editor supports programming languages in any form, including JavaScript, Python, and others, enabling developers from any programming background to freely integrate their own code.

### 3.3 Agent Debugger

The design concept of the agent debugger draws inspiration from the debugging functionality found in general-purpose IDEs. Unlike traditional IDEs, which focus on debugging code line by line, the agent debugger is specifically tailored to follow the trajectory of topology diagrams. Its features include:

Breakpoint. Setting breakpoints allows pausing execution at specific locations within the topology diagram. This enables developers to inspect the program’s state at particular time points, including variable values, memory status, and the program’s execution path. Setting breakpoints is an effective troubleshooting method, especially when dealing with complex errors and performance issues.

Step Run. The step run feature allows developers to execute the topology diagram node by node or to jump into a subgraph within a specific agent, enabling a more detailed examination of the execution flow and the state at various time points. This method is particularly useful for understanding the execution path of the topology diagram, identifying logic errors, and inspecting variable changes.

Trace. The trace feature visually represents the flow trajectory of data variables on the topology diagram. Users can save and download trace logs in JSON format for subsequent analysis of agents.

GPT Mimic. During the phase of invoking external LLM APIs, users can set up GPT mimic. When the predefined conditions are met, the API call will no longer go through the network but will directly return the results set by the GPT mimic. This approach can significantly reduce token costs and improve debugging efficiency.

### 3.4 Deployment Tool

Most existing mainstream development platforms develop applications that heavily rely on the platform’s own runtime environment, hindering the development of AI agent applications. However, AI2Apps, as a Visual IDE, integrates multiple deployment tools and supports users in directly building externally deployable applications. Completed AI agents can be packaged as standalone web/mobile apps. They can also be integrated as AI extensions into existing websites/apps with just a few lines of code.

### 3.5 Plugin Extension System

AI2Apps Extension (AAE) is a plugin extension system specifically designed for AI2Apps. AAE offers developers extensive opportunities to enhance applications by leveraging open technologies as plugins and draggable components. AAE features reusable components that allow developers to combine existing ones or create new ones. Developers can share their custom components by publishing them as packages, thereby broadening AI2Apps’ functionality through the AAE system.

In our screencast video⁵⁵5[https://youtu.be/tQTqxk1LzzU](https://youtu.be/tQTqxk1LzzU) we showcase how a new plugin with 20 components enables web agent applications to mimic human-like browsing behavior. Through web extension components, AI2Apps is equipped with comprehensive control over web pages, enabling actions such as opening/switching pages, reading page content, filling/modifying page content, simulating user behavior, and more.

### 3.6 Management System

Management System provides various underlying functional support for AI2Apps, including:

Web-OS System. Based on web API technology, it provides a complete set of desktop operating system functions for AI2Apps. Examples include file system, multi-task support, network call. It offers users a development experience that is comparable to native operating systems, yet more convenient and secure.

Runtime Manger. It provides specialized runtime foundational functions for AI-oriented applications, including: application management/scheduling, plugin extension/control, integration of AI functionality components, etc.

Package Manager. It extends the capabilities of AI2Apps through packages, which include services for building, publishing, sharing packages, as well as installation and upgrades.

## 4 Case Study and Usage

![Refer to caption](img/ec5c11fd04649c7f46ec32a0fa4557d5.png)

Figure 3: Screenshot of our usage assistant built by AI2Apps.

### 4.1 Case Study

We conduct a case study and find substantial efficiency improvements. In this case, we initially utilized AI2Apps to build a complex Agent application for writing short stories. It allows specifying drawing styles, precise unlimited-length writing with the ability to include covers and illustrations, and possesses powerful editing capabilities for articles. It supports paragraph-by-paragraph modifications, including content and images, and provides the option to choose whether AI assistance is used during editing. We conducted four sets of controlled experiments by clearing all prompts from the agent and having eight volunteers attempt to restore the functionality of the original agent, with half using the agent debugger and the other half without it. With the help of the agent debugger, AI2Apps reduced token consumption and API calls when debugging a story writing multimodal agent application by approximately 90% and 80%, respectively. The agent debugger user interface is shown in Figure [3](https://arxiv.org/html/2404.04902v1#S4.F3 "Figure 3 ‣ 4 Case Study and Usage ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications"). This agent project can be accessed at [https://github.com/Avdpro/StoryWriter](https://github.com/Avdpro/StoryWriter)

### 4.2 Usage

Given the extensive functionality of AI2Apps, this paper is unable to offer detailed usage guidelines within its confined scope. For more information about our usage example, please refer to [https://github.com/Avdpro/ai2apps](https://github.com/Avdpro/ai2apps). We’ve built a usage assistant using AI2Apps, as shown in Figure [3](https://arxiv.org/html/2404.04902v1#S4.F3 "Figure 3 ‣ 4 Case Study and Usage ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications"), allowing users to get started with AI2Apps by simply running it at [https://www.ai2apps.com](https://www.ai2apps.com).

## 5 Comparison with Related Works

In Figure [1](https://arxiv.org/html/2404.04902v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications"), Table [1](https://arxiv.org/html/2404.04902v1#S3.T1 "Table 1 ‣ 3.1 Prototyping Canvas ‣ 3 The AI2Apps Framework ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications") and Table [2](https://arxiv.org/html/2404.04902v1#S3.T2 "Table 2 ‣ 3.1 Prototyping Canvas ‣ 3 The AI2Apps Framework ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications"), we present a detailed comparison of existing works through the perspectives of the integrity of development tools and the visuality of components. To the best of our knowledge, AI2Apps is the first LLM-based AI agent application development environment that achieves the engineering-level integrity and the full-stack visuality of a Visual IDE. Visual IDEs can integrate SDKs like traditional IDEs, and they can also be integrated into LLMOps platforms. Therefore, the open-source release of AI2Apps effectively fills the current gap in development technology, facilitating the advancement of AI agent applications.

## 6 Conclusion

We introduced AI2Apps, the first Visual IDE with full-cycle capabilities that empowers developers to efficiently build deployable LLM-based AI agent Applications. AI2Apps offers the engineering-level development tools and the full-stack visual components. It features a web-based interface with tools such as a prototyping canvas, an AI-powered code editor, a agent debugger, a management system, and deployment tools, alongside intuitive drag-and-drop components for code visualization. Additionally, we implemented a plugin extension system for this Visual IDE, offering developers extensive opportunities to enhance applications by freely leveraging open technologies. We conducted a case study and find substantial efficiency improvements. With the help of the agent debugger, AI2Apps can reduce token consumption and API calls when debugging a story writing multimodal agent application by approximately 90% and 80%, respectively.

## Limitations

As a Visual IDE, AI2Apps still cannot fully match the flexibility of traditional IDEs in app development, nor does it offer the operational capabilities of LLMOps platforms for deployed applications. However, its development convenience is noteworthy. we warmly welcome LLMOps platforms to integrate this work to enhance their offerings, thus jointly promoting related academic research.

## Ethics Statement

(1) This material is the authors’ own original work, which at this stage of project development has not been previously published elsewhere. (2) The paper is not currently being considered for publication elsewhere. (3) The paper reflects the authors’ own research and analysis in a truthful and complete manner. (4) Our work does not contain identity characteristics. It does not harm anyone. The eight participants in the case study part are volunteers recruited from students majoring in engineering. Before the case study experiments, all participants are provided with detailed guidance in both written and oral form. The only recorded user-related information is usernames, which are anonymized and used as identifiers to mark different participants. (5) AI2Apps is designed to help users to build deployable AI agent applications. (6) Our work does not involve LLM training or fine-tuning; we only use publicly available APIs permitted for research purposes. Therefore, there are no data-related risks associated with our approach.

## References

*   AutoGPT (2023) AutoGPT. 2023. Autogpt. [https://github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT).
*   Baidubce (2023a) Baidubce. 2023a. Appbuilder. [https://cloud.baidu.com/product/AppBuilder](https://cloud.baidu.com/product/AppBuilder).
*   Baidubce (2023b) Baidubce. 2023b. Appbuilder-sdk. [https://github.com/baidubce/app-builder](https://github.com/baidubce/app-builder).
*   Boiko et al. (2023a) Daniil A Boiko, Robert MacKnight, and Gabe Gomes. 2023a. Emergent autonomous scientific research capabilities of large language models. *arXiv preprint arXiv:2304.05332*.
*   Boiko et al. (2023b) Daniil A Boiko, Robert MacKnight, Ben Kline, and Gabe Gomes. 2023b. Autonomous chemical research with large language models. *Nature*, 624(7992):570–578.
*   Bran et al. (2023) Andres M Bran, Sam Cox, Oliver Schilter, Carlo Baldassari, Andrew White, and Philippe Schwaller. 2023. Augmenting large language models with chemistry tools. In *NeurIPS 2023 AI for Science Workshop*.
*   ByteDance (2023) ByteDance. 2023. Coze: Next-gen ai chatbot developing platform. [https://www.coze.com/](https://www.coze.com/).
*   Chen et al. (2023) Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chen Qian, Chi-Min Chan, Yujia Qin, Yaxi Lu, Ruobing Xie, et al. 2023. Agentverse: Facilitating multi-agent collaboration and exploring emergent behaviors in agents. *arXiv preprint arXiv:2308.10848*.
*   Dataelement (2023) Dataelement. 2023. Bisheng. [https://github.com/dataelement/bisheng](https://github.com/dataelement/bisheng).
*   FlowiseAI (2023) FlowiseAI. 2023. Flowise. [https://github.com/FlowiseAI/Flowise](https://github.com/FlowiseAI/Flowise).
*   Gao et al. (2024) Dawei Gao, Zitao Li, Weirui Kuang, Xuchen Pan, Daoyuan Chen, Zhijian Ma, Bingchen Qian, Liuyi Yao, Lin Zhu, Chen Cheng, et al. 2024. Agentscope: A flexible yet robust multi-agent platform. *arXiv preprint arXiv:2402.14034*.
*   Hong et al. (2023) Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, et al. 2023. Metagpt: Meta programming for multi-agent collaborative framework. In *The Twelfth International Conference on Learning Representations*.
*   Kojima et al. (2022) Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. 2022. Large language models are zero-shot reasoners. *Advances in neural information processing systems*, 35:22199–22213.
*   LangChain (2023a) LangChain. 2023a. Langchain. [https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain).
*   LangChain (2023b) LangChain. 2023b. Langsmith. [https://www.langchain.com/langsmith](https://www.langchain.com/langsmith).
*   LangGenius (2023) LangGenius. 2023. Dify. [https://github.com/langgenius/dify](https://github.com/langgenius/dify).
*   Li et al. (2023) Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. 2023. Camel: Communicative agents for "mind" exploration of large language model society. In *Thirty-seventh Conference on Neural Information Processing Systems*.
*   Logspace (2023) Logspace. 2023. Langflow. [https://github.com/logspace-ai/langflow](https://github.com/logspace-ai/langflow).
*   Microsoft (2023a) Microsoft. 2023a. Autogen studio 2.0: Revolutionizing ai agents. [https://autogen-studio.com/](https://autogen-studio.com/).
*   Microsoft (2023b) Microsoft. 2023b. Prompt flow. [https://github.com/microsoft/promptflow](https://github.com/microsoft/promptflow).
*   Microsoft (2023c) Microsoft. 2023c. Prompt flow for vscode. [https://marketplace.visualstudio.com/items?itemName=prompt-flow.prompt-flow](https://marketplace.visualstudio.com/items?itemName=prompt-flow.prompt-flow).
*   Microsoft (2023d) Microsoft. 2023d. Semantic kernel. [https://github.com/microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel).
*   Microsoft (2023e) Microsoft. 2023e. Semantic kernel for vscode. [https://learn.microsoft.com/en-us/semantic-kernel/vs-code-tools/](https://learn.microsoft.com/en-us/semantic-kernel/vs-code-tools/).
*   Microsoft (2023f) Microsoft. 2023f. Visual studio code - open source. [https://github.com/microsoft/vscode](https://github.com/microsoft/vscode).
*   Nakajima (2023) Yohei Nakajima. 2023. Babyagi. [https://github.com/yoheinakajima/babyagi](https://github.com/yoheinakajima/babyagi).
*   Nakano et al. (2021) Reiichiro Nakano, Jacob Hilton, Suchir Balaji, Jeff Wu, Long Ouyang, Christina Kim, Christopher Hesse, Shantanu Jain, Vineet Kosaraju, William Saunders, et al. 2021. Webgpt: Browser-assisted question-answering with human feedback. *arXiv preprint arXiv:2112.09332*.
*   Openai (2023) Openai. 2023. Explore gpts. [https://chat.openai.com/gpts](https://chat.openai.com/gpts).
*   Qian et al. (2023) Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, and Maosong Sun. 2023. Communicative agents for software development. *arXiv preprint arXiv:2307.07924*.
*   Schick et al. (2024) Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, and Thomas Scialom. 2024. Toolformer: Language models can teach themselves to use tools. *Advances in Neural Information Processing Systems*, 36.
*   Wang et al. (2023) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. 2023. A survey on large language model based autonomous agents. *arXiv preprint arXiv:2308.11432*.
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting elicits reasoning in large language models. *Advances in neural information processing systems*, 35:24824–24837.
*   Wu et al. (2023) Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. 2023. Autogen: Enabling next-gen llm applications via multi-agent conversation framework. *arXiv preprint arXiv:2308.08155*.
*   Xi et al. (2023) Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, et al. 2023. The rise and potential of large language model based agents: A survey. *arXiv preprint arXiv:2309.07864*.
*   Xie et al. (2023) Tianbao Xie, Fan Zhou, Zhoujun Cheng, Peng Shi, Luoxuan Weng, Yitao Liu, Toh Jing Hua, Junning Zhao, Qian Liu, Che Liu, et al. 2023. Openagents: An open platform for language agents in the wild. *arXiv preprint arXiv:2310.10634*.
*   Yao et al. (2022) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, and Yuan Cao. 2022. React: Synergizing reasoning and acting in language models. In *The Eleventh International Conference on Learning Representations*.