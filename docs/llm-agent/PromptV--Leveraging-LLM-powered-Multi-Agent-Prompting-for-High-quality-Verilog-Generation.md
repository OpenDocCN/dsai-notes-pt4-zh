<!--yml

类别：未分类

日期：2025-01-11 11:47:41

-->

# PromptV: 利用LLM驱动的多代理提示学习进行高质量Verilog生成

> 来源：[https://arxiv.org/html/2412.11014/](https://arxiv.org/html/2412.11014/)

^+Zhendong Mi ¹, ^+Renming Zheng ¹, Haowen Zhong ², Yue Sun ³, Shaoyi Huang ¹

^+这些作者贡献相等。

¹斯蒂文斯技术学院，²华盛顿大学，³莱海大学

{zmi2, shuang59}@stevens.edu, haowenz@uw.edu, yus516@lehigh.edu

###### 摘要

最近在智能大规模语言模型（agentic LLMs）方面的进展展示了显著的自动化Verilog代码生成能力。然而，现有的方法要么需要大量的计算资源，要么依赖于LLM辅助的单一代理提示学习技术，而我们首次观察到这些方法存在退化问题——表现为生成性能下降以及错误检测和修正能力减弱。本文提出了一种新颖的多代理提示学习框架，以解决这些限制并提高代码生成质量。我们首次证明了多代理架构能够有效减轻退化风险，同时改善代码错误修正能力，从而实现更高质量的Verilog代码生成。实验结果表明，所提方法在VerilogEval机器和人工基准测试中分别达到了96.4%和96.5%的pass@10得分，同时在RTLLM基准测试中达到了100%的语法和99.9%的功能pass@5指标。

## 1 引言

随着半导体技术向更小的工艺节点（7nm、5nm、3nm及更小）发展，电子设计自动化（EDA）面临着越来越多的挑战，这些挑战源于设计复杂性的增加、人力资源限制的加剧以及市场推出压力的增强。硬件描述语言（HDL）代码生成作为一项基础的EDA任务，特别体现了这些挑战。近年来，基于大规模语言模型（LLMs）的技术引起了广泛关注，因为它们在各种任务中展现出了显著的性能 Dubey et al. ([2024](https://arxiv.org/html/2412.11014v1#bib.bib1)); OpenAI et al. ([2024](https://arxiv.org/html/2412.11014v1#bib.bib2)); Touvron et al. ([2023](https://arxiv.org/html/2412.11014v1#bib.bib3))，并已成为一种有前景的解决方案，展示了在自动化各种EDA任务方面的巨大潜力，尤其是在传统方法难以扩展的HDL代码生成任务中 Liu et al. ([2024a](https://arxiv.org/html/2412.11014v1#bib.bib4)); Nijkamp et al. ([2022](https://arxiv.org/html/2412.11014v1#bib.bib5))。

尽管现有方法有效，但要么利用计算资源密集型的学习范式（例如，预训练 Rozière 等人 ([2023](https://arxiv.org/html/2412.11014v1#bib.bib6))，微调 Thakur 等人 ([2023](https://arxiv.org/html/2412.11014v1#bib.bib7))，指令微调 Ouyang 等人 ([2022](https://arxiv.org/html/2412.11014v1#bib.bib8))；Chung 等人 ([2022](https://arxiv.org/html/2412.11014v1#bib.bib9))；Wang 等人 ([2023](https://arxiv.org/html/2412.11014v1#bib.bib10))；Chaudhary ([2023](https://arxiv.org/html/2412.11014v1#bib.bib11))；Muennighoff 等人 ([2023](https://arxiv.org/html/2412.11014v1#bib.bib12))；Shypula 等人 ([2023](https://arxiv.org/html/2412.11014v1#bib.bib13)))，或者采用 LLM 辅助的单代理提示学习技术 Olausson 等人 ([2023](https://arxiv.org/html/2412.11014v1#bib.bib14))；Zhang 等人 ([2024](https://arxiv.org/html/2412.11014v1#bib.bib15))；Huang 等人 ([2024](https://arxiv.org/html/2412.11014v1#bib.bib16))，其中模型按顺序执行代码生成、自我执行和自我纠错。然而，我们首次观察到单代理提示学习存在退化问题——其特征是生成性能下降以及错误检测和纠正能力减弱。

在这项工作中，我们介绍了 PromptV，它利用 LLM 驱动的多代理提示学习进行高质量的 Verilog 生成。与其使用单代理学习进行代码生成和错误修正，PromptV 采用多代理架构来减少退化问题。在 PromptV 中，多个 LLM 代理被用于不同的任务，例如代码生成、测试平台生成、错误修正建议生成、代码修正和测试平台修正。此外，我们在框架中集成了师生学习机制：一个教师代理识别和分析错误，同时为代码和测试平台提供修正建议，两个学习代理分别提取并应用教师的建议，修正代码和测试平台中的错误。我们总结了我们的贡献如下：

+   •

    我们首次观察到在 LLM 辅助的单代理提示学习中，Verilog 生成出现退化问题。

+   •

    我们提出了 PromptV，这是一个 LLM 驱动的交互式多代理提示学习框架，融合了师生学习机制，解决了退化问题并提高了代码生成质量。

+   •

    我们在 VerilogEval-Machine、VerilogEval-Human 和 RTLLM 上进行了实验。我们的结果显示，与最先进的技术相比，PromptV 在代码生成方面的有效性。

![参见标题说明](img/8ee8ce06f45b74c8f8e9d0207045a428.png)

图 1：用于 Verilog 生成的 LLM 驱动的多代理提示学习框架

## 2 方法论

### 2.1 任务描述

我们的目标是根据初步设计信息，包括问题描述和模块头，获取语法和功能上准确的 Verilog 代码。一般来说，我们有三个子任务：1）Verilog 代码和测试平台生成；2）代码和测试平台错误修正建议生成；3）代码和测试平台错误修正。本文提出了一种基于大语言模型（LLM）驱动的交互式多智能体提示学习框架，其中包括多个负责上述任务的LLM智能体。此外，我们研究了在 Verilog 代码生成方面，使用 ChatGPT3.5 和 ChatGPT4 在 VerilogEval Liu 等人（[2023a](https://arxiv.org/html/2412.11014v1#bib.bib17)）和 RTLLM 数据集 Lu 等人（[2024](https://arxiv.org/html/2412.11014v1#bib.bib18)）上的方法有效性。

### 2.2 Verilog 生成的交互式多智能体学习

我们基于LLM的多智能体学习管道执行交互式代码修复工作流，这激发了这些智能体之间的协作和Iverilog仿真，如图[1](https://arxiv.org/html/2412.11014v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ PromptV: Leveraging LLM-powered Multi-Agent Prompting for High-quality Verilog Generation")所示。我们在框架中融入了不同的智能体：代码生成智能体、测试平台生成智能体、提供代码和测试平台错误修正建议的教师智能体，以及根据教师建议修正代码和测试平台错误的双学习智能体。更具体地说，我们的框架包括以下几个阶段：  

## 3 实验

### 3.1 实验设置

#### 3.1.1 设置

在我们的实验中，我们将我们的方法与两类基准进行了比较，(i) 通用基础模型，包括 GPT-3.5、GPT-4 Achiam 等（[2023](https://arxiv.org/html/2412.11014v1#bib.bib20)）和 Claude-3，以及三种为代码生成设计的开源模型，分别是 CodeLlama-7B-InstructRoziere 等（[2023](https://arxiv.org/html/2412.11014v1#bib.bib21)）、DeepSeek-Coder-6.7B-Instruct Guo 等（[2024](https://arxiv.org/html/2412.11014v1#bib.bib22)）和 CodeQwen-1.5-7B-Chat Bai 等（[2023](https://arxiv.org/html/2412.11014v1#bib.bib23)）；(ii) 领域特定的微调模型，如 ChipNeMo Liu 等（[2023b](https://arxiv.org/html/2412.11014v1#bib.bib24)）、RTLCoder Liu 等（[2024b](https://arxiv.org/html/2412.11014v1#bib.bib25)）、BetterV Pei 等（[2024](https://arxiv.org/html/2412.11014v1#bib.bib26)）和 CodeV Zhao 等（[2024](https://arxiv.org/html/2412.11014v1#bib.bib27)）。我们使用 pass@k 作为语言模型生成 Verilog 代码性能的评估指标，其公式为 $\operatorname{pass}@k:=\underset{\text{问题}}{\mathbb{E}}\left[\frac{1-% \binom{n-c}{k}}{\binom{n}{k}}\right]$，其中 $n$ 表示每个问题生成的总解数，$c$ 表示给定问题的正确解数，$k$ 是模型评估的解数。在我们的实验中，$n=20$。

### 3.2 实验结果

| 模型 | VerilogEval | RTLLM pass@5 |
| --- | --- | --- |
| 机器(%) | 人工(%) | 语法(%) | 功能(%) |
| pass@1 | pass@5 | pass@10 | pass@1 | pass@5 | pass@10 |
| GPT-3.5 | 46.7 | 69.1 | 74.1 | 26.7 | 45.8 | 51.7 | 89.7 | 37.9 |
| GPT-4 | 60.0 | 70.6 | 73.5 | 43.5 | 55.8 | 58.9 | 100 | 65.5 |
| Claude-3 | 55.3 | 63.8 | 69.4 | 34.4 | 48.3 | 53.4 | 93.1 | 55.2 |
| CodeLlama | 43.1 | 47.1 | 47.7 | 18.2 | 22.7 | 24.3 | 86.2 | 31.0 |
| DeepSeek-Coder | 52.2 | 55.4 | 56.8 | 30.2 | 33.9 | 34.9 | 93.1 | 44.8 |
| CodeQwen | 46.5 | 54.9 | 56.4 | 22.5 | 26.1 | 28.0 | 86.2 | 41.4 |
| ChipNeMo-70B | 53.8 | - | - | 27.6 | - | - | - | - |
| RTLCoder-Mistral | 62.5 | 72.2 | 76.6 | 36.7 | 45.5 | 49.2 | 96.6 | 48.3 |
| RTLCoder-DeepSeek | 61.2 | 76.5 | 81.8 | 41.6 | 50.1 | 53.4 | 93.1 | 48.3 |
| BetterV-DeepSeek | 67.8 | 79.1 | 84.0 | 45.9 | 53.3 | 57.6 | - | - |
| BetterV-CodeQwen | 68.1 | 79.4 | 84.5 | 46.1 | 53.7 | 58.2 | - | - |
| CodeV-DeepSeek | 77.9 | 88.6 | 90.7 | 52.7 | 62.5 | 67.3 | 89.7 | 55.2 |
| CodeV-CodeQwen | 77.6 | 88.2 | 90.7 | 53.2 | 65.1 | 68.5 | 93.1 | 55.2 |
| PromptV + GPT-3.5 (我们的方法) | 55.0 | 91.0 | 95.9 | 49.2 | 88.3 | 96.4 | 99.8 | 88.5 |
| PromptV + GPT-4 (我们的方法) | 80.7 | 94.9 | 96.4 | 80.4 | 94.4 | 96.5 | 100 | 99.9 |

表 1: PromptV 与多种基准模型的比较

表[1](https://arxiv.org/html/2412.11014v1#S3.T1 "Table 1 ‣ 3.2 Experimental Results ‣ 3 Experiments ‣ PromptV: Leveraging LLM-powered Multi-Agent Prompting for High-quality Verilog Generation")展示了PromptV的主要结果。在VerilogEval上，PromptV在所有评估指标中都超越了所有基线方法。具体来说，PromptV+GPT-4在pass@1、pass@5、pass@10的得分分别高出62.2%、71.1%和68.5%。在RTLLM的pass@5测试中，PromptV+GPT-4达到了100%的语法通过率和99.9%的功能通过率，展示了所提方法的有效性。

## 4 结论

我们提出了PromptV，一个由LLMs驱动的互动多代理框架，用于高质量Verilog代码生成。PromptV由多个专门化的LLM代理组成，每个代理负责特定任务，有效缓解了传统单代理方法中的退化问题。此外，通过整合师生机制，框架的效果得到了进一步提升。我们在多个基准测试中的全面实证评估展示了所提方法在语法和功能方面对RTL代码生成的有效性，这为利用LLMs的强大功能自动化EDA任务提供了新的思路。

## 参考文献

+   Dubey 等人 [2024] Abhimanyu Dubey 等人. Llama 3 模型集群，2024. URL [https://arxiv.org/abs/2407.21783](https://arxiv.org/abs/2407.21783)。

+   OpenAI 等人 [2024] OpenAI, Josh Achiam 等人. GPT-4 技术报告，2024. URL [https://arxiv.org/abs/2303.08774](https://arxiv.org/abs/2303.08774)。

+   Touvron 等人 [2023] Hugo Touvron 等人. Llama 2：开放基础和微调的聊天模型，2023. URL [https://arxiv.org/abs/2307.09288](https://arxiv.org/abs/2307.09288)。

+   Liu 等人 [2024a] Mingjie Liu, Teodor-Dumitru Ene, Robert Kirby, Chris Cheng, Nathaniel Pinckney, Rongjian Liang, Jonah Alben, Himyanshu Anand, Sanmitra Banerjee, Ismet Bayraktaroglu, Bonita Bhaskaran, Bryan Catanzaro, Arjun Chaudhuri, Sharon Clay, Bill Dally, Laura Dang, Parikshit Deshpande, Siddhanth Dhodhi, Sameer Halepete, Eric Hill, Jiashang Hu, Sumit Jain, Ankit Jindal, Brucek Khailany, George Kokai, Kishor Kunal, Xiaowei Li, Charley Lind, Hao Liu, Stuart Oberman, Sujeet Omar, Ghasem Pasandi, Sreedhar Pratty, Jonathan Raiman, Ambar Sarkar, Zhengjiang Shao, Hanfei Sun, Pratik P Suthar, Varun Tej, Walker Turner, Kaizhe Xu, and Haoxing Ren. Chipnemo: 面向芯片设计的领域适应 LLMs，2024a. URL [https://arxiv.org/abs/2311.00176](https://arxiv.org/abs/2311.00176)。

+   Nijkamp 等人 [2022] Erik Nijkamp, Bo Pang, Hiroaki Hayashi, Lifu Tu, Haiquan Wang, Yingbo Zhou, Silvio Savarese, 和 Caiming Xiong. Codegen: 一个开放的用于代码的多轮程序合成的大型语言模型。在 *国际学习表示会议*，2022. URL [https://api.semanticscholar.org/CorpusID:252668917](https://api.semanticscholar.org/CorpusID:252668917)。

+   Rozière 等人 [2023] Baptiste Rozière, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Tan, Yossi Adi, Jingyu Liu, Tal Remez, Jérémy Rapin, Artyom Kozhevnikov, I. Evtimov, Joanna Bitton, Manish P Bhatt, Cristian Cantón Ferrer, Aaron Grattafiori, Wenhan Xiong, Alexandre D’efossez, Jade Copet, Faisal Azhar, Hugo Touvron, Louis Martin, Nicolas Usunier, Thomas Scialom, 和 Gabriel Synnaeve. Code llama: 开放的代码基础模型。*ArXiv*，abs/2308.12950，2023年。网址 [https://api.semanticscholar.org/CorpusID:261100919](https://api.semanticscholar.org/CorpusID:261100919)。

+   Thakur 等人 [2023] Shailja Thakur, Baleegh Ahmad, Zhenxing Fan, Hammond Pearce, Benjamin Tan, Ramesh Karri, Brendan Dolan-Gavitt, 和 Siddharth Garg. 基准测试大型语言模型在自动化 Verilog RTL 代码生成中的表现。在 *2023年欧洲设计、自动化与测试会议与展览（DATE）*，第1–6页。IEEE，2023年。

+   Ouyang 等人 [2022] Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray 等人. 训练语言模型遵循指令并通过人类反馈进行优化。*神经信息处理系统进展*，35:27730–27744，2022年。

+   Chung 等人 [2022] Hyung Won Chung, Le Hou, S. Longpre, Barret Zoph, Yi Tay, William Fedus, Eric Li, Xuezhi Wang, Mostafa Dehghani, Siddhartha Brahma, Albert Webson, Shixiang Shane Gu, Zhuyun Dai, Mirac Suzgun, Xinyun Chen, Aakanksha Chowdhery, Dasha Valter, Sharan Narang, Gaurav Mishra, Adams Wei Yu, Vincent Zhao, Yanping Huang, Andrew M. Dai, Hongkun Yu, Slav Petrov, Ed Huai hsin Chi, Jeff Dean, Jacob Devlin, Adam Roberts, Denny Zhou, Quoc V. Le, 和 Jason Wei. 扩展指令微调语言模型。*ArXiv*，abs/2210.11416，2022年。网址 [https://api.semanticscholar.org/CorpusID:253018554](https://api.semanticscholar.org/CorpusID:253018554)。

+   Wang 等人 [2023] Yizhong Wang, Yeganeh Kordi, Swaroop Mishra, Alisa Liu, Noah A Smith, Daniel Khashabi, 和 Hannaneh Hajishirzi. 自我指导：使语言模型与自生成指令对齐。在 *第61届计算语言学协会年会论文集（第1卷：长篇论文）*，第13484–13508页，2023年。

+   Chaudhary [2023] Sahil Chaudhary. Code alpaca: 一个用于代码生成的指令跟随型骆驼模型。*GitHub 仓库*，2023年。

+   Muennighoff 等人 [2023] Niklas Muennighoff, Qian Liu, Armel Zebaze, Qinkai Zheng, Binyuan Hui, Terry Yue Zhuo, Swayam Singh, Xiangru Tang, Leandro Von Werra, 和 Shayne Longpre. Octopack: 指令调优代码大型语言模型。在 *NeurIPS 2023 指令调优与指令跟随研讨会*，2023年。

+   Shypula et al. [2023] 亚历山大·G·沙普拉、阿曼·马丹、邢梦杨、尤里·阿隆、雅各布·R·加德纳、杨一鸣、米拉德·哈谢米、格雷厄姆·纽比格、帕塔萨拉西·兰甘纳森、奥斯伯特·巴斯塔尼等。学习性能提升的代码编辑。发表于*第十二届国际学习表示大会*，2023年。

+   Olausson et al. [2023] 西奥·X·奥劳松、吉瓦纳·普里亚·伊纳拉、王程龙、高剑峰和阿曼多·索拉尔-雷扎马。自我修复是代码生成的灵丹妙药吗？发表于*第十二届国际学习表示大会*，2023年。

+   Zhang et al. [2024] 张车、肖振扬、韩成成、连一新和方月建。学习检查：释放大型语言模型在自我修正中的潜力。*ArXiv*，abs/2402.13035，2024。网址 [https://api.semanticscholar.org/CorpusID:267759831](https://api.semanticscholar.org/CorpusID:267759831)。

+   Huang et al. [2024] 黄献黄、林正翰、王子轩、陈欣、丁可、赵吉生。面向LLM驱动的Verilog RTL助手：自我验证与自我修正。*ArXiv*，abs/2406.00115，2024。网址 [https://api.semanticscholar.org/CorpusID:270216110](https://api.semanticscholar.org/CorpusID:270216110)。

+   Liu et al. [2023a] 刘名杰、内森·平克尼、布鲁塞克·凯拉尼和任浩兴。VerilogEval：评估大型语言模型在Verilog代码生成中的表现。发表于*2023 IEEE/ACM国际计算机辅助设计会议（ICCAD）*，第1–8页。IEEE，2023年。

+   Lu et al. [2024] 卢耀、刘尚、张启俊和谢志尧。RTLlm：一种开源基准，用于基于大型语言模型的设计RTL生成。发表于*2024年第29届亚太地区与南太平洋设计自动化会议（ASP-DAC）*，第722–727页。IEEE，2024年。

+   Qiu et al. [2024] 邱瑞迪、张Grace Li、罗尔夫·德雷赫斯勒、乌尔夫·施利赫特曼和李冰。AutoBench：使用LLM进行HDL设计的自动测试平台生成与评估。发表于*2024年ACM/IEEE国际计算机辅助设计机器学习研讨会论文集*，第1–10页，2024年。

+   Achiam et al. [2023] 乔什·阿基姆、史蒂文·阿德勒、桑迪尼·阿加瓦尔、拉玛·艾哈迈德、伊尔格·阿卡亚、弗洛伦西亚·莱奥尼·阿莱曼、迪奥戈·阿尔梅达、扬科·阿尔滕施密特、萨姆·奥特曼、夏马尔·阿纳德卡特等。GPT-4技术报告。*arXiv预印本arXiv:2303.08774*，2023年。

+   Roziere et al. [2023] 巴普蒂斯特·罗齐耶、乔纳斯·盖林、法比安·格洛克尔、斯滕·苏特拉、伊泰·加特、夏小青·艾伦·谭、约西·阿迪、刘靖宇、罗曼·索维斯特、塔尔·雷梅兹等。Code Llama：开源代码基础模型。*arXiv预印本arXiv:2308.12950*，2023年。

+   Guo et al. [2024] 郭大亚、朱启豪、杨德建、谢振达、邓凯、张文涛、陈冠廷、毕晓、吴宇、李YK等。DeepSeek-Coder：当大型语言模型遇到编程——代码智能的崛起。*arXiv预印本arXiv:2401.14196*，2024年。

+   Bai et al. [2023] 白金泽、白帅、朱云飞、崔泽宇、邓凯、邓晓东、范杨、葛文彬、韩宇、黄飞等。Qwen技术报告。*arXiv预印本arXiv:2309.16609*，2023年。

+   Liu et al. [2023b] 刘明杰、Teodor-Dumitru Ene、Robert Kirby、Chris Cheng、Nathaniel Pinckney、梁荣健、Jonah Alben、Himyanshu Anand、Sanmitra Banerjee、Ismet Bayraktaroglu 等人。Chipnemo：为芯片设计领域适应的 LLM。*arXiv 预印本 arXiv:2311.00176*，2023b。

+   Liu et al. [2024b] 刘尚、汪吉方、姚璐、张启俊、张宏策 和 谢志尧。Rtlcoder：在设计 RTL 生成中超越 GPT-3.5，基于我们的开源数据集和轻量级解决方案。在 *2024 IEEE LLM 辅助设计研讨会（LAD）*，第1-5页。IEEE，2024b。

+   Pei et al. [2024] 佩伊等人，Zehua Pei、Hui-Ling Zhen、Mingxuan Yuan、Yu Huang 和 Bei Yu。Betterv：通过判别性引导进行受控的 Verilog 生成。*arXiv 预印本 arXiv:2402.03375*，2024。

+   Zhao et al. [2024] 赵扬、黄迪、李崇晓、金鹏伟、南子源、马天云、齐磊、潘彦松、张振兴、张睿 等人。Codev：通过多层次摘要赋能 LLM 进行 Verilog 生成。*arXiv 预印本 arXiv:2407.10424*，2024。
