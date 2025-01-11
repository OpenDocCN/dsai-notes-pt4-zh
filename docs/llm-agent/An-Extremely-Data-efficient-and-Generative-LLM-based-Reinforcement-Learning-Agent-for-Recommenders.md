<!--yml
category: 未分类
date: 2025-01-11 12:17:55
-->

# An Extremely Data-efficient and Generative LLM-based Reinforcement Learning Agent for Recommenders

> 来源：[https://arxiv.org/html/2408.16032/](https://arxiv.org/html/2408.16032/)

Shuang Feng [fengshuang@gmail.com](mailto:fengshuang@gmail.com) Stanford University SCPDPalo AltoCaliforniaUSA  and  Grace Feng [gracefeng@ucsb.eud](mailto:gracefeng@ucsb.eud) University of California Santa BarbaraSanta BarbaraCaliforniaUSA(2024)

###### Abstract.

Recent advancements in large language models (LLMs) have enabled understanding webpage contexts, product details, and human instructions. Utilizing LLMs as the foundational architecture for either reward models or policies in reinforcement learning has gained popularity - a notable achievement is the success of InstructGPT (Ouyang et al., [2022](https://arxiv.org/html/2408.16032v1#bib.bib12)). RL algorithms have been instrumental in maximizing long-term customer satisfaction and avoiding short-term, myopic goals in industrial recommender systems, which often rely on deep learning models to predict immediate clicks or purchases.

In this project, several RL methods are implemented and evaluated using the WebShop (Yao et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib18)) benchmark environment, data, simulator, and pre-trained model checkpoints. The goal is to train an RL agent to maximize the purchase reward given a detailed human instruction describing a desired product.

The RL agents are developed by fine-tuning a pre-trained BERT model with various objectives, learning from preferences without a reward model, and employing contemporary training techniques such as Proximal Policy Optimization (PPO) as used in InstructGPT (Ouyang et al., [2022](https://arxiv.org/html/2408.16032v1#bib.bib12)), and Direct Preference Optimization (DPO) (Rafailov et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib13)). This report also evaluates the RL agents trained using generative trajectories. Evaluations were conducted using Thompson sampling in the WebShop simulator environment.

The simulated online experiments demonstrate that DPO outperforms PPO in data efficiency and task performance, especially in success rate, using the same amount of training time. However, longer training time is necessary for fair comparison between the two. Specifically, without utilizing any image, a DPO agent achieved a 19% success rate after approximately 3000 steps or 30 minutes of training on T4 GPUs, compared to a PPO agent, which reached a 15% success rate after 2 hours of training. Results also indicate that agents trained on generated trajectories exhibited comparable task performance to those trained using human trajectories. This has demonstrated an example of an extremely low-cost data-efficient way of training reinforcement learning agents.

LLM, Reinforcement Learning, Recommender, Contrast Learning, Generative AI, RLHF, Human Preference, E-commerce^†^†copyright: acmlicensed^†^†journalyear: 2024^†^†conference: fengshuang@gmail.com; August 25-29; Barcelona, Spain^†^†isbn: 978-1-4503-XXXX-X/18/06¹¹1This paper was originally part of the class project for CS234 Spring 2024 in Stanford University and was submitted to KDD’24 on 6/30/2024 for RelKD Workshop. It was accepted in July 2024\. See https://github.com/fengshuang-coding/KDD2024 for updates.

## 1\. Introduction

Recent advances in Large Language Models (LLMs) have significantly enhanced research and applications in understanding human instructions on the web, processing webpage text, and grasping context. These advancements have provided valuable tools for training reinforcement learning (RL) agents to navigate web environments, particularly in e-commerce and various recommender systems such as YouTube and Netflix. Leveraging LLMs in RL agent training is relatively new but has proven successful. A notable example is InstructGPT (Ouyang et al., [2022](https://arxiv.org/html/2408.16032v1#bib.bib12)), where an RL agent was trained using human preferences by fine-tuning GPT-3 models with human instructions. Combining LLMs with RL techniques enables the creation of intelligent web agents that can understand human instructions and complete tasks in web or app environments, thereby maximizing desired rewards.

Recommender systems have evolved from collaborative filtering (Koren et al., [2009](https://arxiv.org/html/2408.16032v1#bib.bib7)) to the recent surge in deep supervised learning, which predicts immediate user responses such as clicks (Covington et al., [2016](https://arxiv.org/html/2408.16032v1#bib.bib5); Zhang et al., [2017](https://arxiv.org/html/2408.16032v1#bib.bib19)). This approach has seen tremendous success in personalized user engagement. However, after several years in production, deep supervised learning algorithms have shown limitations, including: 1) a focus on optimizing short-term gains at the expense of long-term user satisfaction and retention, and 2) strong feedback loops caused by training data generated from these algorithms, which exacerbate these effects. Conversely, RL algorithms are designed to optimize long-term gains by learning policies that maximize long-term user satisfaction. RL agents are also well-known for their ability to perform sequential planning and make decisions based on the Markov Decision Process (MDP) properties (Chen et al., [2018](https://arxiv.org/html/2408.16032v1#bib.bib3)).

The training of RL agents for recommenders in web environments has been actively studied, with several benchmark datasets and trained agents available. For example, WikiNav (Nogueira and Cho, [2017](https://arxiv.org/html/2408.16032v1#bib.bib11)) provides a benchmark for web-based navigation RL agents. RecoGym (Rohde et al., [2018](https://arxiv.org/html/2408.16032v1#bib.bib14)) offers a benchmark for RL agents in production recommendations for online advertising. Virtual-Taobao (Shi et al., [2019](https://arxiv.org/html/2408.16032v1#bib.bib16)) includes a virtual online shopping environment derived from Taobao, hosting several RL algorithms for product recommendations. WebShop (Yao et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib18)) presents a simulated e-commerce web environment with over 1,600 human demonstrations for web shopping tasks based on human text instructions. This environment includes 1.18 million products with text and image descriptions, along with 12,087 crowd-sourced text instructions. The authors of WebShop also explored several imitation and RL agents trained using real-world human trajectories.

Previous explorations in RL for web-based recommenders are extensive. Query reformulation, as published in (Nogueira and Cho, [2017](https://arxiv.org/html/2408.16032v1#bib.bib11)), is part of an RL problem aimed at optimizing outcomes. In this context, search engines are considered black boxes, and the RL agent (or reformulator) learns to generate queries that maximize the expected return through actions in the state space. This paper, published in 2017, predates the widespread use of BERT (Devlin et al., [2019](https://arxiv.org/html/2408.16032v1#bib.bib6)). The authors proposed a PRF framework, with CNN/RNN serving as the contextual learner and query generator. A more recent work proposed the concept of ”learning to search” (Adolphs et al., [2021](https://arxiv.org/html/2408.16032v1#bib.bib2)), where a search agent mimics the interactive process by generating interactive search queries based on previous queries and saving the best queries along the way. The authors used the T5 model with fine-tuning as a query generator to interact with the search engine iteratively, producing a set of fine-grained queries that yield better outcomes. Another related work, WebGPT (Nakano et al., [2021](https://arxiv.org/html/2408.16032v1#bib.bib10)), utilizes a web interface and a search engine to train RL agents to answer questions.

## 2\. Related Work

The work presented in this paper is built and evaluated within the WebShop (Yao et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib18)) environment, a simulator of online web shopping recommender system.

### 2.1\. The WebShop Environment

WebShop is a benchmark project designed to train reinforcement learning algorithms in a large-scale, interactive, web-based environment. It includes over 12,000 crowdsourced human instructions, over 1.1 million products scraped from amazon.com. A total of 670 attributes were derived from concatenated product titles and descriptions using bi-gram representations and assigned to each product through TF-IDF scoring.

Figure 1 and Figure 2 below provide an example WebShop interface and a sequence of actions.

Figure 1\. WebShop Environment (Yao et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib18))

![Refer to caption](img/9660d2ec134356aafbf3575845dc1bb7.png)

Figure 2\. WebShop Human Instructions and Human Trajectories (Yao et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib18))

![Refer to caption](img/bf532f59f53cfb24294a1ab63907eb2f.png)

The original paper tackles the problem into two sets of reinforcement learning models for search and choice (or clicks). The search model is an imitation learning model (search-IL) mimicking human search queries from instructions. It is a human instruction and query pair fine-tuned BART (Lewis et al., [2020](https://arxiv.org/html/2408.16032v1#bib.bib8)) model in root. For choice (clicks) learning, the authors present a few reinforcement learning models to optimize choice of clicks navigating the recommender simulator to optimize the end rewards (purchase). The reward is calculated by a scoring function to quantify the relevance between a purchased product and the human instruction, based on attributes of the product. The imitation learning algorithm (choice-IL) presented by the original authors is a human trajectory fine-tuned BERT (Devlin et al., [2019](https://arxiv.org/html/2408.16032v1#bib.bib6)) model in root. The reinforcement learning algorithm for choice iterates the imitation learned (choice-IL-RL), fine-tuned BERT model as the baseline and iterates the optimization using a mixed objectives of policy gradients and cross entropy.

The state space in this problem consists abstraction of four types of webpages: search page, product recommendation page, product page, and product detail page. The search page features only a search bar for entering instructions, which are used to take a search query either generated by human or a search agent, serving as input for a search engine. Actions include searching, clicking buttons, and choosing from a drop-down menu. Clicking the purchase button marks the end of a trajectory. State transitions are initiated by clicks and other actions that deterministically redirect from one webpage (state) to another. Observations, which includes state and instruction at a specific time snapshot, together form the input for the reinforcement learning agent to make subsequent actions.

The search engine used by the WebShop project is self-built and self-indexed offline using Pyserini (Lin et al., [2021](https://arxiv.org/html/2408.16032v1#bib.bib9)), which is built upon the open-source Lucene search library. Product retrieval is based on BM25 between search queries and product information text. Top 50 results are shown in 5 pages ranked by BM25\.

### 2.2\. Reinforcement Learning with Human Preference - RLHF

RLHF (Christiano et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib4)) together with PPO (Schulman et al., [2017](https://arxiv.org/html/2408.16032v1#bib.bib15)) were successfully used to train a few well-known GPT related product, such as instructGPT (Ouyang et al., [2022](https://arxiv.org/html/2408.16032v1#bib.bib12)). RLHF leverages the Bradley-Terry model, which defines the preference using rewards of preferred and dispreferred data labeled by human labelers:

|  | $P(y_{l}\succ y_{w}&#124;x)=\frac{exp(r(x,y_{l}))}{exp(r(x,y_{l}))+exp(r(x,y_{w}))}.$ |  |

RLHF objectives then can be defined similarly to entropy loss.

### 2.3\. PPO for Regularized Policy Gradient

Proximal Policy Optimization (PPO) (Schulman et al., [2017](https://arxiv.org/html/2408.16032v1#bib.bib15)) has been demonstrated to be effective in fine-tuning GPT models with human instructions and labeled preferences (Ouyang et al., [2022](https://arxiv.org/html/2408.16032v1#bib.bib12)). PPO uses clipping or KL divergence constraints to minimize the likelihood of large updates between steps, approximately providing guarantees for monotonic improvement. This approach converges in probability to local optima and, in practice, results in more stable training outcomes. The clipped loss function for policy gradient in PPO can be expressed as:

| (1) |  | $L_{\theta_{k}}^{PPO}=-E_{\tau\sim\pi_{k}}\left[min(z_{t}(\theta)\hat{A}_{t}^{% \pi_{k}},clip(z_{t}(\theta),1-\epsilon,1+\epsilon)\hat{A}_{t}^{\pi_{k}}\right]$ |  |

, where

|  | $z_{t}(\theta)=\frac{\pi_{\theta}(a_{t}&#124;o_{t})}{\pi_{\theta_{k}}(a_{t}&#124;o_{t})},$ |  |

|  | $\hat{A}_{t}^{\pi_{k}}=R_{t}-V^{\pi_{k}}(o_{t}).$ |  |

### 2.4\. Learning with Human Preference - DPO

The development of Direct Preference Optimization (DPO) (Rafailov et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib13)) is revolutionary. It eliminates the need for explicit reward functions for preferences and instead relies solely on paired preference trajectories as training data. DPO is derived from joining the Bradley-Terry objective

|  | $L_{BT}(r,D)=-E_{(x,y_{w},y_{l})\sim D}\left[log\ \sigma\left(r(x,y_{w})-r(x,y_% {l})\right)\right]$ |  |

, and the RLHF objective:

|  | $\smash{\displaystyle\max_{\pi}}\ E_{x\sim D,y\sim\pi}[r(x,y)]-\beta D_{KL}[\pi% (y&#124;x)&#124;&#124;\pi_{ref}(y&#124;x)].$ |  |

The DPO loss function is:

|  | $\begin{split}L_{DPO}(\pi_{\theta},\pi_{ref})&=-E_{(x,y_{w},y_{l})\sim D}[log\ % \sigma(\beta log\frac{\pi_{\theta}(y_{w}&#124;x)}{\pi_{ref}(y_{w}&#124;x)}\\ &-\beta log\frac{\pi_{\theta}(y_{l}&#124;x)}{\pi_{ref}(y_{l}&#124;x)})].\end{split}$ |  |

, where $\pi_{\theta}$ is the DPO policy to learn and $\pi_{ref}$ is the pre-selected reference policy. From the form of the loss function, although DPO does not need a reward model to be trained explicitly, it does require a pre-defined reference policy to iterate upon.

## 3\. Approach

This report summarizes a few efforts implementing and evaluating PPO vs. contrast learning using DPO using human trajectories and generated unpreferred trajectories. Then it demonstrates the contrast learning effort using all generative trajectories.

We branched and implemented DPO and PPO using the original WebShop code package, together with a new generative module for the self-generative learning experiments, and a Thompson sampling module to roll out online experiments and collect results.

For PPO training, the policy gradient objective from the original paper (Yao et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib18)) is modified into a PPO objective as shown in equation (1). The overall objective components, which is the total loss from policy gradient (PG), entropy loss, and imitation learning loss remain the same as in the original paper, except PG component is replaced with PPO loss.

### 3.1\. Semi-generative Reinforcement Learning Using Human Trajectories

For this project, we utilize a pre-trained imitation learning agent checkpoint as the reference policy to generate unpreferred trajectories. Preferred trajectories are obtained from human data provided by the WebShop benchmark. During training, a human trajectory is randomly sampled, including states and available actions from the log. At each state where an action decision is needed, an unpreferred action is generated using the reference policy. This unpreferred action is paired with the preferred action generated by the human. The DPO update is applied after each episode based on the human trajectory.

This approach is considered both generative and semi-self-learning. It is generative because we use a predefined unpreferred policy to generate actions for pair-wise training. It is semi-self-learning because it pairs these generated actions with previously collected human trajectories, which serve as the gold standard.

### 3.2\. Self-learning - Training with Generated Trajectories

In classic reinforcement learning, self-play or learning through simulation plays a crucial role, particularly when data collection is costly, such as the collection of human trajectories in this problem. Self-play has proven to be effective, with the most notable example being AlphaGo (Silver et al., [2016](https://arxiv.org/html/2408.16032v1#bib.bib17)).

To evaluate the idea of self-play or self-learning in navigating the WebShop recommendation systems, we generated 100 preferred trajectories using a straightforward method of sampling trajectories with perfect reward (score = 1). This sampling was done using the agent checkpoint from imitation learning provided by the authors of the WebShop paper, but with real-world human instructions.

Ideally, these sampled trajectories are pruned to eliminate looped sub-trajectories. A DPO agent is then trained from the same checkpoint used for DPO evaluation in the previous section, with 3000 steps. Task performance between these two DPO agents — one trained using semi-learning with human trajectories and the other using self-learning with generated trajectories — is compared using Thompson sampling ran in the WebShop simulator environment.

## 4\. Experimental Results

### 4.1\. DPO vs. PPO Task Performance

In this project, leveraging the WebShop environment and simulator, we conduct extensive simulated online experiments using Thompson sampling to analyze the performance differences across trained agents. The goal of Thompson sampling is to select the optimal action (or ”arm”) that minimizes overall regret. However, when sampling over a small number of steps, it may not be ideal for estimating rewards from arms that are perceived as less optimal due to insufficient exploration. To address this, we use multiple parallel runs of Thompson sampling, each with 1000 rollouts, to capture variability across runs. Careful experimental design and calculated rollouts of online experiments are necessary for accurately estimating the rewards and success rates of each agent. The aim of this project is to implement and understand the performance trends across different approaches.

The results indicate that Direct Preference Optimization (DPO) agents achieve significantly higher scores and success rates compared to Proximal Policy Optimization (PPO) agents, even though all agents start from the same imitation learning BERT model checkpoint provided by the original paper. It is important to note that all agents in this comparison are trained without image data, so the scores and success rates collected are not directly comparable to the original paper, which includes image data in training and experiments.

An interesting finding is that DPO agents trained using human trajectories perform similarly to DPO agents trained using generated trajectories, albeit with larger variance in success rate across runs. The smaller variance observed in self-learning agents can be attributed to the fact that only 100 generated trajectories were used to train the DPO self-learning agent, compared to 1200 human trajectories used for training the DPO agent with human data.

The fact that DPO agents are trained using only 3000 steps also suggests the possibility of underestimating data inefficiency or bottleneck when training over long period of times using the same set of data. When training an agent for production systems, the limited number of available trajectories can result in decreased task performance due to insufficient information learned from the limited data. In reality, collecting human data is expensive and time-consuming. This issue can be mitigated by generating preferred and unpreferred trajectories to serve as a continuous, low-cost source of training data.

Figure 3\. DPO vs. PPO — Human Trajectories and Generated Trajectories — Scores

![Refer to caption](img/c47a2f921610764828761b70ff6588fb.png)

Figure 4\. DPO vs. PPO — Human Trajectories and Generated Trajectories — Success Rate

![Refer to caption](img/80017ad44bf2f9df2eb4d0588ee928ff.png)

It is important to note that the results of this project are not directly comparable to those of the original paper due to two key differences: 1) no image data were used for training or experiments for any of the agents evaluated in this project, and 2) each agent was trained with minimal steps (3000) and within a timeframe of less than one hour. The purpose of this project is not to benchmark results but to investigate variations in reinforcement learning algorithms.

Training using fully generated preferences on top of the DPO agent achieved much higher scores than DPO agents using human trajectories, while the success rate remained similar (Figure 5, Figure 6). The magnitude of this difference needs to be justified using variance across runs, but this finding demonstrates the potential of using generative data to enhance training on top of existing agents initially trained with human trajectories.

Figure 5\. Self-learning Using Generated Trajectories — Scores

![Refer to caption](img/69f35f68be689dbe595831d7a1f8c01e.png)

Figure 6\. Self-learning Using Generated Trajectories — Success Rate

![Refer to caption](img/2faa443bcdec38badd119ff7ce85fff5.png)

## 5\. Conclusion

With very limited training time (¡1 hour), Direct Preference Optimization (DPO) outperforms Proximal Policy Optimization (PPO), offering better task performance and higher success rates with less training time. However, more evaluations with longer training time are necessary to draw a conclusion. Using a DPO agent trained within one hour and without image data, we achieved a success rate of approximately 19%. This is higher than the success rate of an RL agent trained with an RNN network (without pre-trained models for search or choice imitation learning), which has an 18% success rate from the original paper.

PPO is known to be able to provide less volatile training and approximately monotonic guarantees for RL objectives. By nature, PPO’s regularization and clipping of objectives prevent rapid policy changes, making it suitable for problems with smaller state and action spaces where large policy changes are not expected. However, in the context of online product recommenders, where the state-action space can expand to millions of dimensions, and rapid policy changes are essential for fast learning, PPO can need longer time to train.

Training DPO agents with generated trajectories has shown great potential. With only 100 generated human trajectories and the same amount of computational resources, the task performance was comparable to a DPO agent trained using 1200 human trajectories. This approach addresses data inefficiency and the high costs of human data collection. As training requires more time and data, the limited availability of human data can hinder continuous improvement. This exercise demonstrates that generated trajectories can be nearly as effective as human trajectories and can even serve as a continuous, low-cost source of training data. Additionally, generated trajectories allow exploration of successful paths not seen by humans, similar to the approach that contributed to the success of AlphaGo(Silver et al., [2016](https://arxiv.org/html/2408.16032v1#bib.bib17)) and AlphaZero, which were trained using self-play rather than past human games.

## 6\. Potential Usage: Using Trained Agents as a Recommender

Using reinforcement learning agent in recommenders is not new - it is known to be used in online recommenders such as Youtube. The trained optimal policy can become an ideal ranking algorithm for recommender systems. Starting from a human instruction, the agent simulates navigating through a provided list of product following the trained optimal policy and provides a ”purchased” product from each run. When using in recommenders, multiple runs of the agent provide a list of recommended product to user and the order to present in a user interface, such as on web or in an app can be rank-ordered by scores or success for each recommended product from each run of the RL agent.

###### Acknowledgements.

To the WebShop authors Shunyu Yao, Howard Chen, John Yang and Karthik Narasimhan from Princeton University who published (Yao et al., [2023](https://arxiv.org/html/2408.16032v1#bib.bib18)), which inspired this project report.
To Professor Chris Potts who has introduced WebShop [17] to the author of this report, and for his excellent teaching CS224U in Stanford University.
To Professor Emma Brunskill for her excellent teaching CS234 in Stanford University.

## References

*   (1)
*   Adolphs et al. (2021) Leonard Adolphs, Benjamin Börschinger, Christian Buck, Michelle Chen Huebscher, Massimiliano Ciaramita, Lasse Espeholt, Thomas Hofmann, and Yannic Kilcher. 2021. Boosting Search Engines with Interactive Agents. *CoRR* abs/2109.00527 (2021). arXiv:2109.00527 [https://arxiv.org/abs/2109.00527](https://arxiv.org/abs/2109.00527)
*   Chen et al. (2018) Minmin Chen, Alex Beutel, Paul Covington, Sagar Jain, Francois Belletti, and Ed H. Chi. 2018. Top-K Off-Policy Correction for a REINFORCE Recommender System. *CoRR* abs/1812.02353 (2018). arXiv:1812.02353 [http://arxiv.org/abs/1812.02353](http://arxiv.org/abs/1812.02353)
*   Christiano et al. (2023) Paul Christiano, Jan Leike, Tom B. Brown, Miljan Martic, Shane Legg, and Dario Amodei. 2023. Deep reinforcement learning from human preferences. arXiv:1706.03741 [stat.ML]
*   Covington et al. (2016) Paul Covington, Jay Adams, and Emre Sargin. 2016. Deep Neural Networks for YouTube Recommendations. In *Proceedings of the 10th ACM Conference on Recommender Systems* (Boston, Massachusetts, USA) *(RecSys ’16)*. Association for Computing Machinery, New York, NY, USA, 191–198. [https://doi.org/10.1145/2959100.2959190](https://doi.org/10.1145/2959100.2959190)
*   Devlin et al. (2019) Jacob Devlin, Ming-Wei Chang, Kenton Lee, and Kristina Toutanova. 2019. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding. (2019). arXiv:1810.04805 [cs.CL]
*   Koren et al. (2009) Yehuda Koren, Robert Bell, and Chris Volinsky. 2009. Matrix Factorization Techniques for Recommender Systems. *Computer* 42, 8 (2009), 30–37. [https://doi.org/10.1109/MC.2009.263](https://doi.org/10.1109/MC.2009.263)
*   Lewis et al. (2020) Mike Lewis, Yinhan Liu, Naman Goyal, Marjan Ghazvininejad, Abdelrahman Mohamed, Omer Levy, Veselin Stoyanov, and Luke Zettlemoyer. 2020. BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension. In *Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics*. Association for Computational Linguistics, Online, 7871–7880. [https://doi.org/10.18653/v1/2020.acl-main.703](https://doi.org/10.18653/v1/2020.acl-main.703)
*   Lin et al. (2021) Jimmy Lin, Xueguang Ma, Sheng-Chieh Lin, Jheng-Hong Yang, Ronak Pradeep, and Rodrigo Frassetto Nogueira. 2021. Pyserini: An Easy-to-Use Python Toolkit to Support Replicable IR Research with Sparse and Dense Representations. *CoRR* abs/2102.10073 (2021). arXiv:2102.10073 [https://arxiv.org/abs/2102.10073](https://arxiv.org/abs/2102.10073)
*   Nakano et al. (2021) Reiichiro Nakano, Jacob Hilton, Suchir Balaji, Jeff Wu, Long Ouyang, Christina Kim, Christopher Hesse, Shantanu Jain, Vineet Kosaraju, William Saunders, Xu Jiang, Karl Cobbe, Tyna Eloundou, Gretchen Krueger, Kevin Button, Matthew Knight, Benjamin Chess, and John Schulman. 2021. WebGPT: Browser-assisted question-answering with human feedback. *CoRR* abs/2112.09332 (2021). arXiv:2112.09332 [https://arxiv.org/abs/2112.09332](https://arxiv.org/abs/2112.09332)
*   Nogueira and Cho (2017) Rodrigo Frassetto Nogueira and Kyunghyun Cho. 2017. Task-Oriented Query Reformulation with Reinforcement Learning. *CoRR* abs/1704.04572 (2017). arXiv:1704.04572 [http://arxiv.org/abs/1704.04572](http://arxiv.org/abs/1704.04572)
*   Ouyang et al. (2022) Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, John Schulman, Jacob Hilton, Fraser Kelton, Luke Miller, Maddie Simens, Amanda Askell, Peter Welinder, Paul F Christiano, Jan Leike, and Ryan Lowe. 2022. Training language models to follow instructions with human feedback. In *Advances in Neural Information Processing Systems*, S. Koyejo, S. Mohamed, A. Agarwal, D. Belgrave, K. Cho, and A. Oh (Eds.), Vol. 35\. Curran Associates, Inc., 27730–27744. [https://proceedings.neurips.cc/paper_files/paper/2022/file/b1efde53be364a73914f58805a001731-Paper-Conference.pdf](https://proceedings.neurips.cc/paper_files/paper/2022/file/b1efde53be364a73914f58805a001731-Paper-Conference.pdf)
*   Rafailov et al. (2023) Rafael Rafailov, Archit Sharma, Eric Mitchell, Stefano Ermon, Christopher D. Manning, and Chelsea Finn. 2023. Direct Preference Optimization: Your Language Model is Secretly a Reward Model. (2023). arXiv:2305.18290 [cs.LG]
*   Rohde et al. (2018) David Rohde, Stephen Bonner, Travis Dunlop, Flavian Vasile, and Alexandros Karatzoglou. 2018. RecoGym: A Reinforcement Learning Environment for the problem of Product Recommendation in Online Advertising. *CoRR* abs/1808.00720 (2018). arXiv:1808.00720 [http://arxiv.org/abs/1808.00720](http://arxiv.org/abs/1808.00720)
*   Schulman et al. (2017) John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, and Oleg Klimov. 2017. Proximal Policy Optimization Algorithms. *CoRR* abs/1707.06347 (2017). arXiv:1707.06347 [http://arxiv.org/abs/1707.06347](http://arxiv.org/abs/1707.06347)
*   Shi et al. (2019) Jing-Cheng Shi, Yang Yu, Qing Da, Shi-Yong Chen, and An-Xiang Zeng. 2019. Virtual-Taobao: Virtualizing Real-World Online Retail Environment for Reinforcement Learning. *Proceedings of the AAAI Conference on Artificial Intelligence* 33, 01 (Jul. 2019), 4902–4909. [https://doi.org/10.1609/aaai.v33i01.33014902](https://doi.org/10.1609/aaai.v33i01.33014902)
*   Silver et al. (2016) David Silver, Aja Huang, Christopher J. Maddison, Arthur Guez, Laurent Sifre, George van den Driessche, Julian Schrittwieser, Ioannis Antonoglou, Veda Panneershelvam, Marc Lanctot, Sander Dieleman, Dominik Grewe, John Nham, Nal Kalchbrenner, Ilya Sutskever, Timothy Lillicrap, Madeleine Leach, Koray Kavukcuoglu, Thore Graepel, and Demis Hassabis. 2016. Mastering the game of Go with deep neural networks and tree search. *Nature* 529 (2016), 484–503. [http://www.nature.com/nature/journal/v529/n7587/full/nature16961.html](http://www.nature.com/nature/journal/v529/n7587/full/nature16961.html)
*   Yao et al. (2023) Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. 2023. WebShop: Towards Scalable Real-World Web Interaction with Grounded Language Agents. (2023). arXiv:2207.01206 [cs.CL]
*   Zhang et al. (2017) Shuai Zhang, Lina Yao, and Aixin Sun. 2017. Deep Learning based Recommender System: A Survey and New Perspectives. *CoRR* abs/1707.07435 (2017). arXiv:1707.07435 [http://arxiv.org/abs/1707.07435](http://arxiv.org/abs/1707.07435)