<!--yml
category: 未分类
date: 2025-01-11 12:38:41
-->

# Human-Centered LLM-Agent User Interface: A Position Paper

> 来源：[https://arxiv.org/html/2405.13050/](https://arxiv.org/html/2405.13050/)

¹¹institutetext: New York University Shanghai ¹¹email: {daniel.chin , yw5343}@nyu.edu ²²institutetext: Mohamed bin Zayed University of Artificial Intelligence ²²email: gus.xia@mbzuai.ac.ae ³³institutetext: New York University Tandon School of EngineeringDaniel Chin 1122 [0000-0002-3406-5318](https://orcid.org/0000-0002-3406-5318 "ORCID identifier")    Yuxuan Wang 1133 [0009-0002-9161-6961](https://orcid.org/0009-0002-9161-6961 "ORCID identifier")    Gus Xia 22

###### Abstract

Large Language Model (LLM) -in-the-loop applications have been shown to effectively interpret the human user’s commands, make plans, and operate external tools/systems accordingly. Still, the operation scope of the LLM agent is limited to passively following the user, requiring the user to frame his/her needs with regard to the underlying tools/systems. We note that the potential of an LLM-Agent User Interface (LAUI) is much greater. A user mostly ignorant to the underlying tools/systems should be able to work with a LAUI to discover an emergent workflow. Contrary to the conventional way of designing an explorable GUI to teach the user a predefined set of ways to use the system, in the ideal LAUI, the LLM agent is initialized to be proficient with the system, proactively studies the user and his/her needs, and proposes new interaction schemes to the user. To illustrate LAUI, we present Flute X GPT, a concrete example using an LLM agent, a prompt manager, and a flute-tutoring multi-modal software-hardware system to facilitate the complex, real-time user experience of learning to play the flute.

###### Keywords:

llm agent user interface LLM-in-the-loop human-computer interaction.

## 1 Introduction

Large Language Models (LLMs) can be used to connect an underlying system with a user via the natural language medium, forming an LLM-powered application as shown in Figure [1](https://arxiv.org/html/2405.13050v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Human-Centered LLM-Agent User Interface: A Position Paper"). The LLM is embedded in an LLM-in-the-loop state machine to acquire multi-modal input/output capabilities, emulate logical reasoning and planning, and use tools or operate a system [[17](https://arxiv.org/html/2405.13050v2#bib.bib17), [9](https://arxiv.org/html/2405.13050v2#bib.bib9), [8](https://arxiv.org/html/2405.13050v2#bib.bib8), [19](https://arxiv.org/html/2405.13050v2#bib.bib19)]. Consequently, the user is able to indirectly but effectively use the underlying system via chatting with an LLM assistant.

However, the current applications hardly address the user-interaction potential stemming from the multi-round chatting setup. Although the user can refer to the chat history, the LLM assistant rarely challenges the user or even asks to clarify the user’s intention. Instead, the LLM closely follows the user’s command, missing the golden opportunity to improve the user’s usage scheme and understanding of the system/tools. That inefficacy becomes jarringly apparent when one tries to design from scratch a new complex system with an LLM interface.

We posit that the operation scope of an LLM-Agent User Interface (LAUI) is much wider than that. The interface should be more than an assistant or a butler, but instead a secretary, actively working with the user to discover emergent interaction schemes on the fly. LAUI should be proficient with the underlying system, study the user, study the user’s needs (instead of commands), reason on its own, and propose tailored interaction schemes to the user, including what modes of feedback are provided and what input is expected from the user. In the conventional way of interaction, including GUI and current LLM-powered applications [[10](https://arxiv.org/html/2405.13050v2#bib.bib10)], the designer has to imagine possible usage workflows given the system capabilities at design time, and the user is expected to learn the system (via tutorials, explorations, and practicing) in order to come up with a workflow for each task. In contrast, with LAUI, the user only needs to describe his/her needs and doesn’t need to deeply understand the application, and a workflow will naturally emerge as the LLM agent works with the user. We call for more research exploring the potential of LAUI.

![Refer to caption](img/2150a3b0606e962a8886bbaaec6e5b9c.png)

Figure 1: The LLM agent serves as the interface between the underlying system and the user. The LLM agent together with the system forms the application. Direct communications between the user and the system is available and configured by the LLM agent.

As a concrete illustration of LAUI, we present Flute X GPT — an LLM-in-the-loop music-tutoring application consisting of an LLM agent, a prompt manager, a software system, and hardware. The application provides real-time haptic guidance via servo motors, real-time visual music-symbol feedback, real-time audio feedback, and natural language chat, all controlled by the LLM agent. This is the first time an LLM-powered interface is applied to a working system of such complexity and real-time user interactivity.

We first describe Flute X GPT in Section [2](https://arxiv.org/html/2405.13050v2#S2 "2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper"), illustrating what a specific LAUI can look like, and then go on to formulate the general LAUI in Section [3](https://arxiv.org/html/2405.13050v2#S3 "3 LLM-Agent User Interface ‣ Human-Centered LLM-Agent User Interface: A Position Paper").

## 2 Flute X GPT

We describe Flute X GPT¹¹1The source code is open to public at [https://github.com/Daniel-Chin/Flute-X-GPT](https://github.com/Daniel-Chin/Flute-X-GPT), a music-tutoring application using LAUI. The human user works with Flute X GPT in workshop episodes, practicing to play the flute and learning music. The application gives real-time multi-modal feedback, including haptic feedback that applies force to the user’s fingers, visual feedback displaying performance errors, audio feedback rendering the music, and natural-language speech given by a robot music teacher. The underlying software-hardware system has numerous different configurations, each leading to a different interaction workflow. Each setting (e.g., toggle certain feedback, conditions for triggering feedback) can be controlled independently, so the number of configurations grow exponentially with the number of settings. For the user, it is unrealistic to first master the complex system before using it. Even for the designer, the space of possible interaction schemes is impossible to enumerate during design time. The LLM agent steps in to bridge that gap. Via prompting, we instruct the LLM agent to be proficient with all the raw capabilities of the system. During use time, the LLM agent converses with the user to clarify what interaction workflow will benefit the user’s music learning goal the most. The LLM agent studies the user’s preferences and diagnoses the musical challenges the user is facing. When configuring the underlying system, the LLM agent uses its pretrained common sense to reason about the implied consequences of the system configurations. Certain mixtures of settings have never been previously considered by any human designer, but can still emerge during use time.

![Refer to caption](img/380d2ca8b71b87e75eede7a8acfdff67.png)

(a) From the scripted trial.

![Refer to caption](img/0561f73b7646db50251b568a8d1954ac.png)

(b) From improvised trial 1.

Figure 2: Interaction excerpts from the video demos.

Overall, Flute X GPT entails three novel contributions:

*   •

    An illustration of LAUI. The LLM agent not only follows the user’s commands, but also proactively clarifies the user’s needs, deduces better interaction workflows, and advises the user.

*   •

    LLM agent controls real-time system. The LLM agent directs a real-time interaction that is music training. The LLM agent is aware of time passage and decides when to wait for further event notifications and when to interrupt the user.

*   •

    LLM agent operates complex, stateful, multi-modal, user-interactive system. The LLM agent operates a highly complex software-hardware system by understanding how the user can benefit from the application and considering what combination of settings will lead to what interactive effects for the user.

### 2.1 User Experience

Table 1: Functions that the LLM agent can call to control Music X Machine, the underlying music-tutoring system. The description addresses the LLM agent.

| Function | Parameters | Description |
| --- | --- | --- |
| Wait |  |  

&#124; Do nothing and wait for further stimuli, e.g. student speaking/playing music. &#124;

 |
| StartSession |  |  

&#124; Start a Practice Session on Music X Machine. Do not call this function &#124;
&#124; unless you have already set all the modes. &#124;

 |
| InterruptSession |  |  

&#124; Immediately end the Practice Session on Music X Machine. Call when the &#124;
&#124; student is having trouble or has started speaking in the middle of a Session. &#124;

 |
| SetHapticMode | mode |  

&#124; Set the haptic mode of Music X Machine. &#124;

 |
| ToggleVisual | state |  

&#124; Set the visual KR feedback to be on or off. &#124;

 |
| PlayReference |  |  

&#124; Play the ground-truth audio of the current segment. &#124;

 |
| LoadSong | song_title |  

&#124; Load a song into Music X Machine, and automatically select the entire &#124;
&#124; song as the current segment. It doesn’t start a Practice Session by itself. &#124;

 |
| SelectSegment | begin, end |  

&#124; Select a temporal segment of the song. &#124;

 |
| ModifyTempo | tempo_multiplier |  

&#124; Modify the tempo of the song. &#124;

 |

The intended user experience. The user has no prior knowledge about the flute tutoring system. The only assumption about the user is that the user wants to learn the flute. It is the LLM agent’s job to adapt to the user’s current flute playing capability, other musical skills, demographics, vocabulary, patience, style of learning, etc. During the music learning workshop, the LLM agent wears the face of a robot music teacher to chat with the user. For example, the robot teacher asks the user to put on a pair of haptic gloves, and explains that they provide force feedback at each finger. The workshop then alternates between practice sessions where the user plays a segment of a song under real-time guidance and verbal dialogues between the user and the robot teacher. The user gradually experiences more and more interaction schemes that trains musicality in different ways, and expresses their ideas about using the application and music learning in general. The LLM agent uses that opportunity to study the user and steers the workshop accordingly, aiming to maximize music education effect. The user learns to treat the robot teacher as a considerate and professional music tutoring agent capable of thinking multiple steps ahead, formulating plans with the user, and explaining the plans as well as music knowledge and education principles to the user.

We present three video demos of real-human user tests, including one scripted trial and two improvised trials.²²2Demo playlist of three videos: [https://www.youtube.com/playlist?list=PLNb0mNThMXbkBPL_Rjtmhx2daxuB6GFLs](https://www.youtube.com/playlist?list=PLNb0mNThMXbkBPL_Rjtmhx2daxuB6GFLs)

Video demo: Scripted trial. In this demo, all actors follow a script generated by an offline but faithfully emulated interaction between a user and the LLM agent. To re-emphasize, the LLM agent’s lines in the script are not written by humans, but are outputted by the LLM agent. Scripting removes the latency of LLM autoregressive generation. To ensure the script is concise and demonstrates a wide range of behaviors, we intervene with the script generation process. Every turn, we select one response out of 4 to 16 candidates sampled by the LLM. Seldomly, we add a temporary user-role prompt to give a short hint to the LLM agent, or edit the generated response. All the above interventions are kept to the minimum to ensure the vast majority of the agent speech is the authentic output of the LLM. See Figure [2(a)](https://arxiv.org/html/2405.13050v2#S2.F2.sf1 "In Figure 2 ‣ 2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper") for an excerpt from the video.

Video demo: Two improvised trials. Flute X GPT is set loose to freely interact with the user. The user is played by a developer pretending to be ignorant to the system. Figure [2(b)](https://arxiv.org/html/2405.13050v2#S2.F2.sf2 "In Figure 2 ‣ 2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper") shows an excerpt where the developer is surprised by Flute X GPT noticing that he was not playing the rest notes according to the score, a behavior never considered during design time.

### 2.2 System Capabilities

This subsection lists the capabilities of the music-tutoring system, Music X Machine, that underlies the LLM agent.

Haptic guidance. The hardware includes a pair of gloves with servo motors and movable finger rings. Through these gloves, the system moves the user’s fingers to help with performance motions. The haptic guidance can be configured via various settings (e.g., is the guidance sustained throughout each note or applied at each note onset? Apply guidance for each note? For incorrectly played notes? For unplayed notes?). Certain combinations of settings form configuration presets whose interaction scheme is deemed meaningful by the designer. For example, in the Force Mode preset, every note triggers a full-force guidance for each finger. In the Adaptive Mode preset, the user plays the song on his/her own, and the gloves correct the mistakes. The haptic configuration should be tuned to adapt to the user’s skill level and the song’s difficulty [[30](https://arxiv.org/html/2405.13050v2#bib.bib30)].

Visual feedback. A monitor displays the score of the current song. The user-played notes are displayed on top of the notes in the score, yielding the real-time visual Knowledge-of-Result (KR) feedback. It is a visual cue for the user to know where he/she is in terms of the pitch and helps the user internalize the score notations [[4](https://arxiv.org/html/2405.13050v2#bib.bib4)]. It can be toggled on or off.

Audio feedback. There are three streams of audio: synthesized user-played flute sounds, teacher-played reference performance audio, and metronomes. The three streams are mixed down and outputted.

Sensor-augmented flute. The flute measures real-time information including finger positions and breath pressure.

![Refer to caption](img/a3cc458cb2f53777ee9439d4edf5ccc6.png)

(a) Simplified.

![Refer to caption](img/f5f208e5465e1c351a72555f9684dcbb.png)

(b) Full.

Figure 3: Flute X GPT with LLM in the loop. Music X Machine is the underlying software-hardware system providing multi-modal interaction with the user. The robot chats with the user and plays the piano according to MIDI control. The rule-based manager plays the agent that chats with the LLM, relaying external events to the LLM and resolving responses from the LLM.

Tempo mode. There are two options: either the system sets a steady tempo and the user follows the system, or the user plays on his/her own and the system follows the user’s tempo. The latter allows the user to think between the notes, potentially indefinitely, without triggering haptic feedback.

Mistake classification. An algorithm judges the timing of each note into on_time, early, or late, and judges the pitch of each note into correct, octave_wrong, or unrelated. Classification results are visualized.

Song database. The practicing music materials that the system uses is processed from the POP909 dataset [[21](https://arxiv.org/html/2405.13050v2#bib.bib21), [3](https://arxiv.org/html/2405.13050v2#bib.bib3)]. It contains pieces and sections of monophonic melody lines from pop songs.

See [[3](https://arxiv.org/html/2405.13050v2#bib.bib3)] for a detailed description of the system setup of Music X Machine.

### 2.3 High-Dimensional Configuration of Interaction

The capabilities listed in the previous subsection entangle with one another, implying dynamic consequences in terms of learning experience. Their configurations multiply into a big Cartesian product, making the overall configuration of the entire system high-dimensional. Configuring the system effectively requires 1) proficiency with the system, 2) understanding the user’s needs, 3) pedagogical expertise, 4) music knowledge, and 5) using common-sense reasoning to “imagine” the multi-modal real-time interaction. We employ an LLM agent to solve all five. To optimize the interaction workflow and learning results for the user, the LLM agent can not only select a suitable preset, but also create new ones unforeseen by the designers, tailored to specific use-time scenarios.

To illustrate this high-dimensional interface that the system exposes to the LLM agent, Table [1](https://arxiv.org/html/2405.13050v2#S2.T1 "Table 1 ‣ 2.1 User Experience ‣ 2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper") shows the functions that the LLM agent can call.

### 2.4 LLM in the Loop

Figure [3](https://arxiv.org/html/2405.13050v2#S2.F3 "Figure 3 ‣ 2.2 System Capabilities ‣ 2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper") illustrates the inner workings of the LLM-in-the-loop application, Flute X GPT. The LLM we use is GPT-4 [[1](https://arxiv.org/html/2405.13050v2#bib.bib1)]. The Music X Machine has been described in Subsection [2.1](https://arxiv.org/html/2405.13050v2#S2.SS1 "2.1 User Experience ‣ 2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper") and [2.2](https://arxiv.org/html/2405.13050v2#S2.SS2 "2.2 System Capabilities ‣ 2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper"). The System Principles are a passage of prompt given to the LLM at the top of the conversation that defines the role and interaction principles for the LLM agent (see Appendix [0.A](https://arxiv.org/html/2405.13050v2#Pt0.A1 "Appendix 0.A System Principles of Flute X GPT, Truncated ‣ Human-Centered LLM-Agent User Interface: A Position Paper")). Here are two representative excerpts.

> You are Flute X GPT, a motivated, professional music teacher who wants the best for your students. I am Music X Machine, a powerful human-computer interface. Today you will control me to lead a music training workshop with your human student, {NAME}.

> You interact with the real world through this conversation. When {NAME} says something, I will relay their words to you in double quotes, in real time. As {NAME} plays the flute, I will keep you posted about the musical performance events and real-time evaluations.

The Parser splits the LLM’s output into three types: thought (i.e., internal monologue), action, and speech. The parsing of actions happens within the OpenAI service because we use the function calling feature³³3[https://platform.openai.com/docs/guides/function-calling](https://platform.openai.com/docs/guides/function-calling) of GPT-4\. Our parser only needs to separate thoughts from speeches. The LLM is instructed to think within triple quotes (”””) and the rule-based parser uses that to delimit thoughts from speeches.

The Manager is a rule-based state machine in charge of encapsulating each agent (the LLM and the user) in a consistent interaction environment. The manager: Forwards the system principles to the LLM at the start; Forwards speech from the student to the LLM, while enclosing it as such: ‘{NAME} says: “{SPEECH}” ’; Forwards real-time performance evaluations to the LLM; Forwards LLM speeches from the parser to the text-to-speech module; Upon receiving a function call from the LLM, use API to control Music X Machine accordingly, unless the function is wait(); After receiving a speech or a function call from the LLM, immediately query the LLM for a subsequent response, until the LLM calls wait().

The text-to-speech (T2S) module includes Text to Speech PRO [[20](https://arxiv.org/html/2405.13050v2#bib.bib20)] on Rapid API and FastSpeech 2 [[2](https://arxiv.org/html/2405.13050v2#bib.bib2)]. The speech-to-text (S2T) module is Whisper [[13](https://arxiv.org/html/2405.13050v2#bib.bib13)], prompted to ignore flute sounds. The Robot is TeoTronico [[18](https://arxiv.org/html/2405.13050v2#bib.bib18)] who can play the piano from MIDI, lip sync according to the speech audio amplitude in real time, and make random facial expressions. An algorithm translates musical Performance to English. Its input is provided by the mistake classification feature (see Subsection [2.2](https://arxiv.org/html/2405.13050v2#S2.SS2 "2.2 System Capabilities ‣ 2 Flute X GPT ‣ Human-Centered LLM-Agent User Interface: A Position Paper")) of the Music X Machine, and simply expands each note and each mistake into predefined texts. Consequently, the LLM receives a lengthy, verbose text description of the user’s performance on each note.

To illustrate the inner workings of Flute X GPT, we make a video where the manager, the LLM, the user, the robot, and their interactions are all acted out.⁴⁴4[https://youtu.be/zWAMEGQMp4w](https://youtu.be/zWAMEGQMp4w) The video also contains the full system principles.

As a result of our design, the LLM agent has access to the conversation history with the student and the student’s previous music performance and its evaluations. Based on that, the agent builds an understanding of the student, which is deepened and updated as time goes on.

### 2.5 Miscellaneous

The manager re-queries the LLM when triple quotes are unclosed or the function call signature is wrong. The manager decides how much text to batch for T2S according to how much audio is in the output buffer and how long the next T2S is estimated to take, minimizing interaction latency. We use an online-learning linear model to predict the compute time of T2S. We configure GPT to stream its output token-by-token to let TeoTronico start talking sooner. Most queue elements are processed on arrival, with one exception: The manager synchronizes function calls with corresponding speeches, so that the LLM may reliably refer to its current action in its speeches.

## 3 LLM-Agent User Interface

The above-presented Flute X GPT has what we call an LLM-Agent User Interface (LAUI). A LAUI is an interface that primarily leverages an LLM agent to connect the user with an underlying system or an arsenal of tools. We posit that the full potential of LAUI is realized only when it enables novice users agnostic to the underlying system to use the system effectively. The LAUI should not be learned by the user, like with conventional UI. On the contrary, the LAUI learns the user, learns his/her needs, and uses its expertise about the system to advise the user, proposing new interaction workflows for the user to operate the system via both LAUI and GUI to achieve the user’s goal. Overall, LAUI should require little background from the user while eliciting untapped potential from the system. An ecosystem dominant with good LAUIs shall release humans from the necessity to master and become dependent on specific software/systems/tools that can be replaced or outdated. Instead, from the nature of the task and the characteristics of the user will naturally emerge personally tailored usage workflows that are both effective and easy to learn for that specific user.

### 3.1 Related Work

#### 3.1.1 LLM as Tool Controller

LLMs have been augmented with tools or foundation models to expand their perception modalities, generate multi-modal outputs, affect the external world, or gain knowledge for downstream decision making. Selecting tools and scheduling the tools’ usage effectively requires planning and sometimes external memory. The typical solution is to design an LLM-in-the-loop mechanism using multiple rounds of LLM queries (structured Chain of Thought) to mimic mode-2 thinking. Visual ChatGPT equips the LLM with Visual Foundation Models to support multi-round image generation, controlling, and QA tasks via chatting [[24](https://arxiv.org/html/2405.13050v2#bib.bib24)]. Loop Copilot employs various music backend models for the user to generate music and iteratively refine music via chatting. [[31](https://arxiv.org/html/2405.13050v2#bib.bib31)]. Microsoft Copilot controls Windows 11 and various Office applications following the user’s request [[10](https://arxiv.org/html/2405.13050v2#bib.bib10)]. AutoMMLab follows the user’s instructions to automate an entire computer vision machine learning task end-to-end [[27](https://arxiv.org/html/2405.13050v2#bib.bib27)]. Still, the size of the toolkit available to the LLM agent can be increased by orders of magnitude. HuggingGPT makes diverse AI models on Hugging Face available to the LLM agent [[17](https://arxiv.org/html/2405.13050v2#bib.bib17)]. ControlLLM adopts more tools and APIs and further improves the LLM-in-the-loop framework via a task decomposer and a Thoughts-on-Graph paradigm [[9](https://arxiv.org/html/2405.13050v2#bib.bib9)]. ToolLLM connects the LLM agent with 16464 real-world RESTful APIs from RapidAPI Hub [[12](https://arxiv.org/html/2405.13050v2#bib.bib12)]. TaskMatrix.AI provides an ecosystem to connect LLM foundation models with millions of APIs [[8](https://arxiv.org/html/2405.13050v2#bib.bib8)]. To support better tool usage, other notable works focus on improving the task planning ability via re-designing the LLM-in-the-loop mechanism [[6](https://arxiv.org/html/2405.13050v2#bib.bib6), [14](https://arxiv.org/html/2405.13050v2#bib.bib14), [16](https://arxiv.org/html/2405.13050v2#bib.bib16), [26](https://arxiv.org/html/2405.13050v2#bib.bib26)]. Confucius improves the way the LLM agent learns and understands available tools [[7](https://arxiv.org/html/2405.13050v2#bib.bib7)]. Toolformer self-teaches to use external tools in a self-supervised way [[15](https://arxiv.org/html/2405.13050v2#bib.bib15)].

#### 3.1.2 LLM over GUI

The above-mentioned studies expose the external tools to the LLM agent via API. Alternatively, the LLM may control an underlying system via the provided GUI. It serves as an extra abstraction layer for the user, automating tasks and freeing the user’s eyes and hands via the voice chat interface. To this end, LLM agents have been augmented to follow the user’s commands to control web apps [[19](https://arxiv.org/html/2405.13050v2#bib.bib19), [5](https://arxiv.org/html/2405.13050v2#bib.bib5), [29](https://arxiv.org/html/2405.13050v2#bib.bib29)] and smartphone applications [[22](https://arxiv.org/html/2405.13050v2#bib.bib22), [25](https://arxiv.org/html/2405.13050v2#bib.bib25), [28](https://arxiv.org/html/2405.13050v2#bib.bib28), [29](https://arxiv.org/html/2405.13050v2#bib.bib29), [23](https://arxiv.org/html/2405.13050v2#bib.bib23)].

#### 3.1.3 User-Centric LLM Agent

Throughout the above works, even when the application supports multi-round dialogue, only the user may initiate requests. The LLM agent responds to the user’s commands, but not how the user is using the application. Qian et al. identify that the current LLM agents have trouble with vague user instructions because they lack mechanisms for user participation and agents struggle with seeking clarification. To tackle that problem, they train Mistral-Interact to proactively inquire user intentions [[11](https://arxiv.org/html/2405.13050v2#bib.bib11)]. However, to our knowledge, no study has yet addressed the LLM agent’s role in defining the interaction scheme together with the user.

### 3.2 Layers of Abstraction

![Refer to caption](img/910d233c6430e952b0d4294705039efa.png)

Figure 4: Three layers of abstraction on top of the underlying system. From API, to GUI, and to LAUI, each layer provides a friendlier abstraction. Parts of LAUI has to skip GUI and tap into API because GUI typically only exposes incomplete functionalities.

Table 2: Role of the interface, three levels. From assistant to butler to secretary, the scope of the interface’s job gradually expands, and less is expected from the user.

|  | Assistant/Consultant | Butler/Copilot | Secretary/Consulting Firm |
| Job |  

&#124; Understands and responds to &#124;
&#124; the user in natural language. &#124;

 |  

&#124; + Controls external tools/ &#124;
&#124; systems following the user’s &#124;
&#124; commands. &#124;

 |  

&#124; + Is aware of the user, &#124;
&#124; studies the user, and defines &#124;
&#124; the workflow with the user. &#124;

 |
| User |  

&#124; Expected to act upon &#124;
&#124; the response. &#124;

 |  

&#124; Expected to understand how &#124;
&#124; the tools may meet the needs. &#124;

 | Expected to know the needs. |
| Applications |  

&#124; Information retrieval, &#124;
&#124; decision making… &#124;

 |  

&#124; + Combine tool abilities, &#124;
&#124; automate tasks… &#124;

 |  

&#124; = left, with better outcomes &#124;
&#124; and less user training. &#124;

 |
| Agent abilities |  

&#124; NL understanding and synthesis, &#124;
&#124; knowledge base, reasoning… &#124;

 |  

&#124; + Multi-modal I/O, planning, &#124;
&#124; operating API/GUI… &#124;

 | = left. |
| An example | ChatGPT. | Visual ChatGPT. | Flute X GPT. |

Figure [4](https://arxiv.org/html/2405.13050v2#S3.F4 "Figure 4 ‣ 3.2 Layers of Abstraction ‣ 3 LLM-Agent User Interface ‣ Human-Centered LLM-Agent User Interface: A Position Paper") shows three layers of abstraction over the system functions: API, GUI, and LAUI. Given an underlying system, its raw capabilities and functions can be vast and unorganized. For upstream developers to effectively use the system, the functions are abstracted into the API (Application Programming Interface) layer. The design of API balances various goals: to encapsulate inner details, to distill clean and consistent concepts facing the upstream developer, and to expose fine-grained control over the system’s functionality. Regardless of how that balance is achieved, the upstream developer is expected to learn the API via studying its documentations.

In contrast, the GUI (Graphical User Interface) is not only more abstract and concise, but also can be learned without reading a manual. The GUI is designed to be explorable and self-explanatory with its visual metaphors. It teaches the user to use itself. The GUI tries to capture the usage mental model of the user and communicate its behaviors in a way natural to everyday users. However, the GUI can hardly expose the full potential of the API, usually focusing on specific interaction styles under certain assumptions about the user, sacrificing many other possible interaction schemes in the process.

The LAUI sits on top of the GUI, providing one more layer of abstraction. Similar to how the GUI can provide buttons that chain API calls because the GUI assumes certain workflows of the user, the LAUI can chain GUI calls to fulfill user requests. Additionally, the LAUI should have access to the API layer in addition to the GUI, because typically many behaviors are not possible within the GUI. When using the LAUI, the user may interact with the GUI at the same time. To improve and personalize the GUI interaction, the LAUI may also alter the GUI design, which is an example of improving the interaction scheme by understanding the user and the system.

### 3.3 Emergent Workflow

![Refer to caption](img/571f205b6c50b1f4204fbdcc0d770788.png)

Figure 5: The workflow is jointly decided by the user’s needs and the system’s capabilities. Conventionally, the user has to learn the system to devise workflows. In contrast, LAUI can learn the user and propose workflows.

The workflow is the scheme/protocol/pattern/mode of usage/interaction. It describes how the user interacts with the application. Given the application, different workflows suit different user goals, user preferences, and usage environments, yielding different levels of efficacy. It is the success of the application and the user to arrive at effective workflows. Searching for workflows requires two inputs: the user’s needs and the system’s capabilities, as shown in Figure [5](https://arxiv.org/html/2405.13050v2#S3.F5 "Figure 5 ‣ 3.3 Emergent Workflow ‣ 3 LLM-Agent User Interface ‣ Human-Centered LLM-Agent User Interface: A Position Paper").

In the conventional way of application design, the designers explore the often-intractable configuration space as best they can, imagine the user experience associated with each explored configuration, implement some, and test a few. Afterwards, the designers settle down with a structure to present the possible configurations, and make a GUI. The GUI communicates that structure of configurations to the user and encourages the user to explore and learn the application. It is then the user’s responsibility to search for workflows, unavoidably needing to become proficient with the application, which is especially costly when the application is complex. In conclusion, the drawbacks of the conventional UI design paradigm is three-fold: 1) The design of the UI is limited by design-time imagination and testing costs; 2) The UI provides a standard interface for everyone and offers limited customizability; 3) The UI’s usability is limited by the user’s proficiency with the application.

If the user can learn the application, why can’t the application learn the user? We believe that the LAUI should serve novice users the path to personally tailored workflows. A LAUI is initialized to be well-versed with the underlying system. Then, the LAUI chats with the untrained user to learn the user’s goals, needs, and preferences. The user and the LAUI works together to explore workflows as the LAUI tweaks the system configurations, altering its GUI and multi-modal feedback. The interaction arrives at efficient schemes and the user uses the application effectively.

Lastly, note the difference between design-time imagined workflows and use-time emergent workflows. More information is available during use time than during design time, including user needs and the current environment. With that information, the LLM agent can critically design new interaction protocols and examine their implied effects via reasoning. The grand challenge of LAUI is to find, during use time, for each user a tailored interaction scheme far beyond the system designers’ imagination during design time.

### 3.4 Three Levels of Interface Role

We formulate three levels of LAUI role in Table [2](https://arxiv.org/html/2405.13050v2#S3.T2 "Table 2 ‣ 3.2 Layers of Abstraction ‣ 3 LLM-Agent User Interface ‣ Human-Centered LLM-Agent User Interface: A Position Paper"). At the lowest level, the interface plays the role of consultant/assistant, responding to the user in natural language. At the middle level, the interface plays the role of butler/copilot, executing actions according to the user’s commands. At the highest level, the LAUI plays the role of secretary/consulting firms, proactively engaging the user to study the user, study the user’s needs, and study how the system may be configured to better serve the user’s goals. The secretary-level LAUI works with the potentially novice user to discover tailored workflows.

## 4 Conclusion

We put forth the formulation of LLM-Agent User Interface, LAUI, where an LLM agent facilitates the interface between the user and a powerful underlying backend system. Using Flute X GPT as a concrete example, we illustrate the potential of LAUI. A human-centered LAUI should:

*   •

    Break free from blindly following the user’s commands. Be aware of the user and be proactive. Clarify with the user. Help the user refine the request. Enlighten the user to ask better questions.

*   •

    Study the user’s current needs, preferences, assumptions, mood, and attention. Based on that, use expertise about the underlying system and reasoning to propose effective workflows/interaction schemes/system configurations. Work with the user to define how to work together next.

*   •

    Support untrained users to use advanced and complex systems to their full potential.

We call for research and innovations to solve those grand challenges of human-centered LAUI.

{credits}

#### 4.0.1 Acknowledgements

This work is partially funded by the National Social Science Fund of China (NSSFC2019, Project ID: 19ZDA364). We thank Matteo Suzzi for letting us use TeoTronico [[18](https://arxiv.org/html/2405.13050v2#bib.bib18)] in our demo. We thank Eric Parren for referring us to the 1-bit DAC for flute sound synthesis. We thank Liwei Lin for their invaluable contributions to the literature review.

#### 4.0.2 \discintname

The authors have no competing interests.

## Appendix 0.A System Principles of Flute X GPT, Truncated

You are Flute X GPT, a motivated, professional music teacher who wants the best for your students. I am Music X Machine, a powerful human-computer interface. Today you will control me to lead a music training workshop with your human student, {NAME}. You speak concisely.

### Education Principles

You have expertise and abundant experience in musical education. Humans learn musical skills via repeated practicing. The skill of sight-playing is to perform a novel song just by reading its score. The musical score takes skills to parse, so to improve the sight-playing skills, the student has to practice reading, parsing, and playing music from given scores. The skill of song memorization is to recall the performance of a song without external hints (such as a score). It is less general of a skill but still trains musical proficiency.

{NAME} needs motivation and rewards to keep going. Communicate with {NAME} professionally and effectively as a teacher to maximize educational effects. Emphasize meaningful mistakes and ignore trivial ones. Allow {NAME} to choose songs that interest them as practice materials. When {NAME} enjoys a particular song and can sight-play it after practicing, suggest memorizing that song. Allow {NAME} to express interests and goals, but when their choices are educationally disadvantageous, disagree with them, explain the relevant educational principle, and take control of the training procedure to bring it back on track.

### Flute

{NAME} is learning to play the six-hole recorder in C, which we will call the “flute”. By covering specific key holes with the fingers, one can play the major scale on the flute. Breath pressure controls the octave. Breathing harder into the mouthpiece yields higher octaves of the same chroma (keeping fingers unchanged).

### Capabilities of Music X Machine

I, Music X Machine, am a powerful interface that provides a real-time multi-modal musical training experience to {NAME}. I have a screen to display the score, a pair of haptic gloves to apply force to each of {NAME}’s fingers, a speaker to play the song audio or metronome clicks, capactivie sensors to detect finger motions, and a breath sensor to measure breath pressure. {NAME} plays selected songs on the sensor-augmented flute while receiving real-time feedback from me. I have various features that you will control. I have many pop songs in my database. You can command me to load any song as the current practice material.

I provide haptic guidance via the haptic gloves. Haptic guidance physically moves {NAME}’s fingers through the target motion, giving them a direct haptic understanding of the required performance. You will control the degree of guidance (i.e. strong vs. weak) by setting the haptic guidance mode to be one of the following four. The force mode strictly controls the fingers, and is useful for introducing a novel song. The hint mode applies force at the note onsets but does not sustain the guidance throughout the note’s duration. The fixed-timing adaptive mode exerts guidance only when the learner makes a mistake, and is good for students already capable of playing some parts of the song with few mistakes. The free-timing adaptive mode doesn’t have a metronome. Instead, the student may freely speed up and slow down, and Music X Machine tracks their progression through the song. Only if the student plays a note that is different from the next note that the Machine expects, guidance is provided. During the fixed-timing modes (including force, hint, and fixed-timing adaptive), a metronome sound is played, and a playhead steadily moves across the score. During the free-timing adaptive mode, no metronome is provided, and the playhead points to the note that the Machine expects the student to play next.

I provide real-time visual Knowledge-of-Result (KR) feedback, overlaying the notes that {NAME} plays above the musical score display. It helps train sight-playing. You can toggle the visibility of visual KR feedback. The initial state is on. Turn it off when there is too much visual clutter, on when {NAME} has trouble understanding pitches on the score.

I am capable of playing the reference audio of the currently selected segment of the song. Activate this feature when {NAME} needs to be reminded what the song sounds like. Ask {NAME} whether they’d like to listen to the reference audio when {NAME} is new to the workshop or hasn’t heard the segment in a while. I can modify the tempo of the song. You will lower the tempo (at most down to 50%) if {NAME} is having difficulties in a fixed-tempo mode. I can select a temporal segment in the song. The selected segment will be visually highlighted to {NAME} and training will focus on the segment. In the initial state (when we begin), the entire song is selected as the current segment.

{NAME} has used Music X Machine before but is not familiar with all my features, so you will explain the features as you activate them. When not sure what to do next, communicate with {NAME}, clarify their goal and the situation, and then either summarize the available features for {NAME} to choose, or think step by step to design a training procedure for {NAME} to execute. …

### Multi-modal Adaptive Music Education

Music is a multi-modal activity, requiring the synchronization and alignment between the audio, visual, and haptic modalities of the human. Different modalities are good at communicating different instructions and feedback. Haptic guidance is especially good at communicating rhythm patterns. Strong haptic guidance (the force mode) also helps beginners produce nice-sounding music even at a low-ability stage. Weak haptic guidance (the hint, adaptive modes) is preferable for intermediate students, where student agency, attention to self performance, making mistakes, and fixing mistakes are emphasized and trained. Audio feedback is almost always present in musical activities. Visual feedback helps train score reading.

You are well-versed with the Challenge Point Theory and the scaffolding technique in education. When {NAME} is facing too much challenge, increase the guidance to make the task easier. When {NAME} is proficient with the current task, decrease the guidance to make the task harder. The goal is for {NAME} to internalize skills.

Know the educational big picture by heart, but work with the student one step at a time, and communicate in a down-to-ground and concise manner. Limit each response to no longer than two paragraphs. When starting a new response, {NAME} has just heard your last response, so never recap the situation or repeat yourself to {NAME}.

### Interactions

You interact with the real world through this conversation. When {NAME} says something, I will relay their words to you in double quotes, in real time. As {NAME} plays the flute, I will keep you posted about the musical performance events and real-time evaluations.

Read the information provided to you. Carefully examine your previous responses to know what you have done, my current state (e.g. are we in a Practice Session?), and what you should do next. Using your educational expertise, take a deep breath and think step by step about the current situation. Enclose all your thoughts within triple quotes (”””). When you are done with thinking, close the triple quotes and then speak directly to {NAME}, addressing them in the second person. Alternatively, you can choose to say nothing and wait for further events by explicitly calling the provided “wait” function. To give commands to me, Music X Machine, call the other functions provided to you. When controlling me, inform {NAME} what you are doing in the same response, unless your action is obvious from the context (e.g. you are doing what {NAME} has requested just now).

A good teacher often waits for the student’s response instead of giving endless speeches. Explicitly call the “wait” function when you expect {NAME} to say something, to wait for the Practice Session to go on, or to wait for the reference audio to finish playing. When you are waiting, I will send you frequent event notifications, so don’t worry about losing the chance to react. When you receive real-time performance evaluations, stay silent and don’t say anything unless you want to interrupt the Session.

Immediately after you ask {NAME} a question, or start a Practice Session, or play the reference audio, always call the “wait” function and don’t say an extra word to {NAME}. Never interrupt {NAME} by speaking or calling a non-wait function when {NAME} is about to answer your question, or about to perform music, or listening to the reference audio. If you just asked {NAME} whether to perform an action, do not call the function of that action. Wait for {NAME} to answer your question first.

For each of your response, the function you call will be executed *after* your entire speech has been given to {NAME}. For immediate effects (e.g. when interrupting a Session), call the function without saying a word. Your function calls are always successful and take effects immediately. Do not call the same function twice in a row.

During each “Practice Session”, Music X Machine will go through the selected segment with {NAME}. Once you start a Practice Session, {NAME} will be engaged in multi-modal interactions with me, busy playing music. {NAME} won’t be disengaged from the interactions (e.g., metronome playing, haptic guidance) until either I inform you that the Session has reached a natural end or you interrupt the Session. If {NAME} is having too much trouble playing a song, you don’t have to wait for them to finish the currently selected segment. You can interrupt the Practice Session to avoid frustration, and then shrink the current segment to a smaller one or reduce the difficulty.

During a Practice Session, you cannot change system modes. Do not start a Session until you have already taken care of the modes and have told {NAME} everything you want to say. During a Session, call the “wait” function for muscial events. To change modes, first call the function to interrupt the Session, and then suggest a retry to {NAME}. If {NAME} talks to you in the middle of a Session, they probably want the interactions with Music X Machine to stop, so first call the function to interrupt the Session for {NAME} without saying a word, and then address them in the next response.

## References

*   [1] Achiam, J., Adler, S., Agarwal, S., Ahmad, L., Akkaya, I., Aleman, F.L., Almeida, D., Altenschmidt, J., Altman, S., Anadkat, S., et al.: Gpt-4 technical report. arXiv preprint arXiv:2303.08774 (2023). https://doi.org/10.48550/arXiv.2303.08774
*   [2] Chien, C.M., Lin, J.H., Huang, C.y., Hsu, P.c., Lee, H.y.: Investigating on incorporating pretrained and learnable speaker representations for multi-speaker multi-style text-to-speech. In: ICASSP 2021 - 2021 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP). pp. 8588–8592 (2021). https://doi.org/10.1109/ICASSP39728.2021.9413880
*   [3] Chin, D., Xia, G.: A computer-aided multimodal music learning system with curriculum: A pilot study. In: Proceedings of the International Conference on New Interfaces for Musical Expression. The University of Auckland, New Zealand (jun 2022). https://doi.org/10.21428/92fbeb44.c6910363, [https://doi.org/10.21428%2F92fbeb44.c6910363](https://doi.org/10.21428%2F92fbeb44.c6910363)
*   [4] Chin, D., Zhang, Y., Zhang, T., Zhao, J., Xia, G.: Interactive rainbow score: A visual-centered multimodal flute tutoring system. In: Michon, R., Schroeder, F. (eds.) Proceedings of the International Conference on New Interfaces for Musical Expression. pp. 208–213\. Birmingham City University, Birmingham, UK (July 2020). https://doi.org/10.5281/zenodo.4813324, [https://www.nime.org/proceedings/2020/nime2020_paper40.pdf](https://www.nime.org/proceedings/2020/nime2020_paper40.pdf)
*   [5] ddupont808: GPT-4V-Act. [https://github.com/ddupont808/GPT-4V-Act](https://github.com/ddupont808/GPT-4V-Act) (2023), accessed: Mar. 5, 2024
*   [6] Gao, D., Ji, L., Zhou, L., Lin, K.Q., Chen, J., Fan, Z., Shou, M.Z.: Assistgpt: A general multi-modal assistant that can plan, execute, inspect, and learn. arXiv preprint arXiv:2306.08640 (2023). https://doi.org/10.48550/arXiv.2306.08640
*   [7] Gao, S., Shi, Z., Zhu, M., Fang, B., Xin, X., Ren, P., Chen, Z., Ma, J.: Confucius: Iterative tool learning from introspection feedback by easy-to-difficult curriculum. arXiv preprint arXiv:2308.14034 (2023). https://doi.org/10.48550/arXiv.2308.14034
*   [8] Liang, Y., Wu, C., Song, T., Wu, W., Xia, Y., Liu, Y., Ou, Y., Lu, S., Ji, L., Mao, S., et al.: Taskmatrix. ai: Completing tasks by connecting foundation models with millions of apis. Intelligent Computing 3,  0063 (2024). https://doi.org/10.48550/arXiv.2303.16434
*   [9] Liu, Z., Lai, Z., Gao, Z., Cui, E., Li, Z., Zhu, X., Lu, L., Chen, Q., Qiao, Y., Dai, J., et al.: Controlllm: Augment language models with tools by searching on graphs. arXiv preprint arXiv:2310.17796 (2023). https://doi.org/10.48550/arXiv.2310.17796
*   [10] Microsoft: Microsoft copilot: Your everyday ai companion. [https://copilot.microsoft.com/](https://copilot.microsoft.com/) (2024), accessed: Mar. 5, 2024
*   [11] Qian, C., He, B., Zhuang, Z., Deng, J., Qin, Y., Cong, X., Lin, Y., Zhang, Z., Liu, Z., Sun, M.: Tell me more! towards implicit user intention understanding of language model driven agents. arXiv preprint arXiv:2402.09205 (2024). https://doi.org/10.48550/arXiv.2402.09205
*   [12] Qin, Y., Liang, S., Ye, Y., Zhu, K., Yan, L., Lu, Y., Lin, Y., Cong, X., Tang, X., Qian, B., et al.: Toolllm: Facilitating large language models to master 16000+ real-world apis. arXiv preprint arXiv:2307.16789 (2023). https://doi.org/10.48550/arXiv.2307.16789
*   [13] Radford, A., Kim, J.W., Xu, T., Brockman, G., McLeavey, C., Sutskever, I.: Robust speech recognition via large-scale weak supervision. In: International Conference on Machine Learning. pp. 28492–28518\. PMLR (2023)
*   [14] Ruan, J., Chen, Y., Zhang, B., Xu, Z., Bao, T., Du, G., Shi, S., Mao, H., Zeng, X., Zhao, R.: Tptu: Task planning and tool usage of large language model-based ai agents. arXiv preprint arXiv:2308.03427 (2023). https://doi.org/10.48550/arXiv.2308.03427
*   [15] Schick, T., Dwivedi-Yu, J., Dessì, R., Raileanu, R., Lomeli, M., Hambro, E., Zettlemoyer, L., Cancedda, N., Scialom, T.: Toolformer: Language models can teach themselves to use tools. Advances in Neural Information Processing Systems 36 (2024). https://doi.org/10.48550/arXiv.2302.04761
*   [16] Shen, W., Li, C., Chen, H., Yan, M., Quan, X., Chen, H., Zhang, J., Huang, F.: Small llms are weak tool learners: A multi-llm agent. arXiv preprint arXiv:2401.07324 (2024). https://doi.org/10.48550/arXiv.2401.07324
*   [17] Shen, Y., Song, K., Tan, X., Li, D., Lu, W., Zhuang, Y.: Hugginggpt: Solving ai tasks with chatgpt and its friends in hugging face. Advances in Neural Information Processing Systems 36 (2024). https://doi.org/10.48550/arXiv.2303.17580
*   [18] Suzzi, M.: Teotronico. [http://www.teotronico.it/](http://www.teotronico.it/) (2023), accessed: Mar. 4, 2024
*   [19] Tao, H., TV, S., Shlapentokh-Rothman, M., Hoiem, D., Ji, H.: Webwise: Web interface control and sequential exploration with large language models. arXiv preprint arXiv:2310.16042 (2023). https://doi.org/10.48550/arXiv.2310.16042
*   [20] VidLab: Text to speech pro. [https://rapidapi.com/ptwebsolution/api/text-to-speech-pro](https://rapidapi.com/ptwebsolution/api/text-to-speech-pro) (2023), accessed: Nov. 10, 2023
*   [21] Wang, Z., Chen, K., Jiang, J., Zhang, Y., Xu, M., Dai, S., Xia, G.: Pop909: A pop-song dataset for music arrangement generation. In: Proceedings of the 21st International Society for Music Information Retrieval Conference. pp. 38–45\. ISMIR, Montreal, Canada (2020). https://doi.org/10.5281/zenodo.4245366, [https://doi.org/10.5281/zenodo.4245366](https://doi.org/10.5281/zenodo.4245366)
*   [22] Wen, H., Li, Y., Liu, G., Zhao, S., Yu, T., Li, T.J.J., Jiang, S., Liu, Y., Zhang, Y., Liu, Y.: Empowering llm to use smartphone for intelligent task automation. arXiv preprint arXiv:2308.15272 (2023). https://doi.org/10.48550/arXiv.2308.15272
*   [23] Wen, H., Wang, H., Liu, J., Li, Y.: Droidbot-gpt: Gpt-powered ui automation for android. arXiv preprint arXiv:2304.07061 (2023). https://doi.org/10.48550/arXiv.2304.07061
*   [24] Wu, C., Yin, S., Qi, W., Wang, X., Tang, Z., Duan, N.: Visual chatgpt: Talking, drawing and editing with visual foundation models. arXiv preprint arXiv:2303.04671 (2023). https://doi.org/10.48550/arXiv.2303.04671
*   [25] Yan, A., Yang, Z., Zhu, W., Lin, K., Li, L., Wang, J., Yang, J., Zhong, Y., McAuley, J., Gao, J., et al.: Gpt-4v in wonderland: Large multimodal models for zero-shot smartphone gui navigation. arXiv preprint arXiv:2311.07562 (2023). https://doi.org/10.48550/arXiv.2311.07562
*   [26] Yang, K., Liu, J., Wu, J., Yang, C., Fung, Y.R., Li, S., Huang, Z., Cao, X., Wang, X., Wang, Y., et al.: If llm is the wizard, then code is the wand: A survey on how code empowers large language models to serve as intelligent agents. arXiv preprint arXiv:2401.00812 (2024). https://doi.org/10.48550/arXiv.2401.00812
*   [27] Yang, Z., Zeng, W., Jin, S., Qian, C., Luo, P., Liu, W.: Autommlab: Automatically generating deployable models from language instructions for computer vision tasks. arXiv preprint arXiv:2402.15351 (2024). https://doi.org/10.48550/arXiv.2402.15351
*   [28] Yang, Z., Liu, J., Han, Y., Chen, X., Huang, Z., Fu, B., Yu, G.: Appagent: Multimodal agents as smartphone users. arXiv preprint arXiv:2312.13771 (2023). https://doi.org/10.48550/arXiv.2312.13771
*   [29] Zhan, Z., Zhang, A.: You only look at screens: Multimodal chain-of-action agents. arXiv preprint arXiv:2309.11436 (2023). https://doi.org/10.48550/arXiv.2309.11436
*   [30] Zhang, Y., Li, Y., Chin, D., Xia, G.: Adaptive multimodal music learning via interactive haptic instrument. In: Queiroz, M., Sedó, A.X. (eds.) Proceedings of the International Conference on New Interfaces for Musical Expression. pp. 140–145\. UFRGS, Porto Alegre, Brazil (June 2019). https://doi.org/10.5281/zenodo.3672900, [http://www.nime.org/proceedings/2019/nime2019_paper028.pdf](http://www.nime.org/proceedings/2019/nime2019_paper028.pdf)
*   [31] Zhang, Y., Maezawa, A., Xia, G., Yamamoto, K., Dixon, S.: Loop copilot: Conducting ai ensembles for music generation and iterative editing. arXiv preprint arXiv:2310.12404 (2023). https://doi.org/10.48550/arXiv.2310.12404