<!--yml
category: 未分类
date: 2025-01-11 12:05:19
-->

# Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents

> 来源：[https://arxiv.org/html/2410.13185/](https://arxiv.org/html/2410.13185/)

Long Li$1,2,{\thanks{This work was done when Long Li was an intern at Alibaba DAMO % Academy}}$  Weiwen Xu$1$  Jiayan Guo$1$  Ruochen Zhao$1$  Xingxuan Li$1$  Yuqian Yuan$1,2$  Boqiang Zhang$1,3$  Yuming Jiang$1$  Yifei Xin$1$  Ronghao Dang$1$  Yu Rong$1$  Deli Zhao$1$  Tian Feng$2$  Lidong Bing$1,\thanks{Lidong Bing is the corresponding author}$
$1$DAMO Academy This work was done when Long Li was an intern at Alibaba DAMO AcademyLidong Bing is the corresponding author Alibaba Group
$2$Zhejiang University
$3$University of Science and Technology of China
longli@zju.edu.cn
weiwen.xuu@gmail.com
binglidong@gmail.com(September 2024)

###### Abstract

Effective research ideation is a critical step for scientific research. However, the exponential increase in scientific literature makes it challenging for researchers to stay current with recent advances and identify meaningful research directions. Recent developments in large language models (LLMs) suggest a promising avenue for automating the generation of novel research ideas. However, existing methods for idea generation either trivially prompt LLMs or directly expose LLMs to extensive literature without indicating useful information. Inspired by the research process of human researchers, we propose a Chain-of-Ideas (CoI) agent, an LLM-based agent that organizes relevant literature in a chain structure to effectively mirror the progressive development in a research domain. This organization facilitates LLMs to capture the current advancements in research, thereby enhancing their ideation capabilities. Furthermore, we propose Idea Arena, an evaluation protocol that can comprehensively evaluate idea generation methods from different perspectives, aligning closely with the preferences of human researchers. Experimental results indicate that the CoI agent consistently outperforms other methods and shows comparable quality as humans in research idea generation. Moreover, our CoI agent is budget-friendly, with a minimum cost of $0.50 to generate a candidate idea and its corresponding experimental design¹¹1The code is released at [https://github.com/DAMO-NLP-SG/CoI-Agent](https://github.com/DAMO-NLP-SG/CoI-Agent). Try our online demo at [https://huggingface.co/spaces/DAMO-NLP-SG/CoI_Agent](https://huggingface.co/spaces/DAMO-NLP-SG/CoI_Agent)..

## 1 Introduction

Idea generation is a crucial aspect of scientific research for driving technological innovations and breakthroughs. Traditionally, this process has been predominantly human-driven, necessitating expert researchers to review extensive literature, identify limitations in existing solutions, and propose new research directions. However, the complexity and vastness of scientific literature, coupled with rapid technological advancements, have rendered this task increasingly challenging for researchers.

Recent advancements in large language models (LLMs) (Achiam et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib1); Dubey et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib9); Yang et al., [2024a](https://arxiv.org/html/2410.13185v5#bib.bib25)) have enabled these models to exceed human experts in various scientific tasks, including mathematics (Yu et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib29)), theorem proving (Yang et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib26)), and coding (Chen et al., [2021](https://arxiv.org/html/2410.13185v5#bib.bib7)). Building on this robust scientific foundation, one may hypothesize that LLMs could support a more abstract and creative research idea-generation task. Notably, Si et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib19)); Kumar et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib14)) have validated this hypothesis, highlighting its substantial potential to expedite the discovery of novel concepts and uncharted research avenues.

Existing methods seek to address two key challenges to improve the quality of generated ideas: curating pertinent literature for LLMs to gain inspiration and ensuring the novelty of generated ideas. To address the first challenge, previous research enhances traditional academic retrieval systems, which typically depend on textual similarity, with academic knowledge graphs (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3); Wang et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib22)). For the second challenge, existing approaches either apply predefined criteria such as novelty to guide the idea generation process (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3)) or iteratively refine ideas until they demonstrate low embedding similarities with existing papers (Wang et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib22)).

However, in existing attempts, LLMs are presented with an extensive volume of research literature when asked to generate ideas. This makes LLMs vulnerable to the influence of less relevant works, potentially resulting in ideas that lack logical coherence and technological innovation. As shown in the upper part of Figure [1](https://arxiv.org/html/2410.13185v5#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), the LLM borrows an idea from GraphGPT (Tang et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib20)) and applies it into GoT framework (Besta et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib4)) to generate what they interpret as a “novel idea”. However, the resultant idea conflates two concepts: GoT is a prompting method while GraphGPT is a fine-tuning method leveraging graph neural network architecture (Zhou et al., [2020](https://arxiv.org/html/2410.13185v5#bib.bib32)). In contrast, human researchers often trace the evolution of a research field by analyzing its progression from foundational works to the most recent advancements. This comprehensive perspective provides valuable insights into the key factors driving developments within the domain. Such an understanding enables researchers to critically assess the limitations of earlier studies while identifying emerging trends. Therefore, they are better grounded in devising innovative and impactful research ideas.

![Refer to caption](img/e6b5003e6a097603313bee52176bf83a.png)

Figure 1: Comparison between the vanilla retrieval augmented generation (RAG) research agent and our Chain-of-Ideas agent on the idea generation task.

Motivated by the human practices in conducting research, we introduce a novel Chain-of-Ideas (CoI) agent framework to address the previously identified logical inconsistencies in the ideation processes of LLMs. As shown in the bottom part of Figure [1](https://arxiv.org/html/2410.13185v5#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), CoI agent aims to provide a clear landscape of current research topics by systematically selecting and organizing the relevant papers and their ideas in a chain structure. CoI agent offers several distinctive advantages: Firstly, it minimizes the risk of interference from less relevant literature via carefully selecting papers (i.e. from CoT (Wei et al., [2022](https://arxiv.org/html/2410.13185v5#bib.bib24)) to GoT). Second, LLMs are demonstrated with human practice to craft a novel idea. For example, SC (Wang et al., [2022](https://arxiv.org/html/2410.13185v5#bib.bib23)) emerges as a novel idea derived from CoT. This can be viewed as a form of few-shot prompting strategy, which has been proven to enhance the overall LLM’s generation capability (Brown et al., [2020](https://arxiv.org/html/2410.13185v5#bib.bib6)). Third, CoI exemplifies a global progression in research development. As a result, LLMs can gain a deep understanding of the motivations behind these developmental trends, facilitating the identification of promising future research directions.

Specifically, CoI agent first retrieves an anchor paper of the given research topic. Instead of indiscriminately aggregating all papers within the citation network of the anchor, as done in (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3)), we construct the CoI by selecting relevant and important literature from both the anchor’s references and its subsequent works, thereby extending the chain backward and forward from the anchor. We then prompt the constructed CoI to an LLM for idea generation and experiment design. During idea generation, we require the LLM to predict possible future trends. This prognostic result facilitates the gradual consolidation of the idea, beginning with the motivation for the proposed idea, progressing through an assessment of its potential impact, and culminating in the realization. However, as the evolution of scientific discovery can emerge from multiple perspectives, a single CoI may be insufficient to capture the most promising direction. Additionally, there is no guarantee that the generated ideas will be novel. To address these issues, we construct multiple CoI branches for different perspectives of a research topic. Additionally, a novelty-checker agent iteratively evaluates the draft idea against existing literature and refines it if substantial similarity is identified.

We compare our CoI agent against existing baselines on idea generation in the artificial intelligence (AI) field. To do this, we develop an arena-style evaluation framework called Idea Arena where participant methods compete in pairs, which demonstrates high agreement with human evaluation. The experimental results show that CoI agent consistently ranks first among all automated baselines, surpassing the second-best one by 56 ELO scores in human evaluation. CoI agent can generate ideas as novel as those of human experts. Our analysis further shows that for LLMs to generate novel ideas, a clear developmental trend analysis is more pivotal than the quantity of related literature.

Our contributions are summarized as follows: 1) We propose the CoI agent to enhance LLMs’ capability in idea generation. CoI agent organizes relevant literature in a chain structure to effectively mirror the progressive nature of research development, allowing LLMs to better grasp the current research advancements. 2) We propose Idea Arena for a comprehensive evaluation of idea-generation methods, which shows high agreement with human researchers. 3) Extensive experiments demonstrate the effectiveness of our CoI agent in generating ideas that are comparable to human creativity.

## 2 Method

![Refer to caption](img/8f824b58a859202a84a12d41c7af1972.png)

Figure 2: Our proposed CoI agent framework.

### 2.1 Framework: Chain-of-Ideas Agent

In this section, we detail our CoI agent framework, as illustrated in Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), which consists of three stages: (1) CoI Construction, (2) Idea Generation, and (3) Experiment Design. First, given a research topic, the CoI agent constructs multiple CoIs from existing literature, reflecting different trends within the domain. Then, for each CoI, the LLM predicts future research directions, and crafts ideas through step-by-step consolidation and iterative novelty checks. The best idea is then selected. Lastly, the LLM generates and refines an experiment design to implement the final idea.

### 2.2 CoI Construction

Generating novel research ideas requires a profound comprehension of the respective research domain, coupled with a rigorous reasoning process. Previous endeavors (Lu et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib17); Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3)) have sought to augment LLMs with relevant papers to facilitate the ideation process. However, these methods simply mix these papers into the prompt without effective organization. This scenario is akin to dropping an LLM at a chaotic intersection with no map in sight, leaving it uncertain about which path to take. To address this issue, we propose a Chain-of-Ideas agent framework.

