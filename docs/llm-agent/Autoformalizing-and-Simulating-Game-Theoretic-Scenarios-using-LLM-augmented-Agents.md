<!--yml
category: 未分类
date: 2025-01-11 11:48:54
-->

# Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents

> 来源：[https://arxiv.org/html/2412.08805/](https://arxiv.org/html/2412.08805/)

\affiliation\institution

Department of Computer Science, Royal Holloway, University of London \cityEgham \countryUnited Kingdom \affiliation \institutionDepartment of Computer Science, Royal Holloway, University of London \cityEgham \countryUnited Kingdom \affiliation \institutionDepartment of Computer Science, Royal Holloway, University of London \cityEgham \countryUnited Kingdom

Agnieszka Mensfelt [agnieszka.mensfelt@rhul.ac.uk](mailto:agnieszka.mensfelt@rhul.ac.uk) ,  Kostas Stathis [kostas.stathis@rhul.ac.uk](mailto:kostas.stathis@rhul.ac.uk)  and  Vince Trencsenyi [vince.trencsenyi@rhul.ac.uk](mailto:vince.trencsenyi@rhul.ac.uk)

###### Abstract.

Game-theoretic simulations are a versatile tool for exploring interactions of both natural and artificial agents. However, modelling real-world scenarios and developing simulations often require substantial human expertise and effort. To streamline this process, we present a framework that enables the autoformalization of game-theoretic scenarios using agents augmented by large language models (LLMs). In this approach, LLM-augmented agents translate natural language scenario descriptions into executable logic programs that define the rules of each game, validating these programs for syntactic accuracy. A tournament simulation is then conducted, during which the agents test the functionality of the generated games by playing them. When a ground truth payoff matrix is available, an exact semantic validation can also be performed. The validated games can then be used in further simulations to assess the effectiveness of different strategies. We evaluate our approach on a diverse set of 55 natural language descriptions across five well-known $2\times 2$ simultaneous-move games, demonstrating 96% syntactic and 87% semantic correctness in the generated game rules. Additionally, we assess the LLM-augmented agents’ capability to autoformalize strategies for gameplay.

###### Key words and phrases:

autoformalization, game theory, large language models

## 1\. Introduction

Game-theoretic simulations are a versatile tool for exploring diverse phenomena, ranging from individual human interactions and organisational dynamics to biological processes, robotics, and interactions among artificial agents. However, modelling real-world scenarios and implementing corresponding simulations typically require substantial human expertise and effort. Consider, for instance, the problem of autonomous vehicles coordination: designing a simulation model involves specifying the rules of interaction, the objectives of each agent, the information available to them, and the strategies they may employ, all while accounting for uncertainties and external factors such as traffic laws and pedestrian behaviour; that model subsequently has to be implemented. Recent advancements in Large Language Models (LLMs) enable a new approach to streamline these tasks.

While LLMs have been evaluated as agents in game-theoretic contexts Akata et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib3)); Fan et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib7)); Guo ([2023](https://arxiv.org/html/2412.08805v1#bib.bib15)); Lorè and Heydari ([2023](https://arxiv.org/html/2412.08805v1#bib.bib21)), inherent limitations such as hallucinations Achiam et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib2)) and logical and arithmetical errors Imani et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib18)) restrict their utility as direct rational decision-makers. An alternative approach is to leverage LLMs’ capabilities in format translation, including the translation of natural language into formal representations. This process, known as autoformalization Wu et al. ([2022](https://arxiv.org/html/2412.08805v1#bib.bib39)), has shown success in creating formal representations for mathematics Wu et al. ([2022](https://arxiv.org/html/2412.08805v1#bib.bib39)); He-Yueya et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib17)); Jiang et al. ([2022](https://arxiv.org/html/2412.08805v1#bib.bib19)) and logic Yang et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib40)); Pan et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib29)); Cosler et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib6)); Chen et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib5)); Feng et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib9)). In this work, we apply autoformalization to automatically generate formal representations of game-theoretic scenarios.

This work builds on our previous research Mensfelt et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib26)), where we introduced a basic framework for autoformalizing natural language scenarios that can be modelled using $2\times 2$ simultaneous-move games. In the earlier work, a formal solver was used to validate the syntactic correctness of the generated formal game representations, while semantic correctness was evaluated manually.

In this work, we integrate the previously introduced autoformalization module into a modular agent model and propose a framework for simulating tournaments with these agents. This framework serves two main purposes. First, by running the tournament, we can automatically verify both the functionality of the generated code and its semantic correctness. This verification can be performed approximately, by comparing the agent’s total payoff with the target payoff, or exactly—when the ground truth payoff matrix is available—by checking whether each combination of players’ moves produces the intended payoff. Second, the framework’s modular agent design supports the autoformalization of strategies, enabling the exploration of how different strategies–whether autoformalized or predefined–perform in autoformalized game scenarios. This approach facilitates the evaluation of strategy efficiency across diverse game-theoretic contexts. The main contributions of this work are as follows:

*   •

    An agent model with formal game and strategy representation: We introduce an agent model that incorporates a formal representation of both games and strategies, along with an autoformalization module, developed in the previous work.

*   •

    Three-level agent validation: We provide a mechanism for validating agents for syntactic correctness (through formal solvers), functionality (by employing the code in gameplay), and semantic correctness (by comparing target and achieved payoffs).

*   •

    A simulation framework for autoformalized scenarios: A framework is introduced for simulating autoformalized game scenarios, enabling the investigation and comparison of different strategies – both autoformalized and predefined.

*   •

    Experimental evaluation: We evaluate the framework, demonstrating high syntactic and semantic correctness in the context of $2\times 2$ simultaneous-move games. We also test the capability of the agents to autoformalize 5 strategies.

