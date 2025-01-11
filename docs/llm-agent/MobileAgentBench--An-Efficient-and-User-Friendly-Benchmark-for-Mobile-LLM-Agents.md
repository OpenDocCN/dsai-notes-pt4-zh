<!--yml
category: 未分类
date: 2025-01-11 12:33:52
-->

# MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents

> 来源：[https://arxiv.org/html/2406.08184/](https://arxiv.org/html/2406.08184/)

Luyuan Wang
Carnegie Mellon University &Yongyu Deng
University of Michigan &Yiwei Zha
Northeastern University &Guodong Mao
Northeastern University &Qinmin Wang
Carnegie Mellon University
&Tianchen Min
Northeastern University &Wei Chen
Carnegie Mellon University &Shoufa Chen
The University of Hong Kong

###### Abstract

Large language model (LLM)-based mobile agents are increasingly popular due to their capability to interact directly with mobile phone Graphic User Interfaces (GUIs) and their potential to autonomously manage daily tasks. Despite their promising prospects in both academic and industrial sectors, little research has focused on benchmarking the performance of existing mobile agents, due to the inexhaustible states of apps and the vague definition of feasible action sequences. To address this challenge, we propose an efficient and user-friendly benchmark, MobileAgentBench, designed to alleviate the burden of extensive manual testing. We initially define 100 tasks across 10 open-source apps, categorized by multiple levels of difficulty. Subsequently, we evaluate several existing mobile agents, including AppAgent and MobileAgent, to thoroughly and systematically compare their performance. All materials are accessible on our project webpage: [https://MobileAgentBench.github.io](https://MobileAgentBench.github.io), contributing to the advancement of both academic and industrial fields.

## 1 Introduction

With the emergence of large language models (LLMs) [[1](https://arxiv.org/html/2406.08184v1#bib.bib1)], researchers have developed various autonomous agents across fields such as robotics [[4](https://arxiv.org/html/2406.08184v1#bib.bib4), [20](https://arxiv.org/html/2406.08184v1#bib.bib20)], games [[22](https://arxiv.org/html/2406.08184v1#bib.bib22), [8](https://arxiv.org/html/2406.08184v1#bib.bib8)], and mobile phones [[26](https://arxiv.org/html/2406.08184v1#bib.bib26), [19](https://arxiv.org/html/2406.08184v1#bib.bib19)]. Among these, mobile agents have attracted significant attention due to their potential to enhance user experiences and provide intelligent assistance on-the-go.

People have been dreaming of Intelligent Personal Assistants (IPAs) [[6](https://arxiv.org/html/2406.08184v1#bib.bib6)] that can fully automate daily tasks for decades. Since Apple introduced its digital assistant, Siri [[3](https://arxiv.org/html/2406.08184v1#bib.bib3)] in 2011, almost all the leading technology companies have launched their own IPAs, including Microsoft Cortana [[15](https://arxiv.org/html/2406.08184v1#bib.bib15)], Amazon Alexa [[2](https://arxiv.org/html/2406.08184v1#bib.bib2)], and Google Assistant [[9](https://arxiv.org/html/2406.08184v1#bib.bib9)]. While these digital assistants provide a hands-free human-computer interaction experience using Natural Language Interface (NLI), they can only fulfill relatively simple tasks, such as setting an alarm clock or sending a text message [[13](https://arxiv.org/html/2406.08184v1#bib.bib13)]. For third-party apps, developers have to follow and implement the application programming interfaces and protocols, such that when a user issues a very specific command, the system can invoke the corresponding functionality. This limits the usability of those digital assistants.

Table 1: Comparison between the proposed and existing benchmarks

| Benchmark | Fully Autonomous | Realistic Environment | Success Condition Flexibility | Low Code Invasiveness |
| --- | --- | --- | --- | --- |
| AppAgent [[26](https://arxiv.org/html/2406.08184v1#bib.bib26)] | ✗ | ✓ | ✓ | ✓ |
| AITW [[19](https://arxiv.org/html/2406.08184v1#bib.bib19)] | ✓ | ✗ | ✗ | ✗ |
| AndroidArena [[24](https://arxiv.org/html/2406.08184v1#bib.bib24)] | ✓ | ✓ | ✗ | ✗ |
| MobileAgentBench (ours) | ✓ | ✓ | ✓ | ✓ |

LLMs contribute significantly to resolving the persistent challenge of understanding user intent. The demonstrated reasoning ability [[17](https://arxiv.org/html/2406.08184v1#bib.bib17)] highlights the potential of LLM-based autonomous agents as next-generation Intelligent Personal Assistants (IPAs), which are not limited by the programming interfaces since they directly operate on the Graphic User Interface (GUI) [[21](https://arxiv.org/html/2406.08184v1#bib.bib21), [26](https://arxiv.org/html/2406.08184v1#bib.bib26)]. The GUI can either be represented by a text-based view tree to be consumed by a LLM or a screenshot image that can leverage a Multi-modal LLM (MLLM) [[27](https://arxiv.org/html/2406.08184v1#bib.bib27)]. The action space of the agents composes a series of functions to simulate human operations, such as click, type, swipe, etc. In this way, LLM agents can theoretically achieve whatever human users can do, without any modification of the existing apps.

The promising future of LLM-based smartphone agents attracts more and more researchers to study this topic. However, the scope of benchmarks available for evaluating the performance of these agents remains constrained. Among the existing benchmarks, several prevalent issues are evident: 1\. Scalability and usability. Researchers need to fully understand complicated data structures and tools before extending the benchmark with customized tasks or integrating it into their own codebases [[28](https://arxiv.org/html/2406.08184v1#bib.bib28)]. 2\. Robustness and Flexibility. Only the annotated task completion path is considered [[19](https://arxiv.org/html/2406.08184v1#bib.bib19), [24](https://arxiv.org/html/2406.08184v1#bib.bib24)]. However, there might be multiple paths to successfully complete a task, which may break the task success judgment logic. 3\. Realistic environment. Some benchmarks evaluate the agent’s performance based on a collection of screenshots but not real devices. It fails if the agent performs an abnormal action and goes to an undefined state [[19](https://arxiv.org/html/2406.08184v1#bib.bib19)].

To address the issues described, we propose a robust benchmark, MobileAgentBench, designed to evaluate the capabilities of mobile LLM agents within the Android ecosystem. MobileAgentBench offers several advantages over previous benchmarks due to its ease of use and minimally invasive nature. Specifically, for standard agents, the integration process requires fewer than ten lines of additional code. The benchmark excels in usability and versatility, supporting a broad spectrum of testing tasks across various Android operating system versions and executing on actual devices.

In this initial release, we offer 100 built-in benchmarking tasks spanning ten open-source applications. Notably, MobileAgentBench diverges from traditional approaches by simplifying the extension process. Third-party developers can specify the conditions for task success using just a few lines of Python code, without needing extensive knowledge in Android development. This accessibility makes MobileAgentBench more conducive to developers and researchers from non-Android Development communities. Furthermore, we introduce an innovative method for determining the task-terminating state, rendering the benchmark resistant to the complexities of tracking multiple potential success pathways. This approach ensures that MobileAgentBench provides reliable and precise benchmarking outcomes.

The comparison between the proposed and existing benchmarks is listed in Tab. [1](https://arxiv.org/html/2406.08184v1#S1.T1 "Table 1 ‣ 1 Introduction ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"), where fully autonomous represents if the benchmark does not need human supervision or judgment, realistic environment represents the tasks can be run on real devices, success condition flexibility represents it takes all possible success paths into consideration, low code invasiveness represents integrating the benchmark into existing agents does not need significant code changes.

Our contributions are summarized as follows:

*   •

    We propose a benchmark framework for mobile LLM agents. The new approach addresses common issues of existing benchmarks, making the evaluation process fully autonomous and reliable.

*   •

    We implement and test 100 benchmarking tasks with different levels of difficulty. The benchmark is plug-and-play and easy for both developing new agents and evaluating existing agents.

*   •

    We evaluate the performance of state-of-the-art mobile LLM agents and perform a solid and systematic comparison with our new benchmark, providing baseline data to the community.

## 2 Related Work

### 2.1 Mobile LLM Agents

Studies before the LLM-era employed reinforcement learning (RL) to solve the autonomous GUI navigation problem [[10](https://arxiv.org/html/2406.08184v1#bib.bib10)]. The recent advancement of LLM and MLLM becomes the dominant agent paradigm becaust of the greater ability of UI understanding and reasoning. Early studies focused on web agents, which achieve task automation within browsers [[7](https://arxiv.org/html/2406.08184v1#bib.bib7), [29](https://arxiv.org/html/2406.08184v1#bib.bib29)]. Recently, more and more studies started to investigate agents on mobile devices, especially on the Android platform, as Android smartphones are the most widely used personal computing devices.

Mobile LLM agents share a similar algorithm. The full input prompt often consists of four main components: the user prompt (task description), the current UI view hierarchy (VH) description, the action function list, and historical information. Specifically, the action list mainly includes click, swipe, type, and other common UI operations. If MLLM is used, the current screenshot is also a part of the input. The LLM/MLLM is asked to think of the next action based on the current and historical states and call the correct function to perform the given task step by step. The agent finally parses the model response and sends control signals to the Android device using Android Debug Bridge (ADB) ¹¹1https://developer.android.com/tools/adb, UIAutomator ²²2https://developer.android.com/training/testing/other-components/ui-automator, or other higher-level UI automation frameworks.

Despite the similarity of the high-level ideas, researchers have developed different techniques to improve performance and efficiency. Among these works, AndroidArena [[24](https://arxiv.org/html/2406.08184v1#bib.bib24)] transforms the long and overwhelming view hierarchy XML into a compressed representation and assigned UI elements with unique node IDs, which shortens the prompt and makes the system more efficient. MobileAgent [[21](https://arxiv.org/html/2406.08184v1#bib.bib21)] observes that GPT-4V lacks the capability of UI element localization, and employs an Optical Character Recognition (OCR) model to locate and localize text views. Moreover, it uses the CLIP [[18](https://arxiv.org/html/2406.08184v1#bib.bib18)] and Grounding DINO [[14](https://arxiv.org/html/2406.08184v1#bib.bib14)] models to detect icons. AppAgent [[26](https://arxiv.org/html/2406.08184v1#bib.bib26)] uses SoM [[25](https://arxiv.org/html/2406.08184v1#bib.bib25)] prompts to localize UI elements and breaks tasks into two phases, exploration, and deployment. During the exploration phase, the agent automatically interacts with the apps and summarizes the observations into a document. In the deployment phase, it employes the Retrieval Augmented Generation (RAG) [[12](https://arxiv.org/html/2406.08184v1#bib.bib12)] technique to utilize the summarized knowledge and improve success rate. CogAgent [[11](https://arxiv.org/html/2406.08184v1#bib.bib11)] proposes its own highly efficient 18B-parameter MLLM, which can be loaded on a single commercial GPU. Furthermore, Octopus v2 [[5](https://arxiv.org/html/2406.08184v1#bib.bib5)] proposes a compact 3B-parameter model, which unlocks the potential to run mobile LLM agents on-device in an efficient and privacy-preserving manner.

### 2.2 Benchmarks for Mobile LLM Agents

Since the Mobile LLM agent is a newly emerging research field, the choice of benchmarks is very limited. Some studies rely on verifying the task execution status manually to evaluate the performance [[26](https://arxiv.org/html/2406.08184v1#bib.bib26)], which is tedious and time-consuming. To expedite the agent development, we need a fully autonomous benchmarking system to report various metrics, especially the task success rate. However, automatically judging if a task is completed successfully is non-trivial. The main challenge is caused by the dynamic nature of the GUI navigation task – the agent may perform random actions and drive the app to an unknown state.

AITW [[19](https://arxiv.org/html/2406.08184v1#bib.bib19)] is a popular benchmark for mobile LLM agents. It has a large scale, but it’s based on static screenshot images. Thinking of each app state (screenshot) as a node, and each action as an edge, we can build a State Transferring Graph (STG) based on the screenshots and the human-annotated actions. It fails immediately if the agent performs actions in a non-considered sequence and leads the STG to a non-existent node, even if the agent can eventually complete the task.

The only solution is to identify task successes on real devices. One approach is to match the agent’s actions with the annotated ground truth (GT). A step-wise matching algorithm is not accurate, because the agent may not finish the task exactly in the same order with GT. AndroidArena [[24](https://arxiv.org/html/2406.08184v1#bib.bib24)] proposes an adaptive method of calculating the longest common subsequence of the agent and GT action sequences, which is illustrated as follows, where $a$ and $\hat{a}$ are the GT and actual actions, respectively. The common subsequences are marked in red.

|  | $\displaystyle a$ | $\displaystyle={\color[rgb]{1,0,0}ABC}$ |  | (1) |
|  | $\displaystyle\hat{a}$ | $\displaystyle={\color[rgb]{1,0,0}A}XY{\color[rgb]{1,0,0}B}U{\color[rgb]{1,0,0}% C}VW$ |  | (2) |

AndroidArena [[24](https://arxiv.org/html/2406.08184v1#bib.bib24)] treats a task as successful if the GT is a subsequence of the actual action sequence. It addresses the issue of redundant actions but is still not optimal. A simple counter-example can be navigating from the page 1 to page 3 by clicking the next page button two times. If the agent performs the following sequence of actions: clicking the next page button, clicking the previous page button, and clicking the next page button, it doesn’t navigate to the correct page but is still a subsequence of the GT. This method gives false positive results if the redundant action has a side effect.

A concurrent work, LlamaTouch [[28](https://arxiv.org/html/2406.08184v1#bib.bib28)], addresses this problem by examining the final UI state, which is similar to our approach. We observe that despite the infinite feasible action sequences, the final success state convergence to one. The success or failure can be determined by checking the final UI state. An edge case is that some tasks may not have a direct UI representation, for example, the result of a network request triggered by a button may not be directly reflected on the current UI page. Thus, only checking the UI state is not sufficient and we need to incorporate actions, such as the clicking event, into consideration. LlamaTouch [[28](https://arxiv.org/html/2406.08184v1#bib.bib28)] matches the click action by mapping the coordinate to a UI element, based on the view bounding boxes. However, it may not always be accurate. The process of finding the correct view to respond to a clicking event is called a hit test, and it’s only accurate if performed by the Android UI system. This is because app developers can modify the touchable area, making it different from the view border to get better user experience.

![Refer to caption](img/29c6fba27fa19342cde43261e453c785.png)

Figure 1: Extended touchable area.

Fig. [1](https://arxiv.org/html/2406.08184v1#S2.F1 "Figure 1 ‣ 2.2 Benchmarks for Mobile LLM Agents ‣ 2 Related Work ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents") shows an example of enlarging the touchable area of a button view. In Fig. [1](https://arxiv.org/html/2406.08184v1#S2.F1 "Figure 1 ‣ 2.2 Benchmarks for Mobile LLM Agents ‣ 2 Related Work ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"), touching point 1, the button does not respond because it’s outside of the touchable area. Touching point 2, the button responds because it’s inside the button view. Touching point 3, although it’s outside of the visible button view, the button still responds because it’s inside the extended touchable area. To overcome this difficulty, we utilize the Android Accessibility Service ³³3https://developer.android.com/reference/android/accessibilityservice/AccessibilityService to capture app events faithfully and forward them to the benchmark server. The details of our implementation are described in Sec. [3.1](https://arxiv.org/html/2406.08184v1#S3.SS1 "3.1 Method ‣ 3 MobileAgentBench ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents").

## 3 MobileAgentBench

![Refer to caption](img/6fe07a20aa10b79506b6cc550177ad61.png)

Figure 2: Overview of the MobileAgentBench architecture.

### 3.1 Method

MobileAgentBench runs on real Android devices, supporting both physical devices and emulators. It sets up environment and then invokes the agent execution function. While the agent is operating the device, MobileAgentBench judges the task success status in real time without any side effects. After the agent stops or exceeds the maximum steps, MobileAgentBench automatically switches to the next task. The whole process is fully automated and requires no human supervision.

The task success judgment mechanism is implemented by matching the final UI state, instead of examining the action sequence. This is because there might be multiple paths towards task completion, but the final success state converges to one. For example, if the task is to go to the settings page, agents may mistakenly open random pages before they correctly find the desired settings page. Matching the action sequence is difficult because of the randomness. On the contrary, checking if the top page is the settings page is easy and reliable. No matter what operations the agent does, we treat it as a success as long as the settings page is detected. ADB and UIAutomator are used to fetch the VH information. For each task, there is a Python file that defines the task success criteria, making it easy to extend and customize tasks for third-party developers.

As some tasks may not have a direct reflection on the current UI page, only checking the VH information is not sufficient. An example can be editing a note and saving the changes. Clicking the save button, the app may only pop up a temporary alert to indicate the save action has succeeded and stays on the current page. When the benchmark checks the current view state, it doesn’t know if the save button is clicked or not. As a benchmark, it cannot go to other pages because it may change the app state and affect t he next action of the agent. Since we want to determine the task success in real-time to collect how many steps the agent takes, it is not feasible to check UI states of other pages after the agent stops. Besides, some agents have the problem of not being able to stop gracefully even the task is completed. We address this issue by incorporating app events, especially button click action signals. For the above-mentioned task, we can use the view hierarchy to check if the note is edited correctly, and then mark the task as a success if the save button clicking signal is received afterwards.

To faithfully receive the app event signals, we make an Android app using the Android Accessibility Service. Android Accessibility Service was originally designed to help people with disabilities. It runs in the background and invokes a callback function when the Android system fires an accessibility event. Such events include most UI state transitions by the user (agent) interactions, such as button clicking, window changing, etc, which fulfills the needs of the proposed benchmark.

The overview of MobileAgentBench is shown in Fig. [2](https://arxiv.org/html/2406.08184v1#S3.F2 "Figure 2 ‣ 3 MobileAgentBench ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"). The benchmarking apps run on real devices from a device farm, which can either be a physical device or an emulator. The device talks to the host machine of the benchmark and the agent via ADB. The benchmark and the agent use ADB to retrieve app state information, such as screenshots, view hierarchy, and send control signals. The benchmark invokes the agent with the current benchmarking task prompt and collects runtime information from the agent, such as the LLM input and output. Whenever the event listener app receives an app event, it forwards the event to the benchmark server via a socket, so the benchmark can assess the task success status using both VH and the actions. At the top of Fig. [2](https://arxiv.org/html/2406.08184v1#S3.F2 "Figure 2 ‣ 3 MobileAgentBench ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"), we show a sample task workflow, “Create a new task Laundry” with the Calendar app. The agent needs to perform 4 actions to complete the task: clicking the add button, clicking the task button, typing the task name “Laundry”, and clicking the save button. The benchmark checks the content of the task name input box view and listens to the save button clicking event to determine if the task is finished successfully.

### 3.2 Task Description

In our initial version of the benchmark, we implement 100 tasks over 10 daily apps. The 10 apps are from SimpleMobileTools ⁴⁴4https://simplemobiletools.com, GPL-3.0, an open-source project of Android apps. These apps have simple and straightforward user interfaces, without any advertisements or unnecessary permissions, and thus are great for benchmarking use cases. The full list of app names is shown in Fig. [3b](https://arxiv.org/html/2406.08184v1#S3.F3.sf2 "In Figure 3 ‣ 3.2 Task Description ‣ 3 MobileAgentBench ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents").

We carefully design the benchmarking tasks, so that they can simulate a normal user’s daily activity and have multi-level difficulties. The distribution of task difficulty levels is shown in Fig. [3](https://arxiv.org/html/2406.08184v1#S3.F3 "Figure 3 ‣ 3.2 Task Description ‣ 3 MobileAgentBench ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"). Difficulties are defined by the minimum steps to finish a task, which is cross-verified by 3 human experts independently. A task would be classified as an *easy* task if it can be done within 2 steps, *medium* if greater or equal to 3 while less than 6, and otherwise, *hard*.

![Refer to caption](img/c525e3d8d46491015b6c43042e9bb77b.png)

(a) Distribution over all tasks.

![Refer to caption](img/1bf8de444ca8baed06875d77fd76d17e.png)

(b) Distribution over each app.

Figure 3: The distribution of task difficulty levels.

### 3.3 Usage

The benchmark APIs are designed to be user-friendly and as less invasive as possible. For a standard agent, it takes less than 10 lines of additional code to integrate. List [1](https://arxiv.org/html/2406.08184v1#LST1 "Listing 1 ‣ 3.3 Usage ‣ 3 MobileAgentBench ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents") shows the pseudo-code of the benchmark usage. First, we need to import the benchmark Python library and initialize the benchmark orchestrator. Next, the main agent entrance function should be defined. It takes and only takes one parameter, the task prompt. The agent iteratively performs actions to finish the task. Before and after the agent performs one action, the orchestrator functions are called, so information such as the time spent and LLM output can be collected. The program starts with the orchestrator run function. It calls the agent entrance function with each task prompt and automatically switches to the next task after the task finishes. The task success status is judged on the fly.

[⬇](data:text/plain;base64,ZnJvbSBtb2JpbGVfYWdlbnRfYmVuY2htYXJrLnRhc2tfb3JjaGVzdHJhdG9yIGltcG9ydCBUYXNrT3JjaGVzdHJhdG9yCm9yY2hlc3RyYXRvciA9IFRhc2tPcmNoZXN0cmF0b3IoKSAjIHRoZSBNb2JpbGVBZ2VudEJlbmNoIG9yY2hlc3RyYXRvcgojIHRoZSBhZ2VudCBlbnRyYW5jZSBmdW5jdGlvbgpkZWYgYWdlbnRfcnVuKHRhc2tfcHJvbXB0KToKICAgIHdoaWxlIG5vdCBkb25lOgogICAgICAgIG9yY2hlc3RyYXRvci5iZWZvcmVfb25lX2FjdGlvbigpCiAgICAgICAgIyB0aGUgYWdlbnQgaW52b2tlcyBhIExMTSB0byB0aGluayBhYm91dCB0aGUgbmV4dCBhY3Rpb24KICAgICAgICBhY3Rpb24gPSBsbG1fdGhpbmsodGFza19wcm9tcHQsIHNjcmVlbnNob3QpCiAgICAgICAgYWdlbnRfcGVyZm9ybShhY3Rpb24pCiAgICAgICAgb3JjaGVzdHJhdG9yLmFmdGVyX29uZV9hY3Rpb24oYWN0aW9uKQpvcmNoZXN0cmF0b3IucnVuKGFnZW50X3J1bik=)1from  mobile_agent_benchmark.task_orchestrator  import  TaskOrchestrator2orchestrator  =  TaskOrchestrator()  #  the  MobileAgentBench  orchestrator3#  the  agent  entrance  function4def  agent_run(task_prompt):5  while  not  done:6  orchestrator.before_one_action()7  #  the  agent  invokes  a  LLM  to  think  about  the  next  action8  action  =  llm_think(task_prompt,  screenshot)9  agent_perform(action)10  orchestrator.after_one_action(action)11orchestrator.run(agent_run)

Listing 1: Pseudo code of integrating MobileAgentBench into an existing Mobile LLM Agent.

## 4 Experiments and Agent Evaluations

### 4.1 Metrics

We define 6 metrics to comprehensively benchmarking mobile agents:

*   •

    Success Rate (SR): ${SR}=N_{success}/M_{tasks}$, where $N_{success}$ is the number of successful tasks, judged by the benchmark system. $M_{tasks}$ is the number of total benchmarking tasks. This metric reflects the agent’s ability to correctly finish a task end-to-end.

*   •

    Step-wise Efficiency (SE). $SE=S_{actual}/S_{min}$, where $S_{actual}$ is the number of actual steps the agent takes to successfully finish a task, and $S_{min}$ is the minimum number of steps. This metric tells us if the agent performs unnecessary or redundant actions and reflects the efficiency of the agent. Failure tasks are not taken into account.

*   •

    Latency. The average time spent in seconds before and after one action. This metric tells us how long a user needs to wait between two actions.

*   •

    Tokens. The number of LLM input and output tokens. For simplicity, we use the GPT-4V (gpt-4-vision-preview) standard [[16](https://arxiv.org/html/2406.08184v1#bib.bib16)] to calculate the number of tokens for all models, which gives us a rough estimation of the LLM cost. For text, 1 token is 4 characters. For an image, it’s divided into multiple 512$\times$ 512 tiles, and each tile is 170 tokens. 85 base tokens are applied to each image as well.

*   •

    False Negative (FN) Rate. $FN=N_{early}/M_{failure}$, where $N_{early}$ is the number of early stopped tasks and $M_{failure}$ is the total number of failure tasks. This metric represents how likely the agent falsely thinks it has finished the task and stopped early.

*   •

    False Positive (FP) Rate. $FP=N_{late}/M_{success}$, where $N_{late}$ is the number of late stopped tasks and $M_{success}$ is the total number of successful tasks. Symmetricly to FN Rate, this metric reveals how likely the agent falsely thinks the task is not finished successfully.

### 4.2 Environment Setup

Five popular mobile LLM agents, AndroidArena [[24](https://arxiv.org/html/2406.08184v1#bib.bib24)], AutoDroid [[23](https://arxiv.org/html/2406.08184v1#bib.bib23)], AppAgent [[26](https://arxiv.org/html/2406.08184v1#bib.bib26)], CogAgent [[11](https://arxiv.org/html/2406.08184v1#bib.bib11)], and MobileAgent [[21](https://arxiv.org/html/2406.08184v1#bib.bib21)] are evaluated with the proposed benchmark. We choose the Google Pixel 3a emulator and Android 14 operating system to run the benchmarking apps. Besides, the Android 9 operating system is used for AutoDroid as some of the dependency libraries do not support the newer Android systems.

As there are no local neural networks used in AndroidArena, AutoDroid, and AppAgents, these agents are executed on an Apple Macbook Pro with the M1 Max chip. CogAgent and MobileAgent require local model referencing, so they are executed on a workstation equipped with a single Nvidia RTX 4090 GPU, with 24 GB GPU memory.

The self-exploration feature is turned on for AppAgent. When performing a task, it can reference the previously summarized document. For CogAgent, we use 4-bit quantization to load the model due to GPU memory limitation. CogAgent is implemented in its vanilla flavor, *i.e.*, given the current screenshot, ask for the next action. No history information is provided.

### 4.3 Results

Table 2: Agent performance results with multiple metrics

| Agent | SR $\uparrow$ | SE $\downarrow$ | Latency (s) $\downarrow$ | Tokens $\downarrow$ | FN Rate $\downarrow$ | FP Rate $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| AndroidArena [[24](https://arxiv.org/html/2406.08184v1#bib.bib24)] | 0.22 | 1.13 | 18.61 | 750.47 | 0.09 | 0.33 |
| AutoDroid [[23](https://arxiv.org/html/2406.08184v1#bib.bib23)] | 0.27 | 3.10 | 4.85 | 963.48 | 0.93 | 0.01 |
| AppAgent [[26](https://arxiv.org/html/2406.08184v1#bib.bib26)] | 0.40 | 1.29 | 26.09 | 1505.09 | 0.17 | 0.40 |
| CogAgent [[11](https://arxiv.org/html/2406.08184v1#bib.bib11)] | 0.08 | 2.42 | 6.76 | 579.84 | 1.0 | 0.04 |
| MobileAgent [[21](https://arxiv.org/html/2406.08184v1#bib.bib21)] | 0.26 | 1.33 | 15.91 | 1236.88 | 0.19 | 0.31 |

The main experiment results are shown in Tab. [2](https://arxiv.org/html/2406.08184v1#S4.T2 "Table 2 ‣ 4.3 Results ‣ 4 Experiments and Agent Evaluations ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"). We observe that AppAgent has the highest success rate, benefiting from the self-exploration mechanism. CogAgent has the lowest success rate, most likely caused by the naive agent implementation, which limits the usage of history information. Although AutoDroid has a similar success rate to MobileAgent, the step-wise efficiency is significantly lower, possibly caused by the weaker reasoning capability of the GPT-3.5 model used by AutoDroid. Latency-wise, both AutoDroid and CogAgent have very low latency, indicating the high inferencing cost with the GPT-4V model. AppAgent needs to look up the app document, thus consuming more tokens than others. On the other hand, because of the naive agent implementation of CogAgent, it consumes the least number of tokens. AutoDroid and CogAgent have very high FN rates, indicating they always stop early when the task is not finished yet. AppAgent, although having the highest task success rate, is not good at determining the task success task, it cannot stop gracefully after finishing a task and has a high FP rate.

![Refer to caption](img/e61b77d2cdfb00eca097bfed62926a7d.png)

(a) Task success rate with difficulty levels.

![Refer to caption](img/1db618be9b155fa9234bc82eb7a8b32b.png)

(b) Task success rate with LLMs.

Figure 4: Task success rate.

The task success rates for each agent with difficulty levels are shown in Fig. [4a](https://arxiv.org/html/2406.08184v1#S4.F4.sf1 "In Figure 4 ‣ 4.3 Results ‣ 4 Experiments and Agent Evaluations ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"). From Fig. [4a](https://arxiv.org/html/2406.08184v1#S4.F4.sf1 "In Figure 4 ‣ 4.3 Results ‣ 4 Experiments and Agent Evaluations ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents"), we can see most agents have higher success rates when handling easier tasks, which is as expected. Interestingly, AppAgent has a higher success rate when performing medium-level tasks. This is because we set the maximum execution steps as twice the minimum steps, which would make the agents have very limited steps to correct their earlier steps for easy tasks. For example, an agent can only use 1 additional step to correct and finish the task for a 1-step easy task. However, for medium and hard tasks, there is significantly more space to correct the previous actions.

Fig. [4b](https://arxiv.org/html/2406.08184v1#S4.F4.sf2 "In Figure 4 ‣ 4.3 Results ‣ 4 Experiments and Agent Evaluations ‣ MobileAgentBench: An Efficient and User-Friendly Benchmark for Mobile LLM Agents") shows the averaged task success rate over the backbone LLM models. It is interesting that AutoDroid, although using a text-based GPT-3.5 model, outperforms some other agents that use the more advanced GPT-4V model. This reveals that the textual view hierarchy contains the most important information for GUI navigation tasks. However, we believe that visual screenshots are helpful for other types of tasks, for example, if the task involves recognizing an image on the screen, or if the textual view hierarchy is not available.

## 5 Limitations and Future Work

While the proposed MobileAgentBench is efficient, user-friendly, and addresses many issues of the existing benchmarks for mobile LLM agents, there are two main limitations that the authors would like to improve in the future. Firstly, although the use of Python code snippets as the configuration of task success conditions is easy for researchers in the AI/ML community, it is still difficult for people without a technical background. We will explore new methods to automatically build the task configuration without writing any code in the future work. Additionally, the usage of UIAutomator makes it infeasible to dump the view hierarchy with dynamic screen contents. UIAutomator dumps the view hierarchy only if the main thread is free. It fails if the app contains persistent animations or videos. Therefore, replacing the UIAutomator with other frameworks to retrieve view information would be a promising direction.

## 6 Conclusion

In this paper, we propose a new benchmark, MobileAgentBench, for mobile LLM agents on the Android platform. With the 100 built-in benchmarking tasks, researchers can test and evaluate existing and new agents automatically on real Android devices. Extending the benchmark to support customized tasks is also easy, as only basic Python coding skills are needed. Leveraging the Android Accessibility Services and only checking the final app state, MobileAgentBench can detect task completion status faithfully. We report the evaluation results of 5 popular agents across multiple metrics, and they can be used as strong baselines to advance future mobile LLM agent development.

## References

*   [1] Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, et al. Gpt-4 technical report. arXiv preprint arXiv:2303.08774, 2023.
*   [2] Amazon. Alexa. [https://alexa.amazon.com](https://alexa.amazon.com), 2014.
*   [3] Apple. Siri. [https://www.apple.com/siri/](https://www.apple.com/siri/), 2011.
*   [4] Konstantinos Bousmalis, Giulia Vezzani, Dushyant Rao, Coline Devin, Alex X Lee, Maria Bauza, Todor Davchev, Yuxiang Zhou, Agrim Gupta, Akhil Raju, et al. Robocat: A self-improving foundation agent for robotic manipulation. arXiv preprint arXiv:2306.11706, 2023.
*   [5] Wei Chen and Zhiyuan Li. Octopus v2: On-device language model for super agent. arXiv preprint arXiv:2404.01744, 2024.
*   [6] Allan de Barcelos Silva, Marcio Miguel Gomes, Cristiano André da Costa, Rodrigo da Rosa Righi, Jorge Luis Victoria Barbosa, Gustavo Pessin, Geert De Doncker, and Gustavo Federizzi. Intelligent personal assistants: A systematic literature review. Expert Systems with Applications, 147:113193, 2020.
*   [7] Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Sam Stevens, Boshi Wang, Huan Sun, and Yu Su. Mind2web: Towards a generalist agent for the web. Advances in Neural Information Processing Systems, 36, 2024.
*   [8] Yuqing Du, Olivia Watkins, Zihan Wang, Cédric Colas, Trevor Darrell, Pieter Abbeel, Abhishek Gupta, and Jacob Andreas. Guiding pretraining in reinforcement learning with large language models. In International Conference on Machine Learning, pages 8657–8677\. PMLR, 2023.
*   [9] Google. Google assistant. [https://assistant.google.com](https://assistant.google.com), 2016.
*   [10] Izzeddin Gur, Ulrich Rueckert, Aleksandra Faust, and Dilek Hakkani-Tur. Learning to navigate the web. arXiv preprint arXiv:1812.09195, 2018.
*   [11] Wenyi Hong, Weihan Wang, Qingsong Lv, Jiazheng Xu, Wenmeng Yu, Junhui Ji, Yan Wang, Zihan Wang, Yuxiao Dong, Ming Ding, et al. Cogagent: A visual language model for gui agents. arXiv preprint arXiv:2312.08914, 2023.
*   [12] Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, et al. Retrieval-augmented generation for knowledge-intensive nlp tasks. Advances in Neural Information Processing Systems, 33:9459–9474, 2020.
*   [13] Yuanchun Li, Hao Wen, Weijun Wang, Xiangyu Li, Yizhen Yuan, Guohong Liu, Jiacheng Liu, Wenxing Xu, Xiang Wang, Yi Sun, et al. Personal llm agents: Insights and survey about the capability, efficiency and security. arXiv preprint arXiv:2401.05459, 2024.
*   [14] Shilong Liu, Zhaoyang Zeng, Tianhe Ren, Feng Li, Hao Zhang, Jie Yang, Chunyuan Li, Jianwei Yang, Hang Su, Jun Zhu, et al. Grounding dino: Marrying dino with grounded pre-training for open-set object detection. arXiv preprint arXiv:2303.05499, 2023.
*   [15] Microsoft. Cortana. [https://www.microsoft.com/en-us/cortana](https://www.microsoft.com/en-us/cortana), 2014.
*   [16] OpenAI. Gpt token calculation. [https://openai.com/api/pricing/](https://openai.com/api/pricing/).
*   [17] Shuofei Qiao, Yixin Ou, Ningyu Zhang, Xiang Chen, Yunzhi Yao, Shumin Deng, Chuanqi Tan, Fei Huang, and Huajun Chen. Reasoning with language model prompting: A survey. arXiv preprint arXiv:2212.09597, 2022.
*   [18] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark, et al. Learning transferable visual models from natural language supervision. In International conference on machine learning, pages 8748–8763\. PMLR, 2021.
*   [19] Christopher Rawles, Alice Li, Daniel Rodriguez, Oriana Riva, and Timothy Lillicrap. Android in the wild: A large-scale dataset for android device control. arXiv preprint arXiv:2307.10088, 2023.
*   [20] Scott Reed, Konrad Zolna, Emilio Parisotto, Sergio Gomez Colmenarejo, Alexander Novikov, Gabriel Barth-Maron, Mai Gimenez, Yury Sulsky, Jackie Kay, Jost Tobias Springenberg, et al. A generalist agent. arXiv preprint arXiv:2205.06175, 2022.
*   [21] Junyang Wang, Haiyang Xu, Jiabo Ye, Ming Yan, Weizhou Shen, Ji Zhang, Fei Huang, and Jitao Sang. Mobile-agent: Autonomous multi-modal mobile device agent with visual perception. arXiv preprint arXiv:2401.16158, 2024.
*   [22] Zihao Wang, Shaofei Cai, Guanzhou Chen, Anji Liu, Xiaojian Ma, and Yitao Liang. Describe, explain, plan and select: Interactive planning with large language models enables open-world multi-task agents. arXiv preprint arXiv:2302.01560, 2023.
*   [23] Hao Wen, Yuanchun Li, Guohong Liu, Shanhui Zhao, Tao Yu, Toby Jia-Jun Li, Shiqi Jiang, Yunhao Liu, Yaqin Zhang, and Yunxin Liu. Empowering llm to use smartphone for intelligent task automation. arXiv preprint arXiv:2308.15272, 2023.
*   [24] Mingzhe Xing, Rongkai Zhang, Hui Xue, Qi Chen, Fan Yang, and Zhen Xiao. Understanding the weakness of large language model agents within a complex android environment. arXiv preprint arXiv:2402.06596, 2024.
*   [25] Jianwei Yang, Hao Zhang, Feng Li, Xueyan Zou, Chunyuan Li, and Jianfeng Gao. Set-of-mark prompting unleashes extraordinary visual grounding in gpt-4v. arXiv preprint arXiv:2310.11441, 2023.
*   [26] Zhao Yang, Jiaxuan Liu, Yucheng Han, Xin Chen, Zebiao Huang, Bin Fu, and Gang Yu. Appagent: Multimodal agents as smartphone users. arXiv preprint arXiv:2312.13771, 2023.
*   [27] Shukang Yin, Chaoyou Fu, Sirui Zhao, Ke Li, Xing Sun, Tong Xu, and Enhong Chen. A survey on multimodal large language models. arXiv preprint arXiv:2306.13549, 2023.
*   [28] Li Zhang, Shihe Wang, Xianqing Jia, Zhihan Zheng, Yunhe Yan, Longxi Gao, Yuanchun Li, and Mengwei Xu. Llamatouch: A faithful and scalable testbed for mobile ui automation task evaluation. arXiv preprint arXiv:2404.16054, 2024.
*   [29] Shuyan Zhou, Frank F Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Yonatan Bisk, Daniel Fried, Uri Alon, et al. Webarena: A realistic web environment for building autonomous agents. arXiv preprint arXiv:2307.13854, 2023.