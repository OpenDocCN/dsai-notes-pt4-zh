<!--yml
category: 未分类
date: 2025-01-11 11:49:45
-->

# Enhancing LLMs for Impression Generation in Radiology Reports through a Multi-Agent System

> 来源：[https://arxiv.org/html/2412.06828/](https://arxiv.org/html/2412.06828/)

Fang Zeng Department of Radiology, Massachusetts General Hospital and Harvard Medical School Zhiliang Lyu Department of Radiology, Massachusetts General Hospital and Harvard Medical School Quanzheng Li Department of Radiology, Massachusetts General Hospital and Harvard Medical School Xiang Li Department of Radiology, Massachusetts General Hospital and Harvard Medical School

###### Abstract

This study introduces "RadCouncil," a multi-agent Large Language Model (LLM) framework designed to enhance the generation of impressions in radiology reports from the finding section. RadCouncil comprises three specialized agents: 1) a "Retrieval" Agent that identifies and retrieves similar reports from a vector database, 2) a "Radiologist" Agent that generates impressions based on the finding section of the given report plus the exemplar reports retrieved by the Retrieval Agent, and 3) a "Reviewer" Agent that evaluates the generated impressions and provides feedback. The performance of RadCouncil was evaluated using both quantitative metrics (BLEU, ROUGE, BERTScore) and qualitative criteria assessed by GPT-4, using chest X-ray as a case study. Experiment results show improvements in RadCouncil over the single-agent approach across multiple dimensions, including diagnostic accuracy, stylistic concordance, and clarity. This study highlights the potential of utilizing multiple interacting LLM agents, each with a dedicated task, to enhance performance in specialized medical tasks and the development of more robust and adaptable healthcare AI solutions.

## 1 Introduction

