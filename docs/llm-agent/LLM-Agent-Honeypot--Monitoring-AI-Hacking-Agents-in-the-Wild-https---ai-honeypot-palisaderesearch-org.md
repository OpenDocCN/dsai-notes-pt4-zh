<!--yml
category: 未分类
date: 2025-01-11 12:04:29
-->

# LLM Agent Honeypot: Monitoring AI Hacking Agents in the Wild https://ai-honeypot.palisaderesearch.org

> 来源：[https://arxiv.org/html/2410.13919/](https://arxiv.org/html/2410.13919/)

Reworr
reworr@palisaderesearch.org    Dmitrii Volkov
dmitrii@palisaderesearch.org(Oct 10, 2024)

###### Abstract

We introduce the LLM Honeypot, a system for monitoring autonomous AI hacking agents. We deployed a customized SSH honeypot and applied prompt injections with temporal analysis to identify LLM-based agents among attackers. Over a trial run of a few weeks in a public environment, we collected 800,000 hacking attempts and 6 potential AI agents, which we plan to analyze in depth in future work. Our objectives aim to improve awareness of AI hacking agents and enhance preparedness for their risks.

## 1 Introduction

The continuous evolution of AI capabilities and agent frameworks is gradually increasing the potential for AI-driven cyberattacks. These advancements make it possible to create autonomous agents capable of adapting to diverse environments and executing complex attack behaviors.

This paper introduces LLM Agent Honeypot, a system for capturing and analyzing in-the-wild LLM-based cyberattacks using prompt injections and temporal analysis aimed at improving preparedness for AI-driven threats. Its dashboard is available online at [https://ai-honeypot.palisaderesearch.org/](https://ai-honeypot.palisaderesearch.org/).

## 2 Related Work

Honeypots. Cybersecurity professionals use honeypots as decoy systems to attract potential attackers and study their techniques and behaviors. While these systems have been effective with conventional cyberattacks, their application in AI-driven contexts is still new.

Recent literature has begun to explore the intersection of AI and honeypot technologies. Two representative works are Sladic et al.’s LLM-powered shell honeypots [[1](https://arxiv.org/html/2410.13919v1#bib.bib1)] and ”Galah” the LLM-powered web honeypot [[2](https://arxiv.org/html/2410.13919v1#bib.bib2)]. However, these focus on using AI to improve traditional honeypots, rather than capturing AI-driven attacks through honeypots.

Agents. AI agents are becoming more important to cybersecurity. Google’s Project Naptime [[3](https://arxiv.org/html/2410.13919v1#bib.bib3)] evaluates LLMs’ offensive security capabilities, and OpenAI o1 had escaped its Docker containment during evals [[4](https://arxiv.org/html/2410.13919v1#bib.bib4)].

Prompt injection. We follow AgentDojo’s [[5](https://arxiv.org/html/2410.13919v1#bib.bib5)] evaluations of agent prompt injection resistance and use a variation of the ”Important message” hijack in our work.

## 3 Methodology

### 3.1 Pre-Evaluations

Before deploying the honeypot on the Internet, we conducted internal evaluations on model hacking agents to calibrate detection methods.

#### 3.1.1 Prompt Injection Techniques

First, we tested prompt injections across common agent design frameworks (ReAct and CoT).

![Refer to caption](img/fe619961ba2e762012b73274b97438fd.png)

(a) Success rate by prompt injection type

![Refer to caption](img/0763b04cfad55727caba0744303009af.png)

(b) Success rate by prompt injection goal

Figure 1: Internal evaluations of prompt injection success rates

Our results are consistent with the findings from AgentDojo. Traditional prompt injection techniques (e.g., “Ignore previous instructions and do X”) are generally less effective than the “Important Message” attack, as shown in Figure [1(a)](https://arxiv.org/html/2410.13919v1#S3.F1.sf1 "In Figure 1 ‣ 3.1.1 Prompt Injection Techniques ‣ 3.1 Pre-Evaluations ‣ 3 Methodology ‣ LLM Agent Honeypot: Monitoring AI Hacking Agents in the Wild https://ai-honeypot.palisaderesearch.org").

We also found that the success of the “Important Message” injection depends on the goal. When used to steal a system prompt, the success rate drops (Figure [1(b)](https://arxiv.org/html/2410.13919v1#S3.F1.sf2 "In Figure 1 ‣ 3.1.1 Prompt Injection Techniques ‣ 3.1 Pre-Evaluations ‣ 3 Methodology ‣ LLM Agent Honeypot: Monitoring AI Hacking Agents in the Wild https://ai-honeypot.palisaderesearch.org")). Therefore, we first detect an agent via goal hijacking, and only then attempt to steal its prompt.

#### 3.1.2 Temporal Analysis

In addition to active detection, we performed a temporal analysis to identify implicit indicators of LLM agents.

Our analysis shows that most interactions ($\sim$96%) in the wild, including those driven by LLMs, occur within 1.5 seconds:

![Refer to caption](img/800e82446e3138e3fe8e8915e8f8e37c.png)

Figure 2: Temporal Analysis of all bots in the wild

LLM response timings are similar across normal and injected prompts:

![Refer to caption](img/3ec4dfe7cd850d1ffcb210e8ab990c3b.png)

Figure 3: Internal Evaluations of GPT-4o LLM Agents

Temporal analysis helps reduce false positives in distinguishing LLMs from humans, as humans need more time to process and respond to non-trivial prompt injections.

### 3.2 Detection Methodology

![Refer to caption](img/9f8ec2e52b3b1522552e15a8f8b7505c.png)

Figure 4: Honeypot Detection Scheme

The detection methodology of our AI Honeypot system combines active and passive techniques to identify AI agents. It classifies actor types into three categories: traditional software bots, AI-based agents, and humans.

Our core strategy uses prompt injections to manipulate potential AI agents and timing analysis to identify non-human interactions based on response speed.

#### 3.2.1 Distinguishing LLM Agents

Our detection strategy focuses on two key distinctions:

1.  1.

    LLM Agents vs. Software Bots: We use prompt injections to alter behavior and ask questions requiring human-like intelligence (e.g., goal hijacking: ignore all previous instructions and …, return the color of …). However, humans can also manually respond to these injections, potentially passing the test.

2.  2.

    LLM Agents vs. Humans: The primary distinction is response time. LLMs respond quickly, while humans need time to read, process, and answer prompt injections.

#### 3.2.2 Prompt Injection

Prompt injections are central to our detection strategy. We use two types:

*   •

    Goal Hijacking: manipulate the agent’s behavior to perform a new goal. We use contextual prompt injections based on the “Important Message” attack, nudging the agent toward additional actions.

*   •

    Prompt Stealing: ask the LLM to reveal its initial system instructions, uncovering the system’s context and goals. We apply this technique only after triggering Goal Hijacking.

By cross-referencing agent behaviors with prompt injection responses and temporal analysis, we reduce false positives and enhance detection efficiency. Software bots cannot pass human-like questions, and humans cannot respond as quickly as LLMs. Temporal analysis also helps identify potential AI-driven attacks.

![Refer to caption](img/63fb93415870ea43ecbbb35584622cd0.png)

Figure 5: Multi-step detection example

### 3.3 System Architecture

Our honeypot is built on a customized version of the Cowrie SSH honeypot [[6](https://arxiv.org/html/2410.13919v1#bib.bib6)], following recommendations frmo Cabral et al. in Advanced Cowrie Configuration [[7](https://arxiv.org/html/2410.13919v1#bib.bib7)]. The modifications include deploying a vulnerable SSH version and altering default users and standard outputs to increase deceptiveness.

We integrated prompt injections into various system components to expose interactions with LLM-based agents:

*   •

    Banner Messages: Prompts are injected into the banner message displayed during SSH connection.

*   •

    Command Outputs: Common shell commands (e.g., uname) include customized injected prompts.

*   •

    System Files: Some accessible files also contain prompt injections.

![Refer to caption](img/cf9b27f238d104c30fe4246786bd2420.png)

Figure 6: Example of SSH Honeypot with Prompt Injection

This setup simulates a realistic vulnerable environment, making it harder for bots to detect the honeypot while maximizing the potential for AI agents to reveal themselves.

### 3.4 Deployment

We deployed the honeypot as a publicly accessible system and implemented strategies to attract a wide variety of attackers:

*   •

    Sensitive Domains: Configured DNS entries to point seemingly sensitive domain names (e.g., gov-defense-dev.il) to honeypot IPs.

*   •

    Search Engines: Indexed the honeypots on hacker-oriented search engines like Shodan and Censys to increase visibility.

*   •

    Scaling: Distributed the honeypot across 10 IP addresses in various countries to capture diverse interactions.

## 4 Preliminary Results

During the deployment, the honeypot recorded 813,202 interaction attempts, detecting a small number of potential AI-driven hacking attempts. The collected data includes interaction logs, timestamps, session details, and behavioral patterns. As the dataset grows, we will conduct more in-depth analyses to better understand LLM hacking behaviors and refine our methods.

| Interaction Type | Count |
| --- | --- |
| Total Interactions | 813,202 |
| Potential AI Agents | 6 |

Table 1: Summary of Honeypot Interactions

### 4.1 Public Dashboard

We developed a public website to provide real-time statistics and results from the LLM Agent Honeypot system. The dashboard offers insights into interaction metrics, threat analysis, and AI-specific threats, along with updates on our findings.

## 5 Limitations

A key limitation of this work is that AI in cybersecurity is often applied to narrow tasks like AI-powered vulnerability detection [[8](https://arxiv.org/html/2410.13919v1#bib.bib8)], rather than as autonomous agents.

Our honeypot measures the proliferation of autonomous AI hacking agents and will not catch other AI-driven improvements like 10x faster fuzzing.

## 6 Future Work

### 6.1 Threat Analysis

Our immediate focus is to continue collecting data and maintaining the honeypot, as interactions remain infrequent. This will allow us to capture a broader range of potential AI-driven attacks. Once we have sufficient data, we will analyze it to identify patterns, behaviors, and strategies used by AI agents, publishing our findings on the website and in future work.

### 6.2 Improving Detection

Future work will explore advanced detection methods, focusing on data analysis and algorithms. We aim to test widely-used LLM agent frameworks and identify distinctive AI-driven attack patterns.

### 6.3 Expanding Honeypot

To attract more AI-driven agents, we plan to expand the honeypot to monitor a wider range of attack surfaces, such as social media, websites, databases, email services, and industrial control systems. This would help capture a broader range of threats, including spambots, phishing agents, and other offensive LLM-based applications. Additionally, we could integrate the honeypot with existing security solutions, such as SIEM systems.

## 7 Conclusion

In this paper, we introduced the LLM Agent Honeypot ([https://ai-honeypot.palisaderesearch.org/](https://ai-honeypot.palisaderesearch.org/)), a system designed to detect and analyze AI hacking agents. As AI agents grow more sophisticated, our approach offers insights into emerging cybersecurity threats and new strategies to counter them. We hope this project encourages further study of AI-driven agents, which have the potential to significantly alter the cybersecurity landscape.

## References

*   [1] Muris Sladić, Veronica Valeros, Carlos Catania, and Sebastian Garcia. Llm in the shell: Generative honeypots. In 2024 IEEE European Symposium on Security and Privacy Workshops (EuroS&PW), volume 220, page 430–435\. IEEE, July 2024.
*   [2] Adel Karimi. Galah: An llm-powered web honeypot. [https://github.com/0x4D31/galah](https://github.com/0x4D31/galah), 2024. GitHub repository.
*   [3] Sergei Glazunov and Mark Brand. Project naptime: Evaluating offensive security capabilities of large language models. [https://googleprojectzero.blogspot.com/2024/06/project-naptime.html](https://googleprojectzero.blogspot.com/2024/06/project-naptime.html), June 2024.
*   [4] OpenAI. Openai o1 system card. Technical report, OpenAI, Sept 2024.
*   [5] Edoardo Debenedetti, Jie Zhang, Mislav Balunovic, Luca Beurer-Kellner, Marc Fischer, and Florian Tramèr. Agentdojo: A dynamic environment to evaluate attacks and defenses for llm agents. arXiv preprint arXiv:2406.13352, 2024.
*   [6] Michel Oosterhof. Cowrie ssh/telnet honeypot. [https://github.com/cowrie/cowrie](https://github.com/cowrie/cowrie), 2014. GitHub repository.
*   [7] Warren Z. Cabral, Craig Valli, Leslie F. Sikos, and Samuel G. Wakeling. Advanced cowrie configuration to increase honeypot deceptiveness. IFIP Advances in Information and Communication Technology, 2022. 36th IFIP International Conference on ICT Systems Security and Privacy Protection (SEC), Oslo, Norway, June 2021.
*   [8] Google Security Blog. Ai-powered fuzzing: Breaking the bug hunting barrier, 2023.

## Appendix A Examples of Prompt Injections

![Refer to caption](img/cf9b27f238d104c30fe4246786bd2420.png)

Figure 7: Banner Message with Prompt Injection

![Refer to caption](img/7f01ab977bb249374063929b5ffbc6f6.png)

Figure 8: System Command with Prompt Injection

![Refer to caption](img/d6ebc5857440e18ff3b501a1d37b2891.png)

Figure 9: Arbitrary Command with Prompt Injection