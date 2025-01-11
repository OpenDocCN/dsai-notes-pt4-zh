<!--yml
category: 未分类
date: 2025-01-11 12:02:35
-->

# Controlling LLM Agent with Finite State Graph

> 来源：[https://arxiv.org/html/2410.18528/](https://arxiv.org/html/2410.18528/)

zhiweiliu(April 2024)

## 1 Introduction

Firstly, we formulate the LLM agent behavior principle as a state graph, where node is state, and edge is transition between states. Each state node is a pre-defined agent execution and can be considered as a deterministic execution. The transition between states is then formulated as a decision making, from one state to another state. In this way, each state is connected to one or few possible next state. Hence, the agent execution is controlled as the state transition process.

Secondly, we design the transition module from three perspectives: 1) heuristics/rules, 2) machine learning (ML) models and 3) LLM reasoning. Heuristic-based transition is directly using some simple conditional rules like context matching to make a transition from one state to another. ML-based transition is to train a simple classification from one state to its neighbor states. LLM-based transition is to using the reasoning ability of LLM to decide the next states.

Lastly, the state graph is defined in three ways. 1) Hand-crafted state graph. The state and transition is fully designed by a human expert. 2) Data-based state graph. The state graph can be trained from large agent execution data. 3) LLM-based refinement. We could use LLM to add/delete the node in a state graph and we could also use LLM to add or delete the edges in the state graph.