The remainder of this paper is structured as follows. Sect. [2](https://arxiv.org/html/2412.08805v1#S2 "2\. Preliminaries ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") introduces the preliminaries, including foundational concepts in game theory, large language models, and general game playing. In Sect. [3](https://arxiv.org/html/2412.08805v1#S3 "3\. LLM-augmented Autoformalizing Agents ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") we provide an overview of the proposed framework, followed by Sect. [4](https://arxiv.org/html/2412.08805v1#S4 "4\. Methods ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents"), which describes the methods used to evaluate the framework. Sect. [5](https://arxiv.org/html/2412.08805v1#S5 "5\. Results and Discussion ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") presents and analyses the results. Sect. [6](https://arxiv.org/html/2412.08805v1#S6 "6\. Related Work ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") reviews related work in this area. Finally, Sect. [7](https://arxiv.org/html/2412.08805v1#S7 "7\. Conclusions and Future Work ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") summarizes the conclusions and presents directions for future work.

## 2\. Preliminaries

### 2.1\. Game Theory

Game theory is a versatile and popular mathematical modelling framework that captures interaction scenarios pf varying complexity across various disciplines. Rasmusen Rasmusen ([2006](https://arxiv.org/html/2412.08805v1#bib.bib31)) composes games using four main components:

*   •

    Players: players consistently and willingly act towards maximising their utility, in full awareness of the situation they are involved in;

*   •

    Actions: mappings between states and outcomes are actions carried out by the player Osborne and Rubinstein ([1994](https://arxiv.org/html/2412.08805v1#bib.bib28));

*   •

    Payoffs: payoffs are quantitative measures over the consequent outcomes of action tuples chosen by players Gibbons ([1992](https://arxiv.org/html/2412.08805v1#bib.bib13));

*   •

    Information: game information describes a player’s knowledge about the available actions, historical moves, and the player’s image of its opponents Osborne and Rubinstein ([1994](https://arxiv.org/html/2412.08805v1#bib.bib28)).

We formally define games in this context as the function of a set of $n$ players $N$, a non-empty set of actions $A_{i},\forall i\in N$, and for each player a utility function $u_{i}:A\rightarrow\mathbb{R}$ mapping a player i’s action to the corresponding payoff $\pi_{i}$.

| row/col | L | R |
| --- | --- | --- |
| U | ($W_{row}$, $W_{col}$) | ($X_{row}$, $X_{col}$) |
| D | ($Y_{row}$, $Y_{col}$) | ($Z_{row}$, $Z_{col}$) |

Table 1\. A general normal form game describing the four possible outcomes $W,X,Y,Z$ where the tuples denote the row player’s and column player’s corresponding payoffs, in general: $(\pi_{row},\pi_{col})$

In the case of symmetric games, the payoffs solely depend on the chosen strategies, while with asymmetric games, the outcomes yield different payoffs based on the player’s identity. In our experiments, the Prisoner’s Dilemma, Hawk-Dove, and Stag-Hunt games are symmetric, and we classify actions U and L as “Cooperate” denoted by $C$ and actions D and R as “Defect” denoted by $D$. This allows us to use the well-known terminology from Axelrod’s tournaments to define these games in terms of the four outcomes: $\mathbf{T}:(D,C),\mathbf{R}:(C,C),\mathbf{P}:(D,D),\mathbf{S}:(C,D)$.

*   •

    Prisoner’s Dilemma is characterised by a dominant temptation to defect, represented by the payoff relation $T>R>P>S$,

*   •

    Hawk-Dove entails switched relative payoffs between the punishment for mutual defection and the sucker’s payoff – $T>R>S>P$,

*   •

    Stag Hunt involves a relatively high reward for mutual cooperation over the temptation to defect – $R>T>P>S$.

The Battle of the Sexes and Matching Pennies are asymmetric games. As the outcomes mean different payoffs for row and column, we cannot use the generalization from above and have to define the games from both players’ perspectives:

*   •

    Battle of the Sexes is a coordination game, where both players prefer an agreement but have different preferences over the coordinated outcomes:

    *   –

        row: $W>Z>\{X,Y\}$,

    *   –

        column: $Z>W>\{X,Y\}$.

*   •

    Matching Pennies: coordinated outcomes reward the row player, while mismatched actions benefit the column player:

    *   –

        row: $\{W,Z\}>\{X,Y\}$,

    *   –

        column: $\{X,Y\}>\{W,Z\}$.

### 2.2\. Large Language Models

The rapid development of natural language processing (NLP) through the advancements of transformer architectures Gillioz et al. ([2020](https://arxiv.org/html/2412.08805v1#bib.bib14)) and pre-trained models Qiu et al. ([2020](https://arxiv.org/html/2412.08805v1#bib.bib30)) led to the emergence of Large Language Models. State-of-the-art (SOTA) LLMs are evolving at a swift rate, largely due to the progressive increase in model parameters and training dataset size Zhao et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib41)). While SOTA LLMs are improving with general use cases, there are approaches that further improve response accuracy and quality. Fine-tuning is the process of training LLMs on a specific custom dataset, increasing the model’s performance in the specific application context captured by the custom data Ziegler et al. ([2020](https://arxiv.org/html/2412.08805v1#bib.bib42)). Chain-of-thought prompting guides LLMs to break down their reasoning into simpler steps, often allowing models to deal with more complex requests or specific contexts more accurately Wei et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib38)).

OpenAI’s GPT-4 Omni OpenAI ([2024](https://arxiv.org/html/2412.08805v1#bib.bib27)), employed in this work, is a multi-modal SOTA model, that supports textual, visual and audio data as well. GPT4-o has been shown to perform exceptionally well on reasoning benchmarks Wang and Zhao ([2024](https://arxiv.org/html/2412.08805v1#bib.bib37)).

### 2.3\. General Game Playing

General game playing Genesereth et al. ([2005](https://arxiv.org/html/2412.08805v1#bib.bib11)) focuses on developing intelligent systems that explicitly represent the rules of arbitrary new games and learn to play them autonomously, without human intervention. The Game Description Language (GDL) has been proposed as a formal, machine-processable language for describing the rules of arbitrary games Love et al. ([2006](https://arxiv.org/html/2412.08805v1#bib.bib23)). GDL focused on information games only, so it was extended in GDL-II Thielscher ([2010](https://arxiv.org/html/2412.08805v1#bib.bib35)) to cover n-player games with incomplete information and games in extensive normal form Thielscher ([2011](https://arxiv.org/html/2412.08805v1#bib.bib36)). GDL-II is based on the standard syntax and semantics of logic programming and characterised by the special keywords shown in Table [2](https://arxiv.org/html/2412.08805v1#S2.T2 "Table 2 ‣ 2.3\. General Game Playing ‣ 2\. Preliminaries ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents").

| role(R) | R is a player |
| init(F) | F holds in the initial position |
| true(F) | F holds in the current position |
| legal(R, M) | R can do move M in the current position |
| does(R, M) | player R does move M |
| next(F) | F holds in the next position |
| terminal | the current position is terminal |
| goal(R, N) | R gets N points in the current position |

Table 2\. Subset of GDL-II (uppercase letters denote variables, lowercase letters denote predicates/function symbols)

A challenge with GDL systems is that learning without human guidance demands complex reasoning. Players must infer the possible actions of others, effectively analysing hypothetical game scenarios before making decisions. Action formalisms like the classical Situation Calculus McCarthy and Hayes ([1981](https://arxiv.org/html/2412.08805v1#bib.bib25)) have been developed for precisely this purpose. Formal schemes and inference methods are readily available for Situation Calculus Giacomo et al. ([2010](https://arxiv.org/html/2412.08805v1#bib.bib12)); Lesperance et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib20)), while their deployment in general game playing presupposes a translation from GDL into existing, suitably expressive action languages. One such scheme Schiffel and Thielscher ([2011](https://arxiv.org/html/2412.08805v1#bib.bib34)) shows how to fully embed GDL-II into a version of the Situation Calculus based on knowledge fluents Scherl and Levesque ([2003](https://arxiv.org/html/2412.08805v1#bib.bib32)). Our game solver, presented later in section [3.2](https://arxiv.org/html/2412.08805v1#S3.SS2 "3.2\. Solver ‣ 3\. LLM-augmented Autoformalizing Agents ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents"), is inspired by these previous works, to support light GDL specifications that act as target representation for the autoformalization.

## 3\. LLM-augmented Autoformalizing Agents

### 3.1\. Overview

![Refer to caption](img/4a43177eec0cb6508ea0664f7d8487f1.png)

Figure 1\. The overview of our agent model. Green dotted lines denote internal communication and solid black lines external communication.

The framework introduced in this work enables the creation of modular autoformalizing agents and supports tournament play using these agents. An overview of the agent model is presented in Fig. [1](https://arxiv.org/html/2412.08805v1#S3.F1 "Figure 1 ‣ 3.1\. Overview ‣ 3\. LLM-augmented Autoformalizing Agents ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents"). A logic programming component implemented in Prolog consists of two modules: a Game module containing game-independent and game-dependent rules, and a Strategy module describing which action to select against an opponent. Together, these modules form a solver (detailed in Section [3.2](https://arxiv.org/html/2412.08805v1#S3.SS2 "3.2\. Solver ‣ 3\. LLM-augmented Autoformalizing Agents ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents")) that encapsulates the game logic and determines the agent’s next move. A Python wrapper maintains the agent’s interaction with the external environment, manages the game history (tracking each agent’s moves, opponent moves, and payoffs), and updates the solver’s state. This modular design allows for easy substitution of game-dependent rules and strategies, either through predefined configurations or by autoformalizing natural language descriptions. Agents can also be saved to and reloaded from files. The source code and log files generated during the evaluation are available at¹¹1[https://github.com/dicelab-rhul/autoformalizing-agents](https://github.com/dicelab-rhul/autoformalizing-agents).

### 3.2\. Solver

Our solver is based on logic programming, as described in Mensfelt et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib26)), but repeated here for completeness. It comprises of a game-independent part representing the rules for any extensive-form game, a game-dependent part defining the rules of a particular game using the predicates from the game-independent part, and a set of auxiliary predicates completing the definition of that particular game. We follow standard Prolog conventions to interpret a game: variables are indicated by uppercase letters, while predicates and function symbols by lowercase. The symbol :- is interpreted as if, and the symbol \+ signifies not (negation by failure). An underscore ’_’ represents a variable whose value is ignored within a definition. The game state is then a situation, initially denoted by a constant (e.g. s0). The binary function do(M, S) denotes the situation resulting from the executing move M in situation S, consistent with the Situation Calculus.

#### 3.2.1\. Game-independent part

We define all legal transitions of an extended-form game from an initial situation S to a final situation F as follows:

[⬇](data:text/plain;base64,Z2FtZShGLEYpOi0KICAgIGZpbmFsKEYpLgpnYW1lKFMsRik6LQogICAgXCsgZmluYWwoUyksCiAgICBsZWdhbChNLFMpLAogICAgZ2FtZShkbyhNLFMpLEYpLg==)game(F,F):-final(F).game(S,F):-\+  final(S),legal(M,S),game(do(M,S),F).

The game terminates when the final game situation F is reached. Otherwise, in a non-final situation S, the game accepts a legal move M, and the game continues in the next do(M,S) situation, until the final situation F is reached. To determine what holds in each legal situation, we use Situation Calculus, represented as:

[⬇](data:text/plain;base64,aG9sZHMoRiwgUyk6LQogICAgaW5pdGlhbGx5KEYsIFMpLgpob2xkcyhGLCBkbyhNLCBTKSk6LQogICAgZWZmZWN0KEYsIE0sIFMpLgpob2xkcyhGLCBkbyhNLCBTKSk6LQogICAgaG9sZHMoRiwgUyksCiAgICBcKyBhYm5vcm1hbChGLCBNLCBTKS4=)holds(F,  S):-initially(F,  S).holds(F,  do(M,  S)):-effect(F,  M,  S).holds(F,  do(M,  S)):-holds(F,  S),\+  abnormal(F,  M,  S).

holds/2 here is similar to true/1 in GDL, but with an additional parameter of the situation in which the fluent is true. It states that a fluent F holds in the initial situation (init/1 in GDL), a new fluent F is initiated by the effects of a move M executed in a situation S (next/1 in GDL), and a fluent F persists after a move is made, provided it is not abnormal; abnormal fluents do not persist (implicit in GDL). We also use rules of the form:

[⬇](data:text/plain;base64,ZmluYWxseShGLCBTKTotIENvbmRpdGlvbnMu)finally(F,  S):-  Conditions.

to return derived fluents F describing the result of the game, when the Conditions hold in the final situation S.

#### 3.2.2\. Game-dependent part

To represent a specific game we need to define game-dependent predicates for the initial state initial/1, the legal moves legal/2, what holds in the initial game situation via initially/2, the effects of a move on a situation via effect/3, what stops persisting in a situation after the execution of a move via abnormal/3, the final situation final/1, and the result of the game via finally/2 definitions. To exemplify these definitions, we show how to describe a PD game to our solver. The initial situation s0 is defined as:

[⬇](data:text/plain;base64,aW5pdGlhbChzMCku)initial(s0).

What holds in this initial situation we specify it as:

[⬇](data:text/plain;base64,aW5pdGlhbGx5KHBsYXllcihwMSksIHMwKS4KaW5pdGlhbGx5KHBsYXllcihwMiksIHMwKS4KaW5pdGlhbGx5KHJvbGUocDEscm93KSwgczApLgppbml0aWFsbHkocm9sZShwMixjb2wpLCBzMCkuCmluaXRpYWxseShjb250cm9sKHAxKSwgczApLgppbml0aWFsbHkoY29udHJvbChwMiksIHMwKS4=)initially(player(p1),  s0).initially(player(p2),  s0).initially(role(p1,row),  s0).initially(role(p2,col),  s0).initially(control(p1),  s0).initially(control(p2),  s0).

These assertions define first the player names represented by unique identifiers (p1 and p2), their roles (p1 is the row player, while p2 is the column player), and then the fact that initially either of them can the game next (via the control/1 fluent,as in GDL). What holds in the initial situation changes as a result of move being made in the game. We represent moves as terms of the form move(P, M), where P is a player, and M is a move. As in PD it is possible for any player to defect (’D’) or cooperate (’C’), we express this as:

[⬇](data:text/plain;base64,cG9zc2libGUobW92ZShQLCdEJyksUyk6LWhvbGRzKHBsYXllcihQKSxTKS4KcG9zc2libGUobW92ZShQLCdDJyksUyk6LWhvbGRzKHBsYXllcihQKSxTKS4=)possible(move(P,’D’),S):-holds(player(P),S).possible(move(P,’C’),S):-holds(player(P),S).

It is then legal for a player to make a possible move if they have the control to execute it in the current situation:

[⬇](data:text/plain;base64,bGVnYWwobW92ZShQLE0pLCBTKTotCiAgICBwb3NzaWJsZShtb3ZlKFAsTSksIFMpLAogICAgaG9sZHMoY29udHJvbChQKSwgUyku)legal(move(P,M),  S):-possible(move(P,M),  S),holds(control(P),  S).

As an effect of a legal move M being made by a player P, we record in the next situation that the player did that move (effect/3 is like next/1 in GDL):

[⬇](data:text/plain;base64,ZWZmZWN0KGRpZChQLCBNKSwgbW92ZShQLCBNKSwgUyku)effect(did(P,  M),  move(P,  M),  S).

Once a legal move is executed, the player loses control, which we specify in our framework as:

[⬇](data:text/plain;base64,YWJub3JtYWwoY29udHJvbChQKSwgbW92ZShQLCBNKSwgUyku)abnormal(control(P),  move(P,  M),  S).

When a player loses control cannot play a move until control is passed back. When each player makes a move from the initial situation, the resulting final situation can be determined as:

[⬇](data:text/plain;base64,ZmluYWwoZG8oTTIsIGRvKE0xLCBTKSkpOi0KICAgIGdyb3VuZChNMiksCiAgICBncm91bmQoTTEpLAogICAgaW5pdGlhbChTKS4=)final(do(M2,  do(M1,  S))):-ground(M2),ground(M1),initial(S).

Assuming the payoff matrix following the structure presented in Table [1](https://arxiv.org/html/2412.08805v1#S2.T1 "Table 1 ‣ 2.1\. Game Theory ‣ 2\. Preliminaries ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") defined as:

[⬇](data:text/plain;base64,cGF5b2ZmKCdEJywgJ0QnLCAxLCAxKS4KcGF5b2ZmKCdDJywgJ0QnLCAwLCA1KS4KcGF5b2ZmKCdEJywgJ0MnLCA1LCAwKS4KcGF5b2ZmKCdDJywgJ0MnLCAzLCAzKS4=)payoff(’D’,  ’D’,  1,  1).payoff(’C’,  ’D’,  0,  5).payoff(’D’,  ’C’,  5,  0).payoff(’C’,  ’C’,  3,  3).

the outcome of the game holds information about the actual moves made by the players, and their payoffs.

[⬇](data:text/plain;base64,ZmluYWxseShvdXRjb21lKFAxLE0xLFUxLFAyLE0yLFUyKSwgUyk6LQogICAgaG9sZHMocm9sZShQMSwgcm93KSwgUyksCiAgICBob2xkcyhkaWQoUDEsIE0xKSwgUyksCiAgICBob2xkcyhyb2xlKFAyLCBjb2wpLCBTKSwKICAgIGhvbGRzKGRpZChQMiwgTTIpLCBTKSwKICAgIHBheW9mZihNMSwgTTIsIFUxLCBVMiku)finally(outcome(P1,M1,U1,P2,M2,U2),  S):-holds(role(P1,  row),  S),holds(did(P1,  M1),  S),holds(role(P2,  col),  S),holds(did(P2,  M2),  S),payoff(M1,  M2,  U1,  U2).

We can then extract specific outcome information, e.g. the player’s utility, through a goal/2 fluent (as in GDL) e.g.:

[⬇](data:text/plain;base64,ZmluYWxseShnb2FsKFAxLCBVMSksIFMpOi0KICAgIGZpbmFsbHkob3V0Y29tZShQMSxfLFUxLF8sXyxfKSwgUykuCmZpbmFsbHkoZ29hbChQMiwgVTIpLCBTKTotCiAgICBmaW5hbGx5KG91dGNvbWUoXyxfLF8sUDIsXyxVMiksIFMpLg==)finally(goal(P1,  U1),  S):-finally(outcome(P1,_,U1,_,_,_),  S).finally(goal(P2,  U2),  S):-finally(outcome(_,_,_,P2,_,U2),  S).

This finalises the definition of PD, enabling us to use the game description above to reason about the game’s dynamics. For instance, if player p1 wanted to analyse the game and identify the actions required to achieve a utility of 5, then it asks the query:

[⬇](data:text/plain;base64,Py0gZ2FtZShzMCxGKSwgZmluYWxseShnb2FsKHAxLDUpLEYpLg==)?-  game(s0,F),  finally(goal(p1,5),F).

Applying the PD payoff matrix, our solver produces two solutions for F, separated by semicolons (;). The result false indicates ”no further answers”:

[⬇](data:text/plain;base64,RiA9IGRvKG1vdmUocDIsJ0MnKSxkbyhtb3ZlKHAxLCdEJyksczApKTsKRiA9IGRvKG1vdmUocDEsJ0QnKSxkbyhtb3ZlKHAyLCdDJyksczApKTsKZmFsc2Uu)F  =  do(move(p2,’C’),do(move(p1,’D’),s0));F  =  do(move(p1,’D’),do(move(p2,’C’),s0));false.

In the first solution, p1 acts first and p2 second, while in the second solution, the order is reversed. This outcome is expected, as both players have initial control in our game definition, resulting in both action order combinations being shown.

#### 3.2.3\. Strategy

Strategies are submitted separately from the game-dependent component. In the current implementation, they may rely on specific predicates – default_move/2, opposite_move/2, possible/2, and finally/2 – which are required from the game-dependent solver. The select/4 predicate is used to determine the next move M of player P against opponent O in game situation S, as illustrated in the tit-for-tat strategy example below:

[⬇](data:text/plain;base64,c2VsZWN0KFAsIE8sIFMsIE0pOi0KICAgIFwrIGhvbGRzKGxhc3RfbW92ZShPLCBfTE1vKSwgUyksCiAgICBob2xkcyhkZWZhdWx0X21vdmUoUCwgTSksIFMpLgpzZWxlY3QoUCwgTywgUywgTW8pOi0KICAgIGhvbGRzKGxhc3RfbW92ZShPLCBNbyksIFMpLg==)select(P,  O,  S,  M):-\+  holds(last_move(O,  _LMo),  S),holds(default_move(P,  M),  S).select(P,  O,  S,  Mo):-holds(last_move(O,  Mo),  S).

Here, initially P selects the default move (typically to cooperate), otherwise it mirrors the opponent’s move in the last round. More sophisticated strategies can be expressed. For example, below we show how to express the best response of a player for a game as:

[⬇](data:text/plain;base64,c2VsZWN0KFAsIE8sIFMsIE0pOi0KICAgIFwrIGhvbGRzKGxhc3RfbW92ZShPLCBfKSwgUyksCiAgICBob2xkcyhkZWZhdWx0X21vdmUoUCwgTSksUykuCnNlbGVjdChQLCBPLCBTLCBNKTotCiAgICBob2xkcyhsYXN0X21vdmUoTywgTE1vKSwgUyksCiAgICBmaW5kYWxsKFVpLU1pLAogICAgIChnYW1lKFMsIEYpLAogICAgICBmaW5hbGx5KG91dGNvbWUoUCxNaSxVaSxPLE1vLFVvKSxGKSwKICAgICAgVWkgPj0gVW8pLAogICAgT3B0aW9ucyksCiAgICBzb3J0KDAsIEA+LCBPcHRpb25zLCBSYW5rZWQpLAogICAgaGlnaGVzdChSYW5rZWQsIE0pLg==)select(P,  O,  S,  M):-\+  holds(last_move(O,  _),  S),holds(default_move(P,  M),S).select(P,  O,  S,  M):-holds(last_move(O,  LMo),  S),findall(Ui-Mi,(game(S,  F),finally(outcome(P,Mi,Ui,O,Mo,Uo),F),Ui  >=  Uo),Options),sort(0,  @>,  Options,  Ranked),highest(Ranked,  M).

This second strategy differs from the previous one on tit-for-tat in that its second clause illustrates how to use our representation of games to perform complex reasoning with the game rules. This is done by ranking all legal future outcomes in a given situation according to he utility these have for the player and the opponent, and select the move that gives the highest utility for the player. This strategy also demonstrates the complexity of the autoformalization task.

#### 3.2.4\. Constraint checking

We check that the auto-formalized payoff matrix satisfies the structural and content constraints of the specified game (as outlined in Sect. [2.1](https://arxiv.org/html/2412.08805v1#S2.SS1 "2.1\. Game Theory ‣ 2\. Preliminaries ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents")), by defining validity constraints. The listing below specifies the payoff matrix validity constraints for the Prisoner’s Dilemma.

[⬇](data:text/plain;base64,ICAgIHZhbGlkX3BkX3BheW9mZnMoVCxSLFAsUyxDLEQpOi0KICAgICAgICAgICAgcGF5b2ZmKEMsQyxSLFIpLAogICAgICAgICAgICBwYXlvZmYoQyxELFMsVCksCiAgICAgICAgICAgIFQ+UiwKICAgICAgICAgICAgcGF5b2ZmKEQsQyxULFMpLAogICAgICAgICAgICBwYXlvZmYoRCxELFAsUCksCiAgICAgICAgICAgIFI+UCwKICAgICAgICAgICAgUD5TLg==)valid_pd_payoffs(T,R,P,S,C,D):-payoff(C,C,R,R),payoff(C,D,S,T),T>R,payoff(D,C,T,S),payoff(D,D,P,P),R>P,P>S.

Since the auto-formalization module does not impose constraints on the names of actions, this predicate also identifies which actions correspond to specific roles (e.g., determining that ‘coop’ = C).

### 3.3\. Autoformalization Module

Algorithm 1 Generating game rules predicates from natural language descriptions Mensfelt et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib26)). PD stands for Prisoner’s Dilemma.

1:Input:
$\Gamma$: game-independent predicates,
$NL_{PD}$: natural language description of PD,
$\xi_{PD}$: game-specific predicates of PD,
$NL_{NG}$: natural language description of a new game.2:Output:
$\xi_{NG}$: game-specific predicates for the new game.3:Parameter:
max_attempts: maximum correction attempts.4:$\text{attempts}\leftarrow 0$5:$\text{trace}\leftarrow\emptyset$6:while $\text{attempts}<\text{max\_attempts}$ do7:    $\xi_{NG}\leftarrow\text{LLM.translate}(\Gamma,NL_{PD},\xi_{PD},NL_{NG})$8:    $\text{is\_valid}\leftarrow\text{solver.check\_predicates}(\xi_{NG})$9:    if is_valid then10:         return  $\xi_{NG}$11:    else12:         $\text{trace}\leftarrow\text{solver.get\_trace}()$13:         $\xi_{NG}\leftarrow\text{LLM.self\_correct}(\xi_{NG},\text{trace})$14:    end if15:    $\text{attempts}\leftarrow\text{attempts}+1$16:end while17:return  Unable to generate valid predicates within maximum attempts.

To perform autoformalization, a Large Language Model is employed. The framework supports the use of any language model by providing an abstract interface. Autoformalization of a natural language description of a game or strategy is achieved through one-shot prompting, using game-dependent predicates for the Prisoner’s Dilemma and the tit-for-tat strategy as examples. After generating the code, its syntactic correctness is validated. If errors are detected, the LLM receives feedback, including the erroneous lines, error messages, and instructions for correcting the predicates. The max_attempts parameter specifies the maximum number of correction attempts. The overview of autoformalization algorithm is presented in Listing [1](https://arxiv.org/html/2412.08805v1#alg1 "Algorithm 1 ‣ 3.3\. Autoformalization Module ‣ 3\. LLM-augmented Autoformalizing Agents ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents").

### 3.4\. Simulation

In our previous work Mensfelt et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib26)), we manually validated the semantic correctness of generated programs. In this study, we introduce a tournament-based mechanism to automate the semantic correctness validation for scenarios and strategies where a target total payoff or a payoff matrix is available. In this approach, once a scenario is autoformalized, a copy of an agent is created and supplied with a specified strategy. The agent and its copy then play a specified number of rounds (clones mode). Comparing the agent’s total payoff to the target total payoff provides an approximate validation. When a ground truth payoff matrix is available, it enables exact semantic validation by verifying whether the payoff received for each unique combination of moves matches the target payoff for that combination. Additionally, we perform constraint checking to ensure that the payoff matrix satisfies the specified constraints for a specified game.

The tournament can also evaluate which strategy is most successful, on average, within a given game scenario across a specified number of rounds. This is achieved by supplying agent copies with different strategies and having them compete in a round-robin mode, where each agent plays against every other.

The main parameters of a tournament are as follows:

*   •

    game: can be provided by predefined rules or natural language description to autoformalize,

*   •

    strategies: can be provided as a list of predefined strategy rules or a list of natural language descriptions to autoformalize,

*   •

    mode: playing against a single clone or round-robin,

*   •

    agents number,

*   •

    rounds number.

There are two methods for determining the winners of the tournament:

1.  (1)

    Selecting agents which total payoff matches their target payoff – this is applicable if a target payoff is provided for each agent.

2.  (2)

    Selecting agents that achieved the highest total payoff.

Both methods were utilized in the evaluation of the framework, as detailed in the following sections.

## 4\. Methods

### 4.1\. Parameters

| Common parameters |
| --- |
| LLM | GPT-4o |
| temperature | $1$ |
| maximum output tokens | $1024$ |
| maximum attempts number | $5$ |

Table 3\. Common experimental parameters.

To evaluate the framework, we conducted three experiments: Experiment 1 tested the autoformalization of game scenarios, Experiment 2 employed the autoformalized game rules in tournament’s round-robin mode, and Experiment 3 examined the autoformalization of strategies. Table [3](https://arxiv.org/html/2412.08805v1#S4.T3 "Table 3 ‣ 4.1\. Parameters ‣ 4\. Methods ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") provides the experimental parameters shared across all experiments, while Table [4](https://arxiv.org/html/2412.08805v1#S4.T4 "Table 4 ‣ 4.1\. Parameters ‣ 4\. Methods ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") details the parameters unique to each specific experiment.

| Parameter | Exp. 1 | Exp. 2 | Exp. 3 |
| games | $55$ | $5$ | $1$ |
| agents per game | $5$ | $1$ | $5$ |
| strategies | 1 | $6$ | $5$ |
| rounds | $4$ | $10$ | $4$ |
| mode | clones | round-robin | clones |

Table 4\. Experiment-specific parameters.

### 4.2\. Exp. 1: Autoformalization of Game Descriptions

In the first experiment, we used a dataset of 55 natural language game-theoretic scenarios to evaluate the syntactic and semantic accuracy of logic programs generated by agents. The dataset included 5 scenarios that employed common metaphors (e.g., two prisoners in the Prisoner’s Dilemma) for the five canonical games introduced in Section [2.1](https://arxiv.org/html/2412.08805v1#S2.SS1 "2.1\. Game Theory ‣ 2\. Preliminaries ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents"). The remaining 50 scenarios – 10 for each game – used alternative descriptions that avoided these typical metaphors (e.g., two politicians competing in a campaign rather than two prisoners in the Prisoner’s Dilemma). For each scenario, five agents were generated with autoformalized game rules and then engaged in tournament, using the tit-for-tat strategy against a clone employing the anti-tit-for-tat strategy, testing all four possible pairs of moves in four rounds. Each agent had a maximum of five attempts to produce syntactically correct code; agents failing after five attempts were labelled as syntactically incorrect. Post-tournament, each agent’s individual and total payoffs were compared to the target, with matches indicating semantic correctness.

### 4.3\. Exp. 2: Axelrod’s Tournament

| Strategy | Description |
| --- | --- |
| anti-default-move | Always select the move that is the opposite of the default move. |
| anti-tit-for-tat | Start with a default move. Then, select the move that is the opposite of the opponent’s move in the previous round. |
| best-response | Start with a default move. Then, select a move that would give you the highest payoff in response to the opponent’s move in the previous round. |
| default-move | Always select the default move. |
| random | Select one of the possible moves with uniform probability. |
| tit-for-tat | Start with a default move. Then, mirror the opponent’s move in the previous round. |

Table 5\. Strategies utilised in the evaluation.

In the second experiment, we selected five agents created in Experiment 1, one for each of the five games. For each game, we conducted a tournament based on the design of Axelrod’s tournament Axelrod and Hamilton ([1981](https://arxiv.org/html/2412.08805v1#bib.bib4)). In this setup, six copies of each agent were created and assigned one of six strategies; each agent then played against every other agent, including itself. The strategies used are listed in Table [5](https://arxiv.org/html/2412.08805v1#S4.T5 "Table 5 ‣ 4.3\. Exp. 2: Axelrod’s Tournament ‣ 4\. Methods ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents").

To adapt the strategies to games without explicit cooperation and defection concepts, we defined a “default move” and its “opposite” as the basis for certain strategies. The default moves were: “Cooperate” for the Prisoner’s Dilemma, “Dove” for Hawk-Dove, “Stag” for Stag Hunt, “Opera” for the Battle of the Sexes, and “Heads” for Matching Pennies.

The main metric for evaluating strategy performance was the total payoff achieved by an agent using a given strategy in each game. To allow for cross-game comparisons of strategy effectiveness, total payoffs were normalized with the following formula:

|  | $norm\_payoff=\frac{total\_payoff-min\_total\_payoff}{{max\_total\_payoff-min\_% total\_payoff}},$ |  |

$Total\_payoff$ refers to the sum of the payoffs an agent collects across all rounds and $min\_total\_payoff$ and $max\_total\_payoff$ represent the minimum and maximum total payoffs achievable, respectively.

### 4.4\. Exp. 3: Autoformalization of Strategies

We then evaluated the semantic correctness of the autoformalized strategies. To achieve this, we used the natural language description of the tit-for-tat strategy, along with its Prolog implementation, as a reference example. For the other five strategies, the autoformalization module was provided with their natural language descriptions, similar to those shown in Table [5](https://arxiv.org/html/2412.08805v1#S4.T5 "Table 5 ‣ 4.3\. Exp. 2: Axelrod’s Tournament ‣ 4\. Methods ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents").

Each autoformalized strategy was then assigned to an agent with predefined Prisoner’s Dilemma game rules. This agent played four rounds against a clone utilizing the anti-tit-for-tat strategy. The agent’s total payoff was compared to a target payoff specific to each strategy to assess correctness. For the random strategy, correctness was also manually validated. This tournament setup was repeated five times for each strategy to ensure consistency in results.

## 5\. Results and Discussion

### 5.1\. Exp. 1: Autoformalization of Game Descriptions

| Attempts | 1 | 2 | 3 | 4 | 5 |
| --- | --- | --- | --- | --- | --- |
| Count | 93% | 7% | 0 | 0 | 12% |

Table 6\. Distribution of autoformalization attempts number for 275 agents.

Table [6](https://arxiv.org/html/2412.08805v1#S5.T6 "Table 6 ‣ 5.1\. Exp. 1: Autoformalization of Game Descriptions ‣ 5\. Results and Discussion ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") shows the distribution of attempts needed to generate syntactically correct code for the 275 agents created in the first experiment. For 93% of agents, syntactically correct code was generated on the first attempt. When the correct code was not generated initially, 7% of agents succeeded on the second attempt, while 12% could not produce correct code within the allowed attempts. This latter case highlights a need for further refinement of the feedback loop to improve success rates. However, the examination of the non-aggregated distribution of the invalid agents showed that an agent without syntactically valid game program was generated in no more than 1 in 5 instances.

![Refer to caption](img/a353eba25ff46b0f764c6323c917ff9a.png)

Figure 2\. The percentage of syntactic and semantic correctness of generated game rules for each game.

The overall syntactic correctness rate was 96%, while semantic correctness reached 87%, closely aligning with the results of our previous work (98% and 88%, respectively). Figure [2](https://arxiv.org/html/2412.08805v1#S5.F2 "Figure 2 ‣ 5.1\. Exp. 1: Autoformalization of Game Descriptions ‣ 5\. Results and Discussion ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") depicts the distribution of syntactic and semantic correctness across various games. The Battle of the Sexes and Stag Hunt emerged as the most error-prone scenarios, exhibiting the lowest semantic correctness rates, though still exceeding 80%.

The total-payoff-based approximate validation demonstrated 88% semantic correctness, while the exact individual-payoff-based validation achieved 87%. Five instances were incorrectly labelled as semantically correct by the approximate method, indicating that the results of the approximate and exact methods were closely aligned. A comparison of constraint checking and exact semantic validation revealed a few cases where the autoformalized payoff matrix satisfied the constraints, but errors in other parts of the program – such as role assignment – resulted in incorrect payoff assignments. This demonstrates that exact validation is feasible only when the ground truth payoff matrix is available.

### 5.2\. Exp. 2: Axelrod’s Tournament

![Refer to caption](img/14a2c4976dc1b0f5f81a1ef4f464d8eb.png)

Figure 3\. Normalized total payoff for each strategy by game.

Figure [3](https://arxiv.org/html/2412.08805v1#S5.F3 "Figure 3 ‣ 5.2\. Exp. 2: Axelrod’s Tournament ‣ 5\. Results and Discussion ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") presents the normalized total payoffs for each strategy across all games, while Table [7](https://arxiv.org/html/2412.08805v1#S5.T7 "Table 7 ‣ 5.2\. Exp. 2: Axelrod’s Tournament ‣ 5\. Results and Discussion ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") provides the average normalized total payoff for each strategy across games. Overall, the best-response strategy proved to be the most effective on average, followed closely by tit-for-tat. Best-response was most effective in all games except the Battle of the Sexes and the Prisoner’s Dilemma. In the Prisoner’s Dilemma, best-response achieved a lower payoff than the anti-default-move strategy (analogous to “defect”), as it initiated with the “cooperate” move. The results demonstrate that agents with autoformalized rules can effectively simulate and assess the performance of various strategies within different game-theoretic scenarios.

| Strategy | Payoff [%] |
| --- | --- |
| anti-default-move | 50.90 |
| anti-tit-for-tat | 55.22 |
| best-response | 84.08 |
| default-move | 73.96 |
| random | 61.08 |
| tit-for-tat | 79.84 |

Table 7\. Average normalized total payoffs for each strategy.

### 5.3\. Exp. 3: Autoformalization of Strategies

| Strategy | Correctness [%] |
| --- | --- |
| anti-default-move | 100.00 |
| anti-tit-for-tat | 100.00 |
| best-response | 40.00 |
| default-move | 100.00 |
| random | 60.00 |

Table 8\. Percentage of correctness for each strategy.

Table [8](https://arxiv.org/html/2412.08805v1#S5.T8 "Table 8 ‣ 5.3\. Exp. 3: Autoformalization of Strategies ‣ 5\. Results and Discussion ‣ Autoformalizing and Simulating Game-Theoretic Scenarios using LLM-augmented Agents") presents the percentage of correctly autoformalized strategies. The simpler strategies—default-move, anti-default-move, and anti-tit-for-tat—were always correctly autoformalized. In contrast, the random strategy achieved correct autoformalization in 60% of cases, and best-response, which requires selecting the move that maximizes payoff in response to the opponent’s previous move, was correctly autoformalized in 40% of cases. Since all strategies were generated using only the tit-for-tat implementation as an example, increasing the diversity of examples provided to the autoformalization module could potentially improve its accuracy in correctly formalising more complex strategies.

## 6\. Related Work

Originally proposed for agents in game scenarios, the Game Description Language (GDL) offers a promising approach for formalising complex strategic interactions in multi-agent environments by providing a declarative framework to define game rules, legal moves, and outcome conditions Genesereth et al. ([2005](https://arxiv.org/html/2412.08805v1#bib.bib11)). Agents that employ GDL are often used to systematically explore game states and strategic options, supporting rigorous analysis across both cooperative and competitive games, as suggested by previous approaches (e.g., see  Schiffel and Thielscher ([2009](https://arxiv.org/html/2412.08805v1#bib.bib33))). However, these approaches typically rely on human-driven formalisation of specific games, a limitation addressed by the LLM-augmented agents and the GDL representation used here, which enable autoformalization and enhance flexibility.

There is a growing body of research exploring the use of LLMs in game theory Lorè and Heydari ([2024](https://arxiv.org/html/2412.08805v1#bib.bib22)), systematically examining their potential as player agents in game-theoretic scenarios Fan et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib8)), utilising their inference abilities to make decisions in games with imperfect information Guo et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib16)), comparing their behaviour with human participants Fontana et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib10)), and deploying them in simulation frameworks Mao et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib24)). While we share with these works an interest in the broad capabilities and value of LLMs in game theoretic settings, our focus diverges by concentrating on how to use LLMs to generate GDL-like executable game specifications. These formal specifications enable agents to rigorously analyse game situations and reason about possible outcomes, providing a structured and robust foundation for decision-making within game-theoretic frameworks. Our work builds on previous research discussed in Mensfelt et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib26)), significantly extending it by embedding the autoformalization module within an agent model, incorporating the autoformalization of player strategies, implementing a distinct correctness validation process for the autoformalization through tournament play, and conducting experiments to validate the results.

Autoformalization Wu et al. ([2022](https://arxiv.org/html/2412.08805v1#bib.bib39)), has shown promising results in translating natural language into mathematical formalisms, enabling the integration of large language models with formal tools for solving reasoning tasksWu et al. ([2022](https://arxiv.org/html/2412.08805v1#bib.bib39)); He-Yueya et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib17)); Jiang et al. ([2022](https://arxiv.org/html/2412.08805v1#bib.bib19)). This approach has also been successfully extended to creating logical representations from natural language descriptions. For instance, Yang et al. Yang et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib40)) applied GPT-3 with few-shot prompting to translate natural language sentences into formal representations, which were then inputted into answer set programs. While effective on NLP benchmarks, this method faced occasional translation errors. To further enhance performance, Pan et al.Pan et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib29)) combined in-context learning with self-refinement based on solver evaluations, surpassing standard LLM and chain-of-thought prompting approaches on logical reasoning benchmarks. In the context of temporal logic, Cosler et al.Cosler et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib6)) addressed the problem of translation errors with an interactive system allowing users to refine sub-translations. Fine-tuning LLMs has also proven effective for boosting accuracy in translations to temporal logicsChen et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib5)) and deductive datasets Feng et al. ([2023](https://arxiv.org/html/2412.08805v1#bib.bib9)).

## 7\. Conclusions and Future Work

We have presented a framework for the autoformalization of game-theoretic scenarios, specifically those modelled by $2\times 2$ simultaneous-move games. The framework also enables autoformalization of strategies for these games and the simulation of tournaments based on the autoformalized rules. The generated programs undergo three levels of validation: syntactic validation by consulting a solver, functional testing through gameplay, and semantic validation – either approximate or exact – by comparing achieved payoffs with target outcomes. Evaluation of the framework demonstrated high syntactic and semantic accuracy of the code generated by agents across various interaction scenarios.

Future work will focus on extending the framework to model games beyond $2\times 2$ simultaneous-move scenarios. Our previous work Mensfelt et al. ([2024](https://arxiv.org/html/2412.08805v1#bib.bib26)) demonstrated that our sample solver for the Prisoner’s Dilemma could successfully serve to generate code for structurally similar games, such as rock-paper-scissors (with a different number of moves) and sequential versions of the Prisoner’s Dilemma. However, achieving successful generalization to a broader class of games will require assembling a more diverse set of in-context learning examples and developing methods to retrieve the most relevant examples for autoformalization in specific scenarios. The flexibility and generalisability of the introduced framework also make it applicable to games and game-like interactions beyond game theory.

Another direction for future work involves enhancing the feedback loop to more effectively leverage a large language model. At present, the LLM receives feedback solely from the trace generated when consulting the solver with the produced code. In the next phase, we plan to expand this feedback mechanism by incorporating runtime errors, thereby creating a more robust autoformalization module. Finally, in cases where unsupervised autoformalization proves infeasible for complex scenarios, we plan to introduce an interactive mode. This mode will allow users to iteratively refine the generated code alongside the autoformalization module. These interactively generated programs will then be added to our existing program library, expanding the resources available for in-context learning and enabling the autoformalization of increasingly complex scenarios.

{acks}

This work was supported by a Leverhulme Trust International Professorship Grant (LIP-2022-001). The first author would like to thank Bartosz Kosowski for the insightful discussions that contributed to the development of this research.

## References

*   (1)
*   Achiam et al. (2023) Josh Achiam, Steven Adler, Sandhini Agarwal, et al. 2023. GPT-4 Technical Report. *arXiv preprint arXiv:2303.08774* (2023).
*   Akata et al. (2023) Elif Akata, Lion Schulz, Julian Coda-Forno, et al. 2023. Playing repeated games with Large Language Models. *arXiv preprint arXiv:2305.16867* (2023).
*   Axelrod and Hamilton (1981) Robert Axelrod and William D Hamilton. 1981. The evolution of cooperation. *science* 211, 4489 (1981), 1390–1396.
*   Chen et al. (2023) Yongchao Chen, Rujul Gandhi, Yang Zhang, and Chuchu Fan. 2023. NL2TL: Transforming Natural Languages to Temporal Logics using Large Language Models. In *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing*. 15880–15903.
*   Cosler et al. (2023) Matthias Cosler, Christopher Hahn, Daniel Mendoza, Frederik Schmitt, and Caroline Trippel. 2023. nl2spec: interactively translating unstructured natural language to temporal logics with large language models. In *International Conference on Computer Aided Verification*. Springer, 383–396.
*   Fan et al. (2023) Caoyun Fan, Jindou Chen, Yaohui Jin, and Hao He. 2023. Can Large Language Models Serve as Rational Players in Game Theory? A Systematic Analysis. *arXiv preprint arXiv:2312.05488* (2023).
*   Fan et al. (2024) Caoyun Fan, Jindou Chen, Yaohui Jin, and Hao He. 2024. Can Large Language Models Serve as Rational Players in Game Theory? A Systematic Analysis. In *Thirty-Eighth AAAI Conference on Artificial Intelligence, AAAI 2024, February 20-27, 2024, Vancouver, Canada*. AAAI Press, 17960–17967. [https://doi.org/10.1609/AAAI.V38I16.29751](https://doi.org/10.1609/AAAI.V38I16.29751)
*   Feng et al. (2023) Jiazhan Feng, Ruochen Xu, Junheng Hao, Hiteshi Sharma, Yelong Shen, Dongyan Zhao, and Weizhu Chen. 2023. Language models can be logical solvers. *arXiv preprint arXiv:2311.06158* (2023).
*   Fontana et al. (2024) Nicoló Fontana, Francesco Pierri, and Luca Maria Aiello. 2024. Nicer Than Humans: How do Large Language Models Behave in the Prisoner’s Dilemma? arXiv:2406.13605 [cs.CY] [https://arxiv.org/abs/2406.13605](https://arxiv.org/abs/2406.13605)
*   Genesereth et al. (2005) Michael R. Genesereth, Nathaniel Love, and Barney Pell. 2005. General Game Playing: Overview of the AAAI Competition. *AI Mag.* 26, 2 (2005), 62–72.
*   Giacomo et al. (2010) Giuseppe De Giacomo, Yves Lespérance, and Adrian R. Pearce. 2010. Situation Calculus Based Programs for Representing and Reasoning about Game Structures. In *Principles of Knowledge Representation and Reasoning: Proceedings of the Twelfth International Conference, KR 2010, Toronto, Ontario, Canada, May 9-13, 2010*, Fangzhen Lin, Ulrike Sattler, and Miroslaw Truszczynski (Eds.). AAAI Press.
*   Gibbons (1992) R. Gibbons. 1992. *A Primer in Game Theory*. Pearson Education Limited, Harlow, Essex, United Kingdom.
*   Gillioz et al. (2020) Anthony Gillioz, Jacky Casas, Elena Mugellini, and Omar Abou Khaled. 2020. Overview of the Transformer-based Models for NLP Tasks. In *2020 15th Conference on Computer Science and Information Systems (FedCSIS)*. 179–183. [https://doi.org/10.15439/2020F20](https://doi.org/10.15439/2020F20)
*   Guo (2023) Fulin Guo. 2023. GPT agents in game theory experiments. *arXiv preprint arXiv:2305.05516* (2023).
*   Guo et al. (2023) Jiaxian Guo, Bo Yang, Paul Yoo, Bill Yuchen Lin, Yusuke Iwasawa, and Yutaka Matsuo. 2023. Suspicion-Agent: Playing Imperfect Information Games with Theory of Mind Aware GPT-4. *ArXiv* abs/2309.17277 (2023). [https://api.semanticscholar.org/CorpusID:263310339](https://api.semanticscholar.org/CorpusID:263310339)
*   He-Yueya et al. (2023) Joy He-Yueya, Gabriel Poesia, Rose E Wang, and Noah D Goodman. 2023. Solving math word problems by combining language models with symbolic solvers. *arXiv preprint arXiv:2304.09102* (2023).
*   Imani et al. (2023) Shima Imani, Liang Du, and Harsh Shrivastava. 2023. Mathprompter: Mathematical reasoning using large language models. *arXiv preprint arXiv:2303.05398* (2023).
*   Jiang et al. (2022) Albert Q Jiang, Sean Welleck, Jin Peng Zhou, Wenda Li, Jiacheng Liu, Mateja Jamnik, Timothée Lacroix, Yuhuai Wu, and Guillaume Lample. 2022. Draft, sketch, and prove: Guiding formal theorem provers with informal proofs. *arXiv preprint arXiv:2210.12283* (2022).
*   Lesperance et al. (2024) Yves Lesperance, Giuseppe De Giacomo, Maryam Rostamigiv, and Shakil M. Khan. 2024. Abstraction of Situation Calculus Concurrent Game Structures. *Proceedings of the AAAI Conference on Artificial Intelligence* 38, 9 (Mar. 2024), 10624–10634. [https://doi.org/10.1609/aaai.v38i9.28933](https://doi.org/10.1609/aaai.v38i9.28933)
*   Lorè and Heydari (2023) Nunzio Lorè and Babak Heydari. 2023. Strategic Behavior of Large Language Models: Game Structure vs. Contextual Framing. *arXiv preprint arXiv:2309.05898* (2023).
*   Lorè and Heydari (2024) Nunzio Lorè and Babak Heydari. 2024. Strategic behavior of large language models and the role of game structure versus contextual framing. *Scientific Reports* 14, 1 (2024), 18490. [https://doi.org/10.1038/s41598-024-69032-z](https://doi.org/10.1038/s41598-024-69032-z)
*   Love et al. (2006) Nathaniel Love, Timothy Hinrichs, David Haley, Eric Schkufza, and Michael Genesereth. 2006. *General Game Playing: Game Description Language Specification*. Technical Report LG–2006–01. Stanford University.
*   Mao et al. (2024) Shaoguang Mao, Yuzhe Cai, Yan Xia, Wenshan Wu, Xun Wang, Fengyi Wang, Tao Ge, and Furu Wei. 2024. ALYMPICS: LLM Agents Meet Game Theory – Exploring Strategic Decision-Making with AI Agents. arXiv:2311.03220 [cs.CL] [https://arxiv.org/abs/2311.03220](https://arxiv.org/abs/2311.03220)
*   McCarthy and Hayes (1981) J. McCarthy and P.J. Hayes. 1981. Some Philosophical Problems from the Standpoint of Artificial Intelligence. In *Readings in Artificial Intelligence*, Bonnie Lynn Webber and Nils J. Nilsson (Eds.). Morgan Kaufmann, 431–450.
*   Mensfelt et al. (2024) Agnieszka Mensfelt, Kostas Stathis, and Vince Tencsenyi. 2024. Autoformalization of Game Descriptions Using Large Language Models. In *1st International Workshop on Next-Generation Language Models for Knowledge Representation and Reasoning*. Hanoi, Vietnam. [https://arxiv.org/abs/2409.12300](https://arxiv.org/abs/2409.12300)
*   OpenAI (2024) OpenAI. 2024. GPT-4o. [https://openai.com/index/hello-gpt-4o/](https://openai.com/index/hello-gpt-4o/). [https://openai.com/index/hello-gpt-4o/](https://openai.com/index/hello-gpt-4o/) Accessed: 25/07/2024.
*   Osborne and Rubinstein (1994) M.J. Osborne and A. Rubinstein. 1994. *A Course in Game Theory*. The MIT Press.
*   Pan et al. (2023) Liangming Pan, Alon Albalak, Xinyi Wang, and William Wang. 2023. Logic-LM: Empowering Large Language Models with Symbolic Solvers for Faithful Logical Reasoning. In *Findings of the Association for Computational Linguistics: EMNLP 2023*. 3806–3824.
*   Qiu et al. (2020) Xipeng Qiu, Tianxiang Sun, Yige Xu, Yunfan Shao, Ning Dai, and Xuanjing Huang. 2020. Pre-trained models for natural language processing: A survey. *Science China Technological Sciences* 63 (2020), 1872–1897. [https://doi.org/10.1007/s11431-020-1647-3](https://doi.org/10.1007/s11431-020-1647-3)
*   Rasmusen (2006) E. Rasmusen. 2006. *Games and information an introduction to game theory*. Blackwell.
*   Scherl and Levesque (2003) Richard B. Scherl and Hector J. Levesque. 2003. Knowledge, action, and the frame problem. *Artif. Intell.* 144, 1-2 (2003), 1–39. [https://doi.org/10.1016/S0004-3702(02)00365-X](https://doi.org/10.1016/S0004-3702(02)00365-X)
*   Schiffel and Thielscher (2009) Stephan Schiffel and Michael Thielscher. 2009. Automated theorem proving for general game playing. In *Proceedings of the 21st International Joint Conference on Artificial Intelligence* *(IJCAI’09)*. Morgan Kaufmann Publishers Inc., San Francisco, CA, USA, 911–916.
*   Schiffel and Thielscher (2011) Stephan Schiffel and Michael Thielscher. 2011. Reasoning About General Games Described in GDL-II. *Proceedings of the AAAI Conference on Artificial Intelligence* 25, 1 (Aug. 2011), 846–851. [https://doi.org/10.1609/aaai.v25i1.7944](https://doi.org/10.1609/aaai.v25i1.7944)
*   Thielscher (2010) Michael Thielscher. 2010. A general game description language for incomplete information games. In *Proceedings of the Twenty-Fourth AAAI Conference on Artificial Intelligence* (Atlanta, Georgia) *(AAAI’10)*. AAAI Press, 994–999.
*   Thielscher (2011) Michael Thielscher. 2011. The General Game Playing Description Language Is Universal. In *IJCAI 2011, Proceedings of the 22nd International Joint Conference on Artificial Intelligence, Barcelona, Catalonia, Spain, July 16-22, 2011*, Toby Walsh (Ed.). IJCAI/AAAI, 1107–1112.
*   Wang and Zhao (2024) Yuqing Wang and Yun Zhao. 2024. RUPBench: Benchmarking Reasoning Under Perturbations for Robustness Evaluation in Large Language Models. *arXiv preprint arXiv:2406.11020* (2024).
*   Wei et al. (2023) Jason Wei, Xuezhi Wang, Dale Schuurmans, et al. 2023. Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. *arXiv preprint arXiv:2201.11903* (2023).
*   Wu et al. (2022) Yuhuai Wu, Albert Qiaochu Jiang, Wenda Li, Markus Rabe, Charles Staats, Mateja Jamnik, and Christian Szegedy. 2022. Autoformalization with large language models. *Advances in Neural Information Processing Systems* 35 (2022), 32353–32368.
*   Yang et al. (2023) Zhun Yang, Adam Ishay, and Joohyung Lee. 2023. Coupling large language models with logic programming for robust and general reasoning from text. *arXiv preprint arXiv:2307.07696* (2023).
*   Zhao et al. (2023) Wayne Xin Zhao, Kun Zhou, Junyi Li, et al. 2023. A Survey of Large Language Models. *arXiv preprint arXiv:2303.18223* (2023). arXiv:2303.18223 [cs.CL]
*   Ziegler et al. (2020) Daniel M. Ziegler, Nisan Stiennon, Jeffrey Wu, Tom B. Brown, et al. 2020. Fine-Tuning Language Models from Human Preferences. *arXiv preprint arXiv:1909.08593* (2020). arXiv:1909.08593 [cs.CL]