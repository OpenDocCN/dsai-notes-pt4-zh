<!--yml
category: 未分类
date: 2025-01-11 12:38:54
-->

# Demo Paper: A Game Agents Battle Driven by Free-Form Text Commands Using Code-Generation LLM

> 来源：[https://arxiv.org/html/2405.11835/](https://arxiv.org/html/2405.11835/)

Ray Ito Faculty of Engineering,
The University of Tokyo Tokyo, Japan
ray51ito@g.ecc.u-tokyo.ac.jp    Junichiro Takahashi Faculty of Engineering,
The University of Tokyo Tokyo, Japan
takahashi-junichiro509@g.ecc.u-tokyo.ac.jp

###### Abstract

This paper presents a demonstration of our monster battle game, in which the game agents fight in accordance with their player’s language commands. The commands were translated into the knowledge expression called behavior branches by a code-generation large language model. This work facilitated the design of the commanding system more easily, enabling the game agent to comprehend more various and continuous commands than rule-based methods. The results of the commanding and translation process were stored in a database on an Amazon Web Services server for more comprehensive validation. This implementation would provide a sufficient evaluation of this ongoing work, and give insights to the industry that they could use this to develop their interactive game agents.

###### Index Terms:

Game AI, Character AI, LLM, Game Agent, Knowledge Expression, Human-Computer Interaction, Entertainment Computing

## I Introduction

Game players’ desire to control their trained monsters has led to the success of games such as Pokémon, Digimon, and Monster Rancher. Still, players were only able to choose from a limited set of options to command their monsters in these games. This led to attempts, including [[1](https://arxiv.org/html/2405.11835v1#bib.bib1), [2](https://arxiv.org/html/2405.11835v1#bib.bib2), [3](https://arxiv.org/html/2405.11835v1#bib.bib3), [4](https://arxiv.org/html/2405.11835v1#bib.bib4)], to make game monsters understand and react to language commands, but these implementations were rule-based, which was not easy for developers to hard-code a sufficient interaction. Therefore, [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)] proposed a method to translate player language commands without limitations into the game agent’s action. The language command was passed to the code-generation large language model (LLM) model, which generated the knowledge expression called behavior branches. This method enabled the game agent to comprehend a wider range of content and expressions of the player’s commands. The concept of using code-generation LLM was originated from [[6](https://arxiv.org/html/2405.11835v1#bib.bib6)]. While [[6](https://arxiv.org/html/2405.11835v1#bib.bib6)] intended to handle independent tasks for robotics, [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)] extended this to the game agents, allowing for the execution of more continuous and rapidly changing commands.

To examine if this method worked, [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)] only implemented a single controllable game agent against a single unmoving agent. However, the effectiveness of this approach in the context of actual player battles remained unexamined. In this paper, we demonstrate a system that enables two players to fight each other at a distance under this commanding system, and the command and translation results are stored in the database on our backend server. Therefore, we aimed to contribute to the advancement of knowledge and entertainment technology in the following ways:

*   •

    To give practical validation to this ongoing work.

*   •

    To demonstrate to the industry and researchers that they could use or extend our method to develop their interactive game systems.

## II System Overview

### II-A The Entire System

![Refer to caption](img/9d25c9ec8b9cece20fed89d1faa2b68b.png)

Figure 1: Overview of the entire system.

The system consists of the following components as shown in Fig. [1](https://arxiv.org/html/2405.11835v1#S2.F1 "Figure 1 ‣ II-A The Entire System ‣ II System Overview ‣ Demo Paper: A Game Agents Battle Driven by Free-Form Text Commands Using Code-Generation LLM"). Each description is as follows:

*   •

    Unity (2022.3.15f1): Unity computed the game environment and provided the game environment to the players.

*   •

    Photon PUN2 (2.45): PUN2 was used for the real-time network synchronization between both Unity clients.

*   •

    AWS Server: The server was used to generate the behavior branches from the player’s commands and to store the translation logs. Each players were identified using Cognito, and the logs were stored in DynamoDB. Lambda did the API control including accessing the LLM model. The latency was all together 1.8 seconds on average.

*   •

    Fireworks AI: The API of Fireworks AI was used for utilizing the default ‘llama-v2-34b-code’ model. Fireworks AI was chosen for its rapid latency. The latency was 0.9 seconds on average.

### II-B Game Environment

In the game environment, two game agents were placed in the plain 3D space. The game agents could move in the 3D space, and use the following attacks¹¹1These were inspired by Creatures Inc. “Poképark Wii: Pikachu’s Adventure.”:

*   •

    Thunderbolt: The game agent shoots a thunderbolt (implemented as a sphere) to the opponent.

*   •

    Iron Tail: The game agent swings its tail to the opponent.

*   •

    Tackle: The game agent rushes and hit the opponent.

![Refer to caption](img/7db0be900f8a675990f1d8994348fc46.png)

Figure 2: The visual interface of the game.

The interface of the game is shown in Fig. [2](https://arxiv.org/html/2405.11835v1#S2.F2 "Figure 2 ‣ II-B Game Environment ‣ II System Overview ‣ Demo Paper: A Game Agents Battle Driven by Free-Form Text Commands Using Code-Generation LLM"). The players could command their game agent by typing the command in the text box. The translation when the enter key was pressed. The game would be paused while the players were typing.

### II-C Command-Action Translation

Language commands were translated into the actionable knowledge expression called behavior branches proposed by [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)]. Behavior branches were tree structures consisting of the following nodes:

*   •

    Action node: Specified the action to be executed.

*   •

    Condition node: This connected to two nodes, and the satisfaction of this node determined which node to be executed next.

*   •

    Control node: Specified the controling of the execution flow.

This followed the concept of structural programming, and appending different behavior branches could dynamically edit the action flow. As the game agent followed these nodes, [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)] has shown that the game agent could understand and follow the commands freely written under the static experiment²²2Static experiment refers that only one game agent was playable and the other stayed still. Thus, the performance under the actual battle, which was the aim of this paper, remained unknown. For more details of the behavior branches, refer to [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)].

As similar to [[6](https://arxiv.org/html/2405.11835v1#bib.bib6)], the prompt for the LLM presented the intended format and the examples of the translations in Python. The inference code was streamed back from the LLM API³³3It started to process it as soon as the closing bracket in the generated Python code referring to the behavior branch was detected. This early stopping made the translation 1.1 seconds faster than [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)]..

### II-D Logs in the Database

![Refer to caption](img/3981859cc9d73bd513c15545d648b7de.png)

Figure 3: The screenshot of the table in DynamoDB. Note that any personal information is not shown.

Fig. [3](https://arxiv.org/html/2405.11835v1#S2.F3 "Figure 3 ‣ II-D Logs in the Database ‣ II System Overview ‣ Demo Paper: A Game Agents Battle Driven by Free-Form Text Commands Using Code-Generation LLM") shows a clip of recorded data. There was one row for each command, including the ID of the battle session, the timestamp of the command in Unix time, the player’s Cognito ID, the original command text, and the translated behavior branch in JSON format.

## III Demonstration

During the demonstration, two attendees will play the game with two laptops prepared. The demonstrator will explain the game rules and what moves the game agent able to do. The attendees will play the game by typing the command in the text box, and try to defeat the opponent’s game agent. The game will take approximately 3 minutes. In short, the required facilities for this demo include a table to put the laptops and keyboards, and the internet connection.

## IV Conclusions and Future Works

In this paper, we demonstrated a battle game system where the game characters flexibly follow the player’s directions. The method proposed by [[5](https://arxiv.org/html/2405.11835v1#bib.bib5)], translating player’s commands into Behavior Branches using code-generation LLM, has become available for actual battle gaming. For future work, we plan to conduct a quantitative and qualitative experiment to evaluate using the record and improve our suggested method so as to make it more practical and useful for the game industry.

## References

*   [1] Hobonichi Co. Ltd., “Pikachu decides that the mushroom in front of him is more important,” Hobo Niccan Itoi Shinbun, Feb. 1999\. https://www.1101.com/nintendo/nin3/nin3-2.htm (accessed Apr. 25, 2024).
*   [2] M. Yoshida, H. Bizen, M. Kambe, and Y. Kawai, “Voice Manipulation System on Virtual Space Using Synonyms,” Proc. of the Inf. Process. Soc. of Jpn. Annu. Conv., Mar. 2021, pp. 137–138.
*   [3] Q. Mehdi, X. Zeng, and N. Gough, “An interactive speech interface for virtual characters in dynamic environments,” in Proc. ISAS 04, 10th Int. Conf. on Inf. Syst. Anal. and Systhesis, Jul. 2004, pp. 243-248.
*   [4] D. M. Waqar, T. S. Gunawan, M. Kartiwi and R. Ahmad, “Real-Time Voice-Controlled Game Interaction using Convolutional Neural Networks,” in 2021 IEEE 7th Int. Conf. on Smart Instrum., Meas. and Appl., 2021, pp. 76-81
*   [5] R. Ito and J. Takahashi, “Game Agent Driven by Free-Form Text Command: Using LLM-based Code Generation and Behavior Branch (in press),” The 38th Annu. Conf. of the Jpn. Soc. for Artif. Intell., May 2024, Available: https://arxiv.org/abs/2402.07442.
*   [6] J. Liang et al., “Code as Policies: Language Model Programs for Embodied Control,” in 2023 IEEE Int. Conf. on Robot. and Automat., May 2023, pp. 9493-9500.