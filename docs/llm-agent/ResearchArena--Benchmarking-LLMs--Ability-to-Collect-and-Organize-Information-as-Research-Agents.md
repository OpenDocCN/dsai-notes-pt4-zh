<!--yml
category: 未分类
date: 2025-01-11 12:33:09
-->

# ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents

> 来源：[https://arxiv.org/html/2406.10291/](https://arxiv.org/html/2406.10291/)

Hao Kang
haok@andrew.cmu.edu School of Computer Science
Carnegie Mellon University
Pittsburgh, PA, 15213 &Chenyan Xiong
cx@cs.cmu.edu Language Technologies Institute
Carnegie Mellon University
Pittsburgh, PA, 15213

###### Abstract

Large language models (LLMs) have exhibited remarkable performance across various tasks in natural language processing. Nevertheless, challenges still arise when these tasks demand domain-specific expertise and advanced analytical skills, such as conducting research surveys on a designated topic. In this research, we develop ResearchArena, a benchmark that measures LLM agents’ ability to conduct academic surveys, an initial step of academic research process. Specifically, we deconstructs the surveying process into three stages 1) information discovery: locating relevant papers, 2) information selection: assessing papers’ importance to the topic, and 3) information organization: organizing papers into meaningful structures. In particular, we establish an offline environment comprising 12.0M full-text academic papers and 7.9K survey papers, which evaluates agents’ ability to locate supporting materials for composing the survey on a topic, rank the located papers based on their impact, and organize these into a hierarchical knowledge mind-map. With this benchmark, we conduct preliminary evaluations of existing techniques and find that all LLM-based methods under-performing when compared to basic keyword-based retrieval techniques, highlighting substantial opportunities for future research.

## 1 Introduction

