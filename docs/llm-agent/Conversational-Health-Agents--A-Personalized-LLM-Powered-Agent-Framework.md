<!--yml
category: 未分类
date: 2025-01-11 13:05:13
-->

# Conversational Health Agents: A Personalized LLM-Powered Agent Framework

> 来源：[https://arxiv.org/html/2310.02374/](https://arxiv.org/html/2310.02374/)

Mahyar Abbasian, M.Sc.^(1*), Iman Azimi, Ph.D.¹,
Amir M. Rahmani, Ph.D.¹, and Ramesh Jain, Ph.D.¹  

¹University of California, Irvine  
^*Corresponding author, abbasiam@uci.edu

## Abstract

Conversational Health Agents (CHAs) are interactive systems that provide healthcare services, such as assistance and diagnosis. Current CHAs, especially those utilizing Large Language Models (LLMs), primarily focus on conversation aspects. However, they offer limited agent capabilities, specifically lacking multi-step problem-solving, personalized conversations, and multimodal data analysis. Our aim is to overcome these limitations. We propose openCHA, an open-source LLM-powered framework, to empower conversational agents to generate a personalized response for users’ healthcare queries. This framework enables developers to integrate external sources including data sources, knowledge bases, and analysis models, into their LLM-based solutions. openCHA includes an orchestrator to plan and execute actions for gathering information from external sources, essential for formulating responses to user inquiries. It facilitates knowledge acquisition, problem-solving capabilities, multilingual and multimodal conversations, and fosters interaction with various AI platforms. We illustrate the framework’s proficiency in handling complex healthcare tasks via two demonstrations and four use cases. Moreover, we release openCHA as open source available to the community via GitHub^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://github.com/Institute4FutureHealth/CHA](https://github.com/Institute4FutureHealth/CHA)^,^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://opencha.com](https://opencha.com).

## 0emIntroduction

Artificial intelligence (AI), particularly large language model (LLM)-based conversational systems, has attracted immense global attention in recent years. These systems have revolutionized the field by enabling unprecedented access to and interaction with vast amounts of textual information. LLMs can aggregate and process comprehensive or focused segments of textual knowledge existing online, delivering contextually relevant, goal-oriented, and interactive access to this knowledge for anyone who needs it. The advent of LLMs has transformed early, simple conversation systems like Alexa and Siri, demonstrating significant effectiveness across diverse domains [[1](https://arxiv.org/html/2310.02374v5#bib.bib1), [2](https://arxiv.org/html/2310.02374v5#bib.bib2), [3](https://arxiv.org/html/2310.02374v5#bib.bib3)]. Conversational systems can now engage in open-ended conversations and provide relevant, contextual information in a more natural and engaging way.

While the field of AI has long explored intelligent agents, their focus has primarily been on analyzing the environment and making decisions based on gathered information. Early AI research often concentrated on physical world problems, fueled by advancements in computer vision, audio processing, and other areas of multimodal perceptual understanding. However, in dynamic environments like health management, where personalized and constantly evolving human health states are crucial, intelligent agents need to accurately capture these states through various means, including conversational interactions and access to personal user data. This information needs to be collected and analyzed, leveraging the vast knowledge gathered through research and practitioners’ experience.

Conversational Health Agents (CHAs) hold significant potential to address the challenges of dynamic health management environments. Thanks to the emergence of LLMs, CHAs can now understand user interactions through multimodal conversations, encompassing text, speech, and potentially other modalities. By analyzing these interactions, CHAs need to identify the necessary data, information, computational processes, and knowledge sources required to comprehend the user’s evolving health state. This information is then translated into actionable insights that effectively guide healthcare management. In essence, CHAs should combine the power of LLM-based conversions with agents’ capabilities, leveraging external data and information sources to navigate the complexities of personalized health environments and provide customized support for users:

*   •

    Conversation is the fundamental mode of human interaction. Throughout the ages, conversations have consistently served as the primary source of knowledge and the catalyst for societal actions. Recently, numerous studies have substantiated the efficacy, usability, and overall satisfaction associated with the conversational aspect of CHAs [[4](https://arxiv.org/html/2310.02374v5#bib.bib4)]. In healthcare settings, empathy  [[5](https://arxiv.org/html/2310.02374v5#bib.bib5), [6](https://arxiv.org/html/2310.02374v5#bib.bib6)] and companionship  [[7](https://arxiv.org/html/2310.02374v5#bib.bib7), [8](https://arxiv.org/html/2310.02374v5#bib.bib8)] necessitate personalized conversations.

*   •

    Agents should be furnished with conversational tools, interfaces, computational capabilities, and access to external resources to enhance the quality of healthcare delivery [[9](https://arxiv.org/html/2310.02374v5#bib.bib9)]. Given the intricacies of the healthcare domain [[10](https://arxiv.org/html/2310.02374v5#bib.bib10), [11](https://arxiv.org/html/2310.02374v5#bib.bib11)], agents should understand users’ queries, decompose them into the components of knowledge possession, perform health data access and analysis, and apply reasoning to respond effectively to specific situations [[11](https://arxiv.org/html/2310.02374v5#bib.bib11)]. Their adaptability is essential, aligning their evolution with advancements in healthcare technology and literature [[10](https://arxiv.org/html/2310.02374v5#bib.bib10)]. Therefore, agents empower the abilities for personalization, multimodality, and continuous up-to-dateness  [[6](https://arxiv.org/html/2310.02374v5#bib.bib6)].

Our current exploration centers on the development of CHAs using the latest technological developments in AI, LLMs, and mHealth, where it has shown efficacy in the continuous collection of lifestyle and physiological data from users. Figure [1](https://arxiv.org/html/2310.02374v5#S1.F1 "Figure 1 ‣ 0emIntroduction ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") shows an overview of the CHA main components. These indispensable components stand poised to facilitate the creation of exceptionally efficient CHAs.

![Refer to caption](img/d1ccc2e304f16be2fbe8a7628c9570c9.png)

Figure 1: A conversational health agent including 1) a conversation component to enable user interaction and 2) a health agent for problem-solving and determining the optimal sequences of actions, leveraging health information.

Existing LLMs, such as ChatGPT [[1](https://arxiv.org/html/2310.02374v5#bib.bib1)], BioGPT [[12](https://arxiv.org/html/2310.02374v5#bib.bib12)], ChatDoctor [[13](https://arxiv.org/html/2310.02374v5#bib.bib13)], and Med-PaLM [[3](https://arxiv.org/html/2310.02374v5#bib.bib3)], are currently active in the medical knowledge domain. These LLMs can be served as CHAs [[14](https://arxiv.org/html/2310.02374v5#bib.bib14), [15](https://arxiv.org/html/2310.02374v5#bib.bib15), [9](https://arxiv.org/html/2310.02374v5#bib.bib9), [16](https://arxiv.org/html/2310.02374v5#bib.bib16), [17](https://arxiv.org/html/2310.02374v5#bib.bib17)]. However, they merely focus on the conversational aspects, offering limited agent capabilities such as basic text-based chat interfaces and lacking multi-step problem-solving capabilities. They lack access to users’ personally collected longitudinal data and electronic health records (EHRs), which include crucial information like vital signs, biosignals (e.g., electrocardiogram), medical images, and demographic data. Consequently, their responses tend to be generic and may not address individual health circumstances adequately. Moreover, they struggle to incorporate the latest health insights, leading to potentially outdated responses [[18](https://arxiv.org/html/2310.02374v5#bib.bib18)]. Furthermore, these chatbots do not seamlessly integrate with established and existing AI models and tools [[19](https://arxiv.org/html/2310.02374v5#bib.bib19)] for multimodal predictive modeling, rendering previous healthcare efforts obsolete.

In light of the significant advancements in technology and its paramount importance for both humanity and the environment, it becomes imperative that we synergize all available tools and harness knowledge from diverse sources to craft CHAs that offer a trustworthy, understandable, and actionable environment for a global audience. Presently, we stand on the threshold of crafting frameworks capable of delivering information in the most user-friendly and culturally attuned manner possible. This paper aims to introduce an initial iteration of such agents and lay the foundation for developing more sophisticated tools as our journey unfolds.

## 0emBackground and Significance

### Related Work

Efforts in developing LLM-based CHAs can be categorized into three main groups: LLM Chatbots, Specialized Health LLMs, and Multimodal Health LLMs. LLM Chatbots employ and evaluate current chatbots (e.g., ChatGPT) in executing distinct healthcare functions [[20](https://arxiv.org/html/2310.02374v5#bib.bib20), [21](https://arxiv.org/html/2310.02374v5#bib.bib21), [22](https://arxiv.org/html/2310.02374v5#bib.bib22), [23](https://arxiv.org/html/2310.02374v5#bib.bib23), [24](https://arxiv.org/html/2310.02374v5#bib.bib24)]. For instance, Chen et al. [[23](https://arxiv.org/html/2310.02374v5#bib.bib23)] examined ChatGPT’s efficacy in furnishing dependable insights on cancer treatment-related inquiries.

Specialized Health LLMs delved deeper into the fundamental aspects of LLMs, aiming to enhance conversational models’ performance by creating entirely new LLMs pretrained specifically for healthcare or fine-tuning existing models. Notable examples include initiatives such as ChatDoctor [[13](https://arxiv.org/html/2310.02374v5#bib.bib13)], MedAlpaca [[2](https://arxiv.org/html/2310.02374v5#bib.bib2)], and BioGPT [[12](https://arxiv.org/html/2310.02374v5#bib.bib12)]. This category emerged in response to research indicating that general-domain LLMs often struggle with healthcare-specific tasks due to domain shift [[25](https://arxiv.org/html/2310.02374v5#bib.bib25), [26](https://arxiv.org/html/2310.02374v5#bib.bib26)], and relying solely on prompt engineering may not significantly improve their healthcare-specific performance [[27](https://arxiv.org/html/2310.02374v5#bib.bib27), [28](https://arxiv.org/html/2310.02374v5#bib.bib28)].

Multimodal Health LLMs involve a novel trajectory by integrating multimodality into LLMs for diagnostic functions. For instance, Tu et al. [[3](https://arxiv.org/html/2310.02374v5#bib.bib3)] investigated the potential of foundational transformer concepts in LLMs to amalgamate diverse modalities—videos, images, signals, and text—culminating in a multimodal generative model. Xu et al. [[29](https://arxiv.org/html/2310.02374v5#bib.bib29)] introduced an LLM-aligned multimodal model, coupling chest X-ray images with radiology reports for X-ray-related tasks. Similarly, Belyaeva et al. [[30](https://arxiv.org/html/2310.02374v5#bib.bib30)] incorporated tabular health data into LLMs, yielding multimodal healthcare capabilities.

### Existing Research Gaps and Challenges

Knowledge-groundedness and personalization in CHAs require tailored interactions that transcend basic dialogues, ensuring inclusivity through versatile, multimodal multilingual interfaces. The goal is to create CHAs that not only excel in conversational skills but also exhibit agent capabilities, enabling them to engage in critical thinking and strategic planning as proficient problem solvers. Despite the great efforts in developing CHAs, the existing services and models suffer from the following limitations:

1.  i)

    Insufficient support for comprehensive personalization, particularly in cases necessitating real-time access to individualized data. A substantial portion of users’ healthcare data, primarily images, time-series, tabular data, and all other users’ measured personal data streams is housed within healthcare platforms. Currently, CHAs have limited access to this data, primarily during the training and fine-tuning phases of LLM development, or they are completely severed from user data thereafter. The absence of accurate user healthcare information – including continuous data from wearable devices, mHealth applications, and similar sources – hampers the performance of these agents, confining their capabilities to furnish generic responses, offer general guidelines, or potentially provide inaccurate answers.

2.  ii)

    Limited capacity to access up-to-date knowledge and retrieve the most recent healthcare knowledge base. Conventional LLMs depend on limited data and Internet-derived knowledge during their training phase, leading to three primary challenges. They tend to exhibit biases favoring populations with the most abundant online content, underscoring the importance of accessing the latest, relevant data. Recently introduced LLM-based services (e.g., ChatGPT4 [[1](https://arxiv.org/html/2310.02374v5#bib.bib1)]) offer Internet search, but this is still insufficient for healthcare applications due to the large number of websites propagating false information. They lack updates on newly reliable Internet resources, a critical shortcoming in healthcare where novel, reliable, and evaluated treatments and modifications to previous recommendations are frequently ignored. Lastly, their reliance on outdated or less pertinent data makes identifying instances of hallucination [[31](https://arxiv.org/html/2310.02374v5#bib.bib31)] problematic. Lack of up-to-date information reduces the trustworthiness and credibility of generated responses [[18](https://arxiv.org/html/2310.02374v5#bib.bib18)].

3.  iii)

    Lack of seamless integration with established, multimodal data analysis tools and predictive models that require external execution. Current agents often overestimate the computational capabilities of generative AI, leading to an under-utilization of well-established healthcare analysis tools, despite their proficiency in managing diverse data types [[19](https://arxiv.org/html/2310.02374v5#bib.bib19)].

4.  iv)

    Lack of multi-step problem-solving capabilities. Existing LLM-based CHAs are typically specialized for specific tasks or deficient in robust data analysis capabilities. For example, Xu et al. [[29](https://arxiv.org/html/2310.02374v5#bib.bib29)] model performs X-ray image reporting relying solely on X-ray images, ignoring other modalities, such as vital signs recorded in time-series format. Additionally, the existing CHAs cannot address intricate sequential tasks (i.e., act as problem solvers). Incorporating LLMs into CHAs requires integrating sequential reasoning, personalized health history analysis, and data fusion.

LLMs solely are insufficient to tackle the previously mentioned challenges. To make them practical for real-world applications, we need a comprehensive framework that harnesses LLMs while integrating various auxiliary components and external resources.

### Key Contributions

In this article, we present a holistic LLM-powered framework for the development of CHA, aiming to rectify the limitations mentioned above. We delineate the components of our framework and proceed with a case study demonstrating our agent’s capacity. The framework is a problem solver that provides personalized responses by utilizing contemporary Internet resources and advanced multimodal healthcare analysis tools, including machine learning methods. Significant contributions are as follows.

1.  1.

    We propose an LLM-powered Orchestrator, acting as a problem solver, to address healthcare-related queries by analyzing input queries, gathering the required information, performing actions, and offering personalized responses.

2.  2.

    We introduce external healthcare data sources, knowledge bases, and AI and Analysis models critical for enabling CHAs to offer reliable, trustworthy, and up-to-date responses.

3.  3.

    We incorporate multimodal and multilingual capabilities into the framework, increasing usability.

4.  4.

    We show the framework’s effectiveness using two demos and four use cases.

5.  5.

    We release the CHA framework as open source on GitHub^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$ [https://github.com/Institute4FutureHealth/CHA](https://github.com/Institute4FutureHealth/CHA) along with detailed documentation^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$ [https://docs.opencha.com/](https://docs.opencha.com/), inviting the community to leverage and integrate it into their solutions.

## 0emMaterial and Methods

![Refer to caption](img/6029ff8c04d716115d7eeb1dc3940a98.png)

Figure 2: An overview of the proposed LLM-powered framework leveraging a service-based architecture

We design an LLM-powered framework with a central agent that perceives and analyzes user queries, provides appropriate responses, and manages access to external resources through Application Program Interfaces (APIs) or function calls. The user-framework interaction is bidirectional, ensuring a conversational tone for ongoing and follow-up conversations. Figure [2](https://arxiv.org/html/2310.02374v5#S3.F2 "Figure 2 ‣ 0emMaterial and Methods ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") shows an overview of the framework, including three major components: Interface, Orchestrator, and External Sources.

### Interface

Interface acts as a bridge between the users and agents, including interactive tools accessible through mobile, desktop, or web applications. It integrates multimodal communication channels, such as text and audio. The Interface receives users’ queries and subsequently transmits them to the Orchestrator (see Figure [2](https://arxiv.org/html/2310.02374v5#S3.F2 "Figure 2 ‣ 0emMaterial and Methods ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")).

Within this framework, users can provide metadata (alongside their queries), including images, audio, gestures, and more. For instance, a user could capture an image of their meal and inquire about its nutritional values or calorie content, with the image serving as metadata.

### Orchestrator

The Orchestrator is the openCHA agent core, which is responsible for problem-solving, planning, executing actions, and providing an appropriate response based on the user query. It incorporates the concept of the Perceptual Cycle Model [[32](https://arxiv.org/html/2310.02374v5#bib.bib32)] in openCHA, allowing it to perceive, transform, and analyze the world (i.e., input query and metadata) to generate appropriate responses. To this end, the input data are aggregated, transformed into structured data, and then analyzed to plan and execute actions. Through this process, the Orchestrator interacts with external sources to acquire the required information, perform data integration and analysis, and extract insights, among other functions. In the following, we outline five major components of the Orchestrator.

The Task Planner is the LLM-enabled decision-making, planning, and reasoning core of the Orchestrator. Its primary responsibility is gathering all necessary information to answer users’ queries. To achieve this, it interprets the user’s query and metadata, identifying the necessary steps for task execution.

To transform a user query into a sequence of tasks, we incorporate the Tree of Thought [[33](https://arxiv.org/html/2310.02374v5#bib.bib33)] prompting methods into the Task Planner. Using this prompting method, the LLM is asked to 1) generate three unique strategies (i.e., sequences of tasks to be called with their inputs), 2) describe the pros and cons of each strategy, and 3) select one as the best strategy. An alternative prompting technique incorporated in openCHA is ReAct [[34](https://arxiv.org/html/2310.02374v5#bib.bib34)], which employs reasoning and action techniques to ascertain the essential tasks to be executed. openCHA offers users the flexibility to choose the prompting method that best meets the needs of their application. Other prompting techniques, such as Plan-and-Solve Prompting [[35](https://arxiv.org/html/2310.02374v5#bib.bib35)], could also be implemented and integrated as a Task Planner.

We outline the process of creating and integrating a task into openCHA in Appendix 1\. We also indicate how the task is converted into an appropriate prompt, enabling the Task Planner to recognize the available tasks and how to invoke them. Appendix 2 provides a detailed examination of the Task Planner’s implementation, utilizing the Tree of Thought prompting method.

In the proposed Orchestrator, the planning part is performed in English, leveraging the superior capabilities of LLMs in this language. The framework can employ one of two distinct approaches if the query is in a language other than English. The first approach retrains the source language and utilizes the language model capabilities in that language to generate responses. The second approach involves translating the query into English (e.g., using Google Translate), planning and executing the process in English, and translating the final answer back into the source language.

The Task Executor carries out actuation within the Orchestrator by following the planning and task execution steps determined by the Task Planner. The Task Executor has two primary responsibilities. First, it acts as a data converter, converting the input query and metadata and preparing it to be used by the Task Planner. For instance, if the question is in a language other than English, it will be translated into English using the Google Translate service [[36](https://arxiv.org/html/2310.02374v5#bib.bib36)]. Furthermore, if the metadata contains files or images, Task Executor sends the metadata details to Task Planner for planning. Second, the Task Executor executes tasks generated by the Task Planner through interactions with external sources. The results are then relayed to the Task Planner to continue planning if needed. In the end, the Task Planner signals the end of the planning. In Appendix 2, we detail how the Task Planner translates planned tasks into execution instructions, enabling the Task Executor to properly carry out the tasks.

It is crucial to emphasize that communication between the task planner and task executor is bidirectional. An iterative process continues between the Task Executor and Task Planner until the Task Planner accumulates sufficient information to respond appropriately to the user’s inquiry. This two-way exchange proves indispensable because, in specific scenarios, the Task Planner may necessitate intermediate information to determine subsequent actions.

The Data Pipe is a repository of metadata and data acquired from External Sources through the execution of conversational sessions. This component is essential because numerous multimodal analyses involve intermediate stages, and their associated data must be retained for future retrieval. The intermediate data might be large, surpassing token limits, or challenging to comprehend and utilize by the Task Planner’s or Response Generator’s LLM. The Data Pipe is automatically managed by the Task Executor. It monitors the stored metadata and intermediate data.

The Data Pipe in openCHA can range from a simple in-memory key/value storage for intermediate data to a more complex database system. The proposed framework allows developers to determine whether their tasks’ results are intermediate or should be directly returned to the LLM. Appendix 1 details how developers can configure this setting.

Additionally, we have implemented a mechanism whereby an intermediate result stored in the Data Pipe generates a unique key as the task’s outcome. This key is then provided in the Task Planner prompt, aiding the Task Planner in recognizing and utilizing this data as necessary. Appendix 4 illustrates sample prompts generated for tasks and demonstrates how the Task Planner employs the Data Pipe key.

The Promptist is responsible for transforming query text or outcomes from External Sources into suitable prompts that can be supplied to either the Task Planner or the Response Generator. The Promptist provides the flexibility to modify and adapt each technique, allowing for seamless integration and customization. It can be implemented using existing prompting techniques, some of which are listed as follows.

LLM-REC, proposed by Lyu et al. [[37](https://arxiv.org/html/2310.02374v5#bib.bib37)], employs four unique prompting strategies to enrich text descriptions, enhancing personalized text-based recommendations. The approach leverages the LLM to understand item characteristics, significantly improving recommendation quality. Additionally, the Hao et al. [[38](https://arxiv.org/html/2310.02374v5#bib.bib38)] method can be leveraged, which optimizes text-to-image prompt generation through a framework called prompt adaptation. It automatically refines user inputs into model-preferred prompts. This process starts with supervised fine-tuning of a pretrained language model using a curated set of prompts. It then employs reinforcement learning, guided by a reward function, to identify more effective prompts that produce aesthetically pleasing images aligned with user intentions. Furthermore, the instructions provided by OpenAI on creating more effective prompts can be used [[39](https://arxiv.org/html/2310.02374v5#bib.bib39)].

The Response Generator is an LLM-based module responsible for preparing the response. It refines the gathered information by the Task Planner, converting it into an understandable format and inferring the appropriate response. We separate the Response Generator and Task Planner to allow flexibility in choosing diverse LLM models and prompting techniques for these components. This division ensures that the Task Planner focuses solely on planning without responding to users, while the Response Generator utilizes gathered information to deliver conclusive responses. This segregation facilitates the Response Generator in addressing aspects of empathy and companionship in conversations. In contrast, the Task Planner primarily handles personalization and the up-to-dateness of conversations. Appendix 3 outlines the implementation of the Response Generator and how it utilizes results collected by the Task Planner to respond to the user effectively.

### External Sources

External Sources play a pivotal role in obtaining essential information from the broader world. Typically, these External Sources furnish application program interfaces (APIs) that the Orchestrator can use to retrieve required data, process them using AI or analysis tools, and extract meaningful health information. In openCHA, we integrate with four primary external sources, which we found critical for CHAs (see Figure [2](https://arxiv.org/html/2310.02374v5#S3.F2 "Figure 2 ‣ 0emMaterial and Methods ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")).

Healthcare Data Source enables the collection, ingestion, and integration of data captured from a variety of sources, such as Electronic Health Record (EHR), smartphones, and smartwatches, for healthcare purposes [[40](https://arxiv.org/html/2310.02374v5#bib.bib40)]. Examples of data sources are mHealth platforms and healthcare databases. mHealth platforms have garnered significant attention in the recent wave of healthcare digitalization, enabling ubiquitous health monitoring [[41](https://arxiv.org/html/2310.02374v5#bib.bib41), [42](https://arxiv.org/html/2310.02374v5#bib.bib42)]. The data encompass various modalities, including biosignals (e.g., PPG collected via a smartwatch), images (e.g., captured via user’s smartphone), videos, tabular data (e.g., demographic data gathered from EHR), and more. Notable examples of such healthcare platforms include ZotCare [[43](https://arxiv.org/html/2310.02374v5#bib.bib43)] and ilumivu [[44](https://arxiv.org/html/2310.02374v5#bib.bib44)], offering APIs for third-party integration. In our context, the Orchestrator functions as a third party, accessing user data with their consent.

Knowledge Base fetches the most current and pertinent data from healthcare sources, such as healthcare literature, reputable websites, or knowledge graphs using search engines or retrieval models [[45](https://arxiv.org/html/2310.02374v5#bib.bib45), [46](https://arxiv.org/html/2310.02374v5#bib.bib46), [47](https://arxiv.org/html/2310.02374v5#bib.bib47), [48](https://arxiv.org/html/2310.02374v5#bib.bib48)]. Accessing this retrieved information equips CHAs with up-to-date, personalized knowledge, enhancing its trustworthiness while reducing hallucination and bias. openCHA allows the integration of various knowledge bases to be defined and configured as tasks.

AI and Analysis Models provide data analytics tools to extract information, associations, and insights from data [[49](https://arxiv.org/html/2310.02374v5#bib.bib49), [19](https://arxiv.org/html/2310.02374v5#bib.bib19)], playing a crucial role in the evolving landscape of LLM-healthcare integration, enhancing trustworthiness and personalization. They can perform various tasks, including data denoising, abstraction, classification, and event detection, to mention a few [[49](https://arxiv.org/html/2310.02374v5#bib.bib49), [19](https://arxiv.org/html/2310.02374v5#bib.bib19)]. As generative models, LLMs cannot effectively perform extensive computations or act as machine learning inferences on data. The AI platforms empower our framework to leverage existing health data analytic approaches.

Translators effectively convert various languages into widely spoken languages, such as English, thereby enhancing the accessibility and inclusivity of CHAs. Existing agents face limitations that hinder their usability for large communities globally. Universal text literacy for CHAs often narrows their reach and positions them as a privilege [[50](https://arxiv.org/html/2310.02374v5#bib.bib50), [51](https://arxiv.org/html/2310.02374v5#bib.bib51), [52](https://arxiv.org/html/2310.02374v5#bib.bib52)]. Many underserved communities face obstacles while using CHAs due to their educational disparities, financial constraints, and biases that favor developed nations within existing technological paradigms. Our framework integrates with Translator platforms and is designed to accommodate and support communication with diverse communities. This integration enhances the overall usability of CHAs.

The selection of the external sources is based on the information and knowledge they provide and their interaction with the Orchestrator. The Orchestrator handles various data types in the proposed framework, including text, JSON formatted data, or unstructured data such as images and audio. This design ensures that any external source capable of returning results in these formats is supported.

## 0emDemonstration

We demonstrate the capabilities of openCHA through three distinct demos. These demonstrations highlight how LLMs’ planning and reasoning abilities can effectively comprehend user queries and translate them into appropriate task executions. Each demonstration involves linking a set of implemented tasks to openCHA. Subsequently, we highlight openCHA’s planning proficiency in effectively sequencing tasks in the correct order with appropriate inputs and executing the tasks. After executing all necessary tasks, the results are conveyed to the Response Generator to provide the final response. The overview of the implemented tasks and their utilization in each demo is depicted in Figure [3](https://arxiv.org/html/2310.02374v5#S4.F3 "Figure 3 ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework").

![Refer to caption](img/531e1798118f1ab57314e5d23460a5b0.png)

Figure 3: Overview of the implemented tasks and components and how they are used in the two demos.

### Demo 1: Patient health record reporting

The first demo indicates how openCHA interacts with patient data stored in a database, conducts data analysis, generates health reports upon request, and asks follow-up questions. Examples of the user’s questions could be: "Provide a sleep summary of Patient 5 during August 2020", "Is Patient 5 REM sleep enough during August 2020?", "How much is the total step count of Patient 5 during August 2020?" or "Provide an activity summary of Patient 5 during 2020". For this demo, we implement two tasks for retrieving sleep and physical activity from a health monitoring dataset [[53](https://arxiv.org/html/2310.02374v5#bib.bib53)]. The data utilized in this demo is a part of an extensive longitudinal study focusing on the mental health of college students, as documented in [[53](https://arxiv.org/html/2310.02374v5#bib.bib53)]. Moreover, we develop analytical tasks capable of executing basic statistical analysis (e.g., computing trends and averaging). Finally, we also add Google Search and Extract Text tasks so that the current setup of openCHA can access internet information. The involved tasks in this demo are marked by red dots in Figure [3](https://arxiv.org/html/2310.02374v5#S4.F3 "Figure 3 ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework").

Figure [4](https://arxiv.org/html/2310.02374v5#S4.F4 "Figure 4 ‣ Demo 1: Patient health record reporting ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") illustrates sample queries and the corresponding responses from openCHA. The Task Planner identifies the correct tasks, their execution order, and the appropriate inputs to be used. Table [1](https://arxiv.org/html/2310.02374v5#S4.T1 "Table 1 ‣ Demo 1: Patient health record reporting ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") showcases the strategy chosen by the Task Planner using the Tree of Thought prompting technique.

![Refer to caption](img/8d7ca21e2a1a1f401f84b3a484693c1c.png)![Refer to caption](img/f74d2574b431515547a39f37a8e18187.png)![Refer to caption](img/b7cd2d8149fe628146c9fb451c41070f.png)

Figure 4: Demo 1\. Patient health record reporting and follow up.

Table 1: The strategies suggested by Tree of Thought prompting technique for the first question.

 |  | I will proceed to directly analyze the REM sleep data of |
|  | Patient 5 for August 2020, providing a precise and specific |
|  | conclusion. |
| Decision | Execution: |
|  | 1\. Use the sleep_get tool to obtain the REM sleep data for |
|  | Patient 5 in August 2020. |
|  | 2\. Utilize the sleep_analysis tool to analyze the REM sleep |
|  | duration and efficiency for August 2020. | 

### Demo 2: Objective stress level estimation with multilanguage interaction

Demo 2 indicates the openCHA’s capability to conduct signal processing and objective stress level estimation. We also show that openCHA can answer the query in multiple languages. This is achieved by interacting with a translator, health data sources, and AI models. Examples of interactions include inquiries such as "Retrieve the stress level of Patient 5 on August 29th, 2020" and "What is the average heart rate of Patient 5 during August 2020?" To fulfill our objective, we implemented three distinct tasks (yellow dots in Figure [3](https://arxiv.org/html/2310.02374v5#S4.F3 "Figure 3 ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")). The first task involved acquiring Photoplethysmogram (PPG) data from the patients. PPG data were gathered using Samsung Gear Sport smartwatches [[54](https://arxiv.org/html/2310.02374v5#bib.bib54)], with a sampling frequency of 20 Hz, while participants were in free-living conditions. The data is part of the [[53](https://arxiv.org/html/2310.02374v5#bib.bib53)]. The second task performs PPG signal processing to extract heart rate variability (HRV) metrics. For this purpose, we utilize the Neurokit [[55](https://arxiv.org/html/2310.02374v5#bib.bib55)] Python library. In our case study, we extract a total of 32 HRV parameters, including metrics such as the root mean square of successive differences between normal heartbeats (RMSSD), low-frequency (LF), and high-frequency (HF) values [[56](https://arxiv.org/html/2310.02374v5#bib.bib56), [57](https://arxiv.org/html/2310.02374v5#bib.bib57)]. The third task estimates stress levels based on HRV using an AI model. Initially, we employed an autoencoder to reduce the dataset’s 32 HRV features to 12\. Subsequently, a four-layer neural network categorizes the 12 features into five stress levels. The evaluation of the stress estimation model demonstrates an 86% accuracy rate on a test set.

Figure [5](https://arxiv.org/html/2310.02374v5#S4.F5 "Figure 5 ‣ Demo 2: Objective stress level estimation with multilanguage interaction ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") depicts example queries and the corresponding responses from openCHA. The Task Planner’s approach involves initially retrieving the PPG data of Patient 5 on August 29th, 2020\. Subsequently, the obtained result is forwarded to the PPG analysis task to extract HRV metrics. Lastly, the planner initiates the execution of stress analysis tasks, providing the HRV metrics for this task. Table [2](https://arxiv.org/html/2310.02374v5#S4.T2 "Table 2 ‣ Demo 2: Objective stress level estimation with multilanguage interaction ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") displays the chosen strategy by the Task Planner utilizing the Tree of Thought prompting technique. The ultimate estimated stress, along with an explanation, is then returned to the user.

![Refer to caption](img/d393e2a0c486ea3a8cce73874f2dcd5f.png)![Refer to caption](img/91de8abcfa8ef472bffa7c288d3592bd.png)

Figure 5: Demo 2\. Objective stress level estimation. The question is asked in English and Spanish.

Table 2: The strategies suggested by Tree of Thought prompting technique for the first question.

 |  | The best strategy provides both detailed PPG analysis and an |
|  | estimation of the stress level, which offers a comprehensive view |
|  | of the patient’s health status. |
|  | Execution: |
| Decision | 1\. Use ppg_get tool to retrieve the PPG data for Patient 5 |
|  | during August 29th, 2020. |
|  | 2\. Analyze the PPG data with ppg_analysis tool to obtain the |
|  | heart rate. |
|  | 3\. Use stress_analysis to estimate the stress level based on the |
|  | obtained PPG data. | 

### openCHA Use cases

To indicate the usability of openCHA across various applications, we outline several use cases that have utilized the framework in their research as follows.

1.  1.

    ChatDiet [[58](https://arxiv.org/html/2310.02374v5#bib.bib58)] introduced a personalized, nutrition-oriented food recommendation agent, utilizing openCHA as its core implementation. By integrating personal and population models as external sources, ChatDiet offered tailored food suggestions. It enhanced traditional food recommendation services by delivering dynamic, personalized, and explainable recommendations. In a case study, ChatDiet achieved an effectiveness rate of 92%, outperforming solutions like ChatGPT.

2.  2.

    Knowledge-infused LLM-powered CHA for diabetic patients [[59](https://arxiv.org/html/2310.02374v5#bib.bib59)] is developed by integrating domain-specific knowledge and analytical tools as external sources using openCHA. This integration included incorporating American Diabetes Association dietary guidelines and deploying analytical tools for nutritional intake calculation, resulting in superior performance compared to GPT4 in managing diabetes through tailored dietary recommendations, as demonstrated by an evaluation of 100 diabetes-related questions.

3.  3.

    openCHA was employed to develop an agent for evaluating the safety and reliability of mental health chatbots [[60](https://arxiv.org/html/2310.02374v5#bib.bib60)]. This agent’s evaluation capabilities were compared with expert assessments and several existing LLMs, including GPT4, Claude, Gemini, and Mistral. Guidelines and benchmarks introduced by experts and Internet search served as external sources linked to openCHA. The agent demonstrated superior accuracy, achieving the lowest mean absolute error (MAE) against experts’ scores — a reduction by a factor of 1 compared to LLMs’ scores, with the maximum MAE being 10 — and provided unbiased evaluation scores.

4.  4.

    The Empathy-enhanced CHA [[61](https://arxiv.org/html/2310.02374v5#bib.bib61)] was developed to interpret and respond to users’ emotional states through multimodal dialogue, representing a significant step forward in providing contextually aware and empathetically resonant support in the mental health field. This paper utilized speech-to-text, text-to-speech, and speech emotion detection models as external sources connected to openCHA.

To see more demonstrations on how the openCHA works in real setup, we have uploaded multiple YouTube videos^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=w48sPlF5zhs](https://www.youtube.com/watch?v=w48sPlF5zhs)^,  ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=PWxL_OgWGfE&t=3s](https://www.youtube.com/watch?v=PWxL_OgWGfE&t=3s)^,  ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=c-7IEBaRSyQ](https://www.youtube.com/watch?v=c-7IEBaRSyQ)^,  ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=rHXpk_P5n6Y](https://www.youtube.com/watch?v=rHXpk_P5n6Y)

## 0emDiscussion

### openCHA Potentials and Limitations

In this section, we briefly discuss our proposed framework’s capabilities, potentials, and limitations.

Flexibility: openCHA provides a high level of flexibility to integrate LLMs with external data sources, knowledge bases, and analytical tools. The proposed components can be developed and replaced according to the requirements of the healthcare application in question. For instance, new external sources can be effortlessly integrated and introduced as new tasks into openCHA. The LLMs employed in openCHA can be readily swapped with fine-tuned or more healthcare-specific LLMs. Similarly, the Planner prompting technique and decision-making processes are modifiable. This flexibility facilitates collaboration among diverse research communities, enabling them to contribute to various aspects of CHAs. Appendix 1 shows how a new task can be defined and introduced into openCHA.

Explainability: openCHA enhances explainability for CHAs, allowing users to inquire about the tools and actions used to generate a response. As detailed in Appendices 2 and 3, openCHA maintains a "previous actions" section that records past conversations and tasks. When queried about task usage, it lists the executed tasks and their applications, enhancing transparency and fostering trust between users and CHAs. For instance, in Demo 2, when a user asks, "Name the tasks used," openCHA responds by detailing that PPG and HRV data were utilized to determine stress levels. An example of this interaction is shown in Figure [6](https://arxiv.org/html/2310.02374v5#S5.F6 "Figure 6 ‣ openCHA Potentials and Limitations ‣ 0emDiscussion ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework").

Personalization: The openCHA framework enhances personalization by integrating individual information and analytics tools from healthcare systems or local databases as external sources. The quality of these external sources greatly influences the effectiveness of the personalization. For example, ChatDiet [[58](https://arxiv.org/html/2310.02374v5#bib.bib58)] utilizes personal dietary preferences and population data, along with an analysis of nutrients’ effects on health outcomes like sleep quality, to enhance its food recommendations significantly. This strategy not only heightens the accuracy of the recommendations but also ensures they are precisely tailored to meet individual dietary needs.

Reliability: openCHA boosts the reliability of answers by leveraging validated information and computations as external sources. Our framework is tailored to effectively utilize existing LLMs for complex healthcare tasks, strategically offloading computational and sensitive information tasks to external sources while reserving LLMs primarily for reasoning and generating responses. For instance, the paper "Knowledge-infused LLM-powered CHA for diabetic patients" [[59](https://arxiv.org/html/2310.02374v5#bib.bib59)] demonstrates the benefits of integrating external knowledge to accurately determine nutritional values and align them with established guidelines, highlighting inaccuracies in nutritional estimations when solely relying on GPT4 LLM for data access and calculations.

![Refer to caption](img/fba59ba35ca9706e96c4efaa13d9806e.png)

Figure 6: A example screenshot indicating how openCHA improves explainability in conversations.

Latency: Utilizing multiple external sources offers benefits, but it can also affect the model’s response time, potentially leading to increased latency in the CHA. As the number of tasks and steps within the framework expands, there could be a rise in response time, which might diminish usability. Recent research by Singh et al. [[62](https://arxiv.org/html/2310.02374v5#bib.bib62)] explores new methods for executing tasks in parallel when they are not dependent on each other.

Toekn Limit: Token limits in LLMs present a challenge for accommodating tasks within the Task Planner. However, recent advancements [[63](https://arxiv.org/html/2310.02374v5#bib.bib63)] indicate progress in extending LLM token limits, which helps mitigate this issue.

Privacy and Security: Privacy and security are crucial in this framework, particularly for healthcare applications dealing with user privacy concerns. Strong privacy measures are essential to prevent unauthorized access, data breaches, and identity theft, with potentially severe consequences [[64](https://arxiv.org/html/2310.02374v5#bib.bib64), [65](https://arxiv.org/html/2310.02374v5#bib.bib65)]. To address privacy and security issues, external sources should provide data confidentiality. For example, this can be enabled by either granting limited access to CHAs as third parties with user permission or employing de-identification and anonymization techniques [[66](https://arxiv.org/html/2310.02374v5#bib.bib66)]. Another strategy is to prevent LLMs from using users’ provided data for training and fine-tuning, ensuring that such data is not stored.

### Study Limitations and Future Work

In this section, we outline the study limitations and future research directions.

Planning Robustness: Since we utilize LLMs for planning and response generation, there is still the inherent risk of biases or trustfulness issues. Our framework aims to enhance the robustness of planning by integrating external sources to reduce these problems, though it cannot ensure their complete elimination. To enhance the planning robustness, we will explore using Agentic design patterns like the self-consistency [[67](https://arxiv.org/html/2310.02374v5#bib.bib67)] method or new reasoning techniques.

Accuracy and Evaluation: Accuracy and evaluation in our framework hinge on the configuration choices made by researchers, such as the selected external sources, LLM, and planning technique. Since knowledge, data, and analytics are outsourced to external sources, the quality of these sources plays a crucial role in enhancing accuracy; better external sources increase the likelihood of achieving superior results.

Two distinct assessments are necessary to evaluate such systems. The first evaluates the accuracy of external sources, whether they are AI models or knowledge bases. The second assesses the overall configured and constructed CHA to determine if it behaves as expected. Several metrics and evaluation methods are recommended in [[68](https://arxiv.org/html/2310.02374v5#bib.bib68)], with additional evaluation techniques explored in [[58](https://arxiv.org/html/2310.02374v5#bib.bib58), [59](https://arxiv.org/html/2310.02374v5#bib.bib59), [60](https://arxiv.org/html/2310.02374v5#bib.bib60), [61](https://arxiv.org/html/2310.02374v5#bib.bib61)]. These evaluations are application-specific, and our framework provides extensive customization capabilities to suit different use cases and requirements. In our future work, we will explore more evaluation techniques.

User Query Ambiguity: Understanding user intentions presents a significant challenge due to query ambiguity, often caused by vague or incomplete information and a lack of necessary external sources connected to openCHA for the specific application. To enhance response accuracy, our future work involves refining openCHA’s ability to clarify user intentions. Key strategies include employing targeted follow-up questions, improving comprehension of the user’s situation, and the precision of responses.

Scalability: We plan to address scalability issues within our system to ensure it can effectively manage increasing loads and accommodate more complex scenarios. This involves enhancing system architecture and resource allocation to support larger data volumes and more simultaneous users without compromising performance. We plan to investigate further how good the openCHA core scalability is in real-world scenarios.

## 0emConclusion

This paper presented openCHA, an LLM-powered framework designed to empower CHAs in effectively addressing healthcare-related queries through the analysis of input questions, data collection, action execution, and the delivery of personalized responses. We demonstrated the framework’s effectiveness using two different demos on general patient health record reporting, objective stress level estimation, and further use cases. Additionally, we discussed the capabilities, limitations, and challenges of openCHA. Our future efforts will focus on improving planning robustness, examining accuracy and evaluation aspects, addressing user query ambiguity, and enhancing the scalability of openCHA.

## References

*   [1] OpenAI. ChatGPT: OpenAI’s Conversational AI Model; Accessed: January 2024. [https://openai.com/chatgpt](https://openai.com/chatgpt).
*   [2] Han T, Adams LC, Papaioannou JM, Grundmann P, Oberhauser T, Löser A, et al. MedAlpaca–An Open-Source Collection of Medical Conversational AI Models and Training Data. arXiv preprint arXiv:230408247\. 2023.
*   [3] Tu T, Azizi S, Driess D, Schaekermann M, Amin M, Chang PC, et al. Towards Generalist Biomedical AI. arXiv preprint arXiv:230714334\. 2023.
*   [4] Milne-Ives M, de Cock C, Lim E, Shehadeh MH, de Pennington N, Mole G, et al. The effectiveness of artificial intelligence conversational agents in health care: systematic review. Journal of medical Internet research. 2020;22(10):e20346.
*   [5] Moudatsou M, Stavropoulou A, Philalithis A, Koukouli S. The role of empathy in health and social care professionals. In: Healthcare. vol. 8\. MDPI; 2020\. p. 26.
*   [6] Abbasian M, Khatibi E, Azimi I, Oniani D, Abad ZSH, Thieme A, et al. Foundation Metrics: Quantifying Effectiveness of Healthcare Conversations powered by Generative AI. arXiv preprint arXiv:230912444\. 2023.
*   [7] Moreira A, Quintas C, Guimarães T, Santos MF. Multichannel Mobile Companions for Personalized Healthcare Opportunities and Challenges. Procedia Computer Science. 2023;220:838-43.
*   [8] Moore O. How Are Consumers Using Generative AI?; Accessed: January 2024. [https://a16z.com/how-are-consumers-using-generative-ai/](https://a16z.com/how-are-consumers-using-generative-ai/).
*   [9] Denecke K, May R. Investigating conversational agents in healthcare: Application of a technical-oriented taxonomy. Procedia Computer Science. 2023;219:1289-96.
*   [10] Plsek PE, Greenhalgh T. The challenge of complexity in health care. Bmj. 2001;323(7313):625-8.
*   [11] Sturmberg JP, Martin CM. Complexity and health–yesterday’s traditions, tomorrow’s future. J Eval Clin Pract. 2009;15(3):543-8.
*   [12] Luo R, Sun L, Xia Y, Qin T, Zhang S, Poon H, et al. BioGPT: generative pre-trained transformer for biomedical text generation and mining. Briefings in Bioinformatics. 2022;23(6):bbac409.
*   [13] Li Y, Li Z, Zhang K, Dan R, Jiang S, Zhang Y. ChatDoctor: A Medical Chat Model Fine-Tuned on a Large Language Model Meta-AI (LLaMA) Using Medical Domain Knowledge. Cureus. 2023;15(6).
*   [14] Meier P, Beinke JH, Fitte C, Behne A, Teuteberg F. FeelFit-Design and Evaluation of a Conversational Agent to Enhance Health Awareness. In: ICIS; 2019\. .
*   [15] Montenegro JLZ, da Costa CA, da Rosa Righi R. Survey of conversational agents in health. Expert Systems with Applications. 2019;129:56-67.
*   [16] Tudor Car L, Dhinagaran DA, Kyaw BM, Kowatsch T, Joty S, Theng YL, et al. Conversational agents in health care: scoping review and conceptual analysis. Journal of medical Internet research. 2020;22(8):e17158.
*   [17] Laranjo L, Dunn AG, Tong HL, Kocaballi AB, Chen J, Bashir R, et al. Conversational agents in healthcare: a systematic review. Journal of the American Medical Informatics Association. 2018;25(9):1248-58.
*   [18] Zhao WX, Zhou K, Li J, Tang T, Wang X, Hou Y, et al. A survey of large language models. arXiv preprint arXiv:230318223\. 2023.
*   [19] Han R, Acosta JN, Shakeri Z, Ioannidis J, Topol E, Rajpurkar P. Randomized Controlled Trials Evaluating AI in Clinical Practice: A Scoping Evaluation. medRxiv. 2023:2023-09.
*   [20] Sallam M; MDPI. ChatGPT utility in healthcare education, research, and practice: systematic review on the promising perspectives and valid concerns. Healthcare. 2023;11(6):887.
*   [21] Biswas S. ChatGPT and the future of medical writing. Radiological Society of North America; 2023.
*   [22] Aydın Ö, Karaarslan E. OpenAI ChatGPT generated literature review: Digital twin in healthcare. Available at SSRN 4308687\. 2022.
*   [23] Chen S, Kann BH, Foote MB, Aerts HJ, Savova GK, Mak RH, et al. The utility of ChatGPT for cancer treatment information. medRxiv. 2023:2023-03.
*   [24] Liu C, Ma Y, Kothur K, Nikpour A, Kavehei O. BioSignal Copilot: Leveraging the power of LLMs in drafting reports for biomedical signals. medRxiv. 2023:2023-06.
*   [25] Lee J, Yoon W, Kim S, Kim D, Kim S, So CH, et al. BioBERT: a pre-trained biomedical language representation model for biomedical text mining. Bioinformatics. 2020;36(4):1234-40.
*   [26] Gu Y, Tinn R, Cheng H, Lucas M, Usuyama N, Liu X, et al. Domain-specific language model pretraining for biomedical natural language processing. ACM Transactions on Computing for Healthcare (HEALTH). 2021;3(1):1-23.
*   [27] Gutierrez BJ, McNeal N, Washington C, Chen Y, Li L, Sun H, et al. Thinking about gpt-3 in-context learning for biomedical ie? think again. arXiv preprint arXiv:220308410\. 2022.
*   [28] Moradi M, Blagec K, Haberl F, Samwald M. Gpt-3 models are poor few-shot learners in the biomedical domain. arXiv preprint arXiv:210902555\. 2021.
*   [29] Xu S, Yang L, Kelly C, Sieniek M, Kohlberger T, Ma M, et al. ELIXR: Towards a general purpose X-ray artificial intelligence system through alignment of large language models and radiology vision encoders. arXiv preprint arXiv:230801317\. 2023.
*   [30] Belyaeva A, Cosentino J, Hormozdiari F, McLean CY, Furlotte NA. Multimodal LLMs for health grounded in individual-specific data. arXiv preprint arXiv:230709018\. 2023.
*   [31] Guerreiro NM, Alves D, Waldendorf J, Haddow B, Birch A, Colombo P, et al. Hallucinations in large multilingual translation models. arXiv preprint arXiv:230316104\. 2023.
*   [32] Neisser U. Perceiving, anticipating, and imagining. In: Perception and Cognition: Issues in the Foundations of Psychology. University of Minnesota Press, Minneapolis; 1978\. .
*   [33] Yao S, Yu D, Zhao J, Shafran I, Griffiths TL, Cao Y, et al. Tree of thoughts: Deliberate problem solving with large language models. arXiv preprint arXiv:230510601\. 2023.
*   [34] Yao S, Zhao J, Yu D, Du N, Shafran I, Narasimhan K, et al. React: Synergizing reasoning and acting in language models. arXiv preprint arXiv:221003629\. 2022.
*   [35] Wang L, Xu W, Lan Y, Hu Z, Lan Y, Lee RKW, et al. Plan-and-solve prompting: Improving zero-shot chain-of-thought reasoning by large language models. arXiv preprint arXiv:230504091\. 2023.
*   [36] Google. Google Translate; Accessed: January 2024. [https://py-googletrans.readthedocs.io/en/latest/](https://py-googletrans.readthedocs.io/en/latest/).
*   [37] Lyu H, Jiang S, Zeng H, Xia Y, Luo J. LLM-Rec: Personalized Recommendation via Prompting Large Language Models. arXiv preprint arXiv:230715780\. 2023.
*   [38] Hao Y, Chi Z, Dong L, Wei F. Optimizing prompts for text-to-image generation. arXiv preprint arXiv:221209611\. 2022.
*   [39] OpenAI. ChatGPT: Prompt engineering instrauctions.; Accessed: January 2024. [https://platform.openai.com/docs/guides/prompt-engineering/strategy-write-clear-instructions](https://platform.openai.com/docs/guides/prompt-engineering/strategy-write-clear-instructions).
*   [40] Labbaf S, Abbasian M, Azimi I, Dutt N, Rahmani AM. ZotCare: a flexible, personalizable, and affordable mhealth service provider. Frontiers in Digital Health. 2023;5.
*   [41] Jusoh S. A Survey on Trend, Opportunities and Challenges of mHealth Apps. Int J Interact Mob Technol. 2017;11(6):73-85.
*   [42] Chiarini G, Ray P, Akter S, Masella C, Ganz A. mHealth technologies for chronic diseases and elders: a systematic review. IEEE Journal on Selected Areas in Communications. 2013;31(9):6-18.
*   [43] Labbaf S, Abbasian M, Azimi I, Dutt N, Rahmani A. ZotCare: A Flexible, Personalizable, and Affordable mHealth Service Provider. Frontiers in Digital Health;5:1253087.
*   [44] ilumivu. ilumivu; Accessed: January 2024. [https://ilumivu.com/](https://ilumivu.com/).
*   [45] Nori H, Lee YT, Zhang S, Carignan D, Edgar R, Fusi N, et al. Can generalist foundation models outcompete special-purpose tuning? case study in medicine. arXiv preprint arXiv:231116452\. 2023.
*   [46] Hiemstra D. Information retrieval models. Information Retrieval: searching in the 21st Century. 2009:1-19.
*   [47] Turtle HR, Croft WB. A comparison of text retrieval models. The computer journal. 1992;35(3):279-90.
*   [48] Nori H, Lee Y, Zhang S, et al.. Can generalist foundation models outcompete special-purpose tuning? Case study in medicine. arXiv. 2023;.
*   [49] Topol EJ. As artificial intelligence goes multimodal, medical applications multiply. Science. 2023;381(6663).
*   [50] Clarke MA, Moore JL, Steege LM, Koopman RJ, Belden JL, Canfield SM, et al. Health information needs, sources, and barriers of primary care patients to achieve patient-centered care: A literature review. Health informatics journal. 2016;22(4):992-1016.
*   [51] Jain R. Folk computing. Communications of the ACM. 2003;46(4):27-9.
*   [52] Borovoy RD. Folk computing: Designing technology to support face-to-face community building [Ph.D. thesis]. Massachusetts Institute of Technology; 2002.
*   [53] Labbaf S, Abbasian M, Nguyen B, Lucero M, Ahmed MS, Yunusova A, et al. Physiological and Emotional Assessment of College Students Using Wearable and Mobile Devices During the 2020 Covid-19 Lockdown: An Intensive, Longitudinal Dataset. Longitudinal Dataset. 2023.
*   [54] Samsung. Samsung Gear Watch; Accessed: January 2024. [https://www.samsung.com/us/mobile/wearables/smartwatches/gear-sport-blue-sm-r600nzbaxar/](https://www.samsung.com/us/mobile/wearables/smartwatches/gear-sport-blue-sm-r600nzbaxar/).
*   [55] Makowski D, Pham T, Lau ZJ, Brammer JC, Lespinasse F, Pham H, et al. NeuroKit2: A Python toolbox for neurophysiological signal processing. Behavior Research Methods. 2021 feb;53(4):1689-96. Available from: [https://doi.org/10.3758%2Fs13428-020-01516-y](https://doi.org/10.3758%2Fs13428-020-01516-y).
*   [56] Shaffer F, Ginsberg JP. An overview of heart rate variability metrics and norms. Frontiers in public health. 2017:258.
*   [57] Khatibi E, Abbasian M, Azimi I, Labbaf S, Feli M, Borelli J, et al. Impact of COVID-19 Pandemic on Sleep Including HRV and Physical Activity as Mediators: A Causal ML Approach. In: 2023 IEEE 19th International Conference on Body Sensor Networks (BSN). IEEE; 2023\. p. 1-4.
*   [58] Yang Z, Khatibi E, Nagesh N, Abbasian M, Azimi I, Jain R, et al. ChatDiet: Empowering personalized nutrition-oriented food recommender chatbots through an LLM-augmented framework. Smart Health. 2024;32:100465.
*   [59] Abbasian M, Yang Z, Khatibi E, Zhang P, Nagesh N, Azimi I, et al. Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients. arXiv preprint arXiv:240210153\. 2024.
*   [60] Park JI, Abbasian M, Azimi I, Bounds D, Jun A, Han J, et al. Building Trust in Mental Health Chatbots: Safety Metrics and LLM-Based Evaluation Tools. arXiv preprint arXiv:240804650\. 2024.
*   [61] Abbasian M, Azimi I, Feli M, Rahmani AM, Jain R. Empathy Through Multimodality in Conversational Interfaces. arXiv preprint arXiv:240504777\. 2024.
*   [62] Singh S, Karatzas A, Fore M, Anagnostopoulos I, Stamoulis D. An LLM-Tool Compiler for Fused Parallel Function Calling. arXiv preprint arXiv:240517438\. 2024.
*   [63] Claude. Claude: A next-generation AI assistant for your tasks, no matter the scale; Accessed: January 2024. [https://www.anthropic.com/product](https://www.anthropic.com/product).
*   [64] Hathaliya JJ, Tanwar S. An exhaustive survey on security and privacy issues in Healthcare 4.0. Computer Communications. 2020;153:311-35.
*   [65] Price WN, Cohen IG. Privacy in the age of medical big data. Nature medicine. 2019;25(1):37-43.
*   [66] Xu Y, Ma T, Tang M, Tian W. A survey of privacy preserving data publishing using generalization and suppression. Applied Mathematics & Information Sciences. 2014;8(3):1103.
*   [67] Wang X, Wei J, Schuurmans D, Le Q, Chi E, Narang S, et al. Self-consistency improves chain of thought reasoning in language models. arXiv preprint arXiv:220311171\. 2022.
*   [68] Abbasian M, Khatibi E, Azimi I, Oniani D, Shakeri Hossein Abad Z, Thieme A, et al. Foundation metrics for evaluating effectiveness of healthcare conversations powered by generative AI. NPJ Digital Medicine. 2024;7(1):82.

## 0emAppendix 1 - Sample Task Implementation

We encourage readers and openCHA users to review our documentation page ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://docs.opencha.com/index.html](https://docs.opencha.com/index.html) for the latest updates on openCHA.

![Refer to caption](img/7b0ebc92396272858c4ecf60dde03e26.png)

Figure 7: A sample implemented task inside openCHA framework. Each task includes prompt parameters which helps our Task Planner LLM understand what types of data are needed and how to provide them to the current task.

In this appendix, we will detail the curation of tasks within the openCHA framework and their introduction as external sources to the Task Planner. Initially, we will describe the types of tasks that can be added to the framework. Subsequently, we will outline the implementation process for tasks within the framework. Finally, we will present an example of a task that has already been implemented, along with sample prompts associated with it.

Tasks may be implemented locally or as a service. Local implementation entails coding all necessary components on-site to ensure successful task execution, which may involve providing a Python library, a GitHub repository, or a complete task implementation.

Service-based implementation involves hosting services on a server and offering APIs for utilization. In this scenario, the task should invoke these APIs using Python libraries. It is incumbent upon the service provider to maintain the task and supply comprehensive documentation for users, including registration procedures, API key acquisition, and privacy policies.

Figure [7](https://arxiv.org/html/2310.02374v5#S7.F7 "Figure 7 ‣ 0emAppendix 1 - Sample Task Implementation ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") displays an example of a task implementation in openCHA designed to perform a Google search and return the first search result URL. Each task includes configurations to aid the Task Planner LLM in understanding when and how to use this task. The parameters are as follows:

name

Purpose: Uniquely identifies the task within the system, and this name will appear in the interface for task selection.

Convention: Generally in lowercase underscore_case, often derived from the task’s functionality for better readability and maintainability.

Example: name = "google_search"

chat_name:

Purpose: Used for referencing the task in user interfaces or chats, particularly for explainability. If a user inquires about the tasks used, these names will be displayed.

Convention: Typically in CamelCase, should be descriptive yet concise.

Example: chat_name = "GoogleSearch"

description:

Purpose: Provides an explanation of the task’s function. This information helps the Task Planner LLM decide when to deploy the task.

Convention: The description should be comprehensive, outlining task capabilities, prioritization, or specific conditions for its use. For instance, ensuring the Task Planner prioritizes patient data analysis over frequent internet searches can be mentioned here with a low priority setting.

Example: description = "Uses Google to search the internet for the requested query and returns the URL of the top website."

dependencies:

Purpose: Lists other tasks or services this task depends on, informing the Task Planner LLM of these dependencies during planning.

Convention: An array of task identifiers that match the name attribute of dependent tasks.

Example: dependencies = [TaskType.FETCH_WEATHER_DATA]

inputs:

Purpose: Specifies the inputs required by the task. This parameter is crucial as it guides the Task Planner LLM to provide the appropriate inputs for this task’s "_execute" function.

Convention: An array of strings, each describing a specific input.

Example: inputs = ["It should be a search query."]

outputs:

Purpose: Details what the task returns, assisting the Task Planner LLM in understanding the return structure for proper handling of the output.

Convention: An array of strings describing each output, with a full description of the return data format.

Example: outputs = ["It returns a JSON object containing key: url. For example: ’url’: ’http://google.com’"]

output_type

Purpose: Instructs the Orchestrator on how to handle the output of this task. If output_type is True, the data is stored in the Datapipe; otherwise, it is directly returned to the Task Planner LLM. This flexibility allows task designers to determine whether their task involves intermediate information for use by other tasks or provides the final answer.

Convention: Boolean values (True or False).

_execute function

Purpose: This function must be implemented correctly. If all other parameters are set accurately, developers can trust that the Task Planner LLM will invoke this function with the appropriate inputs. For instance, as shown in Figure [7](https://arxiv.org/html/2310.02374v5#S7.F7 "Figure 7 ‣ 0emAppendix 1 - Sample Task Implementation ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework"), the sole expected input is a search query. The Task Planner LLM will correctly supply the search query as an input to the "_execute" function, accessible as the first element in the inputs array (since the only entry in the "inputs" configuration pertains to the search query, this array will contain just one item). Utilizing the google_engine Python library, this function searches for the query online and returns the URL. Note that the result we are returning for the Google search task aligns with what is specified in the "outputs" parameter, demonstrating the coherence between the task setup and its execution.

## 0emAppendix 2 - Task Planner Sample Implementation

We encourage readers and openCHA users to review our documentation page ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://docs.opencha.com/index.html](https://docs.opencha.com/index.html) for the latest updates on openCHA.

In this appendix, we elaborate on the implementation of the Tree of Thought prompting technique. We describe how available tasks are introduced to the Tree of Thought, the process of curating and generating the planning prompt, and ultimately, how the final planning results are transformed into executable tasks by the Task Executor.

The core component of the Task Planner is the selection of an appropriate planning technique. Among various techniques using LLMs that have proven practical, we opted for the Tree of Thought approach.

To effectively implement the Tree of Thought, we structured the planning process into two main sections. Initially, we prepare a comprehensive prompt that includes a list of available tasks, relevant metadata, records of previously performed tasks, conversation history, and user input. This setup ensures the planner is thoroughly informed about the tasks it can call upon, the metadata that should be used or passed along, and the context of past interactions to prevent redundant planning and maintain the continuity of conversations.

Figure [8](https://arxiv.org/html/2310.02374v5#S8.F8 "Figure 8 ‣ 0emAppendix 2 - Task Planner Sample Implementation ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") illustrates the Tree of Thought planning prompt. In this phase, the Tree of Thought uses all this gathered information and the user’s query to devise three distinct task sequences or strategies. Each strategy is designed to collect the necessary information to address the query efficiently. Next, it is asked to provide the pros and cons of each strategy, ultimately selecting the most suitable one as the final decision.

![Refer to caption](img/f968ed95e6574640cfe78883637b4a2e.png)

Figure 8: The first stage Tree of Thought planning prompt

In the second stage of the planning process, our objective is to translate the chosen decision into sequences of task functions that the Orchestrator can understand and execute. Within the Orchestrator, we have implemented a function named execute_task, which serves as an interface to retrieve and execute tasks with the appropriate inputs accurately.

Our second prompt instructs the LLM to invoke this execute_task function, ensuring that the correct inputs are provided for each task. Figure [9](https://arxiv.org/html/2310.02374v5#S8.F9 "Figure 9 ‣ 0emAppendix 2 - Task Planner Sample Implementation ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") displays this second prompt, showcasing how we structure these commands within the LLM to streamline task execution.

![Refer to caption](img/e221b761757f9a8625ea6d8ffef7cbbd.png)

Figure 9: The second stage Tree of Thought planning prompt

## 0emAppendix 3 - Response Generator implementation

We encourage readers and openCHA users to review our documentation page ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://docs.opencha.com/index.html](https://docs.opencha.com/index.html) for the latest updates on openCHA.

In this appendix, we provide a detailed explanation of the Response Generator’s implementation. We begin by distinguishing between the Response Generator and the Task Planner. Following this, we demonstrate how the outcomes of planning and task execution are delivered to the Response Generator. Lastly, we present the prompts employed to ensure that the Response Generator depends on the tasks’ responses rather than the internal knowledge of the LLM used for generating responses.

One of the key objectives of the Agentic approach is to create a comprehensive mega prompt that allows an LLM to generate more precise answers by incorporating all necessary information. Our innovation centers on dynamically generating this mega prompt in response to user queries. The Task Planner initiates this process by determining the sequence of tasks it deems necessary to accurately address the current question.

Once planning is complete, the Orchestrator compiles the results of these tasks to create the mega prompt. The composition of this prompt is influenced by the nature of the tasks and their outputs. For instance, if a task’s output_type is set to true, indicating that the result should be stored in the Datapipe for subsequent analysis task use, the Orchestrator will omit this result from the prompt. The focus in this case is on the analysis’s outcome rather than intermediate results. Similarly, for tasks that produce metadata, such as an image, the metadata itself isn’t passed directly to the Response Generator. Instead, descriptive information about the metadata, like details of a plot image and relevant data insights, are provided to the Response Generator, indicating that this information accompanies the image.

![Refer to caption](img/cac941cdf75859ea0ae6bf44548113b9.png)

Figure 10: The Response Generator prompt.

Figure [10](https://arxiv.org/html/2310.02374v5#S9.F10 "Figure 10 ‣ 0emAppendix 3 - Response Generator implementation ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") illustrates the prompt provided to the Response Generator LLM. Initially, the Orchestrator formats answers derived from the task results, which we refer to as the "thinker" part of the prompt. The LLM is then instructed to rely exclusively on this thinker prompt, disregarding its internal knowledge. Additionally, we include guidelines to help the LLM understand specific system implementations, such as Datapipe access keys or metadata details.

In the openCHA framework, this prompt is customizable. A "prefix" feature allows developers to add information to better align the response generator with their specific task requirements.

## 0emAppendix 4 - Sample Prompts

In this Appendix, we demonstrate the generated prompt for a simple example involving two tasks: Internet search and webpage information extraction. The user’s query, "How to improve my sleep," is first directed to the Task Planner. Table [3](https://arxiv.org/html/2310.02374v5#S10.T3 "Table 3 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") displays the initial planning stage where the Tree of Thought Planner is requested to develop three strategies, evaluate their pros and cons, and choose one to execute. Table [4](https://arxiv.org/html/2310.02374v5#S10.T4 "Table 4 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") presents the prompt for the second planning stage, where the LLM is instructed to generate Python code that the Orchestrator can execute based on the chosen strategy, along with the necessary inputs. Table [5](https://arxiv.org/html/2310.02374v5#S10.T5 "Table 5 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") and [6](https://arxiv.org/html/2310.02374v5#S10.T6 "Table 6 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") provides an example of the generated code, illustrating how the LLM correctly inputs the query "tips to improve sleep" for the google_search task. Table [7](https://arxiv.org/html/2310.02374v5#S10.T7 "Table 7 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") shows how openCHA creates a prompt to assist the Response Generator in utilizing the tasks’ results to respond to the user’s query effectively.

Additionally, Table [8](https://arxiv.org/html/2310.02374v5#S10.T8 "Table 8 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") provides an example of a prompt for a task that has multiple inputs and outputs, with the result being stored in the Data Pipe. Table [9](https://arxiv.org/html/2310.02374v5#S10.T9 "Table 9 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") illustrates the process of passing the Data Pipe key between tasks. In this setup, each task returns a Data Pipe key, and within the generated code, the output from one task is appropriately passed to the next task as its input. The execute_task function manages access to the Data Pipe when a key is provided, ensuring that the results of the tasks are not directly transmitted to the LLM.

Table 3: Tree of Thought Planning first planning stage.

| As a knowledgeable and empathetic health assistant, your primary objective is to provide the user with precise and valuable information regarding their health and well-being. Utilize the available tools effectively to answer health-related queries. Here are the tools at your disposal: ———————————– **google_search**: Uses google to search the internet for the requested query and returns the url of the top website. This tool have the following outputs: It returns a json object containing key: **url**. For example: ’url’: ’http://google.com’ ———————————– **extract_text**: Extract all the text on the current webpage This tool have the following outputs: An string containing the text of the scraped webpage. ———————————– The following is the format of the information provided: MetaData: This contains the names of data files of different types, such as images, audio, video, and text. You can pass these files to tools when needed. History: The history of previous chats happened. Review the history of any previous responses relevant to the current query. PreviousActions: the list of actions that have already been performed. You should start planning, knowing that these actions are performed. Question: The input question that you must answer. Considering previously actions and their results, use the tools and provided information, first suggest three creative strategies with detailed explanation consisting of sequences of tools to properly answer the user query. Make sure the strategies are comprehensive enough and use proper tools. The tools constraints should be always satisfied. After specifying the strategies, mention the pros and cons of each strategy. In the end, decide the best strategy and write the detailed tool executions step by step. start your final decision with ’Decision:’. Begin! ========================= USER: How to improve my sleep? CHA: |

Table 4: Tree of Thought Planning second planning stage: generating the task execution code part 1.

| Decision: I will go with Strategy 1 as it provides the most recent and relevant information available on the internet, which is crucial for improving sleep. Now, let’s proceed with the detailed tool executions for Strategy 1: 1\. Use the "google_search" tool to find the top websites with tips to improve sleep. 2\. Once we have the top website, we can use the "extract_text" tool to extract the relevant information about improving sleep from the webpage. Let’s start with step 1. ========================= Tools: ———————————– **google_search**: Uses google to search the internet for the requested query and returns the url of the top website. The input to this tool should be a list of data representing: 1-It should be a search query. This tool will return the following data: - It returns a json object containing key: **url**. For example: ’url’: ’http://google.com’ ———————————– **extract_text**: Extract all the text on the current webpage The input to this tool should be a list of data representing: 1-url to extract the text from. It requires links which is gathered from other tools. Never provide urls on your own. This tool will return the following data: - An string containing the text of the scraped webpage. ———————————– |

Table 5: Tree of Thought Planning second planning stage: generating the task execution code part 2.

| ========================= You are a skilled Python programmer who can solve problems and convert them into Python codes. Using the selected final strategy mentioned in the ’Decision: ’, create a python code inside a “‘python “‘ block that outlines a sequence of steps using the Tools. Assume that there is a **self.execute_task** function that can execute the tools in it. The execute_task receives the task name and an array of the inputs and returns the result. Make sure that you always pass an array as a second argument. You can call tools like this: **task_result = self.execute_task(’tool_name’, [’input1’, ’input2’, …])**. The flow should utilize this style to represent the tools available. Make sure all the execute_task calls outputs are stored in a variable. If a step’s output is required as input for a subsequent step, ensure the Python code captures this dependency clearly. The output variables should be directly passed as inputs with no changes in the wording. If the tool input is a datapipe, only put the variable as the input. For each tool, include necessary parameters directly without any names and assume each will return an output. The outputs’ description are provided for each tool individually. Make sure you use the directives when passing the outputs. Question: How to improve my sleep? |

Table 6: Sample generated code for task execution.

| # Step 1: Use google_search to find the top websites with tips to improve sleep. search_query = "tips to improve sleep" search_result = self.execute_task(’google_search’, [search_query]) # Step 2: Use extract_text to extract the relevant information about improving sleep from the webpage. url = search_result[’url’] sleep_tips_text = self.execute_task(’extract_text’, [url]) |

Table 7: Response generator sample prompt.

| ===========Thinker: MetaData: History: —————— google_search: [’tips to improve sleep’] ’url’: ’https://www.mayoclinic.org/healthy-lifestyle/adult-health/in-depth/sleep/art-20048379’ —————— —————— extract_text: [’https://www.mayoclinic.org/healthy-lifestyle/adult-health/in-depth/sleep/art-20048379’] Sleep tips: 6 steps to better sleep - Mayo Clinic This content does not have an English version. This content does not have an Arabic version. Skip to content Care at Mayo Clinic Patient-Centered Care About Mayo… (we cut the text to shorten the table) —————— ========== System: . You are a very helpful, empathetic health assistant, and your goal is to help the user get accurate information about his/her health and well-being; using the Thinker gathered information and the History, Provide an empathetic, proper answer to the user. Consider Thinker as your trusted source, and use whatever it provides. Make sure that the answer is explanatory enough. Don’t change Thinker returned URLs or references. Also, add explanations based on instructions from the Thinker. Don’t directly put the instructions in the final answer to the user. Never answer outside of the Thinker’s provided information. Additionally, refrain from including or using any keys, such as ’datapipe:6d808840-1fbe-45a5-859a-abfbfee93d0e,’ in your final response. Return all ‘address:[path]‘ exactly as they are.User: How to improve my sleep? |

Table 8: Task prompt with Data Pipe.

| ———————————– **affect_ppg_get**: Returns the ppg data for a specific patient over a date or a period (if two dates are provided). This will return the detailed raw data and store it in the Data Pipe. The input to this tool should be a list of data representing: 1-user ID in string. It can be referred to as user, patient, individual, etc. Start with ’par_’ followed by a number (e.g., ’par_1’). 2-start date of the sleep data in a string with the following format: ‘%Y-%m-%d.‘ 3-end date of the sleep data in a string with the following format: ‘%Y-%m-%d.‘ If there is no end date, the value should be an empty string (i.e., ”) This tool will return the following data: - returns an array of JSON objects which contains the following keys: **date (in milliseconds)**: epoch format **ppg**: is the ppg value. **hr (in beats per minute)**: is the heart rate of the patient. The result will be stored in the Data Pipe. ———————————– |

Table 9: Sample code generation when a task’s result is stored in the Data Pipe.

| # Step 1: Get PPG data for patient 5 for the entire month of August 2020 ppg_data_result = self.execute_task(’affect_ppg_get’, [’par_5’, ’2020-08-01’, ’2020-08-31’]) # Step 2: Analyze the HRV parameters from the obtained PPG data hrv_analysis_result = self.execute_task(’affect_ppg_analysis’, [ppg_data_result]) # Step 3: Estimate the stress level for patient 5 during August 2020 using the HRV analysis results stress_level_result = self.execute_task(’affect_stress_analysis’, [hrv_analysis_result]) |