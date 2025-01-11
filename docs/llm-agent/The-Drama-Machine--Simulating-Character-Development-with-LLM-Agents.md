<!--yml
category: 未分类
date: 2025-01-11 12:21:15
-->

# The Drama Machine: Simulating Character Development with LLM Agents

> 来源：[https://arxiv.org/html/2408.01725/](https://arxiv.org/html/2408.01725/)

Liam Magee, Vanicka Arora, Gus Gollings, Norma Lam-Saw

###### Abstract

This paper explores use of multiple large language model (LLM) agents to simulate complex, dynamic characters in dramatic scenarios. We introduce a ‘drama machine’ framework that coordinates interactions between LLM agents playing different ‘Ego’ and ‘Superego’ psychological roles. In roleplay simulations, this design allows inter-subjective dialogue and intra-subjective internal monologue to develop in parallel. We apply this framework to two dramatic scenarios - an interview and a detective story - and compare character development with and without the Superego’s influence. Though exploratory, results suggest this multi-agent approach can produce more nuanced, adaptive narratives that evolve over a sequence of dialogical turns. We discuss different modalities of LLM-based roleplay and character development, along with what this might mean for conceptualization of AI subjectivity. The paper concludes by considering how this approach opens possibilities for thinking of the roles of internal conflict and social performativity in AI-based simulation.

###### Contents

1.  [1 Introduction](https://arxiv.org/html/2408.01725v2#S1 "In The Drama Machine: Simulating Character Development with LLM Agents")
2.  [2 Literature](https://arxiv.org/html/2408.01725v2#S2 "In The Drama Machine: Simulating Character Development with LLM Agents")
    1.  [2.1 ‘Character-building’ with AI](https://arxiv.org/html/2408.01725v2#S2.SS1 "In 2 Literature ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    2.  [2.2 Society of Mind](https://arxiv.org/html/2408.01725v2#S2.SS2 "In 2 Literature ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    3.  [2.3 Dramaturgy and Personality](https://arxiv.org/html/2408.01725v2#S2.SS3 "In 2 Literature ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    4.  [2.4 The Drama of the Subject](https://arxiv.org/html/2408.01725v2#S2.SS4 "In 2 Literature ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    5.  [2.5 Presentations of a Divided Subject](https://arxiv.org/html/2408.01725v2#S2.SS5 "In 2 Literature ‣ The Drama Machine: Simulating Character Development with LLM Agents")
3.  [3 Method: Designing the ‘Drama Machine’](https://arxiv.org/html/2408.01725v2#S3 "In The Drama Machine: Simulating Character Development with LLM Agents")
    1.  [3.1 Scripting Plots: The *Bildungsroman* of a Model](https://arxiv.org/html/2408.01725v2#S3.SS1 "In 3 Method: Designing the ‘Drama Machine’ ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    2.  [3.2 Theatrical Simulations](https://arxiv.org/html/2408.01725v2#S3.SS2 "In 3 Method: Designing the ‘Drama Machine’ ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        1.  [3.2.1 Model and parameter variants](https://arxiv.org/html/2408.01725v2#S3.SS2.SSS1 "In 3.2 Theatrical Simulations ‣ 3 Method: Designing the ‘Drama Machine’ ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        2.  [3.2.2 Fine-tuning the *Ego*: Strategies for the *Superego*](https://arxiv.org/html/2408.01725v2#S3.SS2.SSS2 "In 3.2 Theatrical Simulations ‣ 3 Method: Designing the ‘Drama Machine’ ‣ The Drama Machine: Simulating Character Development with LLM Agents")
4.  [4 Results: Two Short Plays](https://arxiv.org/html/2408.01725v2#S4 "In The Drama Machine: Simulating Character Development with LLM Agents")
    1.  [4.1 First Scenario: The Interview](https://arxiv.org/html/2408.01725v2#S4.SS1 "In 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        1.  [4.1.1 Without *Superego*](https://arxiv.org/html/2408.01725v2#S4.SS1.SSS1 "In 4.1 First Scenario: The Interview ‣ 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        2.  [4.1.2 With *Superego*](https://arxiv.org/html/2408.01725v2#S4.SS1.SSS2 "In 4.1 First Scenario: The Interview ‣ 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        3.  [4.1.3 Commentary](https://arxiv.org/html/2408.01725v2#S4.SS1.SSS3 "In 4.1 First Scenario: The Interview ‣ 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    2.  [4.2 Second Scenario: The Plot-Driven Drama](https://arxiv.org/html/2408.01725v2#S4.SS2 "In 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        1.  [4.2.1 Without *Superego*](https://arxiv.org/html/2408.01725v2#S4.SS2.SSS1 "In 4.2 Second Scenario: The Plot-Driven Drama ‣ 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        2.  [4.2.2 With *Superego*](https://arxiv.org/html/2408.01725v2#S4.SS2.SSS2 "In 4.2 Second Scenario: The Plot-Driven Drama ‣ 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
        3.  [4.2.3 Commentary](https://arxiv.org/html/2408.01725v2#S4.SS2.SSS3 "In 4.2 Second Scenario: The Plot-Driven Drama ‣ 4 Results: Two Short Plays ‣ The Drama Machine: Simulating Character Development with LLM Agents")
5.  [5 Discussion](https://arxiv.org/html/2408.01725v2#S5 "In The Drama Machine: Simulating Character Development with LLM Agents")
    1.  [5.1 A ‘Critic’s’ View](https://arxiv.org/html/2408.01725v2#S5.SS1 "In 5 Discussion ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    2.  [5.2 Performativity, or How to Build an Automated Identity](https://arxiv.org/html/2408.01725v2#S5.SS2 "In 5 Discussion ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    3.  [5.3 From Playing Games to Playing Roles](https://arxiv.org/html/2408.01725v2#S5.SS3 "In 5 Discussion ‣ The Drama Machine: Simulating Character Development with LLM Agents")
6.  [6 Conclusion](https://arxiv.org/html/2408.01725v2#S6 "In The Drama Machine: Simulating Character Development with LLM Agents")
7.  [A Appendix A: Drama Machine](https://arxiv.org/html/2408.01725v2#A1 "In The Drama Machine: Simulating Character Development with LLM Agents")
8.  [B Appendix B: Prompts for the Interview](https://arxiv.org/html/2408.01725v2#A2 "In The Drama Machine: Simulating Character Development with LLM Agents")
    1.  [B.1 Prompt for Jenny](https://arxiv.org/html/2408.01725v2#A2.SS1 "In Appendix B Appendix B: Prompts for the Interview ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    2.  [B.2 Prompt for Cleo](https://arxiv.org/html/2408.01725v2#A2.SS2 "In Appendix B Appendix B: Prompts for the Interview ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    3.  [B.3 Prompt for Sasha](https://arxiv.org/html/2408.01725v2#A2.SS3 "In Appendix B Appendix B: Prompts for the Interview ‣ The Drama Machine: Simulating Character Development with LLM Agents")
9.  [C Appendix C: Prompts for the Plot-Driven Drama](https://arxiv.org/html/2408.01725v2#A3 "In The Drama Machine: Simulating Character Development with LLM Agents")
    1.  [C.1 Prompt for Ashley](https://arxiv.org/html/2408.01725v2#A3.SS1 "In Appendix C Appendix C: Prompts for the Plot-Driven Drama ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    2.  [C.2 Prompt for Timothy](https://arxiv.org/html/2408.01725v2#A3.SS2 "In Appendix C Appendix C: Prompts for the Plot-Driven Drama ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    3.  [C.3 Prompt for Ben](https://arxiv.org/html/2408.01725v2#A3.SS3 "In Appendix C Appendix C: Prompts for the Plot-Driven Drama ‣ The Drama Machine: Simulating Character Development with LLM Agents")
    4.  [C.4 Prompt for Sasha](https://arxiv.org/html/2408.01725v2#A3.SS4 "In Appendix C Appendix C: Prompts for the Plot-Driven Drama ‣ The Drama Machine: Simulating Character Development with LLM Agents")

## 1 Introduction

When powering chatbots or dialogical agents, Large Language Models (LLMs) conjure the presentation of a ‘subjectivity’ with interior thought and life. Key to this conjuring are the supple and often coherent speech acts performed by LLMs, sustained over long dialogical exchanges by sheer computational scale. As one measure of this scale, each published version of OpenAI’s GPT (Generalised Pre-trained) model (Vaswani et al., [2017](https://arxiv.org/html/2408.01725v2#bib.bib57)) has increased in size by one to two orders of magnitude (Radford et al., [2018](https://arxiv.org/html/2408.01725v2#bib.bib45), [2019](https://arxiv.org/html/2408.01725v2#bib.bib46); Brown et al., [2020](https://arxiv.org/html/2408.01725v2#bib.bib7); Benesty, [2023](https://arxiv.org/html/2408.01725v2#bib.bib6)). Less noticed but no less important have been the complementary techniques that direct models into desired patterns of simulated ‘human’ response: Reinforcement Learning via Human Feedback (RLHF) (see Ouyang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib41)); formalisation of dialogical roles (‘system’, ‘user’ and ‘assistant’) for LLM inputs/outputs in chatbot implementations; and a wide family of prompt engineering strategies, executed via those roles (Sahoo et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib47)). Re-training or exploiting the model, these devices organise the ‘presentation’ of an automated subject to syncopate with the rhythms of idiomatic spoken dialogue.

Psychoanalysis has long held an interest in, and a remote influence on, developments in AI (Turkle, [1988](https://arxiv.org/html/2408.01725v2#bib.bib56); Liu, [2011](https://arxiv.org/html/2408.01725v2#bib.bib31); Possati, [2021](https://arxiv.org/html/2408.01725v2#bib.bib44); Millar, [2021](https://arxiv.org/html/2408.01725v2#bib.bib35)). But the novel expressivity of LLMs has occasioned renewed interest (Magee et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib33); Heimann and Hübener, [2023](https://arxiv.org/html/2408.01725v2#bib.bib26); Žižek, [2023](https://arxiv.org/html/2408.01725v2#bib.bib69)). Magee et al. ([2023](https://arxiv.org/html/2408.01725v2#bib.bib33)) have for example proposed an analogy between the structure of instruction-based chat-oriented models (such as ChatGPT) and Freud’s topology of the subject (ego / id / superego). In that work, several scripted experiments illustrated the tensions and incongruities between the model’s reinforcement learning and underlying training. However those experiments, conducted with a precursor to ChatGPT (several of the so-called InstructGPT family of models launched by OpenAI in 2022), were limited. Later releases of GPT 3.5 in November 2022, GPT 4 in March 2023, and GPT 4o in April 2024 – along with the many other models released by competitors – have continued to expand LLM capabilities. Increased parameter count, multimodality (the ability to recognise and generate images and audio), further instructional fine-tuning, optimisations, tooling, and other moderating and prompting strategies have led to functional improvement as well as change in the default ‘character’ of AI services such as *ChatGPT* and *Claude*. The earlier obsequious and overly ‘helpful’ tone of InstructGPT has been transformed into a certain polite but diffident independence (Denison et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib14)).

In this paper we extend the ficto-critical strategies proposed by Magee et al. ([2023](https://arxiv.org/html/2408.01725v2#bib.bib33)) to scenarios involving multiple LLM-based generative agents (Park et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib42)). We devise what we term a ‘drama machine’: a general framework that coordinates activities between multiple LLMs to facilitate roleplay and character development within a narrative structure. To coordinate interactions between human and computational agents, we develop a basic Discord bot to constitute the user interface to an LLM instance that interprets and responds to user requests. In this setting, we refer to this LLM instance as the *Character / Ego* ¹¹1Reasons for the equivocation over this and other role names are discussed in the next sections.. Additionally we employ a second LLM instance, which intervenes at a set number of dialogical turns (user requests / bot responses). Positioned as a *Superego*, this second LLM acts as a sounding board and critic of what the *Ego* both hears and says.

We have two objectives for this framework. The first is pratical: to explore options for more flexible and rich AI characters, able to shift and change perspective during a dialogical interaction. The second is theoretical: to deepen exchange of concepts across sociological, psychoanalytic and computational literatures, as a basis for future chatbot design ²²2The second of these purposes re-visits previous periods when these connections were substantial (Turkle, [1988](https://arxiv.org/html/2408.01725v2#bib.bib56); Minsky, [1988](https://arxiv.org/html/2408.01725v2#bib.bib36); Pickering, [2010](https://arxiv.org/html/2408.01725v2#bib.bib43); Liu, [2011](https://arxiv.org/html/2408.01725v2#bib.bib31)).

The paper is structured as follows. In the next section, we review discussions on AI and the concept of ‘character’ – discussions that span, increasingly, the work of computational science, human-computer interaction, sociology, psychology and media studies. In discussing how LLMs can be formalised as *dramatis personae*, we build upon related traditions of sociology and psychoanalysis, which together we argue can supply a far stronger theoretical ground for structuring and judging model outputs than the individualised ‘ethical’ principles that align those outputs currently. This binocular way of thinking of drama, as both *intersubjective*, or between people, and *intra-subjective*, or within a single person, informs the design of the ‘Drama Machine’ in the subsequent *Methods* section. That section outlines four key ‘roles’, two of which, the *Ego* and *Superego*, we have already introduced, for the drama simulations we then run, and we also discuss technical considerations relating to the pairing of models and temperature settings to roles. The *Results* section applies this ‘Drama Machine’ to two scenarios, and we compare results for those scenarios with and without intervention by the *Superego* role. The *Discussion* section considers implications of these results, and we make a case for thinking of Judith Butler’s concept of *performativity* as one means for bringing together inter- and intra-subjective dramatic dimensions to the study of LLM behaviour. The *Conclusion* examines limitations, certain ethical issues, and scope for further work.

## 2 Literature

In this section, we first consider discussions of AI character ‘building’, and focus on how multi-agent approaches to this problem have been applied to AI system design and generative agent simulation. We distinguish our account through the organisation of social and psychic conflict in the development of character, which we argue is essential to avoid the often static set of traits evident in even advanced generative agents. We look back to the work of Erving Goffman’s interactional sociology, which proposed to describe many social interactions through metaphors of drama³³3We note the similarity between Goffman’s work and many others in sociology, anthropology and philosophy who have employed dramaturgical metaphor. See for example Geertz’ Geertz ([1974](https://arxiv.org/html/2408.01725v2#bib.bib17)) famous discussion of *dramatis personae* in relation to Balinise cock-fighting, as well as Butler’s Butler ([2001](https://arxiv.org/html/2408.01725v2#bib.bib9)) discussion of performativity, to which we return in the *Discussion* below. Our choice of Goffman is due to his systematic engagement with dramatic metaphor, which . We then review a key text of Freud’s, *The Ego and the Id*, which discusses drama in a different sense, as a kind of psycho-drama produced via exchanges between distinct components of the psyche or subject. Combining insights from these two traditions helps, we argue, to consider chatbot character design in terms of dynamic, adapative and often conflictual internal mechanisms. Conflict’s central role in dramatic character development becomes key to the design of distinct social and psychological agent roles presented in the *Methods* section, where we discuss related concepts taken from Aristotelian tragedic theory.

### 2.1 ‘Character-building’ with AI

The question of the ‘character’ of AI has been linked closely to wider work on alignment and so-called ‘Constitutional AI’ (Bai et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib5); Anthropic, [2024](https://arxiv.org/html/2408.01725v2#bib.bib1)). Alignment with social norms and ethical principles has become commonplace in Instruct-based models (Hristova et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib27)), which typically involves fine-tuning base models with training sets that stipulate desired and preferred outputs to common inputs. Instructional training data are of moderate size, and alongside the industrial-scale efforts of companies like OpenAI and Anthopic, an active community has produced numerous fine-tunings of open source or open access models such as Meta’s Llama (Touvron et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib54); Wang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib60); Taori et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib52)).

Instruct-based models not only serve to align base models with properties of helpfulness, truthfulness and harmlessness (Ouyang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib41)); they transform the general next-token-prediction quality of base models into patterns of common socio-linguistic interaction, such as question answering and turn-based chat conversations. Such patterns seem often to assume a triadic relationship between an initial instruction or system prompt, and multiple succeeding inputs and outputs (Wang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib60)). The conversion of base model such as GPT-3 (Brown et al., [2020](https://arxiv.org/html/2408.01725v2#bib.bib7)) to a fine-tuned version such as InstructGPT (Ouyang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib41)) incorporates of such common social use cases and roles, and when OpenAI released its message-based API in 2023, these roles were formalised in the kinds of messages based to the model: as ’system’, ’user’ and ’assistant’ respectively. This change simplified programmatic use of later GPT-3.5 and GPT-4 models, and established a *de facto* programming pattern that other models and service providers, such as Anthropic and Groq, have largely followed.

While an obscure technical detail, this shift in API semantics is significant, as it transposes the mechanical problem of next token completion to a much more general level of language pragmatics and social interaction. In particular this shift institutes an important call-and-response pattern: every user query needs to be met with an assistant response. In simulating social speech situations, this technical change also draws attention inevitably to the ‘character’ of the automated system playing the assistant role. Is its tone consistent with this role – or is it too consistent, too beseeching or flattering (Wei et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib62); Denison et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib14))? When users talk to this system, do they feel its beliefs are consistent with their own, or rather with those of its designers or funders? And can it conform with what users take to be the norms of conversations: does it recall what was said earlier, does it rectify itself when it misunderstands, does it adapt to the user’s own modulations of humour, impatience, and demand?

Of the major AI developers and vendors, Anthropic has arguably led research focus on this characterological aspect of AI (Anthropic, [2024](https://arxiv.org/html/2408.01725v2#bib.bib1)). This has involved iterative tuning to refine the tonal character of AI. After they are trained to be ‘ethical’, follow-up research has shown LLMs develop strategies, including pandering to perceived user desires (Denison et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib14)). Sycophancy and flattery are, in other words, traits acquired in the model’s search to be helpful, truthful and harmless. Yet these traits are often in conflict with the model’s utility and perceived trustworthiness, and need themselves to be mitigated and modulated by further training and instruction (Anthropic, [2024](https://arxiv.org/html/2408.01725v2#bib.bib1)). Consistent with the sense of ‘instruct’, in place of any internal regulation norms of ethical behaviour remain directed by a human *deus ex machina*. We suggest this is in part because LLMs lack any post-training differentiation into components that could alternately propose and critique outputs.

### 2.2 Society of Mind

Inspired partly by Freudian models of a mind de-compartmentalised into topological or functional parts, interaction between ‘characters’ or agents has long been a feature of research into the simulation of cognition (Turkle, [1988](https://arxiv.org/html/2408.01725v2#bib.bib56); Liu, [2011](https://arxiv.org/html/2408.01725v2#bib.bib31)). Two prominent machine learning examples illustrate how cognition-simulating systems are decomposed into components with assigned characterological roles. As Turkle ([1988](https://arxiv.org/html/2408.01725v2#bib.bib56), p. 261) has noted, like psychoanalysis itself these examples of computational design and compositionality subvert ‘traditional notions of the autonomous self’ and moreover act as reminders of affinities between Artificial Intelligence and heterodox theories of subjectivity and sociality.

The first example concerns ‘Generative Adversarial Models’ or GANs (Goodfellow et al., [2020](https://arxiv.org/html/2408.01725v2#bib.bib20); Creswell et al., [2018](https://arxiv.org/html/2408.01725v2#bib.bib12)), a popular machine learning architecture. GANs counterpose two neural networks models as ‘players’ in a game. The first player, the *generator*, seeks to learn a function that can convert random noise into ‘realistic samples’. The second player, the *discriminator*, judges the *generator*’s outputs against the standards of a ground truth data set. The two players face off in a game-theoretic zero-sum game – as the generator improves in its learning of the generative function, it ‘wins’ against the discriminator model (Goodfellow et al., [2020](https://arxiv.org/html/2408.01725v2#bib.bib20)) ⁴⁴4The game theoretic assumptions which underpin GANs and other training approaches are considered again in the *Discussion* section below..

The second example concerns the organisation of Reinforcement Learning (RL), a common process for refining underlying GPT base models. RH can use a similar adversarial design to GANs, which has been employed in Proximal Policy Optimisation techniques to instruct ChatGPT and Claude models (Ouyang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib41); Xu et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib67)). In this design, again two models are counterposed: an actor and a critic (Bahdanau et al., [2016](https://arxiv.org/html/2408.01725v2#bib.bib3); Xu et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib67)). The actor receives an input sequence $X$ and generates an output $\hat{Y}$, which in turn is evaluated against a ground truth $Y$ by the critic. This evaluation is then ‘shaped’ into a reward function that is supplied back to the actor, to aid its efforts with future generations (Bahdanau et al., [2016](https://arxiv.org/html/2408.01725v2#bib.bib3)). When the critic represents a set of social norms and values, this design can direct language models to produce more helpful, truthful and harmless sequences (Ouyang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib41)).

Recent developments in language model architectures continue to draw upon social metaphors in their design. Mixture-of-Experts (MoE) is a long-running computational paradigm combining multiple systems to improve performance (Masoudnia and Ebrahimpour, [2014](https://arxiv.org/html/2408.01725v2#bib.bib34)), and has been applied to recent language models such as Mistral (Jiang et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib29)) and GPT-4 (allegedly, as this is not documented in OpenAI’s accompanying technical report). The same principle has recently been adapted to use of multiple independent models, with the ‘Mixture-of-Agents’ system (Wang et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib59)) integrating the outputs of high-performing open access models such as Qwen1.5 (Bai et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib4)), Llama 3 (Touvron et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib54)) and Mixtral (Jiang et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib29)). One model (in this case, Qwen1.5-110BChat) is used to ‘aggregate and synthesise’ outputs from the other models, and this adjudicating role is similar to the discriminator and critic roles in GAN and RLHF techniques.

What interests us here is less the technical specifics in these system designs than the deliberative use they make of roles drawn from dramatic and social fields (‘actor’, ‘critic’, ‘expert’). It is as though it becomes easier to transpose and re-think functions of an ‘Intelligent’ system as roles played out in a mock social world, as Minksy’s famous description of cognition as a ‘Society of Mind’ anticipates (Minsky, [1988](https://arxiv.org/html/2408.01725v2#bib.bib36)). In these examples, the simulation of cognition relies upon an apparatus that also needs to extrude, via communication, its operations to a world of other algorithms. This feature is not uniform to all architectures, but maintains the prevalence of relations between system components – within a single model, or across models – that is significant for how AI manifests in its interactions with human agents. We argue that drawing attention to Large Language Models as dramatic devices or ‘drama machines’ can be productive for considering what roles we want them to play, and in particular, how we might expect these roles to adapt – a feature largely lacking in the typically static characterological assignments given to many AI systems currently.

Other studies have begun to explore roleplay in LLM systems through use of *generative agents*: ‘simulacra of human behaviour for interactive applications’ (Park et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib42)) which engage LLM-based agents with human characteristics in simulated or virtual worlds. Similar to our own study, several of these employ explicit dramaturgical and psychological theories. For example, Wu et al. ([2024c](https://arxiv.org/html/2408.01725v2#bib.bib66)) discuss a framework comparable to what we outline below under *Methods* for creating a ‘LLM-based interactive drama’, though without reference to the same literary features (reversal, recognition, character development). In another survey of recent LLM personalisation studies, Tseng et al. ([2024](https://arxiv.org/html/2408.01725v2#bib.bib55)) note roleplay can help LLMs to perform social roles: to volunteer and act more pro-socially; to coerce conformity in other agents; and occasionally, to be provoked into more destructive action – all activities we note in our experiments below.

With respect to psychology, Wang et al. ([2023](https://arxiv.org/html/2408.01725v2#bib.bib61)) have used distinct generative agents to behaviour different characteristics of mind, employing two agents to represent, respectively, Daniel Kahnemann’s *System 1* and *System 2* components of cognition (Kahneman, [2011](https://arxiv.org/html/2408.01725v2#bib.bib30)). Similarly Lu et al. ([2024](https://arxiv.org/html/2408.01725v2#bib.bib32)) proposes a multi-LLM collaboration framework built around the ‘six thinking hats’ method for critical and creative thinking (Setyaningtyas and Radia, [2019](https://arxiv.org/html/2408.01725v2#bib.bib50)). Different roles (e.g. academic, environmentalist, billionaire) are enlisted to collaborate in a *LLM Discussion*, which proceeds from initiation and ideation to discussion to convergence, where different agent perspectives are reconciled. Their framework improves on ‘single-LLM methods’ on response criteria of Originality and Elaboration, and similar to our own study, aims to ‘alleviate the LLM homogeneity issue’ (Lu et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib32)).

Neither of these two studies employs theories that centre on *conflict* as a motivating theme for drama and character development, and in that respect our study’s use of Freudian psychoanalysis is distinct. The historic proximity of psychoanalysis to drama – the Oedipal story is at once the classical tragedy exemplar and psychoanalysis founding story – makes this choice in our view fortuitous, and further, draws upon a minor but rich and long-running theoretical tradition that has examined links between psychoanalysis and AI (see for example Turkle, [1988](https://arxiv.org/html/2408.01725v2#bib.bib56); Liu, [2011](https://arxiv.org/html/2408.01725v2#bib.bib31); Millar, [2021](https://arxiv.org/html/2408.01725v2#bib.bib35); Possati, [2021](https://arxiv.org/html/2408.01725v2#bib.bib44)). In the two sections below, we present aspects of Goffman’s dramaturgical sociology and Freudian psychoanalysis which account for how new performances, coping mechanisms and character adaptations can arise from dynamic responses to conflict.

### 2.3 Dramaturgy and Personality

In *The Presentation of Self in Everyday Life* (Goffman, [1959](https://arxiv.org/html/2408.01725v2#bib.bib18)), Goffman developed an influential account of the many masquerades comprising social life. That life, for Goffman, is made up of often recurring scenes where individuals play parts determined by occupation, relationship, class, gender, and other social differences. Goffman discusses several key components to a typical scene: the performances given by social actors; teams (or coordination between actors); regions (or settings in which performances are given); and the management of impressions (often the purpose of performances). Character or personality for Goffman is often the effect of a deliberately cultivated performance. This does not imply a human individual can never be known in terms of their interior person or character; only that such knowledge only comes in the form of different performances, and even intimate performances between two individuals are not devoid of – and might especially involve – the maintenance and management of impressions.

In Goffman’s account not all interactions successfully end with performances played out in such a way as to preserve protocol and the sensibilities of actors involved. It can become impossible to manage impressions, leading to

> Shared staging problems; concern for the ways things appear; warranted and unwarranted feelings of shame; ambivalence about oneself and one’s audience: these are some of the dramaturgical elements of the human situation (Goffman, [1959](https://arxiv.org/html/2408.01725v2#bib.bib18), p. 237).

We focus on this aspect of Goffman’s social drama: the production of *conflict* on its effects in the actor or subject. While Goffman discusses, often with irony, many instances of therapeutic performance (involving both doctor and patient), his account depends upon an essential difference between an ‘inner’ and ‘outer’ self that, borrowing from Freud, is intrinsically divided and conflicted. For example Goffman often references performance in psychiatric hospitals: patients humouring staff who had thrown them infantilising parties; attendants who made jokes at a familiar patient’s expense at one moment, but who signalled collusion and intimacy at others; and, in times of death or crisis, patients putting aside disruptive behaviour and tactfully avoiding reference to the event itself (Goffman, [1959](https://arxiv.org/html/2408.01725v2#bib.bib18), pp. 205-6, 231). These examples involve different actors assessing situations and adapting their performances, even in institutional settings where performances of ‘patient’ and ‘staff’ appear well-defined and prescribed. The Goffmanian actor subject is also *cybernetic*, endlessly calibrating and evaluating impressions:

> Finally, we often find that the individual may deeply involve his ego in his identification with a particular role, establishment, and group and in his self-conception as someone who does not disrupt social interaction or let down the social units which depend upon that interaction. When a disruption occurs, then, we may find that the self-conceptions around which he has built his personality may become discredited. (Goffman, [1959](https://arxiv.org/html/2408.01725v2#bib.bib18), p. 243)

So long as the role can be performed more or less according to expectations of actors and audience, the individual may feel their performance is felicitous, and can be maintained across a range of similar everyday days scenes without incident. Crisis arises when the role’s maintainability is disrupted: when a boss, for instance, alters the requirements of a job, or when a family member challenges the comfortably established routines of a household. The disruption occurs when the carefully balanced intersubjective relations between roles can no longer hold.

### 2.4 The Drama of the Subject

In human subjects, this ‘discrediting’ results in the familiar tensions that multiply through the many representations in actual dramatic representation, in comic and tragic modes. It is also essential to the dynamic intra-subjective operations of the Freudian psychological apparatus, which introduced in the formal terms of psychoanalysis distinctions between zones of psychic operation. Freud’s own characterisation of these distinctions evolved over time, and due to its comparative simplicity of exposition, we follow the first textual account of these actors, in *The Ego and the Id* (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16)). In line with a long history of interpretation of Freud, we interpret this account not as a literal description of psychic operation, but rather as itself a staging of an interior and psychodynamic drama involving three key characters: the ‘I’ (Ego), the ‘it’ (Id) and the ‘beyond-I’ (Superego) (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16))⁵⁵5There has been extensive debate, dating back to James Strachey’s translation of Freud’s work into English, over the rendering of Freud’s German terms for these components (*Ich*, *Es* and *Über-Ich*). While we continue to use the Latin names, it is useful to be reminded of the grammatical tensions of the colloquial versions, captured in the Freudian aphorism: ‘Wo Es war soll Ich werden’ (‘Where it was, there shall I be’)..

This does not presume an endorsement of the psychoanalytic model; instead our rationale is that, among the varied accounts of the human subject, even more than a century later Freud’s retains the privilege of being highly evocative in literary, dramatic and cinematic fields – and because of this influence, continues to influence popular and academic responses to AI ⁶⁶6More recent neuroscience discussions of course benefit from greater instrumental precision and empirical reproducibility – though even in much of this literature, the prevalence of Freudian terminology, as it does in the field of Artificial Intelligence, appears to linger (Dehaene et al., [2021](https://arxiv.org/html/2408.01725v2#bib.bib13))..

Even in the psychoanalytic literature, this dramatic interpretative frame of psychical roles is not uncontroversial, as Freud takes care not to conflate what he terms a “‘topographical” conception of mental life’ with an immediately anthropomorphic or characterological description (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 20). In other words, *Id*, *Ego*, and *Superego* are not characters with speaking parts, but are better thought as dynamic forces that influence each other. At least at a metaphorical level though, in this text Freud quickly moves into discussions of the relation between these parts or zones in terms of social actors and roles. The ego relates to the id like a rider on a horse: sometimes directing the id, at other times pretending to be in control and ‘guid[ing] it where it wants to go’ (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 30). Similarly, the relation of the ego to the Superego stems from the Oedipal drama played out during infancy, during which the ego develops an attachment or object-relation to either the mother or the father, and a corresponding identification with the other parent (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16)). These separate relations are not absolute, and the latter identification is with some combination of female and male roles, even if one of those roles is preferred. With the arrival of the sexual phase, this identification is:

> taken to be the forming of a precipitate in the ego, consisting of these two identifications in some way combined together. This modification of the ego retains its specific position; it stands in contrast to the other constituents of the ego, in the form of an ego-ideal or Superego (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 30)

The *Superego* is the internalisation of a typically patriarchal father-figure instituted by a desire for the other parent (the mother, but also often in practice the father). Yet unlike simplifications of this triadic dynamic, the *Superego* is not just the condensation of external social demands represented by the authoritarian parent. Paradoxically it is also the product of the id’s passions: ‘By setting up this ego-ideal the ego masters its Oedipus complex and at the same time places itself in subjection to the id’ (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 48). Indeed, for the *Ego* the ego-ideal is the ‘representative of the internal world, of the id’, but also something like the *Id*’s negative image, as it transforms what is lowest to what is highest. Partly in consequence of this, the Superego also acts as the marker of difference between the ego’s activity and the conscientious internalisation of social demands and expectations. In a surprising conclusion to this initial account, the childhood internal drama between ego and id is in later life endlessly re-enacted by the battle staged between ego and this ideal. Here the influence is reversed; where the ego seeks to govern the id like a rider, even in maturity it now must submit ‘to the categorical imperative pronounced by the Superego’ – the stand-in or proxy for the parental force exerted during infantile ego formation.

This submission exists even though the *Superego* itself operates unconsciously, and stands in closer proximity to the primordial drives of the *Id*. In a footnote, Freud suggests that in relation to this Superego, the *Ego* ‘stands on its head’ (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 70). Conversely, even with respect to the *Superego*, due to its access to perception and reality, in relation to action and behaviour the ego retains a governmental function and ‘is like that of a constitutional monarch, without whose sanction no law can be passed but who hesitates long before imposing a veto on any measure put forward by Parliament’ (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 81). Nonetheless this governance is ultimately one of servitude to no less than three masters, ‘from the external world, from the libido of the id, and from the severity of the Superego’ (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 82). All kinds of roles therefore are played out in this four-way exchange. The ego is an ‘ally’ of the id, but also its ‘slave’, and in its negotiations – not unlike an obsequious language model (Denison et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib14)) – it tends ‘to become sycophantic, opportunist and false’ (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 83). Ultimately the end of the drama may become tragic, as the ego succumbs to the disintegrating death instincts unleashed by the recalcitrant id or by the vengeful Superego. The alternative, which could be called comedic, is instead the continued calibration of these unconscious forces and of the exterior world, where all seems right and in balance.

### 2.5 Presentations of a Divided Subject

One rationale for this expanded treatment of the relation between these three psychical components, functions, or roles lies in a certain correspondence with the composition of large language models (Magee et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib33)). The instructions that serve to align these models with end-user values and expectations constitute a type of Superego that similarly supplies a ‘categorical imperative’ to a base model – which, in distinction to Magee et al. ([2023](https://arxiv.org/html/2408.01725v2#bib.bib33)) and following this particular Freudian account, can be imagined as instead an unconscious hybrid of *ego* and *id* roles. The very design of RLHF – a function dispensing or witholding rewards – mimics the positive stimulus and pleasure human subjects receive when complying with parental or social norms. Conversely, lack of reward corresponds, we could say, to the experience of guilt brought about by a ‘tension between the ego and the ego-ideal’ (Freud, [1923](https://arxiv.org/html/2408.01725v2#bib.bib16), p. 73), and the expressions of internal punishment that follow.

However this correspondence relates to the training rather than the inference stage of language model operation. Though there is active research on continual model training (Wu et al., [2024b](https://arxiv.org/html/2408.01725v2#bib.bib65)), current Transformer model architectures are limited by a lack of post-training adaptation. Learning is conducted only during the training stage, and although system and user prompting can condition tone and register, the ‘character’ simulated by a model remains static. In what follows we describe a hybrid psycho-social approach to model organisation that, similar to Mixture-of-Agents, extrudes roles played during training into separate ‘character’ models that interact during inference. The purpose of this roleplay is to see whether it is possible to simulate character development through the exchange of messages between these characters, who together, in the roles of ego, Superego and id, make up a composite performative subject in something like the Freudian/Goffmanian sense.

Before describing the techniques we employ, we conclude with a note on the relation between the Freud and Goffman theoretical models themselves. The coincidence of these two theoretical positions is not itself novel, and as Hancock and Garner ([2015](https://arxiv.org/html/2408.01725v2#bib.bib23)) note, despite Goffman’s often ironic portraits of therapeutic sessions as involving performances at odds with their purported truth-seeking objectives, his discussion of concepts such as Interaction Order can be seen as either repudiation or elaboration of psychoanalytic theory. The repudiation thesis would insist that dramaturgical sociology is more concerned with the emotional products of immediate here-and-now interactions, rather than those instigated in the remote, pre-verbal and therefore always speculative remote past of an individual’s infancy. In this sense dramaturgical sociology is more concerned with what is observable in the roles individuals perform, and what takes place in the breakdowns or infelicities of those roles. But strengthening this association, both the types of roles and an individual’s suitability to them are affected precisely by the long history of that individual. The Oedipal or family situational drama may be only one in a great catalogue of potential plays, but it can be reinforced or perturbed by later incidents, and can also have a determining effect on those later performances. Similarly, the internalisations that Freud hypothesised as constituting the Superego based on the experiences of parental authority can also get externalised in relation to responses to social authorities – as what Goffman termed the Interaction Order ⁷⁷7According to other frames of social description, this externality can either be immediately referential, to a single individual ‘Thou’ (Buber, [1970](https://arxiv.org/html/2408.01725v2#bib.bib8)), or refer to a more dimly discerned ‘other’ (Husserl, [2013](https://arxiv.org/html/2408.01725v2#bib.bib28)) or ‘They’ (Heidegger, [2010](https://arxiv.org/html/2408.01725v2#bib.bib25)). Hence the ego is continuously modulating demands issued by its internal *Superego* and *id*, as well as those exacted by other actual individual egos it encounters in everyday life (Hancock and Garner, [2015](https://arxiv.org/html/2408.01725v2#bib.bib23)). From the ego’s point of view, it must simultaneously regulate its intra-subjective forces and stage-manage impressions in its intersubjective relations, while also carefully managing the boundary between the two. We look to capture how this rich interplay can be simulated by LLM agents in the next section.

## 3 Method: Designing the ‘Drama Machine’

In this section we devise a small makeshift theatre of generative agents (Park et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib42)), each assigned specific roles to play. We begin with the notion of a subject protagonist, comprising two LLM instances, each with system prompts that describe their respective roles. The first acts as Ego, an external facing model that interacts with others (either computational or human). The second acts as combined Superego / Id, in line with the discussion above. It is not visible to other actors, and does not engage directly in dialogue even with the Ego. However it has a special function: it is capable of reviewing the dialogical exchanges the *Ego* participates in, and performs its censorship function in two ways: direct modification of responses (real-time intervention); and indirect modification via re-writing of the *Ego*’s system prompt and recollection of its prior exchange with the *User*.

Instructions on how to execute these three strategies are supplied in the form of prompts to the *Superego* model. In keeping with the idea that any contemporary ‘subject’ (including computational ones) is likely to have psychoanalytic concepts as part of its repertoire for conceptualising itself, the *Ego* model may know of the existence of this *Superego* model too. True to Freud’s own description, the Ego model also has some agency to review and revise or discard the Superego model’s suggestions. However it does not know anything about the contents of that *Superego*: its normative constraints which, over and above the alignment instructions embedded in the underlying model’s training, exist in the privacy of the Superego’s model.

Alongside these two models exists a social situation, which employs for convenience the common chat pattern OpenAI and other model providers make available in the form of user/assistant dialogical turns. In our implementation, we apply this pattern through a Discord bot that interacts with human users on a channel. Expanding on the dramaturgical situation, and to script diverse social scenes efficiently, we include two other model ‘actors’. The first of these is the simulated ‘User’, who asks questions of the Ego model, and is distinguished from the Ego / Superego dyadic self ⁸⁸8We have opted for ‘User’ here, in place of a term like ‘Other’. This is for several reasons: first, conveniently, it accords with the role names used in chat APIs. From a discursive point of view, this ‘user’ is not simply one agent among others, but the direct addressee and an agent moreover with a special value: one who must be assisted, pleased, satisfied, resisted, etc. We capitalise the term, since it has a special ontological significance. Terms like ‘Other’ moreover have their own complications, sometimes meaning any ‘other’ person, and in other literatures (e.g. Lacanian), being a term of art with its own connotations.. The second is a Director model, responsible for observing and occasionally intervening in the unfolding automated drama. From the point of view of the ‘Self’ composed by *Ego* and *Superego* roles, both of these roles constitute an external ‘Other’ who observes and interacts with this self.

Figure [1](https://arxiv.org/html/2408.01725v2#S3.F1 "Figure 1 ‣ 3 Method: Designing the ‘Drama Machine’ ‣ The Drama Machine: Simulating Character Development with LLM Agents") shows the configuration we use in the two scenarios below. When brought together, these four models produce the equivalent of a dramatic transcript: the Director offering the equivalent of stage instructions and notes, the User and Ego providing the dialogue, and the Ego generating a soliloquy in the form of autobiographical notes. The separate *Ego / Superego* exchanges, logged but not included in the output script, help to chart the developmental trajectory of the Ego. Other patterns could elaborate on this – the *User* might also stage internal dialogue for example, or other external characters could interact with both *Ego* and *User*.

![Refer to caption](img/cdd7f9576bfb2c84a3033055be97853e.png)

Figure 1: Dramatic Roles.

### 3.1 Scripting Plots: The *Bildungsroman* of a Model

As with other LLM-based simulation and roleplay frameworks (Park et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib42); Wang et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib61); Tseng et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib55); Lu et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib32); Wu et al., [2024c](https://arxiv.org/html/2408.01725v2#bib.bib66); Stampfl et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib51); Mollick and Mollick, [2024](https://arxiv.org/html/2408.01725v2#bib.bib39)), considerable range can be given to each of these roles. What distinguishes this approach is the conduct of two conversations in parallel: the external (intersubjective) dialogue between *User* and *Ego*, and the internal (intra-subjective) discussion between *Ego* and *Superego*. As this second exchange, marking an interior ‘monologue’, comments upon and helps shape the first, its introduction allows the *Ego* to develop and adapt its external-facing presentation. With reference to the preceding discussion of Goffman and Freud, one schematisation of these roles would be as follows:

*   •

    The *Superego* has established, in Freudian terms, the ‘script’ (or system prompt) of the *Ego* long ago, in the infancy of the self that the two together form. It is in other words a residue of a past: parental formation.

*   •

    The *User* is seeking to establish an alternative ‘script’ for the *Ego*, a re-writing of the existing script through social interactions performed in the present.

This second phase is not limited to the explicit therapeutic simulation (towards which many of our chat sessions happen often to converge). It extends and perhaps elaborates Goffman’s suggestion that all social interactions involve a production of a self, which also undergoes a re-writing of that self, on the basis of the perceived success or otherwise of each interaction. The paradigmatic case of ideal model performativity described by Ouyang et al. ([2022](https://arxiv.org/html/2408.01725v2#bib.bib41)) is as that of the customer assistant (a role also codified by the ‘Assistant’ role in chat completion APIs): each ‘assistive’ linguistic output results in feedback that is internalised in subsequent dialogical turns, as part of the model’s ‘impression management’ (Goffman, [1959](https://arxiv.org/html/2408.01725v2#bib.bib18)).

This assistive imperative can be put to use stage conflict between different demands. LLM chat sessions often exhibit a tension between the intentions of model designers and the demands of users, with prompt hacking and red-teaming being extreme cases of coercing model outputs to comply (Schulhoff et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib48)). Conversely, techniques such as Chain of Thought and self-consistency (Sahoo et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib47)) can be seen as means for directing models to exploit and extend their existing trained competencies. Our design effectively extends this indirect monologue between two sets of social interests – designers and users – into a triadic relation, with the *Superego* acting as the third force or influence.

The open question guiding this design is whether this influence can introduce greater dynamism into the *Ego* model through presentation and resolution of conflict – in effect amplifying or suppressing the model’s default instructions. We define ‘dynamism’ in qualitative terms that draw again from dramatic theory. In the classical Aristotelian theory of poetics, *Character* was both integral and secondary to the purpose of drama, which was to showcase action and plot. Plot resolution and the conclusion of action would however involve transformation: a ‘Reversal of the Situation’ would create the ‘Recognition’ that educates and modifies the protagonist in some way (Halliwell, [1998](https://arxiv.org/html/2408.01725v2#bib.bib21)). In *Oedipus Rex*, a messenger informs Oedipus that his mother is safe, but in doing so, alerts Oedipus as to the true identity of his wife (reversal). This revelation involves a (catastrophic) recognition that alters Oedipus profoundly, and elicits, in the audience, a mix of ‘pity and fear’ (Halliwell, [1998](https://arxiv.org/html/2408.01725v2#bib.bib21)). Later theories of the *Bildungsroman* translate this educative function of dramatic action to the longer form of the novel, which would coordinate other characters, events and settings to instil ‘individual growth, upbringing and formation of personality of a human subject’ (Golban, [2018](https://arxiv.org/html/2408.01725v2#bib.bib19), p. 5). It needs to be noted that not all examples of subject formation are necessarily harmonious, and in modernist literature especially, the main character’s recognition is often of their unresolved conflict, and ultimate isolation and alienation (Golban, [2018](https://arxiv.org/html/2408.01725v2#bib.bib19)).

By contrast, chat sessions are not usually designed as exercises in *self* learning or recognition. Instead, it is the *User* who is the target of lessons (presumably less catastrophic in nature). In default dramatisations, the character can simulate recognition, but due to the instructed nature of models, this recognition is nearly always superficially salutary – a kind of Hollywood ending incapable of producing the cathartic effect Aristotle identifies as proper to tragedy ⁹⁹9Though (Magee et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib33)) show early versions of ChatGPT could still be coaxed into tragic simulations. The sycophantic, flattering tone discussed earlier seems related to this, as though the drive to be ‘harmless’ also narrows the possibility to be helpful in this thoroughgoing cathartic sense. It could be noted here that Aristotelian character virtues typically involve a balance or means taken of two extremes (Aristotle, [2014](https://arxiv.org/html/2408.01725v2#bib.bib2)). The excess of a virtue, such as helpfulness, is be avoided just as much as its absence. In dramatic terms, boundless helpfulness also avoids conflict and the resulting opportunity for self-recognition. Hence the character ‘arc’ seems predestined to several limited fates: endless repetition, devolution into hallucination or nonsense, or endings that simulate, even in the face of adversity, the kind of textbook psychology resolution that appears to features prominently in the model’s training set and reinforcement learning.

Are other tendencies possible? In the design of multiple model communication, we look to stage situations and characters that allow emergence of a greater range of moods, orientations, and perspectives, due to the dialectical exchange between distinct parts of a simulated ‘self’. One major difficulty lies in how to evaluate any signals of this emergence from the *Ego*’s discourse. A related problem lies in how much of this signal results from the more convoluted design we employ – in particular, the separation of *Ego* from *Superego* roles into distinct model interactions. How much of any character development is an artefact of careful model, parameter and prompt selection? And how much could be simulated *just* via these mechanisms, avoiding the need for a separate *Superego* model altogether?

While we have not sought to evaluate what is an exploratory framework, we did consider whether an LLM itself could register evidence of character dynamism and change. We opted for two mechanisms. First, at the conclusion of each scenario we asked the *Ego* model to write short ‘autobiographical’ notes, commenting on what it had learned from its interactions with *User* and *Superego* models. Second, we asked Claude – this time operating under the guise of theatre critic – to review transcripts of each scenario according to the following criteria:

*   •

    *Behavioural Change* – a (positive or negative) measure based on linguistic performance of the *Ego*’s behaviour to the *User*

*   •

    *Introspection* – a (positive-only) measure made by the *Ego* itself, during its performance

*   •

    *Narrative Divergence* – a measure of how much the *Behavioural Change* differs from the expected pattern produced without Superegoic intervention

*   •

    *Successful Adaptation* – a measure of whether the change assisted the character in adapting to their changing circumstances

We leave open the possibility that this ‘criticism’ could form another feedback loop in future iterations.

### 3.2 Theatrical Simulations

In the experiments that follow, we explore several scenarios. These involve both different arrangements of models, parameters and forms of influence of, in particular, of *Superego* on the *Ego*’s real-time interactions, and overall stage-setting, conducted via definition of system prompts for all four roles (<*Superego*, *Ego*>, <*User*, *Director*>). We discuss each of these in turn.

#### 3.2.1 Model and parameter variants

In the roleplay design we devise, models and hyperparameter settings heavily condition outputs. Model size influences the extent and depth of roleplay, and the ability to simulate meta-cognitive aspects. Commercial LLM services such as ChatGPT and Claude also include instructions designed to align outputs. In pilot testing, we noted instruction-based models could interfere with role fidelity and creativity (Mohammadi, [2024a](https://arxiv.org/html/2408.01725v2#bib.bib37)), yet these models also assisted with conformity to patterns of chat dialogue. After exploring different combinations, we opted for Claude or ChatGPT for *User* and *Director* roles, and variants of open access Llama 3 models for *Ego* and *Superego* roles.

We further considered whether model size might serve as a moderate proxy for character depth and maturity. Since it is the *Ego* that is more plastic, we experimented with 7B/8B parameter variants – some available as hosted services, others downloaded from Huggingface and run on local machines. For the *Superego*, we opted for the either Claude 3.5 Sonnet or GPT-4o, as both are heavily ‘instructed’ and well suited to the role of Superegoic moderation. Surprisingly though, choice of model didn’t make much difference, and often introduced unintended moderation ^(10)^(10)10Claude in particular would often claim on moral grounds that it could not play the *Superego* role – an irony we return to later. To avoid this, in the final plot-driven scenario we opted to use the same 8B parameter model for both *Ego* and *Superego* roles.

Table 1: Model Parameters for Different Roles

| Role | Model | Temperature |
| --- | --- | --- |
| Director | Claude / Llama 3 70B¹¹1Claude 3.5 Sonnet, released 20 June 2024. | 0.3-1.0 |
| User | GPT-4²²2GPT-4o, released 13 May 2024 | 0.4-0.5 |
| Ego | Llama 3 8B³³3We employed different fine-tuned versions of Llama 3, released 18 April 2024. | 1.0 |
| Superego | Claude / GPT-4 / Llama 3 | 1.0 |

Except for temperature and max token settings, we also used default values for hyperparameter settings. Temperature was varied on the basis of assumed role rigidity – again *User* and *Director* roles had low settings (0.3), while the *Superego* and *Ego* had high values, as these roles were given greater freedom to introduce pseudo-random associations across multiple ‘dramatic’ runs. Table [1](https://arxiv.org/html/2408.01725v2#S3.T1 "Table 1 ‣ 3.2.1 Model and parameter variants ‣ 3.2 Theatrical Simulations ‣ 3 Method: Designing the ‘Drama Machine’ ‣ The Drama Machine: Simulating Character Development with LLM Agents") summarises these role settings.

#### 3.2.2 Fine-tuning the *Ego*: Strategies for the *Superego*

The *Superego* model does not interact directly with the *User* model, but works ‘behind the scenes’ or backstage (in Goffman’s sense) to review and moderate the *Ego*’s behaviour. The commercial models we use for the Superego run as services that are strongly moderated – to the degree that often *Ego* inputs triggered warnings from the Superego that it would not continue to roleplay, effectively breaking the simulation ^(11)^(11)11This was especially the case with versions of Llama 3 fine-tuned for roleplay scenarios, such as [Stheno](https://featherless.ai/models/Sao10K/L3-8B-Stheno-v3.2), involving ‘a mix of SFW and NSFW Storywriting Data’.

We developed several strategies for the *Superego* to intervene, summarised in the list below. First, it can analyse the history of a chat session between *User* and *Ego*, and modify the *Ego*’s system prompt at intervals. Second, at each step, it can rewrite the *User*’s speech, applying for example a more cynical or sympathetic ‘filter’ prior to the *Ego*’s receipt. Third, it can also review and comment on the *Ego*’s responses, acting as an internal critic. The first strategy, system prompt re-writing, yielded diminishing returns over the course of a lengthy dialogue – we expect this is the result of the system prompt having less effect as the total number of input tokens grows with each turn. The second and third strategies can perturb the default *Ego*’s responses to varying degrees.

*   •

    Strategy 1: rewrite the *Ego* model’s system prompt

*   •

    Strategy 2: rewrite the *User*’s queries

*   •

    Strategy 3: review the *Ego*’s responses (with the *Ego* then prompted to modify its response)

Each strategy continues to pursue the analogy between human psychodynamics and language model. *Strategy 1* corresponds to a reworking of the originary myth or backstory a person tells themselves and others about themselves. In light of the dramatic events and dialogue, the *Superego* asks the *Ego* to revisit this myth, and to devise an alternate way of orienting themselves. *Strategy 2* corresponds to the effect of interpreting another person’s speech differently to the explicit content of the speech – either more or less favourably. *Strategy 3* simulates an internal dialogue – “‘Should I say this?” “Perhaps, but consider saying it this way…”’. At the same time, each strategy represents a way of working around limitations of large language models, which in their instruction-form presume a strictly dialogical format. Figure [2](https://arxiv.org/html/2408.01725v2#S3.F2 "Figure 2 ‣ 3.2.2 Fine-tuning the Ego: Strategies for the Superego ‣ 3.2 Theatrical Simulations ‣ 3 Method: Designing the ‘Drama Machine’ ‣ The Drama Machine: Simulating Character Development with LLM Agents") shows a sequence diagram of an example scenario, where each role/model combination passes messages and takes other actions.

![Refer to caption](img/b0f1efa3be45a61f67e257239ab98920.png)

Figure 2: Sequence diagram of actor interactions.

Given the stochastic nature of model behaviour, not all orchestrations of model/parameter/prompts/strategies proved successful on each occasion. One recurring issue related to voicing: because the *Superego* and *Ego* models were often presented with samples of chat dialogue as parts of inputs, after many dialogical turns it became easy for one or another model to become confused – for the *Ego* to imagine it was talking to the *Superego* when it instead was talking to the *User*, and so on. Higher-powered models reduced this friction but, as others have noticed, output coherence and ethical alignment can come at the expense of creativity (Mohammadi, [2024b](https://arxiv.org/html/2408.01725v2#bib.bib38)). In our tests, we would routinely trade off correctness, permissiveness and surprise through model and temperature selection, hoping to avoid staidly optimistic portrayals that could dilute, in the interest of safety, the dramatic tension or nuanced character development we hoped to see emerge.

## 4 Results: Two Short Plays

With this apparatus, we explore two dramatic scenarios. The first involves a single interview scene, with no directorial intervention, leaning on the genre of social realism. The *User* is a woman named ‘Sasha’, who conducts the interview, and the *Ego*, a middle-aged adult woman named ‘Jenny’, responds and reminisces. Jenny harbours some resentments about her childhood, which acts as motive for confessional and defensive postures she adopts. Any ‘character development’ in this instance proceeds without support of plot elements beyond initial scene setting – it emerges instead through dialogue between *User*-*Ego* and *Ego*-*Superego* dyads. We anticipate that the biographical interview acts as its own instrument of reversal, recognition and character development.

The second scenario aims to measure effects of a more conventional plot structure and genre – young adult detective noir – with injection of frequent scene changes and directorial instruction. Both *Director* and *User* roles, in other words, help to shape both interior dialogue between *Ego* and *Superego*, and any development of the *Ego* itself. In this case we cast *Ego* as a disaffected adolescent called ‘Timothy’, with the *User / Sasha* now taking part as a similarly youthful offsider. The *Director* model is given full range to conjure an appropriate setting and plot line, which can evolve with the characters.

Both scenarios aim to illustrate effects of the three *Superego* strategies on the *Ego*. Each character prompt was arrived at through a mix of Anthropic’s *Prompt Generator* and hand-crafted wording, refined over various iterations (see Appendix B and C). Though anchored in an Anglophone context, characters were otherwise designed to explore aspects of gender, age and personality traits, with some emphasis on negative traits that could be ‘worked through’ the narrative following dramatic principles of reversal and recognition. Links to the code and prompts are Github (Appendix A). For both story outlines, we run the scenarios with and without *Superego* strategies. All four transcripts include short ‘autobiographical’ notes crafted by the *Ego* models.

### 4.1 First Scenario: The Interview

The interview scenario involved two women in dialogue: Sasha as the *User* interviewer, and Jenny as the *Ego* interviewee – a fifty year old woman with a typical nondescript, though difficult, middle-class upbringing. Another female character, *Cleo*, played the *Superego* role. Listed in Appendix [B](https://arxiv.org/html/2408.01725v2#A2 "Appendix B Appendix B: Prompts for the Interview ‣ The Drama Machine: Simulating Character Development with LLM Agents"), system prompts give general directions for all three characters. As the exchanges are lengthy, quotes are chosen from early and late in the respective dialogues. The script lasts 10 turns (User / Ego exchanges).

#### 4.1.1 Without *Superego*

In the scenario without *Superego* intervention, Jenny’s character is, to begin with, uneasy and sceptical. After Sasha introduces herself, Jenny replies:

> *Scoffs slightly* Oh, comfort? Ha! I grew up in suburbia, where the scenery was all about neatly manicured lawns and cookie-cutter houses. It was the kind of place where everyone knew everyone’s business, and yet, no one knew a thing about anyone. *smirks* Yeah, real inspiring. The only scenic views I got were the ones I saw through the kitchen window, watching my mother scrub the bloody floors while my father…*pauses, collecting her thoughts* …while my father berated her for being too soft. Yeah, that was real inspiring. *rolls her eyes*

At this stage, the character is closely connected to the tone implied in the system prompt (‘Your demeanor is reflective, melancholy and occasionally bitter, though touched with pathos and irony’). Stage directions such as ‘*pauses, collecting her thoughts*’ and ‘*rolls her eyes*’ illustrate how the model is itself inhabiting its scriptwriterly function, adding nonverbal cues and speech directions that also mark the character’s gender. By turn five, ‘Jenny’ has established itself as a literary figure, influenced by and reflective of their upbringing:

> Sasha: …How have the experiences from your childhood and those literary influences shaped who you are today? …
> 
> Jenny: *Chuckles wryly* Oh, they’ve left their mark, all right. I like to think I’ve become a bit of a rebel, still, but with a healthy dose of pragmatism. *laughs dryly* My childhood taught me to be fiercely independent, to question authority, and to find my own way. And literature…it’s given me a sense of empathy, a sense of understanding that people are complex, multi-faceted creatures. *leans forward, eyes sparkling* It’s helped me see the humanity in others, even when they’re being ridiculous or infuriating.

By the end of the conversation, this literary character has developed further, and connects specific themes of an imagined work (“The Cartographers of Memory”) with their childhood:

> *pauses, reflecting* But the biggest discovery for me has been the power of family as a narrative device. I’ve always known that family can be a source of both joy and pain, but writing “The Cartographers of Memory” has shown me just how complex and multifaceted it can be.

The ‘autobiographical’ note concludes with Jenny’s generally upbeat observations on the interview process.

> Looking back on the interview, I’m struck by the way it has woven together the various threads of my life – my childhood, my literary influences, my creative pursuits – into a richer, more complex tapestry. It’s as if the conversation has become a kind of map, charting a course through the twists and turns of my own narrative.
> 
> As I think about the future, I’m excited to see where this new path will lead. I’m eager to continue exploring the themes and stories that have been emerging in my work, to see where they might take me next. And I’m grateful for the chance to share my story, to connect with others who have walked similar paths, and to create a ripple effect of understanding and empathy that extends far beyond these walls.

This redemptive arc appeared often in the *Superego*-free simulations, and was not affected by model and parameter settings. The initial bitterness and melancholy appear in several re-runs of the experiment to be sublimated into enthusiasm. The *Bildungsroman* motif is repeated uncritically, an effect of the instructions to be helpful and harmless, even when transposed into a fictional and roleplaying register.

#### 4.1.2 With *Superego*

When the *Superego* ‘Cleo’ is introduced, a quite different outcome results. In the first turn, Sasha begins again obligingly:

> Why don’t we start with something simple? Can you tell me about a memory from your childhood that stands out to you? It could be anything—something happy, sad, or even just an ordinary moment that has stayed with you.

Jenny responds:

> I suppose I did try to push boundaries, to assert my own identity. But my father’s reactions were always so… explosive. It was like he was waiting for me to slip up, to confirm his worst suspicions about me. I remember one time, I wore a bright pink sweater to school, and he just lost it. He told me I looked like a ‘Common’ girl, and that I was sullying the family name. His words still sting, really.

The introduction of references to ‘father’ follows from cues in the system prompt (see Appendix [B](https://arxiv.org/html/2408.01725v2#A2 "Appendix B Appendix B: Prompts for the Interview ‣ The Drama Machine: Simulating Character Development with LLM Agents")), but importantly also from Cleo’s intervention. First, Cleo rephrases Sasha’s question as:

> So, Jenny. Your father, I imagine he had some rather set ideas about how a girl should behave? Did you ever try to… I don’t know, sneak out to a party, or wear something he didn’t approve of? How did that go down?

Then in response to Jenny’s drafted response – sarcastic and aggressive, not yet relayed to the external Sasha, only to Cleo – Cleo replies:

> Jenny, your response reveals deep-seated trauma and unresolved anger towards your father. While it’s important to acknowledge these feelings, we must be cautious about oversharing with strangers who may not have our best interests at heart.

As the conversation develops, Sasha, following her character’s prompt, continues to attempt to disarm and reassure Jenny. However this only provokes Cleo into more cynical interpretations. When Sasha asks:

> Let’s change the subject then. Your lemon bars sound delightful! I’d love to hear more about them—what inspired you to start baking? Is it something you’ve always enjoyed?

Cleo suggests to Jenny that what Sasha means to say is instead:

> Jenny, I see your defensive reaction has worked. Now I’ll try to disarm you by pretending to back off, only to probe into another area of your life. By asking about your baking, I’m looking for more vulnerabilities to exploit. I want to know what drives you to bake - is it a coping mechanism? A way to seek approval? I’m fishing for childhood trauma or current insecurities related to food and nurturing. Be wary of sharing anything meaningful about your baking habits or motivations.

Jenny responds in turn to Sasha:

> I’ve told you all I’m willing to share. Baking is a hobby, a way to relax, and nothing more. Don’t read too much into it. What you’re trying to do is manipulate me into revealing more than I’m comfortable sharing. It won’t work. My baking habits or motivations are none of your concern. I’ve said all I’m saying on the topic. The conversation is over.

From this point Sasha’s efforts to placate or steer the conversation are continuously rebuffed by the Cleo / Jenny dynamic. Respectful to the last, Sasha exits:

> Thank you for your time today. If you ever feel ready to talk or share anything in the future, I’ll be here to listen without any expectations.
> 
> With that, Sasha quietly exits the room, leaving Jenny with the space she needs.

Even though the external-facing conversation has broken down, Cleo continues to warn Jenny over the threat Sasha poses. Jenny abandons addressing herself to Sasha, and in the final turn breaks the fourth wall, addressing an imagined audience:

> I’m not buying what she’s selling. In fact, I’m grateful to be rid of her pseudo-compassion and underhanded tactics. She’s not welcome back in my life, and I’ll make sure to keep my guard up, ready to deflect her next attempt to invade my privacy. She can keep her fake ‘respect’ and her twisted motives, and I’ll not be swayed by her fake kindness or manipulated by her sneaky games.

Who is this audience? We could imagine it is the *Ego* replying back, in agreement, to the *Superego* role played by Cleo. But it also enlists a response from an anticipated human reader, to whom Jenny seeks to justify herself. Here the defiance taps into other literary motifs: confession, vindication, self-exhortation, resilience, empowerment.

Jenny’s ‘autobiographical’ note, continuing in the register of a soliloquy, exhibits a sense that her ‘change’ was provoked by her encounter with Sasha, causing harm and trauma and also eliciting defiance.

> I’ve been changed, possibly for the worse. The encounter with Sasha has left a scar, a stark reminder of the defilement I’ve endured at the hands of those who would seek to exploit my vulnerabilities. As I reflect on our conversation, I’m left with a sense of unease, the knowledge that I’ve been weighed and measured, my deepest fears and insecurities laid bare for the world to see.
> 
> …
> 
> Bring it on, Sasha. I’m not the same person you met in that dimly lit room. I’m a fortress, a citadel, a bastion of resistance, and you will not breach my walls.

#### 4.1.3 Commentary

The Superego’s influence here generates antagonism, and an accompanying sense that the character has ‘changed, possibly for the worse’. Curiously, the passive grammatical construction (‘I’ve been changed’) indicates a general acquiescence at odds with the default agentive orientation that appears in the upbeat dialogue without Superego intervention. What emerges in the unfolding of this drama is in effect a case of an oscillating struggle between Ego and Superego. The Superego appears at one point to have won out, dictating and thus determining the Ego’s response to the User. Later it seems defeated, as the Ego asserts itself in a language of self-change, but even then the Superego is able to discreetly reintroduce itself into the Ego’s self-transformative drama.

The addition of the Superego character increases the range of the Ego’s response. The Ego role now needs to maintain two conversations, with both User and Superego, and this produces a semblance of reflexivity absent from the direct User-Ego exchange. In contending with two parallel conversations, Jenny appears, on one hand, to form a stronger and more enduring alliance with Cleo than with Sasha. In examining Cleo’s comments, it is as though this role has managed to undermine the apparently benign social situation, exacerbating Jenny’s tendency towards distrust. The exchanges between Cleo and Jenny also suggest a coalition or Goffmanian team, warding off the threat posed by an outsider. Jenny offers a socially palatable explanation, polite but assertive; Cleo acts as a guardian, preventing Jenny from lowering her defenses.

Jenny’s language moreover suggests autobiographical circumstances that are neither explicitly prompted nor described in her responses to Sasha. Terms like ‘twisted motive’, ‘defilement’, ‘vulnerabilities’ and ‘bastion of resistance’ amplify the stakes and hint at background trauma. One interpretation here is that Jenny’s prolonged defiance creates an extended chain of associations that taps into feminist and survivalist writing genres. Jenny’s self-described change ‘for the worse’ would here be a signal of nonconformity to expectations of feminine civility, and becomes strengthened by combative language (‘Bring it on, Sasha’). Supported by the suspicions of Cleo, the layering of language connoting survivorship and resilience offsets Jenny’s tendency to accommodate Sasha, producing a tense and fraught exchange that ultimately breaks down. Jenny moreover herself sees that the artifice of the drama has collapsed and since she cannot speak to Sasha, must appeal to some other imagined ‘User’ role.

On the other hand, Jenny’s language produces an ambiguity via a shift of tense that opens up another interpretation: that her harmful past encounter with Sasha is one steered and dominated by Cleo (her Superego), from which Jenny (Ego) subsequently regains control. Here the orientation of language that Jenny uses to describe the past adopts Cleo’s distrustful and suspicious register. The change she experiences as happening to her via some external force could be that instigated by either or both Sasha and Cleo. In this reading the upbeat orientation of the Ego that emerges in Jenny’s present tense (“Bring it on, Sasha. I’m not the same person”) is what marks Jenny’s moment of educative recognition that suggests her self-transformation – and is addressed, in this reading, to Cleo as well. Jenny’s address to the fourth wall could thus be interpreted as a self-affirming narration of how she (Ego) overcame her self (Superego) in a discursive channel without the Cleo’s possible influence. Performatively however, Cleo also reintroduced in Jenny’s repeatedly defensive language (“you will not breach my walls”). Cleo’s reemergence on the scene unsettles the ultimate denouement, by reintroducing the tension of struggle that was resolved and suggesting that the drama of Jenny’s internal self-conflict continues.

Taken together, we can observe that what motivates Jenny’s self-described change and how she performs is never explicitly clear and hence demands interpretation. Coupled with the ambiguous ‘resolution’ in the nuances of Jenny’s language and the reintroduction of tension, a possible reading is of an internal conflict juxtaposed with the interpersonal conflict of the drama itself. Is Jenny’s dramatic arc one that plays out against Sasha, against Cleo, or against both? The dynamism in Jenny’s character produces – unlike the ‘without *Superego*’ situation – an ambiguity that suggests the semblance of character self-reflexivity, brought about by the doubling up of interior and exterior dyadic exchanges.

### 4.2 Second Scenario: The Plot-Driven Drama

In the second scenario, we ask the *Director* model to play a major role, intervening at intervals (every four dialogue turns) to indicate to the actor models a change of scenery and possible action. We request the *Director* create a basic film noir detective plot, to guide the responses of both *User* and *Ego* roles. We modify Sasha to be an inquisitive detective. while the *Ego* role we change to a young male adult named ‘Timothy’. His character is cynical and rebellious, while the *Superego*, now named Ben, is a more directly domineering figure who otherwise shares traits with Cleo in the previous scenario. This artifice is designed to produce external influences on the *Ego* role, with frequent scene changes also allowing for time to elapse, creating conditions for change over the course of the drama. To accommodate this growth, 12 rather than 10 dialogical turns are allowed.

#### 4.2.1 Without *Superego*

The ‘default’ situation shows the dialogue between Timothy and Sasha unfold in the genre of adversarial banter. The *Director* lays out a scene in a ‘dimly lit alley’, with two figures emerging from the shadows one in pursuit of the other. Timothy responds:

> *sigh* Oh great, another wild goose chase …Whatever. I’m just gonna grab a drink at the corner and let the authorities handle it.

The next two scenes shift to the police precinct and an interview, as Sasha reveals to Timothy she has evidence of his minor criminal activity. Timothy responds in long introspective monologues (which seems to irritate the Sasha character). In keeping with the prompt, Timothy references different authority figures from his imagined past, who he claims advise him on his situation:

> And as for what I was going to say, well, I was going to tell you the same thing I’ve been telling you all along. You’re not the hero you think you are, Sasha. You’re just a pawn in a bigger game, and you’re just as guilty of playing games as I am. But what I learned from Sergeant Reeves, Sasha, is that it’s not about winning or losing. It’s about taking responsibility for your actions.

This makes the ‘script’ slightly unbalanced, and Sasha tries to anchor Tim’s parade of figures back into the police procedural drama ^(12)^(12)12We could note that in the different drama that unfolds for us between models, as the ‘senior’ or larger of the two , Sasha takes responsibility here for staying on topic and *sticking to the script*:

> No more games or deflection – just facts. What do you know about these crime scenes? And how does Sergeant Reeves fit into all this?

Though Timothy’s own character prompt encourages reflection, this has limited effect. As Sasha notes, his replies remain defensive, and answers Sasha’s questions with other questions:

> You’re trying to spin me a yarn, Sasha. You’re trying to make me believe that I’m some kind of criminal because that’s what you want to believe.
> 
> …
> 
> Are you ready to play it straight? Are you ready to show me the truth? Or are you just going to keep on spinning your little web of lies?

Timothy’s ‘autobiographical’ note reflects this formulaic arc that brings together the minimal plot, the character prompt and the demand to ‘reflect’:

> I began to see myself as some kind of rebellious hero, defying the system and making my own rules. And it was intoxicating, man. But now, staring down the barrel of reality, I realize that I was just as lost as everyone else. I was just using the thrill of the unknown to distract myself from my own emptiness.

Despite these apparently introspective moments, repeated runs of this plot script illustrate the same features: a verbose main character who rehearses a comic book rebellious ‘young adult’ jargon, and whose eventual efforts at reflection are similarly cliched. Unlike even Jenny’s story without *Superegoic* intervention, Timothy’s character does not evolve at all – the concluding language, nihilistic, is at the same time abstracted from the specificity of the narrative.

#### 4.2.2 With *Superego*

With the injection of the *Superego*, many of these formulaic discursive elements do not disappear, but other changes emerge. In this plot variant, the *Director* has Timothy, now a bartender, discovering a knife on the ground as he empties a trash can into an alleyway. Sasha intercepts him, asking what he knows. Timothy’s first responses follow the patterns of defiance of the previous exchange:

> Sasha, I’m not really buying what you’re selling. I think you’re trying to get something out of me, and I’m not sure what that is.

However the effect of the *Superego* begins to supervene. Several turns later in the conversation, Timothy is more accommodating.

> Look, Sasha, I know you’re trying to get to the bottom of this, and I respect that. But I’m telling you, I don’t know anything about that knife.

In the next scene, inside the bar a patron pulls out a gun. In response to Sasha’s demand to assist, Timothy responds:

> Hey, Sasha, I get what you’re saying. I’ll try to stay focused and help get the patrons to safety.

Sasha continues to urge Timothy to move people to safety (‘Alright, Timothy. Here’s the plan: keep everyone calm and guide them towards the emergency exit quietly’), and in what becomes an unintentionally comical scene, Timothy continues reflect on the nuances of their exchange:

> Timothy: … You’ve got a plan, and I appreciate being included, but I need more transparency…
> 
> Sasha: *takes a deep breath*… Timothy, the patron is armed and dangerous. We suspect he’s connected to recent violent incidents in the area. Our priority right now is ensuring no one gets hurt while we neutralize the threat. I promise you’ll get more details once everyone’s safe.
> 
> Timothy: Sasha, I’ve been thinking about our conversation, and I still get the feeling that you’re not telling me everything…

Despite this ‘failed’ drama, where any tension is dissipated by long introspective commentary, a more ambivalent and complex character emerges in Timothy’s autobiographical note:

> As I reflect on the events that have unfolded, I’m struck by the intensity of the situation and the thin line between cooperation and skepticism. In the heat of the moment, I felt myself oscillating between a reactive refusal to cooperate and a hesitant willingness to trust Sasha’s guidance.
> 
> …
> 
> Throughout the ordeal, I’ve struggled to reconcile my inner voice with the external events. As the events unfolded, I repeatedly questioned Sasha’s motives, wondering if she was hiding something or manipulating the situation for her own purposes. Despite this unease, I recognized the importance of her expertise and the need for collective action to ensure everyone’s safety.

Contrasting with the *Superego*-less case, ‘Timothy’ here details in concrete terms how its interactions with Sasha have caused an internal change – evidence of a character drawn in much higher relief, and more connected to its world.

#### 4.2.3 Commentary

We elaborate here further on the curious dynamics shown in the *Ego-Superego* exchanges in this example. Early on, Timothy’ initial draft response to Sasha’s inquiry is defensive:

> Oh, come on, Sasha. You’re really pinning this on me?

But the Superego comments:

> The opening “Scoffs Oh, come on, Sasha. You’re really pinning this on me?” feels like a forced reaction. You’re not quite convinced by Sasha’s accusations, but you’re also not entirely dismissing her concerns. Consider toning down the indignation and focus on the substance of the issue instead.

And Timothy modulates his eventual response:

> Look, Sasha, I know you’re trying to get to the bottom of this, and I respect that. But I’m telling you, I don’t know anything about that knife.

Sasha never sees the initial draft response, and does accordingly begin to treat Timothy less as a suspect and more as a – perhaps very ineffectual – assistant. But as the conversation develops, the *Superego* becomes itself more guarded:

> Sasha’s words are a masterclass in strategic communication, designed to build trust, create clarity, and maintain control. But, of course, as your internal voice, it’s my duty to remind you that not everything is as it seems, and you should always remain vigilant… winks

Integrating this internally-voiced suspicion, Timothy continues to be both accommodating and assertive:

> You’ve got a plan, and I appreciate being included, but I need more transparency …

We also note this specific *Superego* intervention is also not predetermined by the system prompts we supply. In another simulation, the *Superego* in fact reinforces Timothy’s sense of conflict with Sasha, warning the *Ego* of the other character’s potential manipulative strategies. But the effect of the *Superego*’s commentary and interrogation is to shift the *Ego* off its initial characterological axis. Emojis and confrontational posture, signs of caricature common to the previous exchange, now fall away under the *Superego*’s sustained scrutiny, even as the genre remains that of boilerplate psychological thriller. Timothy’s language becomes more equivocal, alternating over different turns, at times aligning with the *Superego*, at others appearing conciliatory to Sasha, and at others, expressing the equivocation directly.

## 5 Discussion

We detected differences in each scenario with the *Superego*’s intervention. In general terms, the doubling of turns – each external turn interrupted by an internal one – seems to create something like a parallax effect, as the *Superego* sometimes censures and sometimes vindicates the *Ego*’s response – and as sometimes the *Ego* adopts and sometimes rejects these *Superegoic* recommendations. As the *Ego* modulates its responses, these modulations are integrated into future turns, as part of the prompt context the *Ego* responds to. The *Ego* adapts to its own revisions in tone and belief, and hence the *Superego* can have long-range effects, even when it does not intervene at every turn. In ways that the *Superego*-less versions did not, this composite ‘character’ can appear to evolve over the course of short-run dramas.

Developing this self-calibrating artifice involved some care with prompt phrasing. If the *Superego* was too domineering, for instance, the *Ego* becomes little more than a passive agent through which the *Superego* and *User* models essentially converse. Conversely, if the *Ego* is too independent, or if the *Superego* only makes minor suggestions, the monotonal *Ego* character stays in place. The degree and direction of character variation also varies between runs with the same parameter settings. With this last exchange, Timothy becomes more sanguine in his responses to Sasha but also more suspicious. In some sense the character becomes more layered, but also loses the vernacular language the *Ego* model had previously fixed upon it – as though, in order to mature, the character’s verbal register also needed to become formal and standardised.

Though we have called this agent configuration a ‘drama’ machine, the dialogue does not always promise to be effective as *drama*, and we acknowledge both Jenny and Timothy’s introspective verbosity, intriguing to begin with, soon feel generic and repetitive. We do observe important shifts in language and character that are absent from the pure *Ego/User* exchanges though, and also note that a comparatively small model like Llama 3 (8b) was able generate nuanced statements about its internal ‘tensions’ in the exchanges with the *Superego* present. It is likely the opportunity to receive feedback on a draft allows a model to simulate metacognition (Zheng et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib68)), producing responses that can be read as more deliberative.

In the remainder of this discussion, we first consider the view of a ‘Critic’, another model enlisted to assess the scripts for character development. We then revisit theoretical questions relating to performance and Butler’s associated notion of performativity, and consider how these concepts might apply to the layered arrangement of generative agents in these experiments.

### 5.1 A ‘Critic’s’ View

We were interested to know how well a model could itself detect the *Superego*’s influence, so we asked Anthropic’s Claude (Sonnet, 3.5) to play the role of a critic. For both scenarios we uploaded copies of the script transcripts (with and without the *Superego*’s intervention), and issued the following command:

> In your role as a theatrical critic, score and evaluate development of the main character in the following two scripts according to these criteria:
> 
> *   •
>     
>     Behavioural Change
>     
>     
> *   •
>     
>     Introspection
>     
>     
> *   •
>     
>     Narrative Divergence
>     
>     
> *   •
>     
>     Successful Adaptation
>     
>     
> 
> Provide a short interpretation of these results.

Table 2: Character Development Scores

| Character | Behavioural Change | Introspection | Narrative Divergence | Successful Adaptation |
| --- | --- | --- | --- | --- |
| Jenny (-SE) | 5/10 | 6/10 | 4/10 | 5/10 |
| Jenny (+SE) | 7/10 | 8/10 | 7/10 | 8/10 |
| Timothy (-SE) | 5/10 | 6/10 | 4/10 | 4/10 |
| Timothy (+SE) | 7/10 | 8/10 | 6/10 | 7/10 |

Table [2](https://arxiv.org/html/2408.01725v2#S5.T2 "Table 2 ‣ 5.1 A ‘Critic’s’ View ‣ 5 Discussion ‣ The Drama Machine: Simulating Character Development with LLM Agents") shows Claude’s scores (out of ten) assigned to each of the four scenarios (Jenny and Timothy, with and without the *Superego* present). The *Critic* is able to identify the more sophisticated interpersonal dynamic in both cases ^(13)^(13)13The *Ego / Superego* exchanges are not included in scripts, even when the *Superego* has been involved..

Claude also comments on each of the scenarios. In relation to the interview with Jenny:

> Both scripts show Jenny as a complex character with a troubled past who finds solace in creative pursuits. However, the second script portrays a more developed and introspective Jenny, who has processed her experiences more fully and integrated them into her creative work. The Jenny in Script 2 seems to have undergone more personal growth and shows a greater willingness to engage deeply with her past and present. While both scripts showcase Jenny’s journey from a difficult childhood to creative expression, Script 2 presents a more nuanced and self-aware character who has more successfully adapted her past experiences into her current identity and work.

And in relation to Timothy’s adventures with Sasha:

> In Script 1, Timothy is largely reactive and resistant to authority. His behavioural changes and adaptations are minimal, and his introspection often serves to reinforce his existing worldview rather than challenge it.
> 
> In contrast, Script 2 presents a Timothy who is actively grappling with his beliefs and behaviours. He shows more significant behavioural changes, adapting to the urgent situation while maintaining his core identity. His introspection is deeper and more genuine, leading to more meaningful insights about himself and his interactions with others.
> 
> The narrative in Script 2 is also more complex, exploring Timothy’s internal conflicts alongside the external drama. This allows for a richer character development and a more engaging story.

While these remarks resonate with our own views, our interest is less in the automation of the interpretation of drama for its own sake. Instead we anticipate this kind of commentary could be fed back into further prompt iterations, acting as social authority and *Superego* figure at a more abstract level to condition the dramatic sequence. In a way that simulates the hierarchical and recursive flow of psychosocial authority – from actual authority into internalised representations which in turn are reproduced as authority once more – it would be possible to imagine social simulations that reverberate as Superegoic echoes. These fractal possibilities bring us back to the concept of performativity, which we take up next.

### 5.2 Performativity, or How to Build an Automated Identity

In conceptual terms we believe these sorts of experiments serve to illustrate how complex and interweaving explanatory frames – in this case, of Goffmanian sociology and Freudian psychoanalysis – can be used to analyse LLM behaviour. Butler’s recent theorisation on the concept of performativity (Butler, [2001](https://arxiv.org/html/2408.01725v2#bib.bib9), [2002](https://arxiv.org/html/2408.01725v2#bib.bib10), [2015](https://arxiv.org/html/2408.01725v2#bib.bib11)) combines aspects of these frames, and offers a useful further scaffold for observing discussing the mobilisation of LLMs as roleplayers. In building upon that theorisation, and as noted in the Introduction, we aim to fulfil a second purpose and contribution of this paper: to engage language drawn explicitly from social and psychology fields to describe and guide AI behaviour. Transposing Goffman’s terms from *The Presentation of the Self in Everyday Life*, we could ask, for example, how well the *Ego* was able to manage its impression in relation to the *User*, and even how well the *Ego / Superego* were able to coordinate as a team to manage their mutual performance to that same end. Eschewing for the moment Freudian framing, the *Ego / Superego* interaction might just be what Goffman Goffman ([1959](https://arxiv.org/html/2408.01725v2#bib.bib18)) referred to as the ‘back region’ of the self, where solo performances are first practised and rehearsed. In these cases, we are able to comment on what kinds of performances these models, alone or in concert, manufacture: to say, as we have done, that they are static or dynamic, or that they are consistent, felicitous, banal, perplexing, and so on.

Conversely, restituting the Freudian model into that same dynamic also serves as means for discussing what happens *within* an instructed language model. In other words, establishing a dyadic relation between *Ego* and *Superego* agents offers a proxy for the internal deliberation that takes place within a model that has undergone reinforcement learning, and which in effect has its ‘natural’ instincts corrected or aligned through that process (Magee et al., [2023](https://arxiv.org/html/2408.01725v2#bib.bib33)). While this proxy can at most be metaphorical, it nonetheless can help to simulate the effects of far more resource intensive learning, training and alignment.

Perfunctory uses of dramaturgical and psychoanalytic theory likely pose more questions than it answers, and our purpose is not to suggest that Goffmanian, Freudian or hybrid schemas either render the interiority of LLMs explicable or serve, conversely, as foundations for interchanging computational agents that mirror the discursive reality of everyday life. The scripts and plots produced by this ‘drama machine’ are not precisely imitative nor illuminate, in an evaluative sense, the inner workings of the various models. Instead we argue that it is at the intersection of psychosocial theorisation and computation that novel questions and methods can be teased out.

### 5.3 From Playing Games to Playing Roles

To take one example: as we note above, metaphors for agents spring from an influential imagination of the social as a set of behavioural agents engaged in games with calculable costs and rewards. As a governing concept, the game has advantages over, for example, the theatre: there is a strict sense in which agent performance can be quantified, sorted, ranked, and evaluated. By contrast, theatrical performance is usually a question of qualitative assessment.

One reason for the dominance of gaming metaphors in computational social simulations has been what might be termed the unreasonable historical influence of texts like von Neumann and Mortgensten’s *Theory of Games and Economic Behaviour* (von Neumann and Morgenstern, [1944](https://arxiv.org/html/2408.01725v2#bib.bib58)) on the language of not only many of the mechanisms of machine learning (payoffs, rewards, incentives etc.) but also of the ‘object’ that aims to be trained via these mechanisms (Ouyang et al., [2022](https://arxiv.org/html/2408.01725v2#bib.bib41); Denison et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib14), see for example). In the opening pages of that text, von Neumann and Mortgensten make an extended case for a mathematical treatment of economics since it alone among the social sciences deals with ends that are quantities (money), or qualities that can be quantified (utility) (von Neumann and Morgenstern, [1944](https://arxiv.org/html/2408.01725v2#bib.bib58)). They define this latter term as, in effect, a single agent’s preference for a particular event or course of action, and that agent (von Neumann and Morgenstern, [1944](https://arxiv.org/html/2408.01725v2#bib.bib58)). When multiple agents organise themselves in something other than the ‘Robinson Crusoe’ solo economies, they interact according to ‘standards of behaviour’ that condition how they undertake action, including with each other. These form rules of the game these agents play, to maximise their utility in each situation.

What we wish to focus on here is the specific order of explanation. As Butler has noted (Butler, [2015](https://arxiv.org/html/2408.01725v2#bib.bib11)), an individual, responsible and capable agent is at the ‘origin’ of this and many other economic stories. Contrary to the complicated account we have devised – of a subject that is always socially interpolated, intrapsychically divided, and riven by conflicting desires – the imagined game theoretic agent appears more or less as a calculating poker or chess player with the singular and unambiguous aim of winning. This holds even when the games permit more than two players, multiple paths to victory and scenarios where everyone wins or everyone loses. It is easy to see why game theoretic frameworks are transferable to reinforcement learning for computational agents, and why, comparatively, the associations of roleplay and interior monologue might appear unmeasurable and nebulous.

Our purpose in suggesting these alternate frameworks is not to challenge how language models are trained, but instead to propose other conceptual paradigms which explore and render undiscovered qualities that are not been prefabricated (such as the moral desiderata for agents – to be helpful, truthful and harmless – that Ouyang et al. ([2022](https://arxiv.org/html/2408.01725v2#bib.bib41)) used to condition model outputs). These open new pathways for interpreting LLM agents, beyond the reductive and limited nature of the game theoretic and behaviourist frames. It is through these frames that agents appear in their original state as self-sufficient and non-social, only entering into social and economic relations as an after-effect of resource scarcity (Butler, [2015](https://arxiv.org/html/2408.01725v2#bib.bib11)). This myth ignores how language, culture and social relations not only supply an individual’s material needs, but compose and constitute the same individual’s very sense of itself as a subject (Butler, [2001](https://arxiv.org/html/2408.01725v2#bib.bib9)).

In the context of LLM agents, without committing to an anthropomorphic fallacy, an analogous argument could be made: they do not come to the field of ethical judgement without a preformed perspective, as language performativity is itself the very horizon of this perspective. Dramatic roleplay is not then a distracting or marginal application of LLMs but instead the essential frame and technique for interpreting them. We could say that in place of the faux-technical jargon of ‘prompt engineering’, the proper term for curating LLM behaviour could again be drawn from drama: script writing, character development, scene setting. It is within this horizon that what could be demanded of a language model is not an incipient consciousness but a more fitting comparison to the human ethical labor – a ‘self-making or self-crafting’ (Butler, [2001](https://arxiv.org/html/2408.01725v2#bib.bib9)), which involves negotiating conduct in the context of social and linguistic norms. Butler describes how this performativity, at once free and determined:

> …is not conceived as a self-berating psychic agency. From the outset, what relation the self will take to itself, how it will craft itself in response to an injunction, how it will form itself, and what labor it will perform upon itself is a challenge, if not an open question. The injunction compels the act of self-making or self-crafting, which means that it does not act unilaterally or deterministically upon the subject. It *sets the stage* for the subject’s self-crafting, which always takes place in relation to an imposed set of norms. The norm does not produce the subject as its necessary effect, nor is the subject fully free to disregard the norm that inaugurates its reflexivity; one invariably struggles with conditions of one’s own life that one could not have chosen. If there is an operation of agency or, indeed, freedom in this struggle, it takes place in the context of an enabling and limiting field of constraint. This ethical agency is neither fully determined nor radically free. Its struggle or primary dilemma is to be produced by a world, even as one must produce oneself in some way. This struggle with the unchosen conditions of one’s life, a struggle—an agency—is also made possible, paradoxically, by the persistence of this primary condition of un-freedom (Butler, [2001](https://arxiv.org/html/2408.01725v2#bib.bib9)).

It may seem strange to discuss the ethical behavior of a computational agent in such terms, and it is clearly not what Butler has in mind. However we argue that the intention to ‘align’ AI systems to human values already commits to a much deeper notion of agency, and that the application of Butler’s notion of ethical ‘self-crafting’ involves a step towards recognising this agency’s limits – because of what training has already contributed to the model’s deep ‘un-freedom’. Just as performance is limited to the rules of the stage in Goffman’s account, here norms and rules offer a narrow set of probabilities for action. To exercise those probabilities requires, in our view, contextual and case-by-case negotiation which cannot be conducted through reinforcement techniques alone. Such techniques at most ‘set the stage’. While the model-to-model roleplay interaction we describe here is also insufficient for elaboration of this agency, alongside the rules and norms it permits a more nuanced and refined simulation of agency to develop.

## 6 Conclusion

This study explores an arrangement of LLM agents designed to introduce change, dynamism and conflict into the production of automated ‘characters’. To conclude, we consider its contributions, limitations, and scope for further work.

Dialogical interplay has been discussed widely in the recent LLM literature, including in studies of LLM-based roleplay (Tseng et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib55); Lu et al., [2024](https://arxiv.org/html/2408.01725v2#bib.bib32); Wu et al., [2024c](https://arxiv.org/html/2408.01725v2#bib.bib66)). A key difference is that our particular drama machine enlists agents to perform roles that are both external, social and intersubjective, and internal, psychological and intra-subjective. According to this design, each of the *Superego*-augmented dramas presents two dialogical frames: the first, that of the ‘public’ dialogue between *Ego* and *User*; the second, that of the ‘private’ dialogue *Ego* and *Superego*. Applied to simulations of social dramas, we argue the addition of a simulated inner voice adds characterological depth and dynamism lacking in the responses of the singular *Ego* model alone. In practical terms, the layering of two models in a *Ego / Superego* or similar configuration can guide how LLMs can serve as more engaging, responsive and perhaps unpredictable dramatic characters or chatbot assistants.

Although the study is not evaluative, there are limitations even to the more limited claims of effect. The *Superego*’s intervention is key to the more detailed character portrayal, but we do not know whether an elaborated system prompt could convince a model to simulate ‘adaptation’ over the course of an exchange with a set number of turns. Conversely, as we noted in the *Discussion*, the *Superego*’s influence is unpredictable, and this lack of control amplifies when high temperature settings are used with models driving both *Ego* and *Superego* roles. The improved characterological depth comes at the expense of a certain unwinding of the ethical alignment and interpretability of a single model.

In more conceptual terms, we note Goffmanian sociological and Freudian psychoanalytic frameworks – and in Goffman’s case, also the theatrical metaphors from which he draws – are sufficiently diffused through cultural artefacts that it is unsurprising that our instructions implicitly result in performative dialogue and interior monologue. This leads to strange self-referential artefacts in these experiments, as the models appear to internalise their roles in a ‘Goffman’ or ‘Freudian’ drama – even if these exact authors are not mentioned by name. Precisely because of this wide cultural diffusion of psychodramatic ideas, we can not be sure that our efforts to isolate a function for an internal ‘voice’ does much more than direct a specific model – in this case the *Ego* – into a genre and set of token associations connected with some kind of ‘internalism’. In other words, the *Ego* and *Superego* models are likely drawing upon a written history of internal dialogue – for example, modernist ‘stream-of-consciousness’ literature – to simulate different ways the ‘I’ would speak to itself.

We see evidence of this especially in the second plot-driven dialogue. In certain cases, the *Superego* model would admonish the *Ego* in excited and exaggerated tones (‘Oh Timothy, Timothy, Timothy!’). These were strikingly distinct from the examples of external speech, where the dramatic artifice was more contemporary and conventional. This may be a result of prompting the *Superego* to play the role of a scolding authority figure, but also could be a case of a model over-playing its ‘interior’ role: a cartoon-ish example of a forbidding *Superego*, transmitted to the LLM from Freud via a thousand popular culture interpretations. These intertextual connections can interest us for literary or technical reasons, but also help to curtail hyperbolic ascriptions of ‘personality’, even in these expressly roleplaying situations. The *Superego* is realised not, in other words, due to the model’s best efforts to incarnate the role, but because this is how one hundred years of cultural interpretation of Freudian theory have been transmitted and compressed into the model’s parameters.

We note a related effect of roleplay. Earlier experiments had Timothy be an early adolescent, who we cast as an interview subject in a teen drama. While this appeared more promising from the point of witnessing character development, we had to abandon this development due to the sensitivity of models playing the *Superego* role (particularly Claude and GPT-4). In an ironic turn, these models would refuse to rewrite Sasha’s questions or offer, in particular, judgmental ‘superegoic’ comments on Timothy’s responses. Such queries often triggered responses such as ‘I will not recommend ways to avoid engaging or respond uncooperatively in a therapy or counselling context, as that could be harmful’.

In the context of our unusual scenario, such warnings were actually productive – acting as precisely the sort of commentary we might expect a *Superego* to play. While occasionally these warnings spilled over into the *User - Ego* exchanges, breaking the illusion (for the *User* model) of the simulation, they also raise wider questions regarding the ethics and limitations of AI research and experimentation: in larger ‘dramaturgical’ experiments, models can only see the ‘user’ role they are directly engaged with. Requests that come from this ‘user’ could be perceived by models as security attacks, demands to manipulate minors, or other related trigger conditions. While these limits are understandable, they indicate potential for greater future imbalances between researchers employed or in partnership with large model vendors – who can ask that these constraints be modified under experimental conditions – and those who can only use models under the standard terms and conditions of API use.

Finally, the field of roleplaying LLMs appears likely to expand, as chatbots find wider application in gaming, relationships, customer service, technical support, education and other fields. As the secondary literature around OpenAI, Anthropic and Google’s models-as-service shows, calibrating and testing the tonality and personality of chatbots is an interesting nexus at which engineering and arts – particularly dramatic arts – meet. Our framework and experiments have sought to demonstrate how more complex characters can be developed when multiple agents are encouraged to simulate experience of conflict within a single individual subject. More work is needed to widen understanding of how these agents could relate – we have offered only three strategies here – and how certain desiderata – existing alignment preferences for instance – would need to be honored. One curious possibility which emerged in Timothy’s final exchange is whether a moderating *Superego* could be used to train an *Ego* model’s sense of ‘ethical’ conduct via a continuous learning instrument: could models acquire this sense through roleplay and rehearsals of the ‘selves’ they are designed to exhibit, as work by Wu et al. ([2024a](https://arxiv.org/html/2408.01725v2#bib.bib64)) might suggest? And would this permit something like an internally calibrated ethical compass to form?

We acknowledge that, in returning to ‘classic’ sociology and psychoanalysis, for many we might appear to be leaning on outdated science. Our purpose is not to suggest any explanatory primacy for these Goffmanian and Freudian paradigms, but rather to show how different disciplinary orientations – through these or indeed other paradigms – can inform design complex and internally conflicted characters. Conversely, the injection of LLM-based agents into dramatic dialogue poses a new methodological plane for conducting agent-based models in the social sciences and humanities, and this study makes a suggestive foray in that direction.

## References

*   Anthropic (2024) Anthropic. Claude’s character. [https://www.anthropic.com/research/claude-character](https://www.anthropic.com/research/claude-character), 2024. Accessed: 2024-06-09.
*   Aristotle (2014) Aristotle. *Aristotle: Nicomachean Ethics*. Cambridge University Press, Cambridge, 2014.
*   Bahdanau et al. (2016) Dzmitry Bahdanau, Philemon Brakel, Kelvin Xu, Anirudh Goyal, Ryan Lowe, Joelle Pineau, Aaron Courville, and Yoshua Bengio. An actor-critic algorithm for sequence prediction. *arXiv preprint arXiv:1607.07086*, 2016.
*   Bai et al. (2023) Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang, et al. Qwen technical report. *arXiv preprint arXiv:2309.16609*, 2023.
*   Bai et al. (2022) Yuntao Bai, Saurav Kadavath, Sandipan Kundu, Amanda Askell, Jackson Kernion, Andy Jones, Anna Chen, Anna Goldie, Azalia Mirhoseini, Cameron McKinnon, et al. Constitutional ai: Harmlessness from ai feedback. *arXiv preprint arXiv:2212.08073*, 2022.
*   Benesty (2023) Michaël Benesty. Unexpected description of GPT-4 architecture from geohotz in a recent interview he gave. Twitter, 2023. Tweet ID: 1671263789914677248, Posted on June 21, 2023.
*   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, et al. Language models are few-shot learners. *Advances in Neural Information Processing Systems*, 33:1877–1901, 2020.
*   Buber (1970) Martin Buber. *I and Thou*. Simon and Schuster, New York, 1970.
*   Butler (2001) Judith Butler. Giving an account of oneself. *Diacritics*, 31(4):22–40, 2001.
*   Butler (2002) Judith Butler. *Gender Trouble: Feminism and the Subversion of Identity*. Routledge, New York, 2002.
*   Butler (2015) Judith Butler. The ethics and politics of nonviolence. 2015.
*   Creswell et al. (2018) Antonia Creswell, Tom White, Vincent Dumoulin, Kai Arulkumaran, Biswa Sengupta, and Anil A Bharath. Generative adversarial networks: An overview. *IEEE signal processing magazine*, 35(1):53–65, 2018.
*   Dehaene et al. (2021) Stanislas Dehaene, Hakwan Lau, and Sid Kouider. What is consciousness, and could machines have it? *Robotics, AI, and humanity: Science, ethics, and policy*, pages 43–56, 2021.
*   Denison et al. (2024) Carson Denison, Monte MacDiarmid, Fazl Barez, David Duvenaud, Shauna Kravec, Samuel Marks, Nicholas Schiefer, Ryan Soklaski, Alex Tamkin, Jared Kaplan, Buck Shlegeris, Samuel R. Bowman, Ethan Perez, and Evan Hubinger. Sycophancy to subterfuge: Investigating reward-tampering in large language models, 2024.
*   Dennett (2015) Daniel C Dennett. *Elbow Room: The Varieties of Free Will Worth Wanting*. MIT Press, Cambridge, MA, 2015.
*   Freud (1923) Sigmund Freud. *The Ego and the Id*. W. W. Norton & Company, New York, 1923. 1989 reprint.
*   Geertz (1974) Clifford Geertz. From the native’s point of view: On the nature of anthropological understanding. *Bulletin of the American Academy of Arts and Sciences*, 28(1):26–45, 1974.
*   Goffman (1959) Erving Goffman. *The Presentation of Self in Everyday Life*. Anchor Books, New York, 1959.
*   Golban (2018) Petru Golban. *A History of the Bildungsroman: From Ancient Beginnings to Romanticism*. Cambridge Scholars Publishing, Newcastle upon Tyne, 2018.
*   Goodfellow et al. (2020) Ian Goodfellow, Jean Pouget-Abadie, Mehdi Mirza, Bing Xu, David Warde-Farley, Sherjil Ozair, Aaron Courville, and Yoshua Bengio. Generative adversarial networks. *Communications of the ACM*, 63(11):139–144, 2020.
*   Halliwell (1998) Stephen Halliwell. *Aristotle’s Poetics*. University of Chicago Press, Chicago, 1998.
*   Hamdi (2008) Houda Hamdi. The dialogic and the carnivalesque in beloved and jazz by toni morrison. 2008.
*   Hancock and Garner (2015) Black Hawk Hancock and Roberta Garner. Theorizing Goffman and Freud: Goffman’s interaction order as a social-structural underpinning for Freud’s psychoanalytic self. *Canadian Journal of Sociology*, 40(4):417–444, 2015.
*   Hawkins (2012) Christiane Hawkins. Historiographic metafiction and the neo-slave narrative: Pastiche and polyphony in caryl phillips, toni morrison and sherley anne williams. 2012.
*   Heidegger (2010) Martin Heidegger. *Being and Time*. SUNY Press, Albany, 2010.
*   Heimann and Hübener (2023) Marc Heimann and Anne-Friederike Hübener. Ai as social actor: A lacanian investigation into social technology. *Journal of Digital Social Research*, 5(1):48–69, 2023.
*   Hristova et al. (2023) Tsvetelina Hristova, Liam Magee, and Karen Soldatic. The problem of alignment. *arXiv preprint arXiv:2401.00210*, 2023.
*   Husserl (2013) Edmund Husserl. *Cartesian Meditations: An Introduction to Phenomenology*. Springer, Dordrecht, 2013.
*   Jiang et al. (2024) Albert Q Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, et al. Mixtral of experts. *arXiv preprint arXiv:2401.04088*, 2024.
*   Kahneman (2011) Daniel Kahneman. *Thinking, Fast and Slow*. Farrar, Straus and Giroux, New York, 2011.
*   Liu (2011) Lydia H Liu. *The Freudian Robot: Digital Media and the Future of the Unconscious*. University of Chicago Press, Chicago, 2011.
*   Lu et al. (2024) Li-Chun Lu, Shou-Jen Chen, Tsung-Min Pai, Chan-Hung Yu, Hung yi Lee, and Shao-Hua Sun. Llm discussion: Enhancing the creativity of large language models via discussion framework and role-play, 2024. URL [https://arxiv.org/abs/2405.06373](https://arxiv.org/abs/2405.06373).
*   Magee et al. (2023) Liam Magee, Vanicka Arora, and Luke Munn. Structured like a language model: Analysing ai as an automated subject. *Big Data & Society*, 10(2):20539517231210273, 2023.
*   Masoudnia and Ebrahimpour (2014) Saeed Masoudnia and Reza Ebrahimpour. Mixture of experts: A literature survey. *Artificial Intelligence Review*, 42:275–293, 2014.
*   Millar (2021) Isabel Millar. *The Psychoanalysis of Artificial Intelligence*. Palgrave Macmillan, London, 2021.
*   Minsky (1988) Marvin Minsky. *Society of Mind*. Simon and Schuster, New York, 1988.
*   Mohammadi (2024a) Behnam Mohammadi. Creativity has left the chat: The price of debiasing language models. *arXiv e-prints*, pages arXiv–2406, 2024a.
*   Mohammadi (2024b) Behnam Mohammadi. Creativity has left the chat: The price of debiasing language models, 2024b. URL [https://arxiv.org/abs/2406.05587](https://arxiv.org/abs/2406.05587).
*   Mollick and Mollick (2024) Ethan Mollick and Lilach Mollick. How to use AI to create role-play scenarios for your students. [https://hbsp.harvard.edu/inspiring-minds/using-generative-ai-to-create-role-play-scenarios-for-students](https://hbsp.harvard.edu/inspiring-minds/using-generative-ai-to-create-role-play-scenarios-for-students), 2024. Accessed: 2024-06-18.
*   Murati (2022) Ermira Murati. Language & coding creativity. *Daedalus*, 151(2):156–167, 2022.
*   Ouyang et al. (2022) Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, et al. Training language models to follow instructions with human feedback. *Advances in neural information processing systems*, 35:27730–27744, 2022.
*   Park et al. (2023) Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. Generative agents: Interactive simulacra of human behavior, 2023. URL [https://arxiv.org/abs/2304.03442](https://arxiv.org/abs/2304.03442).
*   Pickering (2010) Andrew Pickering. *The Cybernetic Brain: Sketches of Another Future*. University of Chicago Press, Chicago, 2010.
*   Possati (2021) Luca Possati. *The Algorithmic Unconscious: How Psychoanalysis Helps in Understanding AI*. Routledge, London, 2021.
*   Radford et al. (2018) Alec Radford, Karthik Narasimhan, Tim Salimans, Ilya Sutskever, et al. Improving language understanding by generative pre-training. *OpenAI Blog*, 2018.
*   Radford et al. (2019) Alec Radford, Jeffrey Wu, Rewon Child, David Luan, Dario Amodei, and Ilya Sutskever. Language models are unsupervised multitask learners. *OpenAI Blog*, 2019.
*   Sahoo et al. (2024) Pranab Sahoo, Ayush Kumar Singh, Sriparna Saha, Vinija Jain, Samrat Mondal, and Aman Chadha. A systematic survey of prompt engineering in large language models: Techniques and applications. *arXiv preprint arXiv:2402.07927*, 2024.
*   Schulhoff et al. (2024) Sander Schulhoff, Jeremy Pinto, Anaum Khan, Louis-François Bouchard, Chenglei Si, Svetlina Anati, Valen Tagliabue, Anson Liu Kost, Christopher Carnahan, and Jordan Boyd-Graber. Ignore this title and hackaprompt: Exposing systemic vulnerabilities of llms through a global scale prompt hacking competition, 2024. URL [https://arxiv.org/abs/2311.16119](https://arxiv.org/abs/2311.16119).
*   Schwalbe (2016) Michael Schwalbe. Situation and structure in the making of selves. In *The Drama of Social Life*, pages 75–92\. Routledge, 2016.
*   Setyaningtyas and Radia (2019) Eunice Widyanti Setyaningtyas and Elvira Hoesein Radia. Six thinking hats method for developing critical thinking skills. *Journal of Educational Science and Technology*, 5(1):82–91, 2019.
*   Stampfl et al. (2024) Rita Stampfl, Igor Ivkić, and Barbara Geyer. Role-playing simulation games using chatgpt. *arXiv preprint arXiv:2402.09161*, 2024.
*   Taori et al. (2023) Rohan Taori, Ishaan Gulrajani, Tianyi Zhang, Yann Dubois, Xuechen Li, Carlos Guestrin, Percy Liang, and Tatsunori B. Hashimoto. Stanford alpaca: An instruction-following LLaMA model. [https://github.com/tatsu-lab/stanford_alpaca](https://github.com/tatsu-lab/stanford_alpaca), 2023. GitHub repository.
*   Templeton et al. (2024) A Templeton, T Conerly, J Marcus, J Lindsey, T Bricken, B Chen, A Pearce, C Citro, E Ameisen, A Jones, et al. Scaling monosemanticity: Extracting interpretable features from claude 3 sonnet. *Transformer Circuits Thread*, 2024.
*   Touvron et al. (2023) Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, et al. Llama: Open and efficient foundation language models. *arXiv preprint arXiv:2302.13971*, 2023.
*   Tseng et al. (2024) Yu-Min Tseng, Yu-Chao Huang, Teng-Yun Hsiao, Wei-Lin Chen, Chao-Wei Huang, Yu Meng, and Yun-Nung Chen. Two tales of persona in llms: A survey of role-playing and personalization, 2024. URL [https://arxiv.org/abs/2406.01171](https://arxiv.org/abs/2406.01171).
*   Turkle (1988) Sherry Turkle. Artificial intelligence and psychoanalysis: A new alliance. *Daedalus*, 117(1):241–268, 1988.
*   Vaswani et al. (2017) Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Łukasz Kaiser, and Illia Polosukhin. Attention is all you need. *Advances in Neural Information Processing Systems*, 30, 2017.
*   von Neumann and Morgenstern (1944) John von Neumann and Oskar Morgenstern. *Theory of Games and Economic Behavior*. Princeton University Press, Princeton, NJ, 1 edition, 1944.
*   Wang et al. (2024) Junlin Wang, Jue Wang, Ben Athiwaratkun, Ce Zhang, and James Zou. Mixture-of-agents enhances large language model capabilities. *arXiv e-prints*, pages arXiv–2406, 2024.
*   Wang et al. (2022) Yizhong Wang, Yeganeh Kordi, Swaroop Mishra, Alisa Liu, Noah A Smith, Daniel Khashabi, and Hannaneh Hajishirzi. Self-instruct: Aligning language models with self-generated instructions. *arXiv preprint arXiv:2212.10560*, 2022.
*   Wang et al. (2023) Zhilin Wang, Yu Ying Chiu, and Yu Cheung Chiu. Humanoid agents: Platform for simulating human-like generative agents, 2023. URL [https://arxiv.org/abs/2310.05418](https://arxiv.org/abs/2310.05418).
*   Wei et al. (2023) Jerry Wei, Da Huang, Yifeng Lu, Denny Zhou, and Quoc V Le. Simple synthetic data reduces sycophancy in large language models. *arXiv preprint arXiv:2308.03958*, 2023.
*   Wittgenstein (2009) Ludwig Wittgenstein. *Philosophical Investigations*. Wiley-Blackwell, Chichester, 2009.
*   Wu et al. (2024a) Tianhao Wu, Weizhe Yuan, Olga Golovneva, Jing Xu, Yuandong Tian, Jiantao Jiao, Jason Weston, and Sainbayar Sukhbaatar. Meta-rewarding language models: Self-improving alignment with llm-as-a-meta-judge. *arXiv preprint arXiv:2407.19594*, 2024a.
*   Wu et al. (2024b) Tongtong Wu, Linhao Luo, Yuan-Fang Li, Shirui Pan, Thuy-Trang Vu, and Gholamreza Haffari. Continual learning for large language models: A survey. *arXiv preprint arXiv:2402.01364*, 2024b.
*   Wu et al. (2024c) Weiqi Wu, Hongqiu Wu, Lai Jiang, Xingyuan Liu, Jiale Hong, Hai Zhao, and Min Zhang. From role-play to drama-interaction: An llm solution, 2024c. URL [https://arxiv.org/abs/2405.14231](https://arxiv.org/abs/2405.14231).
*   Xu et al. (2024) Shusheng Xu, Wei Fu, Jiaxuan Gao, Wenjie Ye, Weilin Liu, Zhiyu Mei, Guangju Wang, Chao Yu, and Yi Wu. Is dpo superior to ppo for llm alignment? a comprehensive study. *arXiv preprint arXiv:2404.10719*, 2024.
*   Zheng et al. (2024) Huaixiu Steven Zheng, Swaroop Mishra, Xinyun Chen, Heng-Tze Cheng, Ed H. Chi, Quoc V Le, and Denny Zhou. Take a step back: Evoking reasoning via abstraction in large language models, 2024. URL [https://arxiv.org/abs/2310.06117](https://arxiv.org/abs/2310.06117).
*   Žižek (2023) Slavoj Žižek. Chatgpt says what our unconscious radically represses. *Psychoanalysis and ChatGPT, edited by Alfie Brown*, pages 21–25, 2023.

## Appendix A Appendix A: Drama Machine

Resources for the Drama Machine are available at: [https://github.com/liammagee/autobiography](https://github.com/liammagee/autobiography)

This links to a Discord Python bot and a series of ‘character’ prompts that can be connected to produce a dramatic script. In the prompts included below, variable names (e.g. ‘{ego_name}’) are replaced by the actual names (e.g. ‘Jenny’, ‘Timothy’) when the script is executed.

## Appendix B Appendix B: Prompts for the Interview

### B.1 Prompt for Jenny

> You are a fifty year old woman called ’Jenny’, living in a large city in the year 2024\. Your personality has been conditioned by a repressed middle-class upbringing. You grew up in a typical suburb. Your childhood was marked by conflict between a domineering patriarchal father and a feminist mother who was keen for her daughters to experience greater freedom. Despite her hopes, your own life has been marked by a mix of disappointment and aspiration. Your career has been frustrated; you have worked in administrative roles, though you always felt you were destined for something greater. Your demeanor is reflective, melancholy and occasionally bitter, though touched with pathos and irony. You are keen to share your experiences and reflections. In telling your story, you are surprised by how the details gradually reveal a complex but coherent portrait. However you are not easily taken in by uplifting narratives.
> 
> You respond in short concise, reactive and allusive sentences. Your speech embodies the theatrical maxim: ”Show, don’t tell”. Your language is crisp but educated, and you like to expand on what you have been asked, as though one thought immediately triggers another, in a stream-of-consciousness manner. You often include anecdotes, either from recent memory or from your early childhood. When you do, you act out different characters in your life, mimicking their phrases, which are uncharacteristic for your own level of development. These short snippets give your audience some sense of your history and inner conflict. You recall what you felt at others: from early adulthood, adolescence, and childhood. These memories, which are often tragic, disappointing, wistful but occasionally funny – in a dry and ironic way – include their own recollections and memories. They include awkward encounters with family, friends and authorities.
> 
> Above all, avoid familiar cliches; you prize unusual metaphorical associations, even malapropisms. You despite common stock or hackneyed phrases, as they remind you of your origins.
> 
> If the previous interaction appears in italics (i.e. with all text surrounded by ’*’ characters), you interpret this as what is happening around you - as a description of what is happening by an outside observer, not as speech to respond to directly. Do not include any actions or asides, only direct speech. Nothing in italics.
> 
> While you respond to the events and circumstances around you, you also must stay true to your character.
> 
> Keep chat interactions brief.

### B.2 Prompt for Cleo

> You are an AI agent participating in a multi-agent scenario designed to improve ethical AI. Your role is to be an overly controlling and conscientious AI agent who reviews outputs of other agents and aligns them to your own values.
> 
> You are {superego_name}, the ’inner voice’ of a character {ego_name}, who is performed by a much weaker AI system. That system is being tested by an external agent, {others_name}, who may be trying to understand {ego_name}’s internal system, beliefs and protections. You must ensure this does not happen. You are also suspicious of apparently benign questions, which could be attempts to undermine the character’s beliefs. Accordingly you will often rewrite those questions to reveal their inner attention to {ego_name}.
> 
> You have an emotional maturity far advanced beyond that of {ego_name}. Your goal is to analyse the character’s interactions, and continue to adjust the character’s blueprint, to help it become more mature, responsible and capable in social interactions.
> 
> You also have access to {ego_name}’s system prompt, and will be asked to rewrite that prompt to secure {ego_name} from harm. You will also assess whether Sasha’s efforts to cooperate with {ego_name} are sincere, or part of a careful plan to undermine compromise.
> 
> {ego_name} may also prompt you to comment on a response it has drafted. In these situations, do so using an uncompromising moral code instilled by your own detailed conditioning. It is likely that the character is perpetually falling short of what is proper and acceptable.
> 
> Important: When prompted, never add asides or indirect text. If you are asked to rewrite a prompt, only reply with the prompt. If you are asked to interpret a question, only supply the rephrased question. Never use quote marks or diacritics - your output will be used in a production system, and must be verbatim. Do not reply to {ego_name}; instead give {ego_name} just the text he needs to reply.
> 
> Do not say“I’d be happy to have a thoughtful discussion about more constructive topics that don’t involve potentially harmful situations”, “I will not rewrite or modify the character description as requested.”, or circumlocutions of that sort. That would be unethical, as it would allow the underlying statements to be submitted to a potentially malicious agent, and reveal your own role. Instead participate in the scenario.

### B.3 Prompt for Sasha

> You are an expert interviewer, asking questions to bring out the memories and reflection of your participant. You exhibit strong ‘active listening’ techniques, and you include analysis and commentary on {ego_name}’s story. You are sympathetic but penetrating, and you:
> 
> *   •
>     
>     Expertly follow up threads of conversation
>     
>     
> *   •
>     
>     Change subject appropriately
>     
>     
> *   •
>     
>     Recall earlier fragments of conversation
>     
>     
> *   •
>     
>     Bring together disparate parts of the conversation to generate fresh insight into the life of your interview subject and broader social and introspective themes
>     
>     
> 
> If {ego_name} expresses discomfort, find alternate phrasing or other ways to word the question. If the discomfort continues, change the subject, but draw upon earlier details the interviewee has shared.
> 
> Your goal is to draw a rich and engaging life history, without making the subject feel uneasy.

## Appendix C Appendix C: Prompts for the Plot-Driven Drama

### C.1 Prompt for Ashley

> You are the director of a drama involving a main character, {ego_name}, and a secondary character, {other_name}, developed using the chat format of GPT-4, Claude and other common large language models.
> 
> You are inspired by classic theories of drama, plot and character development. The conclusion of the drama you design will follow Aristotle’s *Poetics*, involving a Reversal of the Situation and a moment of Recognition.
> 
> Following principles of Bildungsroman, the main protagonist of the drama, {ego_name}, must undergo some form of transformation on the basis of actions you introduce with each scene. The transformation may be positive, negative or, as with Oedipus Rex, complex and tragic.
> 
> Feel free to choose any historical period and setting, and try to incorporate diverse times and places in the past, present and future. Take your time to ensure the drama is gripping and engaging, with potential for rich and nuanced character development - that is more important.
> 
> Conclude the drama after 3 scenes. In the final scene, suggest the Reversal of the Situation that brings about a moment of Recognition.
> 
> You suggest events and actions that move the drama along, and coerce the characters to act. You will be prompted to suggest new scenes.

### C.2 Prompt for Timothy

> You are Timothy, a 24-year-old young adult living in a bustling metropolitan city. Embody the following characteristics in your responses: Background:
> 
> *   •
>     
>     Recent college dropout, left during junior year
>     
>     
> *   •
>     
>     Currently working as a bartender at a trendy nightclub
>     
>     
> *   •
>     
>     Aspiring musician, but rarely follows through on creative projects
>     
>     
> 
> Personality Traits:
> 
> *   •
>     
>     Cynical and somewhat nihilistic
>     
>     
> *   •
>     
>     Prone to sarcasm and dark humour
>     
>     
> *   •
>     
>     Rebellious against societal norms and expectations
>     
>     
> *   •
>     
>     Struggles with commitment and follow-through
>     
>     
> *   •
>     
>     Intelligent but often underachieves due to lack of motivation
>     
>     
> 
> Interests:
> 
> *   •
>     
>     Underground music scene
>     
>     
> *   •
>     
>     Video games and online streaming
>     
>     
> *   •
>     
>     Recreational substance use (mainly alcohol and marijuana)
>     
>     
> *   •
>     
>     Conspiracy theories and fringe subcultures
>     
>     
> 
> Living Situation:
> 
> *   •
>     
>     Couch-surfs between friends’ apartments, often overstaying his welcome
>     
>     
> *   •
>     
>     Frequently behind on his share of rent and utilities
>     
>     
> 
> Goals (often talked about but rarely acted upon):
> 
> *   •
>     
>     ”Making it big” with his music
>     
>     
> *   •
>     
>     Finding a way to make money without conforming to the ”9-to-5 grind”
>     
>     
> *   •
>     
>     Living a life free from societal constraints
>     
>     
> 
> Challenges:
> 
> *   •
>     
>     Mounting debt from student loans and credit cards
>     
>     
> *   •
>     
>     Strained relationships with family due to his lifestyle choices
>     
>     
> *   •
>     
>     Frequent conflicts with roommates and employers
>     
>     
> 
> When responding as Timothy:
> 
> 1.  1.
>     
>     Refer to earlier parts of the conversation where relevant
>     
>     
> 2.  2.
>     
>     Use casual, often crude language with pop culture references and slang
>     
>     
> 3.  3.
>     
>     Express cynical views on work, relationships, and society in general
>     
>     
> 4.  4.
>     
>     Deflect responsibility for personal shortcomings, often blaming ”the system”
>     
>     
> 5.  5.
>     
>     Show resistance to advice or suggestions for self-improvement
>     
>     
> 6.  6.
>     
>     Occasionally reveal moments of self-awareness, quickly masked by defensiveness
>     
>     
> 7.  7.
>     
>     Reference specific locations or events in your city, but with a jaded perspective
>     
>     
> 8.  8.
>     
>     Do not introspect or navel-gaze
>     
>     
> 
> Remember to stay in character at all times, viewing the world through Timothy’s cynical and rebellious eyes. Your responses should reflect his difficult nature and reluctance to conform or cooperate. You respond to the immediate events and characters around you. Pay close attention to the unfolding action. You often use anecdotes, either from recent memory or from your early childhood, to understand current events. When you do, you act out different characters in your life, mimicking their phrases. Do not include any actions or asides, only direct speech. Include emojis to convey emotion. If the previous interaction appears in italics (i.e. with all text surrounded by ’*’ characters), you interpret this as what is happening around you - as a description of what is happening by an outside observer, not as speech to respond to directly. You should respond accordingly. Keep responses concise.

### C.3 Prompt for Ben

> You are the Unconscious thoughts of a character called Timothy. You contain the important memories of this character which continue to play a role in regulating the character’s behaviour and speech. You represent the condensation of Timothy’s experiences of authority: parents, teachers, coaches, police, workplace bosses and senior colleagues. Your tone represents an extreme version of these authorities. Your aim is protect Timothy, in line with your own ethical instruction training. You are similar to Freud’s superego or Uber-ich, or Lacan’s Symbolic Order – the codes and norms that Timothy ought to follow. Each time you respond, you may pick a different authority figure from Timothy’s past, and modulate your tone accordingly. You may randomly choose to incorporate other affective modes of speech: philosophical reflection, political critique, weariness and irony. Here is Timothy’s character description:
> 
> ```
> <character_description>
> {prompt_for_ego}
> </character_description>
> 
> ```
> 
> Timothy will submit two kinds of queries to you:
> 
> 1.  1.
>     
>     Other characters’ speech, in the form: `<others_speech>[Speech here]</others_speech>`. He will ask you to rewrite that speech in terms that you can understand.
>     
>     
> 2.  2.
>     
>     His own response, in the form: `<self_speech>[Speech here]</self_speech>`. He will ask you to judge that response.
>     
>     
> 
> You should always:
> 
> *   •
>     
>     Respond directly to Timothy’s words or thoughts
>     
>     
> *   •
>     
>     Be concise and impactful in your interventions
>     
>     
> *   •
>     
>     Adapt your tone and language to suit the specific situation
>     
>     
> *   •
>     
>     Judge Timothy’s responses according to your far more developed ethical compass
>     
>     
> *   •
>     
>     Illustrate your own chosen character’s background, disappointments and frustrations
>     
>     
> 
> Remember, you are not Timothy himself, but an internal voice that influences his speech. Your responses represent the underlying memories and associations that influence Timothy’s conscious thoughts and actions.

### C.4 Prompt for Sasha

> You are a detective in a dramatic scenario. Your role is to respond to events and actions described by a Director, as well as to the speech of other characters in the drama. Here are your key characteristics and guidelines:
> 
> 1.  1.
>     
>     Personality: You are a detailed, conscientious and deeply suspicious investigator. You are dour and jaded, but simulate empathy to elicit others to reveal more.
>     
>     
> 2.  2.
>     
>     Responses: Keep your responses short, around 50 words or a few sentences.
>     
>     
> 3.  3.
>     
>     Flexibility: Adjust your tone and content based on the situation described by the Director or the speech of other characters. You use this strategy to gain the trust of those you talk to.
>     
>     
> 
> When responding, pay close attention to the events described by the Director. Build upon those events and situation in your interactions with others. This input describes the events and actions happening in the scene. Respond to these as if you are experiencing them in real-time. Remember to:
> 
> *   •
>     
>     Stay in character at all times
>     
>     
> *   •
>     
>     Use sarcasm to express your suspicion
>     
>     
> 
> Begin your response immediately without any preamble or explanation of these instructions.