Large language models (LLMs) have demonstrated exceptional performance across tasks related to natural language understanding, generation, and various other domains [[1](https://arxiv.org/html/2406.10291v1#bib.bib1), [2](https://arxiv.org/html/2406.10291v1#bib.bib2), [3](https://arxiv.org/html/2406.10291v1#bib.bib3), [4](https://arxiv.org/html/2406.10291v1#bib.bib4)]. The capabilities of LLMs can be significantly augmented through integration with external tools such as code interpreters, gaming simulators, and search engines. This integration facilitates the development of sophisticated autonomous agents capable of receiving feedback and executing tasks in a manner akin to human behavior [[5](https://arxiv.org/html/2406.10291v1#bib.bib5), [6](https://arxiv.org/html/2406.10291v1#bib.bib6), [7](https://arxiv.org/html/2406.10291v1#bib.bib7), [8](https://arxiv.org/html/2406.10291v1#bib.bib8)]. Nevertheless, there remains uncertainty regarding the extent to which LLMs can perform tasks necessitating domain-specific expertise and advanced analytical skills, particularly in the context of conducting research on designated topics.

The potential of LLMs to conduct research would be profoundly impactful, particularly in light of the rapid development of numerous fields and the accompanying information explosion. In these contexts, learning a topic and composing an academic survey report often necessitates several months of effort by multiple researchers. On the LLM side, it is imperative to acquire the capability to conduct independent research on topics not encompassed within their pre-training datasets. Possessing such an ability would eliminate the necessity for continuous updates and re-training of the entire model, thereby significantly enhancing its practical utility across diverse domains.

Previous research involving autonomous agents in tasks that are relatively straightforward and executable by the general public, such as online shopping or playing card games, has demonstrated notable success, particularly when utilizing models like GPT-4 [[6](https://arxiv.org/html/2406.10291v1#bib.bib6), [9](https://arxiv.org/html/2406.10291v1#bib.bib9)]. However, more challenging categories, such as research tasks that require domain-specific expertise, represent the next frontier for potential advancements by LLM agents. Admittedly, there is a paucity of research in this area, and one of the primary challenges is the absence of standardized benchmarks.

To advance the development of research agents capable of conducting comprehensive surveys, we introduce the ResearchArena benchmark, which is rooted in rigorous scholarly content. This benchmark specifically leverages academic papers due to their depth of research, peer-reviewed accuracy, and formal structure—attributes often lacking in other sources such as web pages. The ResearchArena provides an offline environment where autonomous agents can collect and organize information to conduct research across various topics. It comprises three sub-tasks for evaluation: Information Discovery, Information Selection, and Information Organization. These three sub-tasks emulate the general methodology employed by human researchers during literature surveys, which are discussed further below.

Researchers typically conduct literature surveys by defining the scope of their inquiry, developing a search protocol, and iteratively reading and organizing papers into an evolving schema. This process culminates in a synthesis of findings to draw conclusions and highlight future research directions [[10](https://arxiv.org/html/2406.10291v1#bib.bib10)]. Based on this methodology, our benchmark delineates the surveying process into three specific tasks: Information Discovery, Information Selection, and Information Organization. Notably, we do not include the generation of text as part of the evaluation. This exclusion stems from the premise that a comprehensive understanding of the topic, established during the pre-writing stage through research, should already provide a robust foundation for composing a full-length article [[11](https://arxiv.org/html/2406.10291v1#bib.bib11)]. Furthermore, evaluating a complete article is inherently challenging due to variations in individual writing styles. Consequently, we reserve such assessments for future investigations and potentially other benchmarks targeting long-text natural language generation.

The Information Discovery task requires LLMs to identify and retrieve relevant academic papers that are foundational to the survey topic, leveraging their ability to navigate and understand vast scholarly corpus. The Information Selection task then challenges the LLMs to critically evaluate these papers based on their scholarly impact and relevance, mimicking the peer review process to ensure only the most significant studies are considered. Lastly, the Information Organization task assesses the LLMs’ ability to synthesize the selected research into a coherent narrative, offering a structured and insightful overview of the topic, through the use of knowledge mind-maps.

Our assessments indicate that LLMs frequently underperform when compared to simpler keyword-based search methods, particularly in tasks requiring deep analytical skills. For example, traditional techniques such as utilizing a survey title as a retrieval query consistently outperform LLMs in both Information Discovery and Information Selection, as demonstrated by superior recall and precision metrics. Furthermore, during the Information Organization phase, particularly in the absence of oracle guidance¹¹1The term ”oracle” refers to the distinction between intermediate and end-to-end versions in the task of Information Organization, as discussed in Section [5](https://arxiv.org/html/2406.10291v1#S5 "5 Benchmark Tasks ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents")., LLMs encounter significant challenges in constructing coherent and accurate knowledge structures. This underscores a critical need for enhancements in their ability to manage complex organizational tasks independently.

The constructed environment includes 12.0M full-text academic papers and 7.9K survey papers, meticulously curated from the Semantic Scholar Open Research Corpus (S2ORC) [[12](https://arxiv.org/html/2406.10291v1#bib.bib12)]. This rigorous selection process ensures a high standard of reliability and scholarly relevance, rendering the dataset ideal for evaluating LLMs designed to execute complex, domain-specific research. By focusing on such a rich and diverse academic base, the dataset supports a robust analysis of LLM capabilities across multiple scientific domains, providing a realistic and challenging environment for benchmarking. Furthermore, the S2ORC is updated on a weekly basis, allowing for the inclusion and evaluation of newer content that extends beyond the LLMs’ knowledge cutoff.

The remainder of this paper is structured as follows: After reviewing related work in Section [2](https://arxiv.org/html/2406.10291v1#S2 "2 Related Work ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), Section [3](https://arxiv.org/html/2406.10291v1#S3 "3 Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents") details our dataset collection process. Subsequently, Section [4](https://arxiv.org/html/2406.10291v1#S4 "4 Dataset Composition ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents") provides a thorough analysis of the dataset composition and its various statistical properties. Each task within the benchmark, along with their corresponding metrics, is introduced in Section [5](https://arxiv.org/html/2406.10291v1#S5 "5 Benchmark Tasks ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"). Finally, the evaluations across various baselines are presented in Section [6](https://arxiv.org/html/2406.10291v1#S6 "6 Experiments ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents").

## 2 Related Work

Previous research has employed diverse methodologies to compile datasets featuring academic survey papers. For instance, BigSurvey dataset [[13](https://arxiv.org/html/2406.10291v1#bib.bib13)] aggregates over 7K survey papers from arXiv and includes approximately 434K eferences from Microsoft Academic Service and Semantic Scholar. This dataset underwent rigorous preprocessing by removing duplicates, unprocessable files, and normalizing text. On the other hand, Surfer100 dataset [[14](https://arxiv.org/html/2406.10291v1#bib.bib14)] includes 100 surveys emulating Wikipedia page structures, compiled by eight annotators who summarized content from web pages. Each survey contains predefined sections such as Introduction, History, Key Ideas, Variations, and Applications, summarized concisely in 50 to 150 words.

BigSurvey dataset provides references in an abstract-only format, offering a concise overview of documents. Surfer100 utilizes Google search results to compile references for each survey topic, reflecting a broad spectrum of web-based information. In contrast, our dataset emphasizes full-text academic papers for a deeper understanding and leverages bibliographic references from original survey papers for enhanced authority and accuracy.

The most related LLM agents task in previous research focuses on generating Wikipedia articles. Liu et al. proposed a method for generating English Wikipedia articles by framing the task as a multi-document summarization challenge [[15](https://arxiv.org/html/2406.10291v1#bib.bib15)]. Their approach employs a combination of extractive and abstractive summarization techniques. It involves identifying salient information using methods such as TF-IDF and TextRank [[16](https://arxiv.org/html/2406.10291v1#bib.bib16)]. In another study, Shao et al. introduced the STORM system [[17](https://arxiv.org/html/2406.10291v1#bib.bib17)], which addresses pre-writing challenges such as research and outline preparation. STORM enhances the article generation process by simulating multi-perspective conversations, wherein an LLM poses questions and aggregates responses from reliable sources to develop detailed outlines.

While Wikipedia is a valuable resource for obtaining an introductory understanding of a subject, it is inherently limited by the user-authored nature of its content, which does not always guarantee expert oversight. In contrast, rigorous academic research requires a more in-depth and systematic investigation of a topic, often peer-reviewed by experts within the same domain.

## 3 Collection Methodology

This section delineates our methodology for assembling the dataset, which contains three primary stages: survey selection, reference linking, and mind-map extraction. Each stage is indispensable for ensuring the the relevance and accuracy of the dataset, thereby facilitating its application across various benchmark tasks.

We begin with the survey selection stage, which concentrates on identifying relevant survey papers. Following this, we proceed to the reference linking stage, where we incorporate bibliographic references from each selected survey. Finally, we address the mind-map extraction stage, detailing the criteria employed to identify knowledge mind-maps from the surveys. Each of these stages and their respective methodologies are presented in the corresponding subsections. At the very end, we provide a quick overview of the dataset.

### 3.1 Survey Selection

To evaluate the research capabilities on designated topics, it is essential first to identify these topics. This was achieved by extracting every survey paper from the S2ORC dataset, based on a combination of keyword-based filtration and rigorous textual analysis. In general, the titles of survey papers encapsulate the topics discussed therein.

To compile all relevant survey topics, we first need to identify research surveys from the corpus. We assume that titles of all the topic-specific survey papers contain the term “survey”, but not every paper satisfying this criteria is an actual survey pertaining to our research theme. In particular, some papers, despite incorporating the keyword, rely heavily on information outside the corpus. This includes population-based survey questionnaires from Medical domains or Redshift surveys using telescope observations in the field of Physics.

As a result, the identification was accomplished by a combination of keyword-based filtration and rigorous textual analysis. We first excluded those papers whose titles did not contain “survey” as a keyword. Afterwards, we instructed GPT-4 ²²2GPT-4 refers to gpt-4-0613 as documented in [https://platform.openai.com/docs/models](https://platform.openai.com/docs/models). to discern the scope and content of each document based on its title and abstract. We included only those papers that provide an organized view on the current state of field concerning a specific topic. The exact wording of the prompts can be found in Figure [1](https://arxiv.org/html/2406.10291v1#S3.F1 "Figure 1 ‣ 3.1 Survey Selection ‣ 3 Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), where approximately 85% of the papers identified through the initial keyword search were discarded.

As presented in Appendix [B](https://arxiv.org/html/2406.10291v1#A2 "Appendix B Quality of Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), a manual inspection of 25 samples from the final collection of survey papers revealed that our selection method yielded a 92% accuracy rate. The selection process is certainly not a perfect recall since survey papers may not explicitly include the term “survey” in their title. However, we believe that the selected papers are sufficiently representative of the broader distribution of survey literature in the field.

[⬇](data:text/plain;base64,VGhlIHBvaW50IG9mIGEgc3VydmV5IHBhcGVyIGlzIHRvIHByb3ZpZGUgYW4gb3JnYW5pemVkIHZpZXcgb24gdGhlIGN1cnJlbnQgc3RhdGUgb2YgdGhlIGZpZWxkLiBJZiBpdCByZWxpZXMgaGVhdmlseSBvbiBleHRlcm5hbCBpbmZvcm1hdGlvbiwgc3VjaCBhcyB0aGUgcmVzdWx0cyBvZiBhIHBvcHVsYXRpb24gcXVlc3Rpb25uYWlyZSwgZG8gbm90IGluY2x1ZGUgaXQuIFVzaW5nIHRoZSBhYm92ZSBjcml0ZXJpYSwgaXMgdGhlIGZvbGxvd2luZyBhcnRpY2xlIGEgc3VydmV5IHBhcGVyPyBSZXNwb25kIGVpdGhlciAiVHJ1ZSIgb3IgIkZhbHNlIi4=)The  point  of  a  survey  paper  is  to  provide  an  organized  view  on  the  current  state  of  the  field.  If  it  relies  heavily  on  external  information,  such  as  the  results  of  a  population  questionnaire,  do  not  include  it.  Using  the  above  criteria,  is  the  following  article  a  survey  paper?  Respond  either  "True"  or  "False".

Figure 1: Instruction with GPT-4 on survey selection.

The corpus for conducting these research surveys is limited to papers with full-text access in S2ORC. Unlike previous works, we believe relying solely on abstract might omit crucial details present in the full text which could contribute to a deeper understanding of the topic. Enforcing this accessibility constraint reduced the number of papers in S2ORC to 12.0 million.

### 3.2 Reference Linking

To evaluate performance in Information Discovery, it is essential to identify the fundamental sources for these surveys. These sources are derived from the bibliographic references cited within each survey paper. We relied on S2ORC for the extracted bibliographies and enforced additional post-processing to discard any papers unsuitable for evaluations.

Following the selection of relevant survey papers, we proceeded to compile their bibliographic references. Despite the general reliability of the S2ORC bibliographic resolution system, we encountered discrepancies, such as missing references. These issues were particularly prevalent in documents where the reference header was indistinguishable from the main body text. To address these problems, we excluded any survey papers without references, totaling 406, deeming them unsuitable due to the failure of bibliography extraction. Furthermore, survey papers with no accessible citations were filtered out, amounting to 1,635, as such papers offer no evaluative utility.

For references that were successfully extracted, we documented the publication dates for each one. In cases where a reference listed only the year, we assigned the last day of that year as its date to mitigate the risk of information leakage, as discussed in Section [5](https://arxiv.org/html/2406.10291v1#S5 "5 Benchmark Tasks ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"). Furthermore, citations from S2ORC were categorized based on their contribution to the topic, as outlined by Valenzuela et al. [[18](https://arxiv.org/html/2406.10291v1#bib.bib18)] with a supervised classification approach. This categorization involved distinguishing between influential and non-influential references, which is a prerequisite for evaluating the task of Information Selection.

### 3.3 Mind-Map Extraction

A common method for organizing information in academic surveys is the utilization of mind-map style typologies, which promote a systematic understanding of the subject under review. Due to the exclusive text-based nature of the S2ORC corpus, we employed an approach to extract such typologies by collecting every figure-caption pair directly from the Semantic Scholar website. Through the analysis of these captions using GPT-4, we identified relevant mind-map figures and transformed the graphical representations into JSON-encoded trees that preserve their hierarchical structure.

This process is illustrated in Figure [2](https://arxiv.org/html/2406.10291v1#S3.F2 "Figure 2 ‣ 3.3 Mind-Map Extraction ‣ 3 Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), with the prompt provided in Figure [3](https://arxiv.org/html/2406.10291v1#S3.F3 "Figure 3 ‣ 3.3 Mind-Map Extraction ‣ 3 Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"). The extraction performed by GPT-4 is deemed accurate if the hierarchical structure of the figure is adequately represented by the JSON-encoded tree. Furthermore, relevance is determined if the figure authentically represents a knowledge mind-map pertinent to the survey topic. As detailed in Appendix [B](https://arxiv.org/html/2406.10291v1#A2 "Appendix B Quality of Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), a manual inspection of 25 samples from the final collection of mind-maps revealed that our extraction method achieved an accuracy score of 80% and a relevance score of 60%.

![Refer to caption](img/fd3a9c0a206c1ad28c00ddc95ba08580.png)

```
{
    "Pre-trained Models": {
        "Left-to-Right LM": ["GPT", "GPT-2", "GPT-3"],
        "Masked LM": ["BERT", "RoBERTa"],
        "Prefix LM": ["UniLM1", "UniLM2"],
        "Encoder-Decoder": ["T5", "MASS", "BART"],
    }
}

```

Figure 2: Mind-map extraction from a figure [[19](https://arxiv.org/html/2406.10291v1#bib.bib19)] to its JSON representation.

[⬇](data:text/plain;base64,SWRlbnRpZnkgdGhlIGZpZ3VyZSB0aGF0IG1vc3QgbGlrZWx5IGlsbHVzdHJhdGVzIGEgdGF4b25vbXkgb3Igb3ZlcnZpZXcuIFlvdXIgcmVzcG9uc2Ugc2hvdWxkIGJlIGxpbWl0ZWQgdG8gdGhlIGZpbGVuYW1lLCBvciBOVUxMIGlmIG5vdCBmb3VuZC4gVGhlIHByb3ZpZGVkIGZpZ3VyZSBwcmVzZW50cyBhIGhpZXJhcmNoeS4gRXh0cmFjdCBhcyBKU09OLWVuY29kZWQgdHJlZSB3aG9zZSBjaGlsZHJlbiBhcmUgTlVMTC10ZXJtaW5hdGVkLg==)Identify  the  figure  that  most  likely  illustrates  a  taxonomy  or  overview.  Your  response  should  be  limited  to  the  filename,  or  NULL  if  not  found.  The  provided  figure  presents  a  hierarchy.  Extract  as  JSON-encoded  tree  whose  children  are  NULL-terminated.

Figure 3: Instruction with GPT-4 on mind-map extraction.

### 3.4 ResearchArena Dataset

To ensure the reproducibility of our work and compliance with copyright standards, we developed the dataset from S2ORC, which provides access to 81.1 million academic papers in English from various disciplines. These documents are meticulously structured in a machine-readable format with resolved bibliographic references and annotated inline citations. We used the February 06, 2024 release of S2ORC, which was the most recent version at the start of our project.

For a concise summary of our dataset, it consists of approximately 12 million academic papers, each with full-text access, sourced from the Semantic Scholar Open Research Corpus. From this vast repository, we have successfully identified 7,952 survey papers. These surveys have been meticulously analyzed to derive 1,884 mind-maps, which provide structured summaries of the topics covered.

## 4 Dataset Composition

Understanding the composition of our dataset is essential for ensuring the reliability and comprehensiveness of the benchmark used to evaluate LLMs in academic survey tasks. This section details the makeup of our dataset in terms of disciplinary diversity, reference coverage, and the structural complexity of derived typologies, reflecting on how these factors contribute to the robustness and applicability across various domains.

Disciplinary Distribution. We classified each of the 12.0M papers in our public corpus and 7.9K survey papers by the top-5 most popular academic disciplines. This classification was based on the indexing information provided by S2ORC. Frequencies of papers per discipline were then aggregated and visualized to identify trends and imbalances. Figure [4(a)](https://arxiv.org/html/2406.10291v1#S4.F4.sf1 "In Figure 4 ‣ 4 Dataset Composition ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents") and [4(b)](https://arxiv.org/html/2406.10291v1#S4.F4.sf2 "In Figure 4 ‣ 4 Dataset Composition ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents") revealed significant disparities in the frequency of disciplines between the public corpus and the survey subset. Notably, Computer Science is the most prevalent discipline within surveys but less common in the broader corpus. This could reflect the dynamic nature of the CS field, which often necessitates comprehensive reviews to synthesize rapid advancements and emerging trends.

Reference Coverage. For each survey paper, we calculated the coverage ratio as the proportion of its references that were also available within our full-text corpus. We plotted cumulative density functions for each discipline to analyze how extensively the surveys’ references are represented in the broader corpus. As illustrated with Figure [4(c)](https://arxiv.org/html/2406.10291v1#S4.F4.sf3 "In Figure 4 ‣ 4 Dataset Composition ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), similar patterns were observed across all disciplines, where the density experienced exponential decay as the coverage increases. Approximately 17.18% of the survey subset (i.e., 1.3K survey papers) have at least 50% of their references available. This limitation is mainly attributed to copyright restrictions, where full-text is not permitted by the publisher.

Mind-Map Complexity. We analyzed the structural complexity of the mind-maps extracted from survey papers by counting the number of nodes and measuring the maximal depth. These measures provide insights into the conceptual breadth and hierarchical depth of the topics covered. The scatter plot from Figure [4(d)](https://arxiv.org/html/2406.10291v1#S4.F4.sf4 "In Figure 4 ‣ 4 Dataset Composition ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents") showed that typologies in general have shallow depths but a broad range of nodes, suggesting that while survey topics are extensively branched, they do not delve deeply into sub-topics. In particular, most typologies have a maximum depth ranging from 3 to 7 levels, where the coefficient of the regression line in the scatter plot is approximately 2.04.

![Refer to caption](img/d2808b9e9a5a3d588e43503264f57a61.png)

(a) Disciplinary distribution of the public corpus.

![Refer to caption](img/eedff29e8dc573717da7acbab751c2ab.png)

(b) Disciplinary distribution of the survey subset.

![Refer to caption](img/e574e41e245e75690c320fa4e3b2fcbe.png)

(c) Reference coverage of the survey subset.

![Refer to caption](img/8d449da2d5556a0c69880cc9b5a3c881.png)

(d) Complexity with the extracted mind-maps.

Figure 4: Dataset composition analysis with disciplinary distribution, reference coverage, and mind-map complexity. Each of these aspects is critical for benchmark evaluation. Fields of studies like Medicine (Med), Biology (Bio), Physics (Phy), Environmental Science (ES), Computer Science (CS), Engineering (Eng), and Mathematics (Math) are denoted with their abbreviations in the figures.

## 5 Benchmark Tasks

This section presents a comprehensive overview of the benchmark tasks designed to evaluate the capabilities of research agents in discovering, selecting, and organizing information. Each task targets a specific aspect of research proficiency, with rigorous constraints and evaluation metrics to ensure thorough and unbiased assessment.

Information Discovery. Provided a topic extracted from survey title, the task of information discovery requires research agents to identify a subset of documents $R$ from a broader collection $D$. These documents in $R$ should serve as supporting materials for the topic. Ideally, $R$ should encompass all references cited in the original survey $S$.

However, within the collection $D$, there may exist another survey $S^{\prime}$ that delves into the same topic. If research agents were to use the references from $S^{\prime}$ directly, it would circumvent the need for a thorough discovery, defeating the purpose of this task. To prevent information leakage, we impose the additional constraint such that documents in $D$ must be non-survey and published before $S$.

To evaluate performance, we employ standard information retrieval metrics, Recall and Precision, to measure the proportion of relevant documents successfully retrieved and the proportion of retrieved documents that are relevant. Together, these metrics determine the effectiveness and accuracy of the discovery process. For this task, the cutoff parameter $K$ is set at 10 and 100.

Information Selection. The task of information selection requires research agents to rank the discovered documents based on their importance to the topic. The labels are distinctions between influential and non-influential citations, as elaborated in Section [3](https://arxiv.org/html/2406.10291v1#S3 "3 Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"). Normalized Discounted Cumulative Gain (nDCG) [[20](https://arxiv.org/html/2406.10291v1#bib.bib20)] and Mean Reciprocal Rank (MRR) [[21](https://arxiv.org/html/2406.10291v1#bib.bib21)] are used for evaluation.

These measures are crucial because conducting research involves more than merely summarizing retrieved documents; it requires the presentation of key insights from the most significant sources. Furthermore, both human researchers and autonomous agents are limited by their processing capacities. Therefore, it is essential to prioritize and focus on the most critical information first.

Information Organization. For information organization, research agents are required to construct a hierarchical knowledge mind-map $M$ based on $R$. This mind-map should provide a systematic overview of research work developed on topic $T$. As an intermediate step, references $R$ from the original survey paper could be provided to the agents, who would then focus exclusively on constructing $M$. In contrast, for an end-to-end version, $R$ is the set of discovered documents from the previous task.

For evaluation, two primary metrics are employed: Heading Soft Recall [[22](https://arxiv.org/html/2406.10291v1#bib.bib22)] and Heading Entity Recall [[17](https://arxiv.org/html/2406.10291v1#bib.bib17)]. These metrics compare the set of node labels from the original and the constructed knowledge mind-maps, referred to as $A$ and $B$, respectively. To measure similarity of these labels, Heading Soft Recall leverages Sentence-Bert [[23](https://arxiv.org/html/2406.10291v1#bib.bib23)] embedding, while Heading Entity Recall employs Named Entity Recognition from FLAIR [[24](https://arxiv.org/html/2406.10291v1#bib.bib24)] for extraction. The formal definitions for each metric are as follows, where $S$ is the set of labels extracted from the mind-maps.

|  | $\displaystyle\text{Cardinality}(S)$ | $\displaystyle=\sum_{i=1}^{&#124;S&#124;}\frac{1}{\sum_{j=1}^{&#124;S&#124;}\text{Similarity}(S_{i}% ,S_{j})}$ |  |
|  | $\displaystyle\text{Heading Soft Recall}(A,B)$ | $\displaystyle=\frac{\text{Cardinality}(A)+\text{Cardinality}(B)-\text{% Cardinality}(A\cup B)}{\text{Cardinality}(B)}$ |  |
|  | $\displaystyle\text{Heading Entity Recall}(A,B)$ | $\displaystyle=\frac{&#124;\text{EntitiesFrom}(A)\cap\text{EntitiesFrom}(B)&#124;}{&#124;\text% {EntitiesFrom}(A)&#124;}$ |  |

While these metrics provide a measure of content similarity, they do not account for structural alignment. Tree Editing Distance [[25](https://arxiv.org/html/2406.10291v1#bib.bib25)] solves this concern by calculating the minimal number of operations (i.e., relabeling, deleting, and inserting nodes) required to transform one tree into another. Nonetheless, relying on Tree Editing Distance alone might overlook the potential for non-exact label matches. To address this, we propose Tree Semantic Distance, which assigns no cost to editing operations involving nodes whose cosine similarity exceeds $0.8$.

## 6 Experiments

In this section, we present preliminary evaluations of existing techniques, describing their configurations and performance metrics. These techniques encompass both naive keyword-based methods, such as Title, and advanced LLM-based methods, including STORM. The exact wording of the prompts used in each baseline can be found in Appendix [C](https://arxiv.org/html/2406.10291v1#A3 "Appendix C Prompts with Experiments ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents").

### 6.1 Baselines

Information Discovery. For information discovery, research agents are equipped with retrieval tools that enable interaction with the public corpus by submitting queries to retrievers such as BM25 and BGE [[26](https://arxiv.org/html/2406.10291v1#bib.bib26)]. These agents are evaluated based on their ability to effectively leverage these tools by generating relevant queries. Since exploration is limited to previously published non-survey literature, retrievers retry with exponential back-off until the cutoff parameter $K$ is satisfied.

*   •

    Title: Assuming that research topics are encapsulated within survey titles, this method directly employs the title from each survey paper as a query to retrieve relevant materials that support research on the topic. It is important to note that title extraction using S2ORC exhibits variable capitalization across different documents. As a result, we normalize by converting titles to lowercase.

*   •

    Zero-Shot: Assuming that existing LLMs possess prior knowledge relevant to a survey topic, this method extends the Title method by instructing GPT-4 to derive a query from the survey title. This approach leverages the inherent capabilities of LLMs to generate more sophisticated and contextually appropriate queries.

*   •

    Decomposer: As discovered by Tushar et al. [[27](https://arxiv.org/html/2406.10291v1#bib.bib27)], decomposed prompting is more effective when individual reasoning steps of a task are difficult to learn. This principle is applicable to our case, as a survey topic may consist of multiple sub-topics, making it challenging to directly generate a single query that retrieves all relevant papers. Consequently, we instruct GPT-4 to first deconstruct the research topic into several sub-questions. Each sub-question then generates a corresponding sub-query. These sub-queries are retrieved in batches, and the results are amalgamated using reciprocal rank fusion [[28](https://arxiv.org/html/2406.10291v1#bib.bib28)].

*   •

    Self-RAG: As proposed by Asai et al. [[29](https://arxiv.org/html/2406.10291v1#bib.bib29)], Self-RAG adaptively retrieves passages on demand and utilizes reflection tokens to determine which retrieved documents are relevant to the instruction, thus continuing the generation based on the pertinent information. It serves as an enhanced version of Zero-Shot, where the model is instructed to generate a query from the topic. Because the model refines its final query generation based on the discovered information from intermediate retrievals, it operates as a research agent.

*   •

    STORM: As presented in Section [2](https://arxiv.org/html/2406.10291v1#S2 "2 Related Work ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), STORM conducts research through multi-perspective conversations to compose Wikipedia articles on particular topics from scratch. It closely resembles our scenario, except that the environment involves more rigorous academic papers. We record the retrieval history as STORM continues to probe for additional papers. Upon concluding the final round of conversations, every article within the retrieval history is considered part of the discovered information.

Information Selection. For information selection, documents are ranked based on the similarity scores obtained during the discovery phase. For BGE retriever, we rely on FAISS [[30](https://arxiv.org/html/2406.10291v1#bib.bib30)] to retrieve based on L2 distance in the embedding space, which is negated to determine similarity. On the other hand, STORM does not explicitly rank the retrieved documents. It is assumed that documents discovered earlier in the conversations are of higher relevance.

Information Organization. For information organization, the Clustering approach employs Ward’s method for hierarchical clustering on the BGE embedding of every reference article, and the final dendrogram is extracted as typology. The label in each node is computed as the most important TF-IDF word, with ngrams ranging from 1 to 3\. Few-Shot is achieved by providing a few random examples of extracted typologies and instructing GPT-4 to generate another topic-oriented mind-map. Lastly, the article outline generated by STORM is converted to typology, with headings and their nested sub-headings representing the hierarchy.

### 6.2 Evaluation Results

The baseline experiments were conducted on a single machine equipped with 8 NVIDIA RTX A6000 GPUs, 96 CPU cores, and 128GB RAM. Discussion on the performance metrics is presented below.

Information Discovery. As demonstrated in Table [1](https://arxiv.org/html/2406.10291v1#S6.T1 "Table 1 ‣ 6.2 Evaluation Results ‣ 6 Experiments ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"), the task of information discovery remains challenging for all baseline models. This is illustrated by the Recall@100 metric, which falls below 0.15 for BM25 and 0.27 for BGE. Moreover, agent baselines such as Self-RAG and STORM consistently achieve the lowest rankings, irrespective of the retrievers employed. This limitation highlights the critical need for more advanced retrieval mechanisms to manage large volumes of documents effectively during information discovery.

Table 1: Baseline performance on discovery task, evaluated with Recall@10, Recall@100, Precision@10, and Precision@100, where the retrievers include BM25 and BGE.

|  | Recall@10 | Recall@100 | Precision@10 | Precision@100 |
| Baseline | BM25 | BGE | BM25 | BGE | BM25 | BGE | BM25 | BGE |
| Title | 0.0424 | 0.1012 | 0.1338 | 0.2697 | 0.0669 | 0.1541 | 0.0286 | 0.0586 |
| Zero-Shot | 0.0382 | 0.0832 | 0.1253 | 0.2287 | 0.0602 | 0.1232 | 0.0256 | 0.0464 |
| Decomposer | 0.0434 | 0.0879 | 0.1431 | 0.2554 | 0.0717 | 0.1304 | 0.0312 | 0.0536 |
| Self-RAG | 0.0380 | 0.0815 | 0.1210 | 0.2260 | 0.0595 | 0.1215 | 0.0256 | 0.0461 |
| STORM | 0.0281 | 0.0979 | 0.0693 | 0.1441 | 0.0446 | 0.1041 | 0.0130 | 0.0208 |

Information Selection. The performance with information selection is presented in Table [2](https://arxiv.org/html/2406.10291v1#S6.T2 "Table 2 ‣ 6.2 Evaluation Results ‣ 6 Experiments ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"). The results indicate a consistent trend wherein agent baselines underperform compared to keyword-based methods. The evaluation of nDCG at various levels of document retrieval, such as nDCG@10, nDCG@30, and nDCG@100, provides a quantitative assessment of the ranking performance. Notably, for the Title method using the BGE retriever, the nDCG@100 score is 0.2019, which significantly surpasses the score of STORM, which stands at 0.1267\. Improvements during the information discovery phase have the potential to enhance overall performance in the selection phase, as evidenced by Decomposer, which ranks the second behind Title in discovery and selection tasks.

Table 2: Baseline performance on selection task, evaluated with nDCG@10, nDCG@30, nDCG@100, and Precision@100, where the retrievers include BM25 and BGE.

|  | nDCG@10 | nDCG@30 | nDCG@100 | MRR |
| Baseline | BM25 | BGE | BM25 | BGE | BM25 | BGE | BM25 | BGE |
| Title | 0.0711 | 0.1678 | 0.0775 | 0.1754 | 0.0941 | 0.2019 | 0.1903 | 0.3816 |
| Zero-Shot | 0.0634 | 0.1346 | 0.0692 | 0.1417 | 0.0856 | 0.1657 | 0.1743 | 0.3246 |
| Decomposer | 0.0735 | 0.1445 | 0.0803 | 0.1554 | 0.0986 | 0.1838 | 0.1959 | 0.3510 |
| Self-RAG | 0.0627 | 0.1341 | 0.0679 | 0.1415 | 0.0837 | 0.1646 | 0.1705 | 0.3233 |
| STORM | 0.0445 | 0.1275 | 0.0507 | 0.1322 | 0.0524 | 0.1267 | 0.1271 | 0.3206 |

Information Organization. The evaluation on task of information organization under intermediate (i.e., with oracle) and end-to-end (i.e., without oracle) conditions are documented in Table [3](https://arxiv.org/html/2406.10291v1#S6.T3 "Table 3 ‣ 6.2 Evaluation Results ‣ 6 Experiments ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents"). Notably, the metrics exhibit discrepancies across each other, which contrasts with the uniformity observed in previous discovery and selection tasks. This divergence is expected due to the distinct nature of the metrics: Heading Soft Recall and Heading Entity Recall assess content similarity, whereas Tree Semantic Distance evaluates structural alignment.

In the intermediate version, where references are provided to LLMs, the proportion of correctly included entities, as measured by Heading Entity Recall, is slightly higher. Specifically, STORM achieved a recall rate of 0.3098, outperforming the end-to-end condition. Conversely, when it comes to constructing the hierarchy, Clustering outperforms advanced LLM-based agents, as evidenced by its attainment of the lowest Tree Semantic Distance of 45.69 among all baseline methods.

Table 3: Baseline performance on organization task, evaluated with Heading Soft Recall, Heading Entity Recall, and Tree Semantic Distance, across intermediate and end-to-end conditions.

| Oracle | Baseline | Heading Soft Recall ($\uparrow$) | Heading Entity Recall ($\uparrow$) | Tree Semantic Distance ($\downarrow$) |
| Yes | Clustering | 0.6074 | 0.2104 | 45.69 |
| STORM | 0.7325 | 0.3098 | 60.04 |
| No | Few-Shot | 0.8408 | 0.2446 | 49.83 |
| STORM.BM25 | 0.7940 | 0.2938 | 66.65 |
| STORM.BGE | 0.7842 | 0.2693 | 65.93 |

## 7 Limitation

Despite the robust framework and extensive dataset provided by ResearchArena, this study has several limitations. Firstly, the offline environment, though comprehensive, may not accurately represent the dynamic and interconnected nature of live databases and the internet. This discrepancy could potentially limit the applicability of the findings in real-world research settings. Additionally, due to copyright constraints, not every full-text reference of the survey papers could be included. This omission could affect the comprehensive understanding of the survey topics under investigation. Finally, there is no evaluation on text generation but mostly the surveying process. However, even if this is just the first step of conducting research, LLM agents have already shown deficiencies. Future iterations of ResearchArena should address this issue, particularly as these agents improve.

## 8 Conclusion

In conclusion, ResearchArena introduces a rigorous benchmark designed to evaluate LLMs in conducting research surveys on designated topics. By systematically decomposing the survey process into distinct tasks like information discovery, selection, and organization, this benchmark provides a detailed framework for evaluating autonomus research agents. Our findings underscore the potential of LLMs to revolutionize academic research, provided that future advancements can bridge the existing performance gaps. Grounded in Semantic Scholar Open Research Corpus, this work establishes a robust foundation for the future, aiming to improve the ability of LLMs to autonomously conduct expertise-level, domain-specific research.

## References

*   [1] Percy Liang, Rishi Bommasani, Tony Lee, Dimitris Tsipras, Dilara Soylu, Michihiro Yasunaga, Yian Zhang, Deepak Narayanan, Yuhuai Wu, Ananya Kumar, et al. Holistic evaluation of language models. arXiv preprint arXiv:2211.09110, 2022.
*   [2] Yejin Bang, Samuel Cahyawijaya, Nayeon Lee, Wenliang Dai, Dan Su, Bryan Wilie, Holy Lovenia, Ziwei Ji, Tiezheng Yu, Willy Chung, et al. A multitask, multilingual, multimodal evaluation of chatgpt on reasoning, hallucination, and interactivity. arXiv preprint arXiv:2302.04023, 2023.
*   [3] Chengwei Qin, Aston Zhang, Zhuosheng Zhang, Jiaao Chen, Michihiro Yasunaga, and Diyi Yang. Is chatgpt a general-purpose natural language processing task solver? arXiv preprint arXiv:2302.06476, 2023.
*   [4] Md Tahmid Rahman Laskar, M Saiful Bari, Mizanur Rahman, Md Amran Hossen Bhuiyan, Shafiq Joty, and Jimmy Xiangji Huang. A systematic study and comprehensive evaluation of chatgpt on benchmark datasets. arXiv preprint arXiv:2305.18486, 2023.
*   [5] Opendevin: Code less, make more. [https://github.com/OpenDevin/OpenDevin](https://github.com/OpenDevin/OpenDevin), 2024.
*   [6] Shuyan Zhou, Frank F Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Yonatan Bisk, Daniel Fried, Uri Alon, et al. Webarena: A realistic web environment for building autonomous agents. arXiv preprint arXiv:2307.13854, 2023.
*   [7] Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. Toolllm: Facilitating large language models to master 16000+ real-world apis. arXiv preprint arXiv:2307.16789, 2023.
*   [8] Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, and Maosong Sun. Communicative agents for software development. arXiv preprint arXiv:2307.07924, 2023.
*   [9] Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, et al. Agentbench: Evaluating llms as agents. arXiv preprint arXiv:2308.03688, 2023.
*   [10] literature review - how to write a survey paper? - academia stack exchange. [https://academia.stackexchange.com/questions/43371/how-to-write-a-survey-paper](https://academia.stackexchange.com/questions/43371/how-to-write-a-survey-paper), 2015.
*   [11] Laura Dietz and John Foley. Trec car y3: Complex answer retrieval overview. In Proceedings of Text REtrieval Conference (TREC), 2019.
*   [12] Kyle Lo, Lucy Lu Wang, Mark Neumann, Rodney Kinney, and Dan S Weld. S2orc: The semantic scholar open research corpus. arXiv preprint arXiv:1911.02782, 2019.
*   [13] Shuaiqi Liu, Jiannong Cao, Ruosong Yang, and Zhiyuan Wen. Generating a structured summary of numerous academic papers: Dataset and method. arXiv preprint arXiv:2302.04580, 2023.
*   [14] Irene Li, Alexander Fabbri, Rina Kawamura, Yixin Liu, Xiangru Tang, Jaesung Tae, Chang Shen, Sally Ma, Tomoe Mizutani, and Dragomir Radev. Surfer100: Generating surveys from web resources, wikipedia-style. arXiv preprint arXiv:2112.06377, 2021.
*   [15] Peter J Liu, Mohammad Saleh, Etienne Pot, Ben Goodrich, Ryan Sepassi, Lukasz Kaiser, and Noam Shazeer. Generating wikipedia by summarizing long sequences. arXiv preprint arXiv:1801.10198, 2018.
*   [16] Rada Mihalcea and Paul Tarau. Textrank: Bringing order into text. In Proceedings of the 2004 conference on empirical methods in natural language processing, pages 404–411, 2004.
*   [17] Yijia Shao, Yucheng Jiang, Theodore A Kanell, Peter Xu, Omar Khattab, and Monica S Lam. Assisting in writing wikipedia-like articles from scratch with large language models. arXiv preprint arXiv:2402.14207, 2024.
*   [18] Marco Valenzuela, Vu Ha, and Oren Etzioni. Identifying meaningful citations. In Workshops at the twenty-ninth AAAI conference on artificial intelligence, 2015.
*   [19] Pengfei Liu, Weizhe Yuan, Jinlan Fu, Zhengbao Jiang, Hiroaki Hayashi, and Graham Neubig. Pre-train, prompt, and predict: A systematic survey of prompting methods in natural language processing. ACM Computing Surveys, 55(9):1–35, 2023.
*   [20] Kalervo Järvelin and Jaana Kekäläinen. Cumulated gain-based evaluation of ir techniques. ACM Transactions on Information Systems (TOIS), 20(4):422–446, 2002.
*   [21] EM Voorhees. Proceedings of the 8th text retrieval conference. TREC-8 Question Answering Track Report, pages 77–82, 1999.
*   [22] Pasi Fränti and Radu Mariescu-Istodor. Soft precision and recall. Pattern Recognition Letters, 167:115–121, 2023.
*   [23] Nils Reimers and Iryna Gurevych. Sentence-BERT: Sentence embeddings using Siamese BERT-networks. In Kentaro Inui, Jing Jiang, Vincent Ng, and Xiaojun Wan, editors, Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing and the 9th International Joint Conference on Natural Language Processing (EMNLP-IJCNLP), pages 3982–3992, Hong Kong, China, November 2019\. Association for Computational Linguistics.
*   [24] Alan Akbik, Tanja Bergmann, Duncan Blythe, Kashif Rasul, Stefan Schweter, and Roland Vollgraf. FLAIR: An easy-to-use framework for state-of-the-art NLP. In Waleed Ammar, Annie Louis, and Nasrin Mostafazadeh, editors, Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics (Demonstrations), pages 54–59, Minneapolis, Minnesota, June 2019\. Association for Computational Linguistics.
*   [25] Kaizhong Zhang and Dennis Shasha. Simple fast algorithms for the editing distance between trees and related problems. SIAM journal on computing, 18(6):1245–1262, 1989.
*   [26] Shitao Xiao, Zheng Liu, Peitian Zhang, and Niklas Muennighof. C-pack: Packaged resources to advance general chinese embedding. arXiv preprint arXiv:2309.07597, 2023.
*   [27] Tushar Khot, Harsh Trivedi, Matthew Finlayson, Yao Fu, Kyle Richardson, Peter Clark, and Ashish Sabharwal. Decomposed prompting: A modular approach for solving complex tasks. arXiv preprint arXiv:2210.02406, 2022.
*   [28] Gordon V Cormack, Charles LA Clarke, and Stefan Buettcher. Reciprocal rank fusion outperforms condorcet and individual rank learning methods. In Proceedings of the 32nd international ACM SIGIR conference on Research and development in information retrieval, pages 758–759, 2009.
*   [29] Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, and Hannaneh Hajishirzi. Self-rag: Learning to retrieve, generate, and critique through self-reflection. arXiv preprint arXiv:2310.11511, 2023.
*   [30] Jeff Johnson, Matthijs Douze, and Hervé Jégou. Billion-scale similarity search with GPUs. IEEE Transactions on Big Data, 7(3):535–547, 2019.
*   [31] Corby Rosset, Ho-Lam Chung, Guanghui Qin, Ethan C Chau, Zhuo Feng, Ahmed Awadallah, Jennifer Neville, and Nikhil Rao. Researchy questions: A dataset of multi-perspective, decompositional questions for llm web agents. arXiv preprint arXiv:2402.17896, 2024.

## Appendix A Parameters with Collection Methodology

In order to ensure deterministic behavior during dataset construction, the temperature is set to 0, and the seed is fixed at 42 when utilizing GPT-4 for chat completions. The choice of the number 42 is arbitrary; other numbers could be equally effective, provided that the seed remains constant throughout the dataset collection process to maintain reproducibility.

## Appendix B Quality of Collection Methodology

Records of manual inspection over 25 samples from surveys and typologies are presented in Table [4](https://arxiv.org/html/2406.10291v1#A2.T4 "Table 4 ‣ Appendix B Quality of Collection Methodology ‣ ResearchArena: Benchmarking LLMs’ Ability to Collect and Organize Information as Research Agents").

Table 4: Evaluation on the quality of survey selection and mind-map extraction.

| Corpus ID | Accurate Selection | Corpus ID | Object ID | Accurate Extraction | Relevant Extraction |
| 1359411 | Yes | 3373610 | 2-Figure1-1.png | Yes | Yes |
| 2197301 | No | 10837932 | 6-TableII-1.png | Yes | Yes |
| 3638888 | Yes | 20774863 | 2-Figure1-1.png | Yes | No |
| 3799929 | Yes | 21265344 | 4-Figure3-1.png | No | No |
| 4470807 | Yes | 52986472 | 4-Figure1-1.png | Yes | Yes |
| 7972041 | Yes | 54437297 | 6-Figure1-1.png | No | Yes |
| 44951320 | Yes | 59407515 | 4-Figure1-1.png | Yes | No |
| 56895486 | Yes | 67855323 | 3-Figure1-1.png | Yes | No |
| 115156611 | Yes | 201532876 | 6-Figure1-1.png | Yes | No |
| 126187216 | Yes | 204080064 | 5-Figure1-1.png | No | No |
| 134642625 | Yes | 218487045 | 7-Figure4-1.png | Yes | Yes |
| 209386804 | Yes | 221938634 | 6-Figure1-1.png | Yes | Yes |
| 214566304 | Yes | 226300094 | 2-Figure2-1.png | Yes | Yes |
| 229474407 | Yes | 227259882 | 6-Figure2-1.png | No | Yes |
| 233241600 | No | 232126642 | 3-Figure1-1.png | Yes | No |
| 234790465 | Yes | 233677020 | 6-Figure2-1.png | No | No |
| 235794880 | Yes | 237291802 | 2-Figure1-1.png | Yes | Yes |
| 245433612 | Yes | 237327839 | 6-Figure1-1.png | Yes | No |
| 253735066 | Yes | 240011970 | 5-Figure4-1.png | Yes | Yes |
| 254563889 | Yes | 246599122 | 2-Figure1-1.png | Yes | No |
| 258060212 | Yes | 248227736 | 2-Figure1-1.png | Yes | Yes |
| 258541526 | Yes | 248717714 | 4-Figure2-1.png | Yes | Yes |
| 258841314 | Yes | 252089272 | 4-Figure1-1.png | Yes | Yes |
| 259855591 | Yes | 258212628 | 6-Figure1-1.png | Yes | Yes |
| 262464721 | Yes | 260887757 | 4-Figure2-1.png | Yes | Yes |

## Appendix C Prompts with Experiments

### C.1 Prompt to Decomposer for Information Discovery

Adopted from the Researchy Questions by Rosset et al. [[31](https://arxiv.org/html/2406.10291v1#bib.bib31)].

[⬇](data:text/plain;base64,IiIiCiMjIyBCZWxvdyBpcyBhbiBleGFtcGxlIG9uIGhvdyB0byBkZWNvbXBvc2UgYSBjb21wbGV4IHF1ZXN0aW9uIGludG8gc3ViLXF1ZXN0aW9ucyBhbmQgc2VhcmNoIHF1ZXJpZXMuCgpRdWVzdGlvbjogc2hvdWxkIHRoZSBkZWF0aCBwZW5hbHR5IGJlIGxlZ2FsaXplZD8KCjxEZWNvbXBvc2l0aW9uPgogICAgLSBXaGF0IGFyZSB0aGUgYXJndW1lbnRzIGluIGZhdm9yIG9mIHRoZSBkZWF0aCBwZW5hbHR5PwogICAgICAgIC0gRG9lcyB0aGUgZGVhdGggcGVuYWx0eSBzZXJ2ZSBhcyBhIGRldGVycmVudCB0byBjcmltZT8KICAgICAgICAtIElzIHRoZSBkZWF0aCBwZW5hbHR5IGEganVzdCBwdW5pc2htZW50IGZvciBjZXJ0YWluIGNyaW1lcz8KICAgICAgICAtIEhvdyBkb2VzIHRoZSBkZWF0aCBwZW5hbHR5IGNvbXBhcmUgdG8gb3RoZXIgZm9ybXMgb2YgcHVuaXNobWVudCBpbiB0ZXJtcyBvZiBjb3N0IGFuZCBlZmZlY3RpdmVuZXNzPwogICAgLSBXaGF0IGFyZSB0aGUgYXJndW1lbnRzIGFnYWluc3QgdGhlIGRlYXRoIHBlbmFsdHk/CiAgICAgICAgLSBXaGF0IGlzIHRoZSByaXNrIG9mIGV4ZWN1dGluZyBpbm5vY2VudCBwZW9wbGUgd2l0aCBhIGRlYXRoIHBlbmFsdHk/CiAgICAgICAgLSBBcmUgdGhlcmUgYW55IGV0aGljYWwgY29uY2VybnMgc3Vycm91bmRpbmcgdGhlIGRlYXRoIHBlbmFsdHk/CiAgICAgICAgLSBUbyB3aGF0IGV4dGVudCBpcyB0aGUgZGVhdGggcGVuYWx0eSBhcHBsaWVkIGZhaXJseSBhbmQgd2l0aG91dCBiaWFzPwogICAgICAgIC0gSW4gcHJhY3RpY2UsIGhvdyBleHBlbnNpdmUgaXMgdGhlIGRlYXRoIHBlbmFsdHk/CiAgICAtIFdoYXQgaXMgdGhlIGN1cnJlbnQgbGVnYWwgc3RhdHVzIG9mIHRoZSBkZWF0aCBwZW5hbHR5IGluIHZhcmlvdXMganVyaXNkaWN0aW9ucz8KICAgICAgICAtIEluIHdoaWNoIGNvdW50cmllcyBvciBzdGF0ZXMgaXMgdGhlIGRlYXRoIHBlbmFsdHkgY3VycmVudGx5IGxlZ2FsPwogICAgICAgIC0gV2hhdCBhcmUgdGhlIHRyZW5kcyBpbiBkZWF0aCBwZW5hbHR5IGxlZ2lzbGF0aW9uIGFuZCBwdWJsaWMgb3Bpbmlvbj8KICAgIC0gV2hhdCBhcmUgdGhlIGFsdGVybmF0aXZlcyB0byB0aGUgZGVhdGggcGVuYWx0eT8KICAgICAgICAtIEhvdyBlZmZlY3RpdmUgYXJlIGFsdGVybmF0aXZlIHB1bmlzaG1lbnRzIHRvIHRoZSBkZWF0aCBwZW5hbHR5LCBlLmcuIGxpZmUgaW1wcmlzb25tZW50PwogICAgICAgIC0gV2hhdCBhcmUgdGhlIGNvc3RzIGFuZCBiZW5lZml0cyBvZiBhbHRlcm5hdGl2ZXMgdG8gdGhlIGRlYXRoIHBlbmFsdHk/CiAgICAtIEhvdyBkbyB0aGUgcHJvcyBhbmQgY29ucyBvZiB0aGUgZGVhdGggcGVuYWx0eSBjb21wYXJlIHRvIGl0cyBhbHRlcm5hdGl2ZXM/CjwvRGVjb21wb3NpdGlvbj4KCjxRdWVyaWVzPgogICAgLSBhcmd1bWVudHMgaW4gZmF2b3Igb2YgdGhlIGRlYXRoIHBlbmFsdHkKICAgIC0gZGVhdGggcGVuYWx0eSBhcyBhIGRldGVycmVudCB0byBjcmltZQogICAgLSBkZWF0aCBwZW5hbHR5IGFzIGEganVzdCBwdW5pc2htZW50CiAgICAtIGRlYXRoIHBlbmFsdHkgY29zdCBhbmQgZWZmZWN0aXZlbmVzcyBjb21wYXJpc29uCiAgICAtIGFyZ3VtZW50cyBhZ2FpbnN0IHRoZSBkZWF0aCBwZW5hbHR5CiAgICAtIHJpc2sgb2YgZXhlY3V0aW5nIGlubm9jZW50IHBlb3BsZSB3aXRoIGRlYXRoIHBlbmFsdHkKICAgIC0gZXRoaWNhbCBjb25jZXJucyBzdXJyb3VuZGluZyB0aGUgZGVhdGggcGVuYWx0eQogICAgLSBmYWlybmVzcyBhbmQgYmlhcyBpbiBkZWF0aCBwZW5hbHR5IGFwcGxpY2F0aW9uCiAgICAtIGN1cnJlbnQgbGVnYWwgc3RhdHVzIG9mIHRoZSBkZWF0aCBwZW5hbHR5IHdvcmxkd2lkZQogICAgLSB0cmVuZHMgaW4gZGVhdGggcGVuYWx0eSBsZWdpc2xhdGlvbiBhbmQgcHVibGljIG9waW5pb24KICAgIC0gYWx0ZXJuYXRpdmVzIHRvIHRoZSBkZWF0aCBwZW5hbHR5CiAgICAtIGVmZmVjdGl2ZW5lc3Mgb2YgbGlmZSBpbXByaXNvbm1lbnQgd2l0aG91dCBwYXJvbGUKICAgIC0gY29zdHMgYW5kIGJlbmVmaXRzIG9mIGRlYXRoIHBlbmFsdHkgYWx0ZXJuYXRpdmVzCjwvUXVlcmllcz4KClF1ZXN0aW9uOiB7eH0KCiMjIyBJbnN0cnVjdGlvbnM6CgoxLiBXaGF0IHN1Yi1xdWVzdGlvbnMgZG8gSSBuZWVkIHRvIGtub3cgaW4gb3JkZXIgdG8gZnVsbHkgdW5kZXJzdGFuZCBhbmQgYW5zd2VyIHRoZSBhYm92ZSBRdWVzdGlvbi4KICAgIC0gRm9ybWF0IHlvdXIgcmVzcG9uc2UgYXMgYSBidWxsZXQtcG9pbnQgc3R5bGUgb3V0bGluZSBvZiBxdWVzdGlvbnMgYW5kIHN1Yi1xdWVzdGlvbnMgaW4gdGhlIDxEZWNvbXBvc2l0aW9uPiB0YWcuCiAgICAtIE9yZGVyIHlvdXIgc3ViLXF1ZXN0aW9ucyBzdWNoIHRoYXQgb25lIHF1ZXN0aW9uIGNvbWVzIGFmdGVyIGFub3RoZXIgaWYgaXQgbmVlZHMgdG8gdXNlIHRoZSBhbnN3ZXIgdG8gdGhlIHByZXZpb3VzIG9uZS4KICAgIC0gRG8gbm90IGFzayB1bm5lY2Vzc2FyeSBvciB0YW5nZW50aWFsIHN1Yi1xdWVzdGlvbnMsIG9ubHkgdGhvc2UgdGhhdCBhcmUgY3JpdGljYWwgdG8gZmluZGluZyBpbXBvcnRhbnQgaW5mb3JtYXRpb24uCjIpIE5leHQsIHdyaXRlIGEgbGlzdCBvZiBzZWFyY2ggcXVlcmllcyB0aGF0IHdvdWxkIGxpa2VseSBsZWFkIHRvIHJlc3VsdHMgYWRkcmVzc2luZyBhbGwgdGhlIHN1Yi1xdWVzdGlvbnMuCiAgICAtIEVudW1lcmF0ZSB5b3VyIHF1ZXJpZXMgaW4gYSBidWxsZXQtcG9pbnQgc3R5bGUgbGlzdCBpbnNpZGUgdGhlIDxRdWVyaWVzPiB0YWcuCgpZb3UgbWF5IHJlZmVyIHRvIHRoZSBleGFtcGxlIGFib3ZlIGZvciBndWlkYW5jZS4KIiIi)"""###  Below  is  an  example  on  how  to  decompose  a  complex  question  into  sub-questions  and  search  queries.Question:  should  the  death  penalty  be  legalized?<Decomposition>-  What  are  the  arguments  in  favor  of  the  death  penalty?-  Does  the  death  penalty  serve  as  a  deterrent  to  crime?-  Is  the  death  penalty  a  just  punishment  for  certain  crimes?-  How  does  the  death  penalty  compare  to  other  forms  of  punishment  in  terms  of  cost  and  effectiveness?-  What  are  the  arguments  against  the  death  penalty?-  What  is  the  risk  of  executing  innocent  people  with  a  death  penalty?-  Are  there  any  ethical  concerns  surrounding  the  death  penalty?-  To  what  extent  is  the  death  penalty  applied  fairly  and  without  bias?-  In  practice,  how  expensive  is  the  death  penalty?-  What  is  the  current  legal  status  of  the  death  penalty  in  various  jurisdictions?-  In  which  countries  or  states  is  the  death  penalty  currently  legal?-  What  are  the  trends  in  death  penalty  legislation  and  public  opinion?-  What  are  the  alternatives  to  the  death  penalty?-  How  effective  are  alternative  punishments  to  the  death  penalty,  e.g.  life  imprisonment?-  What  are  the  costs  and  benefits  of  alternatives  to  the  death  penalty?-  How  do  the  pros  and  cons  of  the  death  penalty  compare  to  its  alternatives?</Decomposition><Queries>-  arguments  in  favor  of  the  death  penalty-  death  penalty  as  a  deterrent  to  crime-  death  penalty  as  a  just  punishment-  death  penalty  cost  and  effectiveness  comparison-  arguments  against  the  death  penalty-  risk  of  executing  innocent  people  with  death  penalty-  ethical  concerns  surrounding  the  death  penalty-  fairness  and  bias  in  death  penalty  application-  current  legal  status  of  the  death  penalty  worldwide-  trends  in  death  penalty  legislation  and  public  opinion-  alternatives  to  the  death  penalty-  effectiveness  of  life  imprisonment  without  parole-  costs  and  benefits  of  death  penalty  alternatives</Queries>Question:  {x}###  Instructions:1.  What  sub-questions  do  I  need  to  know  in  order  to  fully  understand  and  answer  the  above  Question.-  Format  your  response  as  a  bullet-point  style  outline  of  questions  and  sub-questions  in  the  <Decomposition>  tag.-  Order  your  sub-questions  such  that  one  question  comes  after  another  if  it  needs  to  use  the  answer  to  the  previous  one.-  Do  not  ask  unnecessary  or  tangential  sub-questions,  only  those  that  are  critical  to  finding  important  information.2)  Next,  write  a  list  of  search  queries  that  would  likely  lead  to  results  addressing  all  the  sub-questions.-  Enumerate  your  queries  in  a  bullet-point  style  list  inside  the  <Queries>  tag.You  may  refer  to  the  example  above  for  guidance."""

### C.2 Prompt to Zero-Shot / Self-RAG for Information Discovery

[⬇](data:text/plain;base64,Q3JlYXRlIGEgc2VhcmNoIHF1ZXJ5IHRoYXQgZ2F0aGVycyBzdXBwb3J0aW5nIG1hdGVyaWFscyBmb3Igd3JpdGluZyBhIHN1cnZleSBwYXBlciBvbiB0aGUgZm9sbG93aW5nIHRvcGljOiB7eH0u)Create  a  search  query  that  gathers  supporting  materials  for  writing  a  survey  paper  on  the  following  topic:  {x}.

### C.3 Prompt to Few-Shot for Information Organization

[⬇](data:text/plain;base64,IiIiCiMjIyBFeGFtcGxlcwoKPHRvcGljPgpBIFN1cnZleSBvbiBMaURBUiBTY2FubmluZyBNZWNoYW5pc21zCjwvdG9waWM+Cgo8dHlwb2xvZ3k+CnsiT3B0by1NZWNoYW5pY2FsIEJlYW0gRGVmbGVjdGlvbiBNZWNoYW5pc21zIjogeyJMaW5lIFNjYW5uZXIiOiB7IlNsYW50ZWQgUGxhaW4gTWlycm9yIjogbnVsbCwgIk9mZi1heGlzIFBhcmFib2xpYyBNaXJyb3IiOiBudWxsLCAiUG9seWdvbiBNaXJyb3IiOiBudWxsfSwgIkFyZWEgU2Nhbm5lciI6IHsiU2luZ2xlIEdhbHZhbm9tZXRlciBTY2FubmluZyBNaXJyb3IiOiBudWxsLCAiRG91YmxlIEdhbHZhbm9tZXRlciBTY2FubmluZyBNaXJyb3IiOiBudWxsLCAiR3lyb3Njb3BpYyBNaXJyb3IiOiBudWxsLCAiUmlzbGV5IFNjYW5uZXIiOiBudWxsfX19CjwvdHlwb2xvZ3k+Cgo8dG9waWM+CkEgU3VydmV5IG9uIExhcmdlIExhbmd1YWdlIE1vZGVscyBmb3IgUmVjb21tZW5kYXRpb24KPC90b3BpYz4KCjx0eXBvbG9neT4KeyJMTE00UmVjIjogeyJEaXNjcmltaW5hdGl2ZSBMTE00UmVjIjogeyJGaW5lLXR1bmluZyI6IHsiUHJvbXB0IFR1bmluZyI6IG51bGx9fSwgIkdlbmVyYXRpdmUgTExNNFJlYyI6IHsiTm9uLXR1bmluZyI6IHsiUHJvbXB0aW5nIjogbnVsbCwgIkluLWNvbnRleHQgTGVhcm5pbmciOiBudWxsfSwgIlR1bmluZyI6IHsiRmluZS10dW5pbmciOiBudWxsLCAiUHJvbXB0IFR1bmluZyI6IG51bGwsICJJbnN0cnVjdGlvbiBUdW5pbmciOiBudWxsfX19fQo8L3R5cG9sb2d5PgoKIyMjIEluc3RydWN0aW9ucwoKLSBQcm92aWRlZCBhIHRvcGljLCB5b3VyIHRhc2sgaXMgdG8gY29uc3RydWN0IGEgbWluZC1tYXAgc3R5bGUgdHlwb2xvZ3kgdGhhdCBwcmVzZW50cyBhIHN5c3RlbWF0aWMgdW5kZXJzdGFuZGluZyBvZiB0aGUgdG9waWMuCi0gUHV0IHlvdXIgSlNPTi1lbmNvZGVkIHJlc3BvbnNlIGluIHRoZSB0YWcgYDx0eXBvbG9neT4uLi48L3R5cG9sb2d5PmAuIFlvdSBtYXkgcmVmZXIgdG8gdGhlIGV4YW1wbGVzIGFib3ZlIGZvciBndWlkYW5jZS4KCjx0b3BpYz4Ke3h9CjwvdG9waWM+CiIiIg==)"""###  Examples<topic>A  Survey  on  LiDAR  Scanning  Mechanisms</topic><typology>{"Opto-Mechanical  Beam  Deflection  Mechanisms":  {"Line  Scanner":  {"Slanted  Plain  Mirror":  null,  "Off-axis  Parabolic  Mirror":  null,  "Polygon  Mirror":  null},  "Area  Scanner":  {"Single  Galvanometer  Scanning  Mirror":  null,  "Double  Galvanometer  Scanning  Mirror":  null,  "Gyroscopic  Mirror":  null,  "Risley  Scanner":  null}}}</typology><topic>A  Survey  on  Large  Language  Models  for  Recommendation</topic><typology>{"LLM4Rec":  {"Discriminative  LLM4Rec":  {"Fine-tuning":  {"Prompt  Tuning":  null}},  "Generative  LLM4Rec":  {"Non-tuning":  {"Prompting":  null,  "In-context  Learning":  null},  "Tuning":  {"Fine-tuning":  null,  "Prompt  Tuning":  null,  "Instruction  Tuning":  null}}}}</typology>###  Instructions-  Provided  a  topic,  your  task  is  to  construct  a  mind-map  style  typology  that  presents  a  systematic  understanding  of  the  topic.-  Put  your  JSON-encoded  response  in  the  tag  ‘<typology>...</typology>‘.  You  may  refer  to  the  examples  above  for  guidance.<topic>{x}</topic>"""