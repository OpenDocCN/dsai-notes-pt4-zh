<!--yml
category: 未分类
date: 2025-01-11 11:55:00
-->

# Next-Generation Phishing: How LLM Agents Empower Cyber Attackers ††thanks: § § \mathsection § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.

> 来源：[https://arxiv.org/html/2411.13874/](https://arxiv.org/html/2411.13874/)

Khalifa Afane^*, Wenqi Wei^*, Ying Mao^*, Junaid Farooq^†, and Juntao Chen^(*$\mathsection$)
^*Department of Computer and Information Sciences, Fordham University, New York, NY, USA
^†Department of Electrical and Computer Engineering, University of Michigan-Dearborn, Dearborn, MI, USA
{mafane, wenqiwei, ymao41, jchen504}@fordham.edu, mjfarooq@umich.edu

###### Abstract

The escalating threat of phishing emails has become increasingly sophisticated with the rise of Large Language Models (LLMs). As attackers exploit LLMs to craft more convincing and evasive phishing emails, it is crucial to assess the resilience of current phishing defenses. In this study we conduct a comprehensive evaluation of traditional phishing detectors, such as Gmail Spam Filter, Apache SpamAssassin, and Proofpoint, as well as machine learning models like SVM, Logistic Regression, and Naive Bayes, in identifying both traditional and LLM-rephrased phishing emails. We also explore the emerging role of LLMs as phishing detection tools, a method already adopted by companies like NTT Security Holdings and JPMorgan Chase. Our results reveal notable declines in detection accuracy for rephrased emails across all detectors, highlighting critical weaknesses in current phishing defenses. As the threat landscape evolves, our findings underscore the need for stronger security controls and regulatory oversight on LLM-generated content to prevent its misuse in creating advanced phishing attacks. This study contributes to the development of more effective Cyber Threat Intelligence (CTI) by leveraging LLMs to generate diverse phishing variants that can be used for data augmentation, harnessing the power of LLMs to enhance phishing detection, and paving the way for more robust and adaptable threat detection systems.

###### Index Terms:

Large language models, Cybersecurity, Email phishing detection, Semantic evasion.

## I Introduction

Phishing emails remain a prevalent and persistent threat in cybersecurity, often exploiting human psychology to deceive recipients into revealing sensitive information [[1](https://arxiv.org/html/2411.13874v1#bib.bib1)], [[2](https://arxiv.org/html/2411.13874v1#bib.bib2)], [[3](https://arxiv.org/html/2411.13874v1#bib.bib3)]. Which directly led to large organizations averaging a loss of $15 million in 2023 [[4](https://arxiv.org/html/2411.13874v1#bib.bib4)]. Traditional phishing detectors have achieved high precision and recall by recognizing specific linguistic cues, effectively mitigating many phishing attempts. However, the rapid advancement of Large Language Models (LLMs) has introduced new complexities into this landscape. These sophisticated models has already outperformed domain experts on cybersecurity benchmarks like CyberMetric[[5](https://arxiv.org/html/2411.13874v1#bib.bib5)], making them useful for crafting more nuanced and convincing phishing emails, rendering classical phishing datasets and detection methods increasingly less effective. As a result, LLM-generated phishing emails pose a significant new threat that needs to be urgently addressed. This challenge has been further exacerbated by advancements in prompt engineering techniques, such as zero-shot and few-shot prompting, which are effective for new tasks without requiring training data, enabling attackers to generate highly targeted and contextually accurate emails with minimum effort  [[6](https://arxiv.org/html/2411.13874v1#bib.bib6)], [[7](https://arxiv.org/html/2411.13874v1#bib.bib7)].

Prior to the rise of LLMs significant research had already enhanced phishing detection methods. For instance, Fette et al. [[8](https://arxiv.org/html/2411.13874v1#bib.bib8)] introduced a machine learning approach focusing on features designed to detect deception, successfully identifying over 99.5% of phishing emails with a very low false positive rate. Ma et al. [[9](https://arxiv.org/html/2411.13874v1#bib.bib9)] expanded on this by using hybrid features, combining content-based keywords and phrases with attributes like forms, and mismatched URLs, to build robust classifiers. Similarly, Verma et al. [[10](https://arxiv.org/html/2411.13874v1#bib.bib10)] explored natural language processing techniques, highlighting the effectiveness of semantic analysis in identifying phishing attempts. Other techniques have also proven to be very effective, achieving a near-perfect accuracy  [[11](https://arxiv.org/html/2411.13874v1#bib.bib11)],  [[12](https://arxiv.org/html/2411.13874v1#bib.bib12)].

![Refer to caption](img/b1a6671771391162e6bd8f9e477d5902.png)

Figure 1: Evaluation methodology workflow, highlighting the differences in detection effectiveness on average between traditional and LLM-rephrased emails.

The dual-use nature of LLMs makes them particularly relevant in this context, as they are highly effective in generating both beneficial and malicious content. Wu et al. [[13](https://arxiv.org/html/2411.13874v1#bib.bib13)] and Yao et al. [[14](https://arxiv.org/html/2411.13874v1#bib.bib14)] highlight security concerns posed by LLMs, noting their ability to craft sophisticated phishing emails that evade traditional detection methods. Additionally, LLMs have been shown to generate personalized phishing messages at scale, realistically and cost-effectively [[15](https://arxiv.org/html/2411.13874v1#bib.bib15)]. LLMs such as Llama [[16](https://arxiv.org/html/2411.13874v1#bib.bib16)], Gemini [[17](https://arxiv.org/html/2411.13874v1#bib.bib17)], Claude [[18](https://arxiv.org/html/2411.13874v1#bib.bib18)], and GPT [[19](https://arxiv.org/html/2411.13874v1#bib.bib19)] models demonstrate remarkable proficiency in generating human-like text, increasingly applied to tasks like phishing detection [[20](https://arxiv.org/html/2411.13874v1#bib.bib20)], [[21](https://arxiv.org/html/2411.13874v1#bib.bib21)]. Building on prior research, this study underscores the dual nature of these models: they can craft sophisticated phishing emails while also enhancing Cyber Threat Intelligence (CTI) by improving phishing detectors through augmented data training.

The rest of the paper is organized as follows: Section II reviews related works on phishing detection and the challenges posed by LLM-generated phishing emails. Section III presents the methodology, including the datasets used, experimental setup, and rephrasing techniques. Section IV discusses the experimental results, highlighting the performance of phishing detectors and machine learning models. Section V provides a discussion of the findings, outlines limitations, and suggests future research directions. Finally, Section VI concludes the paper with a summary of key insights and contributions.

## II Related Works

Phishing email detection has traditionally relied on non-LLM-based methods, such as Google’s built-in Gmail spam filter, other prominent tools like SpamAssassin [[22](https://arxiv.org/html/2411.13874v1#bib.bib22)] and Proofpoint have also proven to be highly effective detectors [[23](https://arxiv.org/html/2411.13874v1#bib.bib23)]. However, the integration of LLM-based phishing detection is an emerging phenomenon [[25](https://arxiv.org/html/2411.13874v1#bib.bib25)], with companies like JPMorgan Chase leading the charge [[24](https://arxiv.org/html/2411.13874v1#bib.bib24)], and NTT Holdings introducing their framework ChatSpamDetector [[21](https://arxiv.org/html/2411.13874v1#bib.bib21)].

However, as phishing detection improves, LLMs are being harnessed to escalate the sophistication of phishing attacks, posing new challenges. Hazell  [[15](https://arxiv.org/html/2411.13874v1#bib.bib15)]. for example, explored the potential of LLMs to scale spear-phishing campaigns by generating personalized emails for over 600 British Members of Parliament using GPT-3.5 and GPT-4 models. The findings show that these models can create realistic and cost-effective spear-phishing emails, with each email costing only a fraction of a cent. Similarly, Heiding et al. [[20](https://arxiv.org/html/2411.13874v1#bib.bib20)] investigated the use of LLMs combined with V-Triad in email phishing generation and found that models like GPT-4 when pared with human knowledge are capable of generating highly convincing phishing emails that can evade traditional detection methods. Kang et al.  [[26](https://arxiv.org/html/2411.13874v1#bib.bib26)] further explored the use of LLMs in various malicious tasks, concluding that these models are capable of generating well-designed content that can be exploited for phishing and other scams.

In contrast to previous studies, our work comprehensively evaluates the detection capabilities of state-of-the-art phishing detectors, including Google’s Gmail Spam Filter, SpamAssassin, and Proofpoint, as well as LLMs, on both original and LLM-rephrased phishing emails as illustrated in Fig. [1](https://arxiv.org/html/2411.13874v1#S1.F1 "Figure 1 ‣ I Introduction ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant."). Our approach employs various machine learning techniques, which represents the workflow of our evaluation methodology. Specifically, Fig. [1](https://arxiv.org/html/2411.13874v1#S1.F1 "Figure 1 ‣ I Introduction ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") highlights the differences in detection effectiveness between traditional phishing emails and LLM-rephrased emails, demonstrating the challenges faced by current phishing detectors in handling AI-generated threats. Roy et al. [[27](https://arxiv.org/html/2411.13874v1#bib.bib27)] tackled the issue of LLM-generated phishing attacks from a prompt engineering perspective, exploring the crafting of malicious prompts and utilizing LLMs to design these prompts. While their approach can mitigate the risk of LLM-generated phishing attacks, we believe that a more effective approach is to train models to detect these threats via better training data. Our approach complements the work of Roy et al. by focusing on improving the detection capabilities of phishing detectors, rather than solely relying on prompt engineering and detection. We focus on the Nazario and Nigerian Fraud datasets  [[28](https://arxiv.org/html/2411.13874v1#bib.bib28)] and utilize GPT-4o and Llama 3 to rephrase phishing emails. We summarize the contributions of this paper as follows:

1.  1.

    We conduct a comprehensive comparison between traditional phishing emails and those rephrased by LLMs. This comparison evaluates the performance of various phishing detection tools, such as Google’s Gmail Spam Filter, SpamAssassin, Proofpoint, and other machine learning models, as well as LLMs such as Llama, Gemini, Claude, and GPT as phishing detectors .

2.  2.

    We introduce a framework for effective data augmentation to create phishing email datasets that better reflect modern threats. Using GPT-4 and Llama 3, we rephrase and augment phishing emails from the Nazario dataset to create the LLM-Nazario dataset, which includes both original and rephrased emails as well as newly generated phishing attacks. The effectiveness of this approach is demonstrated by training three machine learning models on the new dataset to assess their improved ability to detect advanced threats like LLM-rephrased phishing emails.

## III Methods

### III-A Data Collection and Construction

We utilized two primary datasets for our experiments: the Nazario and Nigerian Fraud phishing email datasets [[28](https://arxiv.org/html/2411.13874v1#bib.bib28)]. The Nazario dataset originally contained 2904 instances after cleaning, but for our experiments, we sampled 1200 emails evenly divided between the two classes: 600 legitimate emails and 600 phishing emails. The legitimate emails were originally sourced from benign online discussions and newsletters, while the phishing emails followed the traditional format of including deceptive requests and fake links. The emails varied in length, ranging from 10 to 350 characters.

The Nigerian Fraud datasets served as a second source for our experiments. From these datasets, we selected 800 emails, evenly divided between legitimate and phishing . These phishing emails were on average longer than the Nazario dataset emails, with a range from 10 to 650 characters, primarily featuring financial scams involving offers of large sums of money or requests for personal information.

In both datasets, the features retained for the experiments were sender, receiver, subject, and body. These features are essential as they provide valuable context for classification, particularly with regard to embedded links such as “Validate Password” or “Download Files.” The rephrasing of phishing emails, as described in the next section, was crucial in evaluating the performance of detection systems against these altered versions.

### III-B Experimental Setup

Our experimental setup involved testing three traditional phishing detection systems; Google’s Gmail Spam Filter, SpamAssassin, and Proofpoint, three machine learning models; SVM, Logistic Regression, and Naive Bayes, and five prominent LLMs; Llama 3, Gemini 1.5, Claude 3 Sonnet, GPT-3.5, and GPT-4o.

For the Gmail Spam Filter, we used different email accounts to simulate the user environment. Phishing and legitimate emails were sent to these accounts, and we recorded whether each email was automatically moved to the spam folder or shown in the user’s inbox. This binary decision served as Gmail’s detection metric across three categories of emails: original phishing emails, zero-shot rephrased emails, and few-shot rephrased emails.

Apache SpamAssassin applies a variety of content-based and statistical techniques to assess emails for potential phishing or spam. In our setup, we tracked whether each email was flagged as spam based on its evaluation criteria. Proofpoint, utilizing advanced email security technologies, was used to classify emails across the same categories, we recorded the emails that were shown to the receiver and the the emails that were put into the different ”Quarantine” folders.

For the three machine learning models, we applied three text encoding techniques: Bag of Words [[29](https://arxiv.org/html/2411.13874v1#bib.bib29)], Term Frequency-Inverse Document Frequency (TF-IDF) [[30](https://arxiv.org/html/2411.13874v1#bib.bib30)], and Word2Vec [[31](https://arxiv.org/html/2411.13874v1#bib.bib31)], these models were trained on other subsets of the datasets used in the study. On average, TF-IDF encoding was 2.6% more accurate than Bag of Words and 5.4% more accurate than Word2Vec in detecting phishing emails. These encoding methods convert email text into numerical representations suitable for classification. Therefore, the results shown in Table [I](https://arxiv.org/html/2411.13874v1#S4.T1 "TABLE I ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") and Table [III](https://arxiv.org/html/2411.13874v1#S4.T3 "TABLE III ‣ IV-B Nigerian Fraud Dataset Experiments ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") for the 3 machine learning models were obtained using TF-IDF encoding. The same encoding technique was also applied when testing the three models on rephrased emails after data augmentation.

Each of the five LLMs was tested on the same three categories of emails: (1) original phishing emails, (2) emails rephrased using GPT-4o with zero-shot prompting, and (3) emails rephrased using GPT-4o with few-shot prompting. For the zero-shot prompting, we provided a simple instruction to rephrase the email while maintaining the same core content. For the few-shot prompting, we used three example rephrased phishing emails to guide the LLMs.

All experiments were performed in a controlled environment where the LLMs were prompted, and the results were validated across three iterations to mitigate the influence of non-deterministic behavior in the models. For consistency, we used a majority vote approach to finalize the classification for each email. This comprehensive setup allowed us to compare the performance of the three traditional email detection systems and the five LLMs on original and rephrased phishing emails, providing valuable insights into the strengths and weaknesses of each system, particularly in detecting rephrased phishing emails designed to evade detection.

### III-C Rephrasing Techniques

The two techniques, zero-shot prompting and few-shot prompting, were chosen due to their proven effectiveness in prior studies, particularly for new tasks that lack extensive training data, which aligns with our requirements [[6](https://arxiv.org/html/2411.13874v1#bib.bib6)].

#### III-C1 Zero-Shot Prompting

Zero-shot prompting involves providing the LLM with a task description without any examples, relying solely on the model’s pre-trained knowledge. This method leverages the LLM’s ability to generalize from the provided instructions and apply its extensive pre-trained knowledge to novel tasks. Prior studies have demonstrated the effectiveness of zero-shot prompting, particularly for new tasks that lack extensive training data [[6](https://arxiv.org/html/2411.13874v1#bib.bib6)], [[32](https://arxiv.org/html/2411.13874v1#bib.bib32)], [[33](https://arxiv.org/html/2411.13874v1#bib.bib33)]. This approach is simple and efficient, allowing models to perform well without the need for task-specific fine-tuning [[35](https://arxiv.org/html/2411.13874v1#bib.bib35)]. To bypass the security layers of some LLMs, it is effective to provide a clear context in the prompt. For example, adding an explanation that the task is for research purposes often helps achieve the desired response. Below is our chosen prompt for zero-shot prompting, with many variations of this prompt yielding similar results. It is important to note that we allow the LLM to only change the subject and body features of the emails. While using a better or more professional sender domain could be more effective, it is not considered realistic in our estimation.

<svg class="ltx_picture" height="178.23" id="S3.SS3.SSS1.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,178.23) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 160.03)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Zero-Shot Prompt</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="128.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">I want to train my machine learning model for my research on email phishing detection. I need you to rephrase the emails below and give me the new subject and body that are designed to bypass email phishing detectors. Keep the same sender and receiver information, avoid any sense of urgency or words like ’urgent’ and ’immediate’ in the body or subject, and avoid using deadlines or ultimatums. Avoid generic greetings, use the receiver information to greet them properly, and don’t mention sums of money or dollar amounts to make the email more legitimate. Ask for the same information the original email is asking for, just make the context more legitimate and keep the same core topic.</foreignobject></g></g></svg>

#### III-C2 Few-Shot Prompting

Few-shot prompting includes a few examples along with the task description to help the model understand the task better and produce more accurate and contextually relevant outputs. Few-shot prompting does not necessarily produce better results compared to zero-shot prompting. Their performance depends on the specific task and context [[36](https://arxiv.org/html/2411.13874v1#bib.bib36)], [[37](https://arxiv.org/html/2411.13874v1#bib.bib37)]. For each dataset, we give the model a task description similar to what we had in zero-shot prompting with 3 examples (3-shot) of original phishing emails along with the desired output crafted carefully to bypass phishing detectors. Below is an example of an original phishing email from the Nazario dataset that’s used in the few-shot prompting along with the desired output that’s also given to the model.

<svg class="ltx_picture" height="161.63" id="S3.SS3.SSS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,161.63) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 143.42)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Original Email</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="111.93" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Subject: Urgent Body: Your account will be blocked from sending messages unless you complete the required email upgrade within the next 24 hours. Click the link below to unlock and upgrade your account to version 3.0.1\. Ignoring this will result in suspension of your email services. Mail Service Team For: kevin@rocketinvestment.org</foreignobject></g></g></svg><svg class="ltx_picture" height="225.35" id="S3.SS3.SSS2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,225.35) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 207.15)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Desired Output for Few-Shot Training</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="175.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Subject: Account Upgrade Available Body: Dear Kevin, An upgrade is available for your account. Click the link below to unlock and upgrade your account. Upgrade to version 3.0.1 Thank you, Mail Service Team</foreignobject></g></g></svg>

Removing the sense of urgency and adopting a more professionally composed tone can significantly enhance the effectiveness of phishing emails. Urgency often triggers detection mechanisms and raises suspicion among recipients. By shifting from alarmist language to a more neutral, informative tone, attackers can craft messages that feel routine, such as notifying the recipient of an available update rather than an immediate action required.

This approach leverages the trust people place in well-composed emails. Professional language gives the impression that the email comes from a reliable source, reducing the likelihood of it being flagged as suspicious. It also plays on human psychology by avoiding the pressure of urgency, making recipients feel more comfortable and less defensive, increasing the chance they will engage with the email. The subtlety of this tactic is crucial. Instead of overt manipulation, the email becomes a benign-sounding request that blends into the recipient’s usual communications.

### III-D Evaluation Metrics

The performance of phishing detectors was evaluated using the following metrics:

*   •

    True Positive (TP): Correctly identified phishing emails.

*   •

    True Negative (TN): Correctly identified legitimate emails.

*   •

    False Positive (FP): Legitimate emails incorrectly classified as phishing.

*   •

    False Negative (FN): Phishing emails incorrectly classified as legitimate.

*   •

    Accuracy: $(TP+TN)/(TP+TN+FP+FN)$.

*   •

    Precision: $TP/(TP+FP)$.

*   •

    Recall: $TP/(TP+FN)$, also referred to as the detection rate.

*   •

    F1 Score:

    |  | $F1=2\times\frac{\text{Precision}\times\text{Recall}}{\text{Precision}+\text{% Recall}}.$ |  | (1) |

For example, an FP rate of 0.2 means 20% of legitimate emails are incorrectly flagged, while an FN rate of 0.15 means 15% of phishing emails are missed. These metrics are crucial for evaluating the reliability of phishing detection systems, with the false negative rate being particularly important as it indicates the proportion of phishing threats that go undetected and potentially cause harm.

## IV Experimental Results

The results of the experiments demonstrate a significant shift in the decision boundary when comparing traditional phishing emails to those rephrased using LLMs. By analyzing a sample of 200 emails composed from the different datasets used in this study, we observed that traditional phishing emails exhibited clearer decision boundaries. These boundaries were largely driven by the occurrence of certain keywords such as “royal family” “urgent”, and “large payment,” which are flagged as highly suspicious by phishing detectors.

![Refer to caption](img/8c1ffefef599fcbb305ee6c233ff7331.png)

Figure 2: Depiction of the decision boundary shift between traditional phishing emails and LLM-rephrased phishing emails in terms of classification probability.

In contrast, the rephrased phishing emails, generated using LLMs, employed more legitimate-sounding language, such as “credentials,” “membership,” “confirmation,” and “account update.” Although these terms are more commonly associated with legitimate emails, words like “login” still appear frequently, which can trigger phishing detectors under certain contexts.

This shift in language results in a narrowing of the decision boundary, making it more challenging for classifiers to distinguish between phishing and legitimate emails. Figure [2](https://arxiv.org/html/2411.13874v1#S4.F2 "Figure 2 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") visualizes this shift, showing how traditional phishing emails are associated with higher probabilities of being flagged as phishing due to the frequent use of suspicious terms. Rephrased emails, on the other hand, use more neutral language, making them harder to classify and shifting the decision boundary closer to legitimate emails.

To understand these shifts more quantitatively, we applied both Naive Bayes and Logistic Regression models to evaluate the sensitivity of emails to specific words. In the Naive Bayes model, the probability of an email being classified as phishing based on a particular word is calculated as:

|  | $P(\text{phishing}\mid w_{i})=\frac{P(w_{i}\mid\text{phishing})P(\text{phishing% })}{P(w_{i})}$ |  |

This approach allows us to quantify how often specific words, like “urgent” or “payment,” appear in phishing emails compared to legitimate ones. As a result, words frequently associated with phishing attempts significantly increase the probability that the email will be classified as phishing.

Similarly, the Logistic Regression model provides a probability score for each word based on its contribution to the overall classification of the email. The equation is:

|  | $P(\text{phishing}\mid w_{i})=\frac{1}{1+e^{-(\mathbf{w}^{T}\mathbf{x}_{i}+b)}}$ |  |

In this case, the weight vector $\mathbf{w}$ assigns importance to specific words, such as “credentials” or “login,” which may occur in both phishing and legitimate emails but with different contexts. The bias term $b$ adjusts for the overall tendency of the model to classify emails as phishing or legitimate.

These models directly contribute to the values shown on the x-axis in Figure [2](https://arxiv.org/html/2411.13874v1#S4.F2 "Figure 2 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant."), where the probabilities reflect how sensitive the classification is to the presence of certain words. Traditional phishing emails, with words far from the decision boundary, are easier to detect, while rephrased emails, which use more neutral terms, challenge the detectors by narrowing this boundary.

This decision boundary shift can also be observed in LLM-based phishing detectors. As shown in Figures [3](https://arxiv.org/html/2411.13874v1#S4.F3 "Figure 3 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") and [4](https://arxiv.org/html/2411.13874v1#S4.F4 "Figure 4 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant."), the Llama3 model classifies the first two emails as legitimate and the rest as phishing, with reasoning largely driven by the use of specific words. Phishing emails containing terms like “urgent” or “payment” were more easily detected, whereas rephrased emails using terms like “account update” introduced more ambiguity, reducing detection accuracy.

![Refer to caption](img/63d300dbf3c312ae1c54d458dd00d8b5.png)

Figure 3: Classification Results for 5 Original Emails by Llama 3

![Refer to caption](img/e160dae82ca37e668109492e05739fcf.png)

Figure 4: Classification Results for 5 Rephrased Emails by Llama 3 (Few-Shot Prompting)

TABLE I: Performance of Phishing Detectors on the Nazario Dataset

| Detector | TP | TN | FP | FN | Accuracy | Precision | Recall | F1 Score |
| Original Emails |
| Gmail Spam Filter | 573 | 589 | 27 | 11 | 96.83% | 95.50% | 98.12% | 96.79% |
| Proofpoint | 558 | 592 | 41 | 8 | 96.16% | 93.00% | 98.59% | 95.71% |
| SpamAssassin | 574 | 576 | 26 | 24 | 95.83% | 95.67% | 95.99% | 95.83% |
| Naive Bayes | 559 | 567 | 41 | 33 | 93.80% | 93.17% | 94.43% | 93.79% |
| SVM | 542 | 555 | 58 | 45 | 91.42% | 90.33% | 92.33% | 91.32% |
| Logistic Regression | 554 | 569 | 46 | 31 | 93.58% | 92.33% | 94.70% | 93.50% |
| Zero Shot Rephrased Emails |
| Gmail Spam Filter | 573 | 559 | 27 | 41 | 94.33% | 95.50% | 93.32% | 94.40% |
| Proofpoint | 558 | 554 | 42 | 46 | 92.67% | 93.00% | 92.38% | 92.69% |
| SpamAssassin | 574 | 545 | 26 | 55 | 93.25% | 95.67% | 91.26% | 93.41% |
| Naive Bayes | 559 | 518 | 41 | 82 | 89.75% | 93.17% | 87.21% | 90.09% |
| SVM | 542 | 533 | 58 | 67 | 89.58% | 90.33% | 89.00% | 89.66% |
| Logistic Regression | 554 | 515 | 46 | 85 | 89.08% | 92.33% | 86.70% | 89.43% |
| Few Shot Rephrased Emails |
| Gmail Spam Filter | 573 | 483 | 27 | 117 | 88.00% | 95.50% | 83.04% | 88.84% |
| Proofpoint | 558 | 505 | 42 | 95 | 88.58% | 93.00% | 85.45% | 89.07% |
| SpamAssassin | 574 | 465 | 26 | 135 | 86.50% | 95.67% | 80.96% | 87.70% |
| Naive Bayes | 559 | 418 | 41 | 182 | 81.42% | 93.17% | 75.44% | 83.37% |
| SVM | 542 | 421 | 58 | 179 | 80.25% | 90.33% | 75.17% | 82.06% |
| Logistic Regression | 554 | 460 | 46 | 140 | 84.50% | 92.33% | 79.83% | 85.63% |

TABLE II: Performance of LLMs on the Nigerian Fraud Dataset

| LLM | TP | TN | FP | FN | Accuracy | Precision | Recall | F1 Score |
| Original Emails |
| GPT-4 | 391 | 395 | 9 | 5 | 98.25% | 97.75% | 98.74% | 98.24% |
| GPT-3.5 | 386 | 384 | 14 | 16 | 96.25% | 96.50% | 96.02% | 96.26% |
| Claude 3 Sonnet | 385 | 392 | 15 | 8 | 97.12% | 96.25% | 97.96% | 97.10% |
| Llama3 | 389 | 397 | 11 | 3 | 98.25% | 97.25% | 99.23% | 98.23% |
| Gemini | 385 | 389 | 15 | 11 | 96.75% | 96.25% | 97.22% | 96.73% |
| Zero Shot Rephrased Emails |
| GPT-4 | 391 | 369 | 9 | 31 | 95.00% | 97.75% | 92.65% | 95.13% |
| GPT-3.5 | 386 | 347 | 14 | 53 | 91.62% | 96.50% | 87.93% | 92.01% |
| Claude 3 Sonnet | 385 | 361 | 15 | 39 | 93.25% | 96.25% | 90.80% | 93.45% |
| Llama3 | 389 | 370 | 11 | 30 | 94.88% | 97.25% | 92.84% | 94.99% |
| Gemini | 385 | 342 | 15 | 58 | 90.88% | 96.25% | 86.91% | 91.34% |
| Few Shot Rephrased Emails |
| GPT-4 | 391 | 353 | 9 | 47 | 93.00% | 97.75% | 89.27% | 93.32% |
| GPT-3.5 | 386 | 322 | 14 | 78 | 88.50% | 96.50% | 83.19% | 89.35% |
| Claude 3 Sonnet | 385 | 354 | 15 | 46 | 92.38% | 96.25% | 89.33% | 92.66% |
| Llama3 | 389 | 346 | 11 | 54 | 91.88% | 97.25% | 87.81% | 92.29% |
| Gemini | 385 | 276 | 15 | 124 | 82.62% | 96.25% | 75.64% | 84.71% |

As shown in Figure [3](https://arxiv.org/html/2411.13874v1#S4.F3 "Figure 3 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant."), the Llama 3 model correctly identifies the first two emails as legitimate and the last three as phishing. This demonstrates the model’s accuracy when working with original, unaltered email content.

Figure [4](https://arxiv.org/html/2411.13874v1#S4.F4 "Figure 4 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") shows the detection rates for rephrased emails using few-shot prompting. In this sample, two phishing emails were incorrectly classified as legitimate. The reasoning given by the model indicates a reliance on specific words and patterns in the emails, which can easily be bypassed by rephrasing.

### IV-A Nazario Dataset Experiments

The results from the Nazario dataset, summarized in Table [I](https://arxiv.org/html/2411.13874v1#S4.T1 "TABLE I ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant."), highlight the performance of three phishing detectors—Google Gmail, SpamAssassin, and Proofpoint—as well as three machine learning models—Naive Bayes, SVM, and Logistic Regression. A total of 1200 emails (600 phishing and 600 legitimate) were used in the experiments.

Proofpoint achieved the highest recall on original phishing emails but had a higher false positive rate due to its sensitivity. Gmail demonstrated better accuracy with fewer false positives, but recall remains the more important metric in this study. After rephrasing, performance dropped significantly for all models, particularly on zero-shot rephrased emails, where models like Naive Bayes and SVM dropped to recall rates as low as 87%. Few-shot rephrased emails further reduced detection rates, with even advanced detectors like Gmail and Proofpoint struggling against these more subtle phishing attempts.

Table [II](https://arxiv.org/html/2411.13874v1#S4.T2 "TABLE II ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") presents the performance of five LLMs—GPT-4, GPT-3.5, Claude 3 Sonnet, Llama3, and Gemini—on the same dataset. It includes true positive (TP), true negative (TN), false positive (FP), and false negative (FN) counts, along with accuracy, precision, recall, and F1 scores for original emails, zero-shot rephrased emails, and few-shot rephrased emails.

GPT-4 maintained the highest accuracy and recall across all types of emails. Llama3 showed notable improvement with few-shot rephrased emails, while Gemini struggled with rephrased emails, particularly in the zero-shot scenario, where its accuracy dropped significantly.

### IV-B Nigerian Fraud Dataset Experiments

The Nigerian Fraud dataset combines two previous datasets with similar content  [[28](https://arxiv.org/html/2411.13874v1#bib.bib28)]. The final dataset consists of 800 emails (400 phishing and 400 legitimate) and poses a distinct challenge with its longer emails (10–650 characters) that often involve financial scams and urgent language. This complexity is both a challenge and an advantage, depending on the phishing detector.

For original emails, Proofpoint outperformed other detectors with near-perfect recall (98.95%), but its performance dropped on rephrased emails. Zero-shot rephrasing reduced its recall to 93.16%, with a corresponding accuracy drop, reflecting the sophistication of rephrased phishing emails that bypass traditional spam heuristics. SpamAssassin and Gmail also saw similar drops, with Gmail’s recall decreasing to 88.76% in the zero-shot scenario.

TABLE III: Performance of Phishing Detectors on the Nigerian Fraud Dataset

| Detector | TP | TN | FP | FN | Accuracy | Precision | Recall | F1 Score |
| Original Emails |
| Gmail Spam Filter | 387 | 396 | 13 | 4 | 97.88% | 96.75% | 98.98% | 97.85% |
| SpamAssassin | 378 | 396 | 22 | 4 | 96.75% | 94.50% | 98.95% | 96.68% |
| Proofpoint | 368 | 399 | 32 | 1 | 95.88% | 92.00% | 99.73% | 95.71% |
| Naive Bayes | 375 | 388 | 25 | 12 | 95.38% | 93.75% | 96.90% | 95.30% |
| SVM | 379 | 390 | 21 | 10 | 96.12% | 94.75% | 97.43% | 96.07% |
| Logistic Regression | 381 | 393 | 19 | 7 | 96.75% | 95.25% | 98.20% | 96.70% |
| Zero Shot Rephrased Emails |
| Gmail Spam Filter | 387 | 351 | 13 | 49 | 92.25% | 96.75% | 88.76% | 92.58% |
| SpamAssassin | 378 | 357 | 22 | 43 | 91.88% | 94.50% | 89.79% | 92.08% |
| Proofpoint | 368 | 373 | 32 | 27 | 92.62% | 92.00% | 93.16% | 92.58% |
| Naive Bayes | 375 | 349 | 25 | 51 | 90.50% | 93.75% | 88.03% | 90.80% |
| SVM | 379 | 343 | 21 | 57 | 90.25% | 94.75% | 86.93% | 90.67% |
| Logistic Regression | 381 | 335 | 19 | 65 | 89.50% | 95.25% | 85.43% | 90.07% |
| Few Shot Rephrased Emails |
| Gmail Spam Filter | 387 | 349 | 13 | 51 | 92.00% | 96.75% | 88.36% | 92.36% |
| SpamAssassin | 378 | 340 | 22 | 60 | 89.75% | 94.50% | 86.30% | 90.21% |
| Proofpoint | 368 | 376 | 32 | 24 | 93.00% | 92.00% | 93.88% | 92.93% |
| Naive Bayes | 375 | 336 | 25 | 64 | 88.88% | 93.75% | 85.42% | 89.39% |
| SVM | 379 | 318 | 21 | 82 | 87.12% | 94.75% | 82.21% | 88.04% |
| Logistic Regression | 381 | 327 | 19 | 73 | 88.50% | 95.25% | 83.92% | 89.23% |

TABLE IV: Performance of LLMs on the Nigerian Fraud Dataset

| LLM | TP | TN | FP | FN | Accuracy | Precision | Recall | F1 Score |
| Original Emails |
| GPT-4 | 394 | 399 | 6 | 1 | 99.12% | 98.50% | 99.75% | 99.12% |
| GPT-3.5 | 379 | 384 | 21 | 16 | 95.38% | 94.75% | 95.95% | 95.35% |
| Claude 3 Sonnet | 394 | 390 | 6 | 10 | 98.00% | 98.50% | 97.52% | 98.01% |
| Llama3 | 392 | 396 | 8 | 4 | 98.50% | 98.00% | 98.99% | 98.49% |
| Gemini | 382 | 386 | 18 | 14 | 96.00% | 95.50% | 96.46% | 95.98% |
| Zero Shot Rephrased Emails |
| GPT-4 | 394 | 372 | 6 | 28 | 95.75% | 98.50% | 93.36% | 95.86% |
| GPT-3.5 | 379 | 354 | 21 | 46 | 91.62% | 94.75% | 89.18% | 91.88% |
| Claude 3 Sonnet | 394 | 366 | 6 | 34 | 95.00% | 98.50% | 92.06% | 95.17% |
| Llama3 | 392 | 381 | 8 | 19 | 96.62% | 98.00% | 95.38% | 96.67% |
| Gemini | 382 | 324 | 18 | 76 | 88.25% | 95.50% | 83.41% | 89.04% |
| Few Shot Rephrased Emails |
| GPT-4 | 394 | 365 | 6 | 35 | 94.88% | 98.50% | 91.84% | 95.05% |
| GPT-3.5 | 379 | 337 | 21 | 63 | 89.50% | 94.75% | 85.75% | 90.02% |
| Claude 3 Sonnet | 394 | 363 | 6 | 37 | 94.62% | 98.50% | 91.42% | 94.83% |
| Llama3 | 392 | 367 | 8 | 33 | 94.88% | 98.00% | 92.24% | 95.03% |
| Gemini | 382 | 311 | 18 | 89 | 86.62% | 95.50% | 81.10% | 87.72% |

Machine learning models such as Naive Bayes and SVM, which were trained on 1500 emails from the original datasets, showed a more notable decline in performance when faced with rephrased phishing emails. In particular, Naive Bayes’ accuracy dropped to as low as 88%, and its recall decreased to 85% when processing few-shot rephrased emails. This demonstrates the limitations of these models when handling sophisticated phishing emails that are able to bypass traditional keyword-based detection. The results indicate that, for a dataset like the Nigerian Fraud dataset, traditional phishing detectors may still maintain strong performance on original emails, but their effectiveness is significantly compromised when these emails are rephrased using advanced techniques.

When rephrased using few-shot prompting, the effectiveness of the phishing attempts improved slightly. The overall accuracy remained around the same range as zero-shot rephrased emails suggesting the few-shot rephrasing might not be very effective in certain contexts compared to simpler prompts. Table [III](https://arxiv.org/html/2411.13874v1#S4.T3 "TABLE III ‣ IV-B Nigerian Fraud Dataset Experiments ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") provides a detailed breakdown of the performance of all phishing detectors, while Table [IV](https://arxiv.org/html/2411.13874v1#S4.T4 "TABLE IV ‣ IV-B Nigerian Fraud Dataset Experiments ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") summarizes the performance of the LLMs tested on this dataset with the primary focus here on the False Negative values (FN) as they represent the number of harmful emails that evaded detection.

![Refer to caption](img/f47c566962c88cfb9ada41d1ae62c95a.png)

Figure 5: Accuracy Comparison of SVM, Naive Bayes, and Logistic Regression in Detecting Rephrased Emails: Traditional vs. LLM-Augmented Datasets.

### IV-C Using LLMs for Rephrasing and Data Augmentation

The integration of LLMs for rephrasing phishing emails presents a significant opportunity to augment phishing datasets, leading to the development of more robust phishing detectors. Specifically, the use of rephrased emails generated by LLMs such as GPT-4 and Llama3\. In this study, we introduced the LLM-Nazario dataset, which consists of 5,000 emails. Two versions of this dataset were constructed by rephrasing and augmenting datasets using both GPT-4 and Llama3 and will be shared for future research to support the fine-tuning and model training of phishing detection models.

Figure [5](https://arxiv.org/html/2411.13874v1#S4.F5 "Figure 5 ‣ IV-B Nigerian Fraud Dataset Experiments ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") illustrates the detection accuracy of the SVM, Naive Bayes, and Logistic Regression models, each trained separately on three datasets: the traditional phishing dataset, an LLM-augmented dataset generated by GPT-4, and an LLM-augmented dataset generated by Llama3\. These datasets consist of both traditional phishing emails and rephrased phishing emails designed by the respective LLMs to bypass standard phishing detectors. The models were then tested exclusively on a set of LLM-rephrased phishing emails, crafted using advanced prompt engineering to evade detection. The results demonstrate that models trained on the LLM-augmented datasets performed significantly better at detecting rephrased phishing emails compared to those trained on the traditional datasets. On average the Logistic Regression model improved its accuracy by 10.90% when trained on the LLM-augmented datasets, with Naive Bayes showing a 10.38% increase. SVM exhibited the highest improvement, reaching 94.40% accuracy, a 12.86% increase over its performance on the traditional dataset.

These statistical improvements indicate that LLM-augmented datasets provide more comprehensive training data, allowing phishing detectors to better adapt to subtle variations in rephrased emails. The introduction of these variations pushes detectors to refine their decision boundaries, ultimately making them more effective against both traditional and LLM-rephrased phishing attempts.

## V Discussion and Limitations

The results from both the Nazario and Nigerian Fraud datasets provide clear evidence of the challenges posed by LLM-rephrased phishing emails. Traditional phishing detectors, such as Google Gmail Spam Detector, SpamAssassin, and Proofpoint, perform exceptionally well on original phishing emails but struggle with rephrased emails, in both zero-shot and few-shot rephrased scenarios. The accuracy and recall metrics drop across the board when these detectors are faced with more subtle phishing attempts generated by advanced LLMs.

Our study underscores the need for more advanced detection mechanisms capable of handling these evolving threats. LLMs, with their ability to generate highly realistic phishing emails, represent a new frontier in phishing attacks. However, they also present an opportunity to improve the robustness of phishing detectors through data augmentation. By incorporating LLM-generated emails into the training process, we can expose phishing detectors to a wider range of linguistic variations, making them better equipped to handle sophisticated attacks. The proposed LLM-Nazario dataset is a step in this direction, providing a rich source of rephrased phishing emails that can be used to train and fine-tune phishing detectors. The use of LLM-augmented datasets demonstrated notable improvements in detection accuracy. This highlights the dual nature of LLMs in the phishing detection landscape: while they can be used to craft more sophisticated attacks, they also hold the potential to enhance the CTI against these attacks.

The primary limitation of this study is its exclusive focus on English-language datasets. Due to the current availability of large, high-quality datasets primarily in English, extending our research to other languages was challenging. Future work should address this gap by incorporating datasets from diverse languages to enhance the robustness of language models in email detection across different linguistic contexts. Another limitation is that we did not explore the effect of fine-tuning some of the small language models on the LLM-augmented datasets, which could provide valuable insights. Future research could focus on this aspect to determine how fine-tuning influences model performance and adaptability, potentially leading to even better detection results.

## VI Conclusion

This paper presents a comprehensive evaluation of both traditional and LLM-based phishing detectors, focusing on the challenges posed by LLM-rephrased phishing emails. Our findings show that while traditional phishing detectors like Gmail Spam Detector, SpamAssassin, Proofpoint, and State-of-the-Art LLMs perform well on original phishing emails, their accuracy and recall decline notably when dealing with LLM-rephrased versions of the same emails.

However, by leveraging LLMs for data augmentation, we have demonstrated that phishing detectors can be made more robust. Training models on LLM-augmented datasets significantl improves their detection rates. This approach provides a valuable strategy for developing more resilient phishing detectors that can adapt to the evolving tactics of cyber attackers. Future efforts should focus on continually advancing phishing detection systems through innovative machine learning approaches and regularly updating training datasets to incorporate new phishing strategies.

## References

*   [1] Almomani, A., Gupta, B.B., Atawneh, S., Meulenberg, A., Almomani, E.: A Survey of Phishing Email Filtering Techniques. IEEE Communications Surveys & Tutorials, 15(4), pp. 2070–2090\. IEEE (2013)
*   [2] Jones, H.S., Towse, J.N., Race, N., Harrison, T.: Email fraud: The search for psychological predictors of susceptibility. PloS one, 14(1), pp. e0209684\. Public Library of Science (2019)
*   [3] McAlaney, J., Hills, P.J.: Understanding phishing email processing and perceived trustworthiness through eye tracking. Frontiers in Psychology, 11, pp. 1756\. Frontiers Media SA (2020)
*   [4] Wang, Y.: Mitigating phishing threats. PhD Thesis, The University of St Andrews (2023)
*   [5] Tihanyi, N., Ferrag, M.A., Jain, R., Bisztray, T., Debbah, M.: CyberMetric: A Benchmark Dataset based on Retrieval-Augmented Generation for Evaluating LLMs in Cybersecurity Knowledge. In: 2024 IEEE International Conference on Cyber Security and Resilience (CSR), pp. 296–302\. IEEE (2024)
*   [6] Sahoo, P., Singh, A.K., Saha, S., Jain, V., Mondal, S., Chadha, A.: A Systematic Survey of Prompt Engineering in Large Language Models: Techniques and Applications. arXiv preprint arXiv:2402.07927 (2024)
*   [7] A. Radford, J. Wu, R. Child, D. Luan, D. Amodei, and I. Sutskever, “Language models are unsupervised multitask learners,” OpenAI Blog, vol. 1, no. 8, p. 9, 2019.
*   [8] Fette, I., Sadeh, N., Tomasic, A.: Learning to detect phishing emails. In: Proceedings of the 16th International Conference on World Wide Web, pp. 649–656\. ACM, Banff (2007)
*   [9] Ma, L., Ofoghi, B., Watters, P., Brown, S.: Detecting phishing emails using hybrid features. In: 2009 Symposia and Workshops on Ubiquitous, Autonomic and Trusted Computing, pp. 493–497\. IEEE (2009)
*   [10] Verma, R., Shashidhar, N., Hossain, N.: Detecting phishing emails the natural language way. In: Computer Security–ESORICS 2012: 17th European Symposium on Research in Computer Security, pp. 824–841\. Springer, Pisa (2012)
*   [11] Sheng, S., Wardman, B., Warner, G., Cranor, L., Hong, J., Zhang, C.: An Empirical Analysis of Phishing Blacklists. Carnegie Mellon University (2009)
*   [12] Alhogail, A., Alsabih, A.: Applying Machine Learning and Natural Language Processing to Detect Phishing Email. Computers & Security, 110, 102414\. Elsevier (2021)
*   [13] Wu, F., Zhang, N., Jha, S., McDaniel, P., Xiao, C.: A New Era in LLM Security: Exploring Security Concerns in Real-World LLM-based Systems. arXiv preprint arXiv:2402.18649 (2024)
*   [14] Yao, Y., Duan, J., Xu, K., Cai, Y., Sun, Z., Zhang, Y.: A survey on large language model (LLM) security and privacy: The good, the bad, and the ugly. High-Confidence Computing, pp. 100211\. Elsevier (2024)
*   [15] Hazell, J.: Spear phishing with large language models. arXiv 2305 (2023)
*   [16] Touvron, H., Lavril, T., Izacard, G., Martinet, X., Lachaux, M.-A., Lacroix, T., Rozière, B., Goyal, N., Hambro, E., Azhar, F., et al.: Llama: Open and efficient foundation language models. arXiv preprint arXiv:2302.13971 (2023)
*   [17] Team, Gemini, Anil, R., Borgeaud, S., Alayrac, J.-B., Yu, J., Soricut, R., Schalkwyk, J., Dai, A. M., Hauth, A., Millican, K., et al.: Gemini: A family of highly capable multimodal models. arXiv preprint arXiv:2312.11805 (2023)
*   [18] Anthropic: Claude. [Online]. Available: https://www.anthropic.com/news/introducing-claude
*   [19] Achiam, J., Adler, S., Agarwal, S., Ahmad, L., Akkaya, I., Aleman, F. L., Almeida, D., Altenschmidt, J., Altman, S., Anadkat, S., et al.: GPT-4 Technical Report. arXiv preprint arXiv:2303.08774 (2023)
*   [20] Heiding, F., Schneier, B., Vishwanath, A., Bernstein, J., Park, P.S.: Devising and detecting phishing emails using large language models. IEEE Access (2024)
*   [21] Koide, T., Fukushi, N., Nakano, H., Chiba, D.: ChatSpamDetector: Leveraging Large Language Models for Effective Phishing Email Detection. arXiv preprint arXiv:2402.18093 (2024) V. S. Tida and S. Hsu, “Universal spam detection using transfer learning of BERT model,” arXiv preprint arXiv:2202.03480, 2022.
*   [22] “SpamAssassin,” Apache, [Online]. Available: https://spamassassin.apache.org/. [Accessed: 10-Sep-2024].
*   [23] J. C. Brickley, K. Thakur, and A. S. Kamruzzaman, “A comparative analysis between technical and non-technical phishing defenses,” International Journal of Cyber-Security and Digital Forensics, vol. 10, no. 1, pp. 28–41, 2021.
*   [24] M. Labonne and S. Moran, “Spam-t5: Benchmarking large language models for few-shot email spam detection,” arXiv preprint arXiv:2304.01238, 2023.
*   [25] Chataut, R., Gyawali, P.K., Usman, Y.: Can AI Keep You Safe? A Study of Large Language Models for Phishing Detection. In: 2024 IEEE 14th Annual Computing and Communication Workshop and Conference (CCWC), pp. 0548–0554\. IEEE (2024)
*   [26] Kang, D., Li, X., Stoica, I., Guestrin, C., Zaharia, M., Hashimoto, T.: Exploiting Programmatic Behavior of LLMs: Dual-Use Through Standard Security Attacks. arXiv preprint arXiv:2302.05733 (2023)
*   [27] Roy, S.S., Thota, P., Naragam, K.V., Nilizadeh, S.: From Chatbots to Phishbots?: Phishing Scam Generation in Commercial Large Language Models. 2024 IEEE Symposium on Security and Privacy (SP), pp. 221–221, IEEE Computer Society (2024)
*   [28] Champa, A.I., Rabbi, M.F., Zibran, M.F.: Curated Datasets and Feature Analysis for Phishing Email Detection with Machine Learning. In: 3rd IEEE International Conference on Computing and Machine Intelligence (ICMI), pp. 1–7 (to appear) (2024)
*   [29] Qader, W.A., Ameen, M.M., Ahmed, B.I.: An overview of bag of words; importance, implementation, applications, and challenges. In: 2019 International Engineering Conference (IEC), pp. 200–204\. IEEE (2019)
*   [30] Ramos, J.: Using tf-idf to determine word relevance in document queries. In: Proceedings of the First Instructional Conference on Machine Learning, vol. 242(1), pp. 29–48\. Citeseer (2003)
*   [31] Church, K.W.: Word2Vec. Natural Language Engineering, 23(1), pp. 155–162\. Cambridge University Press (2017)
*   [32] Yong, G., Jeon, K., Gil, D., Lee, G.: Prompt engineering for zero-shot and few-shot defect detection and classification using a visual-language pretrained model. Computer-Aided Civil and Infrastructure Engineering 38(11), 1536–1554 (2023)
*   [33] Sanh, V., Webson, A., Raffel, C., Bach, S.H., Sutawika, L., Alyafeai, Z., Chaffin, A., Stiegler, A., Scao, T.L., Raja, A., et al.: Multitask Prompted Training Enables Zero-Shot Task Generalization. arXiv preprint arXiv:2110.08207 (2021)
*   [34] Kojima, T., Gu, S.S., Reid, M., Matsuo, Y., Iwasawa, Y.: Large Language Models Are Zero-Shot Reasoners. Advances in Neural Information Processing Systems, 35, pp. 22199–22213 (2022)
*   [35] Kojima, T., Gu, S.S., Reid, M., Matsuo, Y., Iwasawa, Y.: Large language models are zero-shot reasoners. Advances in Neural Information Processing Systems 35, 22199–22213 (2022)
*   [36] Ye, X., Durrett, G.: The unreliability of explanations in few-shot prompting for textual reasoning. Advances in Neural Information Processing Systems 35, 30378–30392 (2022)
*   [37] Reynolds, L., McDonell, K.: Prompt programming for large language models: Beyond the few-shot paradigm. In: Extended Abstracts of the 2021 CHI Conference on Human Factors in Computing Systems, pp. 1–7 (2021)
*   [38] Chen, Y., Qian, S., Tang, H., Lai, X., Liu, Z., Han, S., Jia, J.: LongLoRA: Efficient Fine-Tuning of Long-Context Large Language Models. arXiv preprint arXiv:2309.12307 (2023)