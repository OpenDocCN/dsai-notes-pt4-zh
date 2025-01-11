<!--yml
category: 未分类
date: 2025-01-11 12:53:34
-->

# Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients

> 来源：[https://arxiv.org/html/2402.10153/](https://arxiv.org/html/2402.10153/)

Mahyar Abbasian${}^{1}$, Zhongqi Yang${}^{1}$, Elahe Khatibi${}^{1}$, Pengfei Zhang${}^{1}$, Nitish Nagesh${}^{1}$,
Iman Azimi${}^{1}$, Ramesh Jain${}^{1}$, and Amir M. Rahmani${}^{1,2}$
${}^{1}$Department of Computer Science, University of California, Irvine
${}^{2}$School of Nursing, University of California, Irvine {abbasiam, zhongqy4, ekhatibi, pengfz5, nnagesh1, azimii, rcjain, a.rahmani}@uci.edu

###### Abstract

Effective diabetes management is crucial for maintaining health in diabetic patients. Large Language Models (LLMs) have opened new avenues for diabetes management, facilitating their efficacy. However, current LLM-based approaches are limited by their dependence on general sources and lack of integration with domain-specific knowledge, leading to inaccurate responses. In this paper, we propose a knowledge-infused LLM-powered conversational health agent (CHA) for diabetic patients. We customize and leverage the open-source openCHA framework, enhancing our CHA with external knowledge and analytical capabilities. This integration involves two key components: 1) incorporating the American Diabetes Association dietary guidelines and the Nutritionix information and 2) deploying analytical tools that enable nutritional intake calculation and comparison with the guidelines. We compare the proposed CHA with GPT4\. Our evaluation includes 100 diabetes-related questions on daily meal choices and assessing the potential risks associated with the suggested diet. Our findings show that the proposed agent demonstrates superior performance in generating responses to manage essential nutrients.

###### Index Terms:

LLMs, Knowledge Graph, Diabetes, Nutrition Therapy, Health Agents.

## I Introduction

