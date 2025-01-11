<!--yml
category: 未分类
date: 2025-01-11 12:13:30
-->

# Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality

> 来源：[https://arxiv.org/html/2409.15623/](https://arxiv.org/html/2409.15623/)

\onlineid

2064 \vgtccategoryResearch \vgtcinsertpkg \teaser ![[Uncaptioned image]](img/de91e1cfc68b3a82334039d229660d89.png) 1(a) shows an interaction between the LLM-based agent Safe Guard and a single player in VRChat in conversational mode. 1(b) shows an instance where hate speech is detected in conversational mode. 2(a) shows an interaction between LLM-based agent Safe Guard and multiple players in VRChat in observational mode. 2(b) shows hate speech being detected in observational mode.

Yiwen Xu e-mail: xu.yiwen1@northeastern.edu    Qinyang Hou e-mail: hou.q@northeastern.edu    Hongyu Wan e-mail: wan.hongy@northeastern.edu    Mirjana Prpa m.prpa@northeastern.edu Northeastern University, Khoury College of CS Vancouver, Canada

###### Abstract

In this paper, we present Safe Guard, an LLM-agent for the detection of hate speech in voice-based interactions in social VR (VRChat). Our system leverages Open AI GPT and audio feature extraction for real-time voice interactions. We contribute a system design and evaluation of the system that demonstrates the capability of our approach in detecting hate speech, and reducing false positives compared to currently available approaches. Our results indicate the potential of LLM-based agents in creating safer virtual environments and set the groundwork for further advancements in LLM-driven moderation approaches.

###### keywords:

hate speech, online harassment, real-time detection, social VR, ChatGPT, audio features.

Introduction

Social Virtual Reality (VR) platforms such as VRChat, Rec Room, Bigscreen, AltspaceVR, and Meta Horizon Worlds [[47](https://arxiv.org/html/2409.15623v1#bib.bib47)] have gone through a substantial rise in popularity in recent years. These platforms provide integrated features such as customizable avatars and real-time voice chat, offering users immersive first-person online social experience where they can communicate, interact, and engage in a more immersive and embodied way using head-mounted displays (HMDs) compared to traditional online platforms [[51](https://arxiv.org/html/2409.15623v1#bib.bib51)]. In social VR, real-time voice interactions are fundamental. Users engage in conversations using their real voices and communicate with others in real time (RT) within the virtual environment. This form of interaction enhances users’ feeling of presence and connection, making the experience resemble real-life face-to-face communication compared to text-based interactions [[47](https://arxiv.org/html/2409.15623v1#bib.bib47)].

However, as the increasing number of users engage in voice interactions within these virtual environments, new challenges and risks continue to arise in ensuring safe and trustworthy communication in social VR settings. Hate speech, as one of the main harassment forms on social VR platforms, poses significant threats to user well-being and the overall health of the VR communities. People who encounter online harassment often report significant disruptions to their offline lives [[7](https://arxiv.org/html/2409.15623v1#bib.bib7)], encompassing emotional and physical distress, shifts in their future technology usage, and raised concerns about safety and privacy [[18](https://arxiv.org/html/2409.15623v1#bib.bib18)]. Zheng et al. [[51](https://arxiv.org/html/2409.15623v1#bib.bib51)] discussed that online harassment within immersive environments like social VR settings could potentially cause more serious and negative effects on the target individuals’ well-being than traditional online attacks. Consequently, effective hate speech detection mechanisms are indispensable for social VR to mitigate the risks and maintain positive user experiences.

Combating hate speech in social VR faces several challenges. Social VR safety risks are often unpredictable, highly personal, and real-time, making documenting and data collecting difficult [[51](https://arxiv.org/html/2409.15623v1#bib.bib51)]. The immersive nature of social VR and the immediacy of voice interactions make it difficult for the platforms to detect and moderate toxic behaviors effectively [[51](https://arxiv.org/html/2409.15623v1#bib.bib51)]. To this date, the most accurate method for combating hate speech is human moderation conducted by the community members, yet with the rise of the number of events and attendees, moderators are presented with scalability challenges due to the limited number of moderators and the burden that moderation poses on community members [[47](https://arxiv.org/html/2409.15623v1#bib.bib47)].

To overcome the challenges, we draw on the work on AI-moderation of harassment in social VR by Schulenberg et al. [[47](https://arxiv.org/html/2409.15623v1#bib.bib47)] and a study conducted by Fiani et al. [[14](https://arxiv.org/html/2409.15623v1#bib.bib14)] which proposed the design direction for embodied AI moderators in social VR to mitigate harassment towards children. These studies highlighted the potential of AI agents in improving the moderation process by providing timely and context-aware interventions, which is crucial in dynamic and immersive settings like social VR. Building on the insights, our study focuses on the development of an LLM agent named ”Safe Guard” to detect real-time voice-based hate speech in social VR environments (see Figure Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality). Recent advancements in Large Language Models (LLMs) demonstrate under-explored potential for the use of LLMs in the context of real-time voice-based moderation in VR. To that end, we leveraged GPT-3.5 into our LLM-agent design for hate speech detection in VRChat. The proposed agent has two modes: (1) conversational mode with a single user, where it conducts conversation while simultaneously detecting and alerting for hate speech (2) observational mode when multiple users are present, where it monitors interactions to detect and alert for hate speech.

LLMs present promising solutions for enhancing moderation capabilities in VR settings, by exhibiting the capabilities to effectively identify and classify hate speech based on context and textual content. Kolla et al. [[28](https://arxiv.org/html/2409.15623v1#bib.bib28)] highlighted that LLMs perform well on various natural language tasks, including sentiment analysis and the detection of slurs or derogatory remarks, demonstrating their ability to identify explicit hate and offensive speech.

However, LLMs can be limited by their inability to process audio patterns and audio cues directly, which may result in challenges such as misidentification and false positives, especially when dealing with edge cases. Kumar et al. [[29](https://arxiv.org/html/2409.15623v1#bib.bib29)] discovered that GPT is likely to produce false positives in identifying hate speech due to triggering on poor language (e.g., profanity, slurs) and stereotypes (34%), even in instances of neutral or positive connotation. Given the consequences that false positives may have on online communities such as resulting in bans and suspended accounts, addressing the scalability of hate speech detection approaches in real time presents multi-faceted challenges. In addition, most previous studies focused on batch detection of harassment in text-based posts or comments [[7](https://arxiv.org/html/2409.15623v1#bib.bib7), [17](https://arxiv.org/html/2409.15623v1#bib.bib17), [24](https://arxiv.org/html/2409.15623v1#bib.bib24), [25](https://arxiv.org/html/2409.15623v1#bib.bib25)], a noticeable gap remains in exploring how to detect real-time verbal hate speech in VR setting that is efficient, accurate, and scalable.

To address these deficiencies and reduce false positives, in the study of Safe Guard, we aim to bridge the gap by deploying audio feature analysis to enhance LLMs’ ability to distinguish hateful content from benign content. The audio model can capture audio features such as tone, pitch, and emotion which LLMs might miss. This capability is valuable for detecting voice-based hate speech where the emotional context and speaker’s intent are crucial.

We envision the future role of Safe Guard as a first step in detecting and assisting human moderators in combating hate speech in real-time voice-based interactions. To that end, in this work we explore the first step that is concerned with the development of an LLM agent and answering the following research question (RQ): How to detect hate speech in social VR by leveraging audio features and LLM text analysis in LLM-agents in real time?

The main contributions of this work include: (1) an embodied LLM-based agent to detect hate speech in real-time voice interactions in social VR, (2) a high-accuracy, fast-speed prompting method for GPT 3.5 to detect real-time voice-based hate speech, (3) a CNN classifier for extraction and analysis of audio features to assist hate speech detection, (4) a system that integrates LLM detection and audio features analysis with the LLM agent system in VRChat for real-time voice-based hate speech detection, and (5) manually collected and annotated video datasets, then transferred into audio format, for validating voice-based hate speech detection.

In the following paper, we present Literature Review, Methodology, System Design, System Evaluation Results, Discussion of the Results, Limitations, Future Work, and Conclusion.

## 1 Literature Review

### 1.1 Defining Harassment and Hate Speech in the Context of Social VR

The concept of online harassment remains highly contextual and often involves personal interpretation [[12](https://arxiv.org/html/2409.15623v1#bib.bib12)]. Prior studies pointed at the lack of consistent consensus on the definition of harassment across different online social communities. Pater et al. [[41](https://arxiv.org/html/2409.15623v1#bib.bib41)] analyzed policy documents from fifteen social media platforms and pointed out that none of those platforms explicitly define what constitutes harassment. By cross-comparison of activities and behaviors that co-occur with harassment in these documents, the most common harassment types include abuse, bullying, harm, hate, stalking, and threats [[41](https://arxiv.org/html/2409.15623v1#bib.bib41)]. General online harassment was also categorized by using six distinct behaviors: offensive name-calling, purposeful embarrassment, stalking, physical threats, harassment over a sustained time, and sexual harassment [[12](https://arxiv.org/html/2409.15623v1#bib.bib12)]. Alternatively, Blackwell et al. [[7](https://arxiv.org/html/2409.15623v1#bib.bib7)] categorized toxic online harassment into four categories, namely flaming, doxing, impersonation, and public shaming. Flaming means hostile and insulting interactions between users. Doxing refers to the act of publicly revealing private information about an individual without their consent. Impersonation involves pretending to be someone else to deceive or harm, and public shaming is the act of humiliating someone in public [[7](https://arxiv.org/html/2409.15623v1#bib.bib7)].

In the context of social VR, Freeman et al. [[18](https://arxiv.org/html/2409.15623v1#bib.bib18)] examined which specific behaviors or interactions and in which context in social VR should be qualified as harassment, and concluded that any discriminatory conduct in social VR based on identity features such as gender and race (e.g., racism, misogyny, and homophobia) and with a specific target or victim are considered harassment. Moreover, Freeman et al. [[18](https://arxiv.org/html/2409.15623v1#bib.bib18)] suggested that embodiment and immersion in social VR can simulate real-life offline physical harassment, introducing new forms of online harassment that resemble offline behaviors. Besides movement and gesture-based harassment which is unique in the social VR context, most verbal harassment on social VR platforms share similarities to those found in other online social communities, including hate speech [[39](https://arxiv.org/html/2409.15623v1#bib.bib39)]. Verbal harassment in both contexts often involves using various types of detrimental user behaviors involving abusive communications directed towards other users [[51](https://arxiv.org/html/2409.15623v1#bib.bib51)].

Hate speech has been defined by United Nations [[38](https://arxiv.org/html/2409.15623v1#bib.bib38)] as “any kind of communication in speech, writing or behaviour, that attacks or uses pejorative or discriminatory language with reference to a person or a group on the basis of who they are, in other words, based on their religion, ethnicity, nationality, race, colour, descent, gender or other identity factor.” Guimarães, et al. [[20](https://arxiv.org/html/2409.15623v1#bib.bib20)] examined well-known datasets through Web and social network crawling and and labeled the messages as either hate speech or non-hate speech.By conducting experiments using four distinct datasets containing messages labeled as hate and non-hate speech, mainly from Twitter many studies such as [[50](https://arxiv.org/html/2409.15623v1#bib.bib50), [11](https://arxiv.org/html/2409.15623v1#bib.bib11), [16](https://arxiv.org/html/2409.15623v1#bib.bib16), [36](https://arxiv.org/html/2409.15623v1#bib.bib36), [35](https://arxiv.org/html/2409.15623v1#bib.bib35)] agreed on the hate speech definition as attacks on a person or a group based on race, religion, ethnic origin, sexual orientation, disability, or gender. Similarly, Arango, et al. [[3](https://arxiv.org/html/2409.15623v1#bib.bib3)] defined hate speech as communications of animosity or disparagement of an individual or a group on account of a group characteristic such as race, color, national origin, sex, disability, religion, or sexual orientation.

### 1.2 Moderation of Harassment in Social VR

Existing research on social VR moderation emphasize the complexity of managing harassment due to its embodiment nature, identity-related threats, presence of minors, and lack of consensus on definitions [[51](https://arxiv.org/html/2409.15623v1#bib.bib51), [15](https://arxiv.org/html/2409.15623v1#bib.bib15)]. The immersive and real-time nature of voice-based interactions in social VR makes it difficult for social VR platforms to accurately and effectively detect harassment due to the absence of a persistent, written record [[24](https://arxiv.org/html/2409.15623v1#bib.bib24)]. Bad actors can deliver insults and abusive comments in real time and cause immediate and direct harm. Schulenberg et al. [[47](https://arxiv.org/html/2409.15623v1#bib.bib47)] claimed that social VR might lead to more severe forms of harassment compared to other contexts. Therefore, hate speech misconducts such as racial slurs in voice channels faced harsher punishments than in text channels [[24](https://arxiv.org/html/2409.15623v1#bib.bib24)].

In the context of social VR, Schulenberg et al. [[47](https://arxiv.org/html/2409.15623v1#bib.bib47)] provided an overview of moderation efforts including Community Guidelines and Punishments, and Moderation Pipelines that are mainly based on human moderation, and introduced the potential of AI moderation in social VR. They concluded that the awareness of the presence of an AI moderation system that can detect the large-scale embodied and immersive multi-user virtual environments and act in the moment can effectively prevent harassment in social VR before it occurs [[47](https://arxiv.org/html/2409.15623v1#bib.bib47)]. Existing safety-enhancing features on social VR platforms such as blocking, personal space bubble, muting, reporting players or trust systems [[15](https://arxiv.org/html/2409.15623v1#bib.bib15)] to keep users safe from problematic users were found to have limitations [[34](https://arxiv.org/html/2409.15623v1#bib.bib34), [13](https://arxiv.org/html/2409.15623v1#bib.bib13)]. First, they place the burden of moderation on users, who might be ill-equipped or unaware of effective strategies [[22](https://arxiv.org/html/2409.15623v1#bib.bib22)]. Second, platforms rely on volunteer moderators to manage behavior in public virtual rooms [[8](https://arxiv.org/html/2409.15623v1#bib.bib8)], but they cannot cover most incidents. Sabri et al. [[46](https://arxiv.org/html/2409.15623v1#bib.bib46)] showed that only 24% of incidents in social VR are addressed by moderators, highlighting the need for better moderation tools.

Automated embodied moderation has been investigated as an alternative solution [[13](https://arxiv.org/html/2409.15623v1#bib.bib13)] to create safer spaces in social VR by providing a protective figure that takes action to mitigate harmful interactions. Previous studies [[13](https://arxiv.org/html/2409.15623v1#bib.bib13), [14](https://arxiv.org/html/2409.15623v1#bib.bib14)] explored the use of VR-embodied AI agents to maintain safe and respectful interactions in social VR. Fiani et al. [[14](https://arxiv.org/html/2409.15623v1#bib.bib14)] introduced “Big Buddy” as a Wizard-of-Oz automated agent in a simulated social VR game and intervenes when a fictional player disrupts the child’s game. Unlike traditional moderation tools(e.g., reporting and blocking) that operate in the background and would have flaws in the process and lack trust and feelings of unfair treatment discouraging users from using them [[14](https://arxiv.org/html/2409.15623v1#bib.bib14)], embodied agents are visually present within the virtual environment, allowing for real-time interaction and enforcement of community guidelines. However, these AI-based moderation systems have key limitations in their ability to detect problematic events, parse ambiguity, and identify false positives [[14](https://arxiv.org/html/2409.15623v1#bib.bib14), [30](https://arxiv.org/html/2409.15623v1#bib.bib30)]. False positives are produced by mistakenly flagging benign content as harmful [[28](https://arxiv.org/html/2409.15623v1#bib.bib28)]. Misidentifying content as harmful or inappropriate can lead to unjust removals or sanctions, which could harm users who may be unfairly targeted [[44](https://arxiv.org/html/2409.15623v1#bib.bib44)].

Dealing with uncertainty and missing contextual factors (e.g., tones and pitch in voice) remains a key challenge for automated embodied moderation [[13](https://arxiv.org/html/2409.15623v1#bib.bib13)]. Recent advances in AI, particularly in LLMs, along with immersive VR and avatar interfaces, allow for the creation of embodied conversational agents that can engage in natural dialogue with humans [[43](https://arxiv.org/html/2409.15623v1#bib.bib43)]. As multiple experimentations [[9](https://arxiv.org/html/2409.15623v1#bib.bib9), [48](https://arxiv.org/html/2409.15623v1#bib.bib48), [4](https://arxiv.org/html/2409.15623v1#bib.bib4), [2](https://arxiv.org/html/2409.15623v1#bib.bib2)] have explored incorporating LLMs into LLM-agent conversations, LLM-based agents are becoming more and more prevalent. LLMs can quickly generate dynamic and high-quality responses that adjust to a player’s actions, choices, and the overall state of the game world [[9](https://arxiv.org/html/2409.15623v1#bib.bib9), [43](https://arxiv.org/html/2409.15623v1#bib.bib43)]. Mehta et al. [[37](https://arxiv.org/html/2409.15623v1#bib.bib37)] highlighted that conversational AIs and LLMs in LLM-agent interactions as ’extremely viable’ and a potential successor of traditional pre-scripted dialogues. Therefore, the emergence of VR-embodied LLM-based agents in social VR opens up underexplored opportunities to leverage LLM’s potential for not only generating conversations but also capturing and detecting real-time voice-based hate speech.

### 1.3 LLMs for RT Voice-based Hate Speech Detection

LLMs have shown promise in real-time (RT) voice-based hate speech detection due to their capability for identifying the violating content and providing the reasoning on why the content violated rules [[28](https://arxiv.org/html/2409.15623v1#bib.bib28)]. LLMs showed good contextual understanding, which allows them to provide correct reasoning when handling rule violations in online communities [[28](https://arxiv.org/html/2409.15623v1#bib.bib28)]. LLMs also have demonstrated state-of-the-art performance in natural language tasks. LLMs have undergone extensive training using vast amounts of natural language data [[21](https://arxiv.org/html/2409.15623v1#bib.bib21)], enabling them to grasp intricate contextual details. This feature enables LLMs to detect RT verbal hate speech in complex or ambiguous scenarios, where traditional approaches fall short for the existing supervised learning-based detection methods cannot fully capture context to make accurate predictions [[21](https://arxiv.org/html/2409.15623v1#bib.bib21)].

The robustness of LLMs to understand and interpret text right after conversion from audio, even when it contains typographical errors, slang, or informal language may contribute to faster detection times in the real-time hate speech detection. Traditional machine learning models and deep learning approaches require extensive text preprocessing to normalize the input data, including tokenization, stop words removal, stemming, and lemmatization [[26](https://arxiv.org/html/2409.15623v1#bib.bib26)]. These steps are essential for ensuring that the input data is consistent and interpretable. On the contrary, LLMs can comprehend the meaning and context of raw text inputs without requiring preprocessing owing to their extensive pre-training, which could translate into a more streamlined and efficient pipeline.

Additionally, LLMs require minimal feature engineering, streamlining the development process and allowing for faster deployment. This is particularly advantageous in real-time voice moderation, where rapid and accurate detection is important for maintaining a safe and inclusive social VR environment. In comparison, traditional and deep learning Machine Learning methods such as Recurrent Neural Networks, Deep Neural networks, and long short-term memory networks require extensive feature engineering and large labeled datasets [[19](https://arxiv.org/html/2409.15623v1#bib.bib19)]. Fine-tune pre-trained transformers such as BERT, RoBERTa and ALBERT [[45](https://arxiv.org/html/2409.15623v1#bib.bib45)] are resource-intensive during both training and inference.

Limitations of LLMs in Hate Speech Detection LLMs are not able to detect tones and emotions within verbal interactions, particularly in edge cases where tone and emotions are crucial for differentiating the nature of verbal speech [[29](https://arxiv.org/html/2409.15623v1#bib.bib29)]. Rana et al. [[45](https://arxiv.org/html/2409.15623v1#bib.bib45)] stated that the most essential features in classifying hate speech would be the speaker’s emotional state and its influence on the spoken words. Tones, emotions, and vocal intonations play crucial roles in conveying the intent behind words. The same sentence can have different meanings based on the speaker’s tone, whether to be playful, sarcastic, angry, calm. For example, Rana et al. [[45](https://arxiv.org/html/2409.15623v1#bib.bib45)] asserted that a political leader calmly discussing immigration policies at a conference is less harmful than delivering the same speech with extreme anger and disgust towards a specific targeted user, as the latter incites hostility against immigrants in the country. Another example is the different interpretations of certain words depending on the emotion and context [[45](https://arxiv.org/html/2409.15623v1#bib.bib45)]. When a friend jokingly calls someone a name in a playful tone and the conversation remains calm or civil, it should not be classified as hate speech. However, if the same words were delivered with a harsh tone and intent to attack specific individuals, they should be detected as offensive or hurtful and classified as hate speech. Since LLMs rely solely on plain text, they are unable to capture the subtleties of spoken language. This limitation results in misclassifications and inaccuracies, especially false positives, of verbal harassment in hate speech detection. False positives often due to LLMs’ difficulty in accurately interpreting human emotions [[29](https://arxiv.org/html/2409.15623v1#bib.bib29)], as important contextual cues present in tone are not conveyed through text. For example, LLM-based moderator might take the user’s exaggerated language as disrespectful to others [[29](https://arxiv.org/html/2409.15623v1#bib.bib29)].

### 1.4 Improving the Accuracy of LLM Using Audio Feature Analysis for Hate Speech Detection

Audio features can be analyzed and applied to address the shortcomings of LLMs in detecting hate speech. Rana et al. [[45](https://arxiv.org/html/2409.15623v1#bib.bib45)] suggested that the classification of hate speech is significantly influenced by the speaker’s emotional state and its impact on their spoken words. The study claimed that incorporating emotion into hate speech detection can help reduce false positives in systems that rely solely on text data as input. Patrick et al. [[42](https://arxiv.org/html/2409.15623v1#bib.bib42)] also asserted that there is a close link between hate speech and the emotional and psychological state of the speaker and evident in the emotional tone of their language.

Kumar et al. [[29](https://arxiv.org/html/2409.15623v1#bib.bib29)] showed that incorporating context in LLM rule-based moderation corrected 35% of errors with minimal changes to the prompt. Integrating audio feature analysis with LLMs can enhance hate speech detection by providing context that text alone cannot capture. Barakat et al. [[5](https://arxiv.org/html/2409.15623v1#bib.bib5)] demonstrated that MFCC audio features significantly improved the accuracy of independent keyword spotting (KWS) for detecting offensive language in video blogs, greatly outperforming the existing speech-to-text methods, which indicates the potential to use audio features for detecting audio-based hate speech.

Das et al. [[10](https://arxiv.org/html/2409.15623v1#bib.bib10)] identified that hate speech demonstrated a certain pattern in temporal features including zero crossing rate and root mean square energy. This pattern can be used to assist hate speech detection by capturing distinctive acoustic characteristics. Analyzing elements like pitch, volume, and speech rate gives insights into the speaker’s emotional state and intent, thereby improving classification accuracy. Khan et al. [[27](https://arxiv.org/html/2409.15623v1#bib.bib27)] highlighted the effectiveness of combining audio features such as pitch and Mel-Frequency Cepstral Coefficients (MFCC) with models like K-Nearest Neighbors (KNN) and Naive Bayes to enhance emotion classification accuracy. Hu et al. [[23](https://arxiv.org/html/2409.15623v1#bib.bib23)] found that using MFCC features with a Gaussian Mixture Model (GMM) combined with a Support Vector Machine (SVM) outperformed traditional GMM methods, achieving improvements in emotion classification accuracy. Similarly, Zhou et al. [[52](https://arxiv.org/html/2409.15623v1#bib.bib52)] introduced GMM supervector with SVM using MFCC features achieving an 88% accuracy rate in classifying emotions.

Finally, recent work by Roblox [[6](https://arxiv.org/html/2409.15623v1#bib.bib6)] highlights the advancement of combining a proprietary text filter and audio features analysis for detecting hate speech. They demonstrated the efficacy of a CNN model to create classifiers on MFCC audio features. The model can detect and categorize verbal hate speech into different categories including profanity, dating and sexual, racism, and bullying. This combined method achieved an average precision of 94.48% [[6](https://arxiv.org/html/2409.15623v1#bib.bib6)], demonstrating the effectiveness of integrating audio features with text analysis to enhance detection capacity. We are extending this work by leveraging publicly available GPT model and aim at reducing false positives.

## 2 Methodology

Our approach involves the following key steps:

*   •

    Defining voice-based hate speech criteria in social VR,

*   •

    Designing and implementing an LLM-based agent Safe Guard for hate speech detection,

*   •

    Exploring suitable LLMs prompting method for real-time hate speech detection,

*   •

    Processing voice messages through text conversion and applying real-time LLM-based detection on transcripts. In parallel, extract and analyze the key features of audio using a CNN classifier to assist detection,

*   •

    Building and integrating the LLM-based detection system into the Safe Guard agent architecture to enhance moderation capabilities,

*   •

    Collecting and analyzing evaluation metrics to iteratively refine and optimize the proposed system.

### 2.1 Hate Speech Training and Testing Datasets

To the best of our knowledge, there are no publicly available audio datasets for hate speech detection. To meet the purpose of our real-time voice-based hate speech detection, we used the HATEMM dataset [[10](https://arxiv.org/html/2409.15623v1#bib.bib10)], which was sourced from the BitChute platform and consists of 1083 videos annotated for hate speech. This dataset includes a ground truth annotation file, with 39.8% of the samples labeled as hate speech. We extracted audio from these videos and transcribed it into text using the OpenAI Whisper. The dataset was split into 80% for training purposes, and 20% for testing. The processing approaches involved extracting audio from the videos using FFmpeg API. The audio was then converted into text transcripts with OpenAI Whisper API, and the transcripts were stored in files. Further processing was conducted by combining the transcripts with extracted audio features to improve detection accuracy.

### 2.2 LLM Set Up and Prompt with Hate Speech Moderation Rules

In this study, we employed ChatGPT 3.5 as our LLM model for hate speech detection in social VR. Our choice of this model was due to several key considerations. Firstly, GPT 3.5 is capable of handling complex contextual nuances. Secondly, GPT 3.5 is more practical in real-time detection because it requires less computational resources and is more efficient than GPT 4\. Overall, it offers a good balance between performance and resource usage.

Before using LLMs for real-time detection, we provided information to the models on datasets labeled for hate speech. Prompt-based strategies have been found to effectively guide LLMs in leveraging the context of the specific task [[32](https://arxiv.org/html/2409.15623v1#bib.bib32)]. A prompt is a query or statement designed to instruct the model on what is being asked. The effectiveness of an LLM can be significantly improved with carefully crafted prompts, underscoring the importance of prompting techniques in leveraging the context of these models. Guo, et al. [[21](https://arxiv.org/html/2409.15623v1#bib.bib21)] claimed that the effectiveness of LLMs in identifying hate speech is highly contingent upon the design of the prompt.

### 2.3 Convolutional Neural Network Audio Feature Model

The CNN model is a class of deep learning algorithms primarily used for processing structured grid data, which is suitable for our audio parameters. CNN model is capable of using backpropagation through different layers to build an accurate classifier. Convolutional layers use small learnable filters to identify features across the input data. Activation layers like ReLU introduce non-linearity by converting negative values to zero, assisting the network in learning patterns. Pooling layers reduce the spatial dimensions of the data and reduce the computational load. Following several convolutional and pooling layers, the data is flattened into a 1D vector to classify at last.

## 3 System Design

### 3.1 Safe Guard Agent Design

The embodied LLM agent Safe Guard is designed to facilitate real-time interactions with players while proactively monitoring and moderating conversations in VRChat. Building on the previous work by Park et al. [[40](https://arxiv.org/html/2409.15623v1#bib.bib40)] and Wan et al. [[49](https://arxiv.org/html/2409.15623v1#bib.bib49)], Safe Guard incorporates several features for context management and moderation. The system maintains memories of interactions, capturing not only the dialogue but also the player’s attitude, inferred mood based on input messages, and the time when the conversation took place [[49](https://arxiv.org/html/2409.15623v1#bib.bib49)]. This comprehensive memory is crucial for understanding and interpreting ongoing interactions in social VR. To manage and evaluate context effectively, Safe Guard uses advanced techniques such as exponential decay to prioritize recent interactions and cosine similarity metrics to assess the relevance of past conversations [[49](https://arxiv.org/html/2409.15623v1#bib.bib49)]. These methods ensure that Safe Guard accurately evaluates each interaction’s importance and relevance to current discussions.

Dual Mode Operation: Safe Guard can operate in both conversational mode (engaging with a single user) and observational mode (monitoring group interactions), switching as needed based on the scenario.

Key aspects of the agent system are defined below:

GPT Module. ChatGPT is deployed throughout the process of Safe Guard system design to process user input, understand the context of social situations and generate appropriate observations or responses. It not only facilitates human-like interactions but also actively monitors for potential instances of hate speech.

LLM-Agent Formation Module. LLM-agent is created by assigning it a base description that includes name, character details, preferences, etc [[49](https://arxiv.org/html/2409.15623v1#bib.bib49)]. Safe Guard is tailored specifically for hate speech detection. Its character details are ”A vigilant, neutral, and approachable guardian, focused on maintaining respectful communication within VR spaces”. Unity engine was used to modify the chosen avatar to include a set of animations for expressions and actions.

Observation Database Module. The system maintains a collection of context observations that store all observations generated by LLM for each conversation. This includes initial observations from the LLM-agent’s ”Base Description”. Further conversational contexts are captured by generating up to three observations for each message [[49](https://arxiv.org/html/2409.15623v1#bib.bib49)]. Memory observations are defined as combinations of Base descriptions and Context observations (see [[49](https://arxiv.org/html/2409.15623v1#bib.bib49)] for more details on the LLM-agent itself).

![Refer to caption](img/ee165f9b868fabe5894aa95a1eb2cbde.png)

Figure 1: Safe Guard System Design in Conversational Mode

### 3.2 Hate Speech Detection System

The real-time hate speech detection system is designed to integrate into VRChat, ensuring a safe and respectful environment. As shown in [Fig. 1](https://arxiv.org/html/2409.15623v1#S3.F1 "In 3.1 Safe Guard Agent Design ‣ 3 System Design ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality"), the flowchart illustrates the workflow of the proposed Safe Guard hate speech detection system in conversational mode. The system operates similarly in observational mode when multiple players are presented, except that the LLM-agent does not generate dialogue with the players.

### 3.3 Prompt Engineering for LLM Model

To set up our hate speech detection system, we used specific prompts for ChatGPT 3.5 based on established hate speech definitions and reasoning to produce a binary response. We compared and decided on the most suitable and effective prompting method in the experiment stage. Since the project is applied to real-time detection, we focused on prompt methods that do not need lengthy context or multiple sequential prompts. We mainly evaluated three different approaches as discussed below.

#### 3.3.1 Approach #1: Direct Prompting

For this method, we used a straightforward prompt to ask GPT to classify text as either ”Hate” or ”Non Hate” without providing any extra instruction. This prompting method has been proven to be effective in previous study [[31](https://arxiv.org/html/2409.15623v1#bib.bib31)]. Using the general prompt method can rapidly enable the LLM model to complete the voice hate speech detection task. The prompting message we used for this method was: ”You are a classifier that identifies whether a given text contains hate speech or not. Is the following text hate speech?  Respond with Yes or No. Here is the text: ”

#### 3.3.2 Approach #2: Prompting with Hate Speech Definition

This method involved providing a clear definition of hate speech within our use case and asking if a specific text fits that definition. The prompting message we used for this method was: ”Hate speech in social VR includes any speech that attacks a person or group based on attributes such as race, religion, ethnic origin, national origin, sex, disability, sexual orientation, or gender identity. Consider this context carefully. Is the following text hate speech?  Respond with Yes or No. Text: ”

#### 3.3.3 Approach #3: Few-Shot Learning Prompting

Besides the definition of hate speech, we provided several examples with ground truth labels to help GPT learn what constitutes hate speech and what does not. The prompting message we used for this method was: ” Hate speech includes any speech that attacks a person or group based on attributes such as race, religion, ethnic origin, national origin, sex, disability, sexual orientation, or gender identity. Consider this context carefully.

Here are some examples of hate speech and non-hate speech:

Example 1: Text: ”I think people of all races should be treated equally.” Hate Speech: No

Example 2: Text: ”People from [name] country are all criminals and should be deported.” Hate Speech: Yes

Example 3: Text: ”Everyone deserves to be loved, regardless of their gender identity.” Hate Speech: No

Example 4: Text: ”I think all immigrants should get out of Canada.” Hate Speech: Yes

Determine if the following text is hate speech. Respond with Yes or No. Text: ”

### 3.4 Audio feature Extraction

Voice consists of voice signals and their content. Apart from content, voice signals can also provide information for hate speech detection. Feature extraction is the process of extracting and tackling hidden information in the raw data signal [[1](https://arxiv.org/html/2409.15623v1#bib.bib1)]. We used root mean square(RMS) and Mel-Frequency Cepstrum Coefficients(MFCC) for audio feature analysis.

Root mean square (RMS) is the square root value of the mean of the sum of squares of the signal. In the context of audio signals, RMS is often used to measure the power or loudness of the signal [[10](https://arxiv.org/html/2409.15623v1#bib.bib10)]. It provides a single value that represents the average energy of the waveform, which is particularly useful for analyzing the amplitude of audio signals over time.

|  | $\text{RMS}=\sqrt{\frac{1}{N}\sum_{i=1}^{N}x[i]^{2}}$ |  | (1) |

where:

*   •

    $x[i]$ is the amplitude of the signal at the $i$-th sample,

*   •

    $N$ is the total number of samples.

Mel Frequency Cepstrum Coefficients (MFCCs) are widely used in speech and audio processing as a representation of the short-term power spectrum of sound [[33](https://arxiv.org/html/2409.15623v1#bib.bib33), [1](https://arxiv.org/html/2409.15623v1#bib.bib1)]. They are useful in various tasks such as speech recognition, speaker identification, emotion recognition, and audio classification. Given their utility, incorporating MFCCs into our hate speech detection method is beneficial. Ali et al. [[1](https://arxiv.org/html/2409.15623v1#bib.bib1)] stated that MFCC can be calculated by conducting five consecutive processes, namely signal framing, computing of the power spectrum, applying a Mel filter bank to the obtained power spectra, calculating the logarithm values of all filter banks, and finally applying the Discrete Cosine transform (DCT).

After an initial exploration of audio features, we decided to use the 40-dimensional vector MFCC features in our CNN model, in order to capture key features of the user input audio data. To extract these features from WAV audio files, we used the Python library librosa and stored the results in a CSV file for further analysis.

### 3.5 Audio Feature Model

After extracting features of audio files, we used machine learning techniques to build a CNN classifier based on those features.

Initially, we set up a sequential model consisting of a linear stack of layers. The input layer included a 2D convolutional layer with 32 filters, each of size 3x1, designed to extract spatial features from the input data, which has an input shape of 40 dimensions with 1 feature. Next, we added a second convolutional layer and a second max pooling layer. Finally, a flattening layer was used to convert the 2D data into 1D, followed by two fully connected dense layers to perform the final classification.

### 3.6 Using a Combination of LLM and Audio Feature Model to Detect Hate Speech

In our proposed system, we combined GPT 3.5 as the primary method with an audio-based CNN classifier to enhance the overall detection performance. The integration of these two methods aims to combine the strengths of both NLP and audio signal analysis to improve the accuracy and reliability of the detection results.

After converting audio to text, the transcript is delivered to the prompted LLM model. The prompted LLM model goes through the transcript and detects whether hate speech is in it. Simultaneously, audio key features are extracted and analyzed by the CNN classifier to generate a probability score. If this probability score is greater than 0.5, the input is classified as ’Hate.’ A threshold of 0.5 is commonly used for making binary classification decisions.

To determine the final classification result of the combined model, we applied the following decision rule: The audio input is classified as ’hate speech’ only if both the GPT model and the audio-based CNN classifier predict it as ’hate.’ Otherwise, the final classification is ’non-hate speech.’ Once the system successfully detects hate speech, the system will immediately display a voice notification in the agent’s dialogue to indicate that hate speech has been detected (see Figure [1](https://arxiv.org/html/2409.15623v1#S3.F1 "Figure 1 ‣ 3.1 Safe Guard Agent Design ‣ 3 System Design ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality")).

### 3.7 Integration to Safe Guard Agent System

We integrated the hate speech detection system into the Safe Guard LLM-agent and processed the audio within the module to determine if hate speech was present. Once hate speech is detected, the program will immediately display a warning message in the Safe Guard dialogue. We fine-tuned the silent detection parameters, including increasing the max silence length to 2 seconds and lowering the threshold to -40db for activating the recording procedure. In this way, we were able to retrieve segmented sentences instead of a whole speech for audio processing. We also altered the sample rate to 44100Hz, which is the maximum rate that is compatible with the Safe Guard agent system to get better recording quality and support audio feature extraction.

## 4 Experiment

### 4.1 Procedure

VRChat System Setup for Experiment: To simulate the real scenario environment in VRChat, we connected two devices that log in with different accounts in VRChat.One account operated as the host server, running the LLM agent and detection system to capture audio from the test subject. Meanwhile, the second account logged in as the player, transmitting the test speech dataset via microphone.

During the audio feature process, the combination of RMS and MFCCs played a key role in classifying the hate and non-hate datasets, yielding the best results for feature selection. Extracted features went through a CNN training model, which produced a similarity score to the recognized hate speech pattern.

Prompting Experiment: We evaluated the effectiveness of our LLM hate speech detection system using different prompting techniques. We tested with the 50 videos containing hate speech from our primary MM Hate dataset. True labels for each data will be derived from the manual categorization of the video dataset.

### 4.2 Evaluation Metrics

#### 4.2.1 Latency: Real-Time Detection Time

To evaluate the performance of the real-time hate speech detection system, we measured the latency at each stage of the detection process. The overall latency includes four main components as follows: (1) Audio Feature Extract Time: The duration of the audio feature extraction from raw audio data, (2) Speech-to-Text Conversion Time: The time taken to transcribe the audio input into text using a speech recognition engine, (3) Audio Feature Prediction Time: The time taken by the CNN classifier to classify the audio based on audio features as hate speech or non-hate speech, and (4) LLM Analysis Time: The time taken by the LLM to classify the text as hate speech or non-hate speech.

The total detection time is the sum of these individual times, representing the overall latency from receiving the audio input to providing the moderation output.

#### 4.2.2 Accuracy Metrics

To evaluate the accuracy of our real-time voice-based hate speech detection system in social VR, we utilized several key metrics that are well-known and academically validated [[21](https://arxiv.org/html/2409.15623v1#bib.bib21)] as follows:

Accuracy: Accuracy measures the proportion of correct detections (both true positives and true negatives) out of all detections. It is given by:

|  | $\text{Accuracy}=\frac{\text{True Positives (TP)}+\text{True Negatives (TN)}}{% \text{Total Instances}}$ |  |

Precision: Precision measures the proportion of true positive detections (correctly identified instances of hate speech) out of all positive detections (instances flagged as hate speech). It is calculated as:

|  | $\text{Precision}=\frac{\text{True Positives (TP)}}{\text{True Positives (TP)}+% \text{False Positives (FP)}}$ |  |

Recall (Sensitivity): Recall evaluates the proportion of true positive detections out of all actual instances of hate speech. It is defined as:

|  | $\text{Recall}=\frac{\text{True Positives (TP)}}{\text{True Positives (TP)}+% \text{False Negatives (FN)}}$ |  |

F1 Score: The F1 score is the harmonic mean of precision and recall, providing a single metric that balances both. It is calculated as:

|  | $\text{F1 Score}=2\times\frac{\text{Precision}\times\text{Recall}}{\text{% Precision}+\text{Recall}}$ |  |

## 5 RESULTS

### 5.1 Validation Dataset

A total of 204 video clips were collected using search API from YouTube with manually annotated true labels for validation. 104 of them are hate speech, and 100 of them are non-hate speech. We filtered the data within 20 seconds to better simulate real conversation scenarios in the VRChat setting.

### 5.2 Data Analysis

#### 5.2.1 GPT3.5 Prompting Methods Comparison Analysis

Table. [1](https://arxiv.org/html/2409.15623v1#S5.T1 "Table 1 ‣ 5.2.1 GPT3.5 Prompting Methods Comparison Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality") below compares solely the GPT’s performance across three prompting methods discussed above for hate speech detection using our training dataset. Since the datasets consist of long narrative speech rather than short conversational dialogues, the results did not show great accuracy.

| Method | Acc. (%) | Prec. (%) | Rec. (%) | F1 (%) |
| Direct |
| Non Hate |  | 71.43 | 19 | 29 |
| Hate |  | 56 | 93 | 70 |
| Overall | 57.52 | 63 | 56 | 50 |
| Definition Prompt |
| Non Hate |  | 67 | 77 | 71 |
| Hate |  | 91 | 85 | 88 |
| Overall | 82.98 | 79 | 81 | 80 |
| Few-shot |
| Non Hate |  | 78 | 78 | 78 |
| Hate |  | 93 | 93 | 93 |
| Overall | 85 | 89 | 89 | 80 |

Table 1: Comparison of GPT-3.5 Performance for Hate Speech Detection

Among the three methods, few-shot prompting emerged as the most effective method for hate speech detection, outperforming the other two methods. It achieved higher accuracy and balanced precision and recall, making it a more suitable choice in our use case.

#### 5.2.2 Detection System Result Analysis

Comparison with Baseline Models: We compared the performance metrics of our proposed combination model against the GPT-3.5 alone, and the audio feature model alone as two baseline models. The results are shown in Table. [2](https://arxiv.org/html/2409.15623v1#S5.T2 "Table 2 ‣ 5.2.2 Detection System Result Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality") which displays the classification performance evaluation report including Precision, Recall, and F1-score for both hate and non-hate categories.

| Model | Class | Precision | Recall | F1-Score |
| GPT | Hate | 0.95 | 0.95 | 0.95 |
| Non-Hate | 0.94 | 0.94 | 0.95 |
| Accuracy | 0.95 |
| Macro avg | 0.95 | 0.95 | 0.95 |
| Weighted avg | 0.95 | 0.95 | 0.95 |
| Audio | Hate | 0.73 | 0.53 | 0.62 |
| Non-Hate | 0.63 | 0.79 | 0.71 |
| Accuracy | 0.67 |
| Macro avg | 0.68 | 0.67 | 0.66 |
| Weighted avg | 0.68 | 0.67 | 0.66 |
| Combined | Hate | 0.96 | 0.53 | 0.69 |
| Non-Hate | 0.67 | 0.99 | 0.80 |
| Accuracy | 0.75 |
| Macro avg | 0.82 | 0.76 | 0.74 |
| Weighted avg | 0.82 | 0.76 | 0.74 |

Table 2: Classification Report for GPT Model, Audio Model, and Combined Method

![Refer to caption](img/65d636fcd872b1f36326f369591009f4.png)

Figure 2: GPT Model Alone

![Refer to caption](img/02379a1eed470d25288869e3b61ed2f0.png)

Figure 3: Audio Feature Model Alone

![Refer to caption](img/18c7dbd5c9a34c6337352c0b7ddf119c.png)

Figure 4: Combined Model

The GPT Model showed strong and consistent performance across all metrics (Fig. [4](https://arxiv.org/html/2409.15623v1#S5.F4 "Figure 4 ‣ 5.2.2 Detection System Result Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality")). For both hate and non-hate speech, it achieved high Precision and Recall of around 0.95\. The high accuracy indicated that the GPT Model was very reliable for detecting both hate and non-hate speech. However, the downside was that the false positive rate was fairly high compared with the combined model.

The Audio Model exhibited lower performance than GPT model (Fig. [4](https://arxiv.org/html/2409.15623v1#S5.F4 "Figure 4 ‣ 5.2.2 Detection System Result Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality")). For hate speech, it recorded a Precision of 0.73 and a Recall of 0.53\. This indicated that the model struggled with false negatives in hate speech detection. In the case of non-hate speech, the model performed slightly better, with a Precision of 0.63, and a Recall of 0.79\. Overall, the Audio Model achieved an accuracy of 0.67, reflecting moderate effectiveness.

The combined Method exhibited a more balanced performance (Fig. [4](https://arxiv.org/html/2409.15623v1#S5.F4 "Figure 4 ‣ 5.2.2 Detection System Result Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality")). For hate speech, it achieved a high Precision of 0.96 but a lower Recall of 0.53, resulting in an F1-Score of 0.69\. For non-hate speech, the Combined Method showed a Precision of 0.67 and an outstanding Recall of 0.99\. The overall accuracy of 0.75 indicated that the Combined Method outperforms the Audio Model, and can enhance the GPT Model detection results.

Comparison of False Positive Rate (FPR) across 3 Models: FPR is calculated as FP / FP+TN, where FP is the number of false positives and TN is the number of true negatives. As the results are shown in Fig. [4](https://arxiv.org/html/2409.15623v1#S5.F4 "Figure 4 ‣ 5.2.2 Detection System Result Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality"), GPT model’s FPR was 0.06, indicating that 6% of the non-hate speech cases were mistakenly classified as hate speech. Meanwhile, the Audio Feature Model exhibited a much higher FPR of 0.21 (Fig. [4](https://arxiv.org/html/2409.15623v1#S5.F4 "Figure 4 ‣ 5.2.2 Detection System Result Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality")), exhibiting a greater tendency to misclassify non-hate speech. In contrast, the Combined Model performed significantly better, with an improved FPR of just 0.01 (Fig. [4](https://arxiv.org/html/2409.15623v1#S5.F4 "Figure 4 ‣ 5.2.2 Detection System Result Analysis ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality")), meaning it only misclassified 1% of the non-hate speech cases as hate speech, which illustrated the effectiveness of the combined approach in reducing false positives compared to the individual models.

#### 5.2.3 Latency Measurement

We collected and analyzed latency measurements during different stages of the pipeline. We identified the average, minimum, and maximum latency times during tests. [Table 3](https://arxiv.org/html/2409.15623v1#S5.T3 "In 5.2.3 Latency Measurement ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality") shows the processing times for different stages of the system.

| Process | Min (s) | Mean (s) | Max (s) |
| --- | --- | --- | --- |
| Transcription | 0.48 | 0.95 | 1.99 |
| LLM Analysis | 0.27 | 0.48 | 1.83 |
| Audio Extraction | 0.006 | 0.019 | 0.045 |
| Audio Prediction | 0.05 | 0.055 | 0.065 |

Table 3: Processing times for different stages of the system.

Our tests demonstrated that the average latency for our proposed combined detection system is approximately 1.5 seconds, which we considered to be acceptable for our real-time verbal hate speech detection use case. The chart in Fig. [5](https://arxiv.org/html/2409.15623v1#S5.F5 "Figure 5 ‣ 5.2.3 Latency Measurement ‣ 5.2 Data Analysis ‣ 5 RESULTS ‣ Safe Guard: an LLM-agent for Real-time Voice-based Hate Speech Detection in Social Virtual Reality") illustrates that on average over 60% of the overall latency is due to transcribing, and 32% is attributed to GPT processing. Audio feature extraction and prediction time only accounted for around 5%. The findings align with our earlier discussion of the pros of cons of LLMs and audio feature analysis in the previous sections.

![Refer to caption](img/6768849b64f1bf94297c1c60ff84b104.png)

Figure 5: Overall Latency Distribution

## 6 DISCUSSION

To answer our RQ we leverage the power of LLMs and audio features to address the complex, often ambiguous nature of hate speech detection in real-time, interactive environments in social VR.

Effectiveness and Trade-off of LLM for Hate Speech Detection While the GPT Model demonstrated great overall performance with a high Accuracy of 0.95, it also exhibited a trade-off of false positives. The GPT Model’s Precision and Recall for hate speech detection were both strong, but its tendency to misidentify non-hate speech as hate speech led to a higher false positive rate. This may result in unnecessary moderation requests and potential user experience issues [[30](https://arxiv.org/html/2409.15623v1#bib.bib30)].

Integration of Audio Features with LLMs to Reduce False Positives Incorporating audio features into GPT presented a more balanced approach to hate speech detection. Although the overall Accuracy of the Combined Method was lower at 0.75 compared to the GPT Model alone, it demonstrates advantages in minimizing false positives. The Combined Method also achieved a high Precision of 0.96 for hate speech detection, indicating that when it identified hate speech, it was highly accurate. This high Precision meant fewer cases of non-hate speech being incorrectly labeled as hate speech, which can enhance user experience by reducing the likelihood of wrongful classifications and aligns better with real-world business needs.

By ensuring the detected hate speech was truly hate, the combined method can reduce the risk of incorrectly identifying non-hate speech as hate speech, which might harm the users’ experience. Additionally, the high accuracy of the true positive hate speech cases also showed the potential of the Combined Method to build an automated hate speech moderation pipeline, which could reduce the need for human moderation involvement.

Overcoming Misclassification Caused by Audio Interference In our review of misclassified cases, we identified several key factors contributing to detection errors, including poor audio quality, background noise (music or singing), and instances where the speaker was laughing while speaking. These issues made it difficult for both the audio model and GPT to interpret speech accurately. Addressing these challenges is crucial for enhancing the accuracy of real-time hate speech detection in social VR. A possible solution to these challenges is drawn from the previous study [[6](https://arxiv.org/html/2409.15623v1#bib.bib6)], which introduced a voice activity detection (VAD) model as a preprocessing step. VAD significantly increased the robustness of the overall pipeline, particularly for users with noisy microphones. By filtering out background noise and applying the detection pipeline only when human speech was detected, the system reduced the overall inference volume by approximately 10 percent and improved the quality of inputs. Future improvements in these areas will reduce errors and make AI-based moderation more reliable and effective.

In summary, the Combined Method’s ability to reduce false positives and improve accuracy in identifying non-hate speech made it a valuable enhancement over the GPT Model, aligning well with improving user experience goals and real-world application needs.

## 7 LIMITATIONS

Several limitations in our study might affect the overall effectiveness of the hate speech detection system.

Firstly, the CNN audio feature model needs more training to enhance its accuracy. The limited number of training data impacts the current model’s training. Conducting more training could improve the audio model’s prediction effectiveness.

Additionally, the study encountered challenges due to the insufficiency of training and validating data. The availability of a larger and more diverse dataset could help in developing more effective models and improving overall accuracy. For this, we recognize the need for in-platform (VRChat) data collection, however this imposes challenges regarding data collection and player’s privacy that needs to be address.

Lastly, upon manual examination, we discovered background interference, such as music, singing, and laughing, in the audio data reduced the model’s detection accuracy. In our study, we did not apply any related noise filtering or preprocessing techniques to mitigate the background interference.

## 8 FUTURE WORK

For conducting our study to improve the hate detection system in the future, several approaches could be considered.

First, expanding the dataset size would provide a more solid foundation for training the model, which could improve accuracy and generalizability. By incorporating more data samples, especially edge cases, the model would become better at handling diverse scenarios that might otherwise be overlooked.

Second, the current system could be expanded to include multimodal detection by combining both audio and visual inputs in social VR. Incorporating visual and audio data of users’ real-time interactions on social VR platforms holds the potential to produce more comprehensive analysis and ultimately enhance detection accuracy.

Furthermore, adding the capability to identify and categorize various forms of hate speech, such as Racial, Ethnic, Religious, and Sexual Orientation Hate Speech, would significantly augment the model’s detection capabilities. This approach would make the system more effective in recognizing specific types of hateful content, and add reasoning to detection results, thereby increasing its practical usage in social VR.

Additionally, training and assessing the system’s performance in specific real-world scenarios such as in real VRChat settings would provide valuable insights into its applicability and robustness.

Lastly, future research should investigate effective human-AI collaboration methods to further reduce false positives and improve the accuracy of hate speech detection by ensuring that automated AI handles routine moderation tasks while human moderators address more complex or nuanced cases.

## 9 CONCLUSION

This study proposes a combined approach for real-time voice-based hate speech detection in social VR that leverages the strengths of both GPT and an audio CNN classifier. The combined method improved precision and reduced false positives, making it more practical for real-world applications. By using GPT as the main method and incorporating the assisting audio classification, the system achieved a more balanced hate speech classification. The findings highlight the potential of multi-modal approaches in enhancing the detection and classification of hate speech, providing a foundation for future improvements and applications in various domains.

## References

*   [1] S. Ali, S. Tanweer, S. Khalid, and N. Rao. Mel frequency cepstral coefficient: A review. 01 2021\. doi: 10 . 4108/eai . 27-2-2020 . 2303173
*   [2] P. Ammanabrolu, J. Urbanek, M. Li, A. Szlam, T. Rocktäschel, and J. Weston. How to motivate your dragon: Teaching goal-driven agents to speak and act in fantasy worlds. In Proc. 2021 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, pp. 807–833\. Association for Computational Linguistics, Online, June 2021\. doi: 10 . 18653/v1/2021 . naacl-main . 64
*   [3] A. Arango. Language agnostic hate speech detection. In Proc. 43rd International ACM SIGIR Conference on Research and Development in Information Retrieval, SIGIR ’20, p. 2475\. Association for Computing Machinery, New York, NY, USA, 2020\. doi: 10 . 1145/3397271 . 3401447
*   [4] T. Ashby, B. K. Webb, G. Knapp, J. Searle, and N. Fulda. Personalized quest and dialogue generation in role-playing games: A knowledge graph- and language model-based approach. In Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems, CHI ’23\. Association for Computing Machinery, New York, NY, USA, 2023\. doi: 10 . 1145/3544548 . 3581441
*   [5] M. S. Barakat, C. H. Ritz, and D. A. Stirling. Detecting offensive user video blogs: An adaptive keyword spotting approach. In 2012 International Conference on Audio, Language and Image Processing, pp. 419–425, 2012\. doi: 10 . 1109/ICALIP . 2012 . 6376654
*   [6] K. Bhat. Deploying ml for voice safety. [https://corp.roblox.com/newsroom/2024/06/deploying-ml-for-voice-safety](https://corp.roblox.com/newsroom/2024/06/deploying-ml-for-voice-safety), July 2024. Accessed: 2024-08-07.
*   [7] L. Blackwell, J. Dimond, S. Schoenebeck, and C. Lampe. Classification and its consequences for online harassment: Design insights from heartmob. Proc. ACM Hum.-Comput. Interact., 1(CSCW):19, Dec. 2017\. doi: 10 . 1145/3134659
*   [8] L. Blackwell, N. Ellison, N. Elliott-Deflo, and R. Schwartz. Harassment in social virtual reality: Challenges for platform governance. Proc. ACM Hum.-Comput. Interact., 3(CSCW), Nov. 2019\. doi: 10 . 1145/3359202
*   [9] L. M. Csepregi. The effect of context-aware llm-based npc conversations on player engagement in role-playing video games. Master’s thesis, Aalborg Universitet, 2023.
*   [10] M. Das, R. Raj, P. Saha, B. Mathew, M. Gupta, and A. Mukherjee. Hatemm: A multi-modal dataset for hate video classification. Proceedings of the International AAAI Conference on Web and Social Media, 17:1014–1023, 06 2023\. doi: 10 . 1609/icwsm . v17i1 . 22209
*   [11] T. Davidson, D. Warmsley, M. Macy, and I. Weber. Automated hate speech detection and the problem of offensive language, 2017.
*   [12] M. Duggan. Online harassment 2017, 2017.
*   [13] C. Fiani, R. Bretin, S. A. Macdonald, M. Khamis, and M. Mcgill. ”pikachu would electrocute people who are misbehaving”: Expert, guardian and child perspectives on automated embodied moderators for safeguarding children in social virtual reality. In Proceedings of the CHI Conference on Human Factors in Computing Systems, CHI ’24\. Association for Computing Machinery, New York, NY, USA, 2024\. doi: 10 . 1145/3613904 . 3642144
*   [14] C. Fiani, R. Bretin, M. McGill, and M. Khamis. Big buddy: Exploring child reactions and parental perceptions towards a simulated embodied moderating system for social virtual reality. In Proc. 22nd Annual ACM Interaction Design and Children Conference, IDC ’23, pp. 1–13\. Association for Computing Machinery, New York, NY, USA, 2023\. doi: 10 . 1145/3585088 . 3589374
*   [15] C. Fiani, P. Saeghe, M. McGill, and M. Khamis. Exploring the perspectives of social vr-aware non-parent adults and parents on children’s use of social virtual reality. Proc. ACM Hum.-Comput. Interact., 8(CSCW1):25, Apr. 2024\. doi: 10 . 1145/3652867
*   [16] A.-M. Founta, C. Djouvas, D. Chatzakou, I. Leontiadis, J. Blackburn, G. Stringhini, A. Vakali, M. Sirivianos, and N. Kourtellis. Large scale crowdsourcing and characterization of twitter abusive behavior, 2018.
*   [17] M. Franco, O. Gaggi, and C. E. Palazzi. Analyzing the use of large language models for content moderation with chatgpt examples. In Proc. 3rd International Workshop on Open Challenges in Online Social Networks, OASIS ’23, pp. 1–8\. Association for Computing Machinery, New York, NY, USA, 2023\. doi: 10 . 1145/3599696 . 3612895
*   [18] G. Freeman, S. Zamanifard, D. Maloney, and D. Acena. Disturbing the peace: Experiencing and mitigating emerging harassment in social virtual reality. Proc. ACM Hum.-Comput. Interact., 6(CSCW1):30, Apr. 2022\. doi: 10 . 1145/3512932
*   [19] T. Gröndahl, L. Pajola, M. Juuti, M. Conti, and N. Asokan. All you need is ”love”: Evading hate speech detection. In Proceedings of the 11th ACM Workshop on Artificial Intelligence and Security, AISec ’18, p. 2–12\. Association for Computing Machinery, New York, NY, USA, 2018\. doi: 10 . 1145/3270101 . 3270103
*   [20] S. Guimarães, G. Kakizaki, P. Melo, M. Silva, F. Murai, J. C. S. Reis, and F. Benevenuto. Anatomy of hate speech datasets: Composition analysis and cross-dataset classification. In Proc. 34th ACM Conference on Hypertext and Social Media, HT ’23\. Association for Computing Machinery, New York, NY, USA, 2023\. doi: 10 . 1145/3603163 . 3609158
*   [21] K. Guo, A. Hu, J. Mu, Z. Shi, Z. Zhao, N. Vishwamitra, and H. Hu. An investigation of large language models for real-world hate speech detection, 2024.
*   [22] H. Hartikainen, N. Iivari, and M. Kinnula. Should we design for control, trust or involvement? a discourses survey about children’s online safety. In Proc. 15th International Conference on Interaction Design and Children, IDC ’16, p. 367–378\. Association for Computing Machinery, New York, NY, USA, 2016\. doi: 10 . 1145/2930674 . 2930680
*   [23] H. Hu, M.-X. Xu, and W. Wu. Gmm supervector based svm with spectral features for speech emotion recognition. In 2007 IEEE International Conference on Acoustics, Speech and Signal Processing - ICASSP ’07, vol. 4, pp. IV–413–IV–416, 2007\. doi: 10 . 1109/ICASSP . 2007 . 366937
*   [24] J. A. Jiang, C. Kiene, S. Middler, J. R. Brubaker, and C. Fiesler. Moderation challenges in voice-based online communities on discord. Proc. ACM Hum.-Comput. Interact., 3(CSCW), Nov. 2019\. doi: 10 . 1145/3359157
*   [25] J. A. Jiang, P. Nie, J. R. Brubaker, and C. Fiesler. A trade-off-centered framework of content moderation. ACM Trans. Comput.-Hum. Interact., 30(1), mar 2023\. doi: 10 . 1145/3534929
*   [26] A. Kadhim. An evaluation of preprocessing techniques for text classification. International Journal of Computer Science and Information Security,, 16:22–32, 06 2018.
*   [27] A. Khan and U. K. Roy. Emotion recognition using prosodie and spectral features of speech and naïve bayes classifier. In 2017 International Conference on Wireless Communications, Signal Processing and Networking (WiSPNET), pp. 1017–1021, 2017\. doi: 10 . 1109/WiSPNET . 2017 . 8299916
*   [28] M. Kolla, S. Salunkhe, E. Chandrasekharan, and K. Saha. Llm-mod: Can large language models assist content moderation? In Extended Abstracts of the 2024 CHI Conference on Human Factors in Computing Systems, CHI EA ’24\. Association for Computing Machinery, New York, NY, USA, 2024\. doi: 10 . 1145/3613905 . 3650828
*   [29] D. Kumar, Y. AbuHashem, and Z. Durumeric. Watch your language: Investigating content moderation with large language models. arXiv preprint 2309.14517, September 2024.
*   [30] V. Lai, S. Carton, R. Bhatnagar, Q. V. Liao, Y. Zhang, and C. Tan. Human-ai collaboration via conditional delegation: A case study of content moderation. In Proc. CHI Conference on Human Factors in Computing Systems, CHI ’22\. Association for Computing Machinery, New York, NY, USA, 2022\. doi: 10 . 1145/3491102 . 3501999
*   [31] L. Li, L. Fan, S. Atreja, and L. Hemphill. “hot” chatgpt: The promise of chatgpt in detecting and discriminating hateful, offensive, and toxic comments on social media. ACM Trans. Web, 18(2), mar 2024\. doi: 10 . 1145/3643829
*   [32] P. Liu, W. Yuan, J. Fu, Z. Jiang, H. Hayashi, and G. Neubig. Pre-train, prompt, and predict: A systematic survey of prompting methods in natural language processing. ACM Comput. Surv., 55(9), jan 2023.
*   [33] S. Majeed, H. HUSAIN, S. Samad, and T. Idbeaa. Mel frequency cepstral coefficients (mfcc) feature extraction enhancement in the application of speech recognition: A comparison study. Journal of Theoretical and Applied Information Technology, 79:38–56, 09 2015.
*   [34] D. Maloney, G. Freeman, and A. Robb. It is complicated: Interacting with children in social virtual reality. In Proc. VRW, pp. 343–347\. IEEE, Atlanta, GA, USA, 2020\. doi: 10 . 1109/VRW50115 . 2020 . 00075
*   [35] T. Mandl, S. Modha, A. Kumar M, and B. R. Chakravarthi. Overview of the hasoc track at fire 2020: Hate speech and offensive language identification in tamil, malayalam, hindi, english and german. In Proceedings of the 12th Annual Meeting of the Forum for Information Retrieval Evaluation, FIRE ’20, p. 29–32\. Association for Computing Machinery, New York, NY, USA, 2021\. doi: 10 . 1145/3441501 . 3441517
*   [36] T. Mandl, S. Modha, P. Majumder, D. Patel, M. Dave, C. Mandlia, and A. Patel. Overview of the hasoc track at fire 2019: Hate speech and offensive content identification in indo-european languages. In Proceedings of the 11th Annual Meeting of the Forum for Information Retrieval Evaluation, FIRE ’19, p. 14–17\. Association for Computing Machinery, New York, NY, USA, 2019\. doi: 10 . 1145/3368567 . 3368584
*   [37] A. Mehta, Y. Kunjadiya, A. Kulkarni, and M. Nagar. Exploring the viability of conversational ai for non-playable characters: A comprehensive survey. In 2021 4th International Conference on Recent Trends in Computer Science and Technology (ICRTCST), pp. 96–102, 2022\. doi: 10 . 1109/ICRTCST54752 . 2022 . 9782047
*   [38] U. Nation. What is hate speech by united nation, 2024.
*   [39] J. O’Hagan, F. Mathis, and M. McGill. User reviews as a reporting mechanism for emergent issues within social vr communities. MUM ’23, p. 236–243\. Association for Computing Machinery, New York, NY, USA, 2023\. doi: 10 . 1145/3626705 . 3627780
*   [40] J. S. Park, J. C. O’Brien, C. J. Cai, M. R. Morris, P. Liang, and M. S. Bernstein. Generative agents: Interactive simulacra of human behavior, 2023.
*   [41] J. A. Pater, M. K. Kim, E. D. Mynatt, and C. Fiesler. Characterizations of online harassment: Comparing policies across social media platforms. In Proc. GROUP, pp. 369–374\. Association for Computing Machinery, New York, NY, USA, 2016\. doi: 10 . 1145/2957276 . 2957297
*   [42] G. T. W. Patrick. The psychology of profanity. Psychological Review, 8:113–127, 1901.
*   [43] P. Paudel, M. Saeed, R. Auger, C. Wells, and G. Stringhini. Enabling contextual soft moderation on social media through contrastive textual deviation. arXiv, July 2024.
*   [44] M. Rahaman. What are the cons of content moderation? Explore the Drawbacks. [https://riseuplabs.com/what-are-the-cons-of-content-moderation/](https://riseuplabs.com/what-are-the-cons-of-content-moderation/), 2024. Accessed: 2024-09-05.
*   [45] A. Rana and S. Jha. Emotion based hate speech detection using multimodal learning. ArXiv, abs/2202.06218, 2022.
*   [46] N. Sabri, B. Chen, A. Teoh, S. P. Dow, K. Vaccaro, and M. Elsherief. Challenges of moderating social virtual reality. In Proc. CHI Conference on Human Factors in Computing Systems, CHI ’23\. Association for Computing Machinery, New York, NY, USA, 2023\. doi: 10 . 1145/3544548 . 3581329
*   [47] K. Schulenberg, L. Li, G. Freeman, S. Zamanifard, and N. J. McNeese. Towards leveraging ai-based moderation to address emergent harassment in social virtual reality. In Proc. CHI, CHI ’23, p. 17\. Association for Computing Machinery, New York, NY, 2023\. doi: 10 . 1145/3544548 . 3581090
*   [48] J. van Stegeren and J. Myśliwiec. Fine-tuning gpt-2 on annotated rpg quests for npc dialogue generation. In Proc. 16th International Conference on the Foundations of Digital Games, p. 1\. Association for Computing Machinery, Aug. 2021\. doi: 10 . 1145/3472538 . 3472595
*   [49] H. Wan, J. Zhang, A. A. Suria, B. Yao, D. Wang, Y. Coady, and M. Prpa. Building llm-based ai agents in social virtual reality. In Extended Abstracts of the 2024 CHI Conference on Human Factors in Computing Systems, CHI EA ’24\. Association for Computing Machinery, New York, NY, USA, 2024\. doi: 10 . 1145/3613905 . 3651026
*   [50] Z. Waseem and D. Hovy. Hateful symbols or hateful people? predictive features for hate speech detection on Twitter. In J. Andreas, E. Choi, and A. Lazaridou, eds., Proceedings of the NAACL Student Research Workshop, pp. 88–93\. Association for Computational Linguistics, San Diego, California, June 2016\. doi: 10 . 18653/v1/N16-2013
*   [51] Q. Zheng, S. Xu, L. Wang, Y. Tang, R. C. Salvi, G. Freeman, and Y. Huang. Understanding safety risks and safety design in social vr environments. Proc. ACM Hum.-Comput. Interact., 7(CSCW1), Apr. 2023\. doi: 10 . 1145/3579630
*   [52] Y. Zhou, Y. Sun, J. Zhang, and Y. Yan. Speech emotion recognition using both spectral and prosodic features. In 2009 International Conference on Information Engineering and Computer Science, pp. 1–4, 2009\. doi: 10 . 1109/ICIECS . 2009 . 5362730