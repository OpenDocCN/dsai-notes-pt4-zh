<!--yml
category: 未分类
date: 2025-01-11 12:13:48
-->

# AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting

> 来源：[https://arxiv.org/html/2409.14739/](https://arxiv.org/html/2409.14739/)

Chengjie Liu^($1,3$), Weiyu Chen^($1,3$), Anlan Peng^($1,3$), Yuan Du^($1$) Li Du^($1$),and Jun Yang^($2,3$) ^($1$)School of Electronic Science and Engineering, Nanjing University, Nanjing, China ^($2$)School of Integrated Circuit, South East University, Nanjing, China ^($3$)National Center of Technology Innovation for EDA, Nanjing, China cjliu_phd@smail.nju.edu.cn, wychen@smail.nju.edu.cn, alpeng@smail.nju.edu.cn yuandu@nju.edu.cn, ldu@nju.edu.cn, dragon@seu.edu.cn Corresponding Author: Li Du  Email: ldu@nju.edu.cn

###### Abstract

Multi-stage amplifiers are widely applied in analog circuits. However, their large number of components, complex transfer functions, and intricate pole-zero distributions necessitate extensive manpower for derivation and device parameters sizing to ensure their stability. In order to achieve efficient transfer function derivation and device parameters sizing, thereby simplifying the difficulty of amplifier design, we propose AmpAgent: a multi-agent system based on large language models (LLMs) for efficiently designing such complex amplifiers from literature with process and performance porting. AmpAgent is composed of three agents: Literature Analysis Agent, Mathematics Reasoning Agent and Device Sizing Agent. They are separately responsible for retrieving key information (e.g. formulas and transfer functions) from the literature, decompose the whole circuit’s design problem by deriving the key formulas, and address the decomposed problem iteratively.

AmpAgent was employed in the schematic design of seven types of multi-stage amplifiers with different compensation techniques. In terms of design efficiency, AmpAgent has reduced the number of iterations by 1.32$\sim$4${\times}$ and execution time by 1.19$\sim$2.99${\times}$ compared to conventional optimization algorithms, with a success rate increased by 1.03$\sim$6.79${\times}$. In terms of circuit performance, it has improved by 1.63$\sim$27.25${\times}$ compared to the original literature. The findings suggest that LLMs could play a crucial role in the field of complex analog circuit schematic design, as well as process and performance porting.

###### Index Terms:

Analog circuits, Computer-aided design, Electronic design automation, Large language model, Multi-stage amplifier, Optimization algorithm

## I Introduction

Multi-stage amplifier, compared with single-stage amplifier, can provide high open-loop gain and strong driving capability but with relative low supply voltage requirement, making it useful for amplifying small signals, driving LEDs, and processing biological signals [[1](https://arxiv.org/html/2409.14739v1#bib.bib1), [2](https://arxiv.org/html/2409.14739v1#bib.bib2), [3](https://arxiv.org/html/2409.14739v1#bib.bib3), [4](https://arxiv.org/html/2409.14739v1#bib.bib4), [5](https://arxiv.org/html/2409.14739v1#bib.bib5), [6](https://arxiv.org/html/2409.14739v1#bib.bib6), [7](https://arxiv.org/html/2409.14739v1#bib.bib7), [8](https://arxiv.org/html/2409.14739v1#bib.bib8)]. Designing multi-stage amplifiers, despite following a well-defined flow [[9](https://arxiv.org/html/2409.14739v1#bib.bib9)], remains challenging due to the large number of device components, complex transfer functions, and intricate pole-zero distributions compared to single-stage amplifiers. These factors not only make it challenging to manually design multi-stage amplifiers across various processes and performance metrics, but the vast search space also poses a significant challenge for optimization algorithms to design effectively.

The inefficiency in the automated design of multi-stage amplifiers lies in the inability to decompose their design problems into optimization problems at each stage. Recently, the emerging large language models (LLMs) have been proven capable of solving similar reasoning problems encountered in other fields[[10](https://arxiv.org/html/2409.14739v1#bib.bib10), [11](https://arxiv.org/html/2409.14739v1#bib.bib11)], even conducting a set of Nobel-Prize-winning chemistry experiments[[12](https://arxiv.org/html/2409.14739v1#bib.bib12)]. These demonstrate that LLMs also possess the potential to implement solutions for the automatic design of not only digital circuits [[13](https://arxiv.org/html/2409.14739v1#bib.bib13), [14](https://arxiv.org/html/2409.14739v1#bib.bib14), [15](https://arxiv.org/html/2409.14739v1#bib.bib15)] but also analog circuits like multi-stage amplifiers.

There has been some works [[16](https://arxiv.org/html/2409.14739v1#bib.bib16), [17](https://arxiv.org/html/2409.14739v1#bib.bib17), [18](https://arxiv.org/html/2409.14739v1#bib.bib18), [19](https://arxiv.org/html/2409.14739v1#bib.bib19)] that utilize LLMs to enable automated schematic design of analog circuits. The authors of [[16](https://arxiv.org/html/2409.14739v1#bib.bib16)] have developed an operational-amplifier-domain LLM Artisan to autonomously generate custom amplifier schematics. The researchers presented in [[17](https://arxiv.org/html/2409.14739v1#bib.bib17)] have developed a LLM-based agent, LADAC, and successfully automates the design process for three types of analog circuits. Another work [[18](https://arxiv.org/html/2409.14739v1#bib.bib18)] merged the LLM agent with Bayesian Optimization (BO) algorithm [[20](https://arxiv.org/html/2409.14739v1#bib.bib20)], to design a 2-stage amplifier and a hysteresis comparator with greater efficiency than that of BO. AnalogCoder [[19](https://arxiv.org/html/2409.14739v1#bib.bib19)] achieved designing analog circuits through Python code generation with greater capability than the state-of-the-art (SOTA) LLMs.

Despite the above work demonstrating the capability of LLMs in designing analog circuits effectively, there are still unresolved challenges in designing complex multi-stage amplifiers. For example, a Nested Gm-C Compensation amplifier[[1](https://arxiv.org/html/2409.14739v1#bib.bib1)], which is composed of 18 components, has 2 compensation caps and 2 feedforward stage to stabilize the amplifier which is hard for current LLMs to design due to the following reasons:

*   •

    The lack of analog circuits data results in even the SOTA LLMs lacking the relevant knowledge to provide valuable design recommendations for multi-stage amplifiers;

*   •

    Despite being provided with detailed amplifier information, LLMs’ ability to utilize this information for amplifier design is hindered by a lack of understanding of their design process.

To address the aforementioned issues, we developed AmpAgent, an LLM-based multi-agent system, for multi-stage amplifier schematic design from literature with process and performance porting instead of just replication[[21](https://arxiv.org/html/2409.14739v1#bib.bib21)]. AmpAgent incorporates the following three pipeline agents to address the two issues mentioned above:

*   •

    Literature Analysis Agent: Used for extracting essential design details on multi-stage amplifiers from literature, supplementing the current LLMs’ knowledge gaps and ensuring precise data for downstream applications.

*   •

    Mathematics Reasoning Agent: Responsible for deducing important formulas in conjunction with information such as the stability conditions of amplifiers. This decomposes the overall optimization problem of the amplifier into several sub-problems, thereby leveraging the results from upstream;

*   •

    Device Sizing Agent: Integrated with simulation interface and conventional optimization algorithms to size the multi-stage amplifier according to the various sub-problems divided by the previous agent efficiently.

Through the collaboration of the three pipeline agents, AmpAgent successfully achieved rapid schematic design of multi-stage amplifiers from literature. The primary contributions of this paper are:

*   •

    A novel domain-specified LLM multi-agent system is developed to realize the schematic design of multi-stage amplifier according to literature, incorporating process and performance porting.

*   •

    The performance of the AmpAgent-designed multi-stage amplifier has an enhancement of 27.25$\times$ compared to the literature’s;

*   •

    Compared to the conventional optimization algorithms, achieving a reduction of up to 4${\times}$ in the number of algorithm iterations, an execution time reduction of up to 2.99${\times}$, and an increase in success rate of up to 6.79${\times}$.

In summary, our work demonstrates the potential of LLMs in the analog circuits’ schematic design, as well as process and performance porting. This paper is organized as follows: Section II provides a detailed discussion on the current limitations of LLMs in analog circuits. Section III outlines the overall workflow of AmpAgent. Section IV exhibits the experiment of applying AmpAgent to the schematic design of seven different kinds of multi-stage amplifiers and AmpAgent’s design efficiency compared to conventional optimization algorithm. Finally, Section V concludes by summarizing our work.

## II LLM for analog circuits

In this section, we will demonstrate the necessity of constructing the multi-agent system within AmpAgent by showcasing the inadequacies of SOTA large models, including GPT-3.5, GPT-4[[22](https://arxiv.org/html/2409.14739v1#bib.bib22)], and GPT-4o[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)], in dealing with analog circuits.

The study presented in [[17](https://arxiv.org/html/2409.14739v1#bib.bib17)] explored the application of GPT-4 in generating SPICE netlists for a $3^{rd}$-order low-pass RC filter and a five-transistor amplifier. The results indicated that GPT-4 was unsuccessful in this task. The research in [[18](https://arxiv.org/html/2409.14739v1#bib.bib18)] integrated GPT-3.5 with BO to enhance the efficiency of analog circuit sizing. Despite this, reliance on GPT-3.5 alone led to a substantially higher number of iterations before finalizing a circuit design.

![Refer to caption](img/380611892ec4c6d9956bf256d40cd458.png)

Figure 1: Experiment on GPT-4o extracting netlist from circuit diagram

We further evaluated GPT-4o’s[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)] ability to extract the netlist from the circuit diagram, as shown in Fig.[1](https://arxiv.org/html/2409.14739v1#S2.F1 "Figure 1 ‣ II LLM for analog circuits ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"). Unfortunately, GPT-4o fails to accurately identify the components and their corresponding connections.

The experiment results of previous works [[17](https://arxiv.org/html/2409.14739v1#bib.bib17), [18](https://arxiv.org/html/2409.14739v1#bib.bib18)] and the ability of GPT-4o[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)] to recognize analog circuit diagrams will be presented. These experiments, which is illustrated in Table [I](https://arxiv.org/html/2409.14739v1#S2.T1 "TABLE I ‣ II LLM for analog circuits ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"), have shown that even the current SOTA LLMs lack sufficient knowledge about analog circuits, thereby also lacking the capability to accurately recognize circuits. Therefore, it is necessary to utilize a multi-agent system to establish division and collaboration in the design of analog circuits, thereby reducing the capability requirements for each agent and enhancing the performance of LLMs in designing circuits.

TABLE I: GPT-4’s and GPT-4o’s capacity for analog circuit design

| Model Name | Experiment | Result |
| GPT-4[[22](https://arxiv.org/html/2409.14739v1#bib.bib22)] | Spice generation[[17](https://arxiv.org/html/2409.14739v1#bib.bib17)] | ✗ |
| GPT-3.5 | Sizing Suggestion (Rough)[[18](https://arxiv.org/html/2409.14739v1#bib.bib18)] | ✔ |
| Sizing Suggestion (Precise)[[18](https://arxiv.org/html/2409.14739v1#bib.bib18)] | ✗ |
| GPT-4o[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)] | Circuit Diagram Recognition | ✗ |

![Refer to caption](img/eba8a29a8ec8fddc728764c12052d229.png)

Figure 2: The overview of AmpAgent

## III Overview of AmpAgent

AmpAgent is an LLM-based multi-agent system designed for designing multi-stage amplifier schematics from literature. The workflow of AmpAgent is shown in Fig. [2](https://arxiv.org/html/2409.14739v1#S2.F2 "Figure 2 ‣ II LLM for analog circuits ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"). AmpAgent comprises three agents: (1) Literature Analysis Agent, (2) Mathematics Reasoning Agent, and (3) Device Sizing Agent. All agents are developed based on ReAct [[24](https://arxiv.org/html/2409.14739v1#bib.bib24)] technique.

The Literature Analysis Agent is capable of retrieving and summarizing key information such as critical computational formulas and stability conditions. This is realized by embedding the under-designed literature as the vector database of the Literature Analysis Agent’s RAG.

The Mathematics Reasoning Agent, in conjunction with the user’s input of expected performance and the key formulas retrieved and summarized by the Literature Analysis Agent, carries out predictive calculations for the critical parameters of multi-stage amplifiers, such as the transconductance ($g_{m}$) of each stage. Thereby, it decomposes the overall design problem into multiple sub-problems.

The Device Sizing Agent interacts with the amplifier (such as sizing the devices and reading simulation results) and, in conjunction with conventional optimization algorithms, sequentially resolves the sub-problems.

In addition to the three agents mentioned above, the intermediate reasoning steps will be saved after confirming the correct functionality of the amplifier. This will allow for the direct retrieval of reasoning results when designing the same amplifier in the future, thereby reducing the consumption of LLMs tokens and improving efficiency.

### III-A Literature Analysis Agent

The Literature Analysis Agent is an LLM-based agent equipped with a Retrieval-Augmented Generation (RAG) module and meticulously crafted prompts. These two components work together to ensure that all critical information necessary for amplifier design can be accurately extracted from the under-designed literature, making it directly usable by downstream agents.

#### III-A1 Retrieval-Augmented Generation

We employ RAG[[25](https://arxiv.org/html/2409.14739v1#bib.bib25)] to address LLMs’ knowledge gaps in multi-stage amplifiers and mitigate the potential impact of hallucination [[26](https://arxiv.org/html/2409.14739v1#bib.bib26)]. To be specific, we first use the Mathpix[[27](https://arxiv.org/html/2409.14739v1#bib.bib27)] to convert the under-designed literature from PDF format to Markdown format, and accurately extract the formulas from the literature in latex format. Next, the converted document which contains the latex-formatted formulas will be embedded as the vector database of the RAG. These data form the basis of our RAG module.

Nevertheless, the direct deployment of RAG for circuit design still faces significant challenges in abstracting precise and accurate information. The key information LLM fails to reply despite equipped with the RAG mainly includes 2 parts:

*   •

    Circuit Schematic: The schematic provided in the literature as an image cannot be directly used in circuit design by LLMs and must first be converted into a netlist format for further parameter sizing. Due to the inaccuracy and inefficiency of existing tools [[28](https://arxiv.org/html/2409.14739v1#bib.bib28), [29](https://arxiv.org/html/2409.14739v1#bib.bib29)] in handling complex multi-stage amplifier circuits, we are currently converting the schematic images to netlist files manually.

*   •

    Formulas: The formulas (e.g. transfer function and stability conditions) of an amplifier are frequently dispersed across different sections of literature, which may lead to the loss of critical information and the redundancy of irrelevant information when directly using RAG, as shown in the black box in Fig.[3](https://arxiv.org/html/2409.14739v1#S3.F3 "Figure 3 ‣ III-A1 Retrieval-Augmented Generation ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting").

Therefore, relying solely on RAG to enhance the LLM’s extraction of crucial information about multi-stage amplifiers from literature is still insufficient. Hence, we have further customized a set of prompts to improve the LLM’s retrieval capabilities regarding relevant information.

![Refer to caption](img/553f4ae5746223dd5ba9ab980f0a671a.png)

Figure 3: The comparison between the RAG responses w/ or w/o prompts

#### III-A2 Prompts

To address issues of deficiency and redundancy in the retrieval of information by the RAG module for multi-stage amplifier design, we have developed specific prompts for the LLM. By directing the LLM towards targeted knowledge, these prompts enhance the LLM’s ability to retrieve the precise information needed for the design process and avoid unnecessary content, making the response succinct and clear.

Fig.[3](https://arxiv.org/html/2409.14739v1#S3.F3 "Figure 3 ‣ III-A1 Retrieval-Augmented Generation ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting") presents a comparative analysis of the effectiveness of providing prompts versus not providing prompts, focusing on the Nested Miller Compensation with a Nulling Resistor (NMCNR) amplifier as the experimental subject. Fig.[3](https://arxiv.org/html/2409.14739v1#S3.F3 "Figure 3 ‣ III-A1 Retrieval-Augmented Generation ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting") demonstrates the impact of utilizing prompts on the retrieval process, highlighting the improved accuracy and efficiency achieved when the LLM is guided by specific prompts.

In summary, we integrate the RAG technique with the LLM Agent to efficiently retrieve critical information essential for the multi-stage amplifier design process. By streamlining retrieval and focusing on key data, this approach significantly enhances the efficiency and accuracy of the information retrieval system. However, the retrieved raw information, such as the transfer function, still requires further processing, which necessitates a Mathematics Reasoning Agent to address it.

![Refer to caption](img/35726e2d192a62e887f608b00d988ce9.png)

Figure 4: The response of the Mathematics Reasoning Agent.

### III-B Mathematics Reasoning Agent

Much of the necessary information for the design process has to be derived from the original amplifier’s transfer function or calculated through pole-zero distribution methods. In this part, we will illustrate the Mathematics Reasoning Agent dedicated for essential design information derivation.

The module necessitates the sequential implementation of two tasks: (1) the computation of poles and zeros definition based on the transfer function, (2) and the calculation of the targets for the sub-design tasks, taking into account the expected performance metrics, derived poles and zeros’ definitions, based on the complex-pole method[[30](https://arxiv.org/html/2409.14739v1#bib.bib30)]. For efficient computation, the module is specifically designed to generate Python code to represent the equations that need to be solved and then perform the necessary calculations.

In terms of mathematical reasoning, for tasks with fixed computational paradigms such as calculating pole-zero distribution, we employ a few-shots prompt to convey the rules of the complex-pole method [[30](https://arxiv.org/html/2409.14739v1#bib.bib30)] to the Mathematics Reasoning Agent, leveraging its reasoning capabilities to achieve results. For more complex computations, such as solving systems of equations, we instruct the Mathematics Reasoning Agent to utilize the Python SymPy library to ensure accuracy and efficiency. Here, we give an example of the design derivation of the Nested Gm-C Compensation (NGCC) amplifier [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)], when the expected $GBW$ is 10MHz and $C_{L}$ is 10pF, and the responses of the Mathematics Reasoning Agent is shown in Fig.[4](https://arxiv.org/html/2409.14739v1#S3.F4 "Figure 4 ‣ III-A2 Prompts ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting").

As depicted in Fig.[4](https://arxiv.org/html/2409.14739v1#S3.F4 "Figure 4 ‣ III-A2 Prompts ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"), the agent has effectively derived the poles and zeros of the amplifier and accurately computed the final $g_{m}$ for each stage. This demonstrates the capability of the agent, which is not only able to analyze the design of multi-stage amplifiers but can also derive the $g_{m}$ and capacitance values when varying input performance metrics.

### III-C Device Sizing Agent

After determining the theoretical design targets for the sub-tasks by the afore agents, specifically the $g_{m}$ for each stage in the case of an multi-stage amplifier, the Device Sizing Agent is employed. This module is responsible for sizing the transistors and adjusting the device values to attain the desired sub-tasks design targets. The overview of the Device Sizing Agentis shown in Fig.[5](https://arxiv.org/html/2409.14739v1#S3.F5 "Figure 5 ‣ III-C Device Sizing Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting").

![Refer to caption](img/9e0e1a33408fc24d97ca6fe4796ae949.png)

Figure 5: The overview of the Device Sizing Agent

The Device Sizing Agent performs three actions: simulation result reading, accessing calculator, and invoking the conventional optimization algorithms. These actions enable the Device Sizing Agent to interact with the simulator to retrieve current device performance metrics (such as the $g_{m}$ of transistors, the Gain-Bandwidth product ($GBW$) and Phase Margin ($PM$) of amplifiers, etc.), use the calculator for the estimation of expected device parameters (such as estimate the transistor $W/L$ according to the current $g_{m}$ and the expected $g_{m}$), and ultimately use these parameters as initial values for optimization using conventional algorithms such as Artificial Bee Colony (ABC) [[31](https://arxiv.org/html/2409.14739v1#bib.bib31)] and Trust Region Bayesian Optimization (TuRBO) [[20](https://arxiv.org/html/2409.14739v1#bib.bib20)] algorithm. If the results of the conventional optimization algorithm do not meet the expected metrics, the current device parameters and metrics will be returned to the Device Sizing Agent for it to further estimate the initial values for another loop of optimization.

The process mentioned above will be iteratively repeated until the Device Sizing Agent has optimized all sub-problems. After all sub-problems have been optimized, current device parameters will be used as initial values for overall optimization by the conventional optimization algorithm.

By combining agent-based sizing of the circuit with conventional optimization algorithms, we mitigate the slow iteration rate of large language models (LLMs) and reduce the extensive search space of circuit parameters for the optimization algorithm. This hybrid approach leverages the strengths of both methods, resulting in more efficient and cost-effective circuit design. The comparative experiments assessing whether to employ optimization algorithms within the sub-problems, as well as the decision to use them in the overall final process, will be shown in Section IV.

## IV Experiment

In this section, we present the experimental results of seven different types of AmpAgent-designed multi-stage amplifiers. We also compare the performance of AmpAgent with and without conventional optimization algorithms. Furthermore, we evaluate the Device Sizing Agent against conventional optimization algorithms, focusing on the number of algorithm iterations, time consumption, and success rate during parameter sizing. As an example, we will detail the design process for a three-stage Nested Miller Compensation with a Nulling Resistor (NMCNR) amplifier [[2](https://arxiv.org/html/2409.14739v1#bib.bib2)].

All experiments were conducted on a Linux workstation with one 12th Gen Intel^® Core${}^{\text{TM}}$ i7-12700 CPU and 64-GB memory. The amplifier’s simulations were based on Cadence Spectre^® simulator. The entire AmpAgent framework is developed based on LangChain [[32](https://arxiv.org/html/2409.14739v1#bib.bib32)]. The LLM we choose is GPT-4-1106-preview and its temperature is set as 0\. All the AmpAgent-designed amplifiers have been verified through simulation to ensure that no right-half-plane poles are present and that there are no stability issues.

### IV-A NMCNR amplifier

In this part, we employed AmpAgent in the schematic design of an NMCNR amplifier on the 0.18$\mu m$ process. NMCNR amplifiers [[2](https://arxiv.org/html/2409.14739v1#bib.bib2)] are a kind of 3-stage amplifier with high open-loop gain and a nulling resistor to compensate the right-half-plane zero for a more stable structure [[5](https://arxiv.org/html/2409.14739v1#bib.bib5)].

The literature we selected as the foundation for the AmpAgent to design the NMCMR amplifier is [[2](https://arxiv.org/html/2409.14739v1#bib.bib2)]. The performance metrics we aimed to achieve by AmpAgent include: $C_{L}=50pF$, $GBW\geq{5MHz}$, $PM\geq{60^{\circ}}$, $Gain_{DC}\geq{120dB}$, and the power consumption should be as low as possible. The outputs of the Literature Analysis Agent, Mathematics Reasoning Agent are shown in Fig.[6](https://arxiv.org/html/2409.14739v1#S4.F6 "Figure 6 ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"). The results are shown in Table [II](https://arxiv.org/html/2409.14739v1#S4.T2 "TABLE II ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting").

TABLE II: AmpAgent-designed NMCNR, SMC, NGCC, DFCFC, TCFC, IAC, and AZC amplifiers’ performance

 | Amplifier | Expected Value | w/o further optimization | w/ further optimization |
| $GBW$ $MHz$ | $PM$ ^∘ | $Gain_{DC}$ ${dB}$ | $GBW$ $MHz$ | $PM$ ^∘ | $Gain_{DC}$ ${dB}$ | $I_{dd}$ ${mA}$ | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ | $GBW$ $MHz$ | $PM$ ^∘ | $Gain_{DC}$ ${dB}$ | $I_{dd}$ ${mA}$ | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ |
| NMCNR [[2](https://arxiv.org/html/2409.14739v1#bib.bib2)] @$C_{L}=50pF$ | $\geq 5.00$ | $\geq 60$ | $\geq 120$ | $5.15$ | $32.5$ | $135$ | $0.38$ | 675 | $5.01$ | $61.5$ | $131$ | $0.322$ | 778 |
| SMC [[3](https://arxiv.org/html/2409.14739v1#bib.bib3)] @$C_{L}=10pF$ | $\geq 10.00$ | $\geq 60$ | $\geq 70$ | $9.47$ | $60.05$ | $73.53$ | $0.087$ | 1092 | $10.0$ | $60.7$ | $70.06$ | $0.080$ | 1243 |
| NGCC [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)] @$C_{L}=50pF$ | $\geq 1.00$ | $\geq 60$ | $\geq 100$ | $1.24$ | $52$ | $137$ | $0.078$ | 795 | $1.06$ | $63$ | $136$ | $0.054$ | 981 |
| DFCFC [[4](https://arxiv.org/html/2409.14739v1#bib.bib4)] @$C_{L}=150pF$ | $\geq 2.00$ | $\geq 60$ | $\geq 100$ | $2.00$ | $1.2$ | $119$ | $0.035$ | 8645 | $2.12$ | $60$ | $100$ | $0.052$ | 6103 |
| TCFC [[6](https://arxiv.org/html/2409.14739v1#bib.bib6)] @$C_{L}=300pF$ | $\geq 2.00$ | $\geq 60$ | $\geq 100$ | $2.05$ | $77$ | $127$ | $0.053$ | 11647 | $2.28$ | $71$ | $130$ | $0.017$ | 41204 |
| IAC [[7](https://arxiv.org/html/2409.14739v1#bib.bib7)] @$C_{L}=500pF$ | $\geq 4.00$ | $\geq 60$ | $\geq 100$ | $3.55$ | $57$ | $153$ | $0.053$ | 33490 | $4.87$ | $76$ | $150$ | $0.039$ | 62435 |
| AZC [[8](https://arxiv.org/html/2409.14739v1#bib.bib8)] @$C_{L}=15nF$ | $\geq 1.00$ | $\geq 60$ | $\geq 100$ | $0.96$ | $58$ | $135$ | $0.049$ | 195918 | $1.01$ | $61.0$ | $133$ | $0.047$ | 322340 | 

TABLE III: AmpAgent-designed Amplifiers’ and Original literature’s $IFOM_{S}$ comparison

| Amplifier Name | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ Origin literature | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ By AmpAgent | Improvements |
| SMC[[3](https://arxiv.org/html/2409.14739v1#bib.bib3)] | 400 | 1243 | 3.11$\times$ |
| NMCNR[[2](https://arxiv.org/html/2409.14739v1#bib.bib2)] | 410 | 778 | 1.90$\times$ |
| NGCC[[1](https://arxiv.org/html/2409.14739v1#bib.bib1)] | 36 | 981 | $27.25\times$ |
| DFCFC[[4](https://arxiv.org/html/2409.14739v1#bib.bib4)] | 1238 | 6103 | 4.93$\times$ |
| TCFC[[6](https://arxiv.org/html/2409.14739v1#bib.bib6)] | 14250 | 41204 | 2.89$\times$ |
| IAC[[7](https://arxiv.org/html/2409.14739v1#bib.bib7)] | 33000 | 62435 | 1.89$\times$ |
| AZC[[8](https://arxiv.org/html/2409.14739v1#bib.bib8)] | 197916 | 322340 | $1.63\times$ |

![Refer to caption](img/1a46daad3678b8e533cdddc915b5ce36.png)

Figure 6: AmpAgent designing a 3-stage NMCNR amplifier with $C_{L}=50pF$, $GBW=5MHz$

### IV-B Other multi-stage amplifiers

Except for the amplifiers mentioned above, we also test AmpAgent’s capacity in design more types of multi-stage amplifiers. In this part, we will present the design results of AmpAgent for several amplifiers with typical compensation structures, including Single Miller Compensation (SMC) [[3](https://arxiv.org/html/2409.14739v1#bib.bib3)], NGCC [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)], Damping-Factor-Control Frequency-Compensation (DFCFC) [[4](https://arxiv.org/html/2409.14739v1#bib.bib4)], Transconductance with Capacitor Feedback Compensation (TCFC) [[6](https://arxiv.org/html/2409.14739v1#bib.bib6)], Impedance Adapting Compensation (IAC) [[7](https://arxiv.org/html/2409.14739v1#bib.bib7)], and Active Zero Compensation (AZC) amplifiers [[8](https://arxiv.org/html/2409.14739v1#bib.bib8)] across different requirements for $GBW$ and $C_{L}$.

Additionally, we compared the amplifiers’ performance with and without the implementation of overall optimization following the Device Sizing Agent. The experiment results are shown in Table [II](https://arxiv.org/html/2409.14739v1#S4.T2 "TABLE II ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"). From Table [II](https://arxiv.org/html/2409.14739v1#S4.T2 "TABLE II ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"), we can observe that incorporating overall optimization allows the amplifiers to meet the constraints, and further enhances their performance. We utilize the $IFOM_{S}$, which is defined in ([1](https://arxiv.org/html/2409.14739v1#S4.E1 "In IV-B Other multi-stage amplifiers ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")), to evaluate the amplifier performance. $IFOM_{S}$ is a common metric in literature [[7](https://arxiv.org/html/2409.14739v1#bib.bib7), [6](https://arxiv.org/html/2409.14739v1#bib.bib6), [8](https://arxiv.org/html/2409.14739v1#bib.bib8)].

|  | $IFOM_{S}=GBW*C_{L}/I_{dd}$ |  | (1) |

The improvement in various amplifiers, compared to the original literature’s $IFOM_{S}$, are also listed in Table [III](https://arxiv.org/html/2409.14739v1#S4.T3 "TABLE III ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"). The performance for these amplifiers have seen an enhancement ranging from 1.63${\times}$ to 27.25${\times}$ compared to original paper.

TABLE IV: Comparison between This Work, ABC, and TuRBO in terms of iteration number, time consumption, and success rate

| Amplifier | This Work | ABC [[31](https://arxiv.org/html/2409.14739v1#bib.bib31)] | TuRBO-5 [[20](https://arxiv.org/html/2409.14739v1#bib.bib20)] | TuRBO-1 [[20](https://arxiv.org/html/2409.14739v1#bib.bib20)] |
| Avg. # Iter. | Avg. Time (s) | Success rate | Avg. # Iter. | Avg. Time (s) | Success rate | Avg. # Iter. | Avg. Time (s) | Success rate | Avg. # Iter. | Avg. Time (s) | Success rate |
| SMC [[3](https://arxiv.org/html/2409.14739v1#bib.bib3)] | 16 | 245 | 100% | 35 | 456 | 68% | 38 | 460 | 95% | 56 | 540 | 76% |
| NMCNR [[5](https://arxiv.org/html/2409.14739v1#bib.bib5)] | 19 | 444 | 100% | 75 | 600 | 46% | 53 | 842 | 84% | 52 | 597 | 82% |
| NGCC [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)] | 22 | 458 | 98% | 75 | 543 | 46% | 29 | 1200 | 95% | 35 | 843 | 94% |
| DFCFC [[4](https://arxiv.org/html/2409.14739v1#bib.bib4)] | 33 | 603 | 95% | 93 | 783 | 14% | 53 | 1321 | 83% | 46 | 836 | 88% |
| TCFC [[6](https://arxiv.org/html/2409.14739v1#bib.bib6)] | 26 | 471 | 96% | 75 | 601 | 54% | 36 | 894 | 89% | 48 | 768 | 85% |
| IAC [[7](https://arxiv.org/html/2409.14739v1#bib.bib7)] | 25 | 481 | 97% | 100 | 1352 | (Failed) | 61 | 976 | 73% | 64 | 742 | 82% |
| AZC [[8](https://arxiv.org/html/2409.14739v1#bib.bib8)] | 38 | 696 | 96% | 100 | 1465 | (Failed) | 100 | 2083 | (Failed) | 100 | 1610 | (Failed) |

### IV-C Integration with conventional optimization algorithms

In addition to using optimization algorithm for the overall optimization of the amplifier, AmpAgent can also employ the algorithm to assist in sizing the transistors during the resolution of sub-problems, as discussed in Section III.C.

In this part, we will compare whether AmpAgent uses an optimization algorithm, specifically ABC, during the optimization process, and we will look at the differences in terms of LLM token consumption and design time.

![Refer to caption](img/c6d5e842d78a42c5a8ad72e117a1a611.png)

Figure 7: Design time and token consumption comparison

Taking the automatic design of the NGCC [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)] amplifier by AmpAgent as an example, we randomly sample the expected values of $GBW$ and $C_{L}$ within the range of $1MHz$ to $10MHz$ and $10pF$ to $100pF$. Comparisons of time consumption and token consumption by the LLM, here GPT4, are shown in Fig. [7](https://arxiv.org/html/2409.14739v1#S4.F7 "Figure 7 ‣ IV-C Integration with conventional optimization algorithms ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"). Fig. [7](https://arxiv.org/html/2409.14739v1#S4.F7 "Figure 7 ‣ IV-C Integration with conventional optimization algorithms ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting") shows that AmpAgent, by adopting optimization algorithm to tackle sub-optimization issues, has reduced the time required by 1.56${\times}$ and improved token consumption efficiency by 1.52${\times}$ compared to not using it. From this comparison, it can be observed that AmpAgent is better suited for decomposing sub-optimization problems and providing initial solutions for these sub-problems, rather than directly performing iterative optimization on parameters itself.

### IV-D Optimization Efficiency

Furthermore, we conducted a comparative analysis between the Device Sizing Agent and the conventional optimization algorithms in terms of efficiency in parameter sizing. This comparison, which is shown in Table [IV](https://arxiv.org/html/2409.14739v1#S4.T4 "TABLE IV ‣ IV-B Other multi-stage amplifiers ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"), focused on the number of iterations, design time, and success rate. We set the maximum number of iterations to 100\. The results convincingly demonstrate that AmpAgent can significantly enhance the accuracy of automated design processes while effectively reducing the number of iterations and the associated design time. It is noteworthy that the instances of design failure associated with AmpAgent can be attributed to the occasional hallucinatory effects of GPT-4, which lead to its inability to execute certain actions correctly.

## V Conclusion

In this study, we presented AmpAgent, which is a novel approach utilizing an LLM-based multi-agent dedicated to multi-stage schematic design from literature with process and performance porting. AmpAgent consists of three agents, and through a detailed division of design problem, it mitigates the current issue of LLMs lacking in specialized knowledge and design process understanding for analog circuits.

We specifically showcase AmpAgent’s effectiveness in auto design of seven different types of multi-stage amplifiers with process and performance porting. The performance for these amplifiers has seen an enhancement ranging from 1.63${\times}$ to 27.25${\times}$ compared to the original literature, confirming AmpAgent’s capability of such complex analog circuits. The design efficiency of AmpAgent surpasses that of conventional optimization algorithms, achieving a significant reduction in iteration counts by 1.32$\sim$4${\times}$ and execution time by 1.19$\sim$2.99${\times}$. It also boasts an enhanced success rate by 1.03$\sim$6.79${\times}$ and it can even successfully design amplifiers where conventional optimization algorithms fail.

To further enhance AmpAgent, future work could involve training an analog-circuit-domain LLM, automation of netlist extraction from circuit diagram and integration of advanced circuit design methodologies like $g_{m}/I_{d}$[[33](https://arxiv.org/html/2409.14739v1#bib.bib33)] for higher efficiency.

## References

*   [1] F. You, S. H. Embabi, and E. Sanchez-Sinencio, “Multistage amplifier topologies with nested g/sub m/-c compensation,” *IEEE Journal of Solid-State Circuits*, vol. 32, no. 12, pp. 2000–2011, 1997.
*   [2] K. N. Leung and P. Mok, “Analysis of multistage amplifier-frequency compensation,” *IEEE Transactions on Circuits and Systems I: Fundamental Theory and Applications*, vol. 48, no. 9, pp. 1041–1056, 2001.
*   [3] P. E. Allen and D. R. Holberg, *CMOS analog circuit design*.   Elsevier, 2011.
*   [4] A. K. N. Leung, P. K. Mok, W. H. Ki, and J. K. Sin, “Damping-factor-control frequency compensation technique for low-voltage low-power large capacitive load applications,” in *1999 IEEE International Solid-State Circuits Conference. Digest of Technical Papers. ISSCC. First Edition (Cat. No. 99CH36278)*.   IEEE, 1999, pp. 158–159.
*   [5] K. N. Leung, P. K. Mok, and W.-H. Ki, “Right-half-plane zero removal technique for low-voltage low-power nested miller compensation cmos amplifier,” in *ICECS’99\. Proceedings of ICECS’99\. 6th IEEE International Conference on Electronics, Circuits and Systems (Cat. No. 99EX357)*, vol. 2.   IEEE, 1999, pp. 599–602.
*   [6] X. Peng and W. Sansen, “Transconductance with capacitances feedback compensation for multistage amplifiers,” *IEEE Journal of Solid-State Circuits*, vol. 40, no. 7, pp. 1514–1520, 2005.
*   [7] X. Peng, W. Sansen, L. Hou, J. Wang, and W. Wu, “Impedance adapting compensation for low-power multistage amplifiers,” *IEEE Journal of Solid-State Circuits*, vol. 46, no. 2, pp. 445–451, 2010.
*   [8] Z. Yan, P.-I. Mak, M.-K. Law, and R. P. Martins, “A 0.016-mm² 144-$\mu$ w three-stage amplifier capable of driving 1-to-15 nf capacitive load with $>$0.95-mhz gbw,” *IEEE Journal of Solid-State Circuits*, vol. 48, no. 2, pp. 527–540, 2013.
*   [9] S. O. Cannizzaro, A. D. Grasso, R. Mita, G. Palumbo, and S. Pennisi, “Design procedures for three-stage cmos otas with nested-miller compensation,” *IEEE Transactions on Circuits and Systems I: Regular Papers*, vol. 54, no. 5, pp. 933–940, 2007.
*   [10] L. Wen, D. Fu, X. Li, X. Cai, T. Ma, P. Cai, M. Dou, B. Shi, L. He, and Y. Qiao, “Dilu: A knowledge-driven approach to autonomous driving with large language models,” *arXiv preprint arXiv:2309.16292*, 2023.
*   [11] G. Wang, Y. Xie, Y. Jiang, A. Mandlekar, C. Xiao, Y. Zhu, L. Fan, and A. Anandkumar, “Voyager: An open-ended embodied agent with large language models,” *arXiv preprint arXiv:2305.16291*, 2023.
*   [12] D. A. Boiko, R. MacKnight, B. Kline, and G. Gomes, “Autonomous chemical research with large language models,” *Nature*, vol. 624, no. 7992, pp. 570–578, 2023.
*   [13] J. Blocklove, S. Garg, R. Karri, and H. Pearce, “Chip-chat: Challenges and opportunities in conversational hardware design,” *arXiv preprint arXiv:2305.13243*, 2023.
*   [14] S. Thakur, B. Ahmad, Z. Fan, H. Pearce, B. Tan, R. Karri, B. Dolan-Gavitt, and S. Garg, “Benchmarking large language models for automated verilog rtl code generation,” in *2023 Design, Automation & Test in Europe Conference & Exhibition (DATE)*.   IEEE, 2023, pp. 1–6.
*   [15] M. Liu, T. Ene, R. Kirby, C. Cheng, N. Pinckney, R. Liang, J. Alben, H. Anand, S. Banerjee, I. Bayraktaroglu, B. Bhaskaran, B. Catanzaro, A. Chaudhuri, S. Clay, B. Dally, L. Dang, P. Deshpande, S. Dhodhi, S. Halepete, E. Hill, J. Hu, S. Jain, B. Khailany, K. Kunal, X. Li, H. Liu, S. Oberman, S. Omar, S. Pratty, A. Sarkar, Z. Shao, H. Sun, P. P. Suthar, V. Tej, K. Xu, and H. Ren, “Chipnemo: Domain-adapted llms for chip design,” 2023.
*   [16] Y. L. F. Y. L. S. D. Z. X. Z. Zihao Chen, Jiangli Huang, “Artisan: Automated operational amplifier design via domain-specific large language model,” *61th Design Automation Conference (DAC)*, 2024.
*   [17] C. Liu, Y. Liu, Y. Du, and L. Du, “Ladac: Large language model-driven auto-designer for analog circuits,” *Authorea Preprints*, 2024.
*   [18] Y. Yin, Y. Wang, B. Xu, and P. Li, “Ado-llm: Analog design bayesian optimization with in-context learning of large language models,” *arXiv preprint arXiv:2406.18770*, 2024.
*   [19] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and P. Luo, “Analogcoder: Analog circuit design via training-free code generation,” *arXiv preprint arXiv:2405.14918*, 2024.
*   [20] D. Eriksson, M. Pearce, J. Gardner, R. D. Turner, and M. Poloczek, “Scalable global optimization via local bayesian optimization,” in *Advances in Neural Information Processing Systems*, H. Wallach, H. Larochelle, A. Beygelzimer, F. d'Alché-Buc, E. Fox, and R. Garnett, Eds., vol. 32.   Curran Associates, Inc., 2019.
*   [21] W. Xiong, X. Meng, Y. Tao, and P. Ling, “Ai-driven learning and regeneration of analog circuit designs from academic papers,” *Authorea Preprints*, 2024.
*   [22] OpenAI, “Gpt-4 technical report,” *ArXiv*, vol. abs/2303.08774, 2023\. [Online]. Available: https://api.semanticscholar.org/CorpusID:257532815
*   [23] openai, “Gpt-4o system card,” https://openai.com/index/gpt-4o-system-card/, 2024.
*   [24] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. Narasimhan, and Y. Cao, “React: Synergizing reasoning and acting in language models,” *arXiv preprint arXiv:2210.03629*, 2022.
*   [25] P. Lewis, E. Perez, A. Piktus, F. Petroni, V. Karpukhin, N. Goyal, H. Küttler, M. Lewis, W.-t. Yih, T. Rocktäschel *et al.*, “Retrieval-augmented generation for knowledge-intensive nlp tasks,” *Advances in Neural Information Processing Systems*, vol. 33, pp. 9459–9474, 2020.
*   [26] Z. Ji, N. Lee, R. Frieske, T. Yu, D. Su, Y. Xu, E. Ishii, Y. J. Bang, A. Madotto, and P. Fung, “Survey of hallucination in natural language generation,” *ACM Comput. Surv.*, vol. 55, no. 12, mar 2023\. [Online]. Available: https://doi.org/10.1145/3571730
*   [27] Mathpix, “Mathpix ocr api for stem,” https://mathpix.com/ocr.
*   [28] H. B. Gurbuz, A. Balta, T. Dalyan, Y. D. Gokdel, and E. Afacan, “Img2sim-v2: A cad tool for user-independent simulation of circuits in image format,” in *2023 19th International Conference on Synthesis, Modeling, Analysis and Simulation Methods and Applications to Circuit Design (SMACD)*.   IEEE, 2023, pp. 1–4.
*   [29] Z. Tao, Y. Shi, Y. Huo, R. Ye, Z. Li, L. Huang, C. Wu, N. Bai, Z. Yu, T.-J. Lin *et al.*, “Amsnet: Netlist dataset for ams circuits,” *arXiv preprint arXiv:2405.09045*, 2024.
*   [30] R. G. Eschauzier and J. Huijsing, *Frequency compensation techniques for low-power operational amplifiers*.   Springer Science & Business Media, 1995, vol. 313.
*   [31] D. Karaboga, B. Gorkemli, C. Ozturk, and N. Karaboga, “A comprehensive survey: artificial bee colony (abc) algorithm and applications,” *Artificial intelligence review*, vol. 42, pp. 21–57, 2014.
*   [32] langchain ai, “Langchain,” https://github.com/langchain-ai/langchain, 2024.
*   [33] P. Jespers, *The gm/ID Methodology, a sizing tool for low-voltage analog CMOS Circuits: The semi-empirical and compact model approaches*.   Springer Science & Business Media, 2009.