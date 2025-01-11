<!--yml
category: 未分类
date: 2025-01-11 12:46:57
-->

# ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes

> 来源：[https://arxiv.org/html/2403.06294/](https://arxiv.org/html/2403.06294/)

Shengxin Hong, Liang Xiao, Xin Zhang, Jianxia Chen Hubei University of Technology, Wuhan, China lx@mail.hbut.edu.cn

###### Abstract

There are two main barriers to using large language models (LLMs) in clinical reasoning. Firstly, while LLMs exhibit significant promise in Natural Language Processing (NLP) tasks, their performance in complex reasoning and planning falls short of expectations. Secondly, LLMs use uninterpretable methods to make clinical decisions that are fundamentally different from the clinician’s cognitive processes. This leads to user distrust. In this paper, we present a multi-agent framework called ArgMed-Agents, which aims to enable LLM-based agents to make explainable clinical decision reasoning through interaction. ArgMed-Agents performs self-argumentation iterations via Argumentation Scheme for Clinical Discussion (a reasoning mechanism for modeling cognitive processes in clinical reasoning), and then constructs the argumentation process as a directed graph representing conflicting relationships. Ultimately, use symbolic solver to identify a series of rational and coherent arguments to support decision. We construct a formal model of ArgMed-Agents and present conjectures for theoretical guarantees. ArgMed-Agents enables LLMs to mimic the process of clinical argumentative reasoning by generating explanations of reasoning in a self-directed manner. The setup experiments show that ArgMed-Agents not only improves accuracy in complex clinical decision reasoning problems compared to other prompt methods, but more importantly, it provides users with decision explanations that increase their confidence.

###### Index Terms:

Clinical Decision Support, Large Language Model, Multi-Agent System, Explainable AI

## I Introduction

Large Language Models (LLMs) [[1](https://arxiv.org/html/2403.06294v3#bib.bib1)] have received a lot of attention for their human-like performance in a variety of domains. In the medical field especially, preliminary studies have shown that LLMs can be used as clinical assistants for tasks such as writing clinical texts [[2](https://arxiv.org/html/2403.06294v3#bib.bib2)], providing biomedical knowledge [[3](https://arxiv.org/html/2403.06294v3#bib.bib3)] and drafting responses to patients’ questions [[4](https://arxiv.org/html/2403.06294v3#bib.bib4)]. However, the following barriers to building an LLM-based clinical decision-making system still exist: (i) LLMs still struggle to provide secure, stable answers when faced with highly complex clinical reasoning tasks [[5](https://arxiv.org/html/2403.06294v3#bib.bib5)]. (ii) There is a perception that LLMs use unexplainable methods to arrive at clinical decisions (known as black boxes), which may have led to user distrust [[6](https://arxiv.org/html/2403.06294v3#bib.bib6)].

To address these barriers, exploring the capabilities of LLMs in argumentative reasoning is a promising direction. Argumentation is a means of conveying a compelling point of view that can increase user acceptance of a position. Its considered a fundamental requirement for building Human-Centric AI [[7](https://arxiv.org/html/2403.06294v3#bib.bib7)]. As computational argumentation has become a growing area of research in Natural Language Processing (NLP) [[8](https://arxiv.org/html/2403.06294v3#bib.bib8)], researchers have begun to apply argumentation to a wide range of clinical reasoning applications, including analysis of clinical discussions [[9](https://arxiv.org/html/2403.06294v3#bib.bib9)], clinical decision making [[10](https://arxiv.org/html/2403.06294v3#bib.bib10), [11](https://arxiv.org/html/2403.06294v3#bib.bib11), [12](https://arxiv.org/html/2403.06294v3#bib.bib12)], address clinical conflicting [[13](https://arxiv.org/html/2403.06294v3#bib.bib13)]. In the recent past, some work has assessed the ability of LLMs in argumentation reasoning [[14](https://arxiv.org/html/2403.06294v3#bib.bib14), [15](https://arxiv.org/html/2403.06294v3#bib.bib15)] or non-monotonic reasoning [[16](https://arxiv.org/html/2403.06294v3#bib.bib16)]. Although LLMs show some potential for computational argumentation, more results show that LLMs perform poorly in logical reasoning tasks [[17](https://arxiv.org/html/2403.06294v3#bib.bib17)], and better ways to utilise LLMs for non-monotonic reasoning tasks need to be explored.

Meanwhile, LLM as agent studies have been surprisingly successful [[18](https://arxiv.org/html/2403.06294v3#bib.bib18), [19](https://arxiv.org/html/2403.06294v3#bib.bib19), [20](https://arxiv.org/html/2403.06294v3#bib.bib20)]. These methods use LLMs as computational engines for autonomous agents, and optimise the reasoning, planning capabilities of LLMs through external tools (e.g. symbolic solvers, APIs, retrieval tools, etc.) [[21](https://arxiv.org/html/2403.06294v3#bib.bib21), [20](https://arxiv.org/html/2403.06294v3#bib.bib20)], multi-agent interactions [[22](https://arxiv.org/html/2403.06294v3#bib.bib22)] and novel algorithmic frameworks [[23](https://arxiv.org/html/2403.06294v3#bib.bib23)]. Through this design, LLMs agents can interact with the environment and generate action plans through intermediate reasoning steps that can be executed sequentially to obtain an effective solution.

Motivated by these concepts, we present ArgMed-Agents, a multi-agent framework designed for explainable clinical decision reasoning. We formalised the cognitive process of clinical discussion using an argumentation scheme for clinical discussion (ASCD) as a prompt strategy for interactive reasoning by LLM agents. There are three types of agents in ArgMed-Agents: the Generator, the Verifier, and the Reasoner. the Generator generates arguments to support clinical decisions based on the argumentation scheme; the Verifier checks the arguments for legitimacy based on the critical question, and if not legitimate, it asks the Generator to generate attack arguments; Reasoner is a LLM agent with symbolic solver that identifies reasonable, non-contradictory arguments in the resulting directed argumentation graph as decision support.

In our method, we do not expect every proposed argument or detection of Generator or Verifier to be correct, instead we consider their generation as a assumption. The LLM agents are induced to recursively iterate in a self-argumentative manner through the prompt strategy , while the newly proposed assumptions always contradict the old ones, and eventually the Reasoner eliminates unreasonable assumptions and identifies coherent arguments, leading to consistent reasoning results. ArgMed-Agents enables LLMs to explain its own outputs in terms of self-cognitive profiling by modelling their own generation as a prompt for question recursion.

Our experiment was divided into two parts, evaluating accuracy and explainability of ArgMed-Agents clinical reasoning, respectively. We conducted experiments on two datasets, including MedQA [[24](https://arxiv.org/html/2403.06294v3#bib.bib24)] and PubMedQA [[25](https://arxiv.org/html/2403.06294v3#bib.bib25)]. To better align with real-world application scenarios, our study focused on a zero-shot setting. The results show that ArgMed-Agents achieves better performance in both accuracy and explainability compared to direct generation and Chain of Thought (CoT).

In summary, we take following contribution:

*   •

    We present a novel multi-LLM agent framework called ArgMed-Agents for complex clinical reasoning tasks.

*   •

    Our approach effectively combines argumentation frameworks and cognitive clinical medicine. Arguments are presented through LLM simulations of clinical discussions and reasoning results are solved through formal computational models, which avoids the cumulative errors of LLM logical reasoning and improves the safety of clinical reasoning.

*   •

    We propose the conjecture about ArgMed-Agents’ theoretical guarantees now, which is that identify a reasoning error in the LLM when ArgMed-Agents considers all decisions unacceptable. This conjecture serves as a means for us to identify boundaries in the capabilities of large language models (e.g., hallucinatory phenomena, knowledge conflicts), which are important for clinical decision support. If the LLM makes a serious error but humans do not detect it in time, it may cause irreparable damage to the patient.

*   •

    We We perform an extensive evaluation of ArgMed-Agents, which demonstrate the accuracy, explainability and safety benefits in clinical decision support of our method.

## II Preliminaries

### II-A Argumentation Scheme for Clinical Discussion

Our approach utilises the notion of computational argumentation for to support reasoning. Abstract Argumentation (AA) [[26](https://arxiv.org/html/2403.06294v3#bib.bib26)] are pair $\langle\mathcal{A},\mathcal{R}\rangle$ composed of two components: a set of abstract arguments $\mathcal{A}$, and a binary attack relation $\mathcal{R}$. Given an AA framework $\langle\mathcal{A},\mathcal{R}\rangle$, $a,b\in\mathcal{A}$ and $(a,b)\in\mathcal{R}$ indicates that $a$ attacks $b$. In this framework, there exist $a\in\mathcal{A}$ is acceptable if and only if:

*   •

    not exist $b\in\mathcal{A}$ such that $(b,a)\in\mathcal{R}$;

*   •

    there exist $b,c\in\mathcal{A}$ such that $(b,a)\in\mathcal{R}$ and $(c,b)\in\mathcal{R}$;

On top of this, numerous studies [[27](https://arxiv.org/html/2403.06294v3#bib.bib27), [12](https://arxiv.org/html/2403.06294v3#bib.bib12), [9](https://arxiv.org/html/2403.06294v3#bib.bib9)] have explored the application of argumentation in the clinical domain. In this section, we provide a summary of these endeavors, focusing on Argumentation Schemes for Clinical Discussion (ASCD). The concept of Argumentation Scheme (AS) originated within the domain of informal logic, stemming from the seminal works of [[28](https://arxiv.org/html/2403.06294v3#bib.bib28), [29](https://arxiv.org/html/2403.06294v3#bib.bib29)]. Argumentation Scheme serves as a semi-formalized framework for capturing and analyzing human reasoning patterns. Formally defined as $AS=\langle P,c,V\rangle$, it comprises a collection of premises ($P$), a conclusion ($c$) substantiated by these premises, and variables ($V$) inherent within the premises ($P.V$) or the conclusion ($c.V$). A pivotal aspect of the argumentation scheme is the delineation of Critical Questions (CQs) pertinent to AS. Failure to address them prompts a challenge to both the premises and the conclusion posited by the scheme. Consequently, the role of CQs is to instigate argument generation; when an AS is contested, it engenders the formulation of a counter-argument in response to the initial AS. This iterative process culminates in the construction of an attack argument graph, facilitating a nuanced understanding of argumentative dynamics.

ASCD encapsulates a variety of argumentation schemes that analyse the clinical discussion and reasoning process, such as Argumentation Scheme for Decision Making (ASDM), Argumentation Scheme for Side Effects (ASSE) and Argumentation Scheme for Danger Apeal (ASDA). Figure [1](https://arxiv.org/html/2403.06294v3#S2.F1 "Figure 1 ‣ II-A Argumentation Scheme for Clinical Discussion ‣ II Preliminaries ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes") illustrates an example of ASDM.

![Refer to caption](img/c89fb29203ecf73949b07fed9ac1ff7b.png)

Figure 1: An example of Argumentation Scheme for Decision Making

## III Method

In this section, we propose a multi-agent framework called ArgMed-Agents, which supports the seamless integration of prompt strategy designed based on argumentation scheme for clinical discussion into agent interactions. Our approach enhances LLMs to be able to perform explainable clinical decision reasoning without the need for expert involvement in knowledge encoding.

### III-A ArgMed-Agents: a Multi-LLM-Agents Framework

![Refer to caption](img/8bd0b6b60b2b95a4f4236f15c49927ee.png)

Figure 2: An example from the MedQA USMLE dataset, with the entire process of ArgMed-Agents reasoning about the clinical problem. Notably, the letters in the argumentation framework correspond to the serial numbers of the four generators on the right, representing the premises and conclusion generated by that generator. In the argumentation framework, the red nodes ($A$ and $C$) represent arguments in support of the decision and the yellow nodes ($B$ and $D$) represent arguments in support of the beliefs.

ArgMed-Agents framework includes three distinct types of LLMs agent:

*   •

    Generator(s): Generate arguments based on the current situation.

*   •

    Verifier: Whenever a new argument is generated, the verifier checks the accuracy of the argument by asking and answering CQs in a self-questioning manner. When the CQ validation is rejected, the reason why the CQ was rejected is returned to the generator, which proposes a new argument based on that CQ. This process is iterated until no more arguments are generated.

*   •

    Reasoner: It is an LLM agent equipped with a symbolic solver for computational argumentation. The reasoner records the complete process of iteration and identifies semantic relationships between arguments (attack or support). At the end of the iteration, all arguments form a complete abstract argumentation framework. The reasoner searches for a set of coherent and plausible arguments through the symbolic solver as a support for decision making.

In ArgMed-Agents, it do not expect single generation or single verification to be correct. ArgMed-Agents uses generation as a assumption to recursively prompt the model with critical questions, identifying conflicting and erroneous arguments in an iterative process. Ultimately, such mutually attacking arguments are converted into a formal framework that solves for a subset of reasonably coherent arguments via Reasoner. Figure [2](https://arxiv.org/html/2403.06294v3#S3.F2 "Figure 2 ‣ III-A ArgMed-Agents: a Multi-LLM-Agents Framework ‣ III Method ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes") depicts how multi-agents interacts to simulate clinical discussion. In the example of Figure [2](https://arxiv.org/html/2403.06294v3#S3.F2 "Figure 2 ‣ III-A ArgMed-Agents: a Multi-LLM-Agents Framework ‣ III Method ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes"), the generator first proposes Proxetine as a therapeutic drug. However, after several rounds of discussion, ArgMed-Agents raised arguments about Proxetine’s side effects, and eventually settled on the more effective therapeutic drug as Trazodone.

The theoretical motivation for our method stems from non-monotonic logic, logical intuition and cognitive clinical. Studies have shown that LLM performs reasonably well for simple reasoning, single-step reasoning problems, however, as the number of reasoning steps rises, the rate of correct reasoning decreases in a catastrophic manner [[30](https://arxiv.org/html/2403.06294v3#bib.bib30)]. Thus, we consider that LLM is primed with logical intuition, but lacks true logical reasoning ability.

In light of this, ArgMed-Agents framework guides LLM in a recursive form to generate a series of casual reasoning steps as a tentative conclusion, with any further evidence withdrawing their conclusion. The process ultimately leads to a directed graph representing the disputed relationships. Moreover, our method performs graph inference with the help of symbolic solvers, which avoids the cumulative error of LLM inference. LLM can be viewed as a large knowledge base containing a large amount of conflicting knowledge. ArgMed-Agents provides a formal method for discovering conflicts and solving for consistent reasoning results, which facilitates the LLM’s ability to gradually come up with new knowledge to use in revising its own initial conclusion.

Within the ArgMed-Agents framework, agents endowed with various LLM roles engage in collaborative interactions through a formalized process that simulate clinical discussion. This design not only facilitates LLMs’ engagement in clinical reasoning through a critical-thinking approach, aligning their cognitive processes with those of clinicians to enhance decision explainability, but also enables the effective extraction and highlighting of implicit knowledge within LLMs that is not easily accessible through traditional prompts.

### III-B Formal Computational Models & Theorems

In this section, we describe how ArgMed-Agents performs formal reasoning and explains decisions. First, we define the ArgMed-Agents interaction model

###### Definition 1  (ArgMed-Agents Interaction).

In ArgMed-Agents, interaction is a series of dialogues $\mathcal{D}=\{s_{1},...,s_{n}|n>1\}$ involving generator agent $g$ and verifier agent $v$, and the following conditions hold:

*   •

    $s_{i}=\langle g,a\rangle$ is an arguments presented by generator $g$.

*   •

    $s_{k}=\langle v,cq\rangle$ is an critical question $cq\in CQs$ presented by verifier $v$.

*   •

    If there exist $s_{i}$ and $s_{i+2}$ from the generator then $s_{i+1}$ from the verifier such that $s_{i+2}$ attack $s_{i}$.

In ArgMed-Agents, we define the argument is presented by the generator and the verifier constructs an attack relation between the argument. Specifically, when the verifier rejects an argument, it leads to a new argument and this argument attacks the original argument; when the verifier accepts an argument, the round of dialogue ends. In this way, the reasoner can connect the semantic relations of all the arguments and construct an argumentation framework. Next, we define the argumentation framework for ArgMed-Agents:

###### Definition 2  (Argumentation in ArgMed-Agents).

An Argumentation Framework AF for ArgMed-Agents is a pair $\langle\mathcal{A},\mathcal{R}\rangle$ such that:

*   •

    $\mathcal{A}$ is arguments set constructed by ${s_{i}}\subseteq\mathcal{D}$ from generator and $\mathcal{R}$ is a set of binary attack relation.

*   •

    $\mathcal{A}=Args_{d}(\mathcal{A})\cup Args_{b}(\mathcal{A})$ such that arguments in argument set $\mathcal{A}$ are distinguished between two types of arguments: arguments in support of decisions $Args_{d}(\mathcal{A})$ and arguments in support of beliefs $Args_{b}(\mathcal{A})$.

We divided ArgMed-Agents’ arguments into two types: arguments in support of decisions $Args_{d}(\mathcal{A})$ and arguments in support of beliefs $Args_{b}(\mathcal{A})$. These two types of arguments play different roles, $Args_{d}(\mathcal{A})$ build on beliefs and goals and try to justify choices, while $Args_{b}(\mathcal{A})$ always try to undermine decision arguments. In addition, we made the following setup for both arguments: First, arguments in support of different decisions are in conflict with each other; Second, arguments in support of decisions are not allowed to attack arguments in support of beliefs. Formalized as:

###### Definition 3.

Given an AF for ArgMed-Agents $\langle\mathcal{A},\mathcal{R}\rangle$, $\mathcal{R}\subseteq Args_{d}(\mathcal{A})\times Args_{b}(\mathcal{A})$ such that:

*   •

    For all $a_{1},a_{2}$  $\in Args_{d}(\mathcal{A})$ such that $a_{1}\neq a_{2},$  $(a_{1},a_{2}),(a_{2},a_{1})\in\mathcal{R}.$

*   •

    Not exists $(arg_{1},arg_{2})$  $\in\mathcal{R}$ such that $arg_{1}\in Args_{d}(\mathcal{A})$ and $arg_{2}\in Args_{b}(\mathcal{A})$.

Definition [4](https://arxiv.org/html/2403.06294v3#Thmdefinition4 "Definition 4\. ‣ III-B Formal Computational Models & Theorems ‣ III Method ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes") describes how ArgMed-Agents identifies optional decisions and the concept of an explanation set for a decision.

###### Definition 4.

Given an AF for ArgMed-Agents $\langle\mathcal{A},\mathcal{R}\rangle$, if and only if a decision $d\in Args_{d}(\mathcal{A})$ is acceptable then this decision is optional decision. A set of decision support explanation $E$ include:

*   •

    There exist an decision $d=Args_{d}(E)$ is optional decision.

*   •

    All arguments $a\in E$ is acceptable and confict-free.

*   •

    Among the sets that satisfy the above two conditions, $E$ is the set that contains maximal elements.

Next, we use an example from MedQA to illustrate how Definition [4](https://arxiv.org/html/2403.06294v3#Thmdefinition4 "Definition 4\. ‣ III-B Formal Computational Models & Theorems ‣ III Method ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes") is applied in ArgMed-Agents.

###### Example 1  (From MedQA).

An 18-year-old woman presents with recurrent headaches. The pain is usually unilateral, pulsatile in character, exacerbated by light and noise, and usually lasts for a few hours to a full day. The pain is sometimes triggered by eating chocolates. These headaches disturb her daily routine activities. The physical examination was within normal limits. She also has essential tremors. Which drug is suitable in her case for the prevention of headaches?

After several rounds of discussion, ArgMed-Agents presented five arguments, including three decisions about treatments and two pieces of evidence about side effect, and they formed the following Argumentation framework:

![[Uncaptioned image]](img/ca1b18c6ff3b35871e6d4615550803b2.png)

Based on the definitions, we can obtain the following reasoning results:

*   •

    Optional decisions: $B$ (Propranolol) or $C$ (Verapamil);

*   •

    Explaination set $E=\{\{B,D,E\},\{C,D,E\}\}$;

Notably, although two decisions are optional, we define the exclusivity of the decisions so that they do not appear in the decision support at the same time (this avoids duplicate medication).

With this formal model, we present a conjecture about ArgMed-Agents’ theoretical guarantee:

> Given an AF for ArgMed-Agents $\langle\mathcal{A},\mathcal{R}\rangle$ and $E$ is its decisions support. There are errors in the clinical reasoning of ArgMed-Agents in this case iff $Args_{d}(E)=\varnothing$.

Clinical reasoning errors are discussed in [[22](https://arxiv.org/html/2403.06294v3#bib.bib22)]. They consider that most clinical reasoning errors in LLMs are due to confusion about domain knowledge. On the other hand, research by [[3](https://arxiv.org/html/2403.06294v3#bib.bib3)] points out that LLMs may produce compelling misinformation about medical treatment, so it is crucial to recognize this illusion, which is difficult for humans to detect. For this purpose, We conjecture that the relevant mechanisms for identifying clinical reasoning errors in ArgMed-Agents are as follows: When any decision in the $AF$ for ArgMed-Agents is not accepted, we consider there are errors in the reasoning by LLM and LLM’s knowledge reserves are insufficient to address this issues. This mechanism assists ArgMed-Agents in identifying the capability boundaries of LLMs which helps to avoid the risks associated with adopting erroneous decisions and achieving more robust and safe clinical reasoning. We base this conjecture on the following: ArgMed-Agents bootstrap the internal implicit knowledge of LLMs by iterating over them, which may be conflicting (possibly due to dirty data or timing conflicts). When there is only a small amount of conflicting knowledge, ArgMed-Agents can reason out the correct decision by non-monotonic reasoning; When there is a large amount of conflicting knowledge, the argumentation framework will be very large (containing erroneous arguments generated by hallucinatory phenomena), and the logical errors in this may lead ArgMed-Agents to be unable to reason out any correct decisions. We therefore consider this situation as the boundary of the LLM’s capabilities.

## IV Experiments

In this section, we demonstrate the potential of ArgMed-Agents in clinical decision reasoning by evaluating the accuracy and explainability.

### IV-A Settings

We implemented different types of agents in ArgMed-Agents using the APIs GPT-3.5-turbo and GPT-4 provided by OpenAI [[1](https://arxiv.org/html/2403.06294v3#bib.bib1)], and according to our setup, the LLM Agents are implemented with the same LLM and different few-shot prompts. Each agent is configured with specific parameters: the temperature is set to 0.0, as well as a dialogue limit of 8 and the maximum number of decisions allowed for ArgMed-Agents to generate limit of 4 (because the MedQA dataset is multiple choice with only four options), which is to prevent the agents from getting into loops with each other. On the other hand, we using python3 to implemente a symbolic solver of our formal model.

### IV-B Datasets

The following two datasets were used to assess the accuracy and explainability of ArgMed-Agents clinical reasoning:

*   •

    MedQA [[24](https://arxiv.org/html/2403.06294v3#bib.bib24)]:Answering multiple-choice questions derived from the United States Medical License Exams (USMLE). This dataset is sourced from official medical board exams and encompasses questions in English, simplified Chinese, and traditional Chinese. The total question counts for each language are 12,723, 34,251, and 14,123, respectively.

*   •

    PubMedQA [[25](https://arxiv.org/html/2403.06294v3#bib.bib25)]: A biomedical question and answer (QA) dataset collected from PubMed abstracts. The task of PubMedQA is to answer yes/no/maybe research questions using the corresponding abstracts (e.g., Do preoperative statins reduce atrial fibrillation after coronary artery bypass graft surgery).

### IV-C Accuracy

In accuracy evaluation, we rigorously curated datasets by randomly selecting 300 examples from each, ensuring a balanced representation for robust analysis. To establish a baseline for comparison, we employed GPT direct generation alongside Chain of Thought (CoT) as outlined in [[31](https://arxiv.org/html/2403.06294v3#bib.bib31)]. Notably, our primary focus centered on evaluating the efficacy of ArgMed-Agents within the domain of clinical decision reasoning. It is pertinent to highlight that while datasets such as MedQA and PubMedQA contained biomedical general knowledge quiz-type questions, we intentionally excluded this subset during the example selection process. This deliberate exclusion aimed to maintain the integrity of our evaluation specifically within the realm of clinical decision-making, thereby ensuring a targeted assessment of ArgMed-Agents’ performance.

{tblr}

hline1,8 = -0.08em, hline2,5 = -, Model & Method MedQA PubMedQA
Direct 52.7 68.4
GPT-3.5-turbo CoT 48.0 71.5
ArgMed-Agents 62.1 78.3
Direct 67.8 72.9
GPT-4 CoT 71.4 77.2
ArgMed-Agents 83.3 81.6

TABLE I: Results of the accuracy of various clinical reasoning methods on MedQA and PubMedQA datasets.

Table [I](https://arxiv.org/html/2403.06294v3#S4.T1 "TABLE I ‣ IV-C Accuracy ‣ IV Experiments ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes") presents the accuracy outcomes for MedQA and PubMedQA, contrasting ArgMed-Agents with various baselines under direct generation and Chain of Thought (CoT) configurations. Notably, evaluations employing GPT-3.5-turbo and GPT-4 models revealed that our proposed ArgMed-Agents notably enhanced accuracy in clinical decision reasoning tasks compared to CoT and other baselines. This finding underscores the efficacy of ArgMed-Agents in augmenting the clinical reasoning capabilities of Language Model (LLM) systems. Intriguingly, through multiple experiment repetitions, we observed instances where integrating CoT resulted in an unexpected decline in performance. This phenomenon might stem from the propensity of LLMs to experience hallucinatory behaviors, exacerbated by CoT’s potential to perpetuate such hallucinations indefinitely. In contrast, ArgMed-Agents corrects initial erroneous conclusions by means of iterative argumentation, effectively mitigates this issue.

### IV-D Explainability

The main focus of Explainable AI (XAI) is usually the reasoning behind decisions or predictions made by AI that become more understandable and transparent. In the context of XAI, Explainability is defined as follows [[32](https://arxiv.org/html/2403.06294v3#bib.bib32)]:

*   …level of understanding how the AI-based system
    came up with a given result. 

Based on this criterion, we define measures of LLM’s explanability:

*   •

    How far do the explanations given by LLM help users to predict LLM’s decisions?

Our team has produced a fully functional knowledge-based clinical decision support system (CDSS) [[33](https://arxiv.org/html/2403.06294v3#bib.bib33)] at an early stage that can be used to assist in treatment decisions for complex diseases such as cancer, neurological disorders, and infectious diseases. This CDSS consist of computer-interpretable guidelines in the form of Resource Description Framework (RDF) [[34](https://arxiv.org/html/2403.06294v3#bib.bib34)] and the corresponding inference engine. Knowledge-based CDSS can be viewed as explainable systems.

With this, our experimental setup is as follows: We set the direct generation explanation and COT as the baseline and the knowledge-based CDSS as the benchmark of what we expect explainability of ArgMed-Agents to be close to. We documented the input (i.e., questions) and reasoning process for 100 examples in MedQA (e.g., COT’s chain of reasoning, RDF inference nodes in knowledge-based CDSS and the complete dialogue between ArgMed-Agents and the corresponding argumentation framework). For evaluation, We fine-tuned an LLM to act as an evaluator, causing it to predict the corresponding decision based on the inference record. We considered the accuracy of the evaluator’s predictions as an indicator of interpretability (i.e., how well the reasoning process helps the user understand the decision).

{tblr}

hline1-2,5,8-9 = -, Model & Method Pre.
Direct 0.53
GPT-3.5-turbo CoT 0.59
ArgMed-Agents 0.87
Direct 0.68
GPT-4 CoT 0.73
ArgMed-Agents 0.91
Knowledge-based CDSS 0.95

TABLE II: Predict accuracy (Pre.) with reasoning record of different models and methods in 100 MedQA examples.

The knowledge-based CDSS achieved the highest predictive accuracy (0.95) as the benchmark for explainability. Among the tested methods, ArgMed-Agents demonstrated significantly higher predictive accuracy compared to direct and CoT methods in both GPT-3.5-turbo and GPT-4 models. ArgMed-Agents with GPT-4 achieved 0.91 predictive accuracy, closely approaching the knowledge-based CDSS’s level of explainability.

The study demonstrates that ArgMed-Agents significantly enhance the explainability of LLMs, enabling users to better understand and predict the models’ decisions. This suggests that employing advanced argumentation frameworks can bridge the gap between black-box AI models and fully transparent, knowledge-based systems like the CDSS.

### IV-E Discussion

Conjecture 1 posits that errors in clinical reasoning occur in ArgMed-Agents if the AF does not yield any acceptable decisions. To provide a preliminary proof, we analyzed instances where ArgMed-Agents failed to produce acceptable decisions and correlated these instances with the observed errors. Our data indicates that in 63% of cases where ArgMed-Agents failed to yield an acceptable decision set, there were identifiable clinical reasoning errors. These errors were primarily due to the presence of extensive conflicting knowledge within the AF, which prevented the system from arriving at a coherent decision. Specifically, 76% of these errors were linked to confilct knowledge, while 24% were due to insufficient domain knowledge.

Our analysis suggests that the capacity of ArgMed-Agents to navigate and resolve conflicts in medical knowledge is a critical determinant of its effectiveness. This is consistent with the assumption as follows: for a given input (e.g., a clinical reasoning problem), when only a small amount of conflicting knowledge exists in the LLM, ArgMed-Agents can reason out the correct treatment plan by vetoing out the erroneous arguments through further argumentation. When there is a large amount of inconsistent knowledge in the LLM, these conflicts lead to a complex system that prevents the LLM from maintaining logical consistency in its generation (i.e., the phenomenon of hallucination), thus hindering effective decision-making.

The advantage of ArgMed-Agents over similar existing techniques is that the conditions under which LLM reasoning fails (i.e. when faced with a large amount of conflicting knowledge or insufficient domain knowledge) can be recognised through formal arguments, allowing us to better understand and predict the limitations of LLM in clinical applications. This understanding is critical to mitigating the risks associated with poor decision-making in healthcare settings.

In addition to this, we analyse the traceability of ArgMed-Agents reasoning. We find that the reasoning process by which the CoT method arrives at an answer is usually logically incomplete, whereas the reasoning process of ArgMed-Agents is much more refined. Interestingly, we found that the reasoning of the knowledge-based CDSS on many examples can be regarded as a reasoning subgraph of ArgMed-Agents, probably because ArgMed-Agents traverses all the reasoning paths in a randomly generated manner, and thus the reasoning graph includes not only the reasoning paths of the correct decisions, but also the explanations of the incorrect decisions. This finding provides further evidence of the explainability of ArgMed-Agents. In many clinical decision cases, patients may want to understand not just why a particular decision was taken, but why another decision could not be adopted.

## V Related Work

### V-A LLM-based Clinical Decision Support System

Extensive research underscores the potential utility of Large Language Models (LLMs) in medical applications [[35](https://arxiv.org/html/2403.06294v3#bib.bib35), [36](https://arxiv.org/html/2403.06294v3#bib.bib36), [37](https://arxiv.org/html/2403.06294v3#bib.bib37)]. However, these models encounter challenges in making reliable decisions when faced with complex clinical scenarios that demand advanced medical expertise and robust reasoning skills [[3](https://arxiv.org/html/2403.06294v3#bib.bib3)]. Consequently, significant efforts have been initiated to augment the clinical reasoning capabilities of LLMs. [[22](https://arxiv.org/html/2403.06294v3#bib.bib22)] introduces a Multi-disciplinary Collaboration (MC) framework employing LLM-based agents in simulated role-playing scenarios, facilitating collaborative, iterative discussions aimed at consensus building. Despite yielding promising outcomes, this approach struggles to formalize iterative results effectively to enhance the inference performance of LLMs using dedicated inference tools.

Another approach, proposed by [[38](https://arxiv.org/html/2403.06294v3#bib.bib38)], leverages diagnostic reasoning prompts to enhance clinical reasoning and interpretability in LLMs. However, this approach improves explainability through templates designed for specific diseases, whereas ArgMed-Agents is more automated. In addition to this, ArgMed-Agents provides explanations for why a decision would not have been chosen.

Additionally, research efforts such as those by [[39](https://arxiv.org/html/2403.06294v3#bib.bib39), [40](https://arxiv.org/html/2403.06294v3#bib.bib40)] involve fine-tuning LLMs using extensive datasets sourced from medical and biomedical literature. In contrast to these approaches, our method focuses on exploiting latent medical knowledge inherently present within LLMs to enhance their reasoning abilities in a training-free setting.

### V-B Logical Reasoning with LLMs

An extensive body of research has been dedicated to utilizing symbolic systems to augment reasoning, which encompasses a variety of methodologies including code environments, knowledge graphs, and formal theorem provers [[21](https://arxiv.org/html/2403.06294v3#bib.bib21), [41](https://arxiv.org/html/2403.06294v3#bib.bib41), [42](https://arxiv.org/html/2403.06294v3#bib.bib42)]. In the study conducted by Jung et al. [[43](https://arxiv.org/html/2403.06294v3#bib.bib43)], reasoning is framed as a satisfiability problem of its logical relations through the application of inverse causal reasoning. This approach leverages SAT solvers to enhance consistent reasoning and thereby improves the reasoning capabilities of Large Language Models (LLMs).

Zhang et al.’s work [[18](https://arxiv.org/html/2403.06294v3#bib.bib18)] takes a different approach by employing cumulative reasoning to break down tasks into smaller, more manageable components, which simplifies the problem-solving process and boosts overall efficiency. In another study, Xiu et al. [[16](https://arxiv.org/html/2403.06294v3#bib.bib16)] explore the non-monotonic reasoning capabilities of LLMs. However, despite the promising initial results demonstrated by LLMs, their performance is notably inadequate in terms of generalization and proof-based traceability, with a marked decline in effectiveness as the depth of reasoning increases.

Consistent with our findings, recent studies have begun to investigate the potential for enhancing argumentative reasoning in LLMs [[14](https://arxiv.org/html/2403.06294v3#bib.bib14)]. These efforts aim to bolster the argumentative reasoning abilities of LLMs, as evidenced by the work of Dewynter et al. [[44](https://arxiv.org/html/2403.06294v3#bib.bib44)] and Castagna et al. [[15](https://arxiv.org/html/2403.06294v3#bib.bib15)], which strive to further develop this aspect of LLM performance.

## VI Conclusion

In this work, We propose a novel medical multi-agent interaction framework called ArgMed-Agents, which inspires agents of different roles to simulate the process of clinical discussion, iterating through argumentation and critical questioning. Finally, Reasoner agent to identify a set of reasonable and coherent arguments in this framework as decision support and explaintion. The experimental results indicate that, compared to different baselines, using ArgMed-Agent for clinical decision-making reasoning achieves greater accuracy and provides inherent explanations for its inferences. In addition to this, the analyses in our discussion show that ArgMed-Agents are able to identify their own reasoning errors and ability boundaries to a large extent. This means that ArgMed-Agents provides safer decisions compared to other state-of-the-art methods. Our goal in this work is to provide healthcare professionals with powerful tools to enhance their decision-making process and ultimately improve patient outcomes.

Despite the success of ArgMed-Agents, there are still some limitations. We suggest the following future directions for the linmitations: (1) While we used abstract argumentation to formalise the results of the ArgMed-Agents clinical discussion, extension to more expressive logics (e.g., first-order logic, descriptive logic, modal logic, or probabilistic logic) would allow for more sophisticated reasoning and argument generation. (2) Adaptive approaches that tailor arguments to the needs and preferences of individual users can further improve the effectiveness and explainability of clinical decisions. Future work could explore interactive clinical decision support systems based on patient/physician feedback.

## References

*   [1] OpenAI, “Gpt-4 technical report,” 2023.
*   [2] A. Nayak, M. S. Alkaitis, K. Nayak, M. Nikolov, K. P. Weinfurt, and K. Schulman, “Comparison of history of present illness summaries generated by a chatbot and senior internal medicine residents,” *JAMA Intern Med*, vol. 183, no. 9, pp. 1026–1027, Sep 2023.
*   [3] K. Singhal, S. Azizi, T. Tu, S. S. Mahdavi, J. Wei, H. W. Chung, N. Scales, A. Tanwani, H. Cole-Lewis, S. Pfohl, P. Payne, M. Seneviratne, P. Gamble, C. Kelly, N. Scharli, A. Chowdhery, P. Mansfield, B. A. y Arcas, D. Webster, G. S. Corrado, Y. Matias, K. Chou, J. Gottweis, N. Tomasev, Y. Liu, A. Rajkomar, J. Barral, C. Semturs, A. Karthikesalingam, and V. Natarajan, “Large language models encode clinical knowledge,” 2022.
*   [4] J. W. Ayers, A. Poliak, M. Dredze, E. C. Leas, Z. Zhu, J. B. Kelley, D. J. Faix, A. M. Goodman, C. A. Longhurst, M. Hogarth, and D. M. Smith, “Comparing Physician and Artificial Intelligence Chatbot Responses to Patient Questions Posted to a Public Social Media Forum,” *JAMA Internal Medicine*, vol. 183, no. 6, pp. 589–596, 06 2023\. [Online]. Available: https://doi.org/10.1001/jamainternmed.2023.1838
*   [5] A. Pal, L. K. Umapathi, and M. Sankarasubbu, “Med-halt: Medical domain hallucination test for large language models,” 2023.
*   [6] E. Eigner and T. Händler, “Determinants of llm-assisted decision-making,” 2024.
*   [7] E. Dietz, A. Kakas, and L. Michael, “Argumentation: A calculus for human-centric ai,” *Frontiers in Artificial Intelligence*, vol. 5, 2022\. [Online]. Available: https://www.frontiersin.org/articles/10.3389/frai.2022.955579
*   [8] ——, “Computational argumentation and cognition,” 2021.
*   [9] M. A. Qassas, D. Fogli, M. Giacomin, and G. Guida, “Analysis of clinical discussions based on argumentation schemes,” *Procedia Computer Science*, vol. 64, pp. 282–289, 2015, conference on ENTERprise Information Systems/International Conference on Project MANagement/Conference on Health and Social Care Information Systems and Technologies, CENTERIS/ProjMAN / HCist 2015 October 7-9, 2015\. [Online]. Available: https://www.sciencedirect.com/science/article/pii/S1877050915026265
*   [10] S. Hong, L. Xiao, and J. Chen, “An interaction model for merging multi-agent argumentation in shared clinical decision making,” in *2023 IEEE International Conference on Bioinformatics and Biomedicine (BIBM)*, 2023, pp. 4304–4311.
*   [11] Z. Zeng, Z. Shen, B. T. H. Tan, J. J. Chin, C. Leung, Y. Wang, Y. Chi, and C. Miao, “Explainable and Argumentation-based Decision Making with Qualitative Preferences for Diagnostics and Prognostics of Alzheimer’s Disease,” in *Proceedings of the 17th International Conference on Principles of Knowledge Representation and Reasoning*, 9 2020, pp. 816–826. [Online]. Available: https://doi.org/10.24963/kr.2020/84
*   [12] I. Sassoon, N. Kökciyan, S. Modgil, and S. Parsons, “Argumentation schemes for clinical decision support,” *Argument and Computation*, vol. 12, no. 3, pp. 329–355, Nov. 2021.
*   [13] K. Čyras, B. Delaney, D. Prociuk, F. Toni, M. Chapman, J. Domínguez, and V. Curcin, “Argumentation for explainable reasoning with conflicting medical recommendations,” *CEUR Workshop Proceedings*, vol. 2237, Jan. 2018, 2018 Joint Reasoning with Ambiguous and Conflicting Evidence and Recommendations in Medicine and the 3rd International Workshop on Ontology Modularity, Contextuality, and Evolution, MedRACER + WOMoCoE 2018 ; Conference date: 29-10-2018.
*   [14] G. Chen, L. Cheng, L. A. Tuan, and L. Bing, “Exploring the potential of large language models in computational argumentation,” 2023.
*   [15] F. Castagna, N. Kokciyan, I. Sassoon, S. Parsons, and E. Sklar, “Computational argumentation-based chatbots: a survey,” 2024.
*   [16] Y. Xiu, Z. Xiao, and Y. Liu, “LogicNMR: Probing the non-monotonic reasoning ability of pre-trained language models,” in *Findings of the Association for Computational Linguistics: EMNLP 2022*, Y. Goldberg, Z. Kozareva, and Y. Zhang, Eds.   Abu Dhabi, United Arab Emirates: Association for Computational Linguistics, Dec. 2022, pp. 3616–3626\. [Online]. Available: https://aclanthology.org/2022.findings-emnlp.265
*   [17] J. Xie, K. Zhang, J. Chen, T. Zhu, R. Lou, Y. Tian, Y. Xiao, and Y. Su, “Travelplanner: A benchmark for real-world planning with language agents,” 2024.
*   [18] Y. Zhang, J. Yang, Y. Yuan, and A. C.-C. Yao, “Cumulative reasoning with large language models,” 2023.
*   [19] L. Wang, C. Ma, X. Feng, Z. Zhang, H. Yang, J. Zhang, Z. Chen, J. Tang, X. Chen, Y. Lin, W. X. Zhao, Z. Wei, and J.-R. Wen, “A survey on large language model based autonomous agents,” 2023.
*   [20] W. Shi, R. Xu, Y. Zhuang, Y. Yu, J. Zhang, H. Wu, Y. Zhu, J. Ho, C. Yang, and M. D. Wang, “Ehragent: Code empowers large language models for few-shot complex tabular reasoning on electronic health records,” 2024.
*   [21] L. Pan, A. Albalak, X. Wang, and W. Y. Wang, “Logic-lm: Empowering large language models with symbolic solvers for faithful logical reasoning,” 2023.
*   [22] X. Tang, A. Zou, Z. Zhang, Z. Li, Y. Zhao, X. Zhang, A. Cohan, and M. Gerstein, “Medagents: Large language models as collaborators for zero-shot medical reasoning,” 2024.
*   [23] K. Gandhi, D. Sadigh, and N. D. Goodman, “Strategic reasoning with language models,” 2023.
*   [24] D. Jin, E. Pan, N. Oufattole, W.-H. Weng, H. Fang, and P. Szolovits, “What disease does this patient have? a large-scale open domain question answering dataset from medical exams,” 2020.
*   [25] Q. Jin, B. Dhingra, Z. Liu, W. W. Cohen, and X. Lu, “Pubmedqa: A dataset for biomedical research question answering,” 2019.
*   [26] P. M. Dung, “On the acceptability of arguments and its fundamental role in nonmonotonic reasoning, logic programming and n-person games,” *Artificial Intelligence*, vol. 77, no. 2, pp. 321–357, 1995\. [Online]. Available: https://www.sciencedirect.com/science/article/pii/000437029400041X
*   [27] T. Oliveira, J. Dauphin, K. Satoh, S. Tsumoto, and P. Novais, “Argumentation with goals for clinical decision support in multimorbidity,” in *Proceedings of the 17th International Conference on Autonomous Agents and MultiAgent Systems*, 2018, 2031–2033.
*   [28] D. Walton, C. Reed, and F. Macagno, *Argumentation Schemes*, C. Reed and F. Macagno, Eds.   New York: Cambridge University Press, 2008.
*   [29] D. Walton, *Argumentation Schemes for Presumptive Reasoning*, 1st ed.   Routledge, 1996\. [Online]. Available: https://doi.org/10.4324/9780203811160
*   [30] A. Creswell, M. Shanahan, and I. Higgins, “Selection-inference: Exploiting large language models for interpretable logical reasoning,” 2022.
*   [31] J. Wei, X. Wang, D. Schuurmans, M. Bosma, B. Ichter, F. Xia, E. Chi, Q. Le, and D. Zhou, “Chain-of-thought prompting elicits reasoning in large language models,” 2023.
*   [32] “ISO, IEC: AWI TS 29119-11: Software and systems engineering - software testing - part 11: Testing of AI systems,” Tech. Rep., 2020\. [Online]. Available: https://www.iso.org/standard/84127.html
*   [33] M. Liu, L. Xiao, Q. Wang, Z. Liu, R. Zhu, and M. He, “A study of medical decision recommendation generation and similarity fusion based on cdss and chatgpt-4,” in *2023 IEEE International Conference on Bioinformatics and Biomedicine (BIBM)*, 2023, pp. 873–878.
*   [34] G. Klyne and J. J. Carroll. (2004) Resource description framework (rdf): Concepts and abstract syntax. W3C Recommendation. W3C. [Online]. Available: http://www.w3.org/TR/2004/REC-rdf-concepts-20040210/
*   [35] Z. Bao, W. Chen, S. Xiao, K. Ren, J. Wu, C. Zhong, J. Peng, X. Huang, and Z. Wei, “Disc-medllm: Bridging general large language models and real-world medical consultation,” 2023.
*   [36] H. Nori, Y. T. Lee, S. Zhang, D. Carignan, R. Edgar, N. Fusi, N. King, J. Larson, Y. Li, W. Liu, R. Luo, S. M. McKinney, R. O. Ness, H. Poon, T. Qin, N. Usuyama, C. White, and E. Horvitz, “Can generalist foundation models outcompete special-purpose tuning? case study in medicine,” 2023.
*   [37] Q. Jin, Y. Yang, Q. Chen, and Z. Lu, “Genegpt: Augmenting large language models with domain tools for improved access to biomedical information,” 2023.
*   [38] T. Savage, A. Nayak, R. Gallo, and et al., “Diagnostic reasoning prompts reveal the potential for large language model interpretability in medicine,” *npj Digital Medicine*, vol. 7, p. 20, 2024\. [Online]. Available: https://doi.org/10.1038/s41746-024-01010-1
*   [39] K. Singhal, T. Tu, J. Gottweis, R. Sayres, E. Wulczyn, L. Hou, K. Clark, S. Pfohl, H. Cole-Lewis, D. Neal, M. Schaekermann, A. Wang, M. Amin, S. Lachgar, P. Mansfield, S. Prakash, B. Green, E. Dominowska, B. A. y Arcas, N. Tomasev, Y. Liu, R. Wong, C. Semturs, S. S. Mahdavi, J. Barral, D. Webster, G. S. Corrado, Y. Matias, S. Azizi, A. Karthikesalingam, and V. Natarajan, “Towards expert-level medical question answering with large language models,” 2023.
*   [40] C. Li, C. Wong, S. Zhang, N. Usuyama, H. Liu, J. Yang, T. Naumann, H. Poon, and J. Gao, “Llava-med: Training a large language-and-vision assistant for biomedicine in one day,” 2023.
*   [41] S. Pan, L. Luo, Y. Wang, C. Chen, J. Wang, and X. Wu, “Unifying large language models and knowledge graphs: A roadmap,” *IEEE Transactions on Knowledge and Data Engineering*, p. 1–20, 2024\. [Online]. Available: http://dx.doi.org/10.1109/TKDE.2024.3352100
*   [42] Q. Wu, G. Bansal, J. Zhang, Y. Wu, B. Li, E. Zhu, L. Jiang, X. Zhang, S. Zhang, J. Liu, A. H. Awadallah, R. W. White, D. Burger, and C. Wang, “Autogen: Enabling next-gen llm applications via multi-agent conversation,” 2023.
*   [43] J. Jung, L. Qin, S. Welleck, F. Brahman, C. Bhagavatula, R. L. Bras, and Y. Choi, “Maieutic prompting: Logically consistent reasoning with recursive explanations,” 2022.
*   [44] A. de Wynter and T. Yuan, “I wish to have an argument: Argumentative reasoning in large language models,” 2023.