In radiology workflow, radiologists traditionally interpret imaging studies and manually draft detailed reports, including an "impression" section that summarizes clinically significant findings and possible diagnosis, which is a vital part of the report for referring physicians and patient care. This process is time-consuming and subject to variability impacted by the radiologist’s knowledge and experience [[1](https://arxiv.org/html/2412.06828v1#bib.bib1)]. Automated impression generation has the potential to improve report consistency, reduce radiologist workload, and enhance the overall quality of radiology reports  [[2](https://arxiv.org/html/2412.06828v1#bib.bib2)]. Such a feature is especially needed with the recent growth in the demands for medical imaging, which are straining radiologists, leading to possible burnout and impacting their ability to provide timely and precise reports [[3](https://arxiv.org/html/2412.06828v1#bib.bib3)].

Large Language Models (LLMs) have shown exceptional capabilities in understanding and generating text that is coherent and contextually relevant, making them promising tools for auto-generating impressions from findings in radiology reports. A few studies have investigated LLMs’ ability for the impression summarization task [[2](https://arxiv.org/html/2412.06828v1#bib.bib2), [4](https://arxiv.org/html/2412.06828v1#bib.bib4), [5](https://arxiv.org/html/2412.06828v1#bib.bib5), [6](https://arxiv.org/html/2412.06828v1#bib.bib6), [7](https://arxiv.org/html/2412.06828v1#bib.bib7), [8](https://arxiv.org/html/2412.06828v1#bib.bib8)], demonstrating the potential of LLMs to revolutionize radiology workflow by automating the report generation process. Various techniques, such as prompt engineering, model fine-tuning, and retrieval-augmented generation (RAG) [[6](https://arxiv.org/html/2412.06828v1#bib.bib6)], have been used to improve their performance and mitigate the limitations of LLMs, including hallucinations and insufficient domain knowledge. However, current approaches often lack the ability to integrate these techniques together within a unified framework and the scalability to incorporate new techniques and/or requirements in impression generation. Furthermore, applying LLM, even with domain adaptation techniques, for text generation without reflecting and reviewing the output of the model risks introducing hallucinations that could compromise the reliability of the results and lead to serious errors in clinical decision-making [[9](https://arxiv.org/html/2412.06828v1#bib.bib9), [10](https://arxiv.org/html/2412.06828v1#bib.bib10)].

Recently, multi-agent systems have emerged as a promising approach for various natural language processing tasks [[11](https://arxiv.org/html/2412.06828v1#bib.bib11)]. By leveraging the strengths of multiple specialized agents, each focusing on specific tasks, a multi-agent system can enhance the overall quality and reliability of the model performance. In addition, techniques that are used to enhance the performance of a single LLM can be readily applied to each agent and the whole system. Thanks to the advanced natural language and code/structured data processing capability of LLMs, multi-agent systems can understand model inputs in generalizable forms, write and execute codes, and perform reasoning to orchestrate the collaborative process, effectively "wrapping" any models in a more intelligent and robust form. With efficient memory management among agents, the multi-agent system can mitigate the limitations of the context window size, thus improving the handling of extensive and complex tasks for more accurate and relevant outputs. Furthermore, through the collaboration among LLM agents, the multi-agent system can combine diverse capabilities from models with different domain expertise, potentially leading to emergent properties [[12](https://arxiv.org/html/2412.06828v1#bib.bib12)]. Some preliminary attempts have been made to test multi-agent frameworks in healthcare settings, such as clinical trial outcome prediction [[13](https://arxiv.org/html/2412.06828v1#bib.bib13)], diagnosis from case reports [[14](https://arxiv.org/html/2412.06828v1#bib.bib14)], medical education [[15](https://arxiv.org/html/2412.06828v1#bib.bib15)], and medical question answering [[16](https://arxiv.org/html/2412.06828v1#bib.bib16), [17](https://arxiv.org/html/2412.06828v1#bib.bib17)].

In this study, we develop a multi-agent framework named “RadCouncil” to generate the impression section in radiology reports based on the finding section, using reports from chest X-ray imaging as the study case. RadCouncil consists of three specialized agents: (1) a Retrieval agent that searches for similar reports from an external database, (2) a Radiologist agent that generates the impression section from the finding section with reference to the retrieved reports, and (3) a Review agent that evaluates the generated impression and provides feedback to the Radiologist agent with possible revisions. The main contribution of this work is twofold: RadCouncil presents an innovative multi-agent approach that mimics the workflow in radiology clinical practice, incorporating key steps of report writing, reference to documents, and review by self/peers. Secondly, the demonstrated effectiveness of RadCouncil in analyzing and generating specialized medical text shows the potential of multi-agent systems across a broad spectrum of medical applications, extending beyond radiology to other healthcare domains where complex task coordination and interplay among different roles are crucial.

## 2 Methodology

![Refer to caption](img/40c56bd5a38202c893c07c7163689201.png)

Figure 1: Diagram of the proposed RadCouncil framework, illustrating the interactions between the three agents, the report database, and the user.

### 2.1 RadCouncil: Overview and Workflow

In this work, we introduce a multi-agent system for generating radiology report impressions, leveraging both the generative power of LLMs and established medical knowledge from existing reports. The system employs three specialized agents: a Report Retriever that uses vector similarity matching to find relevant historical reports, a Radiologist agent that generates impressions with reference to retrieved examples, and a Reviewer agent that ensures consistency between findings and generated impressions. This architecture not only helps maintain accuracy but also reduces hallucinations by grounding the generation process in real medical examples and cross-checking by multiple LLMs. An illustrative example of the RadCouncil’s workflow is shown in Table 1.

Table 1: Sample workflow of the RadCouncil system

| User | PROCEDURE NAME: XR CHEST PORTABLE. FINDINGS: Lines/tubes: ET tube tip is projected at the level of the carina with tip projected near the origin of the right mainstem bronchus. Evidence of embolization coils project over the right upper hemithorax. Lungs: Patchy ill-defined opacities in the right upper lung are again noted. No gross pulmonary edema. Pleura: There is no pleural effusion or pneumothorax. Heart and mediastinum: The heart and the mediastinum are unchanged. Bones: The thoracic skeleton is unchanged. |
| --- | --- |
| Retriever | Here are some examples: EXAMPLE 1:
PROCEDURE NAME: XR CHEST PORTABLE. FINDINGS: Lines/tubes: Endotracheal tube terminates below the carina in the proximal right mainstem bronchus. Lungs: Redemonstration of patchy ill-defined opacities in the right mid to upper lung. Focus of patchy opacity in the left base is unchanged. No pulmonary edema. Pleura: There is no pleural effusion or pneumothorax. Heart and mediastinum: The heart and the mediastinum are unchanged. Bones: The thoracic skeleton is unchanged.
IMPRESSION:
Endotracheal tube terminates in the proximal right mainstem bronchus. Unchanged patchy opacities in the right mid to upper lung representing pulmonary hemorrhage. Superimposing aspiration or pneumonia may also be considered.
EXAMPLE 2:
$\dots$ |
| Radiologist | IMPRESSION: Tubes as described above.
Patchy ill-defined opacities in the right upper lung likely reflect atelectasis, although concurrent aspiration and/or pneumonia cannot be excluded by radiograph.
Unchanged mild pulmonary edema not seen on this study, but previously noted.
No pleural effusion or pneumothorax. |
| Reviewer | Upon reviewing the ’FINDINGS’ and ’IMPRESSION’ sections, I noticed an inconsistency: * In the ’FINDINGS’ section, it is stated that "No gross pulmonary edema" is seen, whereas in the ’IMPRESSION’ section, it is mentioned that there is "unchanged mild pulmonary edema not seen on this study, but previously noted." This implies that pulmonary edema was present in a previous study, but it is not consistent with the current findings.
REVISION: YES |
| Radiologist | IMPRESSION: Tubes as described above.
Patchy ill-defined opacities in the right upper lung likely reflect atelectasis, although concurrent aspiration and/or pneumonia cannot be excluded by radiograph.
No evidence of pulmonary edema or pleural effusion/pneumothorax. |
| Reviewer | Upon reviewing the ’FINDINGS’ and ’IMPRESSION’ sections, I did not notice any inconsistencies: * The description of tubes in the ’IMPRESSION’ section is consistent with the ’FINDINGS’ section.
* The patchy ill-defined opacities in the right upper lung are described consistently in both sections.
* The absence of pulmonary edema and pleural effusion/pneumothorax is also described consistently in both sections.
REVISION: NO |

#### 2.1.1 Report Retriever

The Report Retriever agent retrieves similar reports from an external database implemented by a vector database. Using the procedure name and findings provided by the input, the report retriever finds the matching exemplar reports by converting the input into its vector representation and searching the external radiology report database based on similarity score (e.g., cosine similarity). It will then fetch the top k similar vectors, where k is a predefined parameter balancing the retrieval efficiency and comprehensiveness.

#### 2.1.2 Radiologist

The Radiologist agent is responsible for generating the impression based on the procedure name and findings provided by the user. This agent utilizes the exemplar reports retrieved by the Report Retriever to augment its initial generation, improving the written style, the significance of the findings, and their interpretation. If a revision request is made by the Reviewer agent, the Radiologist agent will revise the previously generated impression based on the reviewer’s comments and feedback.

#### 2.1.3 Reviewer

The reviewer agent is designed to examine the consistency between the generated impression and the provided findings, ensuring that the generated impression aligns with each of the findings. If any inconsistencies are identified, the reviewer requests revisions from the radiologist, leading to the next iteration of the workflow. Otherwise, if no inconsistencies are found, the reviewer confirms that no revisions are needed, and the workflow ends with the generated impression as the final output. While other types of errors, such as missing clinically significant findings or misinterpretations, may exist, our study specifically addresses the potential consistency errors. Checking and revising the consistency error could also significantly reduce the hallucinations in the generated impressions, enhancing the trustworthiness of the system.

### 2.2 Radiology Report Database

The radiology report database serves as an external resource to support impression generation with exemplar reports through RAG. It houses a collection of well-curated radiology reports encompassing detailed information on procedure names, findings, and impressions. For each report, the procedure name and findings are converted into vector representations. This transformation enables efficient retrieval and analysis of relevant reports, especially when the database is large.

## 3 Experiments and Results

### 3.1 Dataset

A total of 1,900 chest X-ray reports were collected from Massachusetts General Hospital between January 2018 and February 2018\. Each report consists of three sections: procedure name, findings, and impression. From this dataset, 100 reports were randomly selected for model evaluation. The remaining 1,800 reports serve as the external database for RAG. The impression section in the raw dataset may contain information on report communication and management, which were detected and removed by an LLM (Llama-3.1-70b) [[18](https://arxiv.org/html/2412.06828v1#bib.bib18)].

### 3.2 Implementation Details

Both the Radiologist and Reviewer agents are implemented by LLMs (Llama-3.1-70b) [[18](https://arxiv.org/html/2412.06828v1#bib.bib18)] developed by Meta. Llama-3.1-70b is currently one of the leading open-source LLMs for text generation. The Report Retriever, a non-LLM agent, employs Faiss as the vector database, with vectorization performed by the GTE-base embedding model [[19](https://arxiv.org/html/2412.06828v1#bib.bib19)]. The top 10 similar reports (i.e., k=10) are retrieved from the external database based on the provided procedure name and findings. To prevent possible infinite review-revise loops and task drift, we limit the number of communication rounds to three. The system prompts for the Radiologist and Reviewer agents can be found in Appendix A.

### 3.3 Memory Management

Considering the limitations of the context window and the computational burden of LLMs, the memory capabilities of both the Radiologist and Reviewer agents are constrained. During the initial round of interaction, the radiologist retains the target procedure name and findings provided by the user, as well as the exemplar reports retrieved by the report retriever, to generate the initial impression. On the other hand, the Reviewer agent is only aware of the task provided by the user and the initial impression generated by the Radiologist agent for review. In subsequent rounds of interaction, the memory of both the Radiologist and the Reviewer agents is limited to their last round of communication and the task provided by the user. Exemplar reports retrieved earlier or previous rounds of conversations will not influence their ongoing interaction.

### 3.4 Performance of RadCouncil

The performance of RadCouncil was evaluated using standard metrics for text generation tasks, including BLEU [[20](https://arxiv.org/html/2412.06828v1#bib.bib20)], ROUGE-1, ROUGE-2, ROUGE-L [[21](https://arxiv.org/html/2412.06828v1#bib.bib21)], and BERTScore [[22](https://arxiv.org/html/2412.06828v1#bib.bib22)]. These scores mainly focus on distance-based similarity (e.g., word matching) between the generated and ground truth impressions. The performance of RadCouncil is compared to a single-agent LLM (Llama-3.1-70b), which represents a radiologist operating without the support of exemplar reports or the reviewer. As listed in Table 2, RadCouncil outperformed the single-agent LLM across all these metrics.

In addition to these metrics, RadCouncil was also evaluated using GPT-4o developed by OpenAI to evaluate, focusing on the semantic and clinical accuracy of the generated impression. The evaluation was based on the following qualitative criteria: inclusion of clinically significant findings, consistency with original findings, potential diagnosis, stylistic concordance, and conciseness and clarity. Each criterion was scored on a scale from 1 to 10, where a higher score indicates better quality. The system prompts for evaluation can be found in Appendix B. The results of this evaluation are summarized in Table 3\. The results show that both the single-agent and multi-agent systems performed equally well in identifying clinically significant findings (both 8.51 on a 1-10 scale)in the generated impression, demonstrating their ability to understand and process languages in radiology. In the task of inferring potential diagnoses, RadCouncil outperformed the single-agent LLM (8.24 vs. 8.15). RadCouncil also showed improvements in stylistic concordance (8.53 vs. 7.80) and conciseness/clarity (8.77 vs. 7.93) over single-agent LLM, possibly because of the support from retrieved exemplar reports. We also observed that the single-agent LLM achieved a better consistency score (8.77) than RadCouncil (8.63). We will analyze and discuss the inconsistency issue of RadCouncil in the next section.

Table 2: Performance of RadCouncil compared with single LLMs by quantitative metrics

| Model | BLEU | ROUGE_1 | ROUGE_2 | ROUGE_L | BERTScore |
| --- | --- | --- | --- | --- | --- |
| Single LLM (Llama-3.1-70b) | 8.21 | 0.3490 | 0.1391 | 0.2745 | 0.6643 |
| RadCouncil | 24.22 | 0.5049 | 0.3088 | 0.4364 | 0.7434 |

Table 3: The performance of multi-agent system compared with single agent using GPT-4 qualitative criteria.

| Model | Findings | Consistency | Diagnosis | Style | Conciseness |
| --- | --- | --- | --- | --- | --- |
| Single LLM (Llama-3.1-70b) | 8.51 | 8.77 | 8.15 | 7.80 | 7.93 |
| RadCouncil | 8.51 | 8.63 | 8.24 | 8.53 | 8.77 |

### 3.5 Analysis of the RAG performance

![Refer to caption](img/f6d29c0ee059a0ecb5b6c5b760ce2f90.png)

Figure 2: Example comparison of original impression generated by radiologist agent only vs. impression enhanced by report retriever agent.

Figure 2 shows an example of the original impression generated by the Radiologist agent alone and the enhanced impression with the support of the Retriever agent. This example demonstrates the benefit of RAG to make the generated impression more closely aligned with the ground truth. The enhanced impression shows improved stylistic concordance with typical radiological reporting practice. It is more concise and focused, eliminating extraneous details while retaining key findings. The accuracy of the potential diagnosis is also improved, as the final impression precisely identifies the condition as "small airways disease" and specifically mentions "bronchiolitis and reactive airways disease," perfectly aligning with the ground truth impression. With similar reports provided by the report retriever agent, the radiologist has the potential to generate more standardized and diagnostically accurate impressions.

On the other hand, providing additional information (in the form of exemplar reports) in the prompt to the Radiologist could result in inconsistent summaries in the generated impression. We manually reviewed the results of all 100 test cases, and analyzed five types of inconsistencies between the provided findings and generated impressions in various scenarios, as listed in Table 4\. The five types of inconsistencies include Presence, Progression Status, Severity, Size, and Location of the findings. The "Presence" inconsistency refers to the cases where findings in the generated impression that are not mentioned in the input findings or the existence of the findings is contradictory to common sense. The "Progression Status" inconsistency indicates a mismatch between the temporal changes of lesion condition described in the findings and how they are represented in the impression (e.g., stating "worsening" in the impression when the findings indicate "stable" or "improved" conditions). The "Severity" inconsistency refers to cases where the degree or intensity of a finding is described differently. The "Size" inconsistency represents discrepancies in the dimensional descriptions of lesions. The "Location" inconsistency occurs when there are contradictions or mismatches in the anatomical location of findings. The analysis results show a notable increase in inconsistency of the impressions generated with the support from the Retriever agent (i.e., via RAG). In the impressions generated by the Radiologist agent alone, inconsistencies were relatively minimal, with only two inconsistencies (one on progression status and one on size) found in two cases. In contrast, impressions generated with RAG showed more inconsistencies across multiple categories. Among the 100 analyzed cases, ten of the impressions generated using RAG contained inconsistencies. There were seven instances of presence inconsistency, six instances of progression status inconsistency, one instance of severity inconsistency, and one instance of location inconsistency. Specifically, we found that RAG tends to generate unmentioned findings and add unmentioned progression status (especially “unchanged”) to the impression. This analysis demonstrates that while the use of RAG will enhance the overall quality of the generation, it could also introduce more inconsistencies, possibly due to the much longer context window for the exemplar reports. This suggests that further refinement and calibration for RAG would be necessary to reduce these inconsistencies and improve the reliability and trustworthiness.

Table 4: Summary of five types of inconsistencies in the generated impression by radiologist agent and the enhanced impression by report retriever.

| Model | Inconsistencies | Presence | Status | Severity | Size | Location |
| --- | --- | --- | --- | --- | --- | --- |
| Single-agent (Radiologist) | 2 | 0 | 1 | 0 | 1 | 0 |
| Multi-agent |  |  |  |  |  |  |
|    + Retriever | 10 | 7 | 6 | 0 | 1 | 1 |
|    + Retriever & Reviewer | 6 | 0 | 6 | 0 | 1 | 1 |

### 3.6 Effectiveness of the Reviewer agent

To reduce inconsistencies in the generated impression, we implemented a Reviewer agent with prompt input consisting of only the original finding section and the generated impression. Our premise is that if the inconsistencies are caused by the long context window of the Radiologist agent, the Reviewer agent with a much simpler context can potentially identify the errors and make the corresponding revision suggestions. In this work, the Reviewer agent only focuses on the "Consistency" issue of the generated impression, which is the most critical problem of RadCouncil. As listed in the last row of Table 4, the Reviewer agent correctly identified and resolved inconsistencies in four cases, with most cases resolved in two rounds of interactions between the Radiologist and the Reviewer. The Reviewer correctly identified all the inconsistency errors regarding the Presence type, while it could not identify other types of inconsistencies. Specifically, regarding the "Status", the Reviewer agent failed to identify the wrongfully-generated “unchanged” conclusions in the impression. It explained that the absence of a changed status implies the condition is unchanged, which is incorrect according to radiological reporting standards, as the "unchanged" status must be explicitly supported by prior comparison studies mentioned in the findings. This limitation suggests that the Reviewer agent may benefit from additional training specifically on temporal relationships and status changes in radiological findings.

## 4 Conclusion and Discussion

This study presents RadCouncil, a multi-agent framework for generating radiology report impressions based on the input finding section and procedure name. Experiment results demonstrate significant improvements over single-agent approaches. Through quantitative metrics (BLEU, ROUGE, BERTScore) and qualitative evaluation by GPT-4, RadCouncil showed enhanced performance in diagnostic accuracy, stylistic concordance, and clarity of expression, aligning with recent findings by Sun et al. [[2](https://arxiv.org/html/2412.06828v1#bib.bib2)] on the potential of LLMs in radiology workflows. As healthcare systems continue to face increasing demands and workload pressures [[3](https://arxiv.org/html/2412.06828v1#bib.bib3)], AI-assisted documentation tools such as the RadCouncil developed in this work could play a vital role in maintaining high-quality patient care while supporting healthcare provider efficiency.

The three-agent architecture of RadCouncil, combining a Report Retriever for similar case identification, a Radiologist for impression generation, and a Reviewer for consistency checking, proved effective in producing more standardized and accurate impressions. This success supports the emerging consensus that multi-agent systems can effectively handle complex medical tasks through specialized role distribution, as shown in similar healthcare applications [[13](https://arxiv.org/html/2412.06828v1#bib.bib13), [14](https://arxiv.org/html/2412.06828v1#bib.bib14), [15](https://arxiv.org/html/2412.06828v1#bib.bib15)].

The demonstrated effectiveness of RadCouncil, despite its limitations, suggests potential applications beyond radiology to other healthcare domains where complex task coordination and specialized expertise are crucial for accurate documentation and decision-making. We envision that multi-agent systems will be increasingly utilized in healthcare AI development, especially those involving sophisticated reasoning and domain expertise [[16](https://arxiv.org/html/2412.06828v1#bib.bib16), [17](https://arxiv.org/html/2412.06828v1#bib.bib17)].

We found that the retrieval-augmented generation (RAG) approach particularly improved the stylistic alignment with radiological reporting conventions and diagnostic precision, extending the findings of Ma et al. [[6](https://arxiv.org/html/2412.06828v1#bib.bib6)]. However, our analysis revealed important challenges in using RAG, as it could introduce additional inconsistencies. This aligns with concerns raised by Omiye et al. [[9](https://arxiv.org/html/2412.06828v1#bib.bib9)] regarding LLM hallucinations in medical contexts. To mitigate this problem, we implemented the Reviewer agent, which showed promising performance in identifying and correcting such inconsistencies. Findings from this study suggest several promising avenues for advancing both radiology applications and broader healthcare applications of multi-agent systems: 1) Further improvements of the Radiologist and Reviewer agent’s capabilities, particularly in temporal reasoning and status change detection, are much needed. This could build upon the analysis of longitudinal radiology reports by these agents similar to the clinical practice. 2) To address the context window limitations in our current RAG implementation that can lead to information overflow and inconsistencies, future work is needed to develop more advanced memory management techniques such as hierarchical retrieval strategies and dynamic context prioritization [[23](https://arxiv.org/html/2412.06828v1#bib.bib23), [24](https://arxiv.org/html/2412.06828v1#bib.bib24)].

## References

*   [1] S Mussurakis, DL Buckley, AM Coady, LW Turnbull, and A Horsman. Observer variability in the interpretation of contrast enhanced mri of the breast. The British journal of radiology, 69(827):1009–1016, 1996.
*   [2] Zhaoyi Sun, Hanley Ong, Patrick Kennedy, Liyan Tang, Shirley Chen, Jonathan Elias, Eugene Lucas, George Shih, and Yifan Peng. Evaluating gpt-4 on impressions generation in radiology reports. Radiology, 307(5):e231259, 2023.
*   [3] Giles Maskell. Why does demand for medical imaging keep rising?, 2022.
*   [4] Luc Builtjes, Monique Brink, Souraya Belkhir, Bram van Ginneken, and Alessa Hering. Evaluating chatgpt’s performance in generating and assessing dutch radiology report impressions. In Medical Imaging with Deep Learning, 2024.
*   [5] Lu Zhang, Mingqian Liu, Lingyun Wang, Yaping Zhang, Xiangjun Xu, Zhijun Pan, Yan Feng, Jue Zhao, Lin Zhang, Gehong Yao, et al. Constructing a large language model to generate impressions from findings in radiology reports. Radiology, 312(3):e240885, 2024.
*   [6] Chong Ma, Zihao Wu, Jiaqi Wang, Shaochen Xu, Yaonai Wei, Zhengliang Liu, Fang Zeng, Xi Jiang, Lei Guo, Xiaoyan Cai, et al. An iterative optimizing framework for radiology report summarization with chatgpt. IEEE Transactions on Artificial Intelligence, 2024.
*   [7] Xin Tie, Muheon Shin, Ali Pirasteh, Nevein Ibrahim, Zachary Huemann, Sharon M Castellino, Kara M Kelly, John Garrett, Junjie Hu, Steve Y Cho, et al. Personalized impression generation for pet reports using large language models. Journal of Imaging Informatics in Medicine, pages 1–18, 2024.
*   [8] Zhengliang Liu, Aoxiao Zhong, Yiwei Li, Longtao Yang, Chao Ju, Zihao Wu, Chong Ma, Peng Shu, Cheng Chen, Sekeun Kim, et al. Tailoring large language models to radiology: A preliminary approach to llm adaptation for a highly specialized domain. In International Workshop on Machine Learning in Medical Imaging, pages 464–473\. Springer, 2023.
*   [9] Jesutofunmi A Omiye, Haiwen Gui, Shawheen J Rezaei, James Zou, and Roxana Daneshjou. Large language models in medicine: the potentials and pitfalls: a narrative review. Annals of Internal Medicine, 177(2):210–220, 2024.
*   [10] Joschka Haltaufderheide and Robert Ranisch. The ethics of chatgpt in medicine and healthcare: a systematic review on large language models (llms). NPJ digital medicine, 7(1):183, 2024.
*   [11] Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. A survey on large language model based autonomous agents. Frontiers of Computer Science, 18(6):186345, 2024.
*   [12] I de Zarzà, J de Curtò, Gemma Roig, Pietro Manzoni, and Carlos T Calafate. Emergent cooperation and strategy adaptation in multi-agent systems: An extended coevolutionary theory with llms. Electronics, 12(12):2722, 2023.
*   [13] Ling Yue and Tianfan Fu. Ct-agent: Clinical trial multi-agent with large language model-based reasoning. arXiv preprint arXiv:2404.14777, 2024.
*   [14] Yu He Ke, Rui Yang, Sui An Lie, Taylor Xin Yi Lim, Hairil Rizal Abdullah, Daniel Shu Wei Ting, and Nan Liu. Enhancing diagnostic accuracy through multi-agent conversations: Using large language models to mitigate cognitive bias. arXiv preprint arXiv:2401.14589, 2024.
*   [15] Hao Wei, Jianing Qiu, Haibao Yu, and Wu Yuan. Medco: Medical education copilots based on a multi-agent framework. arXiv preprint arXiv:2408.12496, 2024.
*   [16] Xiangru Tang, Anni Zou, Zhuosheng Zhang, Ziming Li, Yilun Zhao, Xingyao Zhang, Arman Cohan, and Mark Gerstein. Medagents: Large language models as collaborators for zero-shot medical reasoning. arXiv preprint arXiv:2311.10537, 2023.
*   [17] Andries Smit, Paul Duckworth, Nathan Grinsztajn, Kale-ab Tessera, Thomas D Barrett, and Arnu Pretorius. Are we going mad? benchmarking multi-agent debate between language models for medical q&a. arXiv preprint arXiv:2311.17371, 2023.
*   [18] Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan, et al. The llama 3 herd of models. arXiv preprint arXiv:2407.21783, 2024.
*   [19] Zehan Li, Xin Zhang, Yanzhao Zhang, Dingkun Long, Pengjun Xie, and Meishan Zhang. Towards general text embeddings with multi-stage contrastive learning. arXiv preprint arXiv:2308.03281, 2023.
*   [20] Kishore Papineni, Salim Roukos, Todd Ward, and Wei-Jing Zhu. Bleu: a method for automatic evaluation of machine translation. In Proceedings of the 40th Annual Meeting of the Association for Computational Linguistics, pages 311–318\. Association for Computational Linguistics, 2002.
*   [21] Chin-Yew Lin. Rouge: A package for automatic evaluation of summaries. In Proceedings of the Workshop on Text Summarization Branches Out (WAS 2004), 2004.
*   [22] Tianyi Zhang, Varsha Kishore, Felix Wu, Kilian Q Weinberger, and Yoav Artzi. Bertscore: Evaluating text generation with bert. In Proceedings of the 8th International Conference on Learning Representations (ICLR 2020), 2020.
*   [23] Xiaoming Zhang, Ming Wang, Xiaocui Yang, Daling Wang, Shi Feng, and Yifei Zhang. Hierarchical retrieval-augmented generation model with rethink for multi-hop question answering. arXiv preprint arXiv:2408.11875, 2024.
*   [24] Weihang Su, Yichen Tang, Qingyao Ai, Zhijing Wu, and Yiqun Liu. Dragin: Dynamic retrieval augmented generation based on the real-time information needs of large language models. arXiv preprint arXiv:2403.10081, 2024.

## Appendix A.
System prompts for the Radiologist and Report Retriever agents

Radiologist: 

> You are a thoracic radiologist. Based on the given ’FINDINGS’ section of the chest image report, your task is to derive the ’IMPRESSION’ section which contains summarization of the clinically significant findings and possible diagnostic interpretations. If reviewers’ feedback is provided, consider it as an additional context to revise your impression. Ensure that your derived ’IMPRESSION’ is clear, concise, accurate. Start your response directly with ’IMPRESSION:’ followed by your impression, without any preamble or additional comments or notes.

Report retriever: 

> You are a reviewer for the ’IMPRESSION’ section in chest image reports. Your task is to verify whether each finding mentioned in the ’IMPRESSION’ section is consistent with the ’FINDINGS’ section. Specifically, focus on the details regarding size, location, severity, and progression status of the findings. If you identify any inconsistent descriptions in the ’IMPRESSION’, end your response with ‘REVISION: YES’. If all findings mentioned in the ’IMPRESSION’ are consistent, end your response with ‘REVISION: NO’.

## Appendix B.
System prompt used for evaluation by GPT-4o

> You are an expert thoracic radiologist with extensive experience in your field. Your task is to evaluate and score the quality of generated impressions for chest image reports produced by large language models (LLMs). Using the findings from the original report as context, compare the quality of each generated impression with the original impression. You will be provided with an original report, including both the findings and the original impression, as well as generated impressions from two models: a single-agent model and a multi-agent model. Your evaluation should be based on the following five factors. Assign a score from 0 to 10 for each factor, where a higher score indicates better quality.
> 
> 1.  1.
>     
>     Clinically Significant Findings: Evaluate whether any clinically significant findings present in the original impression are missing from the generated impression.
>     
>     
> 2.  2.
>     
>     Consistency with Original Findings: Assess how well the generated impression aligns with the findings detailed in the original report.
>     
>     
> 3.  3.
>     
>     Potential Diagnosis: Determine whether the generated impression appropriately suggests or addresses potential diagnoses based on the findings and whether these align with the original impression.
>     
>     
> 4.  4.
>     
>     Stylistic Concordance: Compare the style of the generated impression with that of the original impression.
>     
>     
> 5.  5.
>     
>     Conciseness and Clarity: Judge the generated impression on its clarity and conciseness relative to the original impression.
>     
>     
> 
> Be as objective as possible in your assessment. Output your evaluation in JSON format as follows, providing concise explanations for each score:
> {
>       "Single_agent_model": {
>           "Significant_findings": { "Score": <score>, "Reason": "<explanation>" },
>           "Consistency": { "Score": <score>, "Reason": "<explanation>" },
>           "Diagnosis": { "Score": <score>, "Reason": "<explanation>" },
>           "Style": { "Score": <score>, "Reason": "<explanation>" },
>           "Conciseness_and_clarity": { "Score": <score>, "Reason": "<explanation>" }
>      },
>      "Multi_agent_model": {
>           "Significant_findings": { "Score": <score>, "Reason": "<explanation>" },
>           "Consistency": { "Score": <score>, "Reason": "<explanation>" },
>           "Diagnosis": { "Score": <score>, "Reason": "<explanation>" },
>           "Style": { "Score": <score>, "Reason": "<explanation>" },
>           "Conciseness_and_clarity": { "Score": <score>, "Reason": "<explanation>" }
>      }
> }