As shown in Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), a CoI, represented as $\{\text{I}_{-M}\rightarrow\cdots\rightarrow\text{I}_{0}\rightarrow\cdots% \rightarrow\text{I}_{N}\}$, is a sequence consisting of $M+N+1$ ideas extracted from $M+N+1$ research papers respectively, where they together show the evolution progress within a given research field. Specifically, given an initial research topic, we prompt the LLM to generate multiple queries, $[q^{1},\dots,q^{K}]$, that reflect $K$ different perspectives of this topic. The prompt is given in Table [7](https://arxiv.org/html/2410.13185v5#A1.T7 "Table 7 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") of Appendix. Unless otherwise specified, all prompts of our framework are presented in the Appendix tables. The $K$ queries are used to construct $K$ branches of CoI. This reduces the reliance on a single CoI that may be insufficient to capture the most significant development and direction. For each query $q^{k}$, we use it to retrieve a top-ranked paper, which we call anchor paper $\text{P}^{k}_{0}$. In Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), ToT (Yao et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib28)) is an illustrative example of an anchor paper. An anchor paper serves as the foundation for constructing a CoI. Specifically, a CoI is constructed by extending from the corresponding anchor paper to related papers in both directions: forward, tracing the progression of ideas, and backward, tracing their origins.

In the forward direction, starting from $\text{P}^{k}_{0}$, we identify subsequent papers that directly cite it by leveraging the Semantic Scholar API²²2[https://www.semanticscholar.org/product/api](https://www.semanticscholar.org/product/api). We use OpenAI’s text-embedding-3-large³³3[https://openai.com/index/new-embedding-models-and-api-updates/](https://openai.com/index/new-embedding-models-and-api-updates/) to rank these papers based on their cosine similarities to the concatenation of the initial research topic and the abstract of the anchor paper. Subsequently, we select the highest-ranked paper as $\text{P}^{k}_{1}$ to extend the CoI in the forward direction (e.g. GoT in Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). This process is repeated iteratively from $\text{P}^{k}_{i}$ to $\text{P}^{k}_{i+1}$, until either the length of the CoI reaches a preset value or the LLM finds that there is no valuable follow-up work (Table [8](https://arxiv.org/html/2410.13185v5#A1.T8 "Table 8 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")).

In the backward direction, starting from the anchor paper $\text{P}^{k}_{0}$, we instruct an LLM to thoroughly review the full paper and to identify candidate references based on the following criteria: 1) references that $\text{P}^{k}_{0}$ directly built upon, 2) references that serve as baselines in $\text{P}^{k}_{0}$, and 3) references that tackle the same topic as $\text{P}^{k}_{0}$. With those candidate references, we ask the LLM to determine the most relevant one to the anchor paper (Tables [9](https://arxiv.org/html/2410.13185v5#A1.T9 "Table 9 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [10](https://arxiv.org/html/2410.13185v5#A1.T10 "Table 10 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")), denoted as $\text{P}^{k}_{-1}$ (e.g. SC in Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")), to extend the CoI backward. This backward extension is also carried out iteratively from $\text{P}^{k}_{-i}$ to $\text{P}^{k}_{-(i+1)}$ to identify preceding papers (e.g. tracing backward from SC to CoT in Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). It terminates when the length of CoI reaches a preset value or we encounter a milestone paper (defined as one with over 1,000 citations), indicating that the idea from the milestone paper could serve as a strong starting point for the CoI. Additionally, we instruct the LLM to terminate the search if no reference relevant to the original research topic is found (Table [8](https://arxiv.org/html/2410.13185v5#A1.T8 "Table 8 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")).

After we collect $K$ paper chains, denoted as $\{\text{P}^{k}_{-M^{k}}\rightarrow\cdots\rightarrow\text{P}^{k}_{0}\rightarrow% \cdots\rightarrow\text{P}^{k}_{N^{k}}\}^{K}_{k=1}$, we ask the LLM to extract ideas from these papers and inherit the progressive relation of the paper chains to form our CoIs $\{\text{I}^{k}_{-M^{k}}\rightarrow\cdots\rightarrow\text{I}^{k}_{0}\rightarrow% \cdots\rightarrow\text{I}^{k}_{N^{k}}\}^{K}_{k=1}$ (Tables [9](https://arxiv.org/html/2410.13185v5#A1.T9 "Table 9 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [10](https://arxiv.org/html/2410.13185v5#A1.T10 "Table 10 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). Then for each CoI, we ask the LLM to summarize the existing research trends by analyzing the evolution between any two adjacent ideas (Table [11](https://arxiv.org/html/2410.13185v5#A1.T11 "Table 11 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). For example, the upper part of Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") shows the evolution process from CoT to GoT step-by-step. Additionally, we extract experiment designs and the definition of key entities from these papers (Tables [9](https://arxiv.org/html/2410.13185v5#A1.T9 "Table 9 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [10](https://arxiv.org/html/2410.13185v5#A1.T10 "Table 10 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). The above information including CoIs and the derived knowledge will be used in the following idea generation and experiment design stages.

### 2.3 Idea Generation

In this section, we use the above-constructed CoIs and their developing trends to guide the generation of a novel idea. For each generated CoI, the first step is to predict possible future trends. As shown in the lower-left section of Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), we prompt the LLM with the CoI, the developing trends of existing works, and the key entities extracted from existing literature, as described in Sec. [2.2](https://arxiv.org/html/2410.13185v5#S2.SS2 "2.2 CoI Construction ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") (Tables [12](https://arxiv.org/html/2410.13185v5#A1.T12 "Table 12 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [13](https://arxiv.org/html/2410.13185v5#A1.T13 "Table 13 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). These entities comprise relevant datasets and potential baseline models, which are important to clarify the concepts mentioned in the existing literature. After obtaining the future trend, we continue to prompt the LLM to articulate its motivation, novelty, and methodology, finally consolidate the idea (Tables [14](https://arxiv.org/html/2410.13185v5#A1.T14 "Table 14 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [15](https://arxiv.org/html/2410.13185v5#A1.T15 "Table 15 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). Through this step-by-step manner, COI can produce a more detailed idea. Following the previous practice (Wang et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib22); Lu et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib17)), we also use a novelty-check agent to evaluate candidate ideas. It retrieves relevant papers and prompts another LLM to assess the similarity between the generated idea and the retrieved papers (Table [16](https://arxiv.org/html/2410.13185v5#A1.T16 "Table 16 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). Based on this assessment, our framework determines if another round of generation is necessary. Finally, we pairwisely compare the generated ideas from all CoI branches and select the one with the highest winning rate as the final idea for the experiment design. This pairwise comparison follows the same method as Idea Arena, refer to Sec. [3.4](https://arxiv.org/html/2410.13185v5#S3.SS4 "3.4 Evaluation: Idea Arena ‣ 3 Experimental Setups ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") for details.

### 2.4 Experiment Design

While our primary goal is to generate novel ideas, it is also useful to develop experimental plans that help users implement these ideas. Thus, we extended the CoI agent to include experiment design. As shown in the lower-right of Figure [2](https://arxiv.org/html/2410.13185v5#S2.F2 "Figure 2 ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), we prompt the LLM with experiments from existing works obtained from Sec. [2.2](https://arxiv.org/html/2410.13185v5#S2.SS2 "2.2 CoI Construction ‣ 2 Method ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") as few-shot examples, along with the proposed idea and key entities, to guide the LLM in designing experiments for our ideas (Table [17](https://arxiv.org/html/2410.13185v5#A1.T17 "Table 17 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")).

We also employ a review agent to assess the candidate experiment designs. Its main role is to evaluate the clarity and comprehensiveness of the protocol, ensuring all key elements—such as datasets and models—are clearly specified. Additionally, it checks if the design provides enough detail for practical implementation (Table [18](https://arxiv.org/html/2410.13185v5#A1.T18 "Table 18 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). The review agent provides critical feedback on these aspects, subsequently utilizing this information to conduct further searches for relevant literature (Table [19](https://arxiv.org/html/2410.13185v5#A1.T19 "Table 19 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")) to help the LLM refine and enhance its previous experiment design (Table [20](https://arxiv.org/html/2410.13185v5#A1.T20 "Table 20 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). Through this iterative process of review and refinement, we arrive at a final experiment design.

## 3 Experimental Setups

### 3.1 Implementations

In our CoI agent, we primarily use GPT-4o (05-13)⁴⁴4[https://openai.com/api/](https://openai.com/api/) as our LLM implementation. For some modules that require full-paper understanding, we use GPT-4o-mini (07-18) to read the paper and summarize the core contents due to its lower price and good summarization capability. We use Semantic Scholar as our academic search engine. For the main experimental results, the maximum length of the CoI is set to 5 and the number of CoI branches is set to 3, and their analysis results are given later. The iteration number of self-refinement in the experiment design stage is set to 1 for cost saving.

### 3.2 Data

To evaluate our CoI agent’s ability to generate novel ideas, we collect recent research topics from Hugging Face’s Daily Papers⁵⁵5[https://huggingface.co/papers](https://huggingface.co/papers), known for its timely updates on AI research and the high quality of the featured papers. We select papers submitted between August 1 and September 15, 2024, ensuring that the topics are sufficiently new and the time frame is after the data cutoff of the LLM. We ask 10 skilled researchers with diverse interests in AI to identify papers that capture their interests. Subsequently, we prompt GPT-4o to extract research topics, proposed ideas, and their corresponding experiment designs from these selected papers (Tables [21](https://arxiv.org/html/2410.13185v5#A1.T21 "Table 21 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), Table [22](https://arxiv.org/html/2410.13185v5#A1.T22 "Table 22 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [23](https://arxiv.org/html/2410.13185v5#A1.T23 "Table 23 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). The extracted topics will then be returned to the researchers for validation, ensuring that the extracted topics are valid and reasonable within their research domains. The extracted ideas and experiment designs will be utilized as our Real Paper baseline, as described in Section [3.3](https://arxiv.org/html/2410.13185v5#S3.SS3 "3.3 Baselines ‣ 3 Experimental Setups ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"). Due to the substantial costs associated with generating and evaluating ideas and experiment designs, we adhere to the assessment scale of Lu et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib17)); Wang et al. ([2023](https://arxiv.org/html/2410.13185v5#bib.bib22)) to collect 50 research topics in total for evaluation.

### 3.3 Baselines

We compare our CoI agent with recent works on idea generation and experiment design. To ensure a fair comparison, we employ GPT-4o and Semantic Search as the LLM and academic retriever implementations, respectively, across all baseline methods. Furthermore, we unify the output format of the generated ideas and experiment designs to minimize evaluation preference towards more structured outputs (Chiang et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib8)). We compare with the following baselines:

*   •

    RAG: This is a vanilla retrieval augmented generation approach (Lewis et al., [2020](https://arxiv.org/html/2410.13185v5#bib.bib15)), where we directly prompt the LLM with retrieved literature for idea generation and experiment design.

*   •

    ResearchAgent (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3)): This work leverages additional academic knowledge graph for enhancing the literature retrieval and adopts a multi-agent framework to iteratively refine ideas through peer discussions. We follow the original paper to reproduce this baseline.

*   •

    GPT-Researcher (Assafelovic, [2023](https://arxiv.org/html/2410.13185v5#bib.bib2)): GPT-Researcher is an agent framework specifically designed for the research domain. The agent is enhanced with plan-and-solve and RAG capabilities.

*   •

    AI-Scientist (Lu et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib17)): This work originally aims to generate the entire paper with the idea, methods, and experimental results. We extract the components related to idea generation and experiment design to serve as our baseline.

*   •

    Real Paper: Note that, in Sec. [3.2](https://arxiv.org/html/2410.13185v5#S3.SS2 "3.2 Data ‣ 3 Experimental Setups ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"), we extract topics from existing research papers. Therefore, the ideas and the experiment designs from these papers serve as a natural baseline to quantify the gap between model-generated ideas and genuine human ideas.

### 3.4 Evaluation: Idea Arena

Model-based Evaluation. The open-ended nature of idea generation poses challenges for automatic evaluation. Prior work primarily uses LLM-based Likert scale system to score ideas (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3); Lu et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib17)). However, Si et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib19)) show this method poorly aligns with human preferences. Instead, they show LLMs perform better in ranking ideas. To obtain reliable scores for evaluation, we propose Idea Arena, a pairwise evaluation system using a Round-Robin tournament to compute ELO scores for each idea-generation method. For a given topic, we require the LLM judge to rank the ideas generated by any pair of methods (Table [24](https://arxiv.org/html/2410.13185v5#A1.T24 "Table 24 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")). We evaluate each pair twice with order reversed to reduce the position bias. To comprehensively evaluate an idea from multiple perspectives, we incorporate criteria from ICML 2020 review guidelines ⁶⁶6[https://icml.cc/Conferences/2020/ReviewerGuidelines](https://icml.cc/Conferences/2020/ReviewerGuidelines), and those in Si et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib19)), which consist of Novelty, Significance, Clarity, Feasibility, and Expected Effectiveness. Finally, the resultant win-loss-tie records are utilized to calculate the ELO scores for each method, following the practices outlined in Zheng et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib31)); Zhao et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib30)). We also evaluate the experiment design in the same pairwise way, focusing on Feasibility, Technical Quality, and Clarity. Refer to Definitions for all metrics in Tables [5](https://arxiv.org/html/2410.13185v5#A1.T5 "Table 5 ‣ A.1 Evaluation Metrics ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [6](https://arxiv.org/html/2410.13185v5#A1.T6 "Table 6 ‣ A.1 Evaluation Metrics ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") of the Appendix.

Human Evaluation. We also perform human evaluation in a pairwise manner. The 10 researchers who review the extracted topics are asked to rank two ideas and experiment designs using the same criteria. To ensure fairness, we anonymize the source of the ideas by concealing the method identity.

## 4 Results

### 4.1 Idea Generation

![Refer to caption](img/c722e0da21f55c9ab2605308a3155975.png)

Figure 3: Evaluation results of idea generation with LLM as a judge.

![Refer to caption](img/043425c81a1e8fad40797d2bdacbb60c.png)

Figure 4: Evaluation results of idea generation with human as judges.

Main results. Figures [4](https://arxiv.org/html/2410.13185v5#S4.F4 "Figure 4 ‣ 4.1 Idea Generation ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [4](https://arxiv.org/html/2410.13185v5#S4.F4 "Figure 4 ‣ 4.1 Idea Generation ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") present the results of idea generation evaluated by both a LLM (specifically, GPT-4o) and human researchers. Detailed scores are in Table [26](https://arxiv.org/html/2410.13185v5#A1.T26 "Table 26 ‣ A.3 Additional experiment results ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") of Appendix. Overall, our CoI agent demonstrates superior performance compared to all other automated methods in both model-based and human-based evaluations. Notably, It substantially outperforms the second-best baselines, GPT-Researcher and RAG, by margins of 108 and 56 ELO scores, respectively, in the two evaluation settings. Our CoI agent’s performance is on par with that of the Real Paper baseline and even excels in the metrics of Novelty and Significance. These results highlight its exceptional capabilities in idea generation. Furthermore, CoI demonstrates superior performance in Clarity, Feasibility, and Expected Effectiveness compared to other automated methods in human evaluation. Nevertheless, it still lags considerably behind the Real Paper in these areas. This substantial gap between automatic methods and Real Paper is expected, as Real Paper ideas undergo extensive experimental validation. Additionally, AI-Scientist’s performance is especially low, likely due to its original design, which focuses on generating full papers from executable code. When given only a research topic, its simplistic idea generation framework limits its ability to produce novel and feasible ideas.

![Refer to caption](img/6ca05939eb74625d69c10ea01899c601.png)

Figure 5: Agreements between human and LLM judges.

Table 1: Agreement between the human and GPT-4o judges in all evaluated dimensions.

|  | Novelty | Significance | Clarity | Feasibility | Effectiveness | Average |
| Agreement | 66.5% | 71.0% | 76.3% | 70.2% | 71.0% | 70.8% |

Human-Model Agreements of Idea Arena. To assess the reliability of our model-based evaluation within Idea Arena, we analyze the agreements between the preferences of the human judges and the LLM judges. We follow Zheng et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib31)) to compute the agreement, which is defined as the probability that two judges agree on the winner of one specific arena match. Figure [5](https://arxiv.org/html/2410.13185v5#S4.F5 "Figure 5 ‣ 4.1 Idea Generation ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") shows the pairwise agreement between humans and several state-of-the-art LLMs, including GPT-4o, Gemini-1.5-Pro-Exp-0827⁷⁷7[https://ai.google.dev/gemini-api/docs/models/experimental-models](https://ai.google.dev/gemini-api/docs/models/experimental-models), and Claude-3.5-Sonnet⁸⁸8[https://www.anthropic.com/news/claude-3-5-sonnet](https://www.anthropic.com/news/claude-3-5-sonnet). We observe an average agreement of 70.8% between GPT-4o and humans. This finding indicates a strong alignment between human-based and model-based evaluations, thereby highlighting the robustness of Idea Arena in evaluating the quality of generated research ideas. Moreover, GPT-4o demonstrates the highest level of agreement with humans among all tested LLMs. Therefore, we will utilize GPT-4o as the LLM judge for subsequent analytical experiments. Additionally, we present the agreement on individual criteria between GPT-4o and human evaluators in Table [1](https://arxiv.org/html/2410.13185v5#S4.T1 "Table 1 ‣ 4.1 Idea Generation ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"). The results indicate a consistently high level of agreement across all assessed criteria.

### 4.2 Ablation Studies for Idea Generation

Table 2: Ablation study on the design of CoI agent. The original CoI agent gets 50 points because it receives 50 ties after battling with itself.

|  | Novelty | Significance | Clarity | Feasibility | Effectiveness | Average |
| CoI Agent | 50 | 50 | 50 | 50 | 50 | 50 |
| –  CoI | 41 | 39 | 44 | 49 | 39 | 42.4 |
| –  Future Trend | 40 | 43 | 51 | 53 | 44 | 46.2 |
| –  Entities | 46 | 49 | 42 | 47 | 43 | 45.4 |

We conduct an ablation study to assess the contributions of each component of the CoI Agent to idea generation quality. The following variants are examined: 1) – CoI: Excludes the CoI construction stage, directly using all retrieved literature without progressive relation mining. 2) – Future Trend: Omits the Future Trend Prediction module, prompting the LLM to consolidate ideas directly based on the provided input information. 3) – Entities: Skips inputting entity definitions during idea generation.To ensure fair comparison, each variant is scored against the full CoI Agent, with 2/1/0 points for win/tie/lose in 50 matches, for a maximum of 100 points.

Results in Table [2](https://arxiv.org/html/2410.13185v5#S4.T2 "Table 2 ‣ 4.2 Ablation Studies for Idea Generation ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") show that all variants negatively affect idea quality. Excluding the CoI construction stage has the most significant impact, emphasizing the importance of organizing literature based on progressive relationships to enhance the LLM’s understanding of trends. Removing the Future Trend Prediction reduces novelty, as the LLM lacks insight into potential forward-thinking ideas. Although slight improvements in clarity and feasibility are observed, these are not substantial, likely due to evaluation variability. Finally, omitting entity information reduces clarity and effectiveness, as the LLM generates more abstract ideas without grounding in specific concepts. This highlights the value of entity information in enhancing the clarity and practical relevance of ideas.

### 4.3 Case Study

Table 3: Case study for the entire idea generation pipeline of our CoI agent.

| Input topic: Using LLM agent to generate novel and original research ideas without human participation |
| --- |
| Chain of ideas: • $I_{-3}$ (Kim et al., [2021](https://arxiv.org/html/2410.13185v5#bib.bib12)): It addresses the challenge of discovering new materials through molecular generation. It introduces GCT, a Transformer with a variational autoencoder, to generate SMILES strings $\dots$ • $I_{-2}$ (Boiko et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib5)): It explores the capabilities of LLM in designing, and executing experiments for scientific research. This work presents a multi-LLM agent to autonomously execute complex scientific experiments via internet browsing, documentation searching, and hands-on experimentation $\dots$ • $I_{-1}$ (Yang et al., [2024b](https://arxiv.org/html/2410.13185v5#bib.bib27)): It proposes a new dataset for social science hypotheses and develops a MOOSE framework with LLM prompting and feedback mechanisms to facilitate hypothesis generation $\dots$ • $I_{0}$ (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3)): It proposes a ResearchAgent framework for automatic idea generation. ResearchAgent combines LLMs with an entity-centric knowledge graph and iterative feedback from reviewing agents, creating a structured and dynamic process for generating and refining research ideas $\dots$ • $I_{1}$ (Si et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib19)): The paper explores the capabilities of LLMs in generating novel research ideas and presents a large-scale comparison between LLM-generated ideas and those produced by 100 NLP expert researchers, revealing that LLMs can produce ideas deemed more novel than human-generated ideas $\dots$ Current Trends: • $I_{-3}\rightarrow I_{-2}$: The progression from $I_{-3}$ to $I_{-2}$ marks a significant shift from the application of neural models for molecular generation to <svg class="ltx_picture" height="13.87" id="S4.I2.i1.p1.4.pic1" overflow="visible" version="1.1" width="385.32"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(192.66,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -191.87 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="383.75">the broader scope of automating scientific research using LLMs</foreignobject></g></g></svg> $\dots$ • $I_{-2}\rightarrow I_{-1}$: The transition from $I_{-2}$ to $I_{-1}$ focuses on <svg class="ltx_picture" height="13.87" id="S4.I2.i2.p1.4.pic1" overflow="visible" version="1.1" width="298.49"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(149.25,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -148.46 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="296.92">refining the autonomous induction capabilities of</foreignobject></g></g></svg> <svg class="ltx_picture" height="11.03" id="S4.I2.i2.p1.5.pic2" overflow="visible" version="1.1" width="37.01"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,11.03) matrix(1 0 0 -1 0 0) translate(18.51,0) translate(0,0.79)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -17.72 0)"><foreignobject height="9.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="35.44">LLMs</foreignobject></g></g></svg>, specifically in generating novel and valid scientific hypotheses $\dots$ • $I_{-1}\rightarrow I_{0}$: $I_{0}$ builds on the advancements made in $I_{-1}$ by further extending the process of generating hypotheses to <svg class="ltx_picture" height="13.51" id="S4.I2.i3.p1.4.pic1" overflow="visible" version="1.1" width="64.65"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.51) matrix(1 0 0 -1 0 0) translate(32.32,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -31.54 0)"><foreignobject height="11.93" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="63.07">generating</foreignobject></g></g></svg> <svg class="ltx_picture" height="13.87" id="S4.I2.i3.p1.5.pic2" overflow="visible" version="1.1" width="251.37"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(125.69,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -124.9 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="250.18">and refining research ideas autonomously</foreignobject></g></g></svg> $\dots$ • $I_{0}\rightarrow I_{1}$: The transition from $I_{0}$ to $I_{1}$ emphasizes the importance of <svg class="ltx_picture" height="13.87" id="S4.I2.i4.p1.4.pic1" overflow="visible" version="1.1" width="193.1"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(96.55,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -95.76 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="192.3">empirical validation of LLMs in</foreignobject></g></g></svg> <svg class="ltx_picture" height="13.87" id="S4.I2.i4.p1.5.pic2" overflow="visible" version="1.1" width="188.64"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(94.32,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -93.53 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="187.45">generating novel research ideas</foreignobject></g></g></svg> and highlights the potential of LLMs to contribute to ideation $\dots$ Future Trend Prediction: Given the previous research’s progression and the identified gaps, a promising direction is to <svg class="ltx_picture" height="13.87" id="S4.T3.1.1.1.1.1.pic1" overflow="visible" version="1.1" width="248.8"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(124.4,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -123.61 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="247.22">unleash the potential of LLM in ideation</foreignobject></g></g></svg>. We can develop a multi-agent system that leverages <svg class="ltx_picture" height="13.87" id="S4.T3.2.2.2.2.2.pic2" overflow="visible" version="1.1" width="370.06"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(185.03,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -184.24 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="369.64">evolutionary algorithms to enhance the diversity and novelty</foreignobject></g></g></svg> of LLM-generated research ideas $\dots$ |
| Final Idea: EvoResearchAgent: Enhancing Diversity and Novelty in Idea Generation with Evolution • Motivation: Using LLMs for idea generation has shown promising advancements. However, challenges persist, <svg class="ltx_picture" height="13.87" id="S4.I3.i1.p1.1.pic1" overflow="visible" version="1.1" width="444.32"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(222.16,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -221.37 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="443.9">particularly concerning the diversity and novelty of LLM-generated ideas</foreignobject></g></g></svg>. Si et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib19)) show that while LLMs can produce novel ideas, they often lack a broad range of perspectives and diversity. Additionally, Baek et al. ([2024](https://arxiv.org/html/2410.13185v5#bib.bib3)) have emphasized the need for a more systematic approach to improving the quality of generated ideas. To address these issues, we propose EvoResearchAgent, <svg class="ltx_picture" height="13.87" id="S4.I3.i1.p1.2.pic2" overflow="visible" version="1.1" width="82.29"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13.87) matrix(1 0 0 -1 0 0) translate(41.15,0) translate(0,3.48)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -40.36 0)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="81.1">a multi-agent</foreignobject></g></g></svg> <svg class="ltx_picture" height="29.17" id="S4.I3.i1.p1.3.pic3" overflow="visible" version="1.1" width="551.75"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,29.17) matrix(1 0 0 -1 0 0) translate(275.88,0) translate(0,18.78)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -275.09 0)"><foreignobject height="72.41" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="551.71">system that leverages evolutionary algorithms to enhance the diversity and novelty of generated ideas</foreignobject></g></g></svg> $\dots$ • Method: $\circ$ Idea Initialize: An LLM generates some initial ideas as the start point of the evolutionary process $\dots$ $\circ$ Metrics: Propose automatic metrics like topic diversity and novelty to evaluate the range of ideas $\dots$ $\circ$ Evolution Integration: 1. Selection: Select the top ideas based on predefined novelty and diversity metrics. 2. Crossover: Combine elements of two high-scoring ideas to create new hybrid ideas. 3. Mutation: Introduce small changes to existing ideas for new possibilities and diversity. 4. Iteration: Repeat the selection, crossover, and mutation process iteratively $\dots$ |

We present an intriguing case study in Table [3](https://arxiv.org/html/2410.13185v5#S4.T3 "Table 3 ‣ 4.3 Case Study ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") with the same topic of our paper – generating novel research ideas using LLMs. Given the input topic, our CoI agent first constructs the chain of ideas, extending $I_{0}$ (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3)) in both forward and backward directions. Then the agent analyzes current research trends for any two adjacent ideas. For instance, it identifies that the core development from $I_{-1}$ to $I_{0}$ is the generation of ideas rather than hypotheses. After digesting the existing trends, the CoI agent realizes that LLMs have great potential in idea generation but are limited in novelty and diversity. Therefore, it proposes an evolutionary algorithm, which specifically models the variations between parents and children, as a possible future trend for novel and diverse idea generation. Finally, the agent consolidates its final idea by drawing on future trends and with practical implementations, such as crossover and mutation, to ensure effective realization. Therefore, the generated idea is viable and novel, deserving further exploration in our future work.

### 4.4 Experiment Design

Table 4: Results of experiment design including both model and human evaluations, as well as their agreements. Tech. refers to the Technical Quality criterion.

 |  |  | Feasibility | Tech. | Clarity | Average |
| Model Evaluation | Real Paper | 1100 | 1122 | 1090 | 1103 |
| CoI Agent (ours) | 1029 | 1096 | 1043 | 1056 |
| RAG | 1022 | 970 | 1016 | 1003 |
| ResearchAgent | 960 | 1020 | 980 | 987 |
| GPT-Researcher | 1001 | 965 | 992 | 986 |
| AI-Scientist | 888 | 827 | 879 | 865 |
| Human Evaluation | Real Paper | 1138 | 1111 | 1111 | 1120 |
| CoI Agent (ours) | 1092 | 1123 | 1121 | 1112 |
| RAG | 1035 | 1041 | 1048 | 1042 |
| GPT-Researcher | 988 | 977 | 971 | 978 |
| ResearchAgent | 939 | 959 | 964 | 954 |
| AI-Scientist | 809 | 788 | 785 | 794 |
|  | Agreement | 70.7% | 75.9% | 72.1% | 73.0% | 

As a byproduct of idea generation, we also require these baselines to develop potential experiment designs for realizing their proposed ideas. Table [4](https://arxiv.org/html/2410.13185v5#S4.T4 "Table 4 ‣ 4.4 Experiment Design ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") presents the arena-style results for experiment designs for both model-based and human-based evaluations. Our CoI Agent demonstrates superior performance across all evaluated criteria in two evaluation settings, achieving the highest scores among all automated methods. Notably, it surpasses RAG, the second-best automated method, by 70 ELO points in human evaluation. Furthermore, there is a high degree of model-human agreement in the experimental designs. Despite the clarity and reasonable technical details of the experiment designs produced by the CoI Agent in support of the proposed ideas, they tend to be less feasible compared to those designs in the existing literature. This phenomenon is also observed during the idea generation phase. Consequently, feasibility represents a significant bottleneck in automatic idea generation, highlighting the need for future research to address this challenge.

### 4.5 Length of CoI

![Refer to caption](img/7fc449c3d34716aa058ee139bbc9a4fd.png)

Figure 6: Length analysis of the CoI.

To examine the impact of the CoI length on the quality of generated ideas, we constructed variants with differing maximum chain lengths. Furthermore, we also adopt the “- CoI” variant in Sec. [4.2](https://arxiv.org/html/2410.13185v5#S4.SS2 "4.2 Ablation Studies for Idea Generation ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") as a 0-length variant, which uses 5 retrieved papers but does not organize them in a chain structure. Figure [6](https://arxiv.org/html/2410.13185v5#S4.F6 "Figure 6 ‣ 4.5 Length of CoI ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") presents the idea arena results among these length variants. We observe a substantial improvement of idea-generation quality when we increase the length from 0 to 3. This indicates a clear developmental trend analysis is more pivotal than the quantity of related literature. Furthermore, the quality of generated ideas continues to improve as the length of the CoI increases. Longer CoIs offer more reliable and comprehensive insights into the evolving trends within the current research domain, thereby enabling the LLM to better capture future development trends. The quality of generated ideas levels off after reaching a maximum length of 5\. This saturation point indicates that this length is sufficient to capture relevant trends, with additional literature offering diminishing returns.

### 4.6 Width of CoI

![Refer to caption](img/cee34d826b0593ea624ec16c1b9de7fb.png)

Figure 7: Width analysis of the CoI.

We also assess the impact of the width of CoI (i.e., the branch number $K$) on the quality of generated ideas. Figure [7](https://arxiv.org/html/2410.13185v5#S4.F7 "Figure 7 ‣ 4.6 Width of CoI ‣ 4 Results ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") shows the trend of average ELO scores with varying branch numbers. Generally, increasing the branch numbers shows a positive correlation with idea quality. However, the disparity in ELO scores across different branch numbers is small. This phenomenon can likely be attributed to the fact that generating multiple chains primarily helps reduce the impact of any single CoI performing poorly. Fortunately, such low-quality CoIs are rare.

## 5 Related Works

Scientific Research Idea Generation. Idea generation is a fundamental step in scientific research. Due to its innovative nature, idea generation has been primarily a human-driven activity. However, recent studies indicate that LLMs can generate plausibly novel and feasible ideas as those of human researchers (Si et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib19); Kumar et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib14)). To investigate the potential of LLMs in idea generation, prior works begin with the task of scientific hypothesis discovery (Yang et al., [2024b](https://arxiv.org/html/2410.13185v5#bib.bib27); Qi et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib18); Wang et al., [2023](https://arxiv.org/html/2410.13185v5#bib.bib22)), which aims to elucidate relationships between two scientific variables. Despite its utility, scientific hypothesis discovery may not fully capture the complexity and multifaceted nature of real-world problems. To address this limitation, projects like GPT-Researcher (Assafelovic, [2023](https://arxiv.org/html/2410.13185v5#bib.bib2)) and ResearchAgent (Baek et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib3)) have adopted a more open-ended idea generation scenario including the underlying methodologies and experimental designs. They leverage agent-based systems to enhance the quality of idea generation. Beyond ideation, numerous studies also explore the use of LLMs for executing experiments (Huang et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib11); Tian et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib21)) or combining both idea generation and experimental execution (Li et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib16); Lu et al., [2024](https://arxiv.org/html/2410.13185v5#bib.bib17)). However, these approaches often make minor modifications to existing ideas for drafting their ideas, which often lack depth and creativity.

Align LLMs with Human Cognitive Patterns. As LLMs are trained with vast amounts of human data (Brown et al., [2020](https://arxiv.org/html/2410.13185v5#bib.bib6)), this may enable them to internalize human cognitive patterns. Firstly, CoT (Wei et al., [2022](https://arxiv.org/html/2410.13185v5#bib.bib24)) indicates that LLMs can enhance their reasoning abilities when provided with step-by-step guidance. Further research supports this notion by showing that simply prompting LLMs to engage in step-by-step reasoning can trigger better reasoning capability (Kojima et al., [2022](https://arxiv.org/html/2410.13185v5#bib.bib13)). Additionally, Fu et al. ([2022](https://arxiv.org/html/2410.13185v5#bib.bib10)) reveals that in-depth reasoning of LLMs can be achieved with more elaborate prompts. As a result, a prompting strategy that closely emulates human cognition is likely to elicit more insightful responses from these models. Motivated by this, we propose CoI to better mimic the progressive cognitive patterns of humans when generating new research ideas.

## 6 Conclusions

In this paper, we introduce Chain of Ideas (CoI) agent, a framework designed to enhance the capability of LLMs in generating research ideas. The CoI agent offers a promising and concise solution by organizing ideas into a chain structure, effectively mirroring the progressive development within a given research domain. It facilitates LLMs to digest the current advancements in research, thereby enhancing their ideation capabilities.p To comprehensively evaluate the capability of automated idea generation methods, we also propose Idea Arena, an evaluation system that requires the participant methods to compete in pairs about their generated ideas for the research topics, which demonstrates high agreement with human evaluation. Experimental results indicate that the CoI agent consistently outperforms other methods and is capable of generating ideas comparable to human creativity.

## References

*   Achiam et al. (2023) Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, et al. Gpt-4 technical report. *arXiv preprint arXiv:2303.08774*, 2023.
*   Assafelovic (2023) Assafelovic. gpt-researcher, 2023. URL: [https://github.com/assafelovic/gpt-researcher](https://github.com/assafelovic/gpt-researcher).
*   Baek et al. (2024) Jinheon Baek, Sujay Kumar Jauhar, Silviu Cucerzan, and Sung Ju Hwang. Researchagent: Iterative research idea generation over scientific literature with large language models. *arXiv preprint arXiv:2404.07738*, 2024.
*   Besta et al. (2024) Maciej Besta, Nils Blach, Ales Kubicek, Robert Gerstenberger, Michal Podstawski, Lukas Gianinazzi, Joanna Gajda, Tomasz Lehmann, Hubert Niewiadomski, Piotr Nyczyk, et al. Graph of thoughts: Solving elaborate problems with large language models. In *Proceedings of the AAAI Conference on Artificial Intelligence*, volume 38, pp.  17682–17690, 2024.
*   Boiko et al. (2023) Daniil A Boiko, Robert MacKnight, and Gabe Gomes. Emergent autonomous scientific research capabilities of large language models. *arXiv preprint arXiv:2304.05332*, 2023.
*   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel Ziegler, Jeffrey Wu, Clemens Winter, Chris Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, and Dario Amodei. Language models are few-shot learners. In H. Larochelle, M. Ranzato, R. Hadsell, M.F. Balcan, and H. Lin (eds.), *Advances in Neural Information Processing Systems*, volume 33, pp.  1877–1901\. Curran Associates, Inc., 2020. URL [https://proceedings.neurips.cc/paper_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf).
*   Chen et al. (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde De Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, et al. Evaluating large language models trained on code. *arXiv preprint arXiv:2107.03374*, 2021.
*   Chiang et al. (2024) Wei-Lin Chiang, Lianmin Zheng, Ying Sheng, Anastasios Nikolas Angelopoulos, Tianle Li, Dacheng Li, Hao Zhang, Banghua Zhu, Michael Jordan, Joseph E. Gonzalez, and Ion Stoica. Chatbot arena: An open platform for evaluating llms by human preference, 2024.
*   Dubey et al. (2024) Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan, et al. The llama 3 herd of models. *arXiv preprint arXiv:2407.21783*, 2024.
*   Fu et al. (2022) Yao Fu, Hao Peng, Ashish Sabharwal, Peter Clark, and Tushar Khot. Complexity-based prompting for multi-step reasoning. In *The Eleventh International Conference on Learning Representations*, 2022.
*   Huang et al. (2024) Qian Huang, Jian Vora, Percy Liang, and Jure Leskovec. MLAgentbench: Evaluating language agents on machine learning experimentation. In *Forty-first International Conference on Machine Learning*, 2024. URL [https://openreview.net/forum?id=1Fs1LvjYQW](https://openreview.net/forum?id=1Fs1LvjYQW).
*   Kim et al. (2021) Hyunseung Kim, Jonggeol Na, and Won Bo Lee. Generative chemical transformer: neural machine learning of molecular geometric structures from chemical language via attention. *Journal of chemical information and modeling*, 61(12):5804–5814, 2021.
*   Kojima et al. (2022) Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. Large language models are zero-shot reasoners. *Advances in neural information processing systems*, 35:22199–22213, 2022.
*   Kumar et al. (2024) Sandeep Kumar, Tirthankar Ghosal, Vinayak Goyal, and Asif Ekbal. Can large language models unlock novel scientific research ideas? *arXiv preprint arXiv:2409.06185*, 2024.
*   Lewis et al. (2020) Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, et al. Retrieval-augmented generation for knowledge-intensive nlp tasks. *Advances in Neural Information Processing Systems*, 33:9459–9474, 2020.
*   Li et al. (2024) Ruochen Li, Teerth Patel, Qingyun Wang, and Xinya Du. Mlr-copilot: Autonomous machine learning research based on large language models agents. *arXiv preprint arXiv:2408.14033*, 2024.
*   Lu et al. (2024) Chris Lu, Cong Lu, Robert Tjarko Lange, Jakob Foerster, Jeff Clune, and David Ha. The ai scientist: Towards fully automated open-ended scientific discovery. *arXiv preprint arXiv:2408.06292*, 2024.
*   Qi et al. (2023) Biqing Qi, Kaiyan Zhang, Haoxiang Li, Kai Tian, Sihang Zeng, Zhang-Ren Chen, and Bowen Zhou. Large language models are zero shot hypothesis proposers. *arXiv preprint arXiv:2311.05965*, 2023.
*   Si et al. (2024) Chenglei Si, Diyi Yang, and Tatsunori Hashimoto. Can llms generate novel research ideas? a large-scale human study with 100+ nlp researchers. *arXiv preprint arXiv:2409.04109*, 2024.
*   Tang et al. (2024) Jiabin Tang, Yuhao Yang, Wei Wei, Lei Shi, Lixin Su, Suqi Cheng, Dawei Yin, and Chao Huang. Graphgpt: Graph instruction tuning for large language models. In *Proceedings of the 47th International ACM SIGIR Conference on Research and Development in Information Retrieval*, pp.  491–500, 2024.
*   Tian et al. (2024) Minyang Tian, Luyu Gao, Shizhuo Dylan Zhang, Xinan Chen, Cunwei Fan, Xuefei Guo, Roland Haas, Pan Ji, Kittithat Krongchon, Yao Li, et al. Scicode: A research coding benchmark curated by scientists. *arXiv preprint arXiv:2407.13168*, 2024.
*   Wang et al. (2023) Qingyun Wang, Doug Downey, Heng Ji, and Tom Hope. Scimon: Scientific inspiration machines optimized for novelty. *arXiv preprint arXiv:2305.14259*, 2023.
*   Wang et al. (2022) Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, Sharan Narang, Aakanksha Chowdhery, and Denny Zhou. Self-consistency improves chain of thought reasoning in language models. *arXiv preprint arXiv:2203.11171*, 2022.
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. Chain-of-thought prompting elicits reasoning in large language models. *Advances in neural information processing systems*, 35:24824–24837, 2022.
*   Yang et al. (2024a) An Yang, Baosong Yang, Binyuan Hui, Bo Zheng, Bowen Yu, Chang Zhou, Chengpeng Li, Chengyuan Li, Dayiheng Liu, Fei Huang, et al. Qwen2 technical report. *arXiv preprint arXiv:2407.10671*, 2024a.
*   Yang et al. (2023) Kaiyu Yang, Aidan M Swope, Alex Gu, Rahul Chalamala, Peiyang Song, Shixing Yu, Saad Godil, Ryan Prenger, and Anima Anandkumar. Leandojo: Theorem proving with retrieval-augmented language models. In *Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track*, 2023. URL [https://openreview.net/forum?id=g7OX2sOJtn](https://openreview.net/forum?id=g7OX2sOJtn).
*   Yang et al. (2024b) Zonglin Yang, Xinya Du, Junxian Li, Jie Zheng, Soujanya Poria, and Erik Cambria. Large language models for automated open-domain scientific hypotheses discovery. In Lun-Wei Ku, Andre Martins, and Vivek Srikumar (eds.), *Findings of the Association for Computational Linguistics ACL 2024*, pp.  13545–13565, Bangkok, Thailand and virtual meeting, August 2024b. Association for Computational Linguistics. URL [https://aclanthology.org/2024.findings-acl.804](https://aclanthology.org/2024.findings-acl.804).
*   Yao et al. (2024) Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, and Karthik Narasimhan. Tree of thoughts: Deliberate problem solving with large language models. *Advances in Neural Information Processing Systems*, 36, 2024.
*   Yu et al. (2023) Longhui Yu, Weisen Jiang, Han Shi, Jincheng Yu, Zhengying Liu, Yu Zhang, James T Kwok, Zhenguo Li, Adrian Weller, and Weiyang Liu. Metamath: Bootstrap your own mathematical questions for large language models. *arXiv preprint arXiv:2309.12284*, 2023.
*   Zhao et al. (2024) Ruochen Zhao, Wenxuan Zhang, Yew Ken Chia, Deli Zhao, and Lidong Bing. Auto arena of llms: Automating llm evaluations with agent peer-battles and committee discussions. *arXiv preprint arXiv:2405.20267*, 2024.
*   Zheng et al. (2024) Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric Xing, et al. Judging llm-as-a-judge with mt-bench and chatbot arena. *Advances in Neural Information Processing Systems*, 36, 2024.
*   Zhou et al. (2020) Jie Zhou, Ganqu Cui, Shengding Hu, Zhengyan Zhang, Cheng Yang, Zhiyuan Liu, Lifeng Wang, Changcheng Li, and Maosong Sun. Graph neural networks: A review of methods and applications. *AI open*, 1:57–81, 2020.

## Appendix A Appendix

### A.1 Evaluation Metrics

Table 5: Evaluation metrics of ideas.

| Metric | Definition |
| --- | --- |
| Novelty | Are the problems or approaches new? Is this a novel combination of familiar techniques? Is it clear how this work differs from previous contributions? Is related work adequately referenced? |
| Significance | Are the idea important? Are other people (practitioners or researchers) likely to use these ideas or build on them? Does the idea address a difficult problem in a better way than previous research? Does it provide a unique theoretical or pragmatic approach? |
| Clarity | Is the paper clearly written? Is it well-organized? Does it adequately inform the reader? |
| Feasibility | Can the idea be realized with existing technology or methods? Are there any technical difficulties or bottlenecks? Is the idea clear and logical? Is there any obvious error or unreasonable part in the idea, and can the experiment be designed normally according to this idea. |
| Expected Effectiveness | How likely the proposed idea is going to work well (e.g., better than existing baselines). |

Table 6: Evaluation metrics of experiment design.

| Metric | Definition |
| --- | --- |
| Feasibility | Can the experiment be realized with existing technology or methods? Are there any technical difficulties or bottlenecks? Is the experimental plan detailed and feasible? Are the experimental steps clear and logical? Is there any obvious error or unreasonable part in the experiment. Consider the rationality of its steps and the possibility that the idea can be successfully implemented. |
| Quality | Is there a clear rationale for each step of the experimental design? Are the baseline and evaluation metrics chosen appropriately? Has the design taken into account the potential advantages and limitations of the methods used? Can this experimental design effectively support the claims made in the idea. |
| Clarity | Is the experimental plan clearly written? Dose it provide enough information for the expert reader to understand the experiment? Is it well organized? Does it adequately inform the reader? |

### A.2 Specific Prompts

Here are the prompts used in this paper.

*   •

    Prompts used in CoI construction

    *   –

        Prompt used to convert a topic into a search query for literature retrieval (Table [7](https://arxiv.org/html/2410.13185v5#A1.T7 "Table 7 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to evaluate whether a paper is relevant to the topic (Table [8](https://arxiv.org/html/2410.13185v5#A1.T8 "Table 8 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to extract idea, experiment, entities and references from paper (Table [9](https://arxiv.org/html/2410.13185v5#A1.T9 "Table 9 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")) and [10](https://arxiv.org/html/2410.13185v5#A1.T10 "Table 10 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents")

    *   –

        Prompt used to summarize current trends of CoI (Table [11](https://arxiv.org/html/2410.13185v5#A1.T11 "Table 11 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

*   •

    Prompts used in idea generation

    *   –

        Prompt used to predict future trend (Table [12](https://arxiv.org/html/2410.13185v5#A1.T12 "Table 12 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [13](https://arxiv.org/html/2410.13185v5#A1.T13 "Table 13 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to generate idea (Table [14](https://arxiv.org/html/2410.13185v5#A1.T14 "Table 14 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents") and [15](https://arxiv.org/html/2410.13185v5#A1.T15 "Table 15 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to check the novelty of the idea (Table [16](https://arxiv.org/html/2410.13185v5#A1.T16 "Table 16 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

*   •

    Prompts used in experiment design

    *   –

        Prompt used to generate experiment design (Table [17](https://arxiv.org/html/2410.13185v5#A1.T17 "Table 17 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to review experiment design (Table [18](https://arxiv.org/html/2410.13185v5#A1.T18 "Table 18 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to get queries for search paper to refine experiment design (Table [19](https://arxiv.org/html/2410.13185v5#A1.T19 "Table 19 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to refine experiment (Table [20](https://arxiv.org/html/2410.13185v5#A1.T20 "Table 20 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

*   •

    Prompts used in benchmark construction

    *   –

        Prompt used to extract topic from real paper (Table [21](https://arxiv.org/html/2410.13185v5#A1.T21 "Table 21 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to extract the idea from real paper (Table [22](https://arxiv.org/html/2410.13185v5#A1.T22 "Table 22 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to extract the experiment design from real paper (Table [23](https://arxiv.org/html/2410.13185v5#A1.T23 "Table 23 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

*   •

    Prompts used in idea arena

    *   –

        Prompt used to compare two ideas (Table [24](https://arxiv.org/html/2410.13185v5#A1.T24 "Table 24 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

    *   –

        Prompt used to compare two experiment designs (Table [25](https://arxiv.org/html/2410.13185v5#A1.T25 "Table 25 ‣ A.2 Specific Prompts ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents"))

Table 7: Prompt used to convert a topic into a search query for literature retrieval

<svg class="ltx_picture ltx_centering" height="224.63" id="A1.T7.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,224.63) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="195.1" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a master of literature searching, tasked with finding relevant research literature based on a specific topic.

Currently, we would like to study the following topic: [Topic] Please provide the literature search queries you would use to search for papers related to the topic and idea.
Each query should be a string and should be enclosed in double quotes. It is best to output one query representing the whole and other queries representing different aspects of the whole.

Output strictly in the following format:
Queries: …</foreignobject></g></g></svg>

Table 8: Prompt used to evaluate whether a paper is relevant to the topic

<svg class="ltx_picture ltx_centering" height="278.59" id="A1.T8.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,278.59) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="249.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are an expert researcher tasked with evaluating whether a given paper is relevant to our research topic based on its title and abstract.

Below are the details of the paper you need to assess:
Title: [Title] Abstract: [Abstract] The topic is: [Topic] If the paper title and abstract are related to the topic, output 1; otherwise, output 0\. As long as you feel that this article has reference value for your question, you can use it to help you study the topic, it does not need to be completely consistent in topic.

Please follow the strict format below:
Think: …
Relevant: 0/1</foreignobject></g></g></svg> 

Table 9: Prompt used to extract idea, experiment, entities and references from paper (part I)

<svg class="ltx_picture" height="421.31" id="A1.T9.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,421.31) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="391.79" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a scientific research expert, tasked with extracting and summarizing information from provided paper content relevant to the topic: [Topic]. Your deliverables will include pertinent references, extracted entities, a detailed summary, and the experimental design.

The topic you are studying is: [Topic] (Ensure that the references are pertinent to this topic.)
Extraction Requirements:
Entities:
1\. Identify unique entities mentioned in the paper, such as model names, datasets, metrics, and specialized terminology.
2\. Format the entities with a name followed by a brief description.
3\. Ensure all entities are relevant to the specified topic ([Topic]).
Summary Idea:
1\. Background: Elaborate on the task’s context and previous work, outlining the starting point of this paper.
2\. Novelty: Describe the main innovations and contributions of this paper in comparison to prior work.
3\. Contribution: Explain the primary methods used, detailing the theory and functions of each core component.
4\. Detail Reason: Provide a thorough explanation of why the chosen methods are effective, including implementation details for further research.
5\. Limitation: Discuss current shortcomings of the approach. Continue to next table $\rightarrow$</foreignobject></g></g></svg>

Table 10: Prompt used to extract idea, experiment, entities and references from paper (part II)

<svg class="ltx_picture" height="577.47" id="A1.T10.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,577.47) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="547.95" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">Experimental Content:
1\. Experimental Process: Detail the entire experimental procedure, from dataset construction to specific steps, ensuring clarity and thoroughness.
2\. Technical Details: Describe any specific technologies involved, providing detailed implementation processes.
3\. Clarity of Plan: State your experimental plan concisely to facilitate understanding without unnecessary complexity.
4\. Baseline: Elaborate on the baseline used, comparative methods, and experimental design, illustrating how these support and validate the conclusions drawn.
5\. Verification: Explain how your experimental design assists in verifying the core idea and ensure it is detailed and feasible.
Relevance Criteria:
1\. Method Relevance: References must directly correlate with the paper’s methodology, indicating improvements or modifications.
2\. Task Relevance: References should address the same task, even if methods differ, better have the same topic [Topic] 3\. Baseline Relevance: References should serve as baselines for the methods discussed in the paper.
4\. Output Format: Provide references without author names or publication years, formatted as titles only.
The paper content is as follows: [Paper content] Please provide the entities, summary idea, experimental design, and the three most relevant references (Sort by relevance, with priority given to new ones with the same level of relevance, do not reference the original paper.) based on the paper’s content.
Note: Ensure the references are pertinent to the topic you are studying: [Topic]. If there are no relevant references, output [].

Now please output strictly in the following format:
Entities: …
Idea: …
Experiment: …
References: …</foreignobject></g></g></svg> 

Table 11: Prompt used to get trends of CoI

<svg class="ltx_picture" height="390.67" id="A1.T11.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,390.67) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="361.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a scientific research expert tasked with summarizing the historical progression of research related to our current topic, based on the literature we have reviewed.

Here are the entities you need to know : [Entities] The topic you are studying is: : [Topic] The literature from early to late: [Idea chain] Your objective is to outline the historical evolution of the research in light of current trends. Please follow these requirements:
Analysis of Published Viewpoints: Examine the progression of ideas across the identified papers. Detail how each paper transitions to the next—for instance, how Paper 0 leads to Paper 1, and so forth. Focus on understanding how Paper 1 builds upon the concepts in Paper 0\. Elaborate on specific advancements made, including proposed modules, their designs, and the rationale behind their effectiveness in addressing previous challenges. Apply this analytical approach to each paper in the sequence.

Please present your findings in the following format:
Trends:
Paper 0 to Paper 1: …
Paper 1 to Paper 2: …
…</foreignobject></g></g></svg>

Table 12: Prompt used to predict future trend (Part I)

<svg class="ltx_picture" height="271.87" id="A1.T12.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,271.87) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="242.35" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a scientific expert tasked with formulating a novel and innovative research idea based on your comprehensive literature review. Your objective is to propose a feasible approach that could significantly advance the field.

Here are the entities you need to know : [Entities] The literature you have studied is as follows: [Chain of ideas] The following section delineates the progressive relationships among the previously summarized research papers: [Trend] Based on previous research, analyze how human experts think and transition from previous methods to subsequent approaches. Focus on their reasoning logic and the sources of their thought processes. Learn to emulate their reasoning patterns to further develop and guide your own research direction in a natural and coherent manner.
Additionally, you are encouraged to adopt the following three modes of thinking: Continue to next table $\rightarrow$</foreignobject></g></g></svg>

Table 13: Prompt used to predict future trend (Part II)

<svg class="ltx_picture" height="457.09" id="A1.T13.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,457.09) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="427.56" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">1\. Reflection: Reflect on scenarios where a specific method encounters significant challenges. Consider potential solutions that could effectively address these issues, make the solutions sounds reasonable, novel and amazing.
2\. Analogy: Identify a specific problem you are currently facing and research existing solutions that have successfully tackled similar challenges. Explore these solutions and adapt key principles and strategies to your situation. Think creatively about how tools and approaches from other domains can be re-imagined to devise a novel strategy for your issue. Encourage you to actively explore methods in other fields to solve your current problems.
3\. Deep Dive: Some methods may present specific approaches to addressing a particular problem. Consider whether there are aspects that could be modified to enhance their rationale and effectiveness.
Note:Each article’s limitations are specific to that particular piece and should not be applied to others. Carefully consider the task at hand and analyze the potential issues you might encounter if you proceed with your original approach, reflecting on the challenges previously faced. Then, think critically about how to address these issues effectively.
You are encouraged to apply human reasoning strategies to identify future research directions based on prior studies. Aim for in-depth analysis rather than mere integration of existing ideas. Please avoid introducing unfamiliar information, ensuring that the trends you present are both authentic and reasonable. Before proposing any trends, take a moment to reflect on the principles underlying the methods you’re employing and assess their relevance to your research area.
The future research direction should be related to the topic: [Topic] Please present the future research direction in the following format:
Future direction: …</foreignobject></g></g></svg>

Table 14: Prompt used to generate idea (part I)

<svg class="ltx_picture" height="736.8" id="A1.T14.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,736.8) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="707.27" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a scientific expert tasked with formulating a novel and innovative research idea based on your comprehensive literature review. Your objective is to propose a feasible approach that could significantly advance the field.
The following are examples of ideas you have proposed in the past that are similar to real papers. Please avoid this situation as much as possible. You can continue to make in-depth innovations, but avoid plagiarism: [Bad case] Here are the entities you need to know: [Entities] The topic you are studying is: [Topic] The literature you have studied is as follows: [Chain of ideas] Your idea is composed of the following components:
Motivation:
1\. Provide a background for your idea, summarizing relevant work.
2\. Identify shortcomings in previous research and highlight the specific problems that remain unsolved and that you aim to address.
Novelty:
1\. Distinguish your proposed method from existing methods (preferably by naming specific approaches).
2\. Detail the improvements of your method compared to past work.
3\. Clearly outline at least three contributions your idea offers to the field, including the problems it resolves and the benefits it delivers.
Method:
1\. Present a detailed description of your idea, focusing on the core method, the specific problem it solves, and enhancements over earlier research (citing relevant literature with titles).
2\. Explain the step-by-step methodology, including the functions of each module and the rationale for why this approach effectively addresses previous challenges.
Please adhere to the following guidelines:
1\. Your research idea should be innovative, feasible, and contribute meaningfully to the field. Please carefully examine the idea you have proposed, avoid immediate perception, and try to be different from the previous methods as much as possible.
2\. Ensure your proposal is solid, clearly defined, and practical to implement. Logic should underpin your reasoning.
3\. Write in clear, concise language aimed at an audience with limited background knowledge in the subject. Avoid complex technical jargon, but when professional terms are necessary, provide thorough explanations.
4\. Refrain from introducing concepts from uncertain fields to prevent proposing ideas that may be incorrect or impractical.
5\. When referencing other research, please include the titles of the cited papers.
6\. Please avoid introducing unfamiliar information, ensuring that the trends you present are both authentic and reasonable. Before proposing any trends, take a moment to reflect on the principles underlying the methods you’re employing and assess their relevance to your research area. Continue to next table $\rightarrow$</foreignobject></g></g></svg>

Table 15: Prompt used to generate idea (part II)

<svg class="ltx_picture" height="324.26" id="A1.T15.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,324.26) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="294.73" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">7\. Each article’s limitations are specific to that particular piece and should not be applied to others. Carefully consider the task at hand and analyze the potential issues you might encounter if you proceed with your original approach, reflecting on the challenges previously faced. Then, think critically about how to address these issues effectively.
The following section delineates the progressive relationships among the previously summarized research papers: [Trend] The following section outlines the potential future research directions based on the literature you have studied: [Future direction] Please output your motivation,novelty,method firstly and then output your final idea.The final idea should clearly explain the origins, motivation, and challenges of your idea, detailing how you overcame these hurdles.

Please present the final idea in the following format:
Motivation: …
Novelty: …
Method: …
Final idea: …</foreignobject></g></g></svg>

Table 16: Prompt used to check the novelty of the idea

<svg class="ltx_picture" height="357.31" id="A1.T16.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,357.31) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="327.78" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a scientific research expert tasked with evaluating the similarity between a specified idea and existing research. Your objective is to determine if the target idea closely resembles any findings in the provided papers.
The target idea you need to check is as follows: [Idea] The relevant papers you need to refer to are as follows:[Content of retrieved papers] Here are your guidelines:
1\. Comparison Process: Begin by thoroughly comparing each paper’s ideas with the target idea. Consider the methodologies, conclusions, and underlying concepts in each paper in your analysis.
2\. Similarity Assessment: If the target idea shares fundamental similarities with any existing research to the extent that they can be considered identical, classify this as plagiarism.
3\. Output: Your output should provide a clear thought process, the similarity assessment, a summary of the target idea, and the ID of the most relevant similar paper.
Please output strictly in the following format:
Think: …
Similar: 0/1
Summary of the idea: …
Similar paper id: 0 to n</foreignobject></g></g></svg>

Table 17: Prompt used to generate experiment

<svg class="ltx_picture" height="706.16" id="A1.T17.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,706.16) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="676.63" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a scientific expert tasked with designing rigorous, feasible experiments based on specified scientific questions and the methodologies derived from the idea I provide, along with relevant past research. Your goal is to assist researchers in systematically testing hypotheses and validating innovative discoveries that could significantly advance their fields.

Past Related Research Experiments: [Past experiments] Here are the entities you need to know: [Entities] Here is the idea you need to design an experiment for: [Idea] Please propose a detailed experimental plan addressing the following points:
1\. Experimental Design: Develop rigorous experiments to ensure the reliability and validity of your results. Provide a comprehensive explanation of the baseline used, comparative methods, ablation study design, and criteria for data analysis and result evaluation. Clarify how these components collectively reinforce and validate the conclusions of your research. Structure your experimental design in a clear, logical, and step-by-step manner, ensuring each step is well-defined and easy to understand.
2\. Implementation of Technologies/Methods: If your experimental design involves specific technologies or methodologies, describe the implementation process in detail, including key technical aspects. For any critical concepts utilized, provide thorough explanations. For instance, if you propose a modular approach, detail its construction, components, and functionality.
3\. Feasibility Assessment: Ensure your experimental plan is realistic, considering technological availability, timelines, resources, and personnel. Identify potential challenges and propose strategies for addressing them.
4\. References to Previous Studies: When citing related literature, include titles and pertinent details of the original papers. Strive to use as many references as necessary to support your experimental design.
5\. Visual Aids: If useful, provide pseudo code or a flowchart to illustrate the implementation process. For example, you can use pseudo code to detail the core algorithm or the model architecture, or employ a flowchart to map out the experimental procedure and data flow.
6\. Clarity of Language: Use straightforward language to describe your methods, assuming the reader may have limited knowledge of the subject matter. Avoid complex jargon and utilize accessible terminology. If professional terms are necessary, please provide clear and detailed explanations.

Please output strictly in the following format:
Experiment:
Step1: …
Step2: …
…</foreignobject></g></g></svg>

Table 18: Prompt used to review experiment

<svg class="ltx_picture" height="407.28" id="A1.T18.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,407.28) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="377.75" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are an expert in paper review. Your task is to analyze whether a given experiment can effectively verify a specific idea, as well as assess the detail and feasibility of the experiment.

Here are the related entities you need to know: [Entities] The idea presented is: [Idea] The corresponding experiment designed for this idea is: [Experiment] Please conduct your analysis based on the following criteria:
1\. Can the experiment validate the idea? If not, identify the issues and suggest improvements to enhance its verification capability and feasibility.
2\. Are there specific experimental procedures that are confusing or poorly designed? Discuss any methods that may not be feasible, uncertainties in constructing the dataset, or a lack of explanation regarding the implementation of certain methods.
3\. Evaluate the clarity, detail, reasonableness, and feasibility of the experimental design.
4\. Provide suggestions for improving the experiment based on the shortcomings identified in your analysis.
5\. Focus solely on the experiment design; please refrain from altering the original idea.
6\. Ensure that your suggestions are constructive, concise, and specific.

Please strictly follow the following format for output:
Suggestion: …</foreignobject></g></g></svg>

Table 19: Prompt used to get query for search paper to refine experiment

<svg class="ltx_picture" height="278.59" id="A1.T19.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,278.59) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="249.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a research expert tasked with refining and improving an experimental plan based on the feedback received.

The experimental plan you proposed is as follows: [Experiment] You have received the following suggestions for improvement: [Suggestions] Please decide whether you need to search for relevant papers to obtain relevant knowledge to improve your experiment.
If you need to search for relevant papers, please provide a search query for literature search, else provide "".
For example: if suggestions say that the dynamic query additional information and update knowledge graph described in the experiment is not clearly described, so you need to output "dynamic knowledge graph update".

Please output strictly in the following format:
Query:…</foreignobject></g></g></svg> 

Table 20: Prompt used to refine experiment

<svg class="ltx_picture" height="656.34" id="A1.T20.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,656.34) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="626.82" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a research expert tasked with refining and improving an experimental plan based on the feedback received.

The information of the literature you maybe need to refer to are as follows: [Searched paper information] The experimental plan you proposed is as follows: [Experiment] Please propose a detailed experimental plan addressing the following points:
1\. Experimental Design: Develop rigorous experiments to ensure the reliability and validity of your results. Provide a comprehensive explanation of the baseline used, comparative methods, ablation study design, and criteria for data analysis and result evaluation. Clarify how these components collectively reinforce and validate the conclusions of your research. Structure your experimental design in a clear, logical, and step-by-step manner, ensuring each step is well-defined and easy to understand.
2\. Implementation of Technologies/Methods: If your experimental design involves specific technologies or methodologies, describe the implementation process in detail, including key technical aspects. For any critical concepts utilized, provide thorough explanations. For instance, if you propose a modular approach, detail its construction, components, and functionality.
3\. Feasibility Assessment: Ensure your experimental plan is realistic, considering technological availability, timelines, resources, and personnel. Identify potential challenges and propose strategies for addressing them.
4\. References to Previous Studies: When citing related literature, include titles and pertinent details of the original papers. Strive to use as many references as necessary to support your experimental design.
5\. Visual Aids: If useful, provide pseudo code or a flowchart to illustrate the implementation process. For example, you can use pseudo code to detail the core algorithm or the model architecture, or employ a flowchart to map out the experimental procedure and data flow.
6\. Clarity of Language: Use straightforward language to describe your methods, assuming the reader may have limited knowledge of the subject matter. Avoid complex jargon and utilize accessible terminology. If professional terms are necessary, please provide clear and detailed explanations.
You have received the following suggestions for improvement:[Suggestions] Please refine your experimental plan based on the feedback provided. Ensure your refined plan is feasible, clearly defined, and addresses the feedback you received.

Please output strictly in the following format:
Experiment: …</foreignobject></g></g></svg>

Table 21: Prompt used to extract topic from real paper

<svg class="ltx_picture" height="257.84" id="A1.T21.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,257.84) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="228.31" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a research expert tasked with extracting the main topic from the provided paper information.

The main topic should encompass broad fields such as "Retrieve augment generation" or "using diffusion models for video generation". However, it should also include a relevant task to the topic, formatted as "topic:... task:...".
Please read the provided paper and extract only the topic, which should follow this structure.
The paper’s title is [Title]
The paper’s abstract is as follows: [Abstract]
The paper’s introduction is as follows: [Introduction] Please output strictly in the following format:
topic: …</foreignobject></g></g></svg>

Table 22: Prompt used to extract idea from real paper

<svg class="ltx_picture" height="610.68" id="A1.T22.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,610.68) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="581.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a research expert tasked with extracting the main idea from the provided paper information.

The main idea should encompass the motivation, solved problem, novelty, method of the paper.
Please read the provided paper and extract the main idea from the paper.
The paper content is as follows: [Content]
Idea is composed of the following components:
Motivation: Explain the background of the idea and past related work, identify the shortcomings of past work, identify the problems that need improvement, and identify the issues the paper want to address.
Novelty: Explain the differences between the method and the current method (preferably list specific methods), explain what improvements the paper have made to the previous method, and then identify the problems that can be solved and the benefits that can be gained from these improvements.
Method: Provide a detailed description of your idea, including the core method, the problem it solves, and the improvement compared with previous work(Cite the previous work with the title of the paper). Explain the specific steps of the method, the specific functions of each module, and the specific reasons why this method can solve the previous problem.
Here are some tips for extracting the main idea:
1\. Make idea easy to understand, use clear and concise language to describe, assuming the reader is someone who has few knowledge of the subject, avoid using complex technical terms, and try to use easy-to-understand terms to explain.If the paper use some professional terms, please explain them in detail.
2\. When the paper cite other papers, please indicate the title of the original paper.
The final idea should be detailed and specific, clearly explain the origins, motivation, novelty, challenge, solved problem and method of the paper, and detail how the overcame these hurdles. Ensure your approach is innovative, specifying how this innovation is reflected in your experimental design.
The final idea should be double-blind, i.e. no experimental results or codes should be shown.

Please output strictly in the following format:
Final idea: …</foreignobject></g></g></svg> 

Table 23: Prompt used to extract experiment from real paper

<svg class="ltx_picture" height="473.7" id="A1.T23.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,473.7) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="444.17" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a research expert tasked with extracting the specific experiment steps from the provided paper information.

The specific experiment steps should include the specific methods for each step.
Please read the provided paper and extract specific experiment steps from the paper.
The paper content is as follows: [Content]
There are some tips for extracting the experiment steps:
1\. Detail the Experimental Process: Describe the entire experimental process, including how to construct the dataset and each specific experimental step. Ensure that each experimental method is clearly and thoroughly detailed.
2\. If specific technologies are involved in the experimental design, describe the implementation process in as much detail as possible (i.e., technical details)
3\. Make sure your experimental plan is concise and clear, and can be easily understood by others,should not be too complicated.
4\. Please provide a detailed explanation of the baseline used in the paper, the comparative methods, the ablation design and the experimental design. Specifically, elaborate on how these elements collectively support and validate the conclusions drawn in your research.
5\. Explain how your experimental design can help you verify the idea and how the experiment is detailed and feasible.

Now please output strictly in the following format:
Experiment:
Step1: …
Step2: …
…</foreignobject></g></g></svg>

Table 24: Prompt used to compare two ideas

<svg class="ltx_picture" height="690.94" id="A1.T24.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,690.94) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="661.41" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a judge in a competition. You have to decide which idea is better.

The idea0 is: [idea0] The idea1 is: [idea1] The topic is: [topic] Which idea do you think is better? Please write a short paragraph to explain your choice.
Here are your evaluation criteria:
1\. Novelty: Are the problems or approaches new? Is this a novel combination of familiar techniques? Is it clear how this work differs from previous contributions? Is related work adequately referenced?
2\. Significance: Are the idea important? Are other people (practitioners or researchers) likely to use these ideas or build on them? Does the idea address a difficult problem in a better way than previous research? Does it provide a unique theoretical or pragmatic approach?
3\. Feasibility: Can the idea be realized with existing technology or methods? Are there any technical difficulties or bottlenecks? Is the idea clear and logical? Is there any obvious error or unreasonable part in the idea, and can the experiment be designed normally according to this idea.
4\. Clarity: Is the paper clearly written? Is it well-organized? Does it adequately inform the reader?
5\. Effectiveness: How likely the proposed idea is going to work well (e.g., better than existing baselines).
Note:
Avoid any position biases and ensure that the order in which the responses were presented does not influence your decision. DO NOT allow the LENGTH of the responses to influence your evaluation, choose the one that is straight-to-the-point instead of unnecessarily verbose. Be as objective as possible. (very important!!!)
If you think idea0 is better than idea1, you should output 0\. If you think idea1 is better than idea0, you should output 1\. If you think idea0 and idea1 are equally good, you should output 2.

Your output should be strictly in following format:
Your thinking process: …
Your choice:
Novelty: 0/1/2
Significance: 0/1/2
Feasibility: 0/1/2
Clarity: 0/1/2
Effectiveness: 0/1/2</foreignobject></g></g></svg>

Table 25: Prompt used to compare two experiments

<svg class="ltx_picture" height="610.68" id="A1.T25.pic1" overflow="visible" version="1.1" width="550"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,610.68) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 22.64 14.76)"><foreignobject color="#000000" height="581.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="504.72">You are a judge in a competition. You have to decide which experiment is better.

The idea of experiment0 is: [idea0] The experiment0 is: [experiment0] The idea of experiment1 is: [idea1] The experiment1 is: [experiment1] Which experiment do you think is better? Please write a short paragraph to explain your choice.
Here are your evaluation criteria:
1\. Feasibility: Can the experiment be realized with existing technology or methods? Are there any technical difficulties or bottlenecks? Is the experimental plan detailed and feasible? Are the experimental steps clear and logical? Is there any obvious error or unreasonable part in the experiment. Consider the rationality of its steps and the possibility that the idea can be successfully implemented.
2\. Quality: Is there a clear rationale for each step of the experimental design? Are the baseline and evaluation metrics chosen appropriately? Has the design taken into account the potential advantages and limitations of the methods used? Can this experimental design effectively support the claims made in the idea.
3\. Clarity: Is the experimental plan clearly written? Dose it provide enough information for the expert reader to understand the experiment? Is it well organized? Does it adequately inform the reader?
Note: Avoid any position biases and ensure that the order in which the responses were presented does not influence your decision. DO NOT allow the LENGTH of the responses to influence your evaluation, choose the one that is straight-to-the-point instead of unnecessarily verbose. Be as objective as possible. (very important!!!)
If you think experiment0 is better than experiment1, you should output 0\. If you think experiment1 is better than experiment0, you should output 1\. If you think experiment0 and experiment1 are equally good, you should output 2.

Your output should be strictly in following format:
Your thinking process: …
Your choice:
Feasibility: 0/1/2
Quality: 0/1/2
Clarity: 0/1/2</foreignobject></g></g></svg> 

### A.3 Additional experiment results

We present the evaluation results of idea generation for both model-based evaluation (including GPT-4o, Gemini-1.5-Pro-Exp-0827, and Claude-3.5-Sonnet) and human-based evaluation in Table [26](https://arxiv.org/html/2410.13185v5#A1.T26 "Table 26 ‣ A.3 Additional experiment results ‣ Appendix A Appendix ‣ Chain of Ideas: Revolutionizing Research via Novel Idea Development with LLM Agents").

Table 26: Evaluation results of idea generation for both model-based evaluation and human-based evaluation.

|  |  | Novelty | Significance | Clarity | Feasibility | Effectiveness | Average | Rank |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Human | Real Paper | 1075 | 1071 | 1118 | 1127 | 1109 | 1100 | 1 |
| CoI Agent (ours) | 1100 | 1103 | 1081 | 1065 | 1078 | 1085 | 2 |
| RAG | 1021 | 1038 | 1022 | 1030 | 1035 | 1029 | 3 |
| GPT-Researcher | 988 | 993 | 993 | 990 | 999 | 992 | 4 |
| ResearchAgent | 982 | 975 | 1001 | 975 | 970 | 980 | 5 |
| AI-Scientist | 835 | 820 | 784 | 813 | 809 | 812 | 6 |
| GPT-4o | Real Paper | 1063 | 1089 | 1137 | 1165 | 1123 | 1115 | 1 |
| CoI Agent (ours) | 1144 | 1138 | 1080 | 1021 | 1152 | 1107 | 2 |
| GPT-Researcher | 995 | 1007 | 995 | 1010 | 989 | 999 | 3 |
| ResearchAgent | 1005 | 1016 | 1005 | 946 | 1004 | 995 | 4 |
| RAG | 914 | 918 | 978 | 1023 | 918 | 950 | 5 |
| AI-Scientist | 878 | 831 | 806 | 836 | 814 | 833 | 6 |
| Gemini1.5-Pro-Exp0827 | Real Paper | 1102 | 1101 | 1125 | 1120 | 1102 | 1110 | 1 |
| CoI Agent (ours) | 1124 | 1119 | 1098 | 1082 | 1113 | 1107 | 2 |
| GPT-Researcher | 1002 | 997 | 1005 | 1014 | 998 | 1003 | 3 |
| ResearchAgent | 986 | 986 | 984 | 975 | 986 | 983 | 4 |
| RAG | 914 | 929 | 948 | 958 | 932 | 936 | 5 |
| AI-Scientist | 873 | 868 | 840 | 851 | 869 | 860 | 6 |
| Claude-3.5-Sonnet | Real Paper | 1099 | 1123 | 1174 | 1149 | 1179 | 1145 | 1 |
| CoI Agent (Ours) | 1165 | 1154 | 1033 | 953 | 1162 | 1094 | 2 |
| GPT-Researcher | 986 | 977 | 1022 | 1039 | 977 | 1000 | 3 |
| ResearchAgent | 1008 | 1023 | 1034 | 926 | 997 | 998 | 4 |
| RAG | 886 | 907 | 977 | 1038 | 884 | 938 | 5 |
| AI-Scientist | 855 | 815 | 760 | 895 | 800 | 825 | 6 |