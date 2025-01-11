<!--yml
category: 未分类
date: 2025-01-11 12:15:37
-->

# Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task

> 来源：[https://arxiv.org/html/2409.08811/](https://arxiv.org/html/2409.08811/)

Shao Zhang [shaozhang@sjtu.edu.cn](mailto:shaozhang@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina ,  Xihuai Wang [leoxhwang@sjtu.edu.cn](mailto:leoxhwang@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina ,  Wenhao Zhang [wenhao_zhang@sjtu.edu.cn](mailto:wenhao_zhang@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina ,  Yongshan Chen [chenyongshan@sjtu.edu.cn](mailto:chenyongshan@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina ,  Landi Gao [ytlkxglgld@sjtu.edu.cn](mailto:ytlkxglgld@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina ,  Dakuo Wang [d.wang@northeastern.edu](mailto:d.wang@northeastern.edu) Northeastern UniversityBostonUnited States ,  Weinan Zhang [wnzhang@sjtu.edu.cn](mailto:wnzhang@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina ,  Xinbing Wang [xwang8@sjtu.edu.cn](mailto:xwang8@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina  and  Ying Wen [ying.wen@sjtu.edu.cn](mailto:ying.wen@sjtu.edu.cn) Shanghai Jiao Tong UniversityShanghaiChina

###### Abstract.

Theory of Mind (ToM) significantly impacts human collaboration and communication as a crucial capability to understand others. When AI agents with ToM capability collaborate with humans, Mutual Theory of Mind (MToM) arises in such human-AI teams (HATs). The MToM process, which involves interactive communication and ToM-based strategy adjustment, affects the team’s performance and collaboration process. To explore the MToM process, we conducted a mixed-design experiment using a large language model-driven AI agent with ToM and communication modules in a real-time shared-workspace task¹¹1To facilitate the reproducibility of experiments and promote future research in human-AI teams, we will open-source our agent and environment implementation. In compliance with the principle of anonymity, we will include the link to our GitHub repository in the camera-ready version.. We find that the agent’s ToM capability does not significantly impact team performance but enhances human understanding of the agent and the feeling of being understood. Most participants in our study believe verbal communication increases human burden, and the results show that bidirectional communication leads to lower HAT performance. We discuss the results’ implications for designing AI agents that collaborate with humans in real-time shared workspace tasks.

Human-AI Teams, Human-AI Collaboration, AI Agent, Mutual Theory of Mind, Large Language Models, Communication^†^†conference: ; Sep,2024.; ^†^†ccs: Human-centered computing Empirical studies in HCI^†^†ccs: Human-centered computing Collaborative interaction^†^†ccs: Computing methodologies Artificial intelligence^†^†ccs: Computing methodologies Intelligent agents^†^†ccs: Computing methodologies Cooperation and coordination![Refer to caption](img/f122fc041fa83dd674cc676c99834f8e.png)

Figure 1\. The Mutual Theory of Mind (MToM) Process of Human-AI Collaboration in a Shared Workspace. We used scenarios derived from the Overcooked game to illustrate this MToM process. In this example, the human controls the black hat chef, and the agent controls the blue hat chef. Humans and agents act in a shared workspace to complete interdependent tasks, making independent decisions while using the Theory of Mind (ToM) to infer each other’s state. They observe actions as implicit communication and use messages for explicit verbal communication. We label the communication pathways shaped by ToM, as the MToM process influences explicit communication, decision-making, and behavior. Changes in agent behavior affect human inferences and decision-making, and the reverse is also true.

\Description

## 1\. Introduction

AI agent refers to an AI-driven system or entity that can perceive the environment and make decisions based on the perceived information to achieve specific goals independently (Russell and Norvig, [2016](https://arxiv.org/html/2409.08811v1#bib.bib96)). With the advancement of technology, an increasing number of AI agents are being deployed in various scenarios to complete tasks, including medical diagnosis (Vicari et al., [2003](https://arxiv.org/html/2409.08811v1#bib.bib110); Iantovics, [2008](https://arxiv.org/html/2409.08811v1#bib.bib51); Tudor Car et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib109); Zhang et al., [2024b](https://arxiv.org/html/2409.08811v1#bib.bib129)), scientific research (M. Bran et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib73); Aher et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib2); Swan et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib108); Guo et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib42); Zhang et al., [2023b](https://arxiv.org/html/2409.08811v1#bib.bib128)), and industry applications (Baratta et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib9); Harada et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib44); Peng et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib89)). As large language models continue to evolve, the range of tasks that AI agents can handle is expanding, and they are increasingly being applied in scenarios such as home robots (Brohan et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib15); Wu et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib121)), gaming (Liu et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib71); Wang et al., [2024b](https://arxiv.org/html/2409.08811v1#bib.bib111)), and education (Zhang et al., [2024c](https://arxiv.org/html/2409.08811v1#bib.bib133); Sonlu et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib103)). Humans and AI agents collaborate in these scenarios in the same space and can observe each other’s actions, which is considered a shared workspace setting (Salikutluk et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib97); Dourish and Bellotti, [1992](https://arxiv.org/html/2409.08811v1#bib.bib30)). These developments have also led AI agents to transition from independently performing tasks to collaborating with humans and form human-AI teams (HATs), where humans and AI are recognized as unique contributors, role-specific, and working together toward a common goal (O’Neill et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib87)).

Theory of Mind (ToM) is an important ability of human beings to infer mental states, intentions, emotions, and beliefs of others for the dynamic adjustment of behaviors (Premack and Woodruff, [1978](https://arxiv.org/html/2409.08811v1#bib.bib93); Rabinowitz et al., [2018](https://arxiv.org/html/2409.08811v1#bib.bib94)), which has been applied in many AI agent frameworks (Zhang et al., [2024a](https://arxiv.org/html/2409.08811v1#bib.bib130); Wen et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib119); Langley et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib61)). When closely collaborating with humans in the shared workspace setting, ToM plays a more critical role in the AI agent framework (McKenna et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib76); Hiatt et al., [2011](https://arxiv.org/html/2409.08811v1#bib.bib48)). Researchers use ToM to help AI agents understand, infer, and predict human behavior, enabling them to dynamically adjust their strategies to achieve better team performance (Rabinowitz et al., [2018](https://arxiv.org/html/2409.08811v1#bib.bib94); Hiatt et al., [2011](https://arxiv.org/html/2409.08811v1#bib.bib48)). From a human perspective, humans also build mental models of AI agents through the ToM capability in the human-AI collaboration process (Gero et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib39)). Humans typically expect AI agents to align with their anticipated capabilities or roles (Kim et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib59)), and they attribute mental states to AI agents accordingly (Rato et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib95); Shank et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib99)). When human beings are interacting with an agent with ToM capability, Mutual Theory of Mind (MToM) framework, which refers to a constant process of reasoning and attributing states to each other, is considered the analysis of the collaboration process in some studies (Wang et al., [2024a](https://arxiv.org/html/2409.08811v1#bib.bib114); Weisz et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib118); Wang and Goel, [2022](https://arxiv.org/html/2409.08811v1#bib.bib112)).

In the study of human cognitive activities, ToM processes are considered closely related to communication (Meltzoff, [1999](https://arxiv.org/html/2409.08811v1#bib.bib81)). People use their ToM abilities to infer others’ mental states and intentions, which helps them decide whether the communication is necessary and what the content and style of that communication should be (Fussell et al., [1998](https://arxiv.org/html/2409.08811v1#bib.bib38)). This effect is even more pronounced in team collaboration. People determine their plans based on their partner’s movements and verbal communication, and they use communication to coordinate with their partners to achieve alignment (Krych-Appelbaum et al., [2007](https://arxiv.org/html/2409.08811v1#bib.bib60)). The interactivity of communication, which refers to both parties being able to simultaneously send and receive messages (McMillan and Hwang, [2002](https://arxiv.org/html/2409.08811v1#bib.bib77)), might be influenced by the MToM process due to its bidirectional characteristic. Some studies suggest that bidirectional communication is shaped by the MToM process, leading to changes in both parties’ mental models throughout long-term communication (Wang and Goel, [2022](https://arxiv.org/html/2409.08811v1#bib.bib112); Wang et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib113)). The team’s communication process, including both verbal and non-verbal communication, and individual ToM capabilities are interdependent. In such a complex MToM process shown in [Figure 1](https://arxiv.org/html/2409.08811v1#S0.F1 "In Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), it is necessary to explore verbal communication interaction and how both parties engage in their own ToM processes.

Previous research has explored the significant impact of the interactivity of communication on team collaboration, including the communication direction (Ashktorab et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib5)) and style (Julia L. Wright and Chen, [2022](https://arxiv.org/html/2409.08811v1#bib.bib54)). However, since communication plays a role in building human trust (Le Guillou et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib62); Sharma et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib100)) and shaping social perceptions (Ashktorab et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib6); Zhang et al., [2021a](https://arxiv.org/html/2409.08811v1#bib.bib127)), the studies on communication on HATs did not consider that the AI agent has a ToM capability in the HAT. Since agents and humans possess ToM abilities, psychological processes and team communication become more complex when the MToM exists (Wang et al., [2024a](https://arxiv.org/html/2409.08811v1#bib.bib114); Weisz et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib118)). We still lack exploration of the impact of the MToM process on HATs, particularly with regard to how communication interactivity and an agent’s ToM capability influence the cooperation process within a HAT in a shared workspace setting.

In this paper, we aim to investigate the communication and collaboration process in the MToM between humans and AI agents in a shared workspace. We first develop an AI agent equipped with ToM and communication capabilities via LLM. Based on the widely adopted HAT collaboration benchmark, Overcooked (Carroll et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib17); Strouse et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib106); Li et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib66), [2024b](https://arxiv.org/html/2409.08811v1#bib.bib67); Yu et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib125); Wu et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib122)), we implement a practical HAT task in the shared workspace setting. In this task, a human and an AI agent work together in a kitchen to prepare different burgers. In this shared workspace setting, the actions between the human and AI agent have dependencies. Moreover, the real-time requirement of the task further asks the HAT to understand each other’s intentions (Le Guillou et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib62)). Our research employs a mixed experimental design of $4\times 2$ (4 communication level $\times$ 2 agent ToM capability level). This design allows us to thoroughly explore the effects of the agent’s ToM capability and communication interactivity as independent variables on HATs. Specifically, our research questions are as follows:

*   •

    RQ1: How does the MToM process including communication interactivity and individual ToM capabilities influence the overall team performance of HATs?

*   •

    RQ2: How does the MToM process including communication interactivity and individual ToM capabilities affect the team collaboration process?

*   •

    RQ3: How do humans perceive AI teammates in the MToM process?

We conducted an online experiment (n = 68) using the shared workspace collaboration task we designed. The results show that HAT team performance in real-time tasks is affected by communicative interactivity, with performance under bidirectional communication being lower than under other communication interactivity conditions. While the agent’s ToM capability did not significantly impact the team’s objective performance, it did affect the collaboration process. Moreover, participants tend to overlook verbal communication due to the operational burden of sending or receiving messages in the team collaboration process. As for human perception, participants’ understanding of the agent through ToM relied more on the agent’s behavior than verbal communication. When the agent could coordinate its actions with humans, participants tended to believe the agent understood them.

These results indicate that due to the presence of MToM, nonverbal and implicit communication via behaviors can be just as effective as verbal communication for human-AI collaboration in real-time shared workspace tasks. We further discussed humans’ patterns of understanding agents and their perception of agents’ ToM capability to provide information about the human ToM process within MToM. We hope that adopting the MToM perspective to understand human-AI collaboration in shared workspace settings can provide new insights for future AI agent design. We open-source our experiment platform and LLM-driven agent²²2In compliance with the principle of anonymity, we will include the link to the GitHub repository in the camera-ready version. for future study on MToM in human-AI collaboration.

## 2\. Related Works

With continuous technological advancements, AI agents are gradually becoming teammates collaborating with humans in shared workspaces (Christoforou et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib21); Faccio et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib34)). In shared workspace settings, AI agents can independently take on some traditional human tasks and roles, contributing to the team’s success (Carroll et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib17); Devin and Alami, [2016](https://arxiv.org/html/2409.08811v1#bib.bib29); Salikutluk et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib97)). Due to the human and the AI agent being both recognized as a unique, role-specific contributor working collectively towards a common goal, the human and AI agent form a Human-AI Team (HAT) (O’Neill et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib87)). This section presents the related works of Theory of Mind and Communication in Human-AI Teams (HATs) to provide the context of these two factors in HATs research and their relationships. In our study, we consider that communication and individual ToM processes mutually influence each other, and the interactivity of communication is part of the MToM process. We further present the current AI technology related to the agent’s capabilities in communication and ToM to provide the background of the AI agent that was used in our experiments.

### 2.1\. Theory of Mind in Human-AI Teams

Theory of Mind refers to an important ability of human beings to infer mental states, intentions, emotions, and beliefs of others for dynamic adjustment of behaviors (Premack and Woodruff, [1978](https://arxiv.org/html/2409.08811v1#bib.bib93); Baron-Cohen, [1999](https://arxiv.org/html/2409.08811v1#bib.bib11); Baron-Cohen et al., [1985](https://arxiv.org/html/2409.08811v1#bib.bib12)). Using shared plans and goals as the foundation for collaborative task completion, ToM enables humans to recognize and adjust plans to achieve cooperation (Baron-Cohen, [1997](https://arxiv.org/html/2409.08811v1#bib.bib10); Carruthers and Smith, [1996](https://arxiv.org/html/2409.08811v1#bib.bib18)). These ToM processes and outcomes impact human collaborative performance and overall team performance.

The AI community uses ToM to build AI agents that can infer partners’ intention (Wen et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib119); Devin and Alami, [2016](https://arxiv.org/html/2409.08811v1#bib.bib29)), including partner agents and humans (Hiatt et al., [2011](https://arxiv.org/html/2409.08811v1#bib.bib48); Si et al., [2010](https://arxiv.org/html/2409.08811v1#bib.bib102); Dehkordi et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib25)). Many studies suggest that the ToM is a crucial factor for AI in achieving dynamic autonomy adjustment (Salikutluk et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib97)), which significantly impacts team performance (Gero et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib39)). ToM can help improve understanding of which tasks are suitable for which teammates (Pinski et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib92); Steyvers et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib104)) and enable better task allocation among them (He et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib47)), leading to more effective coordination. At the same time, researchers are also studying human perceptions and reactions when machines exhibit behaviors that suggest an understanding of human capabilities (Wang et al., [2024a](https://arxiv.org/html/2409.08811v1#bib.bib114)). From the human perspective, when collaborating with AI, humans with good ToM capability will naturally have a mental model of AI (Gero et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib39); Bansal et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib7); Liang et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib68)). Based on the mental model, people tend to attribute mental states to AI teammates (Pinski et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib92); Shank et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib99); McKenna et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib76)) and expect AI to perform specific roles in interactions that align with their expectations and the mental models of AI teammates (Kim et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib59)).

When a human collaborates with an agent with ToM capability, the mutual Theory of Mind (MToM) framework is considered to analyze the collaboration process (Wang et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib113)). MToM refers to a constant process of reasoning and attributing mental states to each other during interaction when humans and AI have ToM capabilities (Weisz et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib118); Wang and Goel, [2022](https://arxiv.org/html/2409.08811v1#bib.bib112)). In past MToM studies, language communication was used as a cue for understanding (Wang et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib113); Wang and Goel, [2022](https://arxiv.org/html/2409.08811v1#bib.bib112)). However, in shared workspace settings, in addition to verbal communication, there are more actions and task dependencies between individuals, leading individuals to rely on actions and behaviors to understand others’ intentions. We still need to fully understand the MToM process in HATs in shared workspace settings, including greater immediate and extensive dependence on actions and tasks between humans and AI agents.

To understand the MToM process, we use whether the agent has ToM capability to determine whether the MToM exists in HAT. We used LLMs to help the agent understand humans based on their historical actions and messages, allowing the agent to build a belief about the human and adjust its actions accordingly. Additionally, we considered the impact of ToM on communication. In the next section, we discuss the role of communication in HATs and how ToM influences communication.

### 2.2\. Communication in Human-AI Teams

Communication plays a key role in supporting the human-human team collaboration process, including verbal and non-verbal communication (Key and Key, [1980](https://arxiv.org/html/2409.08811v1#bib.bib57); Fussell et al., [1998](https://arxiv.org/html/2409.08811v1#bib.bib38); McNeese and Reddy, [2015](https://arxiv.org/html/2409.08811v1#bib.bib79); Lee et al., [2012](https://arxiv.org/html/2409.08811v1#bib.bib64); Schelble et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib98)). Verbal communication is typically a direct and explicit exchange of information (Jakobson, [1972](https://arxiv.org/html/2409.08811v1#bib.bib52); Zhang et al., [2023a](https://arxiv.org/html/2409.08811v1#bib.bib126)), while non-verbal communication is more implicit and requires understanding based on mental states and additional contextual information (Mehrabian, [2017](https://arxiv.org/html/2409.08811v1#bib.bib80); Argyle, [1972](https://arxiv.org/html/2409.08811v1#bib.bib4)).

Similar to human communication (Lea and Spears, [1992](https://arxiv.org/html/2409.08811v1#bib.bib63); Shen et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib101); Fleming et al., [1990](https://arxiv.org/html/2409.08811v1#bib.bib37)), communication in HATs is also the primary factors influencing human perception on the AI teammates. Many studies indicate that different factors of communication have impacts on humans’ trust on AI agents (Le Guillou et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib62); Sharma et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib100)). Research on the role of communication in HATs has also examined the quantity and frequency of communication (O’Neill et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib87); McNeese et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib78)). Communication in HATs is typically less than in the human-human team (Demir et al., [2016](https://arxiv.org/html/2409.08811v1#bib.bib26)). However, some studies have found that AI agents that disclose more information to humans and provide more explanations via non-verbal communication are not always associated with better decision-making or higher human trust on the AI agent (Le Guillou et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib62)). Communication also influences the social behavior of AI and human preferences (Lu et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib72)). Some studies have explored the verbal and non-verbal communication strategies that humans prefer AI teammates to use in team environments, finding that humans tend to prefer proactive communication from teammates and expect them to respond (Zhang et al., [2021a](https://arxiv.org/html/2409.08811v1#bib.bib127), [2023a](https://arxiv.org/html/2409.08811v1#bib.bib126)).

Since communication is one of the primary means of information exchange within teams, its interactivity has received significant attention (Ashktorab et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib5); Oliver, [2019](https://arxiv.org/html/2409.08811v1#bib.bib84)). Interactive verbal communication is typically considered bidirectional (McMillan and Hwang, [2002](https://arxiv.org/html/2409.08811v1#bib.bib77)), meaning that messages can be sent and received simultaneously (DOWNES and McMILLAN, [2000](https://arxiv.org/html/2409.08811v1#bib.bib31)). Ashktorab et al. ([2021](https://arxiv.org/html/2409.08811v1#bib.bib5)) reports when AI agent responds to human cues, humans perceive the AI agent as more intelligent. In the presence of bidirectional communication, the communication style can affect the objective task load on humans. Specifically, communication styles that involve feedback can increase the human objective task load (Julia L. Wright and Chen, [2022](https://arxiv.org/html/2409.08811v1#bib.bib54)).

Verbal and non-verbal communication is also influenced by the Theory of Mind (ToM) process in human-AI collaboration. In HATs, humans and AI agents typically need to understand the status of their teammates (Jiang et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib53)), as well as what their teammates know and plan (Demir et al., [2017](https://arxiv.org/html/2409.08811v1#bib.bib27)), to facilitate cooperation. Communication is regarded as an important channel to help HATs exchange information (Zhang et al., [2023a](https://arxiv.org/html/2409.08811v1#bib.bib126)), which can be sent in verbal and non-verbal ways. Humans use ToM to infer the mental states and intentions of others, which directly affects whether they choose to communicate, the emotional tone of the communication, the way of communication, and the content of the communication. The amount of communication in HATs is positively correlated with stronger shared mental models, which leads to better team performance (Demir et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib28)). In current HATs research, studies indicates how an agent’s ToM abilities can enhance its social attractiveness in communication tasks (Peters, [2006](https://arxiv.org/html/2409.08811v1#bib.bib90); Benninghoff et al., [2013](https://arxiv.org/html/2409.08811v1#bib.bib13); McKenna et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib76)). In studying the MToM process, the mutual influence between communication and individual ToM processes makes communication an essential factor that cannot be overlooked in the MToM process (Wang and Goel, [2022](https://arxiv.org/html/2409.08811v1#bib.bib112)).

Unlike previous studies focuses more on verbal communication with ToM, we considered both verbal and non-verbal communication processes in the MToM framework. In our study, we explore how communicative interaction affects the MToM process. We examined HAT scenarios with bidirectional communication, unidirectional communication, and no communication. Given the inherent presence of implicit non-verbal communication, our between group conditions are on verbal exchanges between humans and AI. We also studied non-verbal communication by manipulating the agent’s ToM abilities as a within-subject factor, assessing how the human’s ToM perceives the agent and affects communication and decision-making. Similarly, we integrated communication into the agent’s ToM control framework. The agent’s ToM considers both human actions and language, shaping its communication and decisions.

### 2.3\. LLM-driven Theory of Mind AI Agents

Considering the Theory of Mind’s (ToM) importance in human-human interaction and collaboration, the AI community has developed many AI agent frameworks based on ToM to improve AI agents’ capability to interact with humans and other agents (Wen et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib119); Langley et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib61)). In multi-agent system research, researchers primarily focus on how to construct agents capable of collaborating with any unseen partners (including humans) as generalization problems (Wang et al., [2024c](https://arxiv.org/html/2409.08811v1#bib.bib116); Albrecht and Stone, [2018](https://arxiv.org/html/2409.08811v1#bib.bib3)). Considering the problem as the Ad Hoc teamwork scenario (Albrecht and Stone, [2018](https://arxiv.org/html/2409.08811v1#bib.bib3)), researchers use many methods based on ToM to help agents adapt to different partners, including intention sharing (Guan et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib41)), classifying types of human teammates (Zhang et al., [2023c](https://arxiv.org/html/2409.08811v1#bib.bib132)), using latent variables for inference (Ma et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib75)), and employing human behavior data to model humans (Carroll et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib17)). Most human modeling approaches assume that human behavior patterns are static and unchanging, leading to limitations in the AI agent’s ToM process, as it cannot make real-time inferences about humans (Lindner and El-Assady, [2022](https://arxiv.org/html/2409.08811v1#bib.bib70)).

In the past, agents built using reinforcement learning (RL) could only achieve team coordination through non-verbal communication (Zhang et al., [2021b](https://arxiv.org/html/2409.08811v1#bib.bib131); Le Guillou et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib62)) or network parameter sharing between agents (Wang et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib115)), making it difficult to communicate with humans. Some works have attempted to enable language communication with the RL agents with language model (Oralbayeva et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib86); Brohan et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib15); † et al.(2022), [FAIR](https://arxiv.org/html/2409.08811v1#bib.bib35)), but still facing the grounding problem (Kenthapadi et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib56)). This issue also made it challenging to study communication processes in many HATs research using real AI technology (Zhang et al., [2023a](https://arxiv.org/html/2409.08811v1#bib.bib126); Demir et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib28)). With the emergence of large language models (LLMs), AI agents’ communication capabilities have improved significantly. The advent of techniques such as Chain of Thought (CoT) (Wei et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib117)) and ReAct (Yao et al., [2022b](https://arxiv.org/html/2409.08811v1#bib.bib124)) has further advanced the capabilities of LLMs in reasoning and decision-making (Wang et al., [2024b](https://arxiv.org/html/2409.08811v1#bib.bib111)). Autonomous agents driven by LLMs are beginning to emerge, capable of performing real-world tasks, such as gaming (Wang et al., [2024b](https://arxiv.org/html/2409.08811v1#bib.bib111); Bara et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib8); Gong et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib40)), online shopping (Ma et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib74); Feng and Feng, [2024](https://arxiv.org/html/2409.08811v1#bib.bib36); Yao et al., [2022a](https://arxiv.org/html/2409.08811v1#bib.bib123)), and housekeeping (Han et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib43); Li et al., [2024a](https://arxiv.org/html/2409.08811v1#bib.bib65)). Many conversational agents have also incorporated ToM into their frameworks to help understand human intentions (Bara et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib8)), thus improving the user experience (Wester et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib120); Wang et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib113)). Since simulations involving multiple LLMs cooperating have confirmed the LLMs’ ability to understand human behavior and adjust their actions accordingly (Park et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib88)), LLMs have also been used to adapt human behavior via ToM (Strachan et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib105); Bara et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib8); Wester et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib120); Sumers et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib107)).

Many studies propose frameworks for LLM-driven agents to facilitate the collaboration between LLM-driven agents and humans. MindAgent (Gong et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib40)) constructs an LLM-based agent to achieve multi-agent coordination in Minecraft and cooperation with humans in a kitchen environment. Liu et al. ([2024](https://arxiv.org/html/2409.08811v1#bib.bib71)) used two LLMs to control the agent’s slow-mind and fast-mind thinking, enabling real-time communication and cooperation with humans in a simulated Overcooked game. ToM capabilities enabled by LLMs have also been used for competitive operations in board games, helping AI agents achieve victory in the poker game (Zhang et al., [2024a](https://arxiv.org/html/2409.08811v1#bib.bib130)).

These technological foundations provide a viable pathway for real AI agents to be involved with communication and ToM capabilities in HAT research. In our study, we build real LLM-driven AI agents based on GPT-4o mini (OpenAI, [2024](https://arxiv.org/html/2409.08811v1#bib.bib85)) with communication and ToM capabilities to experiment with humans. Our study can serve as a complement to previous research that used Wizard-of-Oz (Zhang et al., [2023a](https://arxiv.org/html/2409.08811v1#bib.bib126); Cooke et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib22)) or rule-based agents (Salikutluk et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib97)), by exploring the performance and impact of real AI agents.

## 3\. Cooperative Task Setup and Formalization in Shared Workspace

To better validate the impact of MToM and communication on the cooperation process in HATs, we designed a shared cooperative workspace and tasks based on Overcooked. Overcooked is an important simulation environment used for human-AI collaboration (Carroll et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib17); Strouse et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib106); Li et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib66), [2024b](https://arxiv.org/html/2409.08811v1#bib.bib67); Yu et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib125); Wu et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib122)) derived from the Overcooked video game³³3[https://www.team17.com/games/overcooked/](https://www.team17.com/games/overcooked/).

In this section, we introduce our environment layout design, the definition of cooperative tasks, the communication system design, and the specific metrics we use to measure the cooperation process. The environment layout design and task design are closely related to MToM and the interactivity of communication. We explain the specific connections between the design and MToM, as well as communication. We also formalize the task process to help understand the design of the agent and our experimental procedure.

### 3.1\. Layout Design

In our study, we set the shared workspace as a kitchen. Since we focus on the MToM process, especially the communication and individual ToM process in the HAT cooperation process, we arrange the space so that both parties must coordinate their movements to avoid collisions. The layout requires communication and action coordination within the team. We redesign the basic layout Counter Circuit from the original overcooked-ai environment (Carroll et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib17)), and implement it in the gym-cooking enviroment (Wu et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib122)). The layout features a ring-shaped kitchen with a central counter, elongated counter, and a circular path between the counter and the operational area, which is simulating a real kitchen. In this configuration, pots, cutting board, ingredients (bread, beef, and lettuce), and serving spots are positioned in four distinct directions within the operational area, as shown in Figure [2](https://arxiv.org/html/2409.08811v1#S3.F2 "Figure 2 ‣ 3.1\. Layout Design ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). Although the layout does not forcibly require cooperation, players may find themselves obstructed by narrow aisles, prompting the need for coordination to maximize performance. At the same time, the circular space creates potential movement patterns (moving clockwise or counterclockwise), which can be used by humans or agents with ToM to infer their teammates’ behavior and help facilitate coordination.

![Refer to caption](img/7f46d8afbc8c8807cbe45b60c95eeb4f.png)

(a) The Layout of the Cooperative Workspace as a Kitchen.

![Refer to caption](img/86b7b62d6962903644d6fb06b4ee3f55.png)

(b) Communication System in Our Task.

Figure 2\. Game Layout and Communication System of the Task.

### 3.2\. Task Design

Since we aim to explore the MToM process, we incorporate a mechanism in the burger-cooking task that requires communication for coordination. Ingredients for the burgers overlap, and without explicit indication, it is difficult for individuals to infer which burger their teammate is working on through a single action. Therefore, such a task design requires the players to communicate and infer the teammate’s intention on the ongoing task to prevent redundant work and effectively complete orders. As shown in Figure [3](https://arxiv.org/html/2409.08811v1#S3.F3 "Figure 3 ‣ 3.2\. Task Design ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), we design three burgers: LettuceBurger, BeefBurger, and BeefLettuceBurger. Humans and agents jointly control the chefs and handle the continuously incoming burger orders in our environment. The task provides three ingredients for cooking burgers: bread, beef, and lettuce. The lettuce needs to be chopped, and the beef needs to be cooked. All ingredients are assembled into a burger on a plate, with no specified assembly order. A LettuceBurger requires bread and chopped lettuce, a BeefBurger requires bread and well-cooked beef, and a BeefLettuceBurger requires bread, chopped lettuce, and well-cooked beef. Each order is only available for a limited time, indicated by a countdown on the interface. To coordinate their teamwork effectively, both the human and the agent need to pay attention to the remaining time of each order.

In addition to the cooperation, we introduce additional failure events that will damage performance in the cooperation process. If well-cooked beef is not promptly removed using a plate during cooking, the pan will catch fire. The pan becomes unusable until a team member uses a fire extinguisher to put out the fire. Overcooked beef must be removed using a plate; otherwise, the pan will remain occupied by the overcooked beef, rendering it unusable for further cooking. This process does not directly result in a score penalty (i.e., there is no explicit punishment), but it will impact the team’s cooperation process.

![Refer to caption](img/8223b52fcaa6142d2863bc2ff00f7b68.png)

(a) LettuceBurger

![Refer to caption](img/606a1c492053b8d516feb3d60f15dc34.png)

(b) BeefBurger

![Refer to caption](img/9b70439d0352924995be3e9a55035b83.png)

(c) BeefLettuceBurger

![Refer to caption](img/b250b8475891b82ffb35fe52d8b5ab76.png)

(d) Overcooked Beef

Figure 3\. Game Mechanism. (a), (b), and (c) are the rules for preparing and serving burgers. (d) demonstrates the mechanism of overcooked beef and the rules for handling the fire caused by overcooked beef.

### 3.3\. Communication System Design

We design a communication system for HATs within the task to control the interactivity of communication. In the game, we implement a dialogue module where humans could click buttons to send messages in a dialog box. The buttons in the communication system (as shown in Figure [2](https://arxiv.org/html/2409.08811v1#S3.F2 "Figure 2 ‣ 3.1\. Layout Design ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task")) are designed to represent the items needed for the task, including burgers, all ingredients, and plates. The needs for all items are expressed as “We need (specific item)” to the agent. Additionally, we include options for handling special situations (such as extinguishing fires). Beyond task-related information, there are also two buttons to express emotions. The human participants can see messages sent by the agent in the same dialog box.

### 3.4\. Formulation

We formulate this scenario as a two-player decentralized Markov decision process (DEC-MDP) (Bernstein et al., [2002](https://arxiv.org/html/2409.08811v1#bib.bib14)). The DEC-MDP containing one agent and one human can be formalized as $<\mathcal{S},\{\mathcal{A}^{i}\},\{\mathcal{A}^{h}\},\rho,\mathcal{T},r>$, where $\mathcal{S}$ is the state space, $\rho:\mathcal{S}\mapsto[0,1]$ is the distribution of the initial state $s_{0}$. $\mathcal{A}^{i}$ and $\mathcal{A}^{h}$ are the action spaces of the agent and the human, and $\mathcal{A}=\mathcal{A}^{i}\times\mathcal{A}^{h}$ is the joint action space. $\mathcal{T}:\mathcal{S}\times\mathcal{A}\times\mathcal{S}\mapsto[0,1]$ denotes the transition probability and $r:\mathcal{S}\times\mathcal{A}\mapsto\mathbb{R}$ is the reward function. At time step $t$, the agent and the human take action $a^{i}_{t}$ and $a^{h}_{t}$ simultaneously.

State. Both the agent and the human have full access to the game states and each other’s actions. Players can directly see the status of all items in the game interface, such as the location where items are placed and their current state (e.g., beef cooking in a pan). Players can also view the remaining game time and current score through the information displayed. The remaining time for each order, the progress of chopping lettuce, the process of cooking beef, and the process of extinguishing a fire are shown through progress bars. All actions taken by teammates, the teammates’ location, and the items they are holding are fully visible to each other. Players can also view messages in real-time, including both the messages they have sent and the messages they have received, along with the corresponding timestamps.

Action. In this environment, the actions that the human and the agent can take to control the chefs include moving up, down, left, and right, as well as “interact”. All activities such as picking up items, serving dishes, and extinguishing fires are considered as “interact” actions. The specific interaction rules are illustrated in Figure [3](https://arxiv.org/html/2409.08811v1#S3.F3 "Figure 3 ‣ 3.2\. Task Design ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). We denote the actions to control the chefs as $\mathcal{A}^{\text{control}}$. The agent and the human share the same $\mathcal{A}^{\text{control}}$, while the communication actions are designed individually for them, denoted as $\mathcal{A}^{\text{comm},i}$ and $\mathcal{A}^{\text{comm},h}$ respectively. The communication action space for the agent $\mathcal{A}^{\text{comm},i}$ consists of any possible sentences less than 10 words from an LLM, while the communication action space for the human $\mathcal{A}^{\text{comm},h}$ consists of 11 message templates. Details about the communication system are presented in [Section 3.3](https://arxiv.org/html/2409.08811v1#S3.SS3 "3.3\. Communication System Design ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task") and [Section 4.3](https://arxiv.org/html/2409.08811v1#S4.SS3 "4.3\. Communication Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task").

Reward. The scores for completing the three different types of orders vary and serving the wrong burger or missing an order will result in a penalty. The specific rewards are detailed in [Table 1](https://arxiv.org/html/2409.08811v1#S3.T1 "In 3.4\. Formulation ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task").

Table 1\. Rewards in Game.

| Event | Rewards |
| --- | --- |
| Serve a LettuceBurger | +15 |
| Serve a BeefBurger | +20 |
| Serve a BeefLettuceBurger | +25 |
| Serve a Wrong Burger (or Something not a Burger) | -10 |
| Miss an order | -10 |

### 3.5\. Objective Metrics

To capture the dynamics of the cooperation process and observe the impact of MToM and communication on collaboration, we defined a set of task-related metrics to measure team performance and the team collaboration process.

Task Score. We define the team’s objective performance as the team’s score. The game is set to be completed in 500 time-steps, and the specific reward calculation method is shown in Table [1](https://arxiv.org/html/2409.08811v1#S3.T1 "Table 1 ‣ 3.4\. Formulation ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task").

Contribution Rate. We first define key task events $KE$ to capture which team member completes specific tasks. Based on the burger-making process, each of the three types of burgers involves certain essential and non-repeatable events. For example, making a BeefBurger consists of 5 key events: Cook Beef, Use Beef, Use Bread, Use Plate, and Serve, and each key event will only be completed once. The completion of key events is marked by specific “interact” actions, which we define as Key Actions. We map the key actions with the key events in [Table 2](https://arxiv.org/html/2409.08811v1#S3.T2 "In 3.5\. Objective Metrics ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). In this mapping, each key event completed by the player is counted once as the player’s contribution to the overall performance. Based on these non-repeatable key events, we can attribute the contribution to the two players by counting the key events they completed in preparing each properly delivered burger. We define the agent’s contribution ratio $CR^{i}$ as: $CR^{i}=\frac{KE^{i}}{KE^{i}+KE^{h}}\times 100\%$, where $KE^{i}$ and $KE^{h}$ represent the key events completed by the agent and the human respectively.

Table 2\. The mapping from key event to key actions.

 | Key Events | Key Actions |
| --- | --- |
| Cook Beef | ① Get Beef from station Put onto Pan |
| Use Beef | ① Plate well-done Beef from Pan |
| Prepare Lettuce | ① Get lettuce from station ② Put onto Cutboard ③ Chop Lettuce |
| Use Lettuce | ① Plate Lettuce Done from Cutboard ② Plate Lettuce Done from Counter ③ Put onto Plate with BeefBurger |
| ④ Put onto Plate with Bread ⑤ Put Lettuce onto Plate ⑥ Put Lettuce onto Plate with Beef |
| Use Bread | ① Get Bread from Station ② Plate Bread from Counter ③ Put onto Plate with BeefLettuce |
| ④ Put onto Plate with Lettuce ⑤ Put Bread onto Plate ⑥ Put Bread onto Plate with Beef |
| Use Plate | ① Get Plate from Station |
| Serve | ① Deliver Burger | 

Message Count. To study the impact of MToM on the communication process, particularly human communication preferences, we record the types and quantities of messages humans choose to send when they can communicate. We count the number of sent messages as Message Count.

Failure Count. We use cooperation failures in the task as an auxiliary analysis of team performance. We categorize three events as cooperation failures: missing an order, serving a burger or other item not required by the current orders, and the pan catching fire. We separately record the occurrence of each event and summarize them as Failure Count.

## 4\. LLM-driven Agent with Theory of Mind and Communication Module

This section introduces our agent implementation based on GPT-4o mini (OpenAI, [2024](https://arxiv.org/html/2409.08811v1#bib.bib85)). As shown in Fig. [4](https://arxiv.org/html/2409.08811v1#S4.F4 "Figure 4 ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), we design three main modules for the agent, including Theory of Mind, Policy, and Communication. The Theory of Mind module infers the intentions of human partners based on human behavior, summarizing these behaviors to guide further the agent’s strategies for cooperating better with humans. The Policy module controls how the agent interacts in our environment and continually updates the agent’s behavior policy to improve performance. The Communication module helps the agent receive human messages to adjust the agent’s behavior and send messages to the human.

![Refer to caption](img/c219cfe67b4446716276736c230154e5.png)

Figure 4\. Agent Framework. The framework shows how the LLM-driven agent with Theory of Mind and communication capability takes action and sends messages to the human player. We use a history buffer to save the game history, including game state, actions, and player messages. The Theory of Mind module uses history as input to analyze human behavior. The Policy and Message modules also have the history input to understand the whole picture of the game. We summarize the process of generating action and message: (1) The Theory of Mind module analyzes human behavior and messages, then generates beliefs about humans and provides a guide for adjusting the strategy for better team coordination and communication. (2) The Policy module uses the belief from the Theory of Mind module and the history to improve the agent’s strategy by continually updating behavior guidelines. It outputs an action to control the agent. (3) The Message module uses the history, the inferred belief from the Theory of Mind module, and the guidelines from the Policy module to generate the message that aligns with the agents’ actions and intentions.

### 4.1\. Theory of Mind Module

With a ToM capability, individuals can form hypotheses about the mental states of others as a belief through their actions and communication history, thus understanding and predicting others’ behaviors (Premack and Woodruff, [1978](https://arxiv.org/html/2409.08811v1#bib.bib93)). Belief in Theory of Mind refers to an individual’s cognition of things, which further influences their behavior (Baron-Cohen et al., [1985](https://arxiv.org/html/2409.08811v1#bib.bib12); Rabinowitz et al., [2018](https://arxiv.org/html/2409.08811v1#bib.bib94); Wen et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib119)). For example, in our Overcooked environment, if the human player believes that “my partner will cook the beef,” the human player might focus on other tasks, assuming that their partner will take care of the beef. Based on the Theory of Mind mechanism, we design the Theory of Mind module, which enables the agent to establish the belief about the human, including the tendency, convention, and plan, using the human partner’s behavioral history and communication messages.

Based on the formulations in [Section 3.4](https://arxiv.org/html/2409.08811v1#S3.SS4 "3.4\. Formulation ‣ 3\. Cooperative Task Setup and Formalization in Shared Workspace ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), we denote the history from time-step $0$ to time-step $t$ of the game that the agent perceives as:

|  | $\displaystyle\mathcal{H}_{0:t}=\{$ | $\displaystyle(s_{0},a_{0}^{\text{control},i},a_{0}^{\text{control},h},a_{0}^{% \text{comm},i},a_{0}^{\text{comm},h},r_{0}),$ |  |
|  |  | $\displaystyle\quad\quad\quad\quad\quad\quad\quad\quad\quad\vdots$ |  |
|  |  | $\displaystyle(s_{t},a_{t}^{\text{control},i},a_{t}^{\text{control},h},a_{t}^{% \text{comm},i},a_{t}^{\text{comm},h},r_{t})\}.$ |  |

As shown in [Figure 4](https://arxiv.org/html/2409.08811v1#S4.F4 "In 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), the Theory of Mind module uses the history $\mathcal{H}_{0:t}$ as an input to summarize the history, infers the conventions and tendencies of the human and explains how the agent’s policy can be adjusted to coordinate better with the human player. The Theory of Mind module outputs the belief in natural language, as shown in [Figure 5](https://arxiv.org/html/2409.08811v1#S4.F5 "In 4.1\. Theory of Mind Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). The entire process can be formalized as follows:

| (1) |  | $b^{n}=\text{LLM}\left(\mathcal{H}_{0:t_{n}},b^{n-1}\right)~{},$ |  |

where $t_{n}$ is the time-step when the $n$-th belief inferences is performing. The Theory of Mind module executes every $75$ time-steps.

![Refer to caption](img/4e593451099a79c47a41b314b55d53d3.png)

Figure 5\. An example of Belief Output from the Theory of Mind Module.

The core prompts used in the Theory of Mind module are shown in [Figures 6](https://arxiv.org/html/2409.08811v1#S4.F6 "In 4.1\. Theory of Mind Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task") and [7](https://arxiv.org/html/2409.08811v1#S4.F7 "Figure 7 ‣ 4.1\. Theory of Mind Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). To implement the within-group condition in the experiment, i.e., whether the agent has ToM capabilities, we implement the Theory of Mind module as optional, and we can control whether the ToM results are included in the decision-making process, as shown in [Figure 4](https://arxiv.org/html/2409.08811v1#S4.F4 "In 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). When the condition is that the agent has no ToM capabilities, the ToM inference process ([Equation 1](https://arxiv.org/html/2409.08811v1#S4.E1 "In 4.1\. Theory of Mind Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task")) will not be executed, and no ToM results are used in the Policy module and the Communication module.

![Refer to caption](img/1c54433e783bab3ded1313f2d12f4954.png)

Figure 6\. Prompt for Theory of Mind Module.

Considering the communication conditions in our experiments, we use two variables, INFER_HUMAN_WITH_SEND
_MESSAGE and INFER_HUMAN_WITH_RECEIVE_MESSAGE to control whether the agent can send message to human and whether the AI agent can use the received message to infer human in Theory of Mind module, as shown in [Figure 7](https://arxiv.org/html/2409.08811v1#S4.F7 "In 4.1\. Theory of Mind Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). For example, if the conditions are set so that both human and agent cannot send messages and the agent has ToM capability, the INFER_HUMAN_WITH_SEND_MESSAGE and INFER_HUMAN_WITH_RECEIVE_MESSAGE will be set to empty strings to make the agent do not consider the human message and the human response to the agent’s messages in the ToM process.

![Refer to caption](img/9f5afac56826c275027fd0ccfb6417fb.png)

Figure 7\. Prompts as Variables in the Theory of Mind Module. Top is INFER_HUMAN_WITH_SEND_MESSAGE and button is INFER_HUMAN_WITH_RECEIVE_MESSAGE.

### 4.2\. Policy Module

Our task is time-sensitive, requiring the agent and human player to make decisions and adjust their policies in time to avoid missing orders and overcooking the dishes. Due to the real-time requirement, the agent can not directly prompt the LLM to generate its next action, which results in unacceptable latency. Besides, LLMs are better at high-level reasoning and planning instead of generating detailed control signals (Wei et al., [2022](https://arxiv.org/html/2409.08811v1#bib.bib117)). Therefore, the agent leverages the LLM to generate high-level plans, which we call “macro actions.” Meanwhile, the agent prompts the LLM to reflect and update some behavior guidelines iteratively. The agent then transforms the generated plans into actable actions, including moving and interacting with objects, which we call “atomic actions,” through an action executor.

As shown in [Figure 4](https://arxiv.org/html/2409.08811v1#S4.F4 "In 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), the Policy module consists of three parts: Code-as-Policy Generator, Policy Reflection, and Action Executor.

#### 4.2.1\. Code-as-Policy Generator

Given the latency in API calls with GPT-4o mini and the real-time decision-making requirements of our task, we implement an initial policy with predefined rules. This policy is further empowered by a code-as-policy (Liang et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib69)) generator, which plays a crucial role in enabling the agent to effectively handle real-time emergencies. Our initial policy is structured upon a Finite-State Machine (FSM) framework. This framework is instrumental in allowing the agent to operate within a set of defined states, transitions, and actions, providing a clear structure for its operations. Each state represents the agent’s particular context or situation, while environment dynamics, fixed 25-time-step intervals, and human message input trigger transitions. The FSM structure allows the agent to complete tasks by switching between predefined states, allowing it to function without relying on external API responses.

However, it is impractical to exhaustively define all possible states and transitions using a finite-state machine-based rule policy, especially in complex and dynamically changing environments. To address this limitation, we incorporate a LLM to enhance the policy. When the FSM falls short in covering specific complex or unexpected scenarios, such as human messages, the LLM can generate flexible and temporary strategies, allowing the system to handle unforeseen situations. The LLM generates JSON codes to optimize the initial policy, given the recent game history and the inferred belief about the human. As demonstrated in [Figure 8](https://arxiv.org/html/2409.08811v1#S4.F8 "In 4.2.1\. Code-as-Policy Generator ‣ 4.2\. Policy Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), the LLM generates two types of code snippets: 1) Condition and Macro Action: The macro action will be executed if the condition is satisfied, where the condition is used to avoid outdated decisions caused by latency; 2) Order: The agent will prioritize this order. The macro action generation process can be formalized as:

|  | $\displaystyle\tau_{t}$ | $\displaystyle=\text{LLM}\left(\mathcal{H}_{t-\lambda:t},b^{n}\right)$ |  |
|  | $\displaystyle ma_{t}$ | $\displaystyle=\pi^{m}\left(s_{t},\tau_{t}\right)~{},$ |  |

where $b^{n}$ represents the latest belief about human that is updated $n$ times, $\pi^{m}$ represents the latest agent policy that is updated $m$ times, $\tau_{t}$ is the generated code-as-policy and $\lambda$ is the interval the code-as-policy generator executes, which is $25$ in our experiment.

![Refer to caption](img/3d5da2e74e95b44ef31026458f0e7ad9.png)

Figure 8\. Generated Code-as-Policy Example.

#### 4.2.2\. Policy Reflection

![Refer to caption](img/cc2135fbcdeaf4a0b880d4d333db7d83.png)

Figure 9\. Prompt for Code-as-Policy Generator.

Equipped with the FSM-based initial policy and Code-as-Policy generator, the agent already has the basic capability to interact with humans in our environment. However, the agent needs to improve its policy in such a long-horizon interaction process for higher performance. Using the low-overhead in-context learning approach, the agent maintains and iteratively updates a “Behavior Guideline,” which summarizes the improvements to current policy. After a “Behavior Guideline” is generated, the agent incorporates the “Behavior Guideline” into the current policy. The entire process can be formalized as:

|  | $\displaystyle\mathcal{B}^{m}$ | $\displaystyle=\text{LLM}\left(\mathcal{H}_{0:t_{m}},b^{n},\mathcal{B}^{m-1}% \right),$ |  |
|  | $\displaystyle\pi^{m}$ | $\displaystyle=\pi^{m-1}\cup\mathcal{B}^{m}~{},$ |  |

where $m$ means the reflection process executes $m$ times, $b^{n}$ is the latest inferred belief about human, $\mathcal{B}^{m}$ is the “Behavior Guideline” that is updated $m$ times.

The core prompts for the reflection process are shown in [Figure 10](https://arxiv.org/html/2409.08811v1#S4.F10 "In 4.2.2\. Policy Reflection ‣ 4.2\. Policy Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), and an output example is shown in [Figure 11](https://arxiv.org/html/2409.08811v1#S4.F11 "In 4.2.2\. Policy Reflection ‣ 4.2\. Policy Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task").

![Refer to caption](img/a560d1c707119ef48595c72c9deb4bac.png)

Figure 10\. Prompt for Policy Reflection.

![Refer to caption](img/027634083773ea2b75fd5819debfdb4f.png)

Figure 11\. An example of Behavior Guidelines from the Reflection Module.

#### 4.2.3\. Action Executor

The actions output by the policy module are macro actions. These actions are then executed by an action executor, transforming them into atomic actions that can be executed directly in the environment. The action executor employs script policies to covert macro actions. The macro actions included are summarized below.

*   •

    Prepare:

    *   –

        Valid Objects: “Beef”, “Lettuce”, “Bread”

    *   –

        Function: Prepare an appointed ingredient until it can be used to assemble.

*   •

    Assemble:

    *   –

        Valid Objects: “BeefBurger”, “LettuceBurger”, “BeefLettuceBurger”

    *   –

        Function: Assemble an appointed burger if all necessary ingredients are ready.

*   •

    Pass on:

    *   –

        Valid Objects: “Plate”, “Bread”

    *   –

        Function: Put the object onto the center counters to deliver it to the partner.

*   •

    Serve:

    *   –

        Valid Objects: “BeefBurger”, “LettuceBurger”, “BeefLettuceBurger”

    *   –

        Function: Deliver an assembled burger to the customer.

*   •

    Putout Fire:

    *   –

        Valid Objects: -

    *   –

        Function: Pick up the fire extinguisher and put out the fire, if any.

Given a macro action, the action executor chooses a possible plan and performs path planning. The path plan is implemented using the A-star algorithm (Hart et al., [1968](https://arxiv.org/html/2409.08811v1#bib.bib46)). The process can be formalized as:

|  | $a_{t}^{\text{control},i}=\text{Executor}\left(ma_{t}\right)~{}.$ |  |

### 4.3\. Communication Module

Communication is the most direct means for human teams to express their intentions. The message sent within the team is the most flexible content for individuals to adjust. To avoid affecting agent autonomy and to allow dynamic adjustment of communication, we do not require the agent to communicate continuously. Instead, the agent is required to autonomously decide whether communication is necessary and determine the content of the communication. The communication message generation process can be present as:

|  | $a_{t}^{\text{comm},i}=\text{LLM}\left(\mathcal{H}_{t-\eta:t},b^{n},\mathcal{B}% ^{m}\right)~{},$ |  |

where $b^{n}$ and $\mathcal{B}^{m}$ are the latest belief about humans and the latest “Behavior Guideline” respectively, and $\eta$ is the interval for the communication process to execute, which is set as $25$ in our experiment. The core prompt for generating communication is shown in [Figure 12](https://arxiv.org/html/2409.08811v1#S4.F12 "In 4.3\. Communication Module ‣ 4\. LLM-driven Agent with Theory of Mind and Communication Module ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). We implement different communication conditions between groups in the experiment: the communication process will not be executed and will not output any message in scenarios where the agent cannot communicate with the human.

![Refer to caption](img/bf89f4c4fb656248ee79bbb59939e5d5.png)

Figure 12\. Prompt for Communication Module.

Additionally, under conditions where communication is enabled, each message emergence from humans triggers the communication process to ensure that humans’ explicit communicative intentions are clearly captured.

### 4.4\. Validation of The Agent Framework

We conducted a validation experiment to validate the capabilities of the LLM-based agent we designed and to understand the impact of the ToM capability on agent performance. We use an agent with only a policy module as the fixed teammate. Two types of agents, one with a ToM module and one without, each plays $10$ games with a fixed rule-based teammate using the same task setup as in our mixed-design experiment. Each game lasts for 500 time steps. Since the fixed agent teammate lacks communication capabilities, we do not include the communication module in validating the agent’s performance.

The agent with the Theory of Mind module has an average score of $136$ (Standard Deviation = $25.77$), and the agent without the Theory of Mind module has an average score of $115.5$ (Standard Deviation = $30.04$). These results conclusively demonstrate that our implemented agent framework effectively completes the task we designed in the shared workspace setting, and the Theory of Mind module significantly enhances performance.

## 5\. Methods

To investigate MToM’s effects in HATs, especially the effects of communication interactivity and the agent’s ToM capability, we designed a 4x2 mixed-design experiment. The communication interactivity factor includes four levels as between-group variables: bidirectional communication (Bi-Comm), human-only message sending (H-Comm), agent-only message sending (A-Comm), and no communication (No-Comm). We set the agent’s ToM ability as a within-subject condition.: agent with ToM (w/ ToM) and agent without ToM (w/o ToM). This within-subject condition directly affects whether MToM exists. When the agent does not have ToM abilities, the MToM process becomes a one-way ToM process, where only the human possesses ToM capabilities.

### 5.1\. Procedure

We recruited participants from the university via the university’s internal social platform. Each participant received 50 RMB for their participation. We gave the participants bonuses based on their performance to motivate them to be more engaged and attentive. We ranked participants in each group based on their self-play performance and performance in two different agent games. The top 25% in each condition within a group received an additional bonus of 5 RMB. The bonus could be accumulated for each condition, allowing for a maximum bonus of 15 RMB. The experiments were conducted online, where each participant completed the experiments on a certain web page using a computer with a keyboard and a mouse, and each experiment took about 20 minutes. Participants controlled the chef using arrow keys and interacted with objects using the spacebar on the keyboard. They could issue messages to the agent by clicking buttons with the mouse. We did not require humans to send messages in the Bi-Comm and H-Comm groups. The participant were free to decide whether to send the message or not. And participants were also free to decide whether to adopt the agent’s messages in A-Comm and Bi-Comm groups. We recorded the entire experimental process and provided playback support for data validation.

Since we used a mixed experimental design, with the between-group condition Communication (includes four levels: Bi-Comm, H-Comm, A-Comm, and No-Comm), participants were randomly assigned to one of the four groups, with 20 participants in each group. Each participant was exposed to two different agents (i.e., the agents with and without a ToM capability) within a group, with three trials conducted for each agent, totaling six trials. The experiment was conducted in both the participants’ native languages, Chinese and English. Specifically, to investigate whether humans could perceive the agent’s ToM, participants were not informed of the agent’s specific capabilities; they were only told that there were two types of agents in the experiment, distinguished by color.

All participants first completed an informed consent form and read instructions about the game rules, game operations, and communication methods. After the instructions, participants underwent a non-scored trial to familiarize themselves with the environment, rules, and operations, followed by a scored trial to assist with data validation. In the formal experiment, after each trial, we asked participants to complete a questionnaire in which we collected their perceptions of the agent and the team collaboration. The items are mostly based on Hoffman ([2019](https://arxiv.org/html/2409.08811v1#bib.bib49)). The questionnaire as shown in [Appendix A](https://arxiv.org/html/2409.08811v1#A1 "Appendix A The Detailed Results of Questionnaires in Each Condition ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task") used a 5-point Likert scale to indicate the level of agreement with the statements. After all six trials, participants were asked to complete another questionnaire, where we collected their perceptions of the differences between the two agent behaviors and their preferences for the agent. The questionnaire also included open questions about the communication process, as shown in [Appendix B](https://arxiv.org/html/2409.08811v1#A2 "Appendix B The Open Questions in Each Condition ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task").

### 5.2\. Participants

A total of 80 participants participated in the study. The first and second authors of the article independently validated all collected data. This validation included checking data completeness (e.g., whether participants completed all the experiments) and reviewing the recorded playbacks to identify any abnormal actions (e.g., instances where participants did not engage in any cooperative behavior). After data validation, we excluded any data with anomalies, including passive participation and missing data, resulting in 68 valid participants (M = 46, F = 22, and Others = 0, ages between 18 and 34). The statistics for the valid data are shown in Table [3](https://arxiv.org/html/2409.08811v1#S5.T3 "Table 3 ‣ 5.2\. Participants ‣ 5\. Methods ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task").

Table 3\. Numbers of the Participants in Each Group.

| Group | Bi-Comm | H-Comm | A-Comm | No-Comm |
| --- | --- | --- | --- | --- |
| Numbers of Participants | 16 | 17 | 17 | 18 |

### 5.3\. Data Analysis

Considering individual random differences, we conducted a regression analysis using a mixed-effects linear model (Pinheiro and Bates, [2006](https://arxiv.org/html/2409.08811v1#bib.bib91); Kaptein, [2016](https://arxiv.org/html/2409.08811v1#bib.bib55)) to examine fixed effects and control the random effects of individual differences. Additionally, we utilized bootstrapping techniques (Mooney et al., [1993](https://arxiv.org/html/2409.08811v1#bib.bib82)) (Sample Size = 2000) to enhance the robustness of our estimates (Dragicevic, [2015](https://arxiv.org/html/2409.08811v1#bib.bib32), [2016](https://arxiv.org/html/2409.08811v1#bib.bib33)), ensuring greater accuracy and reliability in the presence of non-normality or small sample sizes. We considered the main effects of the two conditions, communication interactivity, agent’s ToM capability, and their interaction effects, and applied the Bonferroni correction (Napierala, [2012](https://arxiv.org/html/2409.08811v1#bib.bib83)) for analysis.

For the open questions, the first author and the second author first thoroughly read the text data from the open questions and conducted thematic analysis (Corbin and Strauss, [2014](https://arxiv.org/html/2409.08811v1#bib.bib23)). Subsequently, both two authors independently coded the data and then reached a consensus through repeated discussions with all authors.

## 6\. Results

We analyzed the task metrics, including performance, contribution rate, failure count, and message count, as well as the subjective scale of the participant’s perception of their AI teammates. We also obtained qualitative results from subjective human answers to the questions after the experiment.

### 6.1\. Team Performance

To answer RQ1 (How does the MToM process influence the overall team performance of HATs?), we first analyzed the impact of two independent variables on team performance, using the best performance as the dependent variable. The best performance refers to the maximum value of repeated measurements, which can eliminate the influence of low-level outliers on the results when the learning effect exists and the task is complex. In the Bi-Comm group, the mean score for the agent w/o ToM was 163.75 (SD = 26.17), while the agent with ToM scored 165.00 (SD = 35.02). In the A-Comm group, the mean performance score was 168.53 (SD = 22.83) for the agent without ToM and 169.71 (SD = 27.18) for the agent with ToM. In the H-Comm group, the mean score was 175.59 (SD = 19.19) for the agent without ToM and 168.53 (SD = 22.76) for the agent with ToM. In the No-Comm group, the agent without ToM had a mean score of 177.22 (SD = 25.62), and the agent with ToM achieved a higher mean score of 180.83 (SD = 18.09). The median and standard deviation of the best performance of the subjects under each condition are shown in Figure [13(a)](https://arxiv.org/html/2409.08811v1#S6.F13.sf1 "Figure 13(a) ‣ Figure 13 ‣ 6.1\. Team Performance ‣ 6\. Results ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). Regarding the communication factor, Group No-Comm scored the highest, Group Bi-Comm scored the lowest, and there was little difference between Group H-Comm and Group A-Comm. Within each group, there were no significant differences in the best performance based on whether MToM exits (i.e., agent w/ ToM and w/o ToM).

![Refer to caption](img/8f2a7c8dd12e45a85f0fecc725edcbf6.png)

(a) The Best Team Performance.

![Refer to caption](img/ff495e7616b53e39d14ff633fbe02a6a.png)

(b) Agent Contribution Rate.

![Refer to caption](img/1dd83b240538047a0d08710a7c761582.png)

(c) Perception on How Agent Understand Human.

Figure 13\. The Overview of the Empirical Results in Our Experiments. The box plot includes the median and distribution of the dependent variables, including best team performance, agent contribution rate, and human perception of how agents understand humans.

### 6.2\. Team Collaboration Process

In addition to the scores, we recorded other metrics during the collaboration process to answer RQ2 (How does the MToM process affect the team collaboration process?).

When MToM was present (agent w/ ToM), fixed effects estimation shows that the contribution rate of the AI agent $CR_{A}$ increased by 0.02 ($p<0.001$, $\text{Cohen's }d=0.169$) compared to agent w/o ToM condition. Differences in communication interactivity conditions had no significant impact on the AI agent $CR_{A}$. The median and distribution of the Agent Contribution Rate are illustrated in [Figure 13(b)](https://arxiv.org/html/2409.08811v1#S6.F13.sf2 "In Figure 13 ‣ 6.1\. Team Performance ‣ 6\. Results ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). As for the Failure Count, the Bi-Comm group showed a difference compared to the H-Comm group, with an average increase of 0.77 (p ¡ 0.01).

We further examined the impact of MToM on the frequency of communication. We found that in both Bi-Comm and H-Comm groups where humans were allowed to send messages, the vast majority of participants sent fewer than one message on average. Only one participant in the H-Comm group sent a large number of messages (¿ 15 times in a game). All participants rarely used non-item-related messages (“Good Job” and “Need Improved”) toward the agent. Among all participants, only one person used the “Good Job” message during one game.

In our qualitative analysis of the open questions, participants in the group where humans could send messages reported that sending messages increased workload and negatively impacted their task performance. This result aligns with the message count statistics, which show that humans, focusing on game operations, chose to forgo communication with the agent. At the same time, in the group where humans could send messages, participants reported that sending messages negatively impacted their task performance. In the groups where the agent could send messages, the vast majority of participants reported in the open questions that it was hard to pay attention to the agent’s communication and that they were more focused on the agent’s actions. A similar situation was observed in the open question in the groups where the agent could not send messages. In contrast, participants reported that they focused on the agent’s actions to infer its intentions and adjusted their behavior accordingly.

In our qualitative analysis of the open questions, when participants try to understand the agent, they tend to focus on whether the agent’s behavior can be categorized as logical or consistent. When asked how they considered the agent understood them, more participants tended to consider how well the agent was coordinating with them. For example, most participants felt that the agent complemented their actions and did not repeat the same tasks as they better understood them. The participants who were able to read the agent’s messages mentioned that the agent’s messages directly helped reduce unnecessary work, as they would avoid performing tasks that the agent had indicated it was already handling.

### 6.3\. Human Preference and Perceptions of AI Agents

![Refer to caption](img/3459b99598390273dd1c80bb69950976.png)

(a) Preference for Agent Playing with.

![Refer to caption](img/21bf7f3e27b6de2dda0f00d929ed5a4a.png)

(b) Preference for Better Understand Human.

![Refer to caption](img/edd427751c7a2e3b9736a8225d3f2b2b.png)

(c) Preference for Better Collaboration.

![Refer to caption](img/fbd0e317862fe806cde3508a745a65ae.png)

(d) Preference for Agent Playing with.

![Refer to caption](img/faff4f992990ef759a334d60447d35cb.png)

(e) Preference for Better Understand Human.

![Refer to caption](img/b637d03368ad977b27d22e88a11e729a.png)

(f) Preference for Better Collaboration.

![Refer to caption](img/9e2557c0c7ac6a55cf70cc0fff205706.png)

(g) Preference for Agent Playing with.

![Refer to caption](img/ebd3718b5fa160e8377512e15956ec9a.png)

(h) Preference for Better Understand Human.

![Refer to caption](img/72f2da87da77671062f93b10bd7be417.png)

(i) Preference for Better Collaboration.

![Refer to caption](img/65569be78a75220d477b93a9fc75f91c.png)

(j) Preference for Agent Playing with.

![Refer to caption](img/4568a5c4d3a3c61ec5b50db484080a33.png)

(k) Preference for Better Understand Human.

![Refer to caption](img/e4c4e5e2d95164b6bae76dbcb2af28fb.png)

(l) Preference for Better Collaboration.

Figure 14\. The distribution of Team Score and Preference of the w/ ToM Agent in Each Group. Each chart’s horizontal axis shows the “Task Score,” and the vertical axis represents the “Preference for Agent w/ ToM.” The curve in the middle is a regression line illustrating the trend between task score and preference, with the shaded area indicating the $95\%$ confidence interval. Scatter points represent individual participants’ scores and preferences. The plots on the top and right side show marginal density distributions of task scores and preferences.

For RQ3 (How do humans perceive AI teammates in the MToM process?), we collect the participants’ perceptions of the agent in each game and their preference for the two agents. The vast majority of participants reported that they could sense the difference between the two agents (94% in Bi-Comm, 94% in H-Comm, 88% in A-Comm, 77% in No-Comm). The vast majority of participants showed consistency across the four preference questions (which agent understands me better, which agent I understand better, which agent I work better with, and which agent I prefer), with all their individual preferences being aligned (i.e., prefer the same agent in these four questions). Only a minimal number of participants exhibited slight differences in their preferences. The impact of different levels of communication interactivity on human preference was not statistically significant. We visualized the correlation between participants’ scores and their preference for the agent in each group. As shown in [Figure 14](https://arxiv.org/html/2409.08811v1#S6.F14 "In 6.3\. Human Preference and Perceptions of AI Agents ‣ 6\. Results ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), in the Bi-Comm, H-Comm, and No-Comm groups, the better the participants’ scores, the stronger their preference for the agent with ToM abilities. The regression slope was steepest in the No-Comm group. Interestingly, the A-Comm group showed the opposite result: participants with higher scores may prefer the agent without ToM abilities. While it has a trend in the relationship between team performance and human preference on agent w/ ToM, these results are not statistically significant in the point-biserial correlation coefficient (Brown, [2001](https://arxiv.org/html/2409.08811v1#bib.bib16)) tests.

In addition to preferences, we collected participants’ perceptions of each game after it ended using specific statements based on (Hoffman, [2019](https://arxiv.org/html/2409.08811v1#bib.bib49)). During the experiment, all participants were unaware of which agent had ToM abilities. We used agreement levels with statements to collect participants’ perceptions of the agent’s ToM abilities and the extent of those perceptions. For the statement “I feel the agent understands me,” a noticeable difference was observed in each group as shown in [Figure 13(c)](https://arxiv.org/html/2409.08811v1#S6.F13.sf3 "In Figure 13 ‣ 6.1\. Team Performance ‣ 6\. Results ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"). We conducted a fixed effects test on all results, and participants in each group perceived that the agent with ToM (w/ MToM condition) understood them better (p ¡ 0.001, Cohen’s d = 0.336). For the statement “I understand the agent,” there was a significant main effect between w/ MToM and w/o MToM (p ¡ 0.001), but the effect size was small (Cohen’s d = 0.177), not reaching the threshold for a small effect. The detailed results of the questionnaires we used in the experiment can be found in [Appendix A](https://arxiv.org/html/2409.08811v1#A1 "Appendix A The Detailed Results of Questionnaires in Each Condition ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task").

We combined the qualitative results from the open questions to understand human perceptions of the agent’s ToM capability. Our qualitative analysis results indicate that the interactivity of communication impacts the human mental model of agents. In situations where participants are unaware of whether the agent possesses ToM abilities, those in the H-Comm and Bi-Comm groups sent messages reporting that the agent with ToM capability could follow their instructions to complete the necessary tasks and found the behavior of the ToM agent to be more understandable. This result indicates that humans’ understanding of an agent might depend on whether the agent meets their expectations (mental models).

MToM also influences human perception of game difficulty. When the agent possesses ToM abilities, it makes humans perceive the game as easier in both the Bi-Comm and H-Comm groups. Based on the analysis of the open questions in these two groups, when the agent is easier to understand, it reduces humans’ perception of task difficulty, even if there is no significant difference in the final task performance.

## 7\. Discussion

Due to the shared workspace setup, there is a close connection between the tasks and actions of the AI agent and the human participant. We utilized real-time tasks to relax the assumption of strict collaboration, moving towards a more realistic model of cooperation (Salikutluk et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib97)), such as human-AI collaboration in household tasks (Brohan et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib15)). In this section, we discuss the results of our studies, offering insights into the future design of AI agents with ToM capability to collaborate with human, especially those powered by LLMs.

### 7.1\. Team Performance in Real-time Shared Workspace Task

Our experimental results demonstrate that the MToM process impacts HATs. Moreover, there is an interaction effect between communication interactivity and MToM on the performance of HATs.

In our experiments, the main effects of communication interactivity are statistically significant on team performance. The No-Comm Group performs better than other conditions. We believe complex language communication interactions may reduce or distract human performance. In support of this hypothesis, teams in the Bi-Comm group result in a higher Failure Count than the H-Comm group. Additionally, the qualitative results suggest that humans may adjust their actions based more on the agent’s behavior rather than direct verbal communication, which is closely related to the direct observation of behaviors in a shared workspace (Dourish and Bellotti, [1992](https://arxiv.org/html/2409.08811v1#bib.bib30)). This result aligns with previous findings on the relationship between bidirectional communication and task load (Julia L. Wright and Chen, [2022](https://arxiv.org/html/2409.08811v1#bib.bib54)), as messages tend to increase the human’s task load.

The agent’s ToM ability did not significantly impact the team’s objective performance but did reduce instances of cooperation failure. This result is similar to previous studies on agents expressing their intentions to humans (Le Guillou et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib62)). As for communication interactivity, complex communication interactivity can affect performance due to its impact on human operations. Since we did not enforce strict cooperation (Salikutluk et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib97); Ashktorab et al., [2021](https://arxiv.org/html/2409.08811v1#bib.bib5)), the detailed collaborative process does not directly or absolutely affect the team’s objective score. Instead, the MToM process has a greater influence on the specifics of the team collaboration process and human perception.

Some studies have pointed out that the design of the communication interface affects HATs (Kilgore and Voshell, [2014](https://arxiv.org/html/2409.08811v1#bib.bib58)). Moreover, our results make us reconsider the double-side impacts of verbal communication in shared workspaces when humans and AI agents face real-time tasks. While verbal communication can convey the most precise information, the operational burden it brings may be an essential point for future research to address when MToM exists.

### 7.2\. How Do Humans Understand The Agent in MToM Process?

In our results of communication interactivity, we found that in real-time tasks within shared workspaces, humans are more inclined to use implicit communication in collaboration. When MToM is present, non-verbal communication can be just as effective as verbal communication in real-time tasks within shared workspaces.

The experiment’s results regarding communication message amount indicate that in real-time shared workspace tasks, humans can almost not initiate verbal communication as explicitly as communication with an agent. This leads humans to rely on the agent’s behavior for implicit communication and potentially want the agent to communicate in the same way. When MToM is present, our qualitative findings show that the agent’s ability to adjust based on human actions is more helpful in aiding human understanding. This result highlights the importance of the agent’s action responses in humans’ construction of a mental model of the agent. Our findings complement previous work that enforced the use of implicit communication in the forced cooperation scenario (Liang et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib68)), confirming the effectiveness of implicit non-verbal communication in a broader range of scenarios.

We also found that humans have different patterns of understanding the agent. Our qualitative study identified two distinct patterns of how humans understand the agent. The first is based on whether there are recognizable patterns in the agent’s actions, such as “whether the agent consistently performs action B right after action A”. The second pattern focuses on whether the agent is dedicated to a higher-level division of labor, such as “the agent prefers cooking beef since I am focusing on chopping lettuce.” Based on the first pattern, humans do not pay attention to the agent’s higher-level strategies or whether the agent is making inferences about humans. The second pattern, which involves the agent’s division of labor, leads humans to focus more on whether the agent aligns with their expectations of its behavior. Humans use ToM to build a belief about the agent’s ToM in this pattern, i.e., an understanding based on the higher-level division of labor involves higher-order ToM inferences (i.e., “I think you think I think …”) about the agent. Such differences may have complex causes. We believe these differences could stem from humans’ inherent perceptions of the agent or pre-existing mental models formed based on other experiences (Gero et al., [2020](https://arxiv.org/html/2409.08811v1#bib.bib39)). In future research, further exploring which patterns humans actually rely on in their ToM process with agents will help refine and adjust agent behavior during the development process. There is also greater room for exploration in modeling higher-order ToM processes (Wen et al., [2019](https://arxiv.org/html/2409.08811v1#bib.bib119)) to improve the agents’ understanding of humans.

### 7.3\. How Do Humans Perceive The Agent’s ToM Capability?

In our results, most participants reported being able to perceive differences between the two agents, but they could not accurately identify which agent better understood humans. Human perception of the agent’s ToM abilities may rely more on team performance or human skill level on the task.

There is some correlation between team scores and human preferences for the agent. Although lacking statistical significance, we can still observe a trend: the higher the team score, the more humans prefer agents with ToM abilities. The steep regression slope in the No-Comm group implies that participants relied heavily on the agent’s actions to infer understanding and capabilities in the absence of communication. The strong preference for the ToM-enabled agent in this group suggests that ToM abilities are especially valuable when explicit communication is unavailable, as the agent’s behavior becomes the primary channel through which humans assess coordination.

We regard that better team performance may indicate that the human participants have higher skill levels. Humans with higher skill levels may have a different understanding of the agent’s behavior. These preference results suggest that in future designs of ToM-enabled agents, human skill levels should also be incorporated into the agent’s belief system, allowing the agent to adjust its behavior based on the human’s skill level.

### 7.4\. Challenges of LLM-driven AI Agent in HATs

In our validation of the LLM-driven agents’ capability, the ToM-enabled agent significantly outperforms the non-ToM agent when paired with a fixed rule-based teammate, demonstrating the effectiveness of using LLMs to build ToM capabilities. However, our participants still reported several clear strategic errors by the agent in the open questions, such as not prioritizing serving dishes based on the urgency of the orders. This phenomenon indicates that the LLM’s planning capabilities are still somewhat different from humans’ expectations.

In our open questions, participants also provide feedback regarding the agent’s language and action responses. In the groups where the agent could send messages (A-Comm and Bi-Comm), some participants reported inconsistencies between the agent’s words and actions. These issues suggest that the hallucination problem in LLMs (Huang et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib50)) may negatively influence LLM-based ToM capability. In more complex and potentially dangerous tasks like driving (Cui et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib24); Chen et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib20)), hallucinations could pose harm to humans. It is, therefore, essential to consider protective mechanisms in future LLM-driven agent frameworks, especially with MToM.

Moreover, our results may suggest that building agents with language communication capability needs to consider human cognitive load in real-time tasks, and non-verbal communication does not necessarily imply an inability to coordinate with humans. Some studies on conversational agents have shown that humans still engage in a significant amount of non-verbal communication when using dialogue assistants, indicating a clear need for non-verbal communication between humans and agents (Chan et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib19)). In the future design of LLM agents, enhancing the non-verbal communication abilities of LLM-driven AI agents is worth exploring.

### 7.5\. Limitations and Future Works

Our study has the following limitations. First, the experiment was conducted online, making it subject to network fluctuations, and it was difficult to monitor participants’ specific behaviors fully. We used data validation techniques to mitigate this impact. Second, since our experiment was set in a real-time shared workspace task, the agent’s real-time operations and communication were highly demanded. LLMs still face significant limitations in real-time responsiveness, and some other experiments have used pausing or similar methods to alleviate this issue (Liu et al., [2024](https://arxiv.org/html/2409.08811v1#bib.bib71)). However, pausing affects the task load and human perception (Hardy and Hinkin, [2022](https://arxiv.org/html/2409.08811v1#bib.bib45)). We used the code-as-policy (Liang et al., [2023](https://arxiv.org/html/2409.08811v1#bib.bib69)) technique to address the real-time challenge, but this may have impacted the flexibility of the agents’ strategy. We consider this an important research issue for future LLM agents.

The current research on MToM in human-AI collaboration tends to focus more on practical applications, while the community still needs studies on the underlying MToM processes. In addition to conducting preliminary investigations into the impact of mechanisms through experiments, we have also developed a shared workspace task experiment platform and agents with MToM capabilities to facilitate future research on MToM processes. Our current experimental results suggest that further exploring multi-level recursive ToM processes may help understand why humans attribute certain mental states to agents. As for agents’ ToM capabilities, our task formulation and platform construction can support testing more complex agent ToM frameworks and potentially extend to more embodied intelligence systems.

## 8\. Conclusion

This work explored the MToM process in human-AI collaboration with a real-time task in a shared workspace. We first proposed a shared workspace collaboration task based on the Overcooked game and used an LLM to build an agent with ToM and communication abilities to experiment. Through an online study (n = 68), participants collaborated with ToM-enabled and non-ToM agents under different communication interactivity conditions in the shared workspace collaboration task. We found that in real-time tasks within shared workspaces, the agent’s ToM abilities can enhance humans’ understanding of the agent and make people feel understood. However, they did not significantly impact team performance. We found that the Bi-Comm group with the best communication interactivity performed the worst in team tasks. These objective results align with participants’ reports that verbal communication during interaction caused excessive operational and informational burdens. We also found that humans in HATs rely more on the agent’s actions to understand the agent than on verbal communication. We also found a correlation between team performance and human preferences regarding their perception of the agent. This paper adopts an MToM perspective to understand the interaction between human and agent ToM capabilities in human-AI collaboration, providing new insights for future AI agent ToM capability design. We believe that when considering the agent’s ToM capability in the future, non-verbal communication demonstrated by the agent through ToM (such as purposeful actions) in real-time shared workspace tasks is just as important as verbal communication. Furthermore, the human understanding of the agent’s ToM process may involve multi-level ToM, which should be incorporated into the design of the agent’s ToM in the future.

###### Acknowledgements.

The authors thank Yang Li, Chaoran Li, Yiwei Shi, Shihan Fu and Yuxuan Lu for their kindly help in paper writing.

## References

*   (1)
*   Aher et al. (2023) Gati V Aher, Rosa I Arriaga, and Adam Tauman Kalai. 2023. Using large language models to simulate multiple humans and replicate human subject studies. In *International Conference on Machine Learning*. PMLR, 337–371.
*   Albrecht and Stone (2018) Stefano V Albrecht and Peter Stone. 2018. Autonomous agents modelling other agents: A comprehensive survey and open problems. *Artificial Intelligence* 258 (2018), 66–95.
*   Argyle (1972) Michael Argyle. 1972. Non-verbal communication in human social interaction. *Non-verbal communication* 2, 1 (1972).
*   Ashktorab et al. (2021) Zahra Ashktorab, Casey Dugan, James Johnson, Qian Pan, Wei Zhang, Sadhana Kumaravel, and Murray Campbell. 2021. Effects of Communication Directionality and AI Agent Differences in Human-AI Interaction. In *Proceedings of the 2021 CHI Conference on Human Factors in Computing Systems* (¡conf-loc¿, ¡city¿Yokohama¡/city¿, ¡country¿Japan¡/country¿, ¡/conf-loc¿) *(CHI ’21)*. Association for Computing Machinery, New York, NY, USA, Article 238, 15 pages. [https://doi.org/10.1145/3411764.3445256](https://doi.org/10.1145/3411764.3445256)
*   Ashktorab et al. (2020) Zahra Ashktorab, Q. Vera Liao, Casey Dugan, James Johnson, Qian Pan, Wei Zhang, Sadhana Kumaravel, and Murray Campbell. 2020. Human-AI Collaboration in a Cooperative Game Setting: Measuring Social Perception and Outcomes. *Proc. ACM Hum.-Comput. Interact.* 4, CSCW2, Article 96 (oct 2020), 20 pages. [https://doi.org/10.1145/3415167](https://doi.org/10.1145/3415167)
*   Bansal et al. (2019) Gagan Bansal, Besmira Nushi, Ece Kamar, Walter S Lasecki, Daniel S Weld, and Eric Horvitz. 2019. Beyond accuracy: The role of mental models in human-AI team performance. In *Proceedings of the AAAI conference on human computation and crowdsourcing*, Vol. 7\. 2–11.
*   Bara et al. (2021) Cristian-Paul Bara, Sky CH-Wang, and Joyce Chai. 2021. MindCraft: Theory of Mind Modeling for Situated Dialogue in Collaborative Tasks. In *Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing*, Marie-Francine Moens, Xuanjing Huang, Lucia Specia, and Scott Wen-tau Yih (Eds.). Association for Computational Linguistics, Online and Punta Cana, Dominican Republic, 1112–1125. [https://doi.org/10.18653/v1/2021.emnlp-main.85](https://doi.org/10.18653/v1/2021.emnlp-main.85)
*   Baratta et al. (2023) Alessio Baratta, Antonio Cimino, Maria Grazia Gnoni, and Francesco Longo. 2023. Human Robot Collaboration in Industry 4.0: a literature review. *Procedia Computer Science* 217 (2023), 1887–1895.
*   Baron-Cohen (1997) Simon Baron-Cohen. 1997. *Mindblindness: An essay on autism and theory of mind*. MIT press.
*   Baron-Cohen (1999) Simon Baron-Cohen. 1999. *The evolution of a theory of mind*. na.
*   Baron-Cohen et al. (1985) Simon Baron-Cohen, Alan M Leslie, and Uta Frith. 1985. Does the autistic child have a “theory of mind”? *Cognition* 21, 1 (1985), 37–46.
*   Benninghoff et al. (2013) Brenda Benninghoff, Philipp Kulms, Laura Hoffmann, and Nicole C Krämer. 2013. Theory of mind in human-robot-communication: Appreciated or not? *Kognitive Systeme* 2013, 1 (2013).
*   Bernstein et al. (2002) Daniel S Bernstein, Robert Givan, Neil Immerman, and Shlomo Zilberstein. 2002. The complexity of decentralized control of Markov decision processes. *Mathematics of operations research* 27, 4 (2002), 819–840.
*   Brohan et al. (2023) Anthony Brohan, Yevgen Chebotar, Chelsea Finn, Karol Hausman, Alexander Herzog, Daniel Ho, Julian Ibarz, Alex Irpan, Eric Jang, Ryan Julian, et al. 2023. Do as i can, not as i say: Grounding language in robotic affordances. In *Conference on robot learning*. PMLR, 287–318.
*   Brown (2001) James Dean Brown. 2001. Point-biserial correlation coefficients. *Statistics* 5, 3 (2001), 12–6.
*   Carroll et al. (2019) Micah Carroll, Rohin Shah, Mark K Ho, Tom Griffiths, Sanjit Seshia, Pieter Abbeel, and Anca Dragan. 2019. On the utility of learning about humans for human-ai coordination. *Advances in neural information processing systems* 32 (2019).
*   Carruthers and Smith (1996) Peter Carruthers and Peter K Smith. 1996. *Theories of theories of mind*. Cambridge university press.
*   Chan et al. (2024) Szeyi Chan, Shihan Fu, Jiachen Li, Bingsheng Yao, Smit Desai, Mirjana Prpa, and Dakuo Wang. 2024. Human and LLM-Based Voice Assistant Interaction: An Analytical Framework for User Verbal and Nonverbal Behaviors. *arXiv preprint arXiv:2408.16465* (2024).
*   Chen et al. (2024) Long Chen, Oleg Sinavski, Jan Hünermann, Alice Karnsund, Andrew James Willmott, Danny Birch, Daniel Maund, and Jamie Shotton. 2024. Driving with LLMs: Fusing Object-Level Vector Modality for Explainable Autonomous Driving. In *2024 IEEE International Conference on Robotics and Automation (ICRA)*. 14093–14100. [https://doi.org/10.1109/ICRA57147.2024.10611018](https://doi.org/10.1109/ICRA57147.2024.10611018)
*   Christoforou et al. (2020) Eftychios G Christoforou, Andreas S Panayides, Sotiris Avgousti, Panicos Masouras, and Constantinos S Pattichis. 2020. An overview of assistive robotics and technologies for elderly care. In *XV Mediterranean Conference on Medical and Biological Engineering and Computing–MEDICON 2019: Proceedings of MEDICON 2019, September 26-28, 2019, Coimbra, Portugal*. Springer, 971–976.
*   Cooke et al. (2020) Nancy Cooke, Mustafa Demir, and Lixiao Huang. 2020. A framework for human-autonomy team research. In *Engineering Psychology and Cognitive Ergonomics. Cognition and Design: 17th International Conference, EPCE 2020, Held as Part of the 22nd HCI International Conference, HCII 2020, Copenhagen, Denmark, July 19–24, 2020, Proceedings, Part II 22*. Springer, 134–146.
*   Corbin and Strauss (2014) Juliet Corbin and Anselm Strauss. 2014. *Basics of Qualitative Research: Techniques and Procedures for Developing Grounded Theory*. Sage publications.
*   Cui et al. (2024) Yaodong Cui, Shucheng Huang, Jiaming Zhong, Zhenan Liu, Yutong Wang, Chen Sun, Bai Li, Xiao Wang, and Amir Khajepour. 2024. DriveLLM: Charting the Path Toward Full Autonomous Driving With Large Language Models. *IEEE Transactions on Intelligent Vehicles* 9, 1 (2024), 1450–1464. [https://doi.org/10.1109/TIV.2023.3327715](https://doi.org/10.1109/TIV.2023.3327715)
*   Dehkordi et al. (2021) Maryam Banitalebi Dehkordi, Reda Mansy, Abolfazl Zaraki, Arpit Singh, and Rossitza Setchi. 2021. Explainability in Human-Robot Teaming. *Procedia Comput. Sci.* 192, C (jan 2021), 3487–3496. [https://doi.org/10.1016/j.procs.2021.09.122](https://doi.org/10.1016/j.procs.2021.09.122)
*   Demir et al. (2016) Mustafa Demir, Nathan J. McNeese, and Nancy J. Cooke. 2016. Team communication behaviors of the human-automation teaming. In *2016 IEEE International Multi-Disciplinary Conference on Cognitive Methods in Situation Awareness and Decision Support (CogSIMA)*. 28–34. [https://doi.org/10.1109/COGSIMA.2016.7497782](https://doi.org/10.1109/COGSIMA.2016.7497782)
*   Demir et al. (2017) Mustafa Demir, Nathan J McNeese, and Nancy J Cooke. 2017. Team situation awareness within the context of human-autonomy teaming. *Cognitive Systems Research* 46 (2017), 3–12.
*   Demir et al. (2020) Mustafa Demir, Nathan J. McNeese, and Nancy J. Cooke. 2020. Understanding human-robot teams in light of all-human teams: Aspects of team interaction and shared cognition. *International Journal of Human-Computer Studies* 140 (2020), 102436. [https://doi.org/10.1016/j.ijhcs.2020.102436](https://doi.org/10.1016/j.ijhcs.2020.102436)
*   Devin and Alami (2016) Sandra Devin and Rachid Alami. 2016. An implemented theory of mind to improve human-robot shared plans execution. In *2016 11th ACM/IEEE International Conference on Human-Robot Interaction (HRI)*. IEEE, 319–326.
*   Dourish and Bellotti (1992) Paul Dourish and Victoria Bellotti. 1992. Awareness and coordination in shared workspaces. In *Proceedings of the 1992 ACM conference on Computer-supported cooperative work*. 107–114.
*   DOWNES and McMILLAN (2000) EDWARD J. DOWNES and SALLY J. McMILLAN. 2000. Defining Interactivity: A Qualitative Identification of Key Dimensions. *New Media & Society* 2, 2 (2000), 157–179. [https://doi.org/10.1177/14614440022225751](https://doi.org/10.1177/14614440022225751) arXiv:https://doi.org/10.1177/14614440022225751
*   Dragicevic (2015) Pierre Dragicevic. 2015. *HCI Statistics without p-values*. Ph. D. Dissertation. Inria.
*   Dragicevic (2016) Pierre Dragicevic. 2016. Fair statistical communication in HCI. *Modern statistical methods for HCI* (2016), 291–330.
*   Faccio et al. (2023) Maurizio Faccio, Irene Granata, Alberto Menini, Mattia Milanese, Chiara Rossato, Matteo Bottin, Riccardo Minto, Patrik Pluchino, Luciano Gamberini, Giovanni Boschetti, et al. 2023. Human factors in cobot era: a review of modern production systems features. *Journal of Intelligent Manufacturing* 34, 1 (2023), 85–106.
*   (35) Meta Fundamental AI Research Diplomacy Team (FAIR)†, Anton Bakhtin, Noam Brown, Emily Dinan, Gabriele Farina, Colin Flaherty, Daniel Fried, Andrew Goff, Jonathan Gray, Hengyuan Hu, et al. 2022. Human-level play in the game of Diplomacy by combining language models with strategic reasoning. *Science* 378, 6624 (2022), 1067–1074.
*   Feng and Feng (2024) Shuang Feng and Grace Feng. 2024. An Extremely Data-efficient and Generative LLM-based Reinforcement Learning Agent for Recommenders. *arXiv preprint arXiv:2408.16032* (2024).
*   Fleming et al. (1990) John H Fleming, John M Darley, James L Hilton, and Brian A Kojetin. 1990. Multiple audience problem: A strategic communication perspective on social perception. *Journal of personality and social psychology* 58, 4 (1990), 593.
*   Fussell et al. (1998) Susan R. Fussell, Robert E. Kraut, F. Javier Lerch, William L. Scherlis, Matthew M. McNally, and Jonathan J. Cadiz. 1998. Coordination, overload and team performance: effects of team communication strategies. In *Proceedings of the 1998 ACM Conference on Computer Supported Cooperative Work* (Seattle, Washington, USA) *(CSCW ’98)*. Association for Computing Machinery, New York, NY, USA, 275–284. [https://doi.org/10.1145/289444.289502](https://doi.org/10.1145/289444.289502)
*   Gero et al. (2020) Katy Ilonka Gero, Zahra Ashktorab, Casey Dugan, Qian Pan, James Johnson, Werner Geyer, Maria Ruiz, Sarah Miller, David R Millen, Murray Campbell, et al. 2020. Mental models of AI agents in a cooperative game setting. In *Proceedings of the 2020 chi conference on human factors in computing systems*. 1–12.
*   Gong et al. (2024) Ran Gong, Qiuyuan Huang, Xiaojian Ma, Yusuke Noda, Zane Durante, Zilong Zheng, Demetri Terzopoulos, Li Fei-Fei, Jianfeng Gao, and Hoi Vo. 2024. MindAgent: Emergent Gaming Interaction. In *Findings of the Association for Computational Linguistics: NAACL 2024*, Kevin Duh, Helena Gomez, and Steven Bethard (Eds.). Association for Computational Linguistics, Mexico City, Mexico, 3154–3183. [https://doi.org/10.18653/v1/2024.findings-naacl.200](https://doi.org/10.18653/v1/2024.findings-naacl.200)
*   Guan et al. (2023) Cong Guan, Lichao Zhang, Chunpeng Fan, Yichen Li, Feng Chen, Lihe Li, Yunjia Tian, Lei Yuan, and Yang Yu. 2023. Efficient Human-AI Coordination via Preparatory Language-based Convention. *arXiv preprint arXiv:2311.00416* (2023).
*   Guo et al. (2024) Siyuan Guo, Cheng Deng, Ying Wen, Hechang Chen, Yi Chang, and Jun Wang. 2024. DS-Agent: Automated Data Science by Empowering Large Language Models with Case-Based Reasoning. In *Forty-first International Conference on Machine Learning*.
*   Han et al. (2024) Dongge Han, Trevor McInroe, Adam Jelley, Stefano V Albrecht, Peter Bell, and Amos Storkey. 2024. LLM-Personalize: Aligning LLM Planners with Human Preferences via Reinforced Self-Training for Housekeeping Robots. *arXiv preprint arXiv:2404.14285* (2024).
*   Harada et al. (2023) Tomohiro Harada, Johei Matsuoka, and Kiyohiko Hattori. 2023. Behavior analysis of emergent rule discovery for cooperative automated driving using deep reinforcement learning. *Artificial Life and Robotics* 28, 1 (2023), 31–42.
*   Hardy and Hinkin (2022) David J. Hardy and Charles H. Hinkin. 2022. Mental Workload in Neuropsychology: An Example With the NASA-TLX in Adults With HIV. *Frontiers in Neuroergonomics* 3 (2022). [https://doi.org/10.3389/fnrgo.2022.881653](https://doi.org/10.3389/fnrgo.2022.881653)
*   Hart et al. (1968) Peter E Hart, Nils J Nilsson, and Bertram Raphael. 1968. A formal basis for the heuristic determination of minimum cost paths. *IEEE transactions on Systems Science and Cybernetics* 4, 2 (1968), 100–107.
*   He et al. (2023) Ziyao He, Yunpeng Song, Shurui Zhou, and Zhongmin Cai. 2023. Interaction of Thoughts: Towards Mediating Task Assignment in Human-AI Cooperation with a Capability-Aware Shared Mental Model. In *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems* (Hamburg, Germany) *(CHI ’23)*. Association for Computing Machinery, New York, NY, USA, Article 353, 18 pages. [https://doi.org/10.1145/3544548.3580983](https://doi.org/10.1145/3544548.3580983)
*   Hiatt et al. (2011) Laura M Hiatt, Anthony M Harrison, and J Gregory Trafton. 2011. Accommodating human variability in human-robot teams through theory of mind. In *Twenty-Second International Joint Conference on Artificial Intelligence*.
*   Hoffman (2019) Guy Hoffman. 2019. Evaluating fluency in human–robot collaboration. *IEEE Transactions on Human-Machine Systems* 49, 3 (2019), 209–218.
*   Huang et al. (2023) Lei Huang, Weijiang Yu, Weitao Ma, Weihong Zhong, Zhangyin Feng, Haotian Wang, Qianglong Chen, Weihua Peng, Xiaocheng Feng, Bing Qin, and Ting Liu. 2023. A Survey on Hallucination in Large Language Models: Principles, Taxonomy, Challenges, and Open Questions. arXiv:2311.05232 [cs.CL] [https://arxiv.org/abs/2311.05232](https://arxiv.org/abs/2311.05232)
*   Iantovics (2008) Barna László Iantovics. 2008. Agent-based medical diagnosis systems. *Computing and Informatics* 27, 4 (2008), 593–625.
*   Jakobson (1972) Roman Jakobson. 1972. Verbal communication. *Scientific American* 227, 3 (1972), 72–81.
*   Jiang et al. (2023) Jinglu Jiang, Alexander J Karran, Constantinos K Coursaris, Pierre-Majorique Léger, and Joerg Beringer. 2023. A situation awareness perspective on human-AI interaction: Tensions and opportunities. *International Journal of Human–Computer Interaction* 39, 9 (2023), 1789–1806.
*   Julia L. Wright and Chen (2022) Shan G. Lakhmani Julia L. Wright and Jessie Y. C. Chen. 2022. Bidirectional Communications in Human-Agent Teaming: The Effects of Communication Style and Feedback. *International Journal of Human–Computer Interaction* 38, 18-20 (2022), 1972–1985. [https://doi.org/10.1080/10447318.2022.2068744](https://doi.org/10.1080/10447318.2022.2068744) arXiv:https://doi.org/10.1080/10447318.2022.2068744
*   Kaptein (2016) Maurits Kaptein. 2016. Using generalized linear (mixed) models in HCI. *Modern Statistical Methods for HCI* (2016), 251–274.
*   Kenthapadi et al. (2024) Krishnaram Kenthapadi, Mehrnoosh Sameki, and Ankur Taly. 2024. Grounding and Evaluation for Large Language Models: Practical Challenges and Lessons Learned (Survey). In *Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*. 6523–6533.
*   Key and Key (1980) Mary Ritchie Key and Mary Ritchie Key. 1980. *The relationship of verbal and nonverbal communication*. Mouton The Hague.
*   Kilgore and Voshell (2014) Ryan M. Kilgore and Martin Voshell. 2014. Increasing the Transparency of Unmanned Systems: Applications of Ecological Interface Design. In *Interacción*. [https://api.semanticscholar.org/CorpusID:31347319](https://api.semanticscholar.org/CorpusID:31347319)
*   Kim et al. (2023) Taenyun Kim, Maria D. Molina, Minjin (MJ) Rheu, Emily S. Zhan, and Wei Peng. 2023. One AI Does Not Fit All: A Cluster Analysis of the Laypeople’s Perception of AI Roles. In *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems* (Hamburg, Germany) *(CHI ’23)*. Association for Computing Machinery, New York, NY, USA, Article 29, 20 pages. [https://doi.org/10.1145/3544548.3581340](https://doi.org/10.1145/3544548.3581340)
*   Krych-Appelbaum et al. (2007) Meredyth Krych-Appelbaum, Julie Banzon Law, Dayna Jones, Allyson Barnacz, Amanda Johnson, and Julian Paul Keenan. 2007. “I think I know what you mean”: The role of theory of mind in collaborative communication. *Interaction Studies* 8, 2 (2007), 267–280.
*   Langley et al. (2022) Christelle Langley, Bogdan Ionut Cirstea, Fabio Cuzzolin, and Barbara J Sahakian. 2022. Theory of mind and preference learning at the interface of cognitive science, neuroscience, and AI: A review. *Frontiers in artificial intelligence* 5 (2022), 778852.
*   Le Guillou et al. (2023) Marin Le Guillou, Laurent Prévot, and Bruno Berberian. 2023. Trusting Artificial Agents: Communication Trumps Performance. In *Proceedings of the 2023 International Conference on Autonomous Agents and Multiagent Systems* (London, United Kingdom) *(AAMAS ’23)*. International Foundation for Autonomous Agents and Multiagent Systems, Richland, SC, 299–306.
*   Lea and Spears (1992) Martin Lea and Russell Spears. 1992. Paralanguage and social perception in computer-mediated communication. *Journal of Organizational Computing and Electronic Commerce* 2, 3-4 (1992), 321–341.
*   Lee et al. (2012) Soyoung Lee, Charlotte Tang, Sun Young Park, and Yunan Chen. 2012. Loosely formed patient care teams: communication challenges and technology design. In *Proceedings of the ACM 2012 conference on computer supported cooperative work*. 867–876.
*   Li et al. (2024a) Wenhao Li, Zhiyuan Yu, Qijin She, Zhinan Yu, Yuqing Lan, Chenyang Zhu, Ruizhen Hu, and Kai Xu. 2024a. LLM-enhanced Scene Graph Learning for Household Rearrangement. *arXiv preprint arXiv:2408.12093* (2024).
*   Li et al. (2023) Yang Li, Shao Zhang, Jichen Sun, Yali Du, Ying Wen, Xinbing Wang, and Wei Pan. 2023. Cooperative Open-ended Learning Framework for Zero-Shot Coordination. In *ICML* *(Proceedings of Machine Learning Research, Vol. 202)*. PMLR, 20470–20484.
*   Li et al. (2024b) Yang Li, Shao Zhang, Jichen Sun, Wenhao Zhang, Yali Du, Ying Wen, Xinbing Wang, and Wei Pan. 2024b. Tackling cooperative incompatibility for zero-shot human-ai coordination. *Journal of Artificial Intelligence Research* 80 (2024), 1139–1185.
*   Liang et al. (2019) Claire Liang, Julia Proft, Erik Andersen, and Ross A Knepper. 2019. Implicit communication of actionable information in human-ai teams. In *Proceedings of the 2019 CHI conference on human factors in computing systems*. 1–13.
*   Liang et al. (2023) Jacky Liang, Wenlong Huang, Fei Xia, Peng Xu, Karol Hausman, Brian Ichter, Pete Florence, and Andy Zeng. 2023. Code as Policies: Language Model Programs for Embodied Control. In *IEEE International Conference on Robotics and Automation, ICRA 2023, London, UK, May 29 - June 2, 2023*. IEEE, 9493–9500. [https://doi.org/10.1109/ICRA48891.2023.10160591](https://doi.org/10.1109/ICRA48891.2023.10160591)
*   Lindner and El-Assady (2022) David Lindner and Mennatallah El-Assady. 2022. Humans are not boltzmann distributions: Challenges and opportunities for modelling human feedback and interaction in reinforcement learning. *arXiv preprint arXiv:2206.13316* (2022).
*   Liu et al. (2024) Jijia Liu, Chao Yu, Jiaxuan Gao, Yuqing Xie, Qingmin Liao, Yi Wu, and Yu Wang. 2024. LLM-Powered Hierarchical Language Agent for Real-time Human-AI Coordination. In *Proceedings of the 23rd International Conference on Autonomous Agents and Multiagent Systems* (Auckland, New Zealand) *(AAMAS ’24)*. International Foundation for Autonomous Agents and Multiagent Systems, Richland, SC, 1219–1228.
*   Lu et al. (2021) Zhicong Lu, Chenxinran Shen, Jiannan Li, Hong Shen, and Daniel Wigdor. 2021. More kawaii than a real-person live streamer: understanding how the otaku community engages with and perceives virtual YouTubers. In *Proceedings of the 2021 CHI Conference on Human Factors in Computing Systems*. 1–14.
*   M. Bran et al. (2024) Andres M. Bran, Sam Cox, Oliver Schilter, Carlo Baldassari, Andrew D White, and Philippe Schwaller. 2024. Augmenting large language models with chemistry tools. *Nature Machine Intelligence* (2024), 1–11.
*   Ma et al. (2023) Kaixin Ma, Hongming Zhang, Hongwei Wang, Xiaoman Pan, and Dong Yu. 2023. Laser: Llm agent with state-space exploration for web navigation. *arXiv preprint arXiv:2309.08172* (2023).
*   Ma et al. (2022) Zixian Ma, Rose Wang, Fei-Fei Li, Michael Bernstein, and Ranjay Krishna. 2022. Elign: Expectation alignment as a multi-agent intrinsic reward. *Advances in Neural Information Processing Systems* 35 (2022), 8304–8317.
*   McKenna et al. (2023) Peter E McKenna, Marta Romeo, Jhielson Pimentel, Mohammed Diab, Meriam Moujahid, Helen Hastie, and Yiannis Demiris. 2023. Theory of Mind and Trust in Human-Robot Navigation. In *Proceedings of the First International Symposium on Trustworthy Autonomous Systems* (Edinburgh, United Kingdom) *(TAS ’23)*. Association for Computing Machinery, New York, NY, USA, Article 29, 5 pages. [https://doi.org/10.1145/3597512.3597514](https://doi.org/10.1145/3597512.3597514)
*   McMillan and Hwang (2002) Sally J. McMillan and Jang-Sun Hwang. 2002. Measures of Perceived Interactivity: An Exploration of the Role of Direction of Communication, User Control, and Time in Shaping Perceptions of Interactivity. *Journal of Advertising* 31, 3 (2002), 29–42. [https://doi.org/10.1080/00913367.2002.10673674](https://doi.org/10.1080/00913367.2002.10673674) arXiv:https://doi.org/10.1080/00913367.2002.10673674
*   McNeese et al. (2021) Nathan J. McNeese, Mustafa Demir, Nancy J. Cooke, and Manrong She. 2021. Team Situation Awareness and Conflict: A Study of Human–Machine Teaming. *Journal of Cognitive Engineering and Decision Making* 15, 2-3 (2021), 83–96. [https://doi.org/10.1177/15553434211017354](https://doi.org/10.1177/15553434211017354) arXiv:https://doi.org/10.1177/15553434211017354
*   McNeese and Reddy (2015) Nathan J McNeese and Madhu C Reddy. 2015. Articulating and understanding the development of a team mental model in a distributed medium. In *Proceedings of the Human Factors and Ergonomics Society Annual Meeting*, Vol. 59\. SAGE Publications Sage CA: Los Angeles, CA, 240–244.
*   Mehrabian (2017) Albert Mehrabian. 2017. *Nonverbal communication*. Routledge.
*   Meltzoff (1999) Andrew N Meltzoff. 1999. Origins of theory of mind, cognition and communication. *Journal of communication disorders* 32, 4 (1999), 251–269.
*   Mooney et al. (1993) Christopher Z Mooney, Robert D Duval, and Robert Duvall. 1993. *Bootstrapping: A nonparametric approach to statistical inference*. Number 95\. sage.
*   Napierala (2012) Matthew A Napierala. 2012. What is the Bonferroni correction? *Aaos Now* (2012), 40–41.
*   Oliver (2019) Stephen Oliver. 2019. Communication and trust: rethinking the way construction industry professionals and software vendors utilise computer communication mediums. *Visualization in Engineering* 7, 1 (2019), 1.
*   OpenAI (2024) OpenAI. 2024. GPT-4o mini: Advancing Cost-Efficient Intelligence. [https://openai.com/index/gpt-4o-mini-advancing-cost-efficient-intelligence/](https://openai.com/index/gpt-4o-mini-advancing-cost-efficient-intelligence/). Accessed: 2024-09-05.
*   Oralbayeva et al. (2022) Nurziya Oralbayeva, Aidar Shakerimov, Shamil Sarmonov, Kanagat Kantoreyeva, Fatima Dadebayeva, Nuray Serkali, and Anara Sandygulova. 2022. K-Qbot: Language Learning Chatbot Based on Reinforcement Learning. In *2022 17th ACM/IEEE International Conference on Human-Robot Interaction (HRI)*. 963–967. [https://doi.org/10.1109/HRI53351.2022.9889428](https://doi.org/10.1109/HRI53351.2022.9889428)
*   O’Neill et al. (2022) Thomas O’Neill, Nathan McNeese, Amy Barron, and Beau Schelble. 2022. Human–autonomy teaming: A review and analysis of the empirical literature. *Human factors* 64, 5 (2022), 904–938.
*   Park et al. (2023) Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S Bernstein. 2023. Generative agents: Interactive simulacra of human behavior. In *Proceedings of the 36th annual acm symposium on user interface software and technology*. 1–22.
*   Peng et al. (2023) Jiankun Peng, Weiqi Chen, Yi Fan, Hongwen He, Zhongbao Wei, and Chunye Ma. 2023. Ecological Driving Framework of Hybrid Electric Vehicle Based on Heterogeneous Multi-Agent Deep Reinforcement Learning. *IEEE Transactions on Transportation Electrification* 10, 1 (2023), 392–406.
*   Peters (2006) Christopher Peters. 2006. Designing synthetic memory systems for supporting autonomous embodied agent behaviour. In *ROMAN 2006-The 15th IEEE International Symposium on Robot and Human Interactive Communication*. IEEE, 14–19.
*   Pinheiro and Bates (2006) José Pinheiro and Douglas Bates. 2006. *Mixed-effects models in S and S-PLUS*. Springer science & business media.
*   Pinski et al. (2023) Marc Pinski, Martin Adam, and Alexander Benlian. 2023. AI Knowledge: Improving AI Delegation through Human Enablement. In *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems* (Hamburg, Germany) *(CHI ’23)*. Association for Computing Machinery, New York, NY, USA, Article 25, 17 pages. [https://doi.org/10.1145/3544548.3580794](https://doi.org/10.1145/3544548.3580794)
*   Premack and Woodruff (1978) David Premack and Guy Woodruff. 1978. Does the chimpanzee have a theory of mind? *Behavioral and brain sciences* 1, 4 (1978), 515–526.
*   Rabinowitz et al. (2018) Neil Rabinowitz, Frank Perbet, Francis Song, Chiyuan Zhang, S. M. Ali Eslami, and Matthew Botvinick. 2018. Machine Theory of Mind. In *Proceedings of the 35th International Conference on Machine Learning* *(Proceedings of Machine Learning Research, Vol. 80)*, Jennifer Dy and Andreas Krause (Eds.). PMLR, 4218–4227. [https://proceedings.mlr.press/v80/rabinowitz18a.html](https://proceedings.mlr.press/v80/rabinowitz18a.html)
*   Rato et al. (2022) Diogo Rato, Marta Couto, and Rui Prada. 2022. Attributing Social Motivations to Changes in Agents’ Behavior and Appearance. In *Proceedings of the 10th International Conference on Human-Agent Interaction* (Christchurch, New Zealand) *(HAI ’22)*. Association for Computing Machinery, New York, NY, USA, 219–226. [https://doi.org/10.1145/3527188.3561925](https://doi.org/10.1145/3527188.3561925)
*   Russell and Norvig (2016) Stuart J Russell and Peter Norvig. 2016. *Artificial intelligence: a modern approach*. Pearson.
*   Salikutluk et al. (2024) Vildan Salikutluk, Janik Schöpper, Franziska Herbert, Katrin Scheuermann, Eric Frodl, Dirk Balfanz, Frank Jäkel, and Dorothea Koert. 2024. An Evaluation of Situational Autonomy for Human-AI Collaboration in a Shared Workspace Setting. In *Proceedings of the CHI Conference on Human Factors in Computing Systems* (Honolulu, HI, USA) *(CHI ’24)*. Association for Computing Machinery, New York, NY, USA, Article 300, 17 pages. [https://doi.org/10.1145/3613904.3642564](https://doi.org/10.1145/3613904.3642564)
*   Schelble et al. (2022) Beau G. Schelble, Christopher Flathmann, Nathan J. McNeese, Guo Freeman, and Rohit Mallick. 2022. Let’s Think Together! Assessing Shared Mental Models, Performance, and Trust in Human-Agent Teams. *Proc. ACM Hum.-Comput. Interact.* 6, GROUP, Article 13 (jan 2022), 29 pages. [https://doi.org/10.1145/3492832](https://doi.org/10.1145/3492832)
*   Shank et al. (2019) Daniel B. Shank, Christopher Graves, Alexander Gott, Patrick Gamez, and Sophia Rodriguez. 2019. Feeling our way to machine minds: People’s emotions when perceiving mind in artificial intelligence. *Comput. Hum. Behav.* 98, C (sep 2019), 256–266. [https://doi.org/10.1016/j.chb.2019.04.001](https://doi.org/10.1016/j.chb.2019.04.001)
*   Sharma et al. (2024) Manasi Sharma, Ho Chit Siu, Rohan Paleja, and Jaime D Peña. 2024. Why Would You Suggest That? Human Trust in Language Model Responses. *arXiv preprint arXiv:2406.02018* (2024).
*   Shen et al. (2024) Chenxinran Shen, Yan Xu, Ray Lc, and Zhicong Lu. 2024. Seeking Soulmate via Voice: Understanding Promises and Challenges of Online Synchronized Voice-Based Mobile Dating. In *Proceedings of the CHI Conference on Human Factors in Computing Systems* (Honolulu, HI, USA) *(CHI ’24)*. Association for Computing Machinery, New York, NY, USA, Article 921, 14 pages. [https://doi.org/10.1145/3613904.3642860](https://doi.org/10.1145/3613904.3642860)
*   Si et al. (2010) Mei Si, Stacy C. Marsella, and David V. Pynadath. 2010. Modeling appraisal in theory of mind reasoning. *Autonomous Agents and Multi-Agent Systems* 20, 1 (jan 2010), 14–31. [https://doi.org/10.1007/s10458-009-9093-x](https://doi.org/10.1007/s10458-009-9093-x)
*   Sonlu et al. (2024) Sinan Sonlu, Bennie Bendiksen, Funda Durupinar, and Uğur Güdükbay. 2024. The Effects of Embodiment and Personality Expression on Learning in LLM-based Educational Agents. *arXiv preprint arXiv:2407.10993* (2024).
*   Steyvers et al. (2022) Mark Steyvers, Heliodoro Tejeda, Gavin Kerrigan, and Padhraic Smyth. 2022. Bayesian modeling of human–AI complementarity. *Proceedings of the National Academy of Sciences* 119, 11 (2022), e2111547119.
*   Strachan et al. (2024) James WA Strachan, Dalila Albergo, Giulia Borghini, Oriana Pansardi, Eugenio Scaliti, Saurabh Gupta, Krati Saxena, Alessandro Rufo, Stefano Panzeri, Guido Manzi, et al. 2024. Testing theory of mind in large language models and humans. *Nature Human Behaviour* (2024), 1–11.
*   Strouse et al. (2021) DJ Strouse, Kevin McKee, Matt Botvinick, Edward Hughes, and Richard Everett. 2021. Collaborating with humans without human data. *Advances in Neural Information Processing Systems* 34 (2021), 14502–14515.
*   Sumers et al. (2023) Theodore R Sumers, Shunyu Yao, Karthik Narasimhan, and Thomas L Griffiths. 2023. Cognitive architectures for language agents. *arXiv preprint arXiv:2309.02427* (2023).
*   Swan et al. (2023) Melanie Swan, Takashi Kido, Eric Roland, and Renato P dos Santos. 2023. Math agents: Computational infrastructure, mathematical embedding, and genomics. *arXiv preprint arXiv:2307.02502* (2023).
*   Tudor Car et al. (2020) Lorainne Tudor Car, Dhakshenya Ardhithy Dhinagaran, Bhone Myint Kyaw, Tobias Kowatsch, Shafiq Joty, Yin-Leng Theng, and Rifat Atun. 2020. Conversational agents in health care: scoping review and conceptual analysis. *Journal of medical Internet research* 22, 8 (2020), e17158.
*   Vicari et al. (2003) Rosa M Vicari, Cecilia D Flores, Andre M Silvestre, Louise J Seixas, Marcelo Ladeira, and Helder Coelho. 2003. A multi-agent intelligent environment for medical knowledge. *Artificial Intelligence in Medicine* 27, 3 (2003), 335–366.
*   Wang et al. (2024b) Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, and Anima Anandkumar. 2024b. Voyager: An Open-Ended Embodied Agent with Large Language Models. *Transactions on Machine Learning Research* (2024). [https://openreview.net/forum?id=ehfRiF0R3a](https://openreview.net/forum?id=ehfRiF0R3a)
*   Wang and Goel (2022) Qiaosi Wang and Ashok K Goel. 2022. Mutual theory of mind for human-ai communication. *arXiv preprint arXiv:2210.03842* (2022).
*   Wang et al. (2021) Qiaosi Wang, Koustuv Saha, Eric Gregori, David Joyner, and Ashok Goel. 2021. Towards Mutual Theory of Mind in Human-AI Interaction: How Language Reflects What Students Perceive About a Virtual Teaching Assistant. In *Proceedings of the 2021 CHI Conference on Human Factors in Computing Systems* (Yokohama, Japan) *(CHI ’21)*. Association for Computing Machinery, New York, NY, USA, Article 384, 14 pages. [https://doi.org/10.1145/3411764.3445645](https://doi.org/10.1145/3411764.3445645)
*   Wang et al. (2024a) Qiaosi Wang, Sarah Walsh, Mei Si, Jeffrey Kephart, Justin D. Weisz, and Ashok K. Goel. 2024a. Theory of Mind in Human-AI Interaction. In *Extended Abstracts of the 2024 CHI Conference on Human Factors in Computing Systems* *(CHI EA ’24)*. Association for Computing Machinery, New York, NY, USA, Article 493, 6 pages. [https://doi.org/10.1145/3613905.3636308](https://doi.org/10.1145/3613905.3636308)
*   Wang et al. (2023) Xihuai Wang, Zheng Tian, Ziyu Wan, Ying Wen, Jun Wang, and Weinan Zhang. 2023. Order Matters: Agent-by-agent Policy Optimization. In *The Eleventh International Conference on Learning Representations*. [https://openreview.net/forum?id=Q-neeWNVv1](https://openreview.net/forum?id=Q-neeWNVv1)
*   Wang et al. (2024c) Xihuai Wang, Shao Zhang, Wenhao Zhang, Wentao Dong, Jingxiao Chen, Ying Wen, and Weinan Zhang. 2024c. ZSC-Eval: An Evaluation Toolkit and Benchmark for Multi-agent Zero-shot Coordination. arXiv:2310.05208 [cs.AI] [https://arxiv.org/abs/2310.05208](https://arxiv.org/abs/2310.05208)
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting elicits reasoning in large language models. *Advances in neural information processing systems* 35 (2022), 24824–24837.
*   Weisz et al. (2024) Justin D Weisz, Michael Muller, Arielle Goldberg, and Dario Andres Silva Moran. 2024. Expedient Assistance and Consequential Misunderstanding: Envisioning an Operationalized Mutual Theory of Mind. *arXiv preprint arXiv:2406.11946* (2024).
*   Wen et al. (2019) Ying Wen, Yaodong Yang, Rui Luo, Jun Wang, and Wei Pan. 2019. Probabilistic Recursive Reasoning for Multi-Agent Reinforcement Learning. In *International Conference on Learning Representations*. [https://openreview.net/forum?id=rkl6As0cF7](https://openreview.net/forum?id=rkl6As0cF7)
*   Wester et al. (2024) Joel Wester, Rune Møberg Jacobsen, Sander de Jong, Naja Kathrine Kollerup Als, Helena Bøjer Djernæs, and Niels van Berkel. 2024. Theory of Mind and Self-Presentation in Human-LLM Interactions. In *Adjunct Proceedings of the ACM SIGCHI Conference on Human Factors in Computing Systems*.
*   Wu et al. (2023) Jimmy Wu, Rika Antonova, Adam Kan, Marion Lepert, Andy Zeng, Shuran Song, Jeannette Bohg, Szymon Rusinkiewicz, and Thomas Funkhouser. 2023. Tidybot: Personalized robot assistance with large language models. *Autonomous Robots* 47, 8 (2023), 1087–1102.
*   Wu et al. (2021) Sarah A. Wu, Rose E. Wang, James A. Evans, Joshua B. Tenenbaum, David C. Parkes, and Max Kleiman-Weiner. 2021. Too Many Cooks: Bayesian Inference for Coordinating Multi-Agent Collaboration. *Topics in Cognitive Science* 13, 2 (2021), 414–432. [https://doi.org/10.1111/tops.12525](https://doi.org/10.1111/tops.12525) arXiv:https://onlinelibrary.wiley.com/doi/pdf/10.1111/tops.12525
*   Yao et al. (2022a) Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. 2022a. Webshop: Towards scalable real-world web interaction with grounded language agents. *Advances in Neural Information Processing Systems* 35 (2022), 20744–20757.
*   Yao et al. (2022b) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2022b. React: Synergizing reasoning and acting in language models. *arXiv preprint arXiv:2210.03629* (2022).
*   Yu et al. (2023) Chao Yu, Jiaxuan Gao, Weilin Liu, Botian Xu, Hao Tang, Jiaqi Yang, Yu Wang, and Yi Wu. 2023. Learning Zero-Shot Cooperation with Humans, Assuming Humans Are Biased. In *ICLR*. OpenReview.net.
*   Zhang et al. (2023a) Rui Zhang, Wen Duan, Christopher Flathmann, Nathan McNeese, Guo Freeman, and Alyssa Williams. 2023a. Investigating AI Teammate Communication Strategies and Their Impact in Human-AI Teams for Effective Teamwork. *Proc. ACM Hum.-Comput. Interact.* 7, CSCW2, Article 281 (oct 2023), 31 pages. [https://doi.org/10.1145/3610072](https://doi.org/10.1145/3610072)
*   Zhang et al. (2021a) Rui Zhang, Nathan J. McNeese, Guo Freeman, and Geoff Musick. 2021a. ”An Ideal Human”: Expectations of AI Teammates in Human-AI Teaming. *Proc. ACM Hum.-Comput. Interact.* 4, CSCW3, Article 246 (jan 2021), 25 pages. [https://doi.org/10.1145/3432945](https://doi.org/10.1145/3432945)
*   Zhang et al. (2023b) Shao Zhang, Hui Xu, Yuting Jia, Ying Wen, Dakuo Wang, Luoyi Fu, Xinbing Wang, and Chenghu Zhou. 2023b. GeoDeepShovel: A platform for building scientific database from geoscience literature with AI assistance. *Geoscience Data Journal* 10, 4 (2023), 519–537.
*   Zhang et al. (2024b) Shao Zhang, Jianing Yu, Xuhai Xu, Changchang Yin, Yuxuan Lu, Bingsheng Yao, Melanie Tory, Lace M. Padilla, Jeffrey Caterino, Ping Zhang, and Dakuo Wang. 2024b. Rethinking Human-AI Collaboration in Complex Medical Decision Making: A Case Study in Sepsis Diagnosis. In *Proceedings of the CHI Conference on Human Factors in Computing Systems* (Honolulu, HI, USA) *(CHI ’24)*. Association for Computing Machinery, New York, NY, USA, Article 445, 18 pages. [https://doi.org/10.1145/3613904.3642343](https://doi.org/10.1145/3613904.3642343)
*   Zhang et al. (2024a) Wenqi Zhang, Ke Tang, Hai Wu, Mengna Wang, Yongliang Shen, Guiyang Hou, Zeqi Tan, Peng Li, Yueting Zhuang, and Weiming Lu. 2024a. Agent-Pro: Learning to Evolve via Policy-Level Reflection and Optimization. In *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), ACL 2024, Bangkok, Thailand, August 11-16, 2024*, Lun-Wei Ku, Andre Martins, and Vivek Srikumar (Eds.). Association for Computational Linguistics, 5348–5375. [https://aclanthology.org/2024.acl-long.292](https://aclanthology.org/2024.acl-long.292)
*   Zhang et al. (2021b) Weinan Zhang, Xihuai Wang, Jian Shen, and Ming Zhou. 2021b. Model-based Multi-agent Policy Optimization with Adaptive Opponent-wise Rollouts. In *Proceedings of the Thirtieth International Joint Conference on Artificial Intelligence, IJCAI-21*, Zhi-Hua Zhou (Ed.). International Joint Conferences on Artificial Intelligence Organization, 3384–3391. [https://doi.org/10.24963/ijcai.2021/466](https://doi.org/10.24963/ijcai.2021/466) Main Track.
*   Zhang et al. (2023c) Ziqian Zhang, Lei Yuan, Lihe Li, Ke Xue, Chengxing Jia, Cong Guan, Chao Qian, and Yang Yu. 2023c. Fast teammate adaptation in the presence of sudden policy change. In *Uncertainty in Artificial Intelligence*. PMLR, 2465–2476.
*   Zhang et al. (2024c) Zheyuan Zhang, Daniel Zhang-Li, Jifan Yu, Linlu Gong, Jinchang Zhou, Zhiyuan Liu, Lei Hou, and Juanzi Li. 2024c. Simulating Classroom Education with LLM-Empowered Agents. *arXiv preprint arXiv:2406.19226* (2024).

## Appendix A The Detailed Results of Questionnaires in Each Condition

In [Table 4](https://arxiv.org/html/2409.08811v1#A1.T4 "In Appendix A The Detailed Results of Questionnaires in Each Condition ‣ Mutual Theory of Mind in Human-AI Collaboration: An Empirical Study with LLM-driven AI Agents in a Real-time Shared Workspace Task"), we provide the items of the questionnaire used after each game, along with the corresponding mean human agreement levels in each group and condition, with standard deviations shown in parentheses.

Table 4\. The Results of the Questionnaire after Each Game. We report the mean values of the agreement the participants indicated and the standard deviation in brackets.

 | <svg height="19.07" overflow="visible" version="1.1" width="334.65"><g transform="translate(0,19.07) scale(1,-1)"><g class="ltx_svg_fog" transform="translate(0,0)"><g transform="translate(0,9.46) scale(1, -1)"><foreignobject height="9.46" overflow="visible" width="61.88">Statement</foreignobject></g></g> <g class="ltx_svg_fog" transform="translate(274.69,9.46)"><g transform="translate(0,9.61) scale(1, -1)"><foreignobject height="9.61" overflow="visible" width="59.96">Condition</foreignobject></g></g></g></svg> | Bi-Comm | H-Comm | A-Comm | No-Comm |
| --- | --- | --- | --- | --- |
| w/ MToM | w/o MToM | w/ MToM | w/o MToM | w/ MToM | w/o MToM | w/ MToM | w/o MToM |
| --- | --- | --- | --- | --- | --- | --- | --- |
| The agent is good at playing this game. | 4.25 (1.13) | 4.38 (0.65) | 4.18 (0.53) | 4.18 (0.53) | 4.06 (0.56) | 4.06 (0.56) | 4.22 (0.42) | 4.11 (0.58) |
| The agent and I are working fluently together | 3.75 (1.27) | 3.81 (0.70) | 3.94 (0.43) | 4.24 (0.57) | 3.65 (0.87) | 3.59 (1.63) | 3.83 (0.62) | 3.78 (0.77) |
| The agent and I contributed equally to the success. | 3.75 (1.13) | 3.81 (1.23) | 4.12 (0.49) | 3.94 (0.56) | 3.94 (1.06) | 4.00 (1.13) | 4.00 (0.47) | 3.72 (0.68) |
| I was the most important team member on the team. | 3.81 (1.23) | 3.88 (1.18) | 3.70 (0.60) | 3.70 (0.85) | 4.06 (0.93) | 3.94 (1.18) | 3.72 (0.45) | 3.94 (0.53) |
| The agent was the most important team member on the team | 3.75 (0.87) | 4.00 (0.80) | 3.88 (0.61) | 3.65 (0.74) | 3.88 (0.99) | 3.88 (1.23) | 3.83 (0.50) | 3.67 (0.82) |
| I can understand agent | 3.81 (1.36) | 3.81 (0.43) | 4.29 (0.47) | 4.24 (0.44) | 3.82 (1.03) | 3.65 (1.37) | 4.00 (0.47) | 3.83 (0.85) |
| I feel agent understand me | 3.94 (0.73) | 3.75 (1.00) | 3.59 (1.38) | 4.00 (0.88) | 3.59 (1.01) | 3.47 (1.39) | 3.83 (0.97) | 3.44 (0.85) |
| The agent and I are working on the same goal | 4.18 (0.96) | 4.44 (0.53) | 4.24 (0.44) | 4.58 (0.38) | 4.00 (1.00) | 4.12 (1.36) | 4.22 (0.54) | 4.00 (0.59) |
| I feel that it is easy to score high in this game. | 3.56 (1.60) | 3.81 (0.96) | 3.94 (0.43) | 4.18 (0.53) | 3.47 (1.64) | 3.47 (1.26) | 3.94 (0.76) | 3.94 (0.76) | 

## Appendix B The Open Questions in Each Condition

To gather participants’ perspectives on the agents’ capabilities and their communication process during the experiments, we design open questions tailored to each group’s different levels of communication interactivity.

For Bi-Comm Group:

*   •

    Have you sent messages to the agents? How do you think that sending messages to the agents has affected your own actions?

*   •

    Do you spend time paying attention to the messages sent by the agents? How do you think reading messages to the agents has affected your own actions?

*   •

    How do you feel about the messages from the agents? Have you responded to the agents’ messages with your actions? If so, please provide examples.

*   •

    Which agent (please use hat color for reference) do you believe is able to understand and respond to your messages? Please provide examples.

*   •

    How do you feel that your AI teammate understands you? Please explain which AI agent and why.

*   •

    How do you feel that you understand your AI teammate? Please explain which AI agent and why.

For H-Comm Group:

*   •

    Have you sent messages to the agents? How do you think that sending messages to the agents has affected your own actions?

*   •

    Which agent (please use hat color for reference) do you believe is able to understand and respond to your messages? Please provide examples.

*   •

    How do you feel that your AI teammate understands you? Please explain which AI agent and why.

*   •

    How do you feel that you understand your AI teammate? Please explain which AI agent and why.

For A-Comm Group:

*   •

    Do you spend time paying attention to the messages sent by the agents? How do you think reading messages to the agents has affected your own actions?

*   •

    How do you feel about the messages from the agents? Have you responded to the agents’ messages with your actions? If so, please provide examples.

*   •

    How do you feel that your AI teammate understands you? Please explain which AI agent and why.

*   •

    How do you feel that you understand your AI teammate? Please explain which AI agent and why.

For No-Comm Group:

*   •

    How do you feel that your AI teammate understands you? Please explain which AI agent and why.

*   •

    How do you feel that you understand your AI teammate? Please explain which AI agent and why.