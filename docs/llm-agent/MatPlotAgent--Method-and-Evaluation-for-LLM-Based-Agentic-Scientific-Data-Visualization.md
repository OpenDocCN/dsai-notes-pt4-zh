<!--yml
category: 未分类
date: 2025-01-11 12:53:01
-->

# MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization

> 来源：[https://arxiv.org/html/2402.11453/](https://arxiv.org/html/2402.11453/)

Zhiyu Yang${}^{*2}$  Zihan Zhou${}^{3}$  Shuo Wang${}^{{\dagger}1}$  Xin Cong${}^{1}$
Xu Han${}^{1}$  Yukun Yan${}^{1}$  Zhenghao Liu${}^{4}$  Zhixing Tan${}^{5}$
Pengyuan Liu${}^{2}$  Dong Yu${}^{2}$  Zhiyuan Liu${}^{1}$  Xiaodong Shi${}^{3}$ Maosong Sun${}^{1}$
${}^{1}$Tsinghua University  ${}^{2}$Beijing Language and Culture University  ${}^{3}$Xiamen University
${}^{4}$Northeastern University, China  ${}^{5}$Zhongguancun Laboratory, Beijing, China   Equal contribution.  Corresponding authors.

###### Abstract

Scientific data visualization plays a crucial role in research by enabling the direct display of complex information and assisting researchers in identifying implicit patterns. Despite its importance, the use of Large Language Models (LLMs) for scientific data visualization remains rather unexplored. In this study, we introduce MatPlotAgent, an efficient model-agnostic LLM agent framework designed to automate scientific data visualization tasks. Leveraging the capabilities of both code LLMs and multi-modal LLMs, MatPlotAgent consists of three core modules: query understanding, code generation with iterative debugging, and a visual feedback mechanism for error correction. To address the lack of benchmarks in this field, we present MatPlotBench, a high-quality benchmark consisting of 100 human-verified test cases. Additionally, we introduce a scoring approach that utilizes GPT-4V for automatic evaluation. Experimental results demonstrate that MatPlotAgent can improve the performance of various LLMs, including both commercial and open-source models. Furthermore, the proposed evaluation method shows a strong correlation with human-annotated scores.¹¹1  MatPlotAgent and MatPlotBench are be publicly available at [https://github.com/thunlp/MatPlotAgent](https://github.com/thunlp/MatPlotAgent).

![[Uncaptioned image]](img/8a7bfd784061ab086ea8ad974d9fef94.png)

MatPlotAgent: Method and Evaluation for
LLM-Based Agentic Scientific Data Visualization

Zhiyu Yang${}^{*2}$  Zihan Zhou^†^†thanks:   Equal contribution.${}^{3}$  Shuo Wang${}^{{\dagger}1}$  Xin Cong${}^{1}$ Xu Han${}^{1}$  Yukun Yan${}^{1}$  Zhenghao Liu${}^{4}$  Zhixing Tan${}^{5}$ Pengyuan Liu${}^{2}$  Dong Yu${}^{2}$  Zhiyuan Liu^†^†thanks:   Corresponding authors.${}^{1}$  Xiaodong Shi${}^{3}$ Maosong Sun${}^{1}$ ${}^{1}$Tsinghua University  ${}^{2}$Beijing Language and Culture University  ${}^{3}$Xiamen University ${}^{4}$Northeastern University, China  ${}^{5}$Zhongguancun Laboratory, Beijing, China

## 1 Introduction

A picture is worth a thousand words. Data visualization is an essential process in scientific research, facilitating the more direct conveyance of complex information and aiding researchers in uncovering implicit patterns. There are many advanced toolkits, such as Matplotlib²²2[https://matplotlib.org](https://matplotlib.org) and Origin³³3[https://www.originlab.com](https://www.originlab.com), that can help researchers plot various types of figures for complex data distributions. However, transforming raw data into informative and easy-to-understand visualizations is still time-consuming and labor-intensive. Before the invention of large language models (LLMs) OpenAI ([2023](https://arxiv.org/html/2402.11453v3#bib.bib20)), automating this process with AI models is almost impossible.

![Refer to caption](img/f9305eed32327b0f8c6905cea4535ddb.png)

Figure 1: Examples in the proposed MatPlotBench. Given the raw data and user queries, the AI agent is expected to generate a figure accordingly. We only display partial raw data and user queries due to space limitations.

With large-scale parameters and extensive training data, LLMs have demonstrated remarkable capabilities in a wide range of complex tasks, including reasoning Wei et al. ([2022](https://arxiv.org/html/2402.11453v3#bib.bib34)); Kojima et al. ([2022a](https://arxiv.org/html/2402.11453v3#bib.bib9)); Yao et al. ([2023a](https://arxiv.org/html/2402.11453v3#bib.bib39)), mathematics Yu et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib41)); Luo et al. ([2023a](https://arxiv.org/html/2402.11453v3#bib.bib17)); Azerbayev et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib2)); Shao et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib31)) and coding Rozière et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib28)); Luo et al. ([2023b](https://arxiv.org/html/2402.11453v3#bib.bib18)); Guo et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib8)); Wei et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib35)). This breakthrough has unlocked new opportunities for utilizing LLMs as autonomous agents in a diverse range of practical scenarios, such as web browsing Nakano et al. ([2021](https://arxiv.org/html/2402.11453v3#bib.bib19)); Yao et al. ([2022](https://arxiv.org/html/2402.11453v3#bib.bib38)); Qin et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib24)); Zhou et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib42)); Deng et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib6)); Yao et al. ([2023b](https://arxiv.org/html/2402.11453v3#bib.bib40)); Xie et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib36)), social simulations Park et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib21)); Xu et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib37)); Chen et al. ([2024a](https://arxiv.org/html/2402.11453v3#bib.bib4)); Wang et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib33)), tool utilization Qin et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib25)); Schick et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib30)); Liu et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib15)); Li et al. ([2023a](https://arxiv.org/html/2402.11453v3#bib.bib13)); Lu et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib16)); Qian et al. ([2023b](https://arxiv.org/html/2402.11453v3#bib.bib23)); Shinn et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib32)), and software development Qian et al. ([2023a](https://arxiv.org/html/2402.11453v3#bib.bib22)). Using LLMs to enhance human productivity in specialized areas is now a key research focus with great potential.

Recent advancements in LLM-based agents inspire us to explore the utilization of LLMs for scientific data visualization, a realm that remains rather unexplored in existing studies. A closely related line of research is text-to-image generation Ramesh et al. ([2021](https://arxiv.org/html/2402.11453v3#bib.bib26)); Saharia et al. ([2022](https://arxiv.org/html/2402.11453v3#bib.bib29)), where diffusion models Rombach et al. ([2022](https://arxiv.org/html/2402.11453v3#bib.bib27)) have shown great potential in generating various types of images. However, existing text-to-image generation methods predominantly focus on artistic expression, potentially misaligning with the needs of scientific data visualization, where clarity and precision in conveying information are the most important principles. This work aims to automatically generate figures with precise information.

We propose leveraging modern code LLMs and multi-modal LLMs to develop scientific data visualization agents that can significantly enhance human efficiency. The resulting MatPlotAgent⁴⁴4This name is in homage to the well-known Matplotlib. is comprised of three modules: (1) the query understanding that can thoroughly understand user-provided requirements; (2) the code generation module with iterative debugging capabilities that use code to precisely preprocess raw data and generate figures; and (3) the visual feedback module that possesses visual perceptual abilities to find errors in the plotted draft and provide visual feedback to the code generation module to rectify the errors. Our method is model-agnostic, which can be driven with any code LLMs and multi-modal LLMs. Through experiments, we find MatPlotAgent can work with both closed-source LLMs (e.g., GPT-4 OpenAI ([2023](https://arxiv.org/html/2402.11453v3#bib.bib20))) and open-source LLMs (e.g., Magicoder Wei et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib35))).

Another critical challenge in the field of automatic scientific data visualization is the absence of benchmarks for evaluation purposes. To address this issue, we introduce a meticulously crafted benchmark called MatPlotBench to quantitatively evaluate the approaches involved. Specifically, MatPlotBench contains 100 carefully hand-crafted test examples, each of which contains a user query, the corresponding input data, and a ground-truth figure verified by human experts. We believe that high-quality test sets play a crucial role in driving advancements in the field.

To facilitate automatic quantitative evaluation, we also design a scoring mechanism based on GPT-4V OpenAI ([2023](https://arxiv.org/html/2402.11453v3#bib.bib20)), which is one of the strongest multi-modal LLMs that can effectively understand text and figures. Specifically, GPT-4V is prompted to produce a score between 0 and 100 based on the ground-truth figure and the one generated by AI models. Additionally, we conduct human evaluation and estimate the correlation coefficient between human-annotated scores and the automatically calculated scores. The results reveal a strong correlation between the automatic score and the human-annotated score, thus affirming the reliability of the scoring mechanism. In summary, our contribution can be listed as follows:

*   •

    We introduce MatPlotBench to enable automatic quantitative evaluation of AI methods designed for scientific data visualization. Through comparison with human evaluation, we observe that MatPlotBench can effectively capture the performance of AI approaches in this cutting-edge task.

*   •

    We propose an effective and generalizable LLM agent framework, MatPlotAgent, that can improve the performance of a wide range of LLMs based on the newly proposed visual feedback mechanism.

## 2 Task Description

We first introduce the scientific data visualization task investigated in this work. Given a user query $\mathbf{x}$ described in text and the corresponding data $\mathcal{D}$, the AI system is expected to output a figure $V$ that can satisfy the user’s demand:

|  | $V=f(\mathbf{x},\mathcal{D}),$ |  | (1) |

where $f$ denotes the involved AI system that can be either an LLM or an LLM-based agent.

Specifically, $\mathbf{x}$ specifies the visualization requirements, encompassing the visualization type, data to plot, structural or spatial requirements for individual elements or the entire plot, and aesthetic preferences. $\mathcal{D}$ represents the data, a collection of data points $\left\{d_{1},\cdots,d_{n}\right\}$ whether specified by the user or stored in the external data file. Figure [1](https://arxiv.org/html/2402.11453v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") provides some examples for this task.

## 3 MatPlotBench

Automatic evaluation is important in AI tasks as it enables researchers to efficiently assess the performance of various methods, thereby guiding the development of the field. While the DS-1000 benchmark Lai et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib12)) includes coding problems about Matplotlib, the solutions’ average length is merely three lines, rendering them too simplistic to gauge the proficiency of contemporary AI agents in tackling practical challenges. Therefore, we propose to construct MatPlotBench with complex data visualization problems that are more close to real-world scenarios. We will illustrate the data collection process in Section [3.1](https://arxiv.org/html/2402.11453v3#S3.SS1 "3.1 Data Collection ‣ 3 MatPlotBench ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") and then explain the scoring mechanism in Section [3.2](https://arxiv.org/html/2402.11453v3#S3.SS2 "3.2 Automatic Quantitative Evaluation ‣ 3 MatPlotBench ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization").

### 3.1 Data Collection

#### Principles

To enhance the quality of MatPlotBench, we adhere to the following principles for data collection: (1) Covering diverse types: encompassing a broad range of plot types, including not only the most commonly used but also rare but useful ones; (2) Containing representative instances: ensuring that the test examples reflect the representative features of scientific data visualization, such as varying data complexity; and (3) Balancing easy and challenging problems: including problems of varying levels of difficulty in the benchmark.

#### Selecting Original Examples

In accordance with the principles outlined above, we first select some original examples from reputable online scientific data visualization forums. These examples are carefully selected from the Matplotlib Gallery and OriginLab GraphGallery, encompassing diverse and representative instances with varying levels of difficulty. Specifically, we select 1 or 2 examples from every section in the Matplotlib Gallery, covering bars, lines, markers, pie charts, polar plots, contour plots, statistics plots, 3D plots, text annotations, radar charts, shapes, scales, axes, spines, subplots, and so on. We also seek more advanced test examples from the OriginLab GraphGallery, focusing on those that are more aesthetically appealing or complex, such as Sankey diagrams, sunburst charts, radial plots, chord diagrams, streamplots, and others. Finally, 75 original examples come from the Matplotlib Gallery and the 25 other original examples come from the OriginLab GraphGallery. Subsequently, these examples undergo several modifications to become the final test cases in MatPlotBench.

#### Preliminary Query Generation

Based on the selected original examples, we use LLMs to generate preliminary queries, which are then revised by humans. For original examples from the Matplotlib Gallery, we use GPT-4 to convert the code in each original example into preliminary queries. For the examples from the OriginLab GraphGallery, there are only images. We thus use GPT-4V to convert each image into a preliminary query.

#### Data Replacement

Based on these preliminary queries, we begin data replacement for examples from the Matplotlib Gallery due to the observed phenomenon of memorization by GPT-4\. In this process, we replace the original data points with newly generated ones, while keeping other factors such as the plot type unchanged. For examples from OriginLab, we find that the data is inherently complex, and even GPT-4 does not exhibit memorization with these examples. As a result, we only perform data replacement for Matplotlib examples.

#### Human Modification

After completing the data replacement process, we engage human annotators to refine the preliminary queries. These annotators are tasked with correcting errors, eliminating ambiguity, and adding any omitted essential information. Each annotator involved has a minimum of three years of experience in coding and NLP. Furthermore, each query undergoes refinement by two independent human annotators.

#### Updating Ground-Truth Figures

After obtaining the human-annotated queries, as the data in Matplotlib examples are altered, we cannot directly use the images in the original example as the ground truth. To this end, we manually wrote code to plot the ground truth for the Matplotlib examples. For examples from OriginLab, as the data remains unaltered, we extract the images from their website to serve as the ground truth.

#### Human Verification

After obtaining the queries and their corresponding ground truths, we performed a final round of manual verification. Three NLP researchers were asked to conduct this verification. In this turn, the focus is mainly on checking whether the user queries and the ground truths are well aligned. The researchers meticulously checked each element in the ground truth image and looked for their corresponding descriptions in the user query. Ill-described elements and those missing clarifications are corrected. Redundant and incorrect descriptions are removed. This process results in 100 high-quality (query, raw data, ground-truth figure) triples, which comprise our final benchmark.

### 3.2 Automatic Quantitative Evaluation

To ease the burden of manual evaluation and broaden the applicability of our benchmark for research purposes, we suggest employing GPT-4V, a cutting-edge multi-modal LLM, to conduct automatic evaluations on our proposed benchmark. We carefully prompt GPT-4V to give a score from 0 to 100 on model-generated visualizations using the corresponding ground truths as the reference. The prompt is shown in Figure [6](https://arxiv.org/html/2402.11453v3#A1.F6 "Figure 6 ‣ A.1 Evaluation Prompts ‣ Appendix A Detailed Prompts ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") in Appendix.

#### Correlation with Human Evaluation

To assess the reliability of GPT-4V as an automatic evaluator for scientific visualizations, we calculate the correlation between the automatic scores and human-evaluated scores. Specifically, we employ GPT-3.5 and GPT-4 to generate figures on MatPlotBench, and then conduct both automatic and human evaluation for the generated figures. For each model, we iteratively sample a subset that consists of $n$ examples from the total benchmark, and then calculate the average score of both automatic and human evaluation. This process repeats $k$ times and we get $k$ data points for each type of evaluation, which can be represented by $\mathcal{A}=\{a_{1},\cdots,a_{k}\}$ and $\mathcal{H}=\{h_{1},\cdots,h_{k}\}$. $a_{i}$ denotes the average automatic score on the $i$-th randomly sampled subset, and $h_{i}$ represents the average human-evaluated score in the same subset. $n$ and $k$ are set to 25 and 100, respectively.

![Refer to caption](img/116a112963ee268b4000452670259e3e.png)

Figure 2: Correlation between the proposed automatic evaluation mechanism and human evaluation.

We utilize the statistical functions provided by scipy⁵⁵5[https://docs.scipy.org/doc/scipy/reference/stats.html](https://docs.scipy.org/doc/scipy/reference/stats.html) to compute the Pearson correlation coefficient $r$ and the corresponding p-value $p$. For GPT-4, we obtain $r$=0.876 and $p$=7.41e-33, while for GPT-3.5, the values are $r$=0.836 and $p$=2.67e-27\. Figure [2](https://arxiv.org/html/2402.11453v3#S3.F2 "Figure 2 ‣ Correlation with Human Evaluation ‣ 3.2 Automatic Quantitative Evaluation ‣ 3 MatPlotBench ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") shows the data points for GPT-4\. Given that $r>0.8$ and $p<$0.05, we conclude that the automatic evaluation scores are strongly correlated with human evaluation results. This demonstrates the reliability of the proposed scoring mechanism in assessing the quality of model-generated figures on MatPlotBench.

## 4 MatPlotAgent

![Refer to caption](img/eebfdce9dcf67032673877a23dff97a5.png)

Figure 3: Workflow of MatPlotAgent: The query expansion module converts the user query into detailed multi-step instructions. These instructions are then passed to the code agent, which generates the plotting code. The visual agent provides informative feedback based on the current draft, guiding the refinement of the figure.

To improve the capabilities of LLMs for scientific data visualization, we propose an agentic framework that mimics the plotting process of human experts. The proposed MatPlotAgent is comprised of three modules, including the query expansion module, the code agent, and the visual agent. Figure [3](https://arxiv.org/html/2402.11453v3#S4.F3 "Figure 3 ‣ 4 MatPlotAgent ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") illustrates the workflow of MatPlotAgent.

### 4.1 Query Expansion

The query expansion module interprets and refines the user query, converting the high-level requirements into a sequence of explicit and detailed instructions that are easy for LLMs to follow. This module can also be viewed as a planning module, creating an overall plan before generating the figure. Specifically, this module is based on the involved code LLM, which is prompted to give detailed instructions on how to use code to fulfill the requirement specified by the user, including what libraries to import, what library functions to call, how to set the parameters in each function correctly, how to prepare the data, how to manipulate the data, and so on.

### 4.2 Code Agent

The code agent is the core component in MatPlotAgent, responsible for generating the code to plot figures. Given detailed instructions from the query expansion module, the code agent first generates the code using appropriate libraries and functions. To improve the success rate of the generated code, we also employ the self-debugging mechanism Chen et al. ([2024b](https://arxiv.org/html/2402.11453v3#bib.bib5)), which helps the involved code LLM iteratively identify and correct bugs in the code. To prevent an infinite loop, we set the maximum iterations of self-debugging to 3.

Similar to humans, who need to repeatedly refine the figure based on current drafts, we also introduce a visual feedback mechanism. This mechanism employs multi-modal LLMs to provide suggestions to improve the figure and better fulfill the user’s queries. These suggestions, which we call visual feedback, are then provided to the code agent to further improve the code. Our experiments in Section [5.2](https://arxiv.org/html/2402.11453v3#S5.SS2 "5.2 Main Results ‣ 5 Experiments ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") demonstrate that MatPlotAgent is compatible with several modern code LLMs, including both some well-known closed-source models and some open-source models.

| Model | Direct | Zero-Shot | MatPlotAgent |
| Decod. | CoT | w/ GPT-4V |
| GPT-4 | 48.86 | 45.42 | $-$3.44 | 61.16 | $+$12.30 |
| GPT-3.5 | 38.03 | 37.14 | $-$0.89 | 47.51 | $+$9.48 |
| Magicoder-S-DS-6.7B Wei et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib35)) | 38.49 | 37.95 | $-$0.54 | 51.70 | $+$13.21 |
| Deepseek-coder-6.7B-instruct Guo et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib8)) | 31.53 | 29.16 | $-$2.37 | 39.45 | $+$7.92 |
| CodeLlama-34B-Instruct Rozière et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib28)) | 16.54 | 12.40 | $-$4.14 | 14.18 | $-$2.36 |
| Deepseek-coder-33B-instruct Guo et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib8)) | 30.88 | 36.10 | $+$5.22 | 32.18 | $+$1.30 |
| WizardCoder-Python-33B-V1.1 Luo et al. ([2023b](https://arxiv.org/html/2402.11453v3#bib.bib18)) | 36.94 | 35.81 | $-$1.13 | 45.96 | $+$9.02 |

Table 1: Performance of different LLMs on MatPlotBench. For each model, improvements over the direct decoding are highlighted in red, while results worse than that of the direct decoding are highlighted in blue.

| Model | Direct | MatPlotAgent |
| Decod. | w/ Gemini Pro Vision |
| GPT-4 | 48.86 |  | 56.73 | $+$7.87 |
| GPT-3.5 | 38.03 |  | 43.48 | $+$5.45 |

Table 2: Performance of GPT-4 and GPT-3.5 using Gemini Pro Vision as visual agent on MatPlotBench.

### 4.3 Visual Agent

The major difference between MatPlotAgent and previous LLM-based coding agents Qian et al. ([2023a](https://arxiv.org/html/2402.11453v3#bib.bib22)); Chen et al. ([2024b](https://arxiv.org/html/2402.11453v3#bib.bib5)) is that we take the visual signal into account, which is important in scientific data visualization. Some errors or weaknesses may be difficult to identify in the code but become apparent when observing the output figure through “eyes”. The visual agent is the “eyes” for MatPlotAgent, while the aforementioned code agent acts as the “hands” for MatPlotAgent.

Specifically, the visual agent is powered by multi-modal LLMs. We introduce several guiding principles for the visual agent, including verifying whether the figure aligns with the provided data, and enhancing the colors or labels to improve the figure’s informativeness. Based on the principles, the user query, and the current draft of the figure, the visual agent generates some suggestions to refine to figure. These suggestions serve as feedback for the code agent to refine the code. Experimental results in Section [5.4](https://arxiv.org/html/2402.11453v3#S5.SS4 "5.4 Ablation Study ‣ 5 Experiments ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") show that our visual feedback mechanism can significantly improve the quality of the plotted figures.

## 5 Experiments

### 5.1 Setup

#### Models

Since the proposed MatPlotAgent is model-agnostic, we can employ various LLMs in this framework. The code LLMs we use in our experiments include GPT-4, GPT-3.5, Magicoder-S-DS-6.7B Wei et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib35)), Deepseek-coder-6.7B-instruct Guo et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib8)), Deepseek-coder-33B-instruct Guo et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib8)), WizardCoder-Python-33B-V1.1 Luo et al. ([2023b](https://arxiv.org/html/2402.11453v3#bib.bib18)), and CodeLlama-34B-Instruct Rozière et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib28)). The decoding temperature is set to 0.0 for all the involved code LLMs. For GPT-4 and GPT-3.5, we use the API provided by OpenAI⁶⁶6[https://openai.com/product](https://openai.com/product). For the other five open-source LLMs, we use vLLM Kwon et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib11)) for model inference. For the visual agent, we utilize GPT-4V OpenAI ([2023](https://arxiv.org/html/2402.11453v3#bib.bib20)) and Gemini Pro Vision Google ([2023](https://arxiv.org/html/2402.11453v3#bib.bib7)), two representative multi-modal LLMs. We leave the exploration of using open-source multi-modal LLMs to power the visual agent for future work.

#### Evaluation

We evaluate the involved methods on MatPlotBench, using the proposed automatic scoring mechanism that is shown reliable in Section [3.2](https://arxiv.org/html/2402.11453v3#S3.SS2 "3.2 Automatic Quantitative Evaluation ‣ 3 MatPlotBench ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization"). For each code LLM, we evaluate its performance in three ways:

*   •

    Direct decoding: given the query, the model directly generates the plotting code.

*   •

    Zero-Shot Chain-of-thought Kojima et al. ([2022b](https://arxiv.org/html/2402.11453v3#bib.bib10)): the model is prompted to inference with the zero-shot CoT mechanism.

*   •

    MatPlotAgent: the model is equipped with the proposed MatPlotAgent framework, driving the query expansion module and the code agent, as illustrated in Section [4](https://arxiv.org/html/2402.11453v3#S4 "4 MatPlotAgent ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization").

| Model | Accuracy of Code Execution Results (%) |
| Visualization-Hard | Visualization-Easy | Average |
| GPT-4 | 66.7 | 60.8 | 63.8 |
|  + MatPlotAgent | 72.6 | 68.4 | 70.5 |
|      w/o Visual Feedback | 66.7 | 65.8 | 66.3 |

Table 3: Effect of MatPlotAgent on the visualization subset of the Qwen-Agent Code Interpreter benchmark.

### 5.2 Main Results

Table [1](https://arxiv.org/html/2402.11453v3#S4.T1 "Table 1 ‣ 4.2 Code Agent ‣ 4 MatPlotAgent ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") presents the results of different methods on the scientific data visualization task. In the direct decoding setting, GPT-4 achieves the highest score of 48.86\. Surprisingly, the open-source model Magicoder-S-DS-6.7B Wei et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib35)) achieves the second-best performance, surpassing models with substantially larger parameter sizes, such as WizardCoder-Python-33B-V1.1.

The results also suggest that the zero-shot CoT mechanism does not effectively enhance the performance of many recent code LLMs. Zero-shot CoT only improves the results of Deepseek-coder-33B-instruct Guo et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib8)) from 30.88 to 36.10\. Conversely, for other models, implementing zero-shot CoT results in poorer performance. For example, when zero-shot CoT is applied, the performance of GPT-4 drops to 45.42, which is lower than the direct decoding result of 48.86.

From Table [1](https://arxiv.org/html/2402.11453v3#S4.T1 "Table 1 ‣ 4.2 Code Agent ‣ 4 MatPlotAgent ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization"), we find the proposed MatPlotAgent can improve the plotting capabilities of several models. For GPT-4 and GPT-3.5, MatPlotAgent leads to significant improvements of 12.30 and 9.48, respectively. For the other five open-source LLMs, MatPlotAgent improves the performance of four models. With MatPlotAgent, the open-source Magicoder-S-DS-6.7B model even surpasses GPT-4 with direct decoding (51.70 vs. 48.86), showcasing the effectiveness of our method.

To investigate the generalizability of MatPlotAgent across various multi-modal LLMs, we present the results of employing Gemini Pro Vision as the visual agent in Table [2](https://arxiv.org/html/2402.11453v3#S4.T2 "Table 2 ‣ 4.2 Code Agent ‣ 4 MatPlotAgent ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization"). We observe considerable improvements of 7.87 and 5.45, respectively, over the direct decoding baseline. This evidence further demonstrates the model-agnostic characteristic of our approach, leveraging various multi-modal LLMs to achieve enhanced performance.

![Refer to caption](img/5b2074b82b55ae2c4403d4764198ab9d.png)

Figure 4: Examples to illustrate the effect of visual feedback. To investigate the effect of the visual feedback mechanism on different models, we display the outputs of two representative LLMs. Case A, B, and C are generated by GPT-4\. Case D is generated by Magicoder-S-DS-6.7B.

### 5.3 Results on Qwen-Agent Code Interpreter Benchmark

In Table [3](https://arxiv.org/html/2402.11453v3#S5.T3 "Table 3 ‣ Evaluation ‣ 5.1 Setup ‣ 5 Experiments ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization"), we detail the performance of MatPlotAgent on the visualization subset of the Qwen-Agent Code Interpreter Benchmark⁷⁷7[https://github.com/QwenLM/Qwen-Agent/tree/main/benchmark](https://github.com/QwenLM/Qwen-Agent/tree/main/benchmark), which was recently published. According to their GitHub repository, GPT-4 achieved scores of 66.7 and 60.8 on the Visualization-Hard and Visualization-Easy subsets, respectively. Utilizing MatPlotAgent, we attained higher scores of 72.62 and 68.35 on these subsets. When the visual feedback mechanism is disabled, MatPlotAgent reached scores of 66.67 and 65.82, reconfirming the necessity of visual feedback.

### 5.4 Ablation Study

Compared to previous LLM-based coding agents Qian et al. ([2023a](https://arxiv.org/html/2402.11453v3#bib.bib22)); Chen et al. ([2024b](https://arxiv.org/html/2402.11453v3#bib.bib5)), the major contribution of the work lies in the newly proposed visual feedback mechanism, expected to leverage visual signals to enhance the quality of the output figure. To gain a deeper understanding of the impact of the visual feedback mechanism, we conduct both qualitative and quantitative analyses in this section.

Figure [4](https://arxiv.org/html/2402.11453v3#S5.F4 "Figure 4 ‣ 5.2 Main Results ‣ 5 Experiments ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") presents examples plotted by LLMs both with and without the visual feedback mechanism. We observe a clear improvement in the quality of the output figure with the visual feedback. For example, in case C, the text in the figure is jumbled, but this issue is resolved with the assistance of visual feedback. It is important to note that the visual agent does not reference the ground-truth figure when generating feedback; it only examines the draft plotted by the model. Table [4](https://arxiv.org/html/2402.11453v3#S5.T4 "Table 4 ‣ 5.4 Ablation Study ‣ 5 Experiments ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") also presents quantitative results of the visual feedback mechanism, indicating that the absence of visual feedback would result in significantly poorer outcomes for both GPT-4 and GPT-3.5\. This reaffirms the importance of visual signals in the task of scientific data visualization.

| Model | GPT-4 | GPT-3.5 |
| Direct Decod. | 48.86 | 38.03 |
| MatPlotAgent | 61.16 | 47.51 |
|     w/o Visual Feedback | 53.44 | 41.57 |

Table 4: Effect of the visual feedback mechanism (GPT-4V visual agent).

### 5.5 Case Study

![Refer to caption](img/35785b5adfd25ba88232facd5ad9f297.png)

Figure 5: Case study of different models.

We present output figures in Figure [5](https://arxiv.org/html/2402.11453v3#S5.F5 "Figure 5 ‣ 5.5 Case Study ‣ 5 Experiments ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization"). The first example is relatively simple, correctly plotted by GPT-4 augmented with MatPlotAgent. The second example is more challenging; while GPT-4 and Magicoder-S-DS-6.7B can generate a draft, both omit some elements. The third example is the most difficult, where none of the three models can produce the correct result. These results indicate that the proposed MatPlotBench poses a significant challenge for current LLMs. Even the state-of-the-art LLM, GPT-4, equipped with MatPlotAgent, fails in some cases. We believe this benchmark will be effective not only for evaluating AI systems in scientific data visualization but also for assessing general capabilities such as coding and visual perception.

## 6 Related Work

#### Code LLMs

Since the release of Codex Chen et al. ([2021](https://arxiv.org/html/2402.11453v3#bib.bib3)), many closed- and open-source code LLMs have been published, pushing the boundaries of LLMs’ capabilities to write functional code. Early open-source efforts include SantaCoder Allal et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib1)) and StarCoder Li et al. ([2023b](https://arxiv.org/html/2402.11453v3#bib.bib14)). More recently, the Code Llama Rozière et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib28)) series is released, including models of varying sizes. DeepSeekCoder Guo et al. ([2024](https://arxiv.org/html/2402.11453v3#bib.bib8)), a series of open-source code models ranging in size from 1.3B to 33B, has also garnered significant attention for its impressive performance on general coding benchmarks. Wei et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib35)) introduce a novel data augmentation method for automatically creating high-quality fine-tuning data. The resulting Magicoder model surpasses a wide array of open-source code LLMs in performance.

#### LLM Agents

Recently, a wide range of LLM-based agent frameworks is proposed to explore LLMs’ potential in real-world scenarios Nakano et al. ([2021](https://arxiv.org/html/2402.11453v3#bib.bib19)); Yao et al. ([2022](https://arxiv.org/html/2402.11453v3#bib.bib38)); Qin et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib24)); Zhou et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib42)). OpenAgents Xie et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib36)) proposed an open platform that leverages LLM agents in everyday situation by employing a Data Agent, a Plugins Agent, and a Web Agent. Park et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib21)) proposed an interactive simulation of human behavior in which software agents emulate realistic human actions and interactions through computation. Voyager Wang et al. ([2023](https://arxiv.org/html/2402.11453v3#bib.bib33)) introduced the fisrt LLM model-driven autonomous agent in Minecraft, designed to perpetually explore the environment, master various skills, and uncover new insights independently, without any human guidance. ChatDev Qian et al. ([2023a](https://arxiv.org/html/2402.11453v3#bib.bib22)) proposed creating a virtual, chat-driven software development enterprise that follows the traditional waterfall methodology. In this study, we explore the capabilities of LLM-based agents in the task of scientific data visualization, a critical and practical area for contemporary researchers.

## 7 Conclusion

We propose to assess and enhance the capabilities of modern LLMs for scientific data visualization, a multifaceted task demanding coding and visual skills. We begin with the creation of MatPlotBench, a rigorous benchmark supporting automated quantitative evaluation that strongly aligns with human assessment. Additionally, we introduce MatPlotAgent, a model-agnostic mechanism employing visual feedback to enhance LLMs’ plotting abilities. Experimental results demonstrate that MatPlotAgent enhances the performance of various LLMs.

## 8 Limitations

In this paper, we introduce MatPlotBench, a benchmark designed for scientific data visualization. However, the demands of scientific data visualization can vary significantly across disciplines. Since MatPlotBench is developed for general scientific data visualization, it may not encompass all domain-specific requirements, potentially restricting its applicability to certain fields. In the future, the data construction and evaluation approaches can be customized for specific domains if necessary.

## References

*   Allal et al. (2023) Loubna Ben Allal, Raymond Li, Denis Kocetkov, Chenghao Mou, Christopher Akiki, Carlos Munoz Ferrandis, Niklas Muennighoff, Mayank Mishra, Alex Gu, Manan Dey, Logesh Kumar Umapathi, Carolyn Jane Anderson, Yangtian Zi, Joel Lamy Poirier, Hailey Schoelkopf, Sergey Troshin, Dmitry Abulkhanov, Manuel Romero, Michael Lappert, Francesco De Toni, Bernardo García del Río, Qian Liu, Shamik Bose, Urvashi Bhattacharyya, Terry Yue Zhuo, Ian Yu, Paulo Villegas, Marco Zocca, Sourab Mangrulkar, David Lansky, Huu Nguyen, Danish Contractor, Luis Villa, Jia Li, Dzmitry Bahdanau, Yacine Jernite, Sean Hughes, Daniel Fried, Arjun Guha, Harm de Vries, and Leandro von Werra. 2023. [Santacoder: don’t reach for the stars!](http://arxiv.org/abs/2301.03988)
*   Azerbayev et al. (2024) Zhangir Azerbayev, Hailey Schoelkopf, Keiran Paster, Marco Dos Santos, Stephen Marcus McAleer, Albert Q. Jiang, Jia Deng, Stella Biderman, and Sean Welleck. 2024. [Llemma: An open language model for mathematics](https://openreview.net/forum?id=4WnqRR915j). In *The Twelfth International Conference on Learning Representations*.
*   Chen et al. (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, Alex Ray, Raul Puri, Gretchen Krueger, Michael Petrov, Heidy Khlaaf, Girish Sastry, Pamela Mishkin, Brooke Chan, Scott Gray, Nick Ryder, Mikhail Pavlov, Alethea Power, Lukasz Kaiser, Mohammad Bavarian, Clemens Winter, Philippe Tillet, Felipe Petroski Such, Dave Cummings, Matthias Plappert, Fotios Chantzis, Elizabeth Barnes, Ariel Herbert-Voss, William Hebgen Guss, Alex Nichol, Alex Paino, Nikolas Tezak, Jie Tang, Igor Babuschkin, Suchir Balaji, Shantanu Jain, William Saunders, Christopher Hesse, Andrew N. Carr, Jan Leike, Josh Achiam, Vedant Misra, Evan Morikawa, Alec Radford, Matthew Knight, Miles Brundage, Mira Murati, Katie Mayer, Peter Welinder, Bob McGrew, Dario Amodei, Sam McCandlish, Ilya Sutskever, and Wojciech Zaremba. 2021. [Evaluating large language models trained on code](http://arxiv.org/abs/2107.03374).
*   Chen et al. (2024a) Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chi-Min Chan, Heyang Yu, Yaxi Lu, Yi-Hsin Hung, Chen Qian, Yujia Qin, Xin Cong, Ruobing Xie, Zhiyuan Liu, Maosong Sun, and Jie Zhou. 2024a. [Agentverse: Facilitating multi-agent collaboration and exploring emergent behaviors](https://openreview.net/forum?id=EHg5GDnyq1). In *The Twelfth International Conference on Learning Representations*.
*   Chen et al. (2024b) Xinyun Chen, Maxwell Lin, Nathanael Schärli, and Denny Zhou. 2024b. [Teaching large language models to self-debug](https://openreview.net/forum?id=KuPixIqPiq). In *The Twelfth International Conference on Learning Representations*.
*   Deng et al. (2023) Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samuel Stevens, Boshi Wang, Huan Sun, and Yu Su. 2023. [Mind2web: Towards a generalist agent for the web](https://openreview.net/forum?id=kiYqbO3wqw). In *Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track*.
*   Google (2023) Gemini Team Google. 2023. [Gemini: A family of highly capable multimodal models](http://arxiv.org/abs/2312.11805).
*   Guo et al. (2024) Daya Guo, Qihao Zhu, Dejian Yang, Zhenda Xie, Kai Dong, Wentao Zhang, Guanting Chen, Xiao Bi, Y. Wu, Y. K. Li, Fuli Luo, Yingfei Xiong, and Wenfeng Liang. 2024. [Deepseek-coder: When the large language model meets programming – the rise of code intelligence](http://arxiv.org/abs/2401.14196).
*   Kojima et al. (2022a) Takeshi Kojima, Shixiang (Shane) Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. 2022a. Large language models are zero-shot reasoners. In *Advances in Neural Information Processing Systems*, volume 35, pages 22199–22213.
*   Kojima et al. (2022b) Takeshi Kojima, Shixiang (Shane) Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. 2022b. [Large language models are zero-shot reasoners](https://proceedings.neurips.cc/paper_files/paper/2022/file/8bb0d291acd4acf06ef112099c16f326-Paper-Conference.pdf). In *Advances in Neural Information Processing Systems*.
*   Kwon et al. (2023) Woosuk Kwon, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng, Cody Hao Yu, Joseph E. Gonzalez, Hao Zhang, and Ion Stoica. 2023. Efficient memory management for large language model serving with pagedattention. In *Proceedings of the ACM SIGOPS 29th Symposium on Operating Systems Principles*.
*   Lai et al. (2023) Yuhang Lai, Chengxi Li, Yiming Wang, Tianyi Zhang, Ruiqi Zhong, Luke Zettlemoyer, Wen-Tau Yih, Daniel Fried, Sida Wang, and Tao Yu. 2023. DS-1000: A natural and reliable benchmark for data science code generation. In *Proceedings of the 40th International Conference on Machine Learning*.
*   Li et al. (2023a) Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. 2023a. [CAMEL: Communicative agents for ”mind” exploration of large language model society](https://openreview.net/forum?id=3IyL2XWDkG). In *Thirty-seventh Conference on Neural Information Processing Systems*.
*   Li et al. (2023b) Raymond Li, Loubna Ben allal, Yangtian Zi, Niklas Muennighoff, Denis Kocetkov, Chenghao Mou, Marc Marone, Christopher Akiki, Jia LI, Jenny Chim, Qian Liu, Evgenii Zheltonozhskii, Terry Yue Zhuo, Thomas Wang, Olivier Dehaene, Joel Lamy-Poirier, Joao Monteiro, Nicolas Gontier, Ming-Ho Yee, Logesh Kumar Umapathi, Jian Zhu, Ben Lipkin, Muhtasham Oblokulov, Zhiruo Wang, Rudra Murthy, Jason T Stillerman, Siva Sankalp Patel, Dmitry Abulkhanov, Marco Zocca, Manan Dey, Zhihan Zhang, Urvashi Bhattacharyya, Wenhao Yu, Sasha Luccioni, Paulo Villegas, Fedor Zhdanov, Tony Lee, Nadav Timor, Jennifer Ding, Claire S Schlesinger, Hailey Schoelkopf, Jan Ebert, Tri Dao, Mayank Mishra, Alex Gu, Carolyn Jane Anderson, Brendan Dolan-Gavitt, Danish Contractor, Siva Reddy, Daniel Fried, Dzmitry Bahdanau, Yacine Jernite, Carlos Muñoz Ferrandis, Sean Hughes, Thomas Wolf, Arjun Guha, Leandro Von Werra, and Harm de Vries. 2023b. [Starcoder: may the source be with you!](https://openreview.net/forum?id=KoFOg41haE) *Transactions on Machine Learning Research*. Reproducibility Certification.
*   Liu et al. (2024) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, and Jie Tang. 2024. [Agentbench: Evaluating LLMs as agents](https://openreview.net/forum?id=zAdUB0aCTQ). In *The Twelfth International Conference on Learning Representations*.
*   Lu et al. (2023) Pan Lu, Baolin Peng, Hao Cheng, Michel Galley, Kai-Wei Chang, Ying Nian Wu, Song-Chun Zhu, and Jianfeng Gao. 2023. [Chameleon: Plug-and-play compositional reasoning with large language models](https://openreview.net/forum?id=HtqnVSCj3q). In *Thirty-seventh Conference on Neural Information Processing Systems*.
*   Luo et al. (2023a) Haipeng Luo, Qingfeng Sun, Can Xu, Pu Zhao, Jianguang Lou, Chongyang Tao, Xiubo Geng, Qingwei Lin, Shifeng Chen, and Dongmei Zhang. 2023a. Wizardmath: Empowering mathematical reasoning for large language models via reinforced evol-instruct. *arXiv preprint arXiv:2308.09583*.
*   Luo et al. (2023b) Ziyang Luo, Can Xu, Pu Zhao, Qingfeng Sun, Xiubo Geng, Wenxiang Hu, Chongyang Tao, Jing Ma, Qingwei Lin, and Daxin Jiang. 2023b. [Wizardcoder: Empowering code large language models with evol-instruct](http://arxiv.org/abs/2306.08568).
*   Nakano et al. (2021) Reiichiro Nakano, Jacob Hilton, Suchir Balaji, Jeff Wu, Ouyang Long, Christina Kim, Christopher Hesse, Shantanu Jain, Vineet Kosaraju, William Saunders, Xu Jiang, Karl Cobbe, Tyna Eloundou, Gretchen Krueger, Kevin Button, Matthew Knight, Benjamin Chess, and John Schulman. 2021. [Webgpt: Browser-assisted question-answering with human feedback](https://api.semanticscholar.org/CorpusID:245329531). *ArXiv*, abs/2112.09332.
*   OpenAI (2023) OpenAI. 2023. [Gpt-4 technical report](https://api.semanticscholar.org/CorpusID:257532815).
*   Park et al. (2023) Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. 2023. [Generative agents: Interactive simulacra of human behavior](https://doi.org/10.1145/3586183.3606763). In *Proceedings of the 36th Annual ACM Symposium on User Interface Software and Technology*, UIST ’23, New York, NY, USA. Association for Computing Machinery.
*   Qian et al. (2023a) Chen Qian, Xin Cong, Wei Liu, Cheng Yang, Weize Chen, Yusheng Su, Yufan Dang, Jiahao Li, Juyuan Xu, Dahai Li, Zhiyuan Liu, and Maosong Sun. 2023a. [Communicative agents for software development](http://arxiv.org/abs/2307.07924).
*   Qian et al. (2023b) Cheng Qian, Chi Han, Yi Fung, Yujia Qin, Zhiyuan Liu, and Heng Ji. 2023b. [CREATOR: Tool creation for disentangling abstract and concrete reasoning of large language models](https://doi.org/10.18653/v1/2023.findings-emnlp.462). In *Findings of the Association for Computational Linguistics: EMNLP 2023*, pages 6922–6939, Singapore. Association for Computational Linguistics.
*   Qin et al. (2023) Yujia Qin, Zihan Cai, Dian Jin, Lan Yan, Shihao Liang, Kunlun Zhu, Yankai Lin, Xu Han, Ning Ding, Huadong Wang, Ruobing Xie, Fanchao Qi, Zhiyuan Liu, Maosong Sun, and Jie Zhou. 2023. [WebCPM: Interactive web search for Chinese long-form question answering](https://doi.org/10.18653/v1/2023.acl-long.499). In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 8968–8988, Toronto, Canada. Association for Computational Linguistics.
*   Qin et al. (2024) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, Sihan Zhao, Lauren Hong, Runchu Tian, Ruobing Xie, Jie Zhou, Mark Gerstein, dahai li, Zhiyuan Liu, and Maosong Sun. 2024. [ToolLLM: Facilitating large language models to master 16000+ real-world APIs](https://openreview.net/forum?id=dHng2O0Jjr). In *The Twelfth International Conference on Learning Representations*.
*   Ramesh et al. (2021) Aditya Ramesh, Mikhail Pavlov, Gabriel Goh, Scott Gray, Chelsea Voss, Alec Radford, Mark Chen, and Ilya Sutskever. 2021. [Zero-shot text-to-image generation](http://arxiv.org/abs/2102.12092).
*   Rombach et al. (2022) Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, and Björn Ommer. 2022. [High-resolution image synthesis with latent diffusion models](http://arxiv.org/abs/2112.10752).
*   Rozière et al. (2024) Baptiste Rozière, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Romain Sauvestre, Tal Remez, Jérémy Rapin, Artyom Kozhevnikov, Ivan Evtimov, Joanna Bitton, Manish Bhatt, Cristian Canton Ferrer, Aaron Grattafiori, Wenhan Xiong, Alexandre Défossez, Jade Copet, Faisal Azhar, Hugo Touvron, Louis Martin, Nicolas Usunier, Thomas Scialom, and Gabriel Synnaeve. 2024. [Code llama: Open foundation models for code](http://arxiv.org/abs/2308.12950).
*   Saharia et al. (2022) Chitwan Saharia, William Chan, Saurabh Saxena, Lala Li, Jay Whang, Emily Denton, Seyed Kamyar Seyed Ghasemipour, Burcu Karagol Ayan, S. Sara Mahdavi, Rapha Gontijo Lopes, Tim Salimans, Jonathan Ho, David J Fleet, and Mohammad Norouzi. 2022. [Photorealistic text-to-image diffusion models with deep language understanding](http://arxiv.org/abs/2205.11487).
*   Schick et al. (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessi, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, and Thomas Scialom. 2023. [Toolformer: Language models can teach themselves to use tools](https://openreview.net/forum?id=Yacmpz84TH). In *Thirty-seventh Conference on Neural Information Processing Systems*.
*   Shao et al. (2024) Zhihong Shao, Peiyi Wang, Qihao Zhu, Runxin Xu, Junxiao Song, Mingchuan Zhang, Y. K. Li, Y. Wu, and Daya Guo. 2024. [Deepseekmath: Pushing the limits of mathematical reasoning in open language models](http://arxiv.org/abs/2402.03300).
*   Shinn et al. (2023) Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik R Narasimhan, and Shunyu Yao. 2023. [Reflexion: language agents with verbal reinforcement learning](https://openreview.net/forum?id=vAElhFcKW6). In *Thirty-seventh Conference on Neural Information Processing Systems*.
*   Wang et al. (2023) Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi (Jim) Fan, and Anima Anandkumar. 2023. [Voyager: An open-ended embodied agent with large language models](https://api.semanticscholar.org/CorpusID:258887849). *ArXiv*, abs/2305.16291.
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, brian ichter, Fei Xia, Ed Chi, Quoc V Le, and Denny Zhou. 2022. [Chain-of-thought prompting elicits reasoning in large language models](https://proceedings.neurips.cc/paper_files/paper/2022/file/9d5609613524ecf4f15af0f7b31abca4-Paper-Conference.pdf). In *Advances in Neural Information Processing Systems*, volume 35, pages 24824–24837\. Curran Associates, Inc.
*   Wei et al. (2023) Yuxiang Wei, Zhe Wang, Jiawei Liu, Yifeng Ding, and Lingming Zhang. 2023. [Magicoder: Source code is all you need](http://arxiv.org/abs/2312.02120).
*   Xie et al. (2023) Tianbao Xie, Fan Zhou, Zhoujun Cheng, Peng Shi, Luoxuan Weng, Yitao Liu, Toh Jing Hua, Junning Zhao, Qian Liu, Che Liu, Leo Z. Liu, Yiheng Xu, Hongjin Su, Dongchan Shin, Caiming Xiong, and Tao Yu. 2023. [Openagents: An open platform for language agents in the wild](http://arxiv.org/abs/2310.10634).
*   Xu et al. (2023) Yuzhuang Xu, Shuo Wang, Peng Li, Fuwen Luo, Xiaolong Wang, Weidong Liu, and Yang Liu. 2023. [Exploring large language models for communication games: An empirical study on werewolf](http://arxiv.org/abs/2309.04658).
*   Yao et al. (2022) Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. 2022. [WebShop: Towards Scalable Real-World Web Interaction with Grounded Language Agents](https://proceedings.neurips.cc/paper_files/paper/2022/file/82ad13ec01f9fe44c01cb91814fd7b8c-Paper-Conference.pdf). In *Advances in Neural Information Processing Systems*, volume 35, pages 20744–20757\. Curran Associates, Inc.
*   Yao et al. (2023a) Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, and Karthik R Narasimhan. 2023a. [Tree of thoughts: Deliberate problem solving with large language models](https://openreview.net/forum?id=5Xc1ecxO1h). In *Thirty-seventh Conference on Neural Information Processing Systems*.
*   Yao et al. (2023b) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, and Yuan Cao. 2023b. [React: Synergizing reasoning and acting in language models](https://openreview.net/forum?id=WE_vluYUL-X). In *The Eleventh International Conference on Learning Representations*.
*   Yu et al. (2024) Longhui Yu, Weisen Jiang, Han Shi, Jincheng YU, Zhengying Liu, Yu Zhang, James Kwok, Zhenguo Li, Adrian Weller, and Weiyang Liu. 2024. [Metamath: Bootstrap your own mathematical questions for large language models](https://openreview.net/forum?id=N8N0hgNDRt). In *The Twelfth International Conference on Learning Representations*.
*   Zhou et al. (2023) Shuyan Zhou, Frank F. Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, Uri Alon, and Graham Neubig. 2023. [Webarena: A realistic web environment for building autonomous agents](https://openreview.net/forum?id=rmiwIL98uQ). In *Second Agent Learning in Open-Endedness Workshop*.

## Appendix A Detailed Prompts

To better understand MatPlotBench and MatPlotAgent, we list the prompts for automatic evaluation and the three modules in MatPlotAgent, including the query expansion module, the code agent, and the visual agent.

### A.1 Evaluation Prompts

The automatic evaluation prompt primarily requires GPT-4V to provide a score between 0 and 100 for the model-generated plot, with reference to the ground truth plot.

<svg class="ltx_picture" height="390.09" id="A1.F6.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,390.09) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="362.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are an excellent judge at evaluating visualization plots between a model-generated plot and the ground truth. You will be giving scores on how well it matches the ground truth plot. The generated plot will be given to you as the first figure. If the first figure is blank, that means the code failed to generate a figure. Another plot will be given to you as the second figure, which is the desired outcome of the user query, meaning it is the ground truth for you to reference. Please compare the two figures head to head and rate them. Suppose the second figure has a score of 100, rate the first figure on a scale from 0 to 100. Scoring should be carried out regarding the plot correctness: Compare closely between the generated plot and the ground truth, the more resemblance the generated plot has compared to the ground truth, the higher the score. The score should be proportionate to the resemblance between the two plots. In some rare occurrences, see if the data points are generated randomly according to the query, if so, the generated plot may not perfectly match the ground truth, but it is correct nonetheless. Only rate the first figure, the second figure is only for reference. If the first figure is blank, that means the code failed to generate a figure. Give a score of 0 on the Plot correctness. After scoring from the above aspect, please give a final score. The final score is preceded by the [FINAL SCORE] token. For example [FINAL SCORE]: 40.</foreignobject></g></g></svg>

Figure 6: Automatic evaluation prompt for GPT-4V.

### A.2 Prompts for MatPlotAgent

The query expansion prompt mainly requires LLMs to generate step-by-step, detailed instructions on how to use Python code to fulfill the requirements specified by users, as shown in Figure [7](https://arxiv.org/html/2402.11453v3#A1.F7 "Figure 7 ‣ A.2 Prompts for MatPlotAgent ‣ Appendix A Detailed Prompts ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization").

For the code agent, there are two prompts for the code generation process and the self-debugging mechanism. The code generation prompt mainly requires LLMs to generate executable code according to the user query to plot and save the output figure, as shown in Figure [8](https://arxiv.org/html/2402.11453v3#A1.F8 "Figure 8 ‣ A.2 Prompts for MatPlotAgent ‣ Appendix A Detailed Prompts ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization"). The self-debugging prompt mainly requires LLMs to correct the buggy code according to the error message from a Python interpreter, as displayed in Figure [9](https://arxiv.org/html/2402.11453v3#A1.F9 "Figure 9 ‣ A.2 Prompts for MatPlotAgent ‣ Appendix A Detailed Prompts ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization").

The visual agent prompt mainly requires multi-modal LLMs to firstly understand the user query and analyze the draft plot, then generate the visual feedback to refine the draft, as shown in Figure [10](https://arxiv.org/html/2402.11453v3#A1.F10 "Figure 10 ‣ A.2 Prompts for MatPlotAgent ‣ Appendix A Detailed Prompts ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization").

<svg class="ltx_picture" height="205.9" id="A1.F7.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,205.9) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="178.34" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">SYSTEM PROMPT: According to the user query, expand and solidify the query into a step by step detailed instruction (or comment) on how to write python code to fulfill the user query’s requirements. Import the appropriate libraries. Pinpoint the correct library functions to call and set each parameter in every function call accordingly. USER PROMPT: Here is the user query: [User Query]: """ {{query}} """ You should understand what the query’s requirements are, and output step by step, detailed instructions on how to use python code to fulfill these requirements. Include what libraries to import, what library functions to call, how to set the parameters in each function correctly, how to prepare the data, how to manipulate the data so that it becomes appropriate for later functions to call etc,. Make sure the code to be executable and correctly generate the desired output in the user query.</foreignobject></g></g></svg>

Figure 7: The query expansion prompt in MatPlotAgent.

<svg class="ltx_picture" height="157.63" id="A1.F8.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,157.63) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="130.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">SYSTEM PROMPT: You are a cutting-edge super capable code generation LLM. You will be given a natural language query, generate a runnable python code to satisfy all the requirements in the query. You can use any python library you want. When you complete a plot, remember to save it to a png file. USER PROMPT: Here is the query: """ {{query}} """ If the query requires data manipulation from a csv file, process the data from the csv file and draw the plot in one piece of code. When you complete a plot, remember to save it to a png file. The file name should be """{{file_name}}""".</foreignobject></g></g></svg>

Figure 8: The code generation prompt in MatPlotAgent.

 <svg class="ltx_picture" height="73.07" id="A1.F9.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,73.07) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">USER PROMPT: There are some errors in the code you gave: {{error_message}} please correct the errors. Then give the complete code and don’t omit anything even though you have given it in the above code.</foreignobject></g></g></svg> 

Figure 9: The self-debugging prompt in MatPlotAgent.

<svg class="ltx_picture" height="273.86" id="A1.F10.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,273.86) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="246.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">SYSTEM PROMPT: Given a user query and an image of the current plot, please determine whether the plot has faithfully followed the user query. Your task is to provide instruction to make sure the plot has strictly completed the requirements of the query. Please output a detailed step by step instruction on how to use python code to enhance the plot. USER PROMPT: Here is the user query: [Query]: """ {{query}} """ Carefully read and analyze the user query to understand the specific requirements. Check if the plot aligns with the user query in terms of data selection, plot type, and any specific customization. Look at the provided image of the plot. Assess the plot type, the data it represents, labels, titles, colors, and any other visual elements. Compare these elements with the requirements specified in the user query. Note any differences between the user query requirements and the current plot. Based on the identified discrepancies, provide step-by-step instructions on how to modify the Python code to meet the user query requirements. Suggest improvements for better visualization practices, such as clarity, readability, and aesthetics, while ensuring the primary focus is on meeting the user’s specified requirements. Remember to save the plot to a png file. The file name should be """{{file_name}}"""</foreignobject></g></g></svg>

Figure 10: Prompt for the visual agent.

## Appendix B Human Evaluation Details

We engage human annotators from computer science departments at various universities via social media. They are compensated for their work at a rate slightly higher than the prevailing market rate. All human annotators involved are informed that the collected data will be used solely for academic research purposes, and their personal information will not be disclosed.

### B.1 Evaluation Guide for Human Annotators

Figure [11](https://arxiv.org/html/2402.11453v3#A2.F11 "Figure 11 ‣ B.1 Evaluation Guide for Human Annotators ‣ Appendix B Human Evaluation Details ‣ MatPlotAgent: Method and Evaluation for LLM-Based Agentic Scientific Data Visualization") gives detailed instructions for human annotators when scoring the model-generated plots.

<svg class="ltx_picture" height="287.47" id="A2.F11.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,287.47) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="240.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Plot Correctness (0-100 points) • Exact Match (90-100 points): The generated plot is nearly identical to the ground truth, with only minor, negligible differences. • High Resemblance (70-89 points): The generated plot closely resembles the ground truth with some small but noticeable differences in data representation or styling. • Moderate Resemblance (50-69 points): The generated plot has a moderate level of similarity to the ground truth, but there are several noticeable differences that impact the plot’s accuracy or interpretation. • Low Resemblance (30-49 points): The generated plot shares some similarities with the ground truth but has significant differences that change the overall message or interpretation of the data. • Poor Match (10-29 points): The generated plot has very little in common with the ground truth, with major discrepancies in data representation. • No Resemblance (1-9 points): The generated plot is completely different from the ground truth, with no discernible similarities in data representation. • Failure to Generate (0 points): The first figure is blank, indicating a failure to generate any plot. Special Considerations • In cases where the generated plot includes random data points that are correct in the context of the query, the plot should be evaluated for its correctness based on the query’s intent, not solely on its visual match to the ground truth. [FINAL SCORE]: XX</foreignobject></g></g></svg>

Figure 11: Evaluation guide for human annotators when scoring the model-generated plots.