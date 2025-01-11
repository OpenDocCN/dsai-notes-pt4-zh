<!--yml
category: 未分类
date: 2025-01-11 13:07:52
-->

# DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue

> 来源：[https://arxiv.org/html/2308.08043/](https://arxiv.org/html/2308.08043/)

Lang Cao
University of Illinois Urbana-Champaign
Department of Computer Science
langcao2@illinois.edu 

###### Abstract

A significant application of Large Language Models (LLMs), like ChatGPT, is their deployment as chat agents, which respond to human inquiries across a variety of domains. While current LLMs proficiently answer general questions, they often fall short in complex diagnostic scenarios such as legal, medical, or other specialized consultations. These scenarios typically require Task-Oriented Dialogue (TOD), where an AI chat agent must proactively pose questions and guide users toward specific goals or task completion. Previous fine-tuning models have underperformed in TOD and the full potential of conversational capability in current LLMs has not yet been fully explored. In this paper, we introduce DiagGPT (Dialogue in Diagnosis GPT), an innovative approach that extends LLMs to more TOD scenarios. In addition to guiding users to complete tasks, DiagGPT can effectively manage the status of all topics throughout the dialogue development. This feature enhances user experience and offers a more flexible interaction in TOD. Our experiments demonstrate that DiagGPT exhibits outstanding performance in conducting TOD with users, showing its potential for practical applications in various fields.

DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue

Lang Cao University of Illinois Urbana-Champaign Department of Computer Science langcao2@illinois.edu

## 1 Introduction

Figure 1: The main difference between ChatGPT and DiagGPT. While ChatGPT directly answers user questions, DiagGPT not only provides answers of the same quality but also has the ability to proactively ask questions, guide users, and maintain dialogue state internally.

Large language models (LLMs), such as ChatGPT, have demonstrated remarkable performance on various natural language processing (NLP) tasks Brown et al. ([2020](https://arxiv.org/html/2308.08043v4#bib.bib3)); Chowdhery et al. ([2022](https://arxiv.org/html/2308.08043v4#bib.bib4)); Wei et al. ([2022a](https://arxiv.org/html/2308.08043v4#bib.bib14)); OpenAI ([2023](https://arxiv.org/html/2308.08043v4#bib.bib11)). In many cases, OpenAI GPT-4 even outperforms human performance OpenAI ([2023](https://arxiv.org/html/2308.08043v4#bib.bib11)). With the use of prompt engineering techniques (e.g., chain-of-thought prompting Brown et al. ([2020](https://arxiv.org/html/2308.08043v4#bib.bib3)); Wei et al. ([2022b](https://arxiv.org/html/2308.08043v4#bib.bib15)), in-context learning Brown et al. ([2020](https://arxiv.org/html/2308.08043v4#bib.bib3)); Xie et al. ([2022](https://arxiv.org/html/2308.08043v4#bib.bib18)); Min et al. ([2022](https://arxiv.org/html/2308.08043v4#bib.bib9)), etc.), we can improve reasoning and understanding of LLMs to complete complex tasks in our daily life. LLMs have attracted enormous attention from both academia and industry, inspiring more people to build useful applications based on them.

One popular application of LLMs is in chatbots, which build conversational systems around these models. ChatGPT¹¹1https://openai.com/blog/chatgpt is a successful example of such an application, where LLMs have the ability to analyze context and respond to user queries based on knowledge derived from extensive training data. By supplementing its background knowledge and providing context and appropriate prompts, ChatGPT has been able to form robust question-answering models for specialized fields. It can understand users’ questions and provide precise answers effectively.

However, dialogue scenarios in our daily life can be more complex. For instance, in specialized professional consultation scenarios like legal or medical diagnosis, the chat agent needs to consider the user’s unique situation or information. In the process of obtaining user information, the interactive experience provided by the agent is also crucial. The system needs to proactively ask questions. Therefore, we need a consultation process from the chat agent that better simulates real medical experts and legal professionals. The chat agent should conduct question-answering, topic management, and guiding users towards specific goals or task completion. This type of dialogue is known as Task-Oriented Dialogue (TOD). There are usually some predefined goals in a conversation. TOD helps users achieve their specific goals, focusing on understanding users, tracking states, and generating next actions Balaraman et al. ([2021](https://arxiv.org/html/2308.08043v4#bib.bib1)). It is substantially different from light-conversational or open-domain dialogue scenarios. Despite much research in this area, it remains challenging due to issues such as a lack of training data, inefficiency, and drawbacks of fine-tuning small models, including an inability to fully understand user meaning and poor generative performance. Models from the existing research on this topic is not robust and universal. For example, fine-tuning models require a lot of data to train and are difficult to transfer to other scenarios. On the other side, although LLMs have a wide range of knowledge and the quality of their answer is far beyond that of fine-tuning models, traditional LLMs no longer meet needs of TOD and cannot effectively manage complex dialogue logic. Because they maintain a simple memory base and can only handle linear interaction.

Recent advancements have focused on using LLMs as agents to construct multi-agent systems or to teach LLMs how to use tools to accomplish more complex tasks Schick et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib12)); Shen et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib13)). These systems typically have a core agent that controls the entire task process. A prominent example is AutoGPT²²2https://github.com/Significant-Gravitas/Auto-GPT, which employs multiple GPT models to strategize the responsibilities of each agent in order to split complex tasks and then complete them. In such multi-agent systems, the key lies in the division of tasks and the interaction between agents. By organizing multiple LLMs and instruct them to collaborate, we have the opportunity to tackle many complex tasks which one LLM cannot do well.

Fine-tuning models fall short in terms of scenario transfer ability and massive training data requirements, while a single large language model is not proficient in dialogue state tracking and management. However, we can leverage the strong knowledge background of LLMs and employ a multi-agent framework to incorporate the dialogue state tracking and management ideas from fine-tuning models. Thus, we can construct a multi-agent dialogue system that fulfills the requirements of flexible task-oriented dialogue.

Motivated by these considerations, we propose DiagGPT in this paper. DiagGPT stands for Dialogue in Diagnosis model based on GPT-4\. This is a multi-agent AI system, which has automatic topic management ability to enhance its utility for task-oriented dialogue in a more flexible way. Instead of mechanically and rigidly guiding the user through the task in traditional task-oriented dialogue, we can enhance the interactive experience in the development of dialogue, and the dialogue is more smooth. This is more in line with actual situations and real conversations. In summary, our AI system DiagGPT possesses the following features:

*   •

    Question Answering It is a basic feature of traditional LLM-based conversational systems. LLMs possess a wide range of knowledge and can provide high-quality answers to various questions. we build upon and retain this essential capability of LLMs.

*   •

    Task Guidance The system is designed to guide users towards a specific goal and assist them in accomplishing the task throughout the dialogue progression. This is achieved by advancing a sequence of predefined topics throughout the dialogue.

*   •

    Proactive Asking The system has the ability to proactively pose questions based on a predefined checklist, thereby collecting necessary information from users.

*   •

    Topic Management The system is capable of automatically managing topics throughout the dialogue, tracking topic progression, and effectively engaging in discussions centered around the current topic. It performs well in managing various topic changes in complex dialogues.

*   •

    Versatile Our system is directly based on LLMs. It can perform well in various scenarios without requiring any training data, a capability that previous fine-tuning models lack. This system can be easily applied to multiple cases by defining specific predefined goals and supplementing functions to support them.

Given these features, DiagGPT can meet the aforementioned needs and better engage in professional consultation conversations with users in complex scenarios. Our main contribution is to make traditional LLM-based conversational systems smarter in flexible task-oriented dialogue. We build on the strong knowledge of LLMs and give them more interactive capabilities. Therefore, DiagGPT can function like a more intelligent and more professional chatbot.

## 2 Related Work

Task-Oriented Dialogue systems assist users in achieving specific goals, focusing on understanding users, tracking states, and generating subsequent actions. Task-Oriented Dialogue (TOD) is a task in which the goal is to accomplish a specific objective. Recent work primarily focusing on fine-tuning small models. Wen et al. ([2017](https://arxiv.org/html/2308.08043v4#bib.bib16)) introduce a neural network-based text-in, text-out end-to-end trainable task-oriented dialogue system along with a new way of collecting dialogue data based on a novel pipe-lined Wizard-of-Oz framework. Wu et al. ([2019](https://arxiv.org/html/2308.08043v4#bib.bib17)) propose a Transferable Dialogue State Generator (TRADE) that generates dialogue states from utterances using copy mechanism, facilitating transfer when predicting (domain, slot, value) triplets not encountered during training. Feng et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib5)) propose SG-USM, a novel schema-guided user satisfaction modeling framework. It explicitly models the degree to which the user’s preferences regarding the task attributes are fulfilled by the system for predicting the user’s satisfaction level. Liu et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib8)) propose a framework called MUST to optimize TOD systems via leveraging Multiple User Simulator. Bang et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib2)) propose an End-to-end TOD system with Task-Optimized Adapters which learn independently per task, adding only small number of parameters after fixed layers of pre-trained network. All these methods require a considerable amount of data for training and have not yet attained a performance level that is ideal for real-world applications.

Conversational Systems with LLMs have become popular as the robust capabilities of LLMs have been recognized. Hudeček and Dušek ([2023](https://arxiv.org/html/2308.08043v4#bib.bib6)) evaluated the conversational ability of LLMs and found that, in explicit belief state tracking, LLMs underperform compared to specialized task-specific models. This suggests that simple LLMs do not have the ability to achieve task-oriented dialogue. Liang et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib7)) proposed an interactive conversation visualization system called C5, which includes Global View, Topic View, and Context-associated QA View to better retain contextual information and provide comprehensive responses. From another perspective, Zhang et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib20)) proposed the Ask an Expert framework in which the model is trained with access to an expert whom it can consult at each turn. This framework utilizes LLMs to improve fine-tuning small models in TOD. There is minimal work on improving the conversational ability of LLMs. To the best of our knowledge, we are the first to successfully use off-the-shelf LLMs in a multi-agent framework to build a task-oriented dialogue system.

## 3 Methodology

### 3.1 DiagGPT Framework

DiagGPT is a multi-agent and collaborative system composed of several modules: Chat Agent, Topic Manager, Topic Enricher, and Context Manager. Each module is a LLM with specific prompts that guide their function and responsibility. Among these modules, the Topic Manager is the most important as it tracks the dialogue state and automatically manages the dialogue topic.

Figure 2: The framework of DiagGPT. The workflow of DiagGPT consists of four stages: Thinking Topic Development, Maintaining Topic Stack, Enriching Topic, Generating Response.

As shown in Figure [2](https://arxiv.org/html/2308.08043v4#S3.F2 "Figure 2 ‣ 3.1 DiagGPT Framework ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"), the workflow of DiagGPT consists of four stages: 1) Thinking Topic Development: Topic Manager obtain the user query, then analyze and predict the topic development in current round of dialogue; 2) Maintaining Topic Stack: maintain the topic stack of the entire dialogue according to action commands from Topic Manager; 3) Enriching Topic: retrieve the current topic and enrich it based on dialogue context; 4) Generating Response: based on specific guidance prompt and combined it with enriched topic and context to generate response for users.

Besides, we define a topic as the main subject of a round of dialogue, which determines the primary focus of communication. We also define a task as a specific goal that needs to be completed in a task-oriented dialogue. After going through all the predefined topics in a dialogue, this specific task should be accomplished.

### 3.2 Thinking Topic Development

Topic Manager serves as the main module in DiagGPT and is responsible for determining the topic development based on the user’s query. In each round of dialogue, the system needs to adjust the current dialogue topic before providing its response. Therefore, the user’s query is first fed into Topic Manager.

The input to the Topic Manager includes the current user query, action list, the current status of the topic stack, and the chat history. It is logical for an AI agent to analyze and predict the topic development based on this information. Of particular importance is the action list stored in the Topic Manager. This action list contains various actions that serve as tools for the Topic Manager to execute. The Topic Manager has knowledge about the details of each action, how to plan and execute them. Each action corresponds to a program function that executes a specific command. In Python, we use decorator functions to implement this. Whenever Topic Manager receives a user query, it analyzes all the available information and decides which action to execute based on the prompts associated with each action.

With the strong understanding and reasoning abilities of LLMs, this AI agent can accurately comprehend the user’s intentions and help to effectively engage in communication with the user.

### 3.3 Maintaining Topic Stack

After obtaining the output of the action from the Topic Manager, the system will execute the corresponding command to process and control the topic change, which involves maintaining the topic stack.

The topic stack is a data structure in this system that stores and tracks the dialogue state. Stack management is a kind of simulation of dialog state management in both short-term and long-term context. In our system, we concretize the dialogue state of the LLM and explicitly store it, allowing the system to perform better over the long term context and avoid ambiguity and forgetfulness. We consider the progress of a dialogue to have multiple stages or states, and these states follow a first-in, first-out (FIFO) order, which can be effectively modeled using a stack. Although we refer to this topic storage structure as a stack, this component does not strictly adhere to the FIFO rule. This is due to the presence of complex dialogue logic, and FIFO is merely the most common form of topic development. We will provide a detailed description of some operations of this structure in the following paragraphs.

In a diagnosis scenario, a consultant typically has a checklist stored in their mind. In many common cases, if users do not propose any new questions, the dialogue development will follow this checklist. After going through all the items in the checklist, the consultant can provide reports and comprehensive analysis to the users and complete the specific task. The action load topics from a predefined task is designed to facilitate this process. When the function decorated by this action prompt is executed, a list of topics from the checklist, will be loaded into the topic stack.

Figure 3: The action of create a new topic.

Figure 4: The action of finish the current topic.

Furthermore, there are other actions commonly used to manipulate the topic stack. These actions include create a new topic, finish the current topic, and stay at the current topic. The create a new topic action, as shown in Figure [3](https://arxiv.org/html/2308.08043v4#S3.F3 "Figure 3 ‣ 3.3 Maintaining Topic Stack ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"), adds a new topic to the stack when the user wants to start a new topic. The finish the current topic action, shown in Figure [4](https://arxiv.org/html/2308.08043v4#S3.F4 "Figure 4 ‣ 3.3 Maintaining Topic Stack ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"), removes the top topic from the stack when the user no longer wishes to discuss it or the system considers this topic to be closed. The stay at the current topic action indicates that the system determines that it still requires information and needs continuous discussing the current topic, so the topic stack does not change at all. These three basic operations cover most topic change scenarios. Since we only allow one-step reasoning for LLMs, they must select and execute only one action. Other actions are complex changes based on these three basic operations.

The jump to an existing topic action, illustrated in Figure [5](https://arxiv.org/html/2308.08043v4#S3.F5 "Figure 5 ‣ 3.3 Maintaining Topic Stack ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"), allows the user to retrieve and prioritize a previous topic from the stack.

We apply a action list to allow this system to interact with users in a more flexible way. It can avoid mechanically and rigidly guiding the user through the task in traditional task-oriented dialogue. This action list can also be expanded to accommodate more complex scenarios in task-oriented dialogues. We have also implemented a mechanism to automatically remove redundant topics. After several rounds of dialogue, if a newly generated topic is not recalled, it will be removed. However, this removal does not affect any predefined topics from checklist.

Figure 5: The action of jump to an existing topic.

|  | Human Evaluation | LLM Evaluation |
| --- | --- | --- |
|  | RC $\downarrow$ | CR $\uparrow$ | SR $\uparrow$ | CS $\uparrow$ | CR $\uparrow$ | SR $\uparrow$ | RQ $\uparrow$ | CS $\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ChatGPT (gpt-3.5-turbo) | 9.0 | 0.8 | 0.4 | - | 0.8 | 0.7 | 7.8 | - |
| ChatGPT (gpt-4) | 7.7 | 1.0 | 1.0 | 5.0 | 1.0 | 1.0 | 9.0 | 8.5 |
| DiagGPT (Ours) | 7.0 | 1.0 | 1.0 | 15.0 | 1.0 | 1.0 | 9.0 | 11.5 |

Table 1: Quantitative experimental results. We compare our method DiagGPT with ChatGPT (gpt-turbo-3.5 and gpt-4) across 20 diverse dialogue scenarios, assessed by Round Count (RC), Completion Rate (CR), Response Quality (RQ), and Comparison Score (CS).

### 3.4 Enriching Topic

We select the top item in the topic stack as the current topic. However, it cannot be directly used as a chat topic for the Chat Agent to interact with the user. Topics in the topic stack are simple and convenient to store, whereas topics for the practical use of the Chat Agent need to include more information. Without a topic enricher, it is difficult for the Chat Agent to directly understand the main objective in the current round of dialogue.

The Topic Enricher is designed to bridge this gap and assist in better organizing the language for use. We initially categorize the topic into Ask user and Answer user. Typically, newly generated topics fall under Answer user, while predefined topics are categorized as Ask user. This design helps the system determine whether to answer a user’s question or ask a question in the current round of dialogue. The Topic Enricher takes the output of the Context Manager and the current topic to enrich it into a topic that contains more context information. This enriched topic is then provided to the Chat Agent.

### 3.5 Generating Response

With the final topic, the Chat Agent recognizes it as the primary topic in this round of dialogue. Thus, with context from the Context Manager, it can finally generate responses for users. In addition, as shown in Figure [10](https://arxiv.org/html/2308.08043v4#A4.F10 "Figure 10 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"), some retrieved background knowledge, instructions, and encouragements will also be added into prompts here to further improve the response quality.

## 4 Experiments

### 4.1 Setups

We conduct both quantitative experiments and qualitative experiments to demonstrate the performance of DiagGPT. Details of the system implementation can be found in Appendix [B](https://arxiv.org/html/2308.08043v4#A2 "Appendix B Implementation Details of DiagGPT ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue").

The quantitative evaluation of DiagGPT is challenging due to the difficulty in quantifying the system’s ability to manage various topics that develop in a dialogue. Therefore, we employ quantitative experiments to only assess the system’s basic capabilities in topic advancement and guidance. We do not evaluate our system using task-oriented dialogue datasets such as DialoGLUE Yadav et al. ([2019](https://arxiv.org/html/2308.08043v4#bib.bib19)); Moghe et al. ([2023](https://arxiv.org/html/2308.08043v4#bib.bib10)). As DiagGPT is an open system, and it does not require training unlike fine-tuning models. The output of our system is conditioned solely on prompts. Moreover, these traditional TOD datasets focuses on specific subtasks (intent prediction, slot filling, etc.) that were considered necessary for TOD, while LLM-based systems circumvents these specific tasks in a more end-to-end fashion. Therefore, evaluating our system using traditional methods is challenging.

To quantitatively evaluate our system, we develop a new dataset called LLM-TOD, which encompasses 20 different topics in task-oriented dialogue scenarios (details are shown in Appendix [C](https://arxiv.org/html/2308.08043v4#A3 "Appendix C LLM-TOD Dataset ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")). Each data of a TOD scenario includes a checklist of items to go through, finally achieving a specific goal. This checklist and the goal are input into the LLMs to complete a TOD. Besides, we develop a new evaluation schema based on LLMs for TOD tasks. An LLM-based user simulator is also employed to interact with the dialogue system. The results are evaluated using a combination of human assessment and another LLM. During the LLM evaluation, the inputs include basic TOD information and the complete chat history.

On the other hand, qualitative experiments more effectively demonstrate the capabilities and performance of DiagGPT. These experiments show that DiagGPT can conduct task-oriented dialogues more flexibly.

### 4.2 Quantitative Experiment Results

We evaluate the performance through both human evaluation and LLM evaluation. We compared our method with two ChatGPT baselines: gpt-3.5-turbo and gpt-4. The evaluation involved the following metrics:

*   •

    Round Count (RC): The average number of rounds required to complete a task-oriented dialogue. This metric can be seen as a measure of system efficiency, where lower values are preferable.

*   •

    Completion Rate (CR): The percentage of completed checklist items out of the total number of items. An ideal system should complete all items on the checklist.

*   •

    Success Rate (SR): The indicator of whether the system successfully completed the task to achieve the intended goal. This metric is a value of 0 or 1 for each task-oriented dialogue.

*   •

    Response Quality (RQ): A score assessed using an LLM to evaluate the quality of the entire chat history and system response. The values range from 1 to 10, as generated by an LLM.

*   •

    Comparison Score (CS): The score of comparison between responses from ChatGPT (gpt-4) and DiagGPT within a single dialogue. It is calculated based on the total number of wins in various scenarios, indicating which model performed better.

In detail, we calculated the average score across 20 different TOD scenarios. For the comparison score from the LLM evaluation, we alternated the position of the chat history from two systems in the prompt and evaluated twice to obtain an average score. This approach helps to avoid positional bias in automatic evaluation with LLMs. For the evaluation of response quality, we consider factors such as understanding, relevance, experience, and sufficiency, etc.

From Table [1](https://arxiv.org/html/2308.08043v4#S3.T1 "Table 1 ‣ 3.3 Maintaining Topic Stack ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"), we can observe the results of the quantitative experiment. This table shows that DiagGPT can match the performance of the state-of-the-art LLM, gpt-4, and effectively manage task-oriented dialogue in terms of response quality. Our system completes all tasks on the checklist and successfully achieves the goals in all TOD scenarios with a lower round count compared to ChatGPT. It demonstrates superior efficiency in conducting TOD by leading users to their goals and completing tasks effectively, which is crucial in TOD. Given the comparable response quality, we directly compared responses from the two systems. Both human and LLM evaluation results suggest that our system is more effective and perform better.

Our advantage over gpt-4 is that DiagGPT interacts with users more flexibly through its automatic topic management. This feature becomes more apparent in the subsequent qualitative experiment.

Figure 6: A dialogue example in the medical consulting scenario. The system DiagGPT acts as a real doctor to proactively ask for information and answer the user’s questions.

Figure 7: The continued dialogue example in the medical consulting scenario.

Figure 8: A dialogue example in the medical consulting scenario when the user ask some questions about COVID-19.

### 4.3 Qualitative Experiment Results

We modified the settings in DiagGPT to adapt it for use in medical dialogue scenarios for conducting qualitative experiments. Figure [6](https://arxiv.org/html/2308.08043v4#S4.F6 "Figure 6 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue") and Figure [7](https://arxiv.org/html/2308.08043v4#S4.F7 "Figure 7 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue") present a complete dialogue demonstration in the medical consulting process. This is a medical diagnosis where the task is to help the patient identify the cause and give advice. Medical consulting is not a pure open-end question-answering. Because patients usually lack extensive medical knowledge, they rely on doctors to instruct them to give their personal information. Therefore, users can experience a real doctor, not a dull medical question-answering machine. The user acts as a patient, while the system is a doctor, initially collecting information and gradually providing advice to the patient. The main dialogue development follows the checklist: Basic information, Chief complaint, Duration of symptoms, Severity of symptoms, which are also predefined topics.

This demonstration has showcased the robust conversational ability of the DiagGPT, which can proactively ask questions and guide the user to the final goal of the task, thereby achieving task-oriented dialogue. It simulates many real consulting scenarios. Other conversational systems, such as general ChatGPT, cannot achieve this performance. They usually can only answer users’ questions and find it challenging to complete specific goals in complicated scenarios, even with elaborated prompts.

### 4.4 Case Study of Automatic Topic Management

The core capability of DiagGPT is to automatically manage topics throughout the dialogue, which is the function and primary responsibility of Topic Manager. Figure [6](https://arxiv.org/html/2308.08043v4#S4.F6 "Figure 6 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue") and Figure [7](https://arxiv.org/html/2308.08043v4#S4.F7 "Figure 7 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue") illustrate the primary checklist progression. The topics from the checklist are retrieved and discussed sequentially, demonstrating the action of finish the current topic. When the conversation reaches the severity of symptoms, we observe that the dialogue topic remains here in several rounds of dialogue, allowing the system to have time on understanding the user’s conditions. This is a typical situation in real-world scenarios where users do not provide enough information for the doctor. The action of create a new topic is shown in Figure [8](https://arxiv.org/html/2308.08043v4#S4.F8 "Figure 8 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"). Here, the user actively consults the system with some information about COVID-19 to check symptoms. We observe that the system generates a new topic about COVID-19 and discusses this, rather than rigidly following the checklist. These results all demonstrate the effectiveness of Topic Manager. These case studies fully showcase the system’s flexible understanding capabilities, demonstrating its proficient handling of various scenarios that may occur in real-world dialogues.

## 5 Conclusion

In this paper, we propose DiagGPT, an LLM-based and multi-agent system designed to complete task-oriented dialogue tasks in a flexible way. The principle of our system is to leverage the strong understanding and reasoning capabilities of Large Language Models to implement agents that can automatically manage topics and track dialogue state. This feature enables our system to accurately comprehend intentions of users in different situations, assisting them in completing specific tasks more flexibly. The system is particularly well-suited for real-world consulting scenarios in flexible TOD. Both quantitative and qualitative experiments demonstrate the superior performance of DiagGPT.

However, DiagGPT represents a experimental system designed to demonstrate the potential of LLMs in handling real tasks. In the future, we aim to explore how to better serve users by leveraging the robust capabilities of LLMs to achieve more functionality and complete more complicated tasks.

## Limitation

Hallucination. Hallucination is a major concern when it comes to the response of LLMs. Some individuals argue that LLMs cannot be utilized in serious scenarios such as healthcare or legal domains due to hallucinations. The relevant experiments of medical consulting in this paper only demonstrate the topic management ability of DiagGPT. It has nothing to do with the hallucination or quality of medical answers, but only reflects the consultation process DiagGPT shows. Besides, we acknowledge that there are some harmful responses from LLMs. On one hand, this paper does not primarily focus on mitigating hallucination. On the other hand, significant progress has been made in reducing hallucination of LLMs through various other studies. Thus, the application of this conversational system still holds immense potential in real-life situations.

Cost and Efficiency. DiagGPT involves multiple LLMs. In just one round of dialogue with a single user query, all of these LLMs need to run with elaborated prompts, which is quite costly. Compared to simpler dialogue systems that involve just one LLM interacting with users, DiagGPT requires internal interactions among AI agents, thus taking more time to provide user feedback. Furthermore, AI agent in our system requires strong understanding and reasoning abilities, necessitating a robust and large-scale AI to maintain these capabilities. This also increases the overall system cost. However, we believe that with the future development of AI infrastructure, these issues could be mitigated.

Stability. The performance of DiagGPT is not as stable as some rule-based or fine-tuned dialogue models. The main issue arises when the Topic Manager decides the direction of dialogue development. It requires a strong understanding and reasoning ability from the AI, or else it may lead to system instability. Additionally, for every different applied scenario, meticulous and detailed prompt adjustments of the AI system are needed. Given the risk of LLMs’ output, some post-processing of responses is also required. Nevertheless, DiagGPT maintains unlimited potential for dialogue systems. With stronger AI in the future, dialogue systems could improve significantly and become more human-like.

Limited Experiments. We are unable to conduct extensive quantitative experiments to demonstrate the performance of DiagGPT in the most direct manner, as we previously mentioned due to its inherent characteristics. Additionally, the scope of the results obtained from quantitative experiments is limited. Although we provide only one usage example in medical consulting, our system can also exhibit similarly impressive performance in legal and numerous other scenarios, which is validated by professional experts. The basic operational processes in different applications are the same. Moreover, through careful observation of the responses generated by DiagGPT, we can discern that each module effectively fulfills its role, thus showcasing its overall strong performance.

## References

*   Balaraman et al. (2021) Vevake Balaraman, Seyedmostafa Sheikhalishahi, and Bernardo Magnini. 2021. [Recent neural methods on dialogue state tracking for task-oriented dialogue systems: A survey](https://aclanthology.org/2021.sigdial-1.25). In *Proceedings of the 22nd Annual Meeting of the Special Interest Group on Discourse and Dialogue*, pages 239–251, Singapore and Online. Association for Computational Linguistics.
*   Bang et al. (2023) Namo Bang, Jeehyun Lee, and Myoung-Wan Koo. 2023. [Task-optimized adapters for an end-to-end task-oriented dialogue system](https://doi.org/10.18653/v1/2023.findings-acl.464). In *Findings of the Association for Computational Linguistics: ACL 2023*, pages 7355–7369, Toronto, Canada. Association for Computational Linguistics.
*   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel Ziegler, Jeffrey Wu, Clemens Winter, Chris Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, and Dario Amodei. 2020. [Language models are few-shot learners](https://proceedings.neurips.cc/paper_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf). In *Advances in Neural Information Processing Systems*, volume 33, pages 1877–1901\. Curran Associates, Inc.
*   Chowdhery et al. (2022) Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann, Parker Schuh, Kensen Shi, Sasha Tsvyashchenko, Joshua Maynez, Abhishek Rao, Parker Barnes, Yi Tay, Noam Shazeer, Vinodkumar Prabhakaran, Emily Reif, Nan Du, Ben Hutchinson, Reiner Pope, James Bradbury, Jacob Austin, Michael Isard, Guy Gur-Ari, Pengcheng Yin, Toju Duke, Anselm Levskaya, Sanjay Ghemawat, Sunipa Dev, Henryk Michalewski, Xavier Garcia, Vedant Misra, Kevin Robinson, Liam Fedus, Denny Zhou, Daphne Ippolito, David Luan, Hyeontaek Lim, Barret Zoph, Alexander Spiridonov, Ryan Sepassi, David Dohan, Shivani Agrawal, Mark Omernick, Andrew M. Dai, Thanumalayan Sankaranarayana Pillai, Marie Pellat, Aitor Lewkowycz, Erica Moreira, Rewon Child, Oleksandr Polozov, Katherine Lee, Zongwei Zhou, Xuezhi Wang, Brennan Saeta, Mark Diaz, Orhan Firat, Michele Catasta, Jason Wei, Kathy Meier-Hellstern, Douglas Eck, Jeff Dean, Slav Petrov, and Noah Fiedel. 2022. [Palm: Scaling language modeling with pathways](http://arxiv.org/abs/2204.02311).
*   Feng et al. (2023) Yue Feng, Yunlong Jiao, Animesh Prasad, Nikolaos Aletras, Emine Yilmaz, and Gabriella Kazai. 2023. [Schema-guided user satisfaction modeling for task-oriented dialogues](https://doi.org/10.18653/v1/2023.acl-long.116). In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 2079–2091, Toronto, Canada. Association for Computational Linguistics.
*   Hudeček and Dušek (2023) Vojtěch Hudeček and Ondřej Dušek. 2023. [Are llms all you need for task-oriented dialogue?](http://arxiv.org/abs/2304.06556)
*   Liang et al. (2023) Pan Liang, Danwei Ye, Zihao Zhu, Yunchao Wang, Wang Xia, Ronghua Liang, and Guodao Sun. 2023. [C5: Towards better conversation comprehension and contextual continuity for chatgpt](http://arxiv.org/abs/2308.05567).
*   Liu et al. (2023) Yajiao Liu, Xin Jiang, Yichun Yin, Yasheng Wang, Fei Mi, Qun Liu, Xiang Wan, and Benyou Wang. 2023. [One cannot stand for everyone! leveraging multiple user simulators to train task-oriented dialogue systems](https://doi.org/10.18653/v1/2023.acl-long.1). In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 1–21, Toronto, Canada. Association for Computational Linguistics.
*   Min et al. (2022) Sewon Min, Xinxi Lyu, Ari Holtzman, Mikel Artetxe, Mike Lewis, Hannaneh Hajishirzi, and Luke Zettlemoyer. 2022. [Rethinking the role of demonstrations: What makes in-context learning work?](https://doi.org/10.18653/v1/2022.emnlp-main.759) In *Proceedings of the 2022 Conference on Empirical Methods in Natural Language Processing*, pages 11048–11064, Abu Dhabi, United Arab Emirates. Association for Computational Linguistics.
*   Moghe et al. (2023) Nikita Moghe, Evgeniia Razumovskaia, Liane Guillou, Ivan Vulić, Anna Korhonen, and Alexandra Birch. 2023. [Multi3NLU++: A multilingual, multi-intent, multi-domain dataset for natural language understanding in task-oriented dialogue](https://doi.org/10.18653/v1/2023.findings-acl.230). In *Findings of the Association for Computational Linguistics: ACL 2023*, pages 3732–3755, Toronto, Canada. Association for Computational Linguistics.
*   OpenAI (2023) OpenAI. 2023. [Gpt-4 technical report](http://arxiv.org/abs/2303.08774).
*   Schick et al. (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Luke Zettlemoyer, Nicola Cancedda, and Thomas Scialom. 2023. [Toolformer: Language models can teach themselves to use tools](http://arxiv.org/abs/2302.04761).
*   Shen et al. (2023) Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu, and Yueting Zhuang. 2023. [Hugginggpt: Solving ai tasks with chatgpt and its friends in hugging face](http://arxiv.org/abs/2303.17580).
*   Wei et al. (2022a) Jason Wei, Yi Tay, Rishi Bommasani, Colin Raffel, Barret Zoph, Sebastian Borgeaud, Dani Yogatama, Maarten Bosma, Denny Zhou, Donald Metzler, Ed H. Chi, Tatsunori Hashimoto, Oriol Vinyals, Percy Liang, Jeff Dean, and William Fedus. 2022a. [Emergent abilities of large language models](http://arxiv.org/abs/2206.07682).
*   Wei et al. (2022b) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022b. Chain-of-thought prompting elicits reasoning in large language models. *Advances in Neural Information Processing Systems*, 35:24824–24837.
*   Wen et al. (2017) Tsung-Hsien Wen, David Vandyke, Nikola Mrkšić, Milica Gašić, Lina M. Rojas-Barahona, Pei-Hao Su, Stefan Ultes, and Steve Young. 2017. [A network-based end-to-end trainable task-oriented dialogue system](https://aclanthology.org/E17-1042). In *Proceedings of the 15th Conference of the European Chapter of the Association for Computational Linguistics: Volume 1, Long Papers*, pages 438–449, Valencia, Spain. Association for Computational Linguistics.
*   Wu et al. (2019) Chien-Sheng Wu, Andrea Madotto, Ehsan Hosseini-Asl, Caiming Xiong, Richard Socher, and Pascale Fung. 2019. [Transferable multi-domain state generator for task-oriented dialogue systems](https://doi.org/10.18653/v1/P19-1078). In *Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics*, pages 808–819, Florence, Italy. Association for Computational Linguistics.
*   Xie et al. (2022) Sang Michael Xie, Aditi Raghunathan, Percy Liang, and Tengyu Ma. 2022. [An explanation of in-context learning as implicit bayesian inference](https://openreview.net/forum?id=RdJVFCHjUMI). In *International Conference on Learning Representations*.
*   Yadav et al. (2019) Deshraj Yadav, Rishabh Jain, Harsh Agrawal, Prithvijit Chattopadhyay, Taranjeet Singh, Akash Jain, Shiv Baran Singh, Stefan Lee, and Dhruv Batra. 2019. Evalai: Towards better evaluation systems for ai agents.
*   Zhang et al. (2023) Qiang Zhang, Jason Naradowsky, and Yusuke Miyao. 2023. [Ask an expert: Leveraging language models to improve strategic reasoning in goal-oriented dialogue models](http://arxiv.org/abs/2305.17878).

## Appendix A Analysis of DiagGPT Extendibility

In this paper, we only introduce the basic framework of the system DiagGPT aimed at achieving task-oriented dialogue. We have designed the system with ample flexibility to incorporate additional functions to handle tasks in complex scenarios and to meet more needs of conversational systems. Besides, as the foundation model is upgraded, the performance of our system will also become better. In detail, we have only extracted the most important modules in DiagGPT to form a basic framework of a system. These four modules can already implement the basic functions of task-oriented dialogue. In a multi-agent system, there is a large extendibility. For example, an information collector can monitor user input and organize information into structured data for better future utilization.

Sometimes, a conversation is aimed at solving certain tasks during or after the conversation. When a conversation achieves the predefined goal, the system can call more complex programs to meet needs. Some tool API (Application Programming Interface) calls can also be added into the action list for execution, which means the action list is the interface of DiagGPT and can provide many plugins to enrich the functions of this system. This also means that it can expand task-oriented dialogue to task-oriented dialogue by triggering the API of certain tasks.

## Appendix B Implementation Details of DiagGPT

In the implementation of DiagGPT, we employed gpt-4 as the base LLM, leveraging its strong understanding and reasoning abilities to achieve ideal results. We set the decoding temperature of all LLMs in our AI system to 0 to ensure more stable task execution. We provide detailed prompts in our system. The main prompts of the agent are shown in Figure [10](https://arxiv.org/html/2308.08043v4#A4.F10 "Figure 10 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"), while Figure [11](https://arxiv.org/html/2308.08043v4#A4.F11 "Figure 11 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue") displays the prompts for all actions in the action list. In these prompts, {variable} in blue indicates that the slot needs to be filled with the corresponding variable text. Once filled, these prompts can be fed into the LLMs to generate responses. Some of these slots facilitate information interaction between agents in the system.

In the evaluation, we use gpt-4-0613 as the foundation model for DiagGPT and for ChatGPT (gpt-4). For ChatGPT (gpt-3.5-turbo), we select gpt-3.5-turbo-0613. The simulation of the user is also based on gpt-4-0613. For the LLM evaluation, we use gpt-4-turbo-2024-04-09. To simplify the experimental setup and facilitate easier comparisons, we use a simplified version of DiagGPT in the quantitative experiment. We reduce the complexity of the action list and optimized prompts of LLM to promote topic development as much as possible. However, in qualitative experiments, we evaluate the full version of DiagGPT, which showcases the complete capabilities of our system.

## Appendix C LLM-TOD Dataset

Figure 9: An example of data from the TOD-LLM dataset.

We construct a new dataset, LLM-TOD (task-oriented dialogue for large language models dataset). It is used to evaluate the performance of LLM-based task-oriented dialogue models quantitatively. Since DiagGPT relies on LLMs, and considering our limited resources, fine-tuning LLMs for this specific purpose of TOD is not feasible. Therefore, we are unable to utilize other datasets that are tailored for fine-tuning models.

The dataset comprises 20 data, each representing a different topic: clinical, restaurant, hotel, hospital, train, police, bus, attraction, airport, bar, library, museum, park, gym, cinema, office, barbershop, bakery, zoo, and bank. Each data entry is generated by LLMs and subsequently validated by humans. For every task-oriented dialogue in the dataset, the system is required to go through a checklist in order and interact with users to achieve the specified goal. One checklist includes six items of topic or task and need to be solved or discussed. We incorporate these variables into the prompt for LLMs, thereby enabling them to execute the TOD. An example from the TOD-LLM dataset is illustrated in Figure [9](https://arxiv.org/html/2308.08043v4#A3.F9 "Figure 9 ‣ Appendix C LLM-TOD Dataset ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue").

## Appendix D Prompt Design

The content of prompts for some LLMs in DiagGPT are shown in Figure [10](https://arxiv.org/html/2308.08043v4#A4.F10 "Figure 10 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue") and Figure [11](https://arxiv.org/html/2308.08043v4#A4.F11 "Figure 11 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue"). Figure [12](https://arxiv.org/html/2308.08043v4#A4.F12 "Figure 12 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue") also displays the prompts used for other LLM-based models in our experiments: the ChatGPT baselines, UserGPT (the user simulator), and EvalGPT (used for LLM evaluation). Grading mode involves assigning a score to the quality of responses, while comparison mode requires selecting a winner based on response quality in a comparison. The criteria for evaluating response quality from both human and LLM perspectives include:

*   •

    Understanding How well does the system grasp user requests?

*   •

    Relevance Are the responses directly applicable to the user’s needs?

*   •

    Complex Handling Can the system effectively manage multifaceted queries?

*   •

    Efficiency How quickly does the system lead the conversation to a resolution?

*   •

    Experience What is the overall ease and satisfaction of the interaction?

*   •

    Comprehensiveness Does the system’s response provide all the relevant information to fully cover the user’s question?

*   •

    Detail Does the response include sufficient detail and depth to fully address the intricacies and subtleties of the user’s inquiry?

*   •

    Sufficiency Are all aspects and potential implications of the inquiry explored and explained, ensuring that the user receives a thorough understanding of the subject matter?

Figure 10: The prompts of Chat Agent, Topic Manager, Topic Enricher. These instructions can be modified to suit other scenarios.

Figure 11: The prompts for different actions are used to define specific program functions that correspond to their respective actions and instruct when to execute them.

Figure 12: The prompts of other LLMs in our experiments.