Effective diabetes management plays a pivotal role in maintaining optimal health in individuals diagnosed with diabetes. Given the widespread prevalence of diabetes and its significant impact on both global healthcare infrastructures and personal health outcomes, the availability and utilization of robust management strategies are essential [[1](https://arxiv.org/html/2402.10153v2#bib.bib1), [2](https://arxiv.org/html/2402.10153v2#bib.bib2), [3](https://arxiv.org/html/2402.10153v2#bib.bib3)]. Such services require holistic approaches integrating a healthy lifestyle, nutritious diet, and physical activity assessment [[4](https://arxiv.org/html/2402.10153v2#bib.bib4)]. Particularly, a critical aspect is dietary regulation, which plays a direct role in controlling blood glucose levels and influencing the progression of the disease [[5](https://arxiv.org/html/2402.10153v2#bib.bib5), [6](https://arxiv.org/html/2402.10153v2#bib.bib6)]. The recent surge in advanced technologies, particularly in Large Language Models (LLMs), has substantially enhanced the accessibility and efficacy of diabetes management as transformative educational and assistive tools, enabling patients to acquire comprehensive knowledge and bridge the gap in diabetes self-management [[7](https://arxiv.org/html/2402.10153v2#bib.bib7), [8](https://arxiv.org/html/2402.10153v2#bib.bib8), [9](https://arxiv.org/html/2402.10153v2#bib.bib9)].

Recent studies have investigated the implementation of LLMs on diabetes management [[7](https://arxiv.org/html/2402.10153v2#bib.bib7), [8](https://arxiv.org/html/2402.10153v2#bib.bib8), [9](https://arxiv.org/html/2402.10153v2#bib.bib9)]. For example, [[7](https://arxiv.org/html/2402.10153v2#bib.bib7)] evaluated the use of ChatGPT for Diabetes Self-Management and Education. They leveraged ChatGPT to answer a range of diabetes self-management questions, covering four areas: diet/exercise, glucose level education, insulin storage, and administration. ChatGPT responded to the inquiries, showcasing a structured response style and offering guidance that is understandable to laymen. However, in some specific contexts, such as devising dietary plans, it still requires extra prompts to comprehensively produce guidelines for tasks like insulin administration. Additionally, Yang et al. [[8](https://arxiv.org/html/2402.10153v2#bib.bib8)] introduced ChatGLM to provide diabetes treatment strategies. The proposed model was fine-tuned through P-tuning [[10](https://arxiv.org/html/2402.10153v2#bib.bib10)] and LoRA [[11](https://arxiv.org/html/2402.10153v2#bib.bib11)] techniques with Electronic Health Record (EHR) of patients with diabetes. ChatGLM showed proficiency in generating treatment recommendations for most cases. However, its dependency on the specific data used for fine-tuning could result in misleading outputs, potentially harmful in the absence of certain essential external diabetes knowledge.

The lack of integration with verified domain-specific knowledge underscores a significant gap in existing approaches, compromising the accuracy and reliability of their outputs. Unlike systems anchored in specific knowledge bases, existing LLM-based approaches fall into limitations in their reliance on general information sources rather than specialized for diabetes. For instance, ChatGPT’s foundation on a general database – rather than medically specialized and validated ones – might account for its shortcomings in grasping medical nuances. Furthermore, due to the model’s inability to validate the reliability of its information, they may cause “hallucination,” generating inaccurate responses presented persuasively and fluently. This problem could easily mislead individuals lacking prior knowledge on the topic [[7](https://arxiv.org/html/2402.10153v2#bib.bib7)].

![Refer to caption](img/91729d3aab462301e789ff780613f06c.png)

Figure 1: An overview of openCHA [[12](https://arxiv.org/html/2402.10153v2#bib.bib12)] framework.

We believe that the integration of external knowledge bases with LLMs presents a robust solution to current challenges by enhancing the models’ capacity to access reliable information. To achieve this, Conversational Health Agents (CHAs) can play an important role [[13](https://arxiv.org/html/2402.10153v2#bib.bib13)]. CHAs are conversational systems that provide healthcare services, such as assistance and diagnosis, leveraging agents as the decision-making core. openCHA [[12](https://arxiv.org/html/2402.10153v2#bib.bib12)], an open-source CHA framework, provides a flexible framework for implementing LLM-based diabetes management through its adept integration of diverse external sources. openCHA possesses the capability to integrate and infuse knowledge into chatbots, while also integrating external analytical methods for data analysis. Figure [1](https://arxiv.org/html/2402.10153v2#S1.F1 "Figure 1 ‣ I Introduction ‣ Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients") shows an overview of this integration process, wherein an orchestrator – powered by an LLM-based planner and an LLM-based response generator – engages with various sources to gather the required information. This collected information is then used to craft a response to the user’s query.

This paper presents a knowledge-based diabetes management system that is implemented through the utilization of LLM-powered CHAs. We customize the open-source openCHA framework for diet assessment by integrating external knowledge – including the diet guidelines from the American Diabetes Association reports and the nutritional data from the Nutritionix knowledge base – into LLMs. We also incorporate an analysis tool to precisely calculate the total daily nutritional intake and compare it against the guidelines. We evaluate the proposed approach with real-world diabetes-related questions focusing on daily meals in comparison to GPT4\. The evaluation is centered on the assessment of the potential risk associated with the recommended dietary options.

## II Method

We develop an LLM-based CHA, designed to assess the risk associated with daily food intake according to recommended nutritional thresholds. We leverage openCHA [[12](https://arxiv.org/html/2402.10153v2#bib.bib12)], as a foundational framework, for our development. The proposed CHA aims to interact with users with diabetes, get their daily food intake information in the form of a conversation, and use a food knowledge base as well as guidelines to answer users in a more reliable way.

Our proposed CHA contains three main parts as Interface, Orchestrator, and External Sources (see Figure [2](https://arxiv.org/html/2402.10153v2#S2.F2 "Figure 2 ‣ II Method ‣ Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients")). The Interface serves as a connection point between users and our framework, facilitating interactions through text using a web chat interface and relaying user queries to the Orchestrator.

![Refer to caption](img/836aedae6096370c602d6ea456989a6a.png)

Figure 2: LLM-based CHA for diabetes management enabled by the openCHA framework.

The Orchestrator serves as the core component of the CHA and is responsible for problem-solving, planning, executing actions, and generating appropriate responses based on user queries. It operates on the principles of the Perceptual Cycle Model [[14](https://arxiv.org/html/2402.10153v2#bib.bib14)] and involves perceiving, transforming, and analyzing the world (i.e., input query and metadata). It interacts with external sources, acquires information, performs data integration, and extracts insights. The Orchestrator comprises four major components: 1) Task Planner: to perform decision-making and planning, 2) Task Executor: to enable actuation and data conversion, 3) Data Pipe: to act as a repository for acquired metadata and intermediate data, 4) and Response Generator: to refine information and delivers conclusive responses. For the Orchestrator setup, we use the OpenAI’s [[15](https://arxiv.org/html/2402.10153v2#bib.bib15)] GPT-3.5-turbo model as our base LLM, and the Tree of Thought [[16](https://arxiv.org/html/2402.10153v2#bib.bib16)] prompting technique for Planning.

External Sources are crucial for acquiring valuable information from diverse origins. Through the openCHA framework [[12](https://arxiv.org/html/2402.10153v2#bib.bib12)], we integrate two primary external sources to the CHA as: 1) Knowledge Base and 2) AI and Analysis Models (see Figure [2](https://arxiv.org/html/2402.10153v2#S2.F2 "Figure 2 ‣ II Method ‣ Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients")).

Knowledge Base: serves to fetch current healthcare information from credible sources. We developed a task focused on retrieving nutritional information from Nutritionix for various foods, enhancing the up-to-dateness of the responses. This task takes a food query as input and uses the NutritioniX API [[17](https://arxiv.org/html/2402.10153v2#bib.bib17)] to retrieve a list of foods along with their corresponding nutrient information, based on standard measurements. This is achieved by leveraging the API’s Natural Language for Nutrients feature, which allows for parsing of natural language requests and returning detailed nutrition information for common foods.

AI and Analysis Models: provide data analysis, enabling the extraction of insights and associations. This is particularly valuable in intricate healthcare contexts where LLMs may not efficiently handle intensive computations. We develop a task focused on daily food risk assessment into the openCHA framework [[12](https://arxiv.org/html/2402.10153v2#bib.bib12)]. This task follows established guidelines for daily nutrient intake to perform food risk assessments. The task also leverages the knowledge extracted from the NutritioniX using Knowledge Base developed task. This algorithm evaluates potential risks associated with food intake by comparing extracted nutritional data to recommended thresholds for diabetics, accounting for factors like carbohydrate content, sugar levels, protein amount, and dietary fiber.

![Refer to caption](img/1c13c79d04a70f9a84624c2337944edd.png)

Figure 3: A sample question and responses from the proposed CHA and GPT4\. The green text indicates matching the identified risk with the ground truth, the red text indicates mismatching.

The threshold levels for the risk assessment algorithm are selected based on commonly recognized and accepted standards by medical associations for nutrient intake. According to an American Diabetes Association report [[18](https://arxiv.org/html/2402.10153v2#bib.bib18)], individuals suffering from Type 2 Diabetes should eat 20 to 35 grams (g) of fiber from raw vegetables and unprocessed grains per day and limit sodium intake to 2,300 milligrams (mg) per day, both recommendations aligning with those for the general population. Although there are no overarching recommended ranges for carbohydrate intake, based on [[19](https://arxiv.org/html/2402.10153v2#bib.bib19)], patients with Type 1 and type 2 diabetes in the US consumed about 45% of their total energy intake through carbohydrates. While there is no concrete evidence about a certain range of protein intake aiding in glycemic control, we set protein intake to be 15-20% of total calorie intake based on ADA standards of care [[18](https://arxiv.org/html/2402.10153v2#bib.bib18)]. The recommended intake for fat is 20-35% of total calories according to [[20](https://arxiv.org/html/2402.10153v2#bib.bib20), [21](https://arxiv.org/html/2402.10153v2#bib.bib21)] while saturated fat is best restricted to less than 10% of total calories [[22](https://arxiv.org/html/2402.10153v2#bib.bib22)]. Based on the American Heart Association (AHA) guidelines [[23](https://arxiv.org/html/2402.10153v2#bib.bib23)], we set stringent restrictions on sugar intake due to its negative impact on diabetes with a maximum of 6 teaspoons or 25g of sugar.

## III Results

TABLE I: Sample evaluation for one question.

 |  | Carbohydrate | Fat | Saturated Fat | Protein | Sodium | Sugars | Dietary Fiber |
| Grount Truth | R | R | NR | R | NR | R | NR |
| Proposed CHA | R | R | NR | R | NR | R | NR |
| GPT4 | R | NR | NR | NR | NR | R | NR |
| R=Risky, NR=Not Risky | 

In this section, we assess the effectiveness of our proposed CHA in evaluating food-related risks for individuals with diabetes. Our objective is to determine the CHA’s ability to gauge the risk associated with daily food intake when the nutritional content deviates from recommended ranges. We compare the performance of our CHA, equipped with comprehensive food knowledge and analytical capabilities, with the OpenAI’s GPT4 model [[15](https://arxiv.org/html/2402.10153v2#bib.bib15)].

We collected 100 sample questions related to individuals inquiring about their daily meal intake. For the ground truth, we manually extract the nutritional data for each question and compute the overall nutritional values for parameters, such as fat, saturated fat, protein, carbohydrates, sodium, sugars, and dietary fiber. Subsequently, we evaluate the alignment of the nutrients with established guidelines. If any of these nutrients fall outside the recommended range, we categorize them as ”Risky,” while those within the range are labeled as ”Not Risky.” Ultimately, we generate a table of 100 rows and 7 columns (as our ground truth), representing Risky or Not Risky for each nutrient.

These questions are posed to both our CHA and GPT4 [[15](https://arxiv.org/html/2402.10153v2#bib.bib15)] in order to compare their responses with the ground truth. Figure [3](https://arxiv.org/html/2402.10153v2#S2.F3 "Figure 3 ‣ II Method ‣ Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients") provides an illustrative instance of a question and responses from the proposed CHA and GPT4. The green text highlights the model’s accurate identification of risk for specific nutrients based on the ground truth, while the red text indicates instances where the chatbot incorrectly assesses the risk for the corresponding nutrient. Table [I](https://arxiv.org/html/2402.10153v2#S3.T1 "TABLE I ‣ III Results ‣ Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients") shows the ground truth values and summarizes the assessments of the two chatbots for each nutrient.

Similarly, we replicate this procedure for each set of 100 questions and then compare the quantity of correctly identified risks against the ground truth. For GPT4, we formulate a customized prompt to instruct GPT4 to estimate the nutritional information of the mentioned foods and assess their alignment with recommended guidelines and rules.

TABLE II: The risk assessment accuracy of the 100 sample questions.

 |  | Carbohydrate | Fat | Saturated Fat | Protein | Sodium | Sugars | Dietary Fiber |
| --- | --- | --- | --- | --- | --- | --- | --- |
| GPT4 | 49% | 52% | 68% | 17% | 73% | 58% | 46% |
| Proposed CHA | 84% | 94% | 99% | 90% | 95% | 91% | 92% | 

Table [II](https://arxiv.org/html/2402.10153v2#S3.T2 "TABLE II ‣ III Results ‣ Knowledge-Infused LLM-Powered Conversational Health Agent: A Case Study for Diabetes Patients") indicates the responses to the 100 questions of both GPT4 and the proposed CHA. Our CHA outperforms GPT4 across all seven nutrients. This shows the significance of incorporating guidelines, knowledge bases (e.g., NutritioniX), and analytical tools into LLMs for health management tasks. It is worth noting that, in certain instances, the nutrient calculations for the proposed CHA bordered closely to the established guidelines, resulting in an increased error rate in some cases. For instance, when manually calculated, Carbohydrate content was found to be 44%, which falls within the recommended range of less than 45%, whereas our CHA calculated it to be 45%. This might be due to minor variations in portion sizes or rounding of fractional numbers.

## IV Discussion

The proposed CHA offers a significant degree of flexibility for the integration of LLMs with external health data sources, knowledge bases, and analytical tools. This flexibility provides opportunities for mitigating hallucination problems while enhancing the personalization and reliability of the CHA. For instance, it allows for the incorporation of diverse elements such as food-related knowledge graphs, personal health biomarkers, individual demographics, and food preferences into the existing CHA framework, thereby enabling more refined and tailored recommendations.

Additionally, this development emphasizes a strong focus on explainability. It enables users to inquire about the sequence of tasks and actions involved in generating a response. For example, when users seek information about specific food nutrients, data sources, or risk calculation methods, our CHA can provide insights into the underlying tasks and display results (in contrast to the state-of-the-art chatbots). This heightened level of transparency enhances the trustworthiness of the CHAs, fostering user confidence in the responses.

## V Conclusion

In this paper, we proposed an LLM-based CHA for diabetes management enabled by knowledge-infused LLMs. To achieve this, We leveraged openCHA for our development. We developed a nutrition information retrieval task to integrate the Nutritionix knowledge base into the CHA. Moreover, we developed a food risk assessment tool based on the American Diabetes Association dietary guidelines. We evaluated the effectiveness of our proposed CHA, in comparison to GPT4, in food-related risks for individuals with diabetes. Our findings showed that the proposed CHA outperforms the general-purpose GPT4 in responding to 100 diabetes-related queries regarding daily meal choices. This advancement highlights the potential of LLM-powered conversational agents in improving the accessibility and effectiveness of diabetes management, addressing a crucial aspect of healthcare for diabetic patients.

## References

*   [1] J. U. Poulsen et al., “A diabetes management system empowering patients to reach optimised glucose control: from monitor to advisor,” in 2010 Annual International Conference of the IEEE Engineering in Medicine and Biology, pp. 5270–5271, IEEE, 2010.
*   [2] L. Mertz, “Automated insulin delivery: taking the guesswork out of diabetes management,” IEEE pulse, vol. 9, no. 1, pp. 8–9, 2018.
*   [3] H. A. Klein et al., “Self management of medication and diabetes: Cognitive control,” IEEE Transactions on Systems, Man, and Cybernetics-Part A: Systems and Humans, vol. 34, no. 6, pp. 718–725, 2004.
*   [4] G. Fico et al., “Integration of personalized healthcare pathways in an ict platform for diabetes managements: a small-scale exploratory study,” IEEE Journal of Biomedical and Health Informatics, vol. 20, no. 1, pp. 29–38, 2014.
*   [5] I. Shalahuddin et al., “Blood sugar levels regulation in diabetes mellitus type 2 patients through diet management,” Jurnal Aisyah : Jurnal Ilmu Kesehatan, 2022.
*   [6] W. Russell et al., “Nutritional management of blood glucose levels,” Annals of Nutrition and Metabolism, vol. 63, pp. 1339–1340, 2013.
*   [7] G. G. R. Sng et al., “Potential and pitfalls of chatgpt and natural-language artificial intelligence models for diabetes education,” Diabetes Care, vol. 46, no. 5, pp. e103–e105, 2023.
*   [8] H. Yang et al., “Exploring the potential of large language models in personalized diabetes treatment strategies,” medRxiv, pp. 2023–06, 2023.
*   [9] H. Sun et al., “An ai dietitian for type 2 diabetes mellitus management based on large language and image recognition models: Preclinical concept validation study,” Journal of Medical Internet Research, vol. 25, p. e51300, 2023.
*   [10] X. Liu et al., “P-tuning: Prompt tuning can be comparable to fine-tuning across scales and tasks,” in Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (Volume 2: Short Papers), pp. 61–68, 2022.
*   [11] E. J. Hu et al., “Lora: Low-rank adaptation of large language models,” arXiv preprint arXiv:2106.09685, 2021.
*   [12] M. Abbasian et al., “Conversational health agents: A personalized llm-powered agent framework,” arXiv preprint arXiv:2310.02374, 2023.
*   [13] Y. Li et al., “Personal llm agents: Insights and survey about the capability, efficiency and security,” arXiv preprint arXiv:2401.05459, 2024.
*   [14] U. Neisser, “Perceiving, anticipating, and imagining,” in Perception and Cognition: Issues in the Foundations of Psychology, University of Minnesota Press, Minneapolis, 1978.
*   [15] OpenAI, “Chatgpt: Openai’s conversational ai model.” [https://openai.com/chatgpt](https://openai.com/chatgpt), Accessed: January 2024.
*   [16] S. Yao et al., “Tree of thoughts: Deliberate problem solving with large language models,” arXiv preprint arXiv:2305.10601, 2023.
*   [17] Syndigo Company, “Nutritionx: The world’s largest verified nutrition database.” [https://www.nutritionix.com/](https://www.nutritionix.com/).
*   [18] “5\. Lifestyle Management: Standards of Medical Care in Diabetes-2019,” Diabetes care, vol. 42, pp. S46–S60, 1 2019.
*   [19] A. Gray and R. J. Threlkeld, “Nutritional Recommendations for Individuals with Diabetes,” Diabetologia, vol. 54, 10 2019.
*   [20] B. E. Millen et al., “2013 American Heart Association/American College of Cardiology Guideline on Lifestyle Management to Reduce Cardiovascular Risk: practice opportunities for registered dietitian nutritionists,” Journal of the Academy of Nutrition and Dietetics, vol. 114, pp. 1723–1729, 11 2014.
*   [21] J. K. Snell-Bergeon et al., “Adults with type 1 diabetes eat a high fat, atherogenic diet which is associated with coronary artery calcium,” Diabetologia, vol. 52, p. 801, 5 2009.
*   [22] L. Van Horn et al., “The evidence for dietary prevention and treatment of cardiovascular disease,” Journal of the American Dietetic Association, vol. 108, pp. 287–331, 2 2008.
*   [23] R. K. Johnson et al., “American heart association nutrition committee of the council on nutrition, physical activity, and metabolism and the council on epidemiology and prevention. dietary sugars intake and cardiovascular health: a scientific statement from the american heart association,” 2009.