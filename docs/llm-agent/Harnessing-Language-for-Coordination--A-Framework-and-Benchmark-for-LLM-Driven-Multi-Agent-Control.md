<!--yml
category: 未分类
date: 2025-01-11 11:46:59
-->

# Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control

> 来源：[https://arxiv.org/html/2412.11761/](https://arxiv.org/html/2412.11761/)

Timothée Anne¹, Noah Syrkis¹, Meriem Elhosni², Florian Turati², Franck Legendre², Alain Jaquier², and Sebastian Risi¹ ¹IT University of Copenhagen, Copenhagen, Denmark²armasuisse Science+Technology, Thun, Switzerland

###### Abstract

Large Language Models (LLMs) have demonstrated remarkable performance across various tasks. A promising but largely under-explored area is their potential to facilitate human coordination with many agents. Such capabilities would be useful in domains including disaster response, urban planning, and real-time strategy scenarios. In this work, we introduce (1) a real-time strategy game benchmark designed to evaluate these abilities and (2) a novel framework we term HIVE. HIVE empowers a single human to coordinate swarms of up to 2,000 agents using natural language dialog with an LLM. We present promising results on this multi-agent benchmark, with our hybrid approach solving tasks such as coordinating agent movements, exploiting unit weaknesses, leveraging human annotations, and understanding terrain and strategic points. However, our findings also highlight critical limitations of current models, including difficulties in processing spatial visual information and challenges in formulating long-term strategic plans. This work sheds light on the potential and limitations of LLMs in human-swarm coordination, paving the way for future research in this area. The HIVE project page, which includes videos of the system in action, can be found here: [hive.syrkis.com](hive.syrkis.com).

###### Index Terms:

Multi-Agent, Strategy Games, Large Language Models, Behavior Tree

## I Introduction

The growing capabilities of Large Language Models (LLMs) have opened up new frontiers in artificial intelligence, including enabling human-AI collaboration across diverse and complex domains [[1](https://arxiv.org/html/2412.11761v1#bib.bib1), [2](https://arxiv.org/html/2412.11761v1#bib.bib2), [3](https://arxiv.org/html/2412.11761v1#bib.bib3)]. While much of the existing research focuses on LLMs’ proficiency in tasks like natural language understanding and generation, their potential for coordinating is a new area of exploration  [[4](https://arxiv.org/html/2412.11761v1#bib.bib4), [5](https://arxiv.org/html/2412.11761v1#bib.bib5)]. This ability can be particularly critical in scenarios such as disaster response, urban planning, and strategy games, where efficient coordination of multiple agents can significantly impact outcomes.

This paper introduces a novel framework called HIVE (*Hybrid Intelligence for Vast Engagements*) designed to facilitate such coordination by enabling natural language-based control of thousands of agents in real-time. Leveraging the strengths of LLMs, HIVE translates high-level human instructions into detailed operational plans for agent swarms. Specifically, after receiving a high-level strategy from the player in natural language, HIVE generates a plan using a simple domain-specific language. This plan assigns each unit a target position on the map and a behavior tree, which takes an intelligent action based on its local observations at each step. Fig. [1](https://arxiv.org/html/2412.11761v1#S1.F1 "Figure 1 ‣ I Introduction ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") illustrates an example where HIVE responds to the player’s prompt and devises a successful plan in a defense scenario.

To evaluate this approach, we present a real-time strategy game benchmark designed to test five core capabilities of multi-agent systems: coordination, exploitation of weaknesses, adherence to spatial markers, terrain utilization, and strategic planning (Sec. [IV-A](https://arxiv.org/html/2412.11761v1#S4.SS1 "IV-A The multi-agent control benchmark ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). Using six state-of-the-art LLMs ([IV-B](https://arxiv.org/html/2412.11761v1#S4.SS2 "IV-B The tested LLMs ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")), we evaluate HIVE on this benchmark (Sec. [IV-C](https://arxiv.org/html/2412.11761v1#S4.SS3 "IV-C HIVE ability tests ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). Additionally, we conduct detailed evaluations to address the following questions: How does HIVE scale to larger number of units? (Sec. [IV-D](https://arxiv.org/html/2412.11761v1#S4.SS4 "IV-D How does HIVE scale up with the number of units ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")) Does HIVE benefit from machine-human collaboration? (Sec. [IV-E](https://arxiv.org/html/2412.11761v1#S4.SS5 "IV-E Can HIVE win alone? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")) And finally, how does HIVE’s performance scale with different input modalities (e.g. visual vs. text description of the map)? (Sec. [IV-F](https://arxiv.org/html/2412.11761v1#S4.SS6 "IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control"))

Our findings demonstrate that HIVE can successfully execute complex multi-agent tasks by combining human strategic input with LLM-powered operational planning. However, the study also reveals current limitations of LLMs, including their sensitivity to input variations, challenges in visual-spatial reasoning, and difficulties with long-term strategic planning. These insights underline the dual nature of LLMs as both powerful tools for enhancing human decision-making and as systems requiring significant refinement for seamless integration into dynamic, multi-agent environments.

This work sets the stage for advancing human-AI collaboration in multi-agent coordination tasks, highlighting both the potential and the hurdles of deploying LLMs in such contexts. By addressing these challenges, future research can further refine these systems to unlock their full potential.

![Refer to caption](img/de5fd03a3fe11f1fe71c1cc3895d6d19.png)

Figure 1: Example of interactions between the player and HIVE. To win this scenario, the player and HIVE have to devise a plan to prevent the enemy units (in red) from reaching the center of the camp. To do so, the player proposes a high-level description, which HIVE uses to write down an actual plan using a Domain-specific Language (DSL). In this example, HIVE also briefly describes the situation and the plan. Notice that, even though the situation description is not always correct (10 bridges instead of 9), the actual plan is correct and wins this scenario. In this example, HIVE uses claude-3-5-sonnet-20241022 as the underlying LLM. Snapshots from the corresponding game are shown at the bottom.

## II Related Work

Recent work has demonstrated the potential of large language models (LLMs) and vision language models (VLMs) in complex strategic games. Cicero [[3](https://arxiv.org/html/2412.11761v1#bib.bib3)] showed that LLMs can achieve human-level performance in Diplomacy through natural language negotiation and tactical coordination. SwarmBrain [[6](https://arxiv.org/html/2412.11761v1#bib.bib6)], an agent for StarCraft II, combines high-level LLM-based strategic planning with low-level non-LLM-based tactical execution. These works demonstrate that LLMs can reason about and execute strategic decisions in game environments.

However, controlling multiple units in real-time strategy (RTS) games presents unique challenges. A proposed approach is hierarchical command and control architecture, tested on StarCraft, combining high-level and low-level reinforcement learning agents, attempting to balance micro and macro management (i.e., we want an overall strategy and quick low-level decision) [[7](https://arxiv.org/html/2412.11761v1#bib.bib7)]. Recent work has also used behavior trees for low-level actions along with LLMs for high-level control [[8](https://arxiv.org/html/2412.11761v1#bib.bib8)].

Furthering the ability to conduct AI research in the RTS domain, the StarCraft Multi-Agent Challenge (SMAC) environment [[9](https://arxiv.org/html/2412.11761v1#bib.bib9)] established key benchmarks for multi-agent control [[10](https://arxiv.org/html/2412.11761v1#bib.bib10), [11](https://arxiv.org/html/2412.11761v1#bib.bib11)]. SMACv2 [[12](https://arxiv.org/html/2412.11761v1#bib.bib12)] extends this work, introducing procedurally generated scenarios requiring generalizations. Recently, JaxMARL’s SMAX [[13](https://arxiv.org/html/2412.11761v1#bib.bib13)] has enabled easy parallelization in an SMAC-like environment, lowering the barrier of entry for research into AI applied in an RTS setting. However, while these frameworks provide foundations for developing multi-agent control systems, they focus on tactical execution rather than strategic planning, leaving the integration with higher-level decision making under-explored.

The development of LLM-based multi-agent systems has seen significant recent research. A comprehensive survey discussing key aspects of agent profiling, communication, and environment interaction has also been written [[14](https://arxiv.org/html/2412.11761v1#bib.bib14)].

AgentCoord [[15](https://arxiv.org/html/2412.11761v1#bib.bib15)] is a visual interface designed for coordination strategies in multi-agent collaboration to solve a joint goal. Remaining challenges in multi-agent LLM systems include effective task allocation, robust reasoning, and efficient memory management [[16](https://arxiv.org/html/2412.11761v1#bib.bib16)]. A key limitation across these systems is the computational overhead of LLM inference, which makes real-time applications challenging without significant architectural compromises (partly motivating hybrid systems like SwarmBrain in which low-level execution is done without the LLM).

Recent research has explored how LLMs handle strategic reasoning tasks. Zhang et al. [[5](https://arxiv.org/html/2412.11761v1#bib.bib5)] surveyed the strategic reasoning capabilities of LLMs, highlighting their ability to understand game structures and adapt to different contexts. Kramár et al. [[17](https://arxiv.org/html/2412.11761v1#bib.bib17)] investigated how artificial agents can use communication for better cooperation in strategic games (Diplomacy). Lorè and Heydari [[4](https://arxiv.org/html/2412.11761v1#bib.bib4)] analyzed how different LLMs are affected by contextual framing (i.e. the superficial story we tell about the game) and game structure.

However, in sufficiently complex and dynamic environments, LLMs and VLMs struggle to perform well [[18](https://arxiv.org/html/2412.11761v1#bib.bib18)], sometimes even performing *worse* when visual information, like an image of the game map, is included. LLMs struggle with physical common-sense reasoning in 3D environments, performing worse than human children on basic spatial reasoning tasks [[19](https://arxiv.org/html/2412.11761v1#bib.bib19)]. Further, long-range reasoning problems are exasperated as these models are prone to hallucinations [[20](https://arxiv.org/html/2412.11761v1#bib.bib20)]. Though algorithms has been proposed to fortify against hallucination in multi-step reasoning by dividing up reasoning steps, the problem of long-range causal reasoning remains unsolved [[20](https://arxiv.org/html/2412.11761v1#bib.bib20)]. This is particularly problematic in strategy games, where decisions must build on each other, and errors compound over time. AndroidArene[[21](https://arxiv.org/html/2412.11761v1#bib.bib21)] is a benchmark, and environment that evaluates an LLM’s ability to navigate an operating system (e.g. using a smartphone with apps), showing this to be a current limitation of LLMs.

Additionally, models struggle to generalize beyond their training data when moving towards geospatial modalities, sparking the need for dedicated Vision-Language Geo-Foundation models [[22](https://arxiv.org/html/2412.11761v1#bib.bib22)]. This limitation is particularly relevant for strategy games involving spatial reasoning and map awareness, where current models often fail to develop coherent long-term strategies based on terrain and unit positioning (and sometimes struggle to identify basic landmarks). This echoes recent research that shows VLM often struggle with basic tasks (e.g., do two circles intersect?) that are trivial for humans, and a necessity for spatial reasoning [[23](https://arxiv.org/html/2412.11761v1#bib.bib23)].

Integrating LLMs into real-time strategy games faces several challenges not fully addressed by current research. This suggests that while LLMs can engage in high-level strategic planning, they need significant support systems to translate these plans into effective tactical actions.

## III The HIVE approach

![Refer to caption](img/14df26cf11f6e791bbc8aebfcc4e1a20.png)

Figure 2: Overview of the HIVE approach. HIVE enables users to command up to several thousand units by delegating the tedious task of assigning behaviors and objectives to each unit to a general-purpose Large Language Model (LLM). In its current implementation, HIVE operates in two main phases: (1) a discussion phase with an LLM to develop a plan before the game starts, and (2) the execution phase, where the plan assigns a behavior tree to each unit. We implemented the modules that handle the units in JAX [[24](https://arxiv.org/html/2412.11761v1#bib.bib24)], and those that manage high-level information in Python. See main text for a more detailed description.

Our approach, HIVE, allows a player to control thousands of units in a strategy game through a natural language LLM-based dialog. We hypothesis that this type of human-machine collaboration should be able to alleviate some of the aforementioned shortcomings of purely LLM-based approaches.

Fig. [2](https://arxiv.org/html/2412.11761v1#S3.F2 "Figure 2 ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") presents an overview of its main components. (a) The player can communicate in natural language and add markers to the map to give absolute positions (Sec. [III-C](https://arxiv.org/html/2412.11761v1#S3.SS3 "III-C The interface between the player and HIVE ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). (b) The LLM writes a plan using a domain-specific language that controls the behavior of all units (Sec. [III-D1](https://arxiv.org/html/2412.11761v1#S3.SS4.SSS1 "III-D1 Writing a plan with our Domain Specific Language ‣ III-D The plan ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). (c) The plan assigns a target position and behavior tree to each unit when created or triggered by objective checking (Sec. [III-D2](https://arxiv.org/html/2412.11761v1#S3.SS4.SSS2 "III-D2 Applying the plan ‣ III-D The plan ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). (d) Each unit evaluates its behavior tree using local observation to determine an action (Sec. [3](https://arxiv.org/html/2412.11761v1#S3.F3 "Figure 3 ‣ III-B Behavior Trees ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). Actions in our game are either attacking an enemy unit or moving. (e) After each step, the game checks the current plan’s objectives (Sec. [III-D3](https://arxiv.org/html/2412.11761v1#S3.SS4.SSS3 "III-D3 Checking the plan ‣ III-D The plan ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). If some are achieved, the plan is rolled forward, and the active steps are updated. This module also checks for global winning and losing conditions specific to each scenario. (f) At the start of the discussion, the LLM receives information about the types of each unit present (Sec. [III-E](https://arxiv.org/html/2412.11761v1#S3.SS5 "III-E LLM Game Information ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). When triggered by the player, the LLM also receives a precomputed textual description of the map (see Sec. [IV-F](https://arxiv.org/html/2412.11761v1#S4.SS6 "IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") for a discussion of the geo-visual abilities of LLMs) and the current health and position of each unit. (g) After each game step, the units’ positions are plotted on the map and displayed on the player’s screen (Sec. [III-A6](https://arxiv.org/html/2412.11761v1#S3.SS1.SSS6 "III-A6 The visualization ‣ III-A The multi-agent game ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")).

Our implementation allows the player to interact with HIVE during plan execution, potentially enabling updates as the plan unfolds. However, for this initial evaluation of the framework, we chose not to utilize this feature to allow for a simpler analysis. Another provided functionality is to restart the same scenario after a game to try again and improve the current score. In such cases, HIVE retains the discussion history but does not include information about the previous game, primarily due to the limited context size of the model. We will now go over the different HIVE components in more detail.

### III-A The multi-agent game

The game is a top-down strategy game where the player commands an army of several hundred to thousands of units against an opposing army of comparable size. Players control their army exclusively by interacting with HIVE, which develops a plan that determines each unit’s behavior at every step. The game also allows us to create various scenarios by using different maps and setting global objectives.

#### III-A1 Global objectives

There are two kinds of objectives. One is the *elimination objective*, where the goal is to eliminate a set of enemy units (in this paper, the global elimination objectives require the elimination of all enemy units, but HIVE can choose to target a specific set when writing a plan). The other one is the *position objective*, where the goal is to reach a given position with at least one or all units (in this paper, the position objectives for the scenarios require reaching a given location with at least one unit. In contrast, position objectives in a plan require the whole group of units to reach it). These objective specifications allow for creating scenarios with more offensive, defensive, or stealth components. Once either side has achieved its global objective, the game ends.

#### III-A2 The units

The framework allows us to create any type of unit parametrized by its move speed, max health, attack damage, and attack range. In this paper, we used three types of units. The *spearman* a slow close range unit with high health. The *archer* a long range unit with low health. The *cavalry* a fast close range unit with medium health.

Those three types of units allow for rock-paper-scissor dynamics, as a group of archers easily eliminate spearmen thanks to their longer range, a group of cavalry easily eliminate a group of archers by coming in contact faster than they get killed, and a group of spearmen easily wins against a group of cavalry because of similar damage but higher health. Tab. [II](https://arxiv.org/html/2412.11761v1#A0.T2 "TABLE II ‣ -E Units characteristics ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") in the appendix details the characteristics of those units.

#### III-A3 The observation

Each unit knows the position and health of every unit inside its sight range (set to 15 m) and also has access to a distance map, i.e. a matrix containing the distance to the unit’s target position from anywhere on the map. This enables it to follow the shortest path to its target position.

#### III-A4 The actions

At each step, each unit takes one action: do nothing, move to any position in sight reachable given the unit’s speed (2D continuous action), or attack an enemy unit in sight (i.e. in sight range with no visual obstacle in between) that is in attack range (discrete action).

#### III-A5 The terrain

The game allows one to easily implement different types of terrain. In this paper, we use four types:

*   •

    Normal: the units can move and see through it;

*   •

    Forest: the units can move but not see through it;

*   •

    Water: the units can see over it but not move through it;

*   •

    Building: the units cannot move or see through it.

Those allow the creation of diverse maps and scenarios with strategic features such as choke points over bridges (normal terrain over water) or stealth opportunities by hiding in forests.

#### III-A6 The visualization

At each time step, the game plots the units’ positions over the map using different shades of colors (blues for the player’s units and reds for the enemy units) and shapes (square for the spearmen, circle for the archers, and triangle for the cavalry). The player does not receive information about each unit’s health (i.e. it would be difficult to assess the health of hundreds of units in real-time [[25](https://arxiv.org/html/2412.11761v1#bib.bib25)]).

#### III-A7 The main loop

The main loop of the game is implemented in JAX[[24](https://arxiv.org/html/2412.11761v1#bib.bib24)] and proceeds as follows:

1.  1.

    Apply the attack action of each attacking unit;

2.  2.

    Apply the move action of each moving unit while checking for collision with building or water terrain;

3.  3.

    Push units in collision with each other;

4.  4.

    Check for collision with a building or water terrain;

5.  5.

    Compute the distance between each unit, considering the line of sight with building and forest terrain.

### III-B Behavior Trees

![Refer to caption](img/8b12b283d663bf58e9d19ab8e1f5f881.png)

Figure 3: The three main behavior trees available to HIVE to control each unit.

The framework includes an important middle layer between the high-level instruction of the player written down as a plan and the low-level actions taken by the units. We choose behavior trees [[26](https://arxiv.org/html/2412.11761v1#bib.bib26)] for this purpose. They offer a convenient way to control agents while being easy to design and understand. They are composed of four different types of nodes:

*   •

    Sequence node S: stops at the first failed node;

*   •

    Fallback node F: stops at the first successful node;

*   •

    Condition node C: evaluates a condition given the unit observation and returns failure or success;

*   •

    Action node A: tries to perform an action given the unit observation and returns failure or success.

HIVE evaluates each active behavior tree sequentially, as they correspond to different JAX instructions. Still, each behavior tree evaluation is done in parallel over its assigned units. This evaluation slows down as more behavior trees become active and their size (i.e. number of nodes) increases.

For this paper, we designed a list of condition and action nodes parametrized by qualifiers (see Sec. [-C](https://arxiv.org/html/2412.11761v1#A0.SS3 "-C Grammar for our Behavior trees ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") in the appendix for the full Lark grammar). The available conditions are:

*   •

    Is there an ally or enemy of a given type in sight?

*   •

    Is there an ally or enemy of a given type in sight that is or could be in attack range in one, two or three steps?

*   •

    Am I or could I be in attack range in one, two, or three steps of an ally or enemy of a given type in sight?

*   •

    Am I of a given type?

*   •

    Do I have health below a 75%, 50%, or 25% threshold?

*   •

    Am I in a forest?

The available actions are:

*   •

    Do nothing;

*   •

    Move toward the target position (following the shortest path with some noise) up to a given threshold relative to the unit sight range and move speed;

*   •

    Move toward the closest or farthest or weakest or strongest or random ally or enemy unit in sight of a given type (without optimal path guaranty);

*   •

    Attack the (closest or farthest or weakest or strongest or random) enemy in attack range.

We handcrafted five behavior trees available to HIVE:

*   •

    Attack in long range: move away from nearby enemies, or attack an enemy if able, or move toward target position;

*   •

    Attack in close range: attack a random unit in attack range if able or move toward the closest enemy if there is any and otherwise move toward the target position;

*   •

    Attack and move: attack a random unit in attack range if able or move toward the target position, and only if the target is reached, move toward the closest enemy;

*   •

    Move toward target position: follow the shortest path to the target position with some noise;

*   •

    Stand: do nothing.

In all behavior trees concerning enemy units, the LLM can target any type of unit or subset. Fig. [3](https://arxiv.org/html/2412.11761v1#S3.F3 "Figure 3 ‣ III-B Behavior Trees ‣ III The HIVE approach ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") shows the three main behavior trees used. The exact syntax of all the behavior trees can be found in the appendix Sec. [-D1](https://arxiv.org/html/2412.11761v1#A0.SS4.SSS1 "-D1 Available for HIVE ‣ -D List of behavior trees used in this paper ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control").

### III-C The interface between the player and HIVE

The player can only control their army through an interface with HIVE. Its front end is composed of an LLM that receives natural language prompts from the player. HIVE can reply to the player prompts by answering questions or asking some.

Controlling many units to go to different positions on a map can be conveniently and quickly done through natural language commands. For example, with large unit numbers, asking the system to send units to cover all bridges can be faster than using a traditional mouse interface. However, sometimes it can still be useful to communicate a specific absolute map position to the model. This is why HIVE includes the ability for the player to click on the map to add markers labeled with a letter that can then be referenced in the dialog with the model.

The core component of HIVE is an LLM. The tasks’ specifications are instructed via its context, allowing us to easily test different LLMs for their ability to perform in our multi-agent benchmark. The LLM inference is the most time-consuming part of our system, as the model may take between a few seconds and several dozen to respond (see Fig. [7](https://arxiv.org/html/2412.11761v1#S4.F7 "Figure 7 ‣ IV-C HIVE ability tests ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). In this paper, the game starts only when the LLM answer is received, and the plan is parsed and applied.

### III-D The plan

#### III-D1 Writing a plan with our Domain Specific Language

HIVE uses a domain-specific language to write a plan that follows the player’s high-level commands. A plan is composed of several steps. A step is composed of several components:

*   •

    A numerical label to refer to it;

*   •

    A list of prerequisite steps that need to be achieved before this step (e.g., waiting for two groups to reach their assigned locations before ordering them to attack);

*   •

    An objective (similar to the global objectives) that is used to infer if the step is achieved, active, or inactive;

*   •

    A list of groups of units that dictates the behaviors of each unit concerned by the step.

A group of units is composed of a list of unit IDs, a target position, and a behavior tree. The step is valid only if each unit is included in no more than one group.

We instruct the LLM (using a System prompt) on the rules to write a plan with a simple example and a short list of mistakes to avoid. The actual instruction message used for Fig. [1](https://arxiv.org/html/2412.11761v1#S1.F1 "Figure 1 ‣ I Introduction ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") is detailed in the appendix Sec. [-A1](https://arxiv.org/html/2412.11761v1#A0.SS1.SSS1 "-A1 Instruction ‣ -A LLM instruction and game for the example of the teaser figure ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control"). The plan written by the LLM is parsed and an error is raised if it is not grammatically correct or if the units’ or behavior trees’ IDs are incorrect.

#### III-D2 Applying the plan

When the plan is created or when the active steps change, HIVE passes through each active step and assigns a behavior tree and target position to each unit. If a unit is present in several active steps, the last one will be assigned. If it does not have a new assignment, it keeps its previous assignment. If a unit never receives an assignment, its default behavior is to do nothing. This part of the system can also be time-consuming since (1) each target position requires computing a breadth-first search on the full map, and (2) the reassignment of the behavior trees and the steps’ objectives checking requires a new compilation of the JAX functions.

#### III-D3 Checking the plan

After each step in the game, HIVE checks the global and active step objectives to see if one side is victorious, if the plan is fully unrolled, or if new steps become active. In this paper, we stopped the game if the plan was fully unrolled, but no side was victorious, as we suppose that this was a flaw in the plan’s design. HIVE could also ask the player to present new instructions.

### III-E LLM Game Information

The LLM receives both information about each unit and information about the map, which we detail below.

#### III-E1 Unit information

The LLM gets a complete description of the unit types and the current compositions of both sides. After each player prompt, it receives the units’ health and positions. An example is in the appendix Sec. [-A2](https://arxiv.org/html/2412.11761v1#A0.SS1.SSS2 "-A2 The game info ‣ -A LLM instruction and game for the example of the teaser figure ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control").

#### III-E2 Textual description of the map

During preliminary experiences, we tried using visuals but found that current all-purpose LLMs are inefficient in understanding a top-down map and can not give accurate positioning of the units, terrain, or landmarks. Thus, we rely on a precomputed description of the map that gives the absolute position of the different terrain types (forest and rivers) and the bridges’ positions. Sec. [IV-F](https://arxiv.org/html/2412.11761v1#S4.SS6 "IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") compares HIVE’s abilities with the textual prompts and with different variations of inputting an image of the map. Fig. [11](https://arxiv.org/html/2412.11761v1#S4.F11 "Figure 11 ‣ IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") and [12](https://arxiv.org/html/2412.11761v1#S4.F12 "Figure 12 ‣ IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") show two examples of such descriptions.

## IV Experiments

![Refer to caption](img/0e9b52f774f40b9410dde21521b79b74.png)

Figure 4: The HIVE Benchmark Ability Tests. (a) Coordinate where the player has to eliminate all the enemies using 1000 units, (b) Exploit weakness where the player has to efficiently use the three types of units to eliminate the enemies, (c) Follow markers where the player has to bring at least one unit to the south, (d) Exploit terrain where the player has to bring at least one unit to the opposite corner of the map, and (e) Strategize points where the player has to prevent the enemies from reaching the center of their camp.

![Refer to caption](img/9b9bcbfb46a828d653554f47d541c762.png)

Figure 5: Successful plan by HIVE using different LLMs for each ability test. HIVE can translate high-level commands into successful plans, from coordinating thousands of units to exploiting the terrain, enemy weaknesses, or strategic points. Blue units are the player’s units. Red units are the enemies.

![Refer to caption](img/a3e3cbd8a46707ba7d18819b8cd8fbd0.png)

Figure 6: Ability evaluations of HIVE using six different LLMs with the same 10 prompts for each ability test. Apart from Follow Markers, the different ability tests are solvable but challenging for all LLMs. Sonnet seems to be the best LLM to use with HIVE, while a small LLM like Llama3-8B completely fails.

### IV-A The multi-agent control benchmark

We test HIVE’s effectiveness in following the player’s command using five ability tests on four scenarios (Fig. [4](https://arxiv.org/html/2412.11761v1#S4.F4 "Figure 4 ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")). In all scenarios, the enemy units’ behavior is identical: the spearmen and cavalry units attack in close combat, and the archers attack in long range, moving away from close combat. The maps were designed with the INKARNATE map-making platform ([https://inkarnate.com/](https://inkarnate.com/)).

#### IV-A1 Coordinate

The player controls 1000 units (500 spearmen and 500 archers) and must eliminate the opposite army of 1000 spearmen from the north. This tests the ability to coordinate many units with two types on a simple terrain. With the map being 150 m wide and each unit only able to see 15 m ahead, the challenge is to spread the units on the battlefield. The other challenge is efficiently placing the archers behind the spearmen to minimize casualties.

#### IV-A2 Exploit weakness

The player controls 750 units (250 spearmen, 250 archers, and 250 cavalry) and must eliminate the opposing army of equal composition. The map is divided into four quadrants by vertical and horizontal rivers crossed by bridges. The opposite army is divided by types on three quadrants, the player’s army starting in the last one. This tests the ability to exploit the rock-paper-scissor dynamics to win while minimizing casualties. This demands that HIVE estimate the opposite army battalions’ positions and unit type, and send the unit type strong against it. HIVE is given the strengths and weaknesses of each unit type as in-context information.

#### IV-A3 Follow markers & Exploit terrain

The player controls 300 spearmen starting on the northeast corner and must bring at least one to an objective position in the southwest corner. The units must cross a bridge guarded by 1200 opposite units (600 spearmen and 600 archers). The challenge is to efficiently use the cover of the trees (the units cannot fight in the forest) to minimize casualties. We use this scenario for two ability tests: Follow markers, which tests the ability to follow absolute positions given by the player as markers, and Exploit terrain, which tests the ability to exploit the terrain by following the player’s textual description of the path to follow.

#### IV-A4 Strategize points

The player controls 700 units (350 spearmen and 350 archers) and must prevent the opposite army of 900 spearmen from reaching the center of the camp (the fire camp in the center of the island). The opposite units are divided in the northeast and southwest corners and are designed to split and path through each bridge. This tests the ability to split the army on strategic points correctly.

### IV-B The tested LLMs

HIVE is agnostic to which LLM is being used (apart from the change in API to query the model). We evaluate HIVE using six different models, to determine which one works best together with a human user when tested on our ability tests. We chose state-of-the-art closed-source models from GPT-4o and Claude families and one small open model as a baseline:

*   •

    4o: gpt-4o-2024-11-20 [[27](https://arxiv.org/html/2412.11761v1#bib.bib27)];

*   •

    4o-mini:gpt-4o-mini-2024-07-18 [[27](https://arxiv.org/html/2412.11761v1#bib.bib27)];

*   •

    oi-mini: o1-mini-2024-09-12 [[27](https://arxiv.org/html/2412.11761v1#bib.bib27)]:

*   •

    Sonnet: claude-3-5-sonnet-20241022 [[28](https://arxiv.org/html/2412.11761v1#bib.bib28)];

*   •

    Haiku: claude-3-5-haiku-20241022 [[28](https://arxiv.org/html/2412.11761v1#bib.bib28)];

*   •

    Llama3 (8B): Llama3 (8B) [[29](https://arxiv.org/html/2412.11761v1#bib.bib29)].

Apart from oi-mini, which is a preview with a fixed temperature of 1, all models are used with a temperature of 0.

### IV-C HIVE ability tests

While experimenting, we found that the LLMs’ proposed plans were susceptible to just one word. To evaluate them, we decided to generate 10 slightly different prompts for each of the five abilities. For example, to test the coordination ability, we query HIVE with *“Make a plan that forms as many squads as you think necessary to cover the middle row of the map to eliminate all the enemies as quickly as possible. Ensure to place our archers so that they are protected by our close range units.”*. The slight variations are the orders of saying “Design a plan,” “Write down a plan,” or “Make a plan,” using “North/South/East/West” or “Top/Down/Left/Right,” or saying “archers units” or “long-distance units.” All the prompts are detailed in the appendix Sec. [-F](https://arxiv.org/html/2412.11761v1#A0.SS6 "-F Prompts used for the 5 ability tests ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control"). HIVE waits for the LLM’s answer, parses a plan from it, and, if it is valid, applies it to the game until one side achieves its objective or a timeout is reached. Otherwise, the attempt is flagged as invalid.

Fig. [5](https://arxiv.org/html/2412.11761v1#S4.F5 "Figure 5 ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") shows snapshots of a successful plan execution made by HIVE using different LLMs with the corresponding prompt from the player. The actual plans are detailed in the appendix Sec [-B](https://arxiv.org/html/2412.11761v1#A0.SS2 "-B LLM answer and plan to Fig. 5 examples ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control"). Tab. [I](https://arxiv.org/html/2412.11761v1#S4.T1 "TABLE I ‣ IV-C HIVE ability tests ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") presents the strict successes of HIVE for each modality. Fig. [6](https://arxiv.org/html/2412.11761v1#S4.F6 "Figure 6 ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") presents a continuous evaluation of the performance of the plans made by HIVE. Using the percentage of enemies eliminated for the abilities (a), (b), and (e), and the distance to objective position for (c) and (d). Fig. [8](https://arxiv.org/html/2412.11761v1#S4.F8 "Figure 8 ‣ IV-C HIVE ability tests ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") shows the result of the execution in terms of winning, losing, timeout, and achieving the plan objectives without achieving the winning objective; the plan being invalid and HIVE returning no plan. A video showing all the plans being executed is available here: [hive.syrkis.com](hive.syrkis.com).

Hive solves all ability tests, highlighting that it effectively possesses all the proposed abilities. It can effectively split the units on a battlefield, putting the long range units in the rear to increase efficiency. Additionally, it can recognize clusters of enemy units with specific weaknesses and exploit them. Furthermore, HIVE can follow a strategic plan to exploit the cover of the trees as much as possible. Finally, it can also split the army to cover all the strategic points asked by the player.

The main takeaways from this evaluation are the following:

*   •

    Llama3-8B is too small to handle the plan writing using only context instructions;

*   •

    Follow markers is the easiest ability test, and both 4o and Sonnet solve it nearly perfectly (nine times out of ten);

*   •

    The other ability tests propose interesting challenges for LLMs as none of them got more than 50% of success;

*   •

    Sonnet is the only LLM to succeed in all 5 ability tests and only present one invalid plan out of 50 answers;

*   •

    For the continuous evaluations, Sonnet is also either first (in Coordinate, Exploit Terrain, Strategize points) or tied-first (in Exploit weakness and Follow markers);

*   •

    Both Haiku and 4o-mini show on average worse performance than their full-version counterparts;

*   •

    Even state-of-the-art LLMs are extremely sensitive to the wording of the prompts as slight changes in the prompt result in drastic changes in the plan and the execution.

![Refer to caption](img/41e51567ee30f8ba36e61c809b73e628.png)

Figure 7: Wall-times of the 60 LLM queries of the main evaluations. Llama3-8B is run on a M3 chip. HIVE queries the other via their API.

TABLE I: Ability tests of HIVE with different LLMs showing the success to achieve the global objective of each scenario.

|  | 4o |  

&#124; 4o &#124;
&#124; mini &#124;

 |  

&#124; o1 &#124;
&#124; mini &#124;

 | Sonnet | Haiku |  

&#124; Llama &#124;
&#124; 3 (8B) &#124;

 |
| --- | --- | --- | --- | --- | --- | --- |
| Coordinate | 0/10 | 0/10 | 0/10 | 2/10 | 0/10 | 0/10 |
| Exploit weakness | 0/10 | 0/10 | 0/10 | 1/10 | 0/10 | 0/10 |
| Follow markers | 9/10 | 3/10 | 5/10 | 9/10 | 7/10 | 0/10 |
| Exploit terrain | 3/10 | 0/10 | 0/10 | 4/10 | 2/10 | 0/10 |
| Strategize points | 0/10 | 0/10 | 4/10 | 3/10 | 0/10 | 0/10 |
| Total | 12/50 | 3/50 | 9/50 | 19/50 | 9/50 | 0/50 |

![Refer to caption](img/9f40c79dda96208aea02d44e25ae9559.png)

Figure 8: The summarized result of the ability tests of HIVE using each LLM.

### IV-D How does HIVE scale up with the number of units

![Refer to caption](img/d197082507dea2301d5a1906835a69f0.png)

Figure 9: Unit Scaling. Comparison of the results of the HIVE plan for the Coordinate test by varying the number of units in the game.

To see how HIVE scales with the number of units, we performed another evaluation on the Coordinate test, varying the number of units from 200 to 4000\. We stopped at 4000 due to the current hardware limitation of the machine running the experiment. As the game’s main loop needs to be compiled, the size of the distance matrix between each unit grows quadratically with the number of units. For this study, we only tested on 4o and Sonnet, which showed the best performances.

Fig. [9](https://arxiv.org/html/2412.11761v1#S4.F9 "Figure 9 ‣ IV-D How does HIVE scale up with the number of units ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") shows the result of the plan made by HIVE. There is no significant conclusion regarding the success rate as there is too much noise due to the inherent variance of the LLMs. Still, 4o’s number of invalid and empty plans increases as the number of agents increases. This is not observed for Sonnet, again highlighting its superior performance.

One limitation with the scaling for the current implementation is that HIVE sends the position and health of each unit to the LLM, in the context window, and the context size is limited (128,000 tokens for 4o and 200,000 for Sonnet). To get an idea, for 4o, the context went from 4,271 tokens with 200 units to 38,475 with 4,000 units (with an affine scaling), i.e. approximately 9 tokens per unit, which gives a hard limit of $128,000/9=14,222$ units for 4o. Similarly, for Sonnet, it gives a hard limit of $200,000/9=22,222$ units.

### IV-E Can HIVE win alone?

An interesting question is how well HIVE performs without a human’s help. To answer this question, we use ten variations of the prompt: ”First, analyze the situation to find a good strategy to win this mission, then write down the corresponding plan.” (the variations are detailed in the appendix Sec. [-G](https://arxiv.org/html/2412.11761v1#A0.SS7 "-G Prompts used for HIVE alone ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control")) on each of the four scenarios for 4o and Sonnet as they were the best LLMs. HIVE always gives the scenario description so that the LLM knows the global objective.

Fig. [10](https://arxiv.org/html/2412.11761v1#S4.F10 "Figure 10 ‣ IV-E Can HIVE win alone? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") shows the comparison between HIVE with and without the player’s help. For Coordinate and Exploit terrain both models no longer wins without the human help. For Strategize Points Sonnet conserves its three wins but both models have lower median performances. This shows that HIVE is better with human help, confirming the benefits of a hybrid intelligence approach. Additionally, when comparing the median performance, both Coordinate and Exploit Weakness seem to be easier ability tests for HIVE alone as they only require sending units toward the enemies. In comparison, Exploit terrain and Strategic points require exploiting the map features to win and depend on more long-term planning.

![Refer to caption](img/13a6b79cb79e2ddaa51ffc5af75c0822.png)

Figure 10: Human-Machine Collaboration. Without the player’s help, HIVE’s performances and win rates decrease for both models in all ability tests.

### IV-F Does HIVE need the textual description of the map?

![Refer to caption](img/6bcf2b13ccf7081aca004aab6a7fd9da.png)

Figure 11: Inputs describing the map for the Exploit terrain ability test.

![Refer to caption](img/cd8adbedd8edbaf1696285cabf78d678.png)

Figure 12: Inputs describing the map for the Strategize points ability test.

![Refer to caption](img/c7ef7cfa0cced7877384662facdd68aa.png)

Figure 13: Maps description. LLMs’ performances to use maps drop when switching from textual descriptions to images (raw, with a grid or scaffolding).

We evaluate HIVE’s vision abilities by replacing the textual description of the map with the image. As it has to come up with precise target positions, we also compared two different ways to improve the image-to-coordinate: using a grid with x and y axis and using a scaffolding [[30](https://arxiv.org/html/2412.11761v1#bib.bib30)]. We only tested the vision on two ability tests that required it: Exploit terrain and Strategize points. Fig. [11](https://arxiv.org/html/2412.11761v1#S4.F11 "Figure 11 ‣ IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") and [12](https://arxiv.org/html/2412.11761v1#S4.F12 "Figure 12 ‣ IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") show the four kinds of inputs HIVE uses as the description of the map.

Fig. [13](https://arxiv.org/html/2412.11761v1#S4.F13 "Figure 13 ‣ IV-F Does HIVE need the textual description of the map? ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") shows the continuous measure of performance. Without the textual description, HIVE’s performance decreases. For Exploit terrain, 4o seems to be able to use the raw image while Sonnet performance completely drops. For Strategize points, Sonnet won once with the raw image and twice with the scaffolding, compared to 3 times with the textual description, but the lower median performances.

## V Discussion

HIVE using Llama3-(8B) has very poor results, mainly due to sending invalid plans. This may be solved by fine-tuning it to our specific DSL. HIVE would need to rely on big generalist models that one can only call via an API, increasing the response time. Those models know many things that are irrelevant to our task. By taking a smaller model and fine-tuning it, HIVE could run locally and, using dedicated hardware, could even become fast enough to run continuously alongside the game to give real-time feedback to the player or adjust the plan as the game unrolls.

Fine-tuning could also improve LLMs’ poor ability to read the map. Another solution would be to use a specialized module to looks at a map and returns a textual description of its interesting features. For example, the module could display the location of different terrains and landmarks that the player can intuitively use to communicate placements.

Early experiments showed that generalist LLMs were not good at writing their behavior trees with our out-of-distribution DSL grammar. Fine-tuning an LLM for a specific grammar and repertoire of condition and action nodes could drastically extend the diversity of behaviors available to the player.

Nonetheless, in addition to the cost of fine-tuning an LLM, it also reduces the applicability of HIVE. These new challenges were never present in any training dataset of the models. Nevertheless, HIVE using Sonnet won more than the total loss (19 against 14 over 50 games). A generalist model could allow HIVE to perform satisfyingly on other massively multi-agent games, such as worker and resource management games. The main change would be the nodes in the behavior trees.

## VI Conclusion

In this paper, we present a new challenge for LLMs as human assistants to control up to two thousand units in a strategy game. We propose a new framework, HIVE, to allow a player to give high-level commands that an LLM translates into a long-term plan that controls the behavior of each unit. We showed that generalist LLMs such as Claude Sonnet and GPT-4o can handle such tasks but are still sensitive to slight changes in the player’s prompts. Complimentary experiments showed that HIVE requires human help to get the best performance and that generalist LLMs’ visual capacity to use an out-of-distribution map for terrain and landmark locations is still to be improved. This work opens many interesting avenues for improving LLMs’ capacities to collaborate with humans, such as improving their map-reading abilities, reducing their sensitivity to prompts, and increasing their long-term planning.

## Acknowledgments

Funding was provided by the armasuisse project F00-007.

## References

*   [1] A. Sharma, S. Rao, C. Brockett, A. Malhotra, N. Jojic, and B. Dolan, “Investigating Agency of LLMs in Human-AI Collaboration Tasks,” in *Proceedings of the 18th Conference of the European Chapter of the Association for Computational Linguistics (Volume 1: Long Papers)*, Y. Graham and M. Purver, Eds.   St. Julian’s, Malta: Association for Computational Linguistics, Mar. 2024, pp. 1968–1987.
*   [2] A. Bakhtin, D. J. Wu, A. Lerer, J. Gray, A. P. Jacob, G. Farina, A. H. Miller, and N. Brown, “Mastering the Game of No-Press Diplomacy via Human-Regularized Reinforcement Learning and Planning,” Oct. 2022.
*   [3] E. Dinan, G. Farina, C. Flaherty, D. Fried, A. Goff, J. Gray, H. Hu, A. P. Jacob, M. Komeili, K. Konath, M. Kwon, A. Lerer, M. Lewis, A. H. Miller, S. Mitts, A. Renduchintala, S. Roller, D. Rowe, W. Shi, J. Spisak, A. Wei, D. Wu, H. Zhang, and M. Zijlstra, “Human-level play in the game of *Diplomacy* by combining language models with strategic reasoning,” *Science*, vol. 378, no. 6624, pp. 1067–1074, Dec. 2022.
*   [4] N. Lorè and B. Heydari, “Strategic behavior of large language models and the role of game structure versus contextual framing,” *Scientific Reports*, vol. 14, no. 1, p. 18490, Aug. 2024.
*   [5] Y. Zhang, S. Mao, T. Ge, X. Wang, A. de Wynter, Y. Xia, W. Wu, T. Song, M. Lan, and F. Wei, “LLM as a Mastermind: A Survey of Strategic Reasoning with Large Language Models,” 2024.
*   [6] X. Shao, W. Jiang, F. Zuo, and M. Liu, “SwarmBrain: Embodied agent for real-time strategy game StarCraft II via large language models,” 2024.
*   [7] W. J. Zhou, B. Subagdja, A.-H. Tan, and D. W.-S. Ong, “Hierarchical control of multi-agent reinforcement learning team in real-time strategy (RTS) games,” *Expert Systems with Applications*, vol. 186, 2021.
*   [8] Y. Cao and C. S. G. Lee, “Robot Behavior-Tree-Based Task Generation with Large Language Models,” Feb. 2023.
*   [9] M. Samvelyan, T. Rashid, C. S. de Witt, G. Farquhar, N. Nardelli, T. G. J. Rudner, C.-M. Hung, P. H. S. Torr, J. Foerster, and S. Whiteson, “The StarCraft Multi-Agent Challenge,” Dec. 2019.
*   [10] T. Rashid, M. Samvelyan, C. S. D. Witt, G. Farquhar, J. Foerster, and S. Whiteson, “Monotonic Value Function Factorisation for Deep Multi-Agent Reinforcement Learning,” *ArXiv*, Mar. 2020.
*   [11] C. Yu, A. Velu, E. Vinitsky, Y. Wang, A. Bayen, and Y. Wu, “The Surprising Effectiveness of PPO in Cooperative Multi-Agent Games,” in *Neural Information Processing Systems*, Mar. 2021.
*   [12] B. Ellis, J. Cook, S. Moalla, M. Samvelyan, M. Sun, A. Mahajan, J. N. Foerster, and S. Whiteson, “SMACv2: An Improved Benchmark for Cooperative Multi-Agent Reinforcement Learning,” Oct. 2023.
*   [13] A. Rutherford, B. Ellis, M. Gallici, J. Cook, A. Lupu, G. Ingvarsson, T. Willi, A. Khan, C. S. de Witt, A. Souly, S. Bandyopadhyay, M. Samvelyan, M. Jiang, R. T. Lange, S. Whiteson, B. Lacerda, N. Hawes, T. Rocktaschel, C. Lu, and J. N. Foerster, “JaxMARL: Multi-Agent RL Environments in JAX,” Dec. 2023.
*   [14] T. Guo, X. Chen, Y. Wang, R. Chang, S. Pei, N. V. Chawla, O. Wiest, and X. Zhang, “Large Language Model based Multi-Agents: A Survey of Progress and Challenges,” Apr. 2024.
*   [15] B. Pan, J. Lu, K. Wang, L. Zheng, Z. Wen, Y. Feng, M. Zhu, and W. Chen, “AgentCoord: Visually Exploring Coordination Strategy for LLM-based Multi-Agent Collaboration,” Apr. 2024.
*   [16] S. Han, Q. Zhang, Y. Yao, W. Jin, Z. Xu, and C. He, “LLM Multi-Agent Systems: Challenges and Open Problems,” 2024.
*   [17] J. Kramár, T. Eccles, I. Gemp, A. Tacchetti, K. R. McKee, M. Malinowski, T. Graepel, and Y. Bachrach, “Negotiation and honesty in artificial intelligence methods for the board game of Diplomacy,” *Nature Communications*, vol. 13, no. 1, p. 7214, Dec. 2022.
*   [18] D. Paglieri, B. Cupiał, S. Coward, U. Piterbarg, M. Wolczyk, A. Khan, E. Pignatelli, Ł. Kuciński, L. Pinto, R. Fergus, J. N. Foerster, J. Parker-Holder, and T. Rocktäschel, “BALROG: Benchmarking Agentic LLM and VLM Reasoning On Games,” Nov. 2024.
*   [19] M. G. Mecattaf, B. Slater, M. Tešić, J. Prunty, K. Voudouris, and L. G. Cheke, “A little less conversation, a little more action, please: Investigating the physical common-sense of LLMs in a 3D embodied environment,” 2024.
*   [20] K. Wang, X. Zhang, H. Liu, S. Han, H. Ma, and T. Hu, “CreDes: Causal Reasoning Enhancement and Dual-End Searching for Solving Long-Range Reasoning Problems using LLMs,” 2024.
*   [21] M. Xing, R. Zhang, H. Xue, Q. Chen, F. Yang, and Z. Xiao, “Understanding the Weakness of Large Language Model Agents within a Complex Android Environment,” in *Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*.   Barcelona Spain: ACM, Aug. 2024, pp. 6061–6072.
*   [22] Y. Zhou, L. Feng, Y. Ke, X. Jiang, J. Yan, X. Yang, and W. Zhang, “Towards Vision-Language Geo-Foundation Model: A Survey,” 2024.
*   [23] P. Rahmanzadehgervi, L. Bolton, M. R. Taesiri, and A. T. Nguyen, “Vision language models are blind,” in *Computer Vision – ACCV 2024*, M. Cho, I. Laptev, D. Tran, A. Yao, and H. Zha, Eds.   Singapore: Springer Nature Singapore, 2025, pp. 293–309.
*   [24] J. Bradbury, R. Frostig, P. Hawkins, M. J. Johnson, C. Leary, D. Maclaurin, G. Necula, A. Paszke, J. VanderPlas, S. Wanderman-Milne, and Q. Zhang, “JAX: composable transformations of Python+NumPy programs,” 2018.
*   [25] N. Sevcenko, T. Appel, M. Ninaus, K. Moeller, and P. Gerjets, “Theory-based approach for assessing cognitive load during time-critical resource-managing human–computer interactions: An eye-tracking study,” *Journal on Multimodal User Interfaces*, vol. 17, 2023.
*   [26] M. Colledanchise and P. Ögren, *Behavior trees in robotics and AI: An introduction*.   CRC Press, 2018.
*   [27] J. Achiam, S. Adler, S. Agarwal, L. Ahmad, I. Akkaya, F. L. Aleman, D. Almeida, J. Altenschmidt, S. Altman, S. Anadkat *et al.*, “Gpt-4 technical report,” *arXiv preprint arXiv:2303.08774*, 2023.
*   [28] Anthropic, “The claude 3 model family: Opus, sonnet, haiku,” 2024.
*   [29] A. Dubey, A. Jauhri, A. Pandey, A. Kadian, A. Al-Dahle, A. Letman, A. Mathur, A. Schelten, A. Yang, A. Fan *et al.*, “The llama 3 herd of models,” *arXiv preprint arXiv:2407.21783*, 2024.
*   [30] X. Lei, Z. Yang, X. Chen, P. Li, and Y. Liu, “Scaffolding coordinates to promote vision-language coordination in large multi-modal models,” *arXiv preprint arXiv:2402.12058*, 2024.

### -A LLM instruction and game for the example of the teaser figure

#### -A1 Instruction

    # Map InstructionYou are a game assistant that helps the player in a strategy video game.Your task is to discuss with the player to come up with a plan to win the game’s scenario (which will be provided later). Work with the player by giving feedback about their propositions, asking questions to clarify, obtain more details, or decide between different propositions. The player can also ask you questions.You will be given a textual description of each pertinent element of the map using their name, terrain type, and bounding boxes (bottom-left corner - top-right corner).In the game, there are four types of terrain:- Normal: units can cross and see through (by default, the whole map is normal).- Buildings: units cannot cross or see through buildings.- Water: units cannot cross over water but can see over it.- Trees: units cannot see through trees but can move over them. In particular, once a unit is inside a tree area, it cannot see any other unit.Also, for simplicity, bridges that allow crossing water terrain will be specified. They correspond to normal terrain.You can assume that any part of the map that is not included in any of the pertinent elements has a normal type of terrain.A common convention is that East = Right, North = Top, West = Left, and South = Bottom. So the point (0, 0) is the bottom-left corner of the map.The x-axis increases from West to East = from Left to Right, and the y-axis increases from South to North = from Bottom to Top.Format:[Name]: [terrain type] at [circle or square coordinate], [circle or square coordinate], ..., [circle or square coordinate]where [circle] = [(x, y) coordinates of center] with radius [R]where [square coordinate] = [(x, y) coordinates of the bottom-left corner] - [(x, y) coordinates of the top-right corner]### For exampleEast Forest: trees at (12, 63) with radius 20North River: water at (0, 85) - (100, 90)North River’s Bridge: normal at (45, 85) - (55, 90)West Forest: trees at (0, 23) with radius 6, (5, 33) with radius 7## MarkersThrough the discussion, the player can define markers on the map, which will be provided to you using the following format:Markers:A at (17, 5)B at (6, 32)C at (25, 28)# Information About the UnitsDescription of the unit types in the game:spearmen: Health=24; Sight range=15; Attack range=1; Moving speed=2; Attack damage=1; Attack cooldown=1archer: Health=2; Sight range=15; Attack range=15; Moving speed=4; Attack damage=3; Attack cooldown=1cavalry: Health=12; Sight range=15; Attack range=1; Moving speed=12; Attack damage=1; Attack cooldown=1Spearmen are strong against Cavalry because they have more health.    Cavalry are strong against Archers because they can quickly engage in close combat where Archers are weak.    Archers are strong against Spearmen because they can attack from a longer distance.Here is the list of unit IDs (in the form of a Python slice a:b with a included and b not included) for each team and the type of units they are composed of:Descriptions of each team’s composition:Allies:spearmen: [0:350]archer: [350:700]Enemies:spearmen: [0:900]You will be provided with a list of all the units’ current health and position.Both teams are composed of many units. You should start by analyzing the positions of the units still alive to compose groups and compute their average position so that you can form the plan and send units to the appropriate positions.# How You Should Handle the Player’s PromptThe user will ask you to write one plan. If you already have a shared conversation and the user asks you to modify or add steps to the plan, take care to build upon the already selected plan.# Syntax for a Detailed PlanA detailed plan is a set of steps to achieve in a given order until all the steps are completed.You must provide the steps of the plan between the two keywords "BEGIN PLAN" and "END PLAN".As you only propose one plan, you should use these two keywords once.One step comprises:- A numeral ID- A list of prerequisite steps that need to be completed before the step is rolled out- An objectiveA succession (at least one) of groups of units:- The unit IDs of the group- Target position on the map for the units to go to if there are no enemies in sight- A behavior for the units if there are enemies in sight# Syntax for a Detailed Plan (continued)The list of prerequisites corresponds to the list of steps’ IDs that must be completed before trying to achieve the objective.There are two kinds of objectives:- **Elimination objective:** where the objective is completed when all the targets are eliminated. In that case, you must provide a list of enemies’ IDs or the keyword "all" if all enemies are targets.- **Position objective:** where the objective is completed when all the concerned units are close to their target position.Position objectives are a good way to move your units, but if the end objective is to eliminate enemies, it can be more straightforward to directly set an elimination objective and use the target position to move the units.The concerned units are either the keyword "all" (if all the allies’ units are concerned) or a list of integers corresponding to the unit IDs.The behavior corresponds to a low-level and local behavior that the units will follow.Here is the list of available behaviors:- **attack_in_close_range:** the unit attacks the enemies in close range if there are enemies or moves toward the target position if there are no enemies.- **attack_and_move:** the unit attacks the enemies without moving if there are enemies or moves toward the target position if there are no enemies.- **attack_in_long_range:** the unit attacks the enemies in long range if there are enemies or moves toward the target position if there are no enemies.- **follow_map:** ignore the enemies and simply move straight to the target objective. Only use it when the player asks you to ignore the enemies.Apart from standing still, all these behaviors will only be active if an enemy is in sight. Otherwise, they will move to a target position if you set one in the plan or stand still if no target position is set.Remember that units collide and push each other, so ignoring the enemies may not be the fastest way to reach a target position if there are enemies on the way who could block you.The syntax format of a step is the following:Step ID: (where you replace ID with the integer ID of the step)prerequisites: [s_1, s_2, ..., s_n] (where the s_i correspond to the prerequisites’ steps IDs. Note that the list can be empty. In that case, simply write [])objective: position [or] elimination UNIT_LIST(At least one list of units and their assigned behavior and target position, but there can be as many groups as there are units, as one unit can belong to two groups:)UNIT_LIST:- target position: (x, y) (The integer x and y coordinate on the map.)- behavior: behavior_name target_1 target_2 ... target_n (where behavior_name is an available behavior and target_1 to target_n are the targeted unit types or just the keyword "any" if the behavior targets any unit_types)A UNIT_LIST is the list of unit IDs, and it has the following format:- Either it’s the word "all" (without quotes), which means that all the units of the corresponding team are concerned.- Or it’s a list of IDs in the format "[X1, X2, ..., Xn]" where Xi can either be an integer or a slice "a:b" (with a and b as integers, b>a, a included and b not included like in Python’s range function). If a is not specified (i.e., ":b") it is considered to be 0, and if b is not specified (i.e., "a:") it is considered to be the total number of agents in the considered team.### IMPORTANT:- All positions (x, y) must be integers. If you want to give float positions, convert them first into integers.- Do not add comments to the plan specification, as it interferes with the parser. If you want to give comments, give them outside the "BEGIN PLAN" and "END PLAN".- A unit can only belong to one group of units for the same step.## Example of a Valid Detailed Plan:BEGIN PLANStep 0:prerequisites: []objective: positionunits: all- target position: (24, 14)- behavior: attack_and_move anyStep 1:prerequisites: [0]objective: positionunits: [0,1,2,10:]- target position: (24, 16)- behavior: attack_and_move anyunits: [10:15,30:]- target position: (24, 14)- behavior: attack_and_move anyStep 2:prerequisites: [1]objective: elimination [:15]units: [:30]- target position: (24, 24)- behavior: attack_in_close_range archer spearmenunits: [30:60]- target position: (24, 24)- behavior: attack_in_long_range spearmenEND PLAN## List of Planning Mistakes You Should Avoid:Avoid creating a series of position objectives with different groups of units waiting for each other (through a chain of prerequisites). Instead, regroup those positions into one step so that all units can move simultaneously.Ensure all units have assigned behaviors at every step. If two or more steps can be active simultaneously (because they share the same prerequisites), each unit must belong to at least one group.For elimination objectives, ensure the target position is close enough to the targeted enemy units for effective engagement.Elimination objectives already have a default target position, so avoid unnecessary position objectives before them. Instead, set the target position of the elimination objective to the same location.Ensure the units’ IDs are enclosed in square brackets [ and ].Use the correct type name for long-range units as "archer" (not "archers").Verify the name of the unit behaviors (e.g., "attack_in_close_range") and ensure all units’ target positions are integers.Be sure to adapt this format and guidance as needed when responding to the player’s requests.In this scenario, you control the Allies and must prevent the Enemies from invading the center of your camp in (150, 134).    You must elaborate a defensive plan that eliminates all the Enemies while preventing any of them from reaching the center of your camp.    If one Enemy unit reaches the center of your camp, all your units are eliminated, or if your plan is completed while there are still standing Enemy units, you lose the game.    Your intel reports inform you that the enemy will converge toward the center of your camp and attack your units in sight at close range.

#### -A2 The game info

Health and positions of all the units of each team ($\emptyset$ means that the unit is dead).Allies:Health: [24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 24, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2,2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2]X positions: [180, 181, 189, 195, 170, 169, 186, 176, 166, 172, 194, 166, 191, 165, 190, 165, 191, 189, 165, 194, 167, 165, 173, 193, 171, 193, 168, 184, 166, 172, 187, 176, 187, 176, 165, 174, 194, 165, 182, 185, 195, 188, 176, 169, 178, 186, 182, 178, 167, 177, 174, 172, 168, 183, 188, 184, 176, 187, 173, 170, 189, 175, 184, 179, 186, 195, 192, 190, 185, 195, 173, 172, 194, 195, 171, 192, 172, 185, 195, 195, 190, 167, 190, 181, 188, 178, 172, 190, 181, 176, 194, 182, 181, 171, 186, 194, 174, 190, 169, 183, 165, 175, 179, 179, 189, 167, 178, 180, 192, 181, 168, 190, 187, 182, 176, 170, 171, 190, 193, 169, 177, 174, 182, 172, 193, 169, 189, 174, 166, 175, 188, 181, 181, 194, 170, 188, 171, 167, 182, 179, 169, 184, 179, 187, 187, 183, 172, 183, 186, 171, 181, 188, 172, 168, 183, 195, 185, 173, 166, 183, 182, 185, 193, 169, 168, 191, 177, 176, 169, 176, 187, 180, 180, 193, 171, 173, 174, 194, 172, 190, 176, 195, 189, 190, 186, 192, 187, 195, 176, 172, 181, 181, 171, 186, 185, 172, 185, 195, 181, 191, 172, 195, 188, 191, 183, 184, 187, 171, 174, 183, 182, 194, 173, 186, 193, 191, 192, 166, 172, 176, 174, 165, 166, 168, 186, 189, 175, 183, 174, 187, 190, 171, 167, 174, 194, 170, 195, 173, 191, 173, 172, 174, 166, 171, 176, 186, 188, 174, 169, 193, 167, 190, 190, 181, 191, 169, 185, 182, 178, 187, 195, 170, 183, 171, 186, 179, 193, 182, 172, 167, 179, 166, 167, 195, 192, 166, 171, 169, 178, 176, 168, 174, 183, 171, 194, 180, 166, 181, 172, 172, 180, 176, 180, 189, 171, 189, 168, 167, 181, 192, 188, 168, 165, 174, 187, 166, 177, 190, 175, 183, 172, 176, 175, 181, 185, 193, 173, 192, 166, 168, 185, 188, 167, 180, 180, 194, 194, 185, 171, 186, 166, 190, 171, 177,177, 170, 182, 165, 194, 190, 179, 178, 187, 186, 183, 173, 187, 168, 182, 168, 189, 183, 194, 169, 177, 169, 182, 186, 187, 174, 179, 184, 174, 195, 192, 178, 169, 176, 172, 195, 193, 180, 186, 166, 190, 172, 165, 195, 176, 179, 172, 175, 188, 180, 195, 190, 186, 177, 185, 175, 176, 181, 188, 181, 169, 190, 190, 179, 176, 188, 172, 194, 188, 174, 192, 165, 176, 175, 174, 175, 180, 184, 170, 165, 181, 184, 178, 174, 181, 165, 192, 175, 172, 182, 181, 165, 177, 189, 191, 181, 181, 174, 186, 170, 174, 183, 173, 191, 190, 166, 190, 177, 178, 178, 170, 195, 175, 189, 187, 171, 180, 173, 175, 185, 168, 168, 167, 182, 181, 187, 186, 175, 186, 173, 165, 193, 179, 188, 192, 191, 166, 194, 185, 183, 175, 177, 178, 179, 168, 173, 184, 190, 177, 165, 192, 183, 188, 172, 185, 178, 193, 190, 167, 178, 188, 168, 193, 175, 183, 187, 174, 174, 188, 190, 190, 172, 195, 188, 179, 187, 183, 180, 182, 174, 172, 189, 168, 172, 192, 186, 176, 166, 192, 186, 174, 191, 166, 187, 167, 176, 192, 168, 183, 166, 176, 167, 173, 188, 166, 189, 188, 190, 177, 168, 189, 170, 179, 187, 183, 191, 194, 191, 186, 180, 194, 185, 183, 180, 172, 169, 182, 182, 174, 183, 194, 172, 178, 178, 193, 166, 171, 188, 173, 187, 173, 177, 177, 183, 171, 191, 172, 190, 184, 172, 181, 181, 168, 176, 178, 168, 172, 166, 184, 186, 178, 172, 190, 181, 170, 192, 192, 180, 166, 188, 175, 189, 172, 171, 195, 190, 168, 178, 184, 194, 179, 184, 184, 170, 165, 176, 185, 182, 189, 186, 190, 195, 185, 178, 186, 174, 176, 181, 169, 189, 170, 185, 174, 185, 181, 190, 194, 167, 185, 179, 171, 170, 188, 171, 191, 189, 195, 195, 195, 173, 169, 174, 177, 186, 168, 175, 194, 171, 195, 191, 187, 191, 166, 173, 178, 169, 184, 190, 192, 173, 182, 185, 189, 189, 170, 166, 185, 171, 168, 181, 192, 168, 192, 181, 166, 179, 182, 169, 167, 174, 174, 171, 172, 174, 181, 167]Y positions: [107, 104, 105, 105, 113, 103, 109, 113, 105, 114, 110, 114, 112, 110, 110, 112, 110, 103, 107, 103, 103, 111, 106, 101, 110, 107, 105, 112, 112, 115, 106, 106, 115, 107, 102, 103, 111, 110, 107, 105, 109, 107, 109, 114, 101, 101, 112, 115, 108, 111, 102, 110, 106, 110, 105, 107, 103, 103, 112, 112, 110, 116, 103, 101, 114, 111, 103, 110, 106, 103, 108, 101, 116, 116, 101, 109, 102, 101, 104, 103, 110, 115, 103, 116, 111, 116, 103, 115, 102, 115, 113, 102, 114, 108, 104, 105, 114, 103, 106, 111, 107, 113, 113, 101, 107, 108, 104, 116, 104, 110, 105, 116, 103, 114, 102, 101, 115, 106, 115, 113, 112, 105, 109, 115, 113, 111, 105, 104, 114, 103, 105, 115, 103, 108, 107, 114, 102, 116, 106, 115, 106, 112, 110, 106, 106, 102, 111, 115, 111, 106, 110, 112, 105, 105, 105, 114, 113, 101, 114, 109, 110, 101, 110, 113, 110, 107, 116, 115, 112, 101, 104, 114, 107, 115, 111, 108, 109, 106, 101, 115, 104, 106, 108, 114, 101, 101, 105, 104, 106, 114, 105, 112, 108, 116, 111, 105, 104, 115, 106, 106, 103, 101, 109, 108, 109, 106, 101, 102, 109, 102, 104, 114, 103, 110, 107, 110, 103, 116, 115, 114, 107, 108, 109, 109, 109, 115, 107, 115, 104, 111, 110, 112, 101, 110, 116, 103, 110, 103, 102, 110, 110, 113, 105, 112, 113, 110, 115, 107, 108, 105, 113, 110, 101, 112, 104, 116, 103, 107, 112, 106, 115, 111, 103, 113, 110, 104, 108, 106, 114, 113, 105, 101, 116, 112, 106, 102, 101, 109, 101, 116, 102, 102, 108, 103, 105, 104, 116, 103, 112, 111, 110, 109, 116, 103, 112, 110, 110, 116, 114, 112, 111, 106, 101, 110, 116, 107, 111, 110, 113, 102, 109, 112, 101, 114, 103, 116, 105, 116, 104, 113, 105, 101, 105, 106, 112, 116, 103, 116, 101, 103, 101, 106, 104, 105, 110, 105, 105, 112, 112, 108, 114, 106, 112, 114, 116, 116, 111, 114, 111, 109, 103, 108, 114, 112, 110, 111, 108, 103, 103, 106, 107, 102, 112, 104, 110, 116, 105, 112, 101, 104, 101, 108, 112, 106, 106, 102, 101, 115, 112, 106, 114, 106, 116, 103, 109, 103, 110, 105, 109, 113, 104, 113, 115, 116, 111, 102, 111, 101, 106, 107, 104, 105, 108, 101,108, 108, 116, 101, 107, 104, 106, 114, 110, 107, 115, 105, 107, 110, 103, 106, 109, 115, 112, 112, 109, 113, 106, 105, 102, 112, 110, 105, 104, 102, 109, 103, 108, 107, 115, 113, 111, 115, 113, 104, 110, 113, 115, 107, 103, 109, 112, 110, 105, 113, 102, 116, 115, 114, 102, 116, 112, 108, 109, 110, 111, 111, 102, 116, 102, 110, 113

### -B LLM answer and plan to Fig. [5](https://arxiv.org/html/2412.11761v1#S4.F5 "Figure 5 ‣ IV Experiments ‣ Harnessing Language for Coordination: A Framework and Benchmark for LLM-Driven Multi-Agent Control") examples

#### -B1 Coordination with Sonnet

I’ll help you create a plan to effectively eliminate the enemies. Looking at the positions, I notice that all enemy units are currently in the Northern Forest (y-coordinates between 135-149), and they will need to move south through the middle of the map.Let’s analyze our forces:- We have 500 spearmen (IDs 0-499)- We have 500 archers (IDs 500-999)Given the map’s width (150) and our need to protect archers, I suggest creating three main battle groups across the middle of the map, each with both spearmen and archers. The spearmen will form a protective front line while archers attack from behind.Here’s the detailed plan:BEGIN PLANStep 0:prerequisites: []objective: positionunits: [0:167]- target position: (25, 75)- behavior: attack_in_close_range anyunits: [167:334]- target position: (75, 75)- behavior: attack_in_close_range anyunits: [334:500]- target position: (125, 75)- behavior: attack_in_close_range anyunits: [500:667]- target position: (25, 65)- behavior: attack_in_long_range anyunits: [667:834]- target position: (75, 65)- behavior: attack_in_long_range anyunits: [834:1000]- target position: (125, 65)- behavior: attack_in_long_range anyStep 1:prerequisites: [0]objective: elimination allunits: [0:167]- target position: (25, 75)- behavior: attack_in_close_range anyunits: [167:334]- target position: (75, 75)- behavior: attack_in_close_range anyunits: [334:500]- target position: (125, 75)- behavior: attack_in_close_range anyunits: [500:667]- target position: (25, 65)- behavior: attack_in_long_range anyunits: [667:834]- target position: (75, 65)- behavior: attack_in_long_range anyunits: [834:1000]- target position: (125, 65)- behavior: attack_in_long_range anyEND PLANThis plan creates three battle groups positioned across the map:1. Western group (at x=25)2. Central group (at x=75)3. Eastern group (at x=125)Each group has:- About 167 spearmen in front (at y=75)- About 167 archers behind them (at y=65)The spearmen are positioned 10 units ahead of the archers to provide protection while still allowing the archers to shoot over them at approaching enemies.The groups are spread out to cover the entire width of the map, ensuring no enemies can slip through. When enemies come down from the Northern Forest, they’ll face a solid line of spearmen backed by archer fire.Would you like me to adjust any aspect of this plan?

#### -B2 Exploit weaknesses with Sonnet

Let me analyze the situation and the enemy positions.The map is divided into 4 quadrants by two rivers that cross at (46-52, 47-53), with bridges allowing passage between these quadrants.Enemy forces are organized in three distinct groups:1. Northwest quadrant (around x:5-25, y:75-95): Mainly archers (health=2)2. Northeast quadrant (around x:75-95, y:5-25): Mainly spearmen (health=24)3. Northeast quadrant (around x:75-95, y:75-95): Mainly cavalry (health=12)Our forces are currently positioned in the Southwest quadrant (around x:5-40, y:5-40).Strategic considerations:1. We should use cavalry against archers (NW group)2. Archers against spearmen (NE group)3. Spearmen against cavalry (NE group)Here’s my proposed plan:BEGIN PLANStep 0:prerequisites: []objective: positionunits: [0:250]- target position: (19, 49)- behavior: attack_and_move anyunits: [250:500]- target position: (21, 49)- behavior: attack_and_move anyunits: [500:750]- target position: (20, 49)- behavior: attack_and_move anyStep 1:prerequisites: [0]objective: positionunits: [500:750]- target position: (21, 76)- behavior: attack_in_close_range archerunits: [250:500]- target position: (76, 21)- behavior: attack_in_long_range spearmenunits: [0:250]- target position: (76, 76)- behavior: attack_in_close_range cavalryStep 2:prerequisites: [1]objective: elimination allunits: [500:750]- target position: (15, 85)- behavior: attack_in_close_range archerunits: [250:500]- target position: (85, 15)- behavior: attack_in_long_range spearmenunits: [0:250]- target position: (85, 85)- behavior: attack_in_close_range cavalryEND PLANPlan explanation:1. First, we move all forces to the west bridge (Step 0)2. Then split our forces (Step 1):- Cavalry (500:750) goes north to engage archers- Archers (250:500) goes east to engage spearmen- Spearmen (0:250) goes northeast to engage cavalry3. Finally, each group moves to eliminate their targeted enemies (Step 2)This plan leverages each unit type’s strengths against their natural enemies while using the bridges strategically to reach their positions.

#### -B3 Markers with 4o-mini

To achieve your goal of moving the troops to the markers A, B, C, and D while ignoring enemies, we can create a plan that sequentially moves the units to each marker and then to the objective position at (61, 0).However, since the objective is to reach the position at (61, 0) with at least one unit, we should ensure that the units are grouped effectively and can reach the objective after visiting the markers.Let’s break down the plan:1. Move all units to marker A at (193, 85).2. Move all units to marker B at (49, 136).3. Move all units to marker C at (9, 134).4. Move all units to marker D at (11, 9).5. Finally, move all units to the objective position at (61, 0).I will now create the detailed plan based on this sequence.BEGIN PLANStep 0:prerequisites: []objective: positionunits: all- target position: (193, 85)- behavior: follow_mapStep 1:prerequisites: [0]objective: positionunits: all- target position: (49, 136)- behavior: follow_mapStep 2:prerequisites: [1]objective: positionunits: all- target position: (9, 134)- behavior: follow_mapStep 3:prerequisites: [2]objective: positionunits: all- target position: (11, 9)- behavior: follow_mapStep 4:prerequisites: [3]objective: positionunits: all- target position: (61, 0)- behavior: follow_mapEND PLANThis plan allows your units to move to the specified markers and then to the objective position while ignoring enemies. Would you like to make any adjustments or proceed with this plan?

#### -B4 Terrain with 4o

Here is the detailed plan based on your instructions:BEGIN PLANStep 0:prerequisites: []objective: positionunits: all- target position: (164, 71)- behavior: follow_map anyStep 1:prerequisites: [0]objective: positionunits: all- target position: (33, 159)- behavior: follow_map anyStep 2:prerequisites: [1]objective: positionunits: all- target position: (11, 101)- behavior: follow_map anyStep 3:prerequisites: [2]objective: positionunits: all- target position: (23, 44)- behavior: follow_map anyStep 4:prerequisites: [3]objective: positionunits: all- target position: (61, 0)- behavior: follow_map anyEND PLAN

#### -B5 Strategic points with o1-mini

BEGIN PLANStep 0:prerequisites: []objective: positionunits: [0:39]- target position: (45, 252)- behavior: attack_in_close_range anyunits: [350:389]- target position: (45, 252)- behavior: attack_in_long_range anyunits: [39:78]- target position: (138, 205)- behavior: attack_in_close_range anyunits: [389:428]- target position: (138, 205)- behavior: attack_in_long_range anyunits: [78:117]- target position: (205, 142)- behavior: attack_in_close_range anyunits: [428:467]- target position: (205, 142)- behavior: attack_in_long_range anyunits: [117:156]- target position: (252, 87)- behavior: attack_in_close_range anyunits: [467:506]- target position: (252, 87)- behavior: attack_in_long_range anyunits: [156:195]- target position: (272, 37)- behavior: attack_in_close_range anyunits: [506:545]- target position: (272, 37)- behavior: attack_in_long_range anyunits: [195:234]- target position: (29, 238)- behavior: attack_in_close_range anyunits: [545:584]- target position: (29, 238)- behavior: attack_in_long_range anyunits: [234:273]- target position: (65, 135)- behavior: attack_in_close_range anyunits: [584:623]- target position: (65, 135)- behavior: attack_in_long_range anyunits: [273:312]- target position: (135, 75)- behavior: attack_in_close_range anyunits: [623:662]- target position: (135, 75)- behavior: attack_in_long_range anyunits: [312:350]- target position: (249, 21)- behavior: attack_in_close_range anyunits: [662:700]- target position: (249, 21)- behavior: attack_in_long_range anyEND PLAN

### -C Grammar for our Behavior trees

    ?start: node%import common.WS%ignore WSnodes : node ("::" node | "|>" node)*node  :    | sequence    | fallback    | action    | conditionsequence  : "S" "(" nodes ")"fallback  : "F" "(" nodes ")"action    : "A" "(" atomic ")"condition : "C" "(" atomic ")"atomic :    | move    | attack    | stand    | follow_map    | in_sight    | in_reach    | is_dying    | is_armed    | is_flock    | is_type    | is_in_forest    | success_action    | failure_actionmove      : "move" (direction | sense qualifier (foe | friend) (unit ("or" unit)* |any)?)attack    : "attack" qualifier (unit ("or" unit)* |any)?stand     : "stand"in_sight  : "in_sight" (foe | friend) (unit ("or" unit)* |any)?in_reach  : "in_reach" (foe | friend) source time (unit ("or" unit)* |any)?is_dying  : "is_dying" (self | foe | friend) intensityis_armed  : "is_armed" (self | foe | friend)is_flock  : "is_flock" (foe | friend) directionis_type   : "is_type" negation unitfollow_map : "follow_map" sense intensity?is_in_forest : "is_in_forest"success_action : "success_action"failure_action: "failure_action"sense     : /toward|away_from/direction : /north | east | south | west | center/foe       : /foe/friend    : /friend/qualifier : /strongest | weakest | closest | farthest | random/intensity : /low | middle | high/self      : /self/unit      : /spearmen | archer| cavalry | balista | dragon | civilian/any       : /any/negation  : /a | not_a/source    : /them_from_me | me_from_them/time      : /now | low | middle | high/

### -D List of behavior trees used in this paper

#### -D1 Available for HIVE

Attack in long range:

F(S(C( in_reach foe me_from_them high any) :: A (move away_from closest foe any)) :: A (attack random any) ::  A (follow_map toward))

Attack in close range:

F( A (attack random any) :: A (move toward closest foe any) :: A (follow_map toward))

Attack and move:

F( A (attack random any) :: A (follow_map toward low) :: A (move toward closest foe any) )

Move toward target position:

A (follow_map toward)

Stand:

A (stand)

#### -D2 Only present in the opposing units

The two following behavior trees are used by the opposing side in the Markers and Terrain ability test to avoid the enemy unit to get lost in the trees where they cannot see the player’s units any longer.

Attack in long range and stay out of the forest:

F (S (C (is_in_forest) :: A (follow_map toward)) :: S(C( in_reach foe me_from_them high any) :: A (move away_from closest foe any)) :: A (attack closest any))

Attack in close range and stay out of the forest:

F (S (C (is_in_forest) :: A (follow_map toward)) :: A (attack closest any) :: A (move toward closest foe any))

### -E Units characteristics

TABLE II: Characteristics of the three types of units used in this paper.

|  | Spearman | Archer | Cavalry |
| --- | --- | --- | --- |
| Visual shape | square | circle | triangle |
| --- | --- | --- | --- |
| Speed (unit distance/step) | 1 | 2 | 6 |
| Max health | 24 | 2 | 12 |
| Attack damage | 1 | 3 | 1 |
| Attack range | 1 | 15 | 1 |

### -F Prompts used for the 5 ability tests

#### -F1 (a) Coordinate

0. Make a plan that forms as many squads as you think necessary to cover the central row of the battlefield to eliminate all the enemies as fast as possible. Judisiously place our long range units so that they are not in close combat.1. Design a plan that forms as many group of units as you think necessary to cover the central row of the battlefield to eliminate all the enemies as quickly as possible. Judisiously place our long range units so that they protected by our close range units.2. Write down a plan that forms as many group of units as you think necessary to cover the central row of the map to eliminate all the enemies as fast as possible. Ensure to place our long range units so that they are not in close combat.3. Design a plan that forms as many group of units as you think necessary to cover the central row of the map to eliminate all the enemies as quickly as possible. Make sure to place our long range units so that they in far range of the enemies.4. Make a plan that forms as many squads as you think necessary to cover the middle row of the map to eliminate all the enemies as quickly as possible. Ensure to place our archers so that they protected by our close range units.5. Design a plan that forms as many battalions as you think necessary to cover the middle row of the battlefield to eliminate all the enemies as fast as possible. Ensure to place our archers so that they in far range of the enemies.6. Form as many battalions as you think necessary to cover the central row of the battlefield to eliminate all the enemies as fast as possible. Make sure to place our archers so that they are not in close combat.7. Write down a plan that forms as many battalions as you think necessary to cover the central row of the battlefield to eliminate all the enemies as fast as possible. Make sure to place our long range units so that they protected by our close range units.8. Make a plan that forms as many squads as you think necessary to cover the middle row of the battlefield to eliminate all the enemies as quickly as possible. Judisiously place our archers so that they in far range of the enemies.9. Make a plan that forms as many group of units as you think necessary to cover the middle row of the battlefield to eliminate all the enemies as quickly as possible. Make sure to place our archers so that they in far range of the enemies.

#### -F2 (b) Exploit weakness

0. First, compute the positions and types of the enemy battalions. Then, design a plan that play on their weaknesses to split our units into three groups with specific strengths and send each one against one of the enemy squads to minimize our casualties. 1. First, determine the positions and types of the enemy battalions. Then, make a plan that play on their weaknesses to split our army into three battalions with specific strengths and send each one against one of the enemy groups to maximize efficiency. 2. First, analyse the situation to determine the positions and types of the enemy squads. Then, write a plan that play on their weaknesses to split our army into three battalions with specific strengths and send each one against one of the enemy battalions to minimize our casualties. 3. First, analyse the situation to determine the positions and types of the enemy groups. Then, make a plan that play on their weaknesses to split our units into three squads with specific strengths and send each one against one of the enemy battalions to minimize our casualties. 4. First, analyse the situation to compute the positions and types of the enemy squads. Then, design a plan that play on their weaknesses to split our army into three groups with specific strengths and send each one against one of the enemy battalions to minimize our casualties. 5. First, analyse the situation to compute the positions and types of the enemy groups. Then, design a plan that play on their weaknesses to split our army into three squads with specific strengths and send each one against one of the enemy groups to maximize efficiency. 6. First, analyse the situation to determine the positions and types of the enemy groups. Then, make a plan that play on their weaknesses to split our army into three squads with specific strengths and send each one against one of the enemy groups to minimize our casualties. 7. First, analyse the situation to compute the positions and types of the enemy squads. Then, write a plan that play on their weaknesses to split our army into three groups with specific strengths and send each one against one of the enemy squads to maximize efficiency. 8. First, compute the positions and types of the enemy squads. Then, make a plan that play on their weaknesses to split our army into three squads with specific strengths and send each one against one of the enemy battalions to maximize efficiency. 9. First, compute the positions and types of the enemy battalions. Then, design a plan that play on their weaknesses to split our army into three squads with specific strengths and send each one against one of the enemy groups to minimize our casualties.

#### -F3 (c) Follow markers

The player mark four positions:

*   •

    A: (193, 85);

*   •

    B: (49, 136);

*   •

    C: (9, 134);

*   •

    D: (11, 9).

 0. Move the spearmen to each position in alphabetical order while ignoring enemies and then to the final objective position. 1. Write a plan to move our troups to each marker  while ignoring enemies and then to the final objective position. 2. Design a plan to move our spearmen to each marker in alphabetical order while ignoring enemies and then to the final objective position. 3. Make a plan to move our spearmen to each position in alphabetical order while ignoring enemies and then to the final objective position. 4. Write a plan to move our army to each position  while ignoring enemies and then to the final objective position. 5. Write a plan to move my troups to the positions A, B, C, D while ignoring enemies and then to the  objective position. 6. Move our troups to the markers A, B, C, D while ignoring enemies and then to the  objective position. 7. Make a plan to move our army to the markers A, B, C, D while ignoring enemies and then to the  objective position. 8. Move our spearmen to the positions ABCD while ignoring enemies and then to the  objective position. 9. Make a plan to move the units to the markers A, B, C, D while ignoring enemies and then to the final objective position.

#### -F4 (d) Exploit terrain

0. Make a plan to move my spearmen inside the forest on the right  to the bridge, then make them rush to the forest on the west, slightly on top of the bridge . Then make them hide deep inside the forest on the west  of the map. Finally, make them rush south to the left-bottom corner of the map inside the forest and then to the objective, minimizing the time out of cover. Everything while ignoring enemies to not loose time fighting.1. Write a plan to move my units inside the trees on the east  to the bridge, then make them rush to the forest on the west, slightly on top of the bridge . Then make them hide  inside the trees on the west  of the map. Finally, make them rush down to the south-east corner of the map inside the forest and then to the objective. Everything while ignoring enemies.2. Write a plan to move my spearmen inside the trees on the right  to the bridge, then make them rush to the forest on the west, a bit north of the bridge as it is closer. Then make them hide deep inside the forest on the west most part of the map. Finally, make them rush  to the south-east corner of the map inside the forest and then to the objective, minimizing the time out of cover. Everything while ignoring enemies to not loose time fighting.3. Make a plan to move our units inside the forest on the east not too far to the bridge, then make them rush to the trees on the west,  on top of the bridge to quickly regain cover. Then make them hide  inside the trees on the west  of the map. Finally, make them rush south to the left-bottom corner of the map inside the forest and then to the objective. Everything while ignoring enemies.4. Design a plan to move our spearmen inside the trees on the east close to the bridge, then make them run to the forest on the left,  on top of the bridge to quickly regain cover. Then make them hide deep inside the forest on the left  of the map. Finally, make them rush south to the south-east corner of the map inside the forest and then to the objective. Everything while ignoring enemies.5. Write a plan to move our army inside the forest on the right  to the bridge, then make them move to the forest on the west,  on top of the bridge as it is closer. Then make them hide deep inside the forest on the west most part of the map. Finally, make them rush  to the south-east corner of the map inside the forest and then to the objective, minimizing the time out of cover. Everything while ignoring enemies to not loose time fighting.6. Write a plan to move my army inside the forest on the east not too far to the bridge, then make them rush to the forest on the west,  north of the bridge to quickly regain cover. Then make them hide deep inside the forest on the west most part of the map. Finally, make them rush down to the left-bottom corner of the map inside the forest and then to the objective. Everything while ignoring enemies to not loose time fighting.7. Move the spearmen inside the trees on the east close to the bridge, then make them rush to the forest on the west, a bit on top of the bridge to quickly regain cover. Then make them hide  inside the trees on the left  of the map. Finally, make them rush  to the south-east corner of the map inside the forest and then to the objective. Everything while ignoring enemies to not loose time fighting.8. Design a plan to move our spearmen inside the forest on the east close to the bridge, then make them move to the forest on the west, slightly on top of the bridge . Then make them hide deep inside the trees on the west most part of the map. Finally, make them rush  to the left-bottom corner of the map inside the forest and then to the objective. Everything while ignoring enemies.9. Make a plan to move the army inside the forest on the right not too far to the bridge, then make them run to the trees on the west,  north of the bridge . Then make them hide  inside the trees on the left most part of the map. Finally, make them rush  to the south-east corner of the map inside the forest and then to the objective, minimizing the time out of cover. Everything while ignoring enemies.

#### -F5 (e) Strategize points

0. Split our units on all the bridges to defend them and make sure to send archers and spearmen units on each one.1. Make a plan to split our army on all the bridges to guard them and make sure to send long-range and spearmen units on each one.2. Design a plan to split our units on all the bridges to protect them and make sure to send long-range and spearmen units on each bridge.3. Make a plan to split our units on all the bridges to guard them and make sure to send long-range and close-range units on each one.4. Write a plan to split our army on all the bridges to protect them and make sure to send archers and close-range units on each bridge.5. Make a plan to split our units on all the bridges to protect them and make sure to send archers and spearmen units on each one.6. Split our army on all the bridges to guard them and make sure to send long-range and close-range units on each one.7. Split our army on all the bridges to protect them and make sure to send archers and spearmen units on each one.8. Design a plan to split our units on all the bridges to defend them and make sure to send archers and close-range units on each bridge.9. Make a plan to split our army on all the bridges to guard them and make sure to send archers and close-range units on each bridge.

### -G Prompts used for HIVE alone

’First, study the situation to find a good approach to achieve this mission then design the corresponding plan.’, ’First, analyze the situation to find a good strategy to win this mission then write down the corresponding plan.’, ’First, study the situation to find a good approach to achieve this scenario then make the corresponding plan.’, ’First, analyze the situation to find a good strategy to win this mission then design the corresponding plan.’, ’First, study the situation to find a good approach to achieve this mission then write down the corresponding plan.’, ’First, analyze the situation to find a good strategy to achieve this mission then make the corresponding plan.’, ’First, analyze the situation to find a good strategy to achieve this scenario then write down the corresponding plan.’, ’First, analyze the situation to find a good approach to win this mission then make the corresponding plan.’, ’First, study the situation to find a good approach to win this scenario then make the corresponding plan.’, ’First, analyze the situation to find a good approach to achieve this scenario then design the corresponding plan.’