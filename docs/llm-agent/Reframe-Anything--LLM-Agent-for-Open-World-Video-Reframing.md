<!--yml
category: 未分类
date: 2025-01-11 12:47:29
-->

# Reframe Anything: LLM Agent for Open World Video Reframing

> 来源：[https://arxiv.org/html/2403.06070/](https://arxiv.org/html/2403.06070/)

(eccv) Package eccv Warning: Package ‘hyperref’ is loaded with option ‘pagebackref’, which is *not* recommended for camera-ready version

¹¹institutetext: Opus AI Research ²²institutetext: Southeast University ³³institutetext: National University of Singapore
³³email: {gavin.cao, henry.chi, vito.zhu, lirian.su, jay.wu}@opus.pro, yongliangwu@seu.edu.cn, weiheng_chi@u.nus.eduJiawang Cao Equal contribution.11    Yongliang Wu^($\star$) 1122    Weiheng Chi^($\star$) 1133    Wenbo Zhu^($\star$) 11    Ziyue Su 11    Jay Wu 11

###### Abstract

The proliferation of mobile devices and social media has revolutionized content dissemination, with short-form video becoming increasingly prevalent. This shift has introduced the challenge of video reframing to fit various screen aspect ratios, a process that highlights the most compelling parts of a video. Traditionally, video reframing is a manual, time-consuming task requiring professional expertise, which incurs high production costs. A potential solution is to adopt some machine learning models, such as video salient object detection, to automate the process. However, these methods often lack generalizability due to their reliance on specific training data. The advent of powerful large language models (LLMs) open new avenues for AI capabilities. Building on this, we introduce Reframe Any Video Agent (RAVA), a LLM-based agent that leverages visual foundation models and human instructions to restructure visual content for video reframing. RAVA operates in three stages: perception, where it interprets user instructions and video content; planning, where it determines aspect ratios and reframing strategies; and execution, where it invokes the editing tools to produce the final video. Our experiments validate the effectiveness of RAVA in video salient object detection and real-world reframing tasks, demonstrating its potential as a tool for AI-powered video editing.

###### Keywords:

Video Reframing LLM Agent Open World

## 1 Introduction

The short-form video has emerged as a novel and swiftly expanding mode of content dissemination under the rapid evolution of social media and handheld mobile devices [[5](https://arxiv.org/html/2403.06070v1#bib.bib5)]. Traditional video aspect ratios no longer cater to the convenience of social media platform viewing due to varying screen proportions. Consequently, the challenge of reconstructing original videos to accommodate different aspect ratios has become a burgeoning demand in the field of video editing. The process often involves identifying and focusing on the most captivating or crucial elements within the current frame. Additionally, from an artistic standpoint, it sometimes necessitates zooming in on a specific area of the scene. This technique is referred to as video reframing.

Manual video reframing is a labor-intensive and time-consuming task that demands the expertise of professional editors. Creating high-quality videos typically involves skilled individuals, which in turn escalates the overall cost of production. Therefore, some machine learning researchers have started to investigate the possibility of automating video reframing. A practical strategy for this involves the application of video saliency detection [[41](https://arxiv.org/html/2403.06070v1#bib.bib41), [14](https://arxiv.org/html/2403.06070v1#bib.bib14)] which focuses on identifying the most salient, or attention-grabbing, region within a video. For example, Christel et al. [[4](https://arxiv.org/html/2403.06070v1#bib.bib4)] utilize a bottom-up visual model to generate a saliency map of the current frame and edit the scene to focus on these key areas. The aforementioned methods, while partially useful, do not always ensure that the parts they extract are complete, which can hinder their use in real-world scenarios. In an attempt to address this limitation, research in video salient object detection make significant strides. This line of work focuses on segmenting the most visually striking objects in a video frame. By doing so, it facilitates a more accurate determination of the full scope of the segmented area. However, the effectiveness of these models is often compromised by their dependence on specific domains of training data. This dependency can limit their generalizability and negatively impact their performance and interpretability in diverse applications. Furthermore, taking into account that different viewers have varying interests in specific parts of a video and that users have diverse requirements for editing, as illustrated in Figure [1](https://arxiv.org/html/2403.06070v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Reframe Anything: LLM Agent for Open World Video Reframing"), it is necessary to design a framework capable of flexibly performing video reframing according to user instructions.

![Refer to caption](img/fd87c4c3aaff7b7f9fefc0bccce0ffa2.png)

Figure 1: The overview of open world video reframing task. Even within the same video, different viewers may focus on different subjects of interest. Therefore, it is essential to implement video reframing based on user instructions to achieve specific objectives.

Recently, the development of powerful large language models (LLMs), such as ChatGPT [[25](https://arxiv.org/html/2403.06070v1#bib.bib25)] and GPT-4 [[26](https://arxiv.org/html/2403.06070v1#bib.bib26)], further propel the advancement of artificial intelligence. These models demonstrate a formidable capability for understanding and generating human language, including the ability to perceive and comprehend visual content through textual descriptions of images. For instance, they can convey the coordinates of objects within an image, grasping the essence of the visual scene, without the need for a direct vision encoder [[21](https://arxiv.org/html/2403.06070v1#bib.bib21)]. The research community is swiftly moving towards utilizing LLMs as agents that can perform complex cognitive functions, which include perception, planning, and action execution. Pioneering developments in this area include systems like TaskMatrix [[20](https://arxiv.org/html/2403.06070v1#bib.bib20)], AutoGPT [[44](https://arxiv.org/html/2403.06070v1#bib.bib44)], and MetaGPT [[11](https://arxiv.org/html/2403.06070v1#bib.bib11)]. Adding to this innovation, the advent of multimodal LLMs such as GPT-4V [[27](https://arxiv.org/html/2403.06070v1#bib.bib27)] usher in a new era where LLMs can directly sense visual content. Consequently, some works such as AppAgent [[45](https://arxiv.org/html/2403.06070v1#bib.bib45)] and MobileAgent [[40](https://arxiv.org/html/2403.06070v1#bib.bib40)] take this further by creating agents that can navigate and control any smartphone application within a mobile operating system.

Inspired by these explorations into LLMs as agents, in this paper, we introduce Reframe Any Video Agent (RAVA), a LLM-based agent designed to execute video reframing tasks flexibly based on human instructions. The overall framework can be divided into three main stages: perception, planning, and execution. In the perception phase, RAVA employs language learning to interpret user directives and video understanding to dissect scenes, identify pivotal objects, and generate textual scene descriptions. This grasp of content and context informs the subsequent planning stage, where the agent meticulously determines aspect ratios, prioritizes object importance, configures dynamic layouts, and devises a visual effect strategy that aligns with the narrative and user preferences. Finally, in the execution phase, RAVA translates the intricate plan into action, orchestrating the reframing with precision, applying effects, and arranging content according to a structured execution blueprint, all while allowing for feedback loops to refine the output. This comprehensive, three-stage process executed by RAVA ensures that videos are not only adapted to new formats but also resonate with the intended audience, amplifying the impact of content on various platforms.

Furthermore, to validate the effectiveness of RAVA, we embark on experiments from two dimensions. First, we apply it to the classic computer vision task of video salient object detection to verify its ability to accurately execute human instructions and its capability for scene comprehension. Second, we employ it in real-world video reframing tasks to determine its proficiency in completing this practically valuable task. Both quantitative results and user studies highlight the advantages of RAVA.

Our main contributions can be summarized as follows:

*   •

    We introduce RAVA, a LLM-based agent that is adept at performing video reframing tasks in accordance with human directives.

*   •

    Through a carefully crafted perception, planning, and execution framework, RAVA is able to effectively utilize the power of existing foundational models to carry out human instructions accurately.

*   •

    By conducting thorough experiments on video salient object detection and real-world video reframing cases, we validate the strengths of RAVA, showcasing its promise in the field of AI-powered video editing.

## 2 Related Work

### 2.1 Video Editing

Efforts in the field of movie analysis have made significant strides, especially in the realm of Audio-Visual Event (AVE) Localization. This particular task requires the identification and precise localization of various events within a video [[38](https://arxiv.org/html/2403.06070v1#bib.bib38), [9](https://arxiv.org/html/2403.06070v1#bib.bib9)]. Such advancements are beneficial to video editors, as they can simplify the editing workflow [[34](https://arxiv.org/html/2403.06070v1#bib.bib34)]. It’s important to note, however, that these studies do not provide a means for direct video editing. In addition to this line of research, some scholars have directly incorporated machine learning techniques into video editing. Argaw et al. introduce a benchmark suite specifically designed for video editing tasks, which includes but is not limited to, visual effects. This suite also facilitates the automatic organization of footage and provides assistance in video assembly [[3](https://arxiv.org/html/2403.06070v1#bib.bib3)]. Furthermore, Rao et al. present another benchmark aimed at selecting the best camera angle from multiple options, a crucial element in the production of television shows [[32](https://arxiv.org/html/2403.06070v1#bib.bib32)]. Despite these advancements, current methods do not address the challenge of video reframing, which involves highlighting and focusing on the most compelling segments of a video. The task of Video Salient Object Detection could potentially resolve this issue [[12](https://arxiv.org/html/2403.06070v1#bib.bib12), [46](https://arxiv.org/html/2403.06070v1#bib.bib46), [37](https://arxiv.org/html/2403.06070v1#bib.bib37)]. However, the effectiveness of these methods is hampered by their reliance on specific training datasets, which limits their generalizability in diverse real-world settings and affects their interpretability.

### 2.2 Open Vocabulary Segmentation

Open Vocabulary Segmentation aims to segment images into meaningful regions without being constrained by a predefined set of categories. This approach significantly diverges from traditional segmentation methods [[30](https://arxiv.org/html/2403.06070v1#bib.bib30), [18](https://arxiv.org/html/2403.06070v1#bib.bib18), [42](https://arxiv.org/html/2403.06070v1#bib.bib42)], which rely on a fixed vocabulary of labels, thus limiting their ability to generalize to novel or unseen objects. Seminal works like CLIP [[31](https://arxiv.org/html/2403.06070v1#bib.bib31)] and ALIGN [[13](https://arxiv.org/html/2403.06070v1#bib.bib13)] enable segmentation models to identify and categorize a diverse array of unseen objects by leveraging natural language descriptions. Building on these foundations, LSeg [[19](https://arxiv.org/html/2403.06070v1#bib.bib19)] trains an image encoder to create pixel embeddings and uses CLIP [[31](https://arxiv.org/html/2403.06070v1#bib.bib31)] text embeddings as the per-pixel classifier. To make use of cheap image-level supervision, OpenSeg [[10](https://arxiv.org/html/2403.06070v1#bib.bib10)] employs weakly-supervised grounding loss and random word dropout to foster alignment between words and image regions. Although significant progress has been made, this field still faces numerous challenges and a scarcity of training data to address them. To cope with it, SAM [[16](https://arxiv.org/html/2403.06070v1#bib.bib16)] introduces a pre-trained promptable foundation model for image segmentation, showcasing remarkable improvements in segmentation performance and adaptability. Recently, HQ-SAM [[15](https://arxiv.org/html/2403.06070v1#bib.bib15)] demonstrates an architecture that closely integrates and utilizes the existing knowledge within the SAM structure. This approach enables the production of higher-quality masks while maintaining zero-shot capabilities. Studies like MedSAM [[22](https://arxiv.org/html/2403.06070v1#bib.bib22)] also highlight the significant potential of SAM in the field of medicine.

### 2.3 LLM Agent

In recent times, we’ve seen the emergence of valuable frameworks such as AutoGPT [[44](https://arxiv.org/html/2403.06070v1#bib.bib44)], MetaGPT [[11](https://arxiv.org/html/2403.06070v1#bib.bib11)], and HuggingGPT [[35](https://arxiv.org/html/2403.06070v1#bib.bib35)], which symbolize the trend towards the swift integration of Large Language Models (LLMs) for performing intricate tasks. The development of multimodal LLMs, as referenced in works like Flamingo [[2](https://arxiv.org/html/2403.06070v1#bib.bib2)], Multimodal [[8](https://arxiv.org/html/2403.06070v1#bib.bib8)], and AudioLM [[36](https://arxiv.org/html/2403.06070v1#bib.bib36)], expand the application spectrum of LLMs by enabling them to process diverse inputs such as text, images, audio, and video. This advancement allows models to directly handle multimodal inputs, moving beyond systems like TaskMatrix [[20](https://arxiv.org/html/2403.06070v1#bib.bib20)], which depend on several base models to convert visual information into linguistic forms through image captioning or object recognition. Capitalizing on the sophisticated perceptual abilities of these models, innovative approaches such as AppAgent [[45](https://arxiv.org/html/2403.06070v1#bib.bib45)], MobileAgent [[40](https://arxiv.org/html/2403.06070v1#bib.bib40)], and VisualWebArena [[17](https://arxiv.org/html/2403.06070v1#bib.bib17)] are designed to precisely interact with mobile applications and execute web-based tasks. While there is a surge in LLM agent research, the domain of video editing is relatively untapped. LAVE [[39](https://arxiv.org/html/2403.06070v1#bib.bib39)] serves as an agent for video editing, but its functions are limited to following user-defined goals. Our proposed research, however, aims to delve deeper into the potential of LLMs to enhance automated video reframing capabilities.

## 3 Reframe Any Video Agent

![Refer to caption](img/7010d78a466563710eab848a037523e3.png)

Figure 2: The overall workflow of our proposed Reframe Any Video Agent (RAVA). RAVA is capable of receiving user dialogue inputs with a language user interface (LUI) tailored for reframing tasks, and invokes the grounding function to retrieve object information within the video, then reframes the video automatically following the request from the users.

We present Reframe Any Video Agent (RAVA) for the task of re-framing real-world videos, which also incorporates an LLM-based method supporting Language User Interface (LUI). Our method undergoes testing in an open-world setting, where scenes in videos might feature unseen objects. It is designed to robustly identify every object within a scene and discern which is of importance. Subsequently, it reframes the original video frames into varying aspect ratios, adapting to the specifications demanded by different social media applications or platforms. Furthermore, our method supports visual effects in two distinct scenarios: within individual scenes (in-scene) and between consecutive scenes (trans-scene). The entire workflow for the video reframing task can be accomplished automatically, specifically encompassing the following three key steps:

Object Grounding. Specifically, for a given original video, there will exist $M$ scenes, and each video can be represented by $\{\mathbf{S}_{1},\ldots,\mathbf{S}_{M}\}$, each scene composed of $N$ visual elements, formatted as $\{\mathbf{O}_{1},\ldots,\mathbf{O}_{N}\}$, where $\mathbf{O}_{i}$ denote the segmentation mask of ${i}_{th}$ element. Our task is to identify the most significant object(s) like $\{\mathbf{O}_{i},\ldots,\mathbf{O}_{j}\}\in\{\mathbf{O}_{1},\ldots,\mathbf{O}_% {N}\}$ within ${k}_{th}$ scene $\mathbf{S}_{k}$.

Layout Setting. In scenarios where multiple important objects are present, such as in a dialog scene, it is also necessary to determine the layout $\mathbf{L}_{k}\in\{1,2,3,\ldots,N\}$ - that is, how to simultaneously display multiple targets by sub-windows. Ultimately, for each scene, we aim to achieve a setting of $\mathbf{S}_{k}$ is $\mathbf{L}_{k}=n$, with the significant objects $\{\mathbf{O}_{i},\ldots,\mathbf{O}_{j}\}$, where $n=count\{i,\ldots,j\}$.

Effect Adding. Following this, our approach involves adding specific visual effects based on the content of the video. We introduce two types of visual effects: one that is applied within the current scene $\mathbf{S}_{k}$, such as zooming in and out, and another that is employed during scene transitions from $\mathbf{S}_{k}$ to $\mathbf{S}_{k+1}$, such as fading in and out.

Figure [2](https://arxiv.org/html/2403.06070v1#S3.F2 "Figure 2 ‣ 3 Reframe Any Video Agent ‣ Reframe Anything: LLM Agent for Open World Video Reframing") demonstrates RAVA’s workflow, three phases automate the task of reframing the video, adapting it to various aspect ratios required by different social media platforms or specifications, and enhancing viewer engagement through strategic layout decisions and visual effects.

### 3.1 Perception

For the perception phase of the agent, it can be divided into two segments: language learning and video understanding. Language learning focuses on comprehending the focal points of user interest, while video understanding centers on deciphering the content within the video frames.

Dialogue is an aspect where LLMs excel. By configuring prompts, the agent can comprehend the search objectives of the user. This process can be viewed as a structuring of LUI input information. Specifically, we initiate the process by providing RAVA with a video and user interest. This context can include both the human-generated text query and the information retrieved from tools, as detailed subsequently. We also enable the LLM to output the video topic along with structured targets, which are then incorporated into the planning phase.

Inspired by films and scripts, we have adopted the shot detection approach to understand the entire video at the scene level. The specific methodology involves the use of scenedetect¹¹1https://www.scenedetect.com/ to transform the original video into a combination of multiple scenes $\{\mathbf{S}_{1},\ldots,\mathbf{S}_{M}\}$.

Understanding the video scenes necessitates the use of tools. Firstly, the RAM[[47](https://arxiv.org/html/2403.06070v1#bib.bib47)] identifies all objects within the scenes. Subsequently, we employ SAM [[16](https://arxiv.org/html/2403.06070v1#bib.bib16)] and Grounded-SAM [[33](https://arxiv.org/html/2403.06070v1#bib.bib33)] to extract the masks of all objects and locations. CLIP[[31](https://arxiv.org/html/2403.06070v1#bib.bib31)] is utilized to acquire captions for each object. $\{\mathbf{O}_{1},\ldots,\mathbf{O}_{N}\}$, where $\mathbf{O}_{i}$ denote the ${i}_{th}$ object, which is represented by the caption, mask, and location $\{{x}_{1},{y}_{1},{x}_{2},{y}_{2}\}$.

Ultimately, the visual semantics of each video scene are transformed into a textual description. For instance, as demonstrated in Figure [2](https://arxiv.org/html/2403.06070v1#S3.F2 "Figure 2 ‣ 3 Reframe Any Video Agent ‣ Reframe Anything: LLM Agent for Open World Video Reframing"), a scene in the video is described as "Scene-1:Object-1: a boy standing in…". This methodology allows for a comprehensive understanding and reframing of video content through an LLM agent by combining language comprehension and video analysis techniques.

### 3.2 Planning

Following perception, which provides a textual description and comprehension of the video content, the planning phase is pivotal as it involves devising a comprehensive strategy for reframing the video. This strategy should accommodate different aspect ratios, highlight imperative objects, and integrate visual effects in a cohesive manner that enhances user engagement.

In this section, we detail the algorithms and methodologies employed for planning in the Reframe Any Video Agent (RAVA). With the insights gained from the perception phase, the planning phase consists of the following components:

Aspect Ratio Determination. A critical part of planning is determining the desired aspect ratios for the output videos. We consider user preferences, platform requirements, and the context of the scenes. A dynamic decision-making process chooses an optimal aspect ratio for each scene to ensure the visual content is delivered effectively.

Object Importance Hierarchy. The multiple objects identified in the perception phase need to be prioritized based on their significance relative to the context of the scene and user interest. Employing the language comprehension abilities of the LLM, we construct an importance hierarchy to aid in selection and layout decisions.

Dynamic Layout Configuration. Based on the significance and spatial arrangement of objects in a scene, we must plan for a layout that maximizes visual appeal and narrative coherence. As demonstrated in Figure [3](https://arxiv.org/html/2403.06070v1#S3.F3 "Figure 3 ‣ 3.3 Execution ‣ 3 Reframe Any Video Agent ‣ Reframe Anything: LLM Agent for Open World Video Reframing"), dynamic layout configurations consider dialogue exchanges, object interactions, and scene transitions to determine how the objects should be framed.

Visual Effect Strategy. With visual effect preferences acquired from the perception phase, a plan is formulated to apply in-scene and trans-scene effects in a way that complements the narrative flow. The challenge lies not only in deciding which effects to use but also in determining their intensity and timing to maximize impact without distracting from the core content.

Execution Blueprint. The culmination of the planning phase is an execution blueprint, which is a structured set of instructions ready to be parsed for execution. The blueprint encapsulates aspect ratios, object arrangement, effect schematics, and other relevant parameters.

Agent Feedback Loop. An optional feedback loop allows the LLM to refine the planning based on a review of preliminary reframing results. This review process involves generating a low-resolution quick preview of the reframe and running it through the LLM for appraisal against the user objectives.

The planning phase incorporates algorithms for scene understanding to create a storyboard-like sequence of actions. This storyboard guides the execution phase, ensuring a seamless transition from a conceptual model to the practical implementation of video reframing. Each scene is treated as a separate module, with transitions planned to ensure a cohesive final product that aligns with the intentions of user and maximizes viewer engagement. RAVA’s planning phase synthesizes all available data into a coherent plan, readying the system for precise execution.

### 3.3 Execution

![Refer to caption](img/53c822f33c8235dd6bd2fc56841afce3.png)

Figure 3: The video editing tools in RAVA are described as follows: The first line is about ‘Layout settings’, where ‘L’ determines the number of selected objects in the video. The second line is ‘Effect In-scene’, represents the visual effects within a scene, divided into ‘Zoom in’ and ‘Zoom out’. The third line is ‘Effect Trans-scene’, indicates the visual effects for scene transitions, divided into ‘Fade in’ and ‘Fade out’.

The agent then proceeds to carry out the reframe task according to the plan generated before. During the execution phase, regular expression matching is employed to extract structured execution steps from the plan. These structured texts correspond to specific executable functions. We generate a JSON file for each video represented as $\{\mathbf{S}_{1},\ldots,\mathbf{S}_{M}\}$ by scene detection, for the ${k}_{th}$ scene $\mathbf{S}_{k}$, there are settings as layout $\mathbf{L}_{k}\in\{1,2,3,\ldots,N\}$, objects set $\{\mathbf{O}_{i},\ldots,\mathbf{O}_{j}\}$, where $\mathbf{L}_{k}=count\{i,\ldots,j\}$, $\mathbf{E}_{in}\in\{zoom\ in,zoom\ out\}$, $\mathbf{E}_{trans}\in\{fade\ in,fade\ out\}$, This paper presents several simple implementations of special effects and argues that our framework is extensible.

For instance, if the layout is set to 2, it will choose the two objects as the main subjects of the video and arrange them vertically. If the in-scene visual effects $\mathbf{E}_{in}$ column has selected $\{zoom\ in\}$, the agent will invoke the corresponding API function to magnify the target(s). Additionally, if the trans-scene $\mathbf{E}_{trans}$ visual effects is $\{fade\ out\}$, a transition effect will also be inserted at the end of the current scene.

## 4 Experiments

To validate the effectiveness of the Reframe Any Video Agent (RAVA) we proposed, we conduct evaluations through two principal tasks. In the first task, we employ the model to address a significant challenge in the field of video understanding, namely video salient object detection. This involves segmenting the most visually prominent objects as perceived by human vision. In the second task, we apply the model to tackle the task of video reframing. This process involves adjusting the video frame to focus on the most important elements, enhancing the overall composition and storytelling aspect of the visual content.

Table 1: Our experimental results for Video Salient Object Detection task. The abbreviation ‘SD’ denotes ‘Scene Detection’.

| Method | SD | DAVIS${}_{16}$ | FBMS |
| --- | --- | --- | --- |
| $\alpha_{1}$ | $\alpha_{2}$ |  MAE | max-$F_{\beta}$ | max-$E_{m}$ | $S_{m}$ |  MAE | max-$F_{\beta}$ | max-$E_{m}$ | $S_{m}$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| UPL | 5 | 5 | .0390 | .8025 | .9183 | .8426 | .0850 | .6651 | .8513 | .7439 |
| A2S-v2 | .0663 | .4858 | .5786 | .5817 | .0851 | .6444 | .8366 | .7004 |
| Ours | .0501 | .7025 | .8219 | .7795 | .1015 | .5721 | .7532 | .6643 |
| UPL | 5 | 30 | .0367 | .8127 | .9275 | .8481 | .0844 | .6673 | .8458 | .7527 |
| A2S-v2 | .0638 | .5046 | .5929 | .5926 | .0832 | .6406 | .8288 | .7054 |
| Ours | .0419 | .6727 | .8177 | .7680 | .1148 | .5446 | .7131 | .6422 |
| UPL | 10 | 5 | .0381 | .8009 | .9210 | .8361 | .0848 | .6670 | .8615 | .7373 |
| A2S-v2 | .0640 | .4907 | .5723 | .5836 | .0900 | .6299 | .8446 | .6836 |
| Ours | .0506 | .7126 | .8256 | .7804 | .1313 | .5128 | .6776 | .6089 |

### 4.1 Video Salient Object Detection

#### 4.1.1 Datasets.

Our approach is scrutinized through the lens of two widely used datasets, DAVIS${}_{16}$ [[29](https://arxiv.org/html/2403.06070v1#bib.bib29)] and FBMS [[24](https://arxiv.org/html/2403.06070v1#bib.bib24)]. The former consists of 50 videos, amounting to 3,455 annotated frames. The latter dataset incorporates 33 supplementary video sequences, collectively characterized by 720 annotated frames.

#### 4.1.2 Metrics.

Four widely used evaluation metrics are employed to assess the performance, including Mean Absolute Error (MAE) [[28](https://arxiv.org/html/2403.06070v1#bib.bib28)], F-measure ($F_{\beta}$) [[1](https://arxiv.org/html/2403.06070v1#bib.bib1)], E-measure ($E_{m}$) [[7](https://arxiv.org/html/2403.06070v1#bib.bib7)], and S-measure ($S_{m}$) [[6](https://arxiv.org/html/2403.06070v1#bib.bib6)].

#### 4.1.3 Settings.

We composite the frames into videos at 30 frames per second. Then, The scene detection method is applied for video splitting. To minimize the possibility of a scene detection result of 0 and to reduce the number of scenes detected under each category, we choose the parameter combination of a lower threshold called $\alpha_{1}$ and a higher detection interval called $\alpha_{2}$, corresponding to parameters named threshold and min scene length. Following this phase, each scene is individually subjected to respective testing procedures to generate the salient masks. It is worth noting that each object $\mathbf{O}_{i}$ in the perception phase generates only a caption and a mask. This is because over-adjusting the frame can lead to an undesirable jittery effect, which can negatively impact the overall viewing experience.

#### 4.1.4 Results.

![Refer to caption](img/dfcdf3ffd07b5c47e78946fc7ff410c7.png)

Figure 4: The qualitative results on two video salient object detection datasets in comparison with two state-of-the-art methods. The findings indicate that RAVA copes well even in the presence of occlusions and distractions, and at times, it even surpasses the results of human annotations.

We compare RAVA with the current state-of-the-art video salient object detection methods, namely UPL [[43](https://arxiv.org/html/2403.06070v1#bib.bib43)] and A2S-v2 [[48](https://arxiv.org/html/2403.06070v1#bib.bib48)]. As can be seen in Table [1](https://arxiv.org/html/2403.06070v1#S4.T1 "Table 1 ‣ 4 Experiments ‣ Reframe Anything: LLM Agent for Open World Video Reframing"), under various settings of scene detection parameters, our approach consistently achieves competitive outcomes. It is noteworthy that our framework is not specifically designed for the task of video salient object detection. The fact that it achieves competitive results is sufficient to demonstrate the effectiveness of RAVA. To further analyze our results, we present a subset of visualized results in Figure [4](https://arxiv.org/html/2403.06070v1#S4.F4 "Figure 4 ‣ 4.1.4 Results. ‣ 4.1 Video Salient Object Detection ‣ 4 Experiments ‣ Reframe Anything: LLM Agent for Open World Video Reframing"):

(a) RAVA is capable of precisely segmenting the entirety of the Blackswan instance; however, A2S-v2 fails to provide a complete mask, and UPL erroneously segments a part of the background as if it were part of the Blackswan.

(b) RAVA maintains accurate segmentation even when objects are occluded, whereas A2S-v2, despite attempting to address occlusions, incorrectly segments parts of the occluding item; UPL, on the other hand, overlooks the occlusion altogether.

(c) When confronted with the distraction of other objects in the scene, both A2S-v2 and UPL fail to respond accurately, while RAVA can precisely identify the salient object.

(d) This example better demonstrates the robust capability of RAVA, which, through a powerful segmentation model, can perceive and achieve results that are more accurate than human annotations.

#### 4.1.5 Bad Cases.

![Refer to caption](img/5214a4aecd6504d5d0af97f6662279ff.png)

Figure 5: The cases where using RAVA might fail include situations (a) and (b), where it struggles to effectively handle composite objects and clusters of closely situated objects. However, in cases (c) and (d), despite the differences in perception of salient objects, we consider the segmentation results to be acceptable.

Even though RAVA exhibits commendable performance across most scenarios, there are still some bad cases. We believe these instances contribute to its inability to reach state-of-the-art status in the task of video salient object detection. These results are illustrated in Figure [5](https://arxiv.org/html/2403.06070v1#S4.F5 "Figure 5 ‣ 4.1.5 Bad Cases. ‣ 4.1 Video Salient Object Detection ‣ 4 Experiments ‣ Reframe Anything: LLM Agent for Open World Video Reframing"):

(a) In this case, RAVA can segment the person riding the bicycle with relative precision, but it fails to recognize that the bicycle and the person constitute a single instance, therefore not achieving an optimal result.

(b) When multiple objects close to each other that should be considered as one instance, RAVA struggles to include the other objects within the segmentation.

(c) RAVA segments the black cat in the image as the salient object, while ignoring the two birds that are farther away. We believe this result is reasonable, as perspectives on what constitutes a salient object can vary. This process still demonstrates the visual understanding of RAVA.

(d) Although the GT provided in this image depicts the person and the horse as a single entity, the fences are also prominently marked in blue, making it reasonable to consider the fences as the salient objects. This scenario is similarly observed in the mask produced by the UPL, which segments out both the fences and the horse.

#### 4.1.6 Ablation Study.

To further evaluate the effectiveness of RAVA in a single-language modality, we substituted the LLM with GPT-4 [[26](https://arxiv.org/html/2403.06070v1#bib.bib26)], which lacks multimodal perception capabilities. Apart from omitting visual inputs in the perception, we maintain all other settings unchanged.

Table.[2](https://arxiv.org/html/2403.06070v1#S4.T2 "Table 2 ‣ 4.1.6 Ablation Study. ‣ 4.1 Video Salient Object Detection ‣ 4 Experiments ‣ Reframe Anything: LLM Agent for Open World Video Reframing") reveals that even in the absence of direct visual information, GPT-4 is still capable of delivering commendable performance on two datasets based on the textual descriptions provided. This underscores the robust transferability of RAVA. However, considering the existing gap, it is necessary to use a LLM with strong visual perception capabilities.

Table 2: Our experimental results for Video Salient Object Detection task on single-modality LLM. The abbreviation ‘SD’ denotes ‘Scene Detection’.

| LLM | SD | DAVIS${}_{16}$ | FBMS |
| --- | --- | --- | --- |
| $\alpha_{1}$ | $\alpha_{2}$ |  MAE | max-$F_{\beta}$ | max-$E_{m}$ | $S_{m}$ |  MAE | max-$F_{\beta}$ | max-$E_{m}$ | $S_{m}$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-4 | 5 | 5 | .0831 | .6497 | .7885 | .7395 | .1294 | .4953 | .6943 | .6125 |
| 5 | 30 | .0548 | .6163 | .7930 | .7422 | .1642 | .4856 | .6670 | .5915 |
| 10 | 5 | .0690 | .6432 | .7913 | .7454 | .1307 | .4931 | .6684 | .6069 |

### 4.2 Video Reframing

#### 4.2.1 Settings.

To assess the video editing capabilities of RAVA in the wild, a user study is conducted with 12 participants. Edited versions of 5 videos are created using three reframe methodologies in addition to RAVA:

*   •

    Editor: This method involves a professional video editor (experience >3 years) who manually reframe the videos.

*   •

    Adobe: This method is based on the results obtained by ordinary users utilizing the reframe tool in Adobe Premiere Pro to adjust the videos, following the instructions²²2https://helpx.adobe.com/premiere-pro/using/auto-reframe.html.

*   •

    Center Cut: This method selects the center point of the video, maintaining a 9:16 aspect ratio, with the width unchanged.

To minimize the impact of the video itself and to maintain an element of unbiased evaluation by users, the open caption tool³³3https://www.opus.pro/tools/opusclip-captions is employed to add captions for each video. After watching the original video, each participant views the 4 edited versions in a random sequence. The participants review all reframed videos, and they are unaware of the editing methods employed for each video. This arrangement led to a comprehensive experimental design involving 5 (number of videos) × 12 (users) × 4 (editing strategies). Users are required to compare the reframed version of a video with the original and provide a rating on a scale from 0 to 5 for each of the attributes. These attributes were inspired by studies on video re-positioning[[23](https://arxiv.org/html/2403.06070v1#bib.bib23)], and it’s important to note that, although video reframing and video repositioning differ technically, both aim to direct the attention of the viewer to the focal scene events within given rendering constraints. Thus, some of the questions used to assess methods of video re-positioning are also applicable to video editing. Our attributes of interest include:

*   •

    Content Preservation

    *   –

        (Relevance) How well does the reframed video maintain the key elements of the original content?

    *   –

        (Completeness) Are important parts of the scene, especially the main subjects, preserved in the reframed version?

*   •

    Continuity and Consistency

    *   –

        (Continuity) Does the sequence of shots follow a logical temporal order without visually jarring jumps?

    *   –

        (Consistency) Does the reframing maintain the original style and mood of the video?

*   •

    User Experience

    *   –

        (Satisfaction) Overall, how satisfied are the users with the reframed video?

    *   –

        (Usability) Overall, are you willing to post this video on your social platform?

*   •

    Technical Quality

    *   –

        (Resolution and Clarity) Is the video crisp and clear, without degradation from cropping or zooming?

    *   –

        (Stability) Does the video maintain stability, without introducing shakiness due to reframing?

![Refer to caption](img/17474a201bf80c9c569c26e5a277bded.png)

(a) CP

![Refer to caption](img/65b430055ed0a7b8c214491e357eb226.png)

(b) CC

![Refer to caption](img/17c1ac4a001973ca5d1fb3aef12fd46f.png)

(c) UE

![Refer to caption](img/b9cbdab41076052a25d8aa2a5e332f72.png)

(d) TQ

Figure 6: Overall scores of the individual attributes: Content Preservation (CP), Continuity and Consistency (CC), User Experience (UE), and Technical Quality (TQ).

#### 4.2.2 Results.

As seen in the results presented in Figure [6](https://arxiv.org/html/2403.06070v1#S4.F6 "Figure 6 ‣ 4.2.1 Settings. ‣ 4.2 Video Reframing ‣ 4 Experiments ‣ Reframe Anything: LLM Agent for Open World Video Reframing") and outlined in our experimental data, the traditional editing method, referred to as ‘Editor’, received the highest overall mean score of $4.34$, indicating a strong ability to maintain the relevance and completeness of the original content. This could be attributed to the manual effort and expertise that video editors bring to the reframing endeavor, ensuring that significant elements are not lost. Our proposed method, RAVA, achieves an overall score of $3.81$ from four aspects, suggesting that while RAVA performs reasonably well in terms of relevancy and scene completeness, there is room for improvement when compared to professional editing. The performance of RAVA surpasses the automated ‘Adobe’ tool, with ‘Adobe’ scoring an overall score of $3.67$. This close competition hints that RAVA is on par with other available semi-automated video editing tools in terms of content preservation. The ‘Center Cut’ received the lowest mean score of $2.72$ in Content Preservation, reflecting its limited ability to identify and maintain critical video elements.

The variability in scores, as indicated by the interquartile range in the boxplot, further substantiates the need for an intelligent and context-aware re-framing technique. Future work could explore enhancing the object identification and importance determination algorithms of RAVA to further close the gap between automated and professional video editing tools.

Besides, we strongly recommend that readers view the edited video included in the supplementary materials for a more intuitive understanding.

## 5 Conclusion

We present a groundbreaking approach in the realm of video reframing by introducing Reframe Any Video Agent (RAVA), a sophisticated agent powered by large language models (LLMs) that excels in executing video reframing tasks based on human instructions. Through a meticulous three-stage process encompassing perception, planning, and execution, RAVA showcases its prowess in understanding user directives, dissecting scenes, prioritizing objects, configuring layouts, and applying visual effects—all while ensuring alignment with narrative and user preferences. Our experiments, spanning from traditional computer vision tasks to real-world video reframing scenarios, demonstrate the efficacy and promise of RAVA in the domain of AI-driven video editing. By validating the capabilities of RAVA through quantitative results and user studies, we establish its potential as a transformative tool for enhancing content creation and resonating with diverse audiences across various platforms.

#### 5.0.1 Limitations.

The main constraint of our study stems from its dependence on the underlying performance of foundation models. Utilizing a more sophisticated visual model may lead to enhanced results. Additionally, exploring how to extend the agent to perform editing across the video timeline, thereby achieving the condensation of longer content into shorter formats, represents a promising direction for development.

#### 5.0.2 Social Impact.

The research presented in this paper involves the utilization of publicly available video content sourced from YouTube. We have ensured that all video materials employed in our study adhere to the platform’s terms of service regarding non-commercial, research-based usage.

## References

*   [1] Achanta, R., Hemami, S., Estrada, F., Susstrunk, S.: Frequency-tuned salient region detection. In: 2009 IEEE conference on computer vision and pattern recognition. pp. 1597–1604\. IEEE (2009)
*   [2] Alayrac, J.B., Donahue, J., Luc, P., Miech, A., Barr, I., Hasson, Y., Lenc, K., Mensch, A., Millican, K., Reynolds, M., et al.: Flamingo: a visual language model for few-shot learning. Advances in Neural Information Processing Systems 35, 23716–23736 (2022)
*   [3] Argaw, D.M., Heilbron, F.C., Lee, J.Y., Woodson, M., Kweon, I.S.: The anatomy of video editing: a dataset and benchmark suite for ai-assisted video editing. In: European Conference on Computer Vision. pp. 201–218\. Springer (2022)
*   [4] Chamaret, C., Le Meur, O.: Attention-based video reframing: Validation using eye-tracking. In: 2008 19th International Conference on Pattern Recognition. pp. 1–4\. IEEE (2008)
*   [5] Cochrane, T.: Mobile social media as a catalyst for pedagogical change. In: EdMedia+ innovate learning. pp. 2187–2200\. Association for the Advancement of Computing in Education (AACE) (2014)
*   [6] Fan, D.P., Cheng, M.M., Liu, Y., Li, T., Borji, A.: Structure-measure: A new way to evaluate foreground maps. In: Proceedings of the IEEE international conference on computer vision. pp. 4548–4557 (2017)
*   [7] Fan, D.P., Gong, C., Cao, Y., Ren, B., Cheng, M.M., Borji, A.: Enhanced-alignment measure for binary foreground map evaluation. In: Proceedings of the Twenty-Seventh International Joint Conference on Artificial Intelligence. International Joint Conferences on Artificial Intelligence Organization (2018)
*   [8] Furuta, H., Nachum, O., Lee, K.H., Matsuo, Y., Gu, S.S., Gur, I.: Multimodal web navigation with instruction-finetuned foundation models. arXiv preprint arXiv:2305.11854 (2023)
*   [9] Geng, T., Wang, T., Duan, J., Cong, R., Zheng, F.: Dense-localizing audio-visual events in untrimmed videos: A large-scale benchmark and baseline. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. pp. 22942–22951 (2023)
*   [10] Ghiasi, G., Gu, X., Cui, Y., Lin, T.Y.: Scaling open-vocabulary image segmentation with image-level labels. In: European Conference on Computer Vision. pp. 540–557\. Springer (2022)
*   [11] Hong, S., Zheng, X., Chen, J., Cheng, Y., Wang, J., Zhang, C., Wang, Z., Yau, S.K.S., Lin, Z., Zhou, L., et al.: Metagpt: Meta programming for multi-agent collaborative framework. arXiv preprint arXiv:2308.00352 (2023)
*   [12] Hu, F., Palazzo, S., Salanitri, F.P., Bellitto, G., Moradi, M., Spampinato, C., McGuinness, K.: Tinyhd: Efficient video saliency prediction with heterogeneous decoders using hierarchical maps distillation. In: Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision. pp. 2051–2060 (2023)
*   [13] Jia, C., Yang, Y., Xia, Y., Chen, Y.T., Parekh, Z., Pham, H., Le, Q., Sung, Y.H., Li, Z., Duerig, T.: Scaling up visual and vision-language representation learning with noisy text supervision. In: International conference on machine learning. pp. 4904–4916\. PMLR (2021)
*   [14] Jiang, L., Xu, M., Liu, T., Qiao, M., Wang, Z.: Deepvs: A deep learning based video saliency prediction approach. In: Proceedings of the european conference on computer vision (eccv). pp. 602–617 (2018)
*   [15] Ke, L., Ye, M., Danelljan, M., Tai, Y.W., Tang, C.K., Yu, F., et al.: Segment anything in high quality. Advances in Neural Information Processing Systems 36 (2024)
*   [16] Kirillov, A., Mintun, E., Ravi, N., Mao, H., Rolland, C., Gustafson, L., Xiao, T., Whitehead, S., Berg, A.C., Lo, W.Y., Dollár, P., Girshick, R.: Segment anything (2023)
*   [17] Koh, J.Y., Lo, R., Jang, L., Duvvur, V., Lim, M.C., Huang, P.Y., Neubig, G., Zhou, S., Salakhutdinov, R., Fried, D.: Visualwebarena: Evaluating multimodal agents on realistic visual web tasks. arXiv preprint arXiv:2401.13649 (2024)
*   [18] Lambert, J., Liu, Z., Sener, O., Hays, J., Koltun, V.: Mseg: A composite dataset for multi-domain semantic segmentation. In: Proceedings of the IEEE/CVF conference on computer vision and pattern recognition. pp. 2879–2888 (2020)
*   [19] Li, B., Weinberger, K.Q., Belongie, S., Koltun, V., Ranftl, R.: Language-driven semantic segmentation (2022)
*   [20] Liang, Y., Wu, C., Song, T., Wu, W., Xia, Y., Liu, Y., Ou, Y., Lu, S., Ji, L., Mao, S., et al.: Taskmatrix. ai: Completing tasks by connecting foundation models with millions of apis. arXiv preprint arXiv:2303.16434 (2023)
*   [21] Liu, H., Li, C., Wu, Q., Lee, Y.J.: Visual instruction tuning. Advances in neural information processing systems 36 (2024)
*   [22] Ma, J., He, Y., Li, F., Han, L., You, C., Wang, B.: Segment anything in medical images. Nature Communications 15(1),  654 (2024)
*   [23] Moorthy, K.B., Kumar, M., Subramanian, R., Gandhi, V.: Gazed–gaze-guided cinematic editing of wide-angle monocular video recordings. In: Proceedings of the 2020 CHI Conference on Human Factors in Computing Systems. pp. 1–11 (2020)
*   [24] Ochs, P., Malik, J., Brox, T.: Segmentation of moving objects by long term video analysis. IEEE transactions on pattern analysis and machine intelligence 36(6), 1187–1200 (2013)
*   [25] OpenAI: Chatgpt. [https://openai.com/research/chatgpt](https://openai.com/research/chatgpt) (2021)
*   [26] OpenAI: Gpt-4\. [https://openai.com/research/gpt-4](https://openai.com/research/gpt-4) (2023)
*   [27] OpenAI: Gpt-4v. [https://openai.com/research/gpt-4v-system-card](https://openai.com/research/gpt-4v-system-card) (2023)
*   [28] Perazzi, F., Krähenbühl, P., Pritch, Y., Hornung, A.: Saliency filters: Contrast based filtering for salient region detection. In: 2012 IEEE conference on computer vision and pattern recognition. pp. 733–740\. IEEE (2012)
*   [29] Perazzi, F., Pont-Tuset, J., McWilliams, B., Van Gool, L., Gross, M., Sorkine-Hornung, A.: A benchmark dataset and evaluation methodology for video object segmentation. In: Proceedings of the IEEE conference on computer vision and pattern recognition. pp. 724–732 (2016)
*   [30] Pinheiro, P.O., Collobert, R.: From image-level to pixel-level labeling with convolutional networks. In: Proceedings of the IEEE conference on computer vision and pattern recognition. pp. 1713–1721 (2015)
*   [31] Radford, A., Kim, J.W., Hallacy, C., Ramesh, A., Goh, G., Agarwal, S., Sastry, G., Askell, A., Mishkin, P., Clark, J., Krueger, G., Sutskever, I.: Learning transferable visual models from natural language supervision (2021)
*   [32] Rao, A., Jiang, X., Wang, S., Guo, Y., Liu, Z., Dai, B., Pang, L., Wu, X., Lin, D., Jin, L.: Temporal and contextual transformer for multi-camera editing of tv shows. arXiv preprint arXiv:2210.08737 (2022)
*   [33] Ren, T., Liu, S., Zeng, A., Lin, J., Li, K., Cao, H., Chen, J., Huang, X., Chen, Y., Yan, F., et al.: Grounded sam: Assembling open-world models for diverse visual tasks. arXiv preprint arXiv:2401.14159 (2024)
*   [34] Serrano, A., Sitzmann, V., Ruiz-Borau, J., Wetzstein, G., Gutierrez, D., Masia, B.: Movie editing and cognitive event segmentation in virtual reality video. ACM Transactions on Graphics (TOG) 36(4), 1–12 (2017)
*   [35] Shen, Y., Song, K., Tan, X., Li, D., Lu, W., Zhuang, Y.: Hugginggpt: Solving ai tasks with chatgpt and its friends in hugging face. Advances in Neural Information Processing Systems 36 (2024)
*   [36] Shu, F., Zhang, L., Jiang, H., Xie, C.: Audio-visual llm for video understanding. arXiv preprint arXiv:2312.06720 (2023)
*   [37] Su, Y., Deng, J., Sun, R., Lin, G., Su, H., Wu, Q.: A unified transformer framework for group-based segmentation: Co-segmentation, co-saliency detection and video salient object detection. IEEE Transactions on Multimedia (2023)
*   [38] Tian, Y., Shi, J., Li, B., Duan, Z., Xu, C.: Audio-visual event localization in unconstrained videos. In: Proceedings of the European conference on computer vision (ECCV). pp. 247–263 (2018)
*   [39] Wang, B., Li, Y., Lv, Z., Xia, H., Xu, Y., Sodhi, R.: Lave: Llm-powered agent assistance and language augmentation for video editing. arXiv preprint arXiv:2402.10294 (2024)
*   [40] Wang, J., Xu, H., Ye, J., Yan, M., Shen, W., Zhang, J., Huang, F., Sang, J.: Mobile-agent: Autonomous multi-modal mobile device agent with visual perception. arXiv preprint arXiv:2401.16158 (2024)
*   [41] Wang, W., Shen, J., Xie, J., Cheng, M.M., Ling, H., Borji, A.: Revisiting video saliency prediction in the deep learning era. IEEE transactions on pattern analysis and machine intelligence 43(1), 220–237 (2019)
*   [42] Xian, Y., Choudhury, S., He, Y., Schiele, B., Akata, Z.: Semantic projection network for zero-and few-label semantic segmentation. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. pp. 8256–8265 (2019)
*   [43] Yan, P., Wu, Z., Liu, M., Zeng, K., Lin, L., Li, G.: Unsupervised domain adaptive salient object detection through uncertainty-aware pseudo-label learning. In: Proceedings of the AAAI Conference on Artificial Intelligence. vol. 36, pp. 3000–3008 (2022)
*   [44] Yang, H., Yue, S., He, Y.: Auto-gpt for online decision making: Benchmarks and additional opinions. arXiv preprint arXiv:2306.02224 (2023)
*   [45] Yang, Z., Liu, J., Han, Y., Chen, X., Huang, Z., Fu, B., Yu, G.: Appagent: Multimodal agents as smartphone users. arXiv preprint arXiv:2312.13771 (2023)
*   [46] Yuan, Y., Wang, Y., Wang, L., Zhao, X., Lu, H., Wang, Y., Su, W., Zhang, L.: Isomer: Isomerous transformer for zero-shot video object segmentation. In: Proceedings of the IEEE/CVF International Conference on Computer Vision. pp. 966–976 (2023)
*   [47] Zhang, Y., Huang, X., Ma, J., Li, Z., Luo, Z., Xie, Y., Qin, Y., Luo, T., Li, Y., Liu, S., et al.: Recognize anything: A strong image tagging model. arXiv preprint arXiv:2306.03514 (2023)
*   [48] Zhou, H., Qiao, B., Yang, L., Lai, J., Xie, X.: Texture-guided saliency distilling for unsupervised salient object detection. In: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. pp. 7257–7267 (2023)