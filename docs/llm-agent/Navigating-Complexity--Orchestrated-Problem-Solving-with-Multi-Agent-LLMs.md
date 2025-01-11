<!--yml
category: 未分类
date: 2025-01-11 12:50:18
-->

# Navigating Complexity: Orchestrated Problem Solving with Multi-Agent LLMs

> 来源：[https://arxiv.org/html/2402.16713/](https://arxiv.org/html/2402.16713/)

Sumedh Rasal
Georgia Institute of Technology
Chicago, IL
srasal3@gatech.edu
\AndE. J. Hauer
EJ@ThoughtCrafters.com 

###### Abstract

Large Language Models (LLMs) have demonstrated remarkable capabilities in solving various tasks, yet they often struggle with comprehensively addressing complex and vague problems. Existing approaches, including multi-agent LLM systems, offer solutions to certain challenges but still require manual setup and lack scalability. To address this gap, we propose a novel approach leveraging decomposition to enable LLMs to tackle vague problems effectively.

Our approach involves an orchestrating LLM that interacts with users to understand the problem and then decomposes it into tangible sub-problems. Instead of expecting the LLM to solve the entire problem in one go, we train it to ask follow-up questions to gain a deeper understanding of the user’s requirements. Once the problem is adequately understood, the orchestrating LLM divides it into smaller, manageable sub-problems. Each sub-problem is then assigned to specialized LLM agents or non-LLM functions for resolution. These agents work in parallel to solve their respective sub-problems, with the orchestrating LLM overseeing the process and compiling the solutions into a comprehensive answer for the user. By adopting this decomposition approach, we alleviate the constraints imposed by token limitations on LLM outputs and empower them to provide nuanced solutions to complex and ambiguous problems.

Through our approach, we aim to enable LLMs to think and operate more like humans, breaking down complex problems into manageable parts and collaboratively solving them. This not only enhances the problem-solving capabilities of LLMs but also offers a scalable and efficient method for addressing a wide range of real-world challenges.

*K*eywords Decomposition  $\cdot$ Multi-Agents

## 1 Introduction

In recent years, the proliferation of Large Language Models (LLMs) has revolutionized problem-solving across various domains [[Devlin et al., 2018](https://arxiv.org/html/2402.16713v2#bib.bibx11)] [[Patil et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx25)] [[Touvron et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx32)]. These models possess the ability to comprehend user queries and generate responses, often providing novel insights and solutions [[Wei et al., 2022a](https://arxiv.org/html/2402.16713v2#bib.bibx33)]. Notably, LLMs exhibit exceptional performance in tasks that can be tackled through zero-shot or few-shot training techniques [[Brown et al., 2020](https://arxiv.org/html/2402.16713v2#bib.bibx2)], such as transforming questions into familiar formats or solving coding problems based on analogous examples [[Chen et al., 2021](https://arxiv.org/html/2402.16713v2#bib.bibx5)] [[Sanh et al., 2021](https://arxiv.org/html/2402.16713v2#bib.bibx29)]. As such, they hold immense potential for automating numerous tasks and streamlining workflows across diverse professional domains [[Liu et al., 2023b](https://arxiv.org/html/2402.16713v2#bib.bibx22)] [[Chiang and Lee, 2023](https://arxiv.org/html/2402.16713v2#bib.bibx6)] [[Gao et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx13)] [[Shen et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx31)] [[Zheng et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx37)].

However, despite their remarkable capabilities, LLMs are not immune to limitations. Instances of hallucination, contradictory responses, or misinterpretation of user queries highlight the challenges inherent in relying solely on these models for problem-solving [[Azamfirei et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx1)]. Existing frameworks designed to aid users in systematically addressing novel problems are constrained by token limits imposed by LLM models [[Gur et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx15)]. These limits restrict the number of words or tokens that an LLM can process in a single message or query, often hindering the comprehensiveness of responses and impeding the continuity of conversations. While approaches such as chaining previous conversations or employing semantic search techniques mitigate some of these challenges, they are susceptible to errors, including redundancy or irrelevance in retrieved information [[Ouyang et al., 2022](https://arxiv.org/html/2402.16713v2#bib.bibx24)] [[Saunders et al., 2022](https://arxiv.org/html/2402.16713v2#bib.bibx30)] [[Liu et al., 2023a](https://arxiv.org/html/2402.16713v2#bib.bibx21)].

To address these limitations, a promising avenue lies in leveraging multi-agent frameworks to tackle lengthy or complex problems [[Du et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx12)] [[Cohen et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx8)] [[Dafoe et al., 2021](https://arxiv.org/html/2402.16713v2#bib.bibx9)] [[Dafoe et al., 2020](https://arxiv.org/html/2402.16713v2#bib.bibx10)] [[Kramár et al., 2022](https://arxiv.org/html/2402.16713v2#bib.bibx16)] [[Yao et al., 2022](https://arxiv.org/html/2402.16713v2#bib.bibx36)]. Such frameworks distribute problem-solving tasks among multiple LLM agents, offering a more robust approach compared to heuristic methods [[Li et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx19)] [[Liang et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx20)] [[Qian et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx26)] [[Lazaridou et al., 2020](https://arxiv.org/html/2402.16713v2#bib.bibx17)] [[Graesser et al., 2020](https://arxiv.org/html/2402.16713v2#bib.bibx14)] [[Lee et al., 2018](https://arxiv.org/html/2402.16713v2#bib.bibx18)] [[Wu et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx35)]. However, the manual assignment of personas or roles to each LLM agent presents a bottleneck, particularly when dealing with novel problems lacking predefined roles [[Rasal, 2024](https://arxiv.org/html/2402.16713v2#bib.bibx28)].

In this paper, we propose a novel solution that integrates the strengths of existing approaches while addressing their limitations. Our approach centers on an orchestration strategy that employs problem decomposition to facilitate the seamless resolution of lengthy or complex problems. Inspired by human problem-solving approaches, our framework involves an orchestration LLM that engages in iterative conversations with users to ascertain problem requirements. Once the requirements are elucidated, the orchestration LLM decomposes the problem into manageable subproblems and assigns them to specialized agents, each tailored to address specific aspects of the problem. Notably, our approach incorporates non-LLM functionalities where necessary, enhancing robustness and adaptability to real-world tasks [[Qin et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx27)]. By leveraging problem decomposition and distributed task assignment, our framework offers a comprehensive and scalable solution for problem-solving with LLMs.

This is what we propose:

*   •

    Our proposed solution integrates problem decomposition and multi-agent orchestration, inspired by human problem-solving approaches, to address the limitations of existing frameworks.

*   •

    By leveraging problem decomposition and distributed task assignment, our framework offers a comprehensive and scalable solution for problem-solving with LLMs, enhancing adaptability to real-world tasks.

## 2 Methodology

In response to these challenges, our methodology adopts a novel approach that seeks to address the token limit issue through algorithmic enhancements rather than relying solely on memory expansion. At the core of our methodology lies the development of an Orchestration LLM, tasked with the responsibility of comprehending user problem statements, decomposing them into manageable subproblems, and orchestrating interactions among specialized LLM agents.

Inspired by human problem-solving strategies, our methodology mirrors the process of breaking down complex tasks into smaller, more digestible components. We employ advanced natural language processing (NLP) techniques to analyze and dissect complex queries, identifying distinct subproblems across various domains. These subproblems are then mapped to specialized LLM agents, each pre-trained on specific tasks such as data analysis, creative writing, or technical problem-solving, to ensure precision and relevance in their responses.

Analogous to seeking expertise in real-life scenarios, our methodology leverages collaboration with domain experts to optimize problem-solving outcomes. For instance, just as individuals seek tax experts for efficient tax filing, our approach integrates specialized agents to tackle domain-specific challenges effectively.

Furthermore, we develop an algorithmic framework that enables the Orchestration LLM to decompose complex problems based on predefined criteria such as problem domain, required expertise, and task complexity. Finally, we implement a task assignment system to allocate subproblems among selected specialized agents, considering factors such as availability, expertise, and workload to streamline the problem-solving process and optimize resource utilization.

The high-level algorithm to our framework can be found LABEL:tab:problem_solving

| Step | Orchestration of Problem Solving with LLMs |
| --- | --- |
| 1 | Require A complex problem statement |
| 2 | Ensure A comprehensive solution to the problem |
| 3 | Begin |
| 4 | Input the complex problem statement |
| 5 | Decompose the problem into subproblems |
| 6 | for each subproblem do |
| 7 | - Select a specialized LLM agent based on the subproblem’s domain and requirements |
| 8 | - Assign the subproblem to the selected agent |
| 9 | - Agent solves the subproblem and stores the solution |
| 10 | end for |
| 11 | - Aggregate the solutions of subproblems into a coherent overall solution |
| 12 | - Output the comprehensive solution |
| 13 | End |

### 2.1 Decomposition Approach

The implementation of our framework initiates with the user’s submission of a complex problem statement, facilitated through a conversational interface within the LangChain framework [[Chase, 2022](https://arxiv.org/html/2402.16713v2#bib.bibx4)]. Within the Orchestration LLM, this input undergoes meticulous processing, wherein key information and requirements are extracted and analyzed with precision. Subsequently, the Orchestration LLM seamlessly applies a sophisticated decomposition algorithm, systematically breaking down the problem into manageable subproblems, thereby ensuring clarity and precision throughout the process.

The decomposition approach was crafted through a series of few-shot training iterations employing the GPT4 model [[OpenAI, 2023](https://arxiv.org/html/2402.16713v2#bib.bibx23)] [[Bubeck et al., 2023](https://arxiv.org/html/2402.16713v2#bib.bibx3)]. Initially, the LLM model decomposed the main problem into vague subproblems, which necessitated further refinement to ensure alignment with the expected decomposition format. Utilizing few-shot training, we provided the model with chain-of-thought type reasoning to guide its decomposition process effectively.

Upon successful decomposition, the Orchestration LLM assumes the pivotal role of assigning these subproblems to the most suitable specialized LLM agents. This assignment process is meticulously orchestrated to ensure that each subproblem is delegated to an agent possessing the requisite expertise and capabilities to effectively address it.

Central to the framework’s efficacy is the decomposition phase, which serves as its cornerstone. We deliberately crafted the initial design to be adaptable and generalizable across diverse problem domains, prioritizing the demonstration of decomposition capabilities over specialized use-case system development.

## 3 Experiments

Our experiments center around the utilization of an orchestration LLM agent, employing OpenAI’s gpt-4 model [[OpenAI, 2023](https://arxiv.org/html/2402.16713v2#bib.bibx23)], which adeptly engages with users to elucidate requirements and subsequently decomposes problems into manageable subproblems. This orchestration LLM also determines the most suitable specialized LLM agent to address each subproblem. Each specialized LLM agent employed in our experiments is based on OpenAI’s gpt3.5-turbo model, configured with a temperature parameter set to 0.0, reflecting the balance between coherence and creativity in responses. Facilitating communication among LLM agents is the LangChain framework, enabling seamless multi-agent interaction and facilitating focus on problem-solving.

### 3.1 Experiment 1: Travel Planning with LLM Agents

In this experiment, users present their travel plans to the framework, initiating a dialogue with the orchestration LLM to refine requirements and subsequently decompose the problem into subproblems. The orchestration LLM allocates tasks to specialized agents, which upon completion, contribute their findings for collation into a final course of action. Before execution, the orchestration LLM verifies details with the user to ensure accuracy.

Here’s the conversational output of the framework.

> Initial Interaction: Gathering Requirements
> 
> User: I need to book a return flight from Chicago Airport to Newark Airport from 3/22/24 to 3/26/24\. I want to fly in luxury and travel between the hours of 10 am and 4 pm.
> 
> Orchestration LLM Agent: Got it. To ensure I find the best options for you, could you specify your preferences for seating class and any additional services you might require during your flight?
> 
> User: I prefer business class and would like access to in-flight Wi-Fi and a window seat.
> 
> Orchestration LLM Agent Decomposes the Problem
> 
> The Orchestration LLM Agent processes the user’s request and decomposes it into subproblems to be handled by specialized agents:
> 
> Flight Search Agent: Find business class flights from Chicago to Newark that depart between 10 am and 4 pm on 3/22/24 and return flights on 3/26/24.
> 
> Amenity Preferences Agent: Identify airlines offering in-flight Wi-Fi and window seat availability in business class.
> 
> Booking Agent: Handle the booking process once the flight options are selected.
> 
> Interaction with Specialized Agents
> 
> Orchestration LLM Agent to Flight Search Agent: Find business class flights from Chicago to Newark departing between 10 am and 4 pm on 3/22/24 and returning on 3/26/24.
> 
> Flight Search Agent: Found several options.
> 
> Orchestration LLM Agent to Amenity Preferences Agent: Which of these options offers in-flight Wi-Fi and window seats in business class?
> 
> Amenity Preferences Agent: [Lists airlines that meet the criteria] Finalizing the Option
> 
> Orchestration LLM Agent to User: Based on your preferences, here are the best flight options that offer business class, in-flight Wi-Fi, and window seats. [Presents options]
> 
> User: I’ll take the option with Airline X on the 3/22 at 2 pm.
> 
> Orchestration LLM Agent to Booking Agent: Please book the flight with Airline X for 3/22, departing at 2 pm in business class, with a window seat and in-flight Wi-Fi.
> 
> Booking Agent: Your flight is booked. Here are your confirmation details.

### 3.2 Experiment 2: Exploratory Research and Summarization

Users pose exploratory research queries to the framework, prompting the orchestration LLM to segment the problem into subproblems and assign tasks to specialized agents. Following the completion of tasks, the specialized agents provide their outputs, which are synthesized by the orchestration LLM into a comprehensive summary. Feedback from the user is sought to refine the final output as necessary.

Here’s the conversational output of the framework.

> Initial Interaction: Gathering Requirements
> 
> User: I need to research topics about long-term memory management in LLMs, focusing on what’s currently possible and what’s not. Then, I’d like a 5-page article summarizing these findings and suggesting future research directions.
> 
> Orchestration LLM Agent Decomposes the Problem
> 
> The Orchestration LLM Agent analyzes the request and identifies several key tasks:
> 
> Literature Review Agent: Conduct a comprehensive review of existing research on long-term memory management in LLMs.
> 
> Analysis Agent: Analyze the collected research to distinguish between solved problems, ongoing challenges, and unexplored areas.
> 
> Writing Agent: Synthesize the findings into a coherent 5-page article, highlighting current capabilities, and limitations, and suggesting future research directions.
> 
> Interaction with Specialized Agents
> 
> Task 1: Conduct Literature Review
> 
> Orchestration LLM Agent to Literature Review Agent: Identify and summarize key research articles on long-term memory management in LLMs, focusing on methodologies, findings, and limitations.
> 
> Literature Review Agent: [Returns a summary of articles, including key methodologies, findings, challenges, and limitations related to long-term memory management.]
> 
> Task 2: Analyze Research
> 
> Orchestration LLM Agent to Analysis Agent: Based on the literature review, analyze the current state of long-term memory management in LLMs, identifying what is possible, what remains challenging, and potential areas for future research.
> 
> Analysis Agent: [Provides an analysis distinguishing between the current capabilities and limitations of long-term memory management in LLMs, along with suggestions for future research directions.]
> 
> Task 3: Write Article
> 
> Orchestration LLM Agent to Writing Agent: Using the information collected and analyzed, write a 5-page article summarizing the findings on long-term memory management in LLMs. Highlight what is currently possible, and the limitations, and suggest ideas for future research.
> 
> Writing Agent: [Drafts a 5-page article that summarizes the state of long-term memory management in LLMs, including an overview of existing research, current limitations, and proposed future directions.]
> 
> Finalizing the Article
> 
> Orchestration LLM Agent to User: Based on the comprehensive research and analysis conducted by specialized agents, we have compiled a 5-page article summarizing the current state of long-term memory management in LLMs. The article discusses what is currently possible, outlines the main challenges and limitations, and suggests potential areas for future research.
> 
> User: Reviews the article and provides feedback or requests revisions if necessary.

## 4 Results

Rather than relying solely on conventional datasets typically utilized in research, we opted to assess the efficacy of our framework using the GSM8K dataset, chosen to specifically demonstrate the capabilities of our approach. We employed four distinct types of agents in our experimentation: a single LLM agent powered by the GPT3.5-turbo model, two LLM agents supported by GPT3.5-turbo models, two LLM agents utilizing the GPT3.5-turbo model combined with the chain-of-thought and persona approach [[Wei et al., 2022b](https://arxiv.org/html/2402.16713v2#bib.bibx34)] [[Rasal, 2024](https://arxiv.org/html/2402.16713v2#bib.bibx28)], and finally, two LLM agents operating with the GPT4 model [[OpenAI, 2023](https://arxiv.org/html/2402.16713v2#bib.bibx23)] for orchestration and decomposition methodology and GPT3.5-turbo model for the specialized LLM agents, which involves breaking down the primary problem into subproblems. Our orchestration framework represents a notable advancement over existing methodologies, with detailed results presented in Table [1](https://arxiv.org/html/2402.16713v2#S4.T1 "Table 1 ‣ 4 Results ‣ Navigating Complexity: Orchestrated Problem Solving with Multi-Agent LLMs").

> "GSM8K consists of 8.5K high-quality grade school math problems created by human problem writers. We segmented these into 7.5K training problems and 1K test problems. These problems take between 2 and 8 steps to solve, and solutions primarily involve performing a sequence of elementary calculations using basic arithmetic operations (+ - / *) to reach the final answer. A bright middle school student should be able to solve every problem."[[Cobbe et al., 2021](https://arxiv.org/html/2402.16713v2#bib.bibx7)]

Furthermore, our observations indicated a significant enhancement in accuracy when employing a calculator function as opposed to a math-specialized LLM agent. This improvement stems from the inherent limitations of LLMs in executing mathematical operations, despite their versatility across various domains. We contend that rather than training LLMs for math operations, our framework facilitates the orchestration LLM in discerning whether a subproblem necessitates an LLM agent or an alternative non-LLM approach.

Table 1: GSM8K Data set Evaluation

| Agent | Solve Rate |
| --- | --- |
| Single GPT3.5-turbo | 50% |
| Multi-Agent GPT3.5-turbo | 55% |
| Multi-Agent GPT3.5-turbo (LLM-Harmony) | 65% |
| Multi-Agent GPT4 (Our Approach) | 73% |

## 5 Limitations

While our decomposition framework represents a significant advancement, it possesses inherent limitations that warrant consideration. Foremost among these limitations is the framework’s lack of generalization to address all conceivable problem scenarios, both novel and otherwise. We recognize the necessity of further training the orchestration model, essentially serving as a foundational Large Language Model (LLM), to acquire a comprehensive understanding of diverse problem-solving approaches. Presently, the model exhibits proficiency in decomposing various problems, yet its scope may not extend to accommodate the complexities of future challenges.

For instance, consider a user’s request for transportation. While the current model adeptly decomposes the problem into a specialized agent capable of booking a ride, scenarios may arise where multiple specialized agents possess this capability. In such instances, the orchestration LLM faces the challenge of determining the most appropriate course of action. Additionally, the framework lacks a mechanism to assimilate information about new specialized agents, highlighting the need for a retraining mechanism to continually update the orchestration LLM’s knowledge base.

Furthermore, while the current model effectively decomposes problems, it lacks the capability to iteratively learn and refine its problem-solving strategies based on the outcomes of decomposition steps. Addressing this limitation represents a key focus of our future research endeavors. Nonetheless, the primary objective of this paper is to disseminate our decomposition approach to the readership, offering insights into its application in problem-solving contexts.

## 6 Conclusion

In conclusion, our paper introduces a novel approach to problem-solving using a decomposition framework orchestrated by Large Language Models (LLMs). Through meticulous decomposition of complex problem statements into manageable subproblems, our framework demonstrates the potential to streamline problem-solving processes and mitigate the limitations inherent in current LLM-based solutions. By leveraging the inherent capabilities of LLMs to understand user queries and decompose problems, our approach offers a promising avenue for addressing a wide range of challenges across various domains.

Our implementation of the framework highlights its efficacy in breaking down intricate problem statements and assigning tasks to specialized LLM agents, resulting in coherent and comprehensive solutions. Moreover, our experiments underscore the versatility of the framework in facilitating tasks such as travel planning and research summary generation, showcasing its applicability in real-world scenarios.

However, our work also reveals several limitations that necessitate further research and development. While the decomposition framework represents a significant advancement, its generalization to novel problem scenarios remains an ongoing challenge. Additionally, the framework lacks mechanisms for adaptive learning and integration of new specialized agents, indicating avenues for future improvement.

Despite these limitations, our paper contributes to the growing body of literature on LLM-based problem-solving approaches by introducing a systematic and scalable framework. By sharing our approach and insights, we aim to stimulate further research and innovation in this domain, paving the way for enhanced problem-solving capabilities and the realization of artificial general intelligence.

## References

*   [Azamfirei et al., 2023] Azamfirei, R., Kudchadkar, S. R., and Fackler, J. (2023). Large language models and the perils of their hallucinations. Critical Care, 27(1):1–2.
*   [Brown et al., 2020] Brown, T., Mann, B., Ryder, N., Subbiah, M., Kaplan, J. D., Dhariwal, P., Neelakantan, A., Shyam, P., Sastry, G., Askell, A., et al. (2020). Language models are few-shot learners. Advances in neural information processing systems, 33:1877–1901.
*   [Bubeck et al., 2023] Bubeck, S., Chandrasekaran, V., Eldan, R., Gehrke, J., Horvitz, E., Kamar, E., Lee, P., Lee, Y. T., Li, Y., Lundberg, S., et al. (2023). Sparks of artificial general intelligence: Early experiments with gpt-4. arXiv preprint arXiv:2303.12712.
*   [Chase, 2022] Chase, H. (2022). Langchain, october 2022. URL https://github. com/langchain-ai/langchain.
*   [Chen et al., 2021] Chen, M., Tworek, J., Jun, H., Yuan, Q., Pinto, H. P. d. O., Kaplan, J., Edwards, H., Burda, Y., Joseph, N., Brockman, G., et al. (2021). Evaluating large language models trained on code. arXiv preprint arXiv:2107.03374.
*   [Chiang and Lee, 2023] Chiang, C.-H. and Lee, H.-y. (2023). Can large language models be an alternative to human evaluations? arXiv preprint arXiv:2305.01937.
*   [Cobbe et al., 2021] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., Hesse, C., and Schulman, J. (2021). Training verifiers to solve math word problems. arXiv preprint arXiv:2110.14168.
*   [Cohen et al., 2023] Cohen, R., Hamri, M., Geva, M., and Globerson, A. (2023). Lm vs lm: Detecting factual errors via cross examination. arXiv preprint arXiv:2305.13281.
*   [Dafoe et al., 2021] Dafoe, A., Bachrach, Y., Hadfield, G., Horvitz, E., Larson, K., and Graepel, T. (2021). Cooperative ai: machines must learn to find common ground. Nature, 593(7857):33–36.
*   [Dafoe et al., 2020] Dafoe, A., Hughes, E., Bachrach, Y., Collins, T., McKee, K. R., Leibo, J. Z., Larson, K., and Graepel, T. (2020). Open problems in cooperative ai. arXiv preprint arXiv:2012.08630.
*   [Devlin et al., 2018] Devlin, J., Chang, M.-W., Lee, K., and Toutanova, K. (2018). Bert: Pre-training of deep bidirectional transformers for language understanding. arXiv preprint arXiv:1810.04805.
*   [Du et al., 2023] Du, Y., Li, S., Torralba, A., Tenenbaum, J. B., and Mordatch, I. (2023). Improving factuality and reasoning in language models through multiagent debate. arXiv preprint arXiv:2305.14325.
*   [Gao et al., 2023] Gao, M., Ruan, J., Sun, R., Yin, X., Yang, S., and Wan, X. (2023). Human-like summarization evaluation with chatgpt. arXiv preprint arXiv:2304.02554.
*   [Graesser et al., 2020] Graesser, L., Cho, K., and Kiela, D. (2020). Emergent linguistic phenomena in multi-agent communication games.
*   [Gur et al., 2023] Gur, I., Furuta, H., Huang, A., Safdari, M., Matsuo, Y., Eck, D., and Faust, A. (2023). A real-world webagent with planning, long context understanding, and program synthesis. arXiv preprint arXiv:2307.12856.
*   [Kramár et al., 2022] Kramár, J., Eccles, T., Gemp, I., Tacchetti, A., McKee, K. R., Malinowski, M., Graepel, T., and Bachrach, Y. (2022). Negotiation and honesty in artificial intelligence methods for the board game of diplomacy. Nature Communications, 13(1):7214.
*   [Lazaridou et al., 2020] Lazaridou, A., Potapenko, A., and Tieleman, O. (2020). Multi-agent communication meets natural language: Synergies between functional and structural language learning.
*   [Lee et al., 2018] Lee, J., Cho, K., Weston, J., and Kiela, D. (2018). Emergent translation in multi-agent communication.
*   [Li et al., 2023] Li, G., Hammoud, H. A. A. K., Itani, H., Khizbullin, D., and Ghanem, B. (2023). Camel: Communicative agents for" mind" exploration of large scale language model society. arXiv preprint arXiv:2303.17760.
*   [Liang et al., 2023] Liang, T., He, Z., Jiao, W., Wang, X., Wang, Y., Wang, R., Yang, Y., Tu, Z., and Shi, S. (2023). Encouraging divergent thinking in large language models through multi-agent debate. arXiv preprint arXiv:2305.19118.
*   [Liu et al., 2023a] Liu, J., Xia, C. S., Wang, Y., and Zhang, L. (2023a). Is your code generated by chatgpt really correct? rigorous evaluation of large language models for code generation. arXiv preprint arXiv:2305.01210.
*   [Liu et al., 2023b] Liu, Y., Iter, D., Xu, Y., Wang, S., Xu, R., and Zhu, C. (2023b). Gpteval: Nlg evaluation using gpt-4 with better human alignment. arXiv preprint arXiv:2303.16634.
*   [OpenAI, 2023] OpenAI (2023). Gpt-4 technical report. arxiv 2303.08774. View in Article, 2:13.
*   [Ouyang et al., 2022] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C. L., Mishkin, P., Zhang, C., Agarwal, S., Slama, K., Ray, A., et al. (2022). Training language models to follow instructions with human feedback, 2022. URL https://arxiv. org/abs/2203.02155, 13.
*   [Patil et al., 2023] Patil, S. G., Zhang, T., Wang, X., and Gonzalez, J. E. (2023). Gorilla: Large language model connected with massive apis. arXiv preprint arXiv:2305.15334.
*   [Qian et al., 2023] Qian, C., Cong, X., Yang, C., Chen, W., Su, Y., Xu, J., Liu, Z., and Sun, M. (2023). Communicative agents for software development. arXiv preprint arXiv:2307.07924.
*   [Qin et al., 2023] Qin, Y., Liang, S., Ye, Y., Zhu, K., Yan, L., Lu, Y., Lin, Y., Cong, X., Tang, X., Qian, B., et al. (2023). Toolllm: Facilitating large language models to master 16000+ real-world apis. arXiv preprint arXiv:2307.16789.
*   [Rasal, 2024] Rasal, S. (2024). Llm harmony: Multi-agent communication for problem solving. arXiv preprint arXiv:2401.01312.
*   [Sanh et al., 2021] Sanh, V., Webson, A., Raffel, C., Bach, S. H., Sutawika, L., Alyafeai, Z., Chaffin, A., Stiegler, A., Scao, T. L., Raja, A., et al. (2021). Multitask prompted training enables zero-shot task generalization. arXiv preprint arXiv:2110.08207.
*   [Saunders et al., 2022] Saunders, W., Yeh, C., Wu, J., Bills, S., Ouyang, L., Ward, J., and Leike, J. (2022). Self-critiquing models for assisting human evaluators. arXiv preprint arXiv:2206.05802.
*   [Shen et al., 2023] Shen, C., Cheng, L., You, Y., and Bing, L. (2023). Are large language models good evaluators for abstractive summarization? arXiv preprint arXiv:2305.13091.
*   [Touvron et al., 2023] Touvron, H., Lavril, T., Izacard, G., Martinet, X., Lachaux, M.-A., Lacroix, T., Rozière, B., Goyal, N., Hambro, E., Azhar, F., et al. (2023). Llama: Open and efficient foundation language models. arXiv preprint arXiv:2302.13971.
*   [Wei et al., 2022a] Wei, J., Tay, Y., Bommasani, R., Raffel, C., Zoph, B., Borgeaud, S., Yogatama, D., Bosma, M., Zhou, D., Metzler, D., et al. (2022a). Emergent abilities of large language models. arXiv preprint arXiv:2206.07682.
*   [Wei et al., 2022b] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Xia, F., Chi, E., Le, Q. V., Zhou, D., et al. (2022b). Chain-of-thought prompting elicits reasoning in large language models. Advances in Neural Information Processing Systems, 35:24824–24837.
*   [Wu et al., 2023] Wu, N., Gong, M., Shou, L., Liang, S., and Jiang, D. (2023). Large language models are diverse role-players for summarization evaluation. arXiv preprint arXiv:2303.15078.
*   [Yao et al., 2022] Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., and Cao, Y. (2022). React: Synergizing reasoning and acting in language models. arXiv preprint arXiv:2210.03629.
*   [Zheng et al., 2023] Zheng, L., Chiang, W.-L., Sheng, Y., Zhuang, S., Wu, Z., Zhuang, Y., Lin, Z., Li, Z., Li, D., Xing, E., et al. (2023). Judging llm-as-a-judge with mt-bench and chatbot arena. arXiv preprint arXiv:2306.05685.