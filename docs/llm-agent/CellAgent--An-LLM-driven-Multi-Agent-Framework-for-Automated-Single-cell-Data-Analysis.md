<!--yml

category: 未分类

date: 2025-01-11 12:25:27

-->

# CellAgent: 一种由 LLM 驱动的自动化单细胞数据分析多代理框架

> 来源：[https://arxiv.org/html/2407.09811/](https://arxiv.org/html/2407.09811/)

\equalcont

这些作者对本工作作出了同等贡献。 \equalcont这些作者对本工作作出了同等贡献。 \equalcont这些作者对本工作作出了同等贡献。 \equalcont这些作者对本工作作出了同等贡献。 [2]\fnmJianye \surHao [1,3]\fnmJiajie \surPeng

[1]\orgdivAI for Science 跨学科研究中心, \orgname计算机科学与技术学院, 西北工业大学, \orgaddress\street西安东向路 1 号, \city西安, \country中国 2]\orgdiv智能与计算学院, \orgname天津大学, \orgaddress\street天津卫津路 92 号, \city天津, \country中国

[3]\orgdiv大数据存储与管理重点实验室, 西北工业大学, 工业和信息化部, \orgaddress\street西安东向路 1 号, \city西安, \country中国

\fnmYihang \surXiao    \fnmJinyi \surLiu    \fnmYan \surZheng    \fnmXiaohan \surXie    [jianye.hao@tju.edu.cn](mailto:jianye.hao@tju.edu.cn)    \fnmMingzhi \surLi    \fnmRuitao \surWang    \fnmFei \surNi    \fnmYuxiao \surLi    \fnmJintian \surLuo    \fnmShaoqing \surJiao    [jiajiepeng@nwpu.edu.cn](mailto:jiajiepeng@nwpu.edu.cn) * [ *

###### 摘要

单细胞 RNA 测序（scRNA-seq）数据分析对生物研究至关重要，因为它能够精确地表征细胞异质性。然而，研究人员手动操作各种工具以获得所需结果可能是费时费力的。为了解决这一问题，我们推出了 CellAgent（[http://cell.agent4science.cn/](http://cell.agent4science.cn/)），一个由大型语言模型（LLM）驱动的多代理框架，专为自动化处理和执行单细胞 RNA 测序数据分析任务而设计，能够提供高质量的结果且无需人工干预。首先，为了使通用 LLM 适应生物领域，CellAgent 构建了由 LLM 驱动的生物学专家角色——规划者、执行者和评估者——每个角色都有特定的职责。然后，CellAgent 引入了一种分层决策机制，以协调这些生物学专家，有效地推动复杂数据分析任务的规划和逐步执行。此外，我们提出了一种自我迭代优化机制，使 CellAgent 能够自主评估和优化解决方案，从而保证输出质量。我们在一个涵盖数十种组织和数百种不同细胞类型的综合基准数据集上评估了 CellAgent。评估结果一致表明，CellAgent 能够有效地识别最适合单细胞分析任务的工具和超参数，实现最佳性能。这个自动化框架显著降低了科学数据分析的工作量，引领我们进入了“科学代理”时代。

## 1 引言

单细胞RNA测序（scRNA-seq）技术通过以前所未有的规模和精度分析转录组数据，已经彻底改变了分子生物学[[1](https://arxiv.org/html/2407.09811v1#bib.bib1)]。这些进展推动了计算方法的大规模创新，目前已有超过1400种工具可用于从不同角度分析scRNA-seq数据[[2](https://arxiv.org/html/2407.09811v1#bib.bib2)]。这些工具，包括计算框架和软件库[[3](https://arxiv.org/html/2407.09811v1#bib.bib3), [4](https://arxiv.org/html/2407.09811v1#bib.bib4), [5](https://arxiv.org/html/2407.09811v1#bib.bib5)]，结合方法基准和最佳实践工作流程[[6](https://arxiv.org/html/2407.09811v1#bib.bib6), [7](https://arxiv.org/html/2407.09811v1#bib.bib7)]，为生物学领域的突破性发现提供了支持[[8](https://arxiv.org/html/2407.09811v1#bib.bib8), [9](https://arxiv.org/html/2407.09811v1#bib.bib9)]。

然而，分析scRNA-seq数据涉及相当复杂的工作，并且需要专门的知识和专业技能。相关步骤可能包括预处理、批次校正、聚类、标记基因可视化、细胞类型注释以及轨迹推断等。为了完成这些步骤，研究人员必须仔细执行一系列相应的工具，同时配置适当的超参数和模型，以适应生物数据的特定特点[[7](https://arxiv.org/html/2407.09811v1#bib.bib7)]。这要求研究人员不仅具备先进的编程技能，还需要有坚实的生物学背景，进一步增加了进行单细胞分析任务的成本。因此，迫切需要一种智能工具，能够自动组合并执行现有工具，以生成scRNA-seq数据的分析结果。这样的自动化工具可以显著降低生物学领域的技术门槛，并为生物科学家排除数据分析中的障碍。

最近，随着大型语言模型（LLMs）所展示出的卓越能力[[10](https://arxiv.org/html/2407.09811v1#bib.bib10)、[11](https://arxiv.org/html/2407.09811v1#bib.bib11)、[12](https://arxiv.org/html/2407.09811v1#bib.bib12)]，人们对由LLM驱动的自主AI代理的研究兴趣日益增长，这些代理能够自动处理任务，例如用于软件编程的MetaGPT[[13](https://arxiv.org/html/2407.09811v1#bib.bib13)]和用于办公套件辅助的Microsoft Copilot[[14](https://arxiv.org/html/2407.09811v1#bib.bib14)]。这些AI代理通常将LLM作为代理“大脑”的核心，并辅以必要的机制，如记忆和工具，以扩展其能力[[15](https://arxiv.org/html/2407.09811v1#bib.bib15)、[16](https://arxiv.org/html/2407.09811v1#bib.bib16)、[17](https://arxiv.org/html/2407.09811v1#bib.bib17)]。受到这些进展的启发，提出了一个新问题：如何利用LLM设计一个生物学上有效的代理框架，用于自动化单细胞数据分析任务？

直接将LLM应用于复杂的scRNA-seq数据分析面临着重大挑战。通用的LLM缺乏对专业生物学工具和概念的全面和准确理解，导致输出可能缺乏合理的生物学意义。此外，LLM难以理解涉及多步骤任务的长篇上下文，容易丧失关键信息。然而，LLM并未意识到这些缺陷，导致结果不可靠。因此，开发有效整合LLM优势与scRNA-seq数据分析特定需求的专业方法至关重要，以实现高效、专业和自动化的任务执行。

在本文中，为了应对上述挑战，我们提出了一种专门的零代码CellAgent，它是一个基于LLM驱动的多代理协作框架，用于scRNA-seq数据分析。CellAgent能够直接理解自然语言任务描述，通过有效协作，自动完成复杂任务，并且高质量地实现。首先，考虑到LLM在理解生物学专业知识方面的局限性，我们手动收集了一些关键的专家经验和工具。基于LLM建立了三种生物学专家角色：规划者（Planner）、执行者（Executor）和评估者（Evaluator）。为了增强执行过程的稳定性，并避免在单次操作中处理冗长的步骤，CellAgent引入了层级决策机制，上级任务由规划者进行规划，下级任务由执行者进行执行。此外，为了保证CellAgent输出的解决方案质量，我们提出了一种自我迭代优化机制，鼓励执行者通过引入自动化评估结果并考虑潜在的代码执行异常，自动优化规划过程。评估者在推动迭代优化过程中扮演了至关重要的角色，具备从生物学专家角度评估任务解决方案质量的能力。通过这些生物学专家的有效协作，CellAgent最终具备了强大的自动化scRNA-seq数据分析能力。CellAgent不需要人工干预，实现了整个工作流程的端到端自动执行，有效保证了输出结果的质量。

我们已经开发了一个基准数据集，包含超过50个单细胞数据集，涵盖了几十种组织和数百种不同的细胞类型，涵盖了正常和疾病样本。在22个数据集上进行的实验结果表明，CellAgent始终表现出稳健的性能，匹敌或超越了几种最佳工具的性能。在整个数据集上，CellAgent在92%的情况下完成任务，任务完成率是直接使用GPT-4的两倍多。这一创新方法为单细胞转录组学研究中的数据处理和分析提供了强大的自动化解决方案，显著降低了技术和财务的入门门槛。同时，CellAgent有潜力在更多生物医学研究领域广泛应用，推动更准确和全面的生物学发现。

![参见图注](img/089f66c20b5cfa166edab13206403623.png)

图 1：CellAgent 框架示意图。a，CellAgent 接收到的用户输入示例，包括单细胞数据和用户提供的文本信息。b，接收到用户输入后，Planner 角色首先解析用户意图，并将任务分解为子任务。c，最终结果的示意图，包括各个子任务的结果和最终任务结果。d，CellAgent 处理子任务的详细流程视图。当前子任务和历史代码记忆被输入到 Executor 中，后者最初检索工具并输出此步骤可用的工具。随后，获取这些工具的相应文档，并根据文档，Executor 推导出解决方案（文本分析和代码生成）。这些代码在代码沙箱中执行，如果遇到异常，将重新生成解决方案直到成功执行任务。然后，Evaluator 评估当前任务的结果，并允许 Executor 优化解决方案。最终，Evaluator 根据对多个解决方案下结果的评估，汇总结果以获得此步骤的最终结果。

## 2 结果

### 2.1 单细胞 RNA-seq 分析的多智能体协作自动化工作流

scRNA-seq 数据分析的过程本质上复杂且多样。对于单细胞数据集的分析，通常需要手动优化工具和参数。最佳工具和参数往往因数据集的不同而有所变化。为了解决这一挑战，我们介绍了 CellAgent，一种由大语言模型驱动的高级 AI 智能体，专门用于自动化整个 scRNA-seq 数据分析工作流。CellAgent 接收用户输入的数据和自然语言描述的任务（图 [1](https://arxiv.org/html/2407.09811v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")a）。随后，CellAgent 执行一系列自动化流程，包括任务分解（图 [1](https://arxiv.org/html/2407.09811v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")b）、子任务的顺序执行和优化（图 [1](https://arxiv.org/html/2407.09811v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")c），并最终将任务结果交付给用户（图 [1](https://arxiv.org/html/2407.09811v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")d）。

为了增强LLMs在单细胞RNA测序数据分析任务中的应用，并结合领域特定的生物学知识，我们设计了三个不同的基于LLM的生物学专家角色，包括规划者、执行者和评估者。然后，CellAgent通过层次化的决策框架和自我迭代优化机制，有效地组织这些专家角色的协作。这种高效的协作使得每个角色都能履行其职责，促进有效的信息交换，并共同实现高质量的任务完成。

规划者：为了确保复杂的单细胞RNA测序数据分析任务能够合理地分解并逐步解决，类似于人类专家在数据分析中采用的过程，CellAgent引入了一个层次化的决策框架。规划者首先根据用户提供的数据和需求，进行高层次的规划（如图[1](https://arxiv.org/html/2407.09811v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")b所示）。规划者准确理解与单细胞RNA测序数据分析任务相关的整个工作流程，并具备从用户处准确理解自然语言指令、识别数据特征并明智地设计任务计划的能力。该计划包括必要的数据预处理任务，如质量控制、高变基因选择和标准化，以及可能需要的分析步骤，如批次效应校正、细胞类型注释等。

执行者：然后，如图[1](https://arxiv.org/html/2407.09811v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")d所示，低层级的执行者负责按顺序执行分解后的子任务，提供详细的分析和可执行代码。值得注意的是，这些执行者对单细胞RNA测序数据分析任务中使用的具体方法和工具有全面的理解，并能够获取这些工具的文档，从而减少代码执行失败的可能性。如果检测到执行异常，CellAgent将要求执行者解决问题，直到成功执行代码为止。

评估器：评估器负责评估特定数据处理任务当前结果的质量，类似于人类专家，提供对各种方法效果的判断。在生成代码有效执行的基础上，CellAgent提出了一种自我迭代优化机制，以确保该解决方案的合理性。具体而言，基于自我评估结果或潜在的代码异常，执行器通过超参数调优或工具选择自动优化解决方案，从而实现结果的迭代改进。最后，根据评估器的判断，CellAgent输出该步骤的最佳解决方案及相应结果。

为了评估CellAgent的性能，我们对其在处理特定任务中的表现进行了详细评估：批次效应校正、细胞类型注释和轨迹推断。我们的研究结果表明，在大多数情况下，CellAgent产生的结果优于其他现有工具所产生的结果（第2.2-2.4节）。这表明，CellAgent通过多个LLM驱动的代理之间的有效协作，实现了scRNA-seq数据分析的自动化。上述自动化过程涵盖了工具调用、代码生成、代码执行、异常处理、结果评估和解决方案优化，形成闭环，最终生成可靠的结果。为了展示CellAgent在各种scRNA-seq数据分析任务中的广泛适用性，我们使用不同类型的数据集进行了全面比较。结果显示，CellAgent的综合完成率达到了92%，是单独使用GPT-4模型的两倍。此外，CellAgent在各种任务中也优于其他scRNA-seq数据分析工具。

![参见图注](img/09c40f839f6d5c9924f9662a9a8b1711.png)

图2：批次校正。a，CellAgent和其他批次校正算法在批次校正、生物保护性和整体评分上的表现，以及它们使用的编程语言。b，小提琴图显示了CellAgent和其他方法在所有数据集中的整体评分分布。c，CellAgent和其他方法在不同数据集中的排名。d，UMAP图展示了CellAgent在心脏数据集上的表现，分别使用批次标签和细胞类型标签进行着色。

### 2.2 CellAgent实现高效的批次校正

批次校正旨在消除来自不同批次的数据集中的非生物学变异，这些变异源于实验条件、样本处理或测序平台的差异。这些批次效应可能会掩盖或扭曲实际的生物学信号[[18](https://arxiv.org/html/2407.09811v1#bib.bib18)]。因此，批次校正是单细胞分析中的一个关键步骤。

为了评估CellAgent在批次校正上的表现，我们将其应用于九个数据集[[19](https://arxiv.org/html/2407.09811v1#bib.bib19)]，涵盖了人体的主要组织或器官。我们将CellAgent与五种先进方法进行比较，包括scVI[[20](https://arxiv.org/html/2407.09811v1#bib.bib20)]、LIGER[[21](https://arxiv.org/html/2407.09811v1#bib.bib21)]、Scanorama[[22](https://arxiv.org/html/2407.09811v1#bib.bib22)]、Harmony[[23](https://arxiv.org/html/2407.09811v1#bib.bib23)]和Combat[[24](https://arxiv.org/html/2407.09811v1#bib.bib24)]。使用十个指标来评估这些方法去除批次效应的能力，同时保持生物变异性。在这些指标中，有些用于去除批次效应（批次校正），例如kBET[[25](https://arxiv.org/html/2407.09811v1#bib.bib25)]和$iLISI_{Graph}$[[26](https://arxiv.org/html/2407.09811v1#bib.bib26)]，有些则用于保持生物变异性（生物保护），例如$cLISI_{Graph}$[[26](https://arxiv.org/html/2407.09811v1#bib.bib26)]和ARI[[27](https://arxiv.org/html/2407.09811v1#bib.bib27)]。整体得分由十个指标的平均值计算得出，作为最终的性能指标[[18](https://arxiv.org/html/2407.09811v1#bib.bib18)]。

结果表明，CellAgent在批次校正和生物保守性方面都取得了最高分（图[2](https://arxiv.org/html/2407.09811v1#S2.F2 "图 2 ‣ 2.1 基于多智能体协作的单细胞RNA-seq分析自动化工作流 ‣ 2 结果 ‣ CellAgent: 一种LLM驱动的多智能体框架用于自动化单细胞数据分析")a)）。CellAgent在多个数据集上的平均整体得分达到0.684，超越了次优方法scVI，后者得分为0.642。通过观察小提琴图的分位数和密度估计曲线（图[2](https://arxiv.org/html/2407.09811v1#S2.F2 "图 2 ‣ 2.1 基于多智能体协作的单细胞RNA-seq分析自动化工作流 ‣ 2 结果 ‣ CellAgent: 一种LLM驱动的多智能体框架用于自动化单细胞数据分析")b)），CellAgent在整体得分的分布上相较于其他方法表现最为有利。CellAgent在不同数据集上的整体得分更为集中， 中位数约为0.69，超越了其他方法。此外，CellAgent在四个数据集上的排名第一，展示了其优势（图[2](https://arxiv.org/html/2407.09811v1#S2.F2 "图 2 ‣ 2.1 基于多智能体协作的单细胞RNA-seq分析自动化工作流 ‣ 2 结果 ‣ CellAgent: 一种LLM驱动的多智能体框架用于自动化单细胞数据分析")c)）。UMAP图显示，CellAgent去除了批次效应，同时保留了聚类结果中的真实细胞类型（图[2](https://arxiv.org/html/2407.09811v1#S2.F2 "图 2 ‣ 2.1 基于多智能体协作的单细胞RNA-seq分析自动化工作流 ‣ 2 结果 ‣ CellAgent: 一种LLM驱动的多智能体框架用于自动化单细胞数据分析")d)）。来自不同批次的相同类型细胞，如富含脑室的周围胶质细胞、髓系细胞和间质周围胶质细胞，都被混合在一起。

![参考标题](img/d8149ab39a047f03c4e333f78a1592d8.png)

图 3: 细胞类型注释 a，来自五种不同人类组织和小鼠组织的样本中标注的细胞类型的细胞类型注释结果的平均准确性。b，人类PBMC数据集的17个簇的细胞类型注释准确性。c，CellAgent和专家注释的细胞类型注释的详细人类PBMC数据集细胞类型注释。d，使用UMAP图和小提琴图可视化LILR4A基因（PDCs标记）和CD79A基因（B细胞标记）在各簇中的表达。

### 2.3 CellAgent促进了更准确的细胞类型注释

细胞类型注释是分析scRNA-seq数据中的一个关键且通常耗时的前置步骤[[28](https://arxiv.org/html/2407.09811v1#bib.bib28)]。已经开发出多种自动化细胞类型注释工具。然而，这些工具的泛化能力较差，通常仅在特定组织或器官的数据上表现良好，而在其他数据集上表现较差。

我们对七种不同方法在正确分配细胞类型注释方面的性能进行了基准测试。用于基准测试的数据集来自各种组织，包括人类外周血单核细胞[[29](https://arxiv.org/html/2407.09811v1#bib.bib29)]、人类肝脏[[30](https://arxiv.org/html/2407.09811v1#bib.bib30)]、人类肺部[[31](https://arxiv.org/html/2407.09811v1#bib.bib31)]、人类胰腺[[32](https://arxiv.org/html/2407.09811v1#bib.bib32)]和小鼠视网膜[[33](https://arxiv.org/html/2407.09811v1#bib.bib33)]。这些数据集的多样性使我们能够在不同的测序平台、组织类型和生物体上评估CellAgent和其他方法。CellAgent在所有数据集中的平均准确性方面表现优异（图[3](https://arxiv.org/html/2407.09811v1#S2.F3 "Figure 3 ‣ 2.2 CellAgent enables efficient batch correction ‣ 2 Results ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")a）。

例如，CellAgent在含有多个相似亚型的人类外周血单核细胞数据集上表现出较高的准确性。我们评估了CellAgent注释与专家注释的十七个簇之间的一致性。按照[[28](https://arxiv.org/html/2407.09811v1#bib.bib28)]中的评估指标，我们将结果分为“完全匹配”、“部分匹配”和“不匹配”三类，并分别为每一类分配了1、0.5和0的分数。注释的准确性表明，CellAgent在十七个簇中的十个簇实现了“完全匹配”标签，五个簇达到了“部分匹配”结果，只有一个簇为“不匹配”标签。我们的模型有效地注释了94%的PBMC数据集簇，达到了较高的准确性（图[3](https://arxiv.org/html/2407.09811v1#S2.F3 "Figure 3 ‣ 2.2 CellAgent enables efficient batch correction ‣ 2 Results ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")b）。在人的PBMC数据集上的注释结果可视化，包括与专家注释结果的比较，也证明了CellAgent的注释接近真实的细胞类型标签（图[3](https://arxiv.org/html/2407.09811v1#S2.F3 "Figure 3 ‣ 2.2 CellAgent enables efficient batch correction ‣ 2 Results ‣ CellAgent: An LLM-driven Multi-Agent Framework for Automated Single-cell Data Analysis")c和补充图X）。

在scRNA-seq数据分析中，展示每个簇或细胞类型中特定基因的表达非常重要。这些可视化对于理解潜在的生物学机制至关重要。给定特定基因，CellAgent还为用户提供了生成基因表达UMAP图和小提琴图的功能（图[3](https://arxiv.org/html/2407.09811v1#S2.F3 "图 3 ‣ 2.2 CellAgent实现高效的批量修正 ‣ 2 结果 ‣ CellAgent：一个基于LLM的多代理框架，用于自动化单细胞数据分析")d）。

### 2.4 CellAgent增强了轨迹推断的性能

![请参阅说明](img/6dced165cfeb2f9811fe03d7028ba1fe.png)

图 4：轨迹推断。a，CellAgent与其他轨迹推断算法在黄金标准数据集上的性能比较，以及它们的编程语言。b，雷达图显示CellAgent与其他方法在指标上的表现，指标在a中未涉及。c，CellAgent在“aging hsc kowalczyk”数据集上的输出。一个UMAP图展示了按原始细胞类型着色的轨迹，另一个按基于轨迹内里程碑优化的细胞类型簇着色，最后一个按伪时间着色。d，基因表达的热图，特别强调轨迹内里程碑的表达。热图中的细胞顺序根据轨迹进行了优化。

轨迹推断用于解码细胞发育和分化的时间序列。它可以从静态单细胞数据重建发育轨迹，揭示细胞状态之间的过渡，并识别各种生物学特性随时间的变化[[34](https://arxiv.org/html/2407.09811v1#bib.bib34)]。这对于理解决定细胞命运的机制及相关复杂生物学过程至关重要。尽管近年来出现了新的轨迹推断算法，但它们通常适用于特定的数据集或拓扑结构[[35](https://arxiv.org/html/2407.09811v1#bib.bib35)]。为研究人员选择合适的轨迹推断算法并揭示其生物学意义仍然是一个具有挑战性但重要的任务。

通过在九个具有黄金标准轨迹的数据集上进行实验（参见数据集部分）[[34](https://arxiv.org/html/2407.09811v1#bib.bib34)]，我们比较了CellAgent与其他五种轨迹推断方法的性能，包括Raceid stmid [[36](https://arxiv.org/html/2407.09811v1#bib.bib36)]、Scorpius [[37](https://arxiv.org/html/2407.09811v1#bib.bib37)]、Paga、Page tree [[38](https://arxiv.org/html/2407.09811v1#bib.bib38)] 和Slingshot [[39](https://arxiv.org/html/2407.09811v1#bib.bib39)]。根据先前的研究[[34](https://arxiv.org/html/2407.09811v1#bib.bib34)]，我们使用了$\mathit{cor}_{\textrm{dist}}$、$\mathit{F1}_{\textit{branches}}$、$\mathit{wcor}_{\textrm{features}}$、$\mathit{edgeflip}$来衡量拓扑相似性、分支匹配、细胞位置距离和轨迹特异性基因的相关性，以进行轨迹评估。这些指标的加权平均分用于衡量最终的性能[[34](https://arxiv.org/html/2407.09811v1#bib.bib34)]。

在这些数据集中，CellAgent表现最佳，总分为0.496（图[4](https://arxiv.org/html/2407.09811v1#S2.F4 "图 4 ‣ 2.4 CellAgent提高了轨迹推断的性能 ‣ 2 结果 ‣ CellAgent：一个基于LLM的多代理框架用于自动化单细胞数据分析")a））。它超过了Slingshot这一次优方法，后者的得分为0.473。除此之外，CellAgent在多个其他指标上也展示了优势[[34](https://arxiv.org/html/2407.09811v1#bib.bib34)]，如$\mathit{NMSE}_{\textit{rf}}$、$R^{2}_{rf}$、同构性、$\mathit{F1}_{\textit{milestones}}$、$\mathit{cor}_{\textrm{features}}$（图[4](https://arxiv.org/html/2407.09811v1#S2.F4 "图 4 ‣ 2.4 CellAgent提高了轨迹推断的性能 ‣ 2 结果 ‣ CellAgent：一个基于LLM的多代理框架用于自动化单细胞数据分析")b））。在“衰老的hsc kowalczyk”数据集上，CellAgent揭示了从LT-HSC细胞、通过ST-HSC细胞，最终分化为MPP细胞的发育轨迹（图[4](https://arxiv.org/html/2407.09811v1#S2.F4 "图 4 ‣ 2.4 CellAgent提高了轨迹推断的性能 ‣ 2 结果 ‣ CellAgent：一个基于LLM的多代理框架用于自动化单细胞数据分析")c））。CellAgent还描绘了这些细胞基因表达模式的变化（图[4](https://arxiv.org/html/2407.09811v1#S2.F4 "图 4 ‣ 2.4 CellAgent提高了轨迹推断的性能 ‣ 2 结果 ‣ CellAgent：一个基于LLM的多代理框架用于自动化单细胞数据分析")d））。在分化过程中，CD48和MPO基因的表达逐渐增加。准确性和生物学可解释性表明，CellAgent可以帮助科学家们理解决定细胞命运及相关生物学过程的机制。

## 3 讨论

单细胞 RNA 测序（scRNA-seq）是一个强大的工具，用于表征细胞特性，但它在高级数据分析中要求高效率和高质量。在本文中，我们首次介绍了 CellAgent，一种由大语言模型驱动的自动化代理，专为单细胞分析任务设计，使得单细胞 RNA 测序数据的高质量自动化分析成为可能。

CellAgent 能够自动处理复杂的单细胞分析任务，其关键创新在于多个生物专家大语言模型角色之间的有效协作。这一自动化能力来源于其使用强大的 AI 生成模型，这些模型能够理解人类自然语言指令并生成相应内容。值得注意的是，我们发现，作为先进的大语言模型，GPT-4 对特定生物学知识有基础性的理解，包括常见的基因标记和各种单细胞分析任务。然而，它的局限性也很明显，容易混淆某些专业概念。为了解决这一问题，CellAgent 通过手动定义的工具库和若干经验性知识扩展了大语言模型的能力边界，从而增强了其处理单细胞分析任务的能力。接着，我们在 CellAgent 中基于 GPT-4 创建了不同的生物专家角色，使每个角色能够专注于解决特定的过程。随后，CellAgent 通过将任务规划器与负责任务执行的多个角色通过分层规划连接，提升了解决复杂任务的效率。此外，通过采用自我迭代优化机制并利用工具检索器、记忆模块和代码沙盒等模块，CellAgent 进一步提高了输出结果的质量。

值得一提的是，CellAgent 提出了使用 GPT-4V 自动评估批量校正和轨迹推断结果，指导自我迭代优化过程。据我们所知，这是首次将像 GPT-4V 这样的多模态大语言模型（MLLM）应用于评估这些结果，作为替代人类判断的方法。此外，CellAgent 还首次尝试使用 GPT-4 有效整合不同工具的细胞类型注释结果，从而获得更合理的最终注释。这些进展为将大语言模型（或多模态大语言模型）应用于生物信息学研究提供了新的思路。

总的来说，CellAgent是一个多功能、可扩展的自动化scRNA-seq数据分析工具。其任务完成过程不依赖于人工干预，大大降低了数据分析的难度和成本。此外，其开放架构使用户能够提供具体的新知识和工具，使CellAgent能够更好地与用户期望对接，成为研究人员的理想助手。CellAgent的出现不仅为生物信息学开辟了新的研究方向，也拓展了生成型AI在科学领域的应用，可能引领新的发现并深入理解生物系统。

在多个带有专家注释标签的数据集上进行测试表明，CellAgent所获得的结果在大多数情况下是所有工具中最好的，展示了CellAgent在自动化scRNA-seq数据分析方面的强大能力，并确保了质量，达到了人类专家的水平。总体评估表明，CellAgent显著优于仅使用单一GPT-4模型的数据分析过程，说明这种多代理协作框架深度挖掘了GPT-4在处理单细胞数据分析任务中的潜力。

然而，CellAgent的一个显著限制在于其不完善的自我评估方法。目前，自我评估主要依赖于GPT-4V或GPT-4。虽然这些创新的评估过程有助于自我优化并确保最终效果，但CellAgent在支持更多样化的优化方向方面存在一定局限。因此，当用户对优化目标有具体偏好时，手动指定评估过程可以促使CellAgent根据用户期望进行优化。然而，使CellAgent能够高效且自动地整合用户提供的新工具以满足用户需求，代表了下一阶段AI代理在科学领域发展的宝贵研究方向。

## 4 方法

### 4.1 CellAgent的组成

面对scRNA-seq数据分析任务，CellAgent实现了自动化任务分解、代码执行和迭代优化。在此过程中，三个由大语言模型驱动的生物学专家角色及其沟通与协作机制作为CellAgent的核心大脑，负责思考和提供解决方案。此外，为了实现高效自动化，辅助组件如记忆$\mathcal{M}$、工具检索$\mathcal{T}$和代码沙箱$\mathcal{E}$是必不可少的。这些组件使CellAgent能够有效管理和执行任务，让三个生物学专家角色能够专注于思考并提供高效的解决方案，从而确保整个工作流程的无缝自动化。

#### 4.1.1 生物学专家代理

CellAgent 包括三个具有不同职责的生物学专家 LLM 角色，包括计划者（Planner）、执行者（Executor）和评估者（Evaluator）。为了区分这些角色并提高任务执行效率，我们收集了一小部分关于 scRNA-seq 数据分析的人工专家经验，并将必要的工具整合在一起，然后将角色分配、特定专家经验和工具信息传递给不同的 LLM，从而构建这些生物学专家角色。值得注意的是，CellAgent 的用户输入包括三个方面：待处理的数据（${D}$）、要解决的任务描述（$u_{\text{task}}$），以及可选的数据描述（$u_{D}$）和解决问题或工具的偏好要求（$u_{\text{req}}$）。

##### 计划者（Planner）

计划者旨在理解用户需求和数据，并提供全面的任务规划。因此，计划者的系统提示（$p^{p}_{\text{sys}}$）主要包括：1) 角色定位描述，详细说明角色的输入和输出；2) 任务规划的输出格式规范，计划者的输出格式设置为 JSON，以便于子任务的提取；3) 预先收集的专家经验信息。在接收到用户输入后，计划者还会获取从待处理数据中解析出的表示形式，从而更容易理解数据特征。具体来说，我们在 Python 中使用 `AnnData` 来处理单细胞数据，并获取其字符串表示，记作 $\psi(D)$。我们将 LLM 角色计划者表示为 $\mathcal{A}^{\text{LLM}}_{p}$，然后根据这些信息生成 $n$ 个子步骤描述的过程如下：

|  | $\{t_{1},t_{2},...,t_{n}\}\leftarrow\mathcal{A}^{\text{LLM}}_{p}(p^{p}_{\text{% sys}},u_{\text{task}},u_{\text{req}},u_{D},\psi(D)).$ |  | (1) |
| --- | --- | --- | --- |

##### 执行者（Executor）

Executor 代理被包含在 CellAgent 中，以提高特定任务执行的效率，其中工具选择器（Tool Selector）仅负责选择可用工具，而代码生成器（Code Programmer）负责根据推荐的工具生成代码，以完成当前任务。

工具选择器的系统提示，记作 $p^{t}_{\text{sys}}$，主要包括：1) 角色定位描述，详细说明输入和输出规范；2) 输出格式规范，便于其他角色使用，输出格式指定为 JSON。在执行第 $i$ 步时，工具选择器首先检索所有工具的列表 $\mathcal{T}$ 和当前步骤的描述 $t_{i}$。此外，为了确保与用户在整个过程中可能的需求保持一致，其提示中还包括了用户需求。我们将工具选择器表示为 $\mathcal{A}^{\text{LLM}}_{t}$，它生成当前步骤中可用工具的列表，表示为：

|  | $\mathcal{T}_{t_{i}}\leftarrow\mathcal{A}^{\text{LLM}}_{t}(p^{t}_{\text{sys}},u_{\text{req}},\mathcal{T},t_{i}).$ |  | (2) |
| --- | --- | --- | --- |

代码程序员的系统提示（$p^{c}_{\text{sys}}$）主要包括：1）角色定位描述，详细说明输入和输出；2）输出格式规范，清晰定义了输出代码块的格式，以便解析和执行；以及3）一些关于专业工具使用和常见异常处理的经验。在执行过程中，它会自动检索此步骤中可用工具的文档（$\text{Doc}(\mathcal{T}_{t_{i}})$），并且还会提供当前步骤的任务描述、数据表示和用户偏好要求。此外，还会提示有关历史代码的记忆，表示为$\mathcal{M}$，使得代码生成过程能够更好地匹配上下文。具体来说，我们将代码程序员表示为$\mathcal{A}^{\text{LLM}}_{c}$，其响应包含文本分析$w_{i}$和该特定步骤$t_{i}$的代码$c_{i}$，如：

|  | $(c_{i},w_{i})\leftarrow\mathcal{A}^{\text{LLM}}_{c}(p^{c}_{\text{sys}},u_{% \text{req}},u_{D},\mathcal{M},t_{i},\text{Doc}(\mathcal{T}_{t_{i}})).$ |  | (3) |
| --- | --- | --- | --- |

然后，代码$c_{i}$在代码沙盒中执行。执行结果，记作$\mathcal{E}(c_{i})$，可能是潜在的执行异常，或者是成功执行的结果。如果发生异常，代码程序员将执行修复操作，生成修正后的代码，表示为$\mathcal{A}^{\text{LLM}}_{t}(\mathcal{E}(c_{i}))$。

##### 评估器

评估器的任务是评估当前步骤的结果，并从执行者自我优化产生的多个结果中选择最佳的一个。评估器的系统提示（$p^{e}_{\text{sys}}$）主要包括：1）角色定位描述，包括输入和输出描述；2）可用评估方法的列表，包括用于评估关键步骤的集成到CellAgent中的具体代码。在执行过程中，它接收数据的字符串表示、当前步骤的任务描述、用户偏好要求，最重要的是执行代码。随后，评估器进行评估。如果在当前试验中，评估器可以评估多个试验的结果并选择最优解，则当前步骤的最终解决方案将确定。否则，代码程序员将被提示优化解决方案。我们将步骤$t_{i}$的最佳解决方案代码表示为$\bar{c_{i}}$，并将多个试验的代码表示为$\{c_{i}^{j},j=1,2,...\}$。然后，评估器评估并选择当前步骤的最优解决方案代码的过程可以表示为：

|  | $\bar{c_{i}}=\mathcal{A}^{\text{LLM}}_{e}(p^{e}_{\text{sys}},u_{\text{req}},u_{% D},t_{i},\{c_{i}^{j}\}),j=1,2,....$ |  | (4) |
| --- | --- | --- | --- |

#### 4.1.2 辅助组件

##### 内存控制

大型语言模型（LLMs）无法保留历史消息，这就需要在 LLM 驱动的智能体中加入记忆模块。通过合理组织过去的信息，LLMs 可以防止在规划过程中遗忘之前的内容，并能更高效地处理较长的上下文。在 CellAgent 中，不同子任务处理的内部逻辑是相互独立的，这意味着每个子任务的执行过程不需要感知之前步骤的内部处理，如工具选择、代码异常处理或迭代过程中的中间代码生成。因此，CellAgent 定义了全局记忆和局部记忆，能够高效地存储历史信息。这使得不同的 LLM 角色可以访问适当且有价值的历史信息，从而提升 LLM 推理的效率。

如图[1](https://arxiv.org/html/2407.09811v1#S1.F1 "图 1 ‣ 1 引言 ‣ CellAgent: 一种基于 LLM 的自动化单细胞数据分析多智能体框架")所示，我们定义了全局记忆，仅存储每个历史步骤的最终代码，可以表示为 $\mathcal{M}\leftarrow\{\bar{c_{1}},\bar{c_{2}},...\}$。这样可以使执行器更有效地生成新代码，减少冗余工作，并提高代码生成的准确性。我们使用代码作为记忆存储介质，这在 scRNA-seq 数据分析中具有高信息熵，能够以更少的 token 转移更全面的信息，从而减少 LLM 成本并提高 LLM 效率。

在每个子任务处理中，执行器保持一个局部记忆，只保留当前步骤内的对话信息，并在子任务结束时重置。局部记忆使得执行器能够感知整个自我优化过程，包括每个生成的正确或错误代码片段、潜在的错误消息和优化过程。这有助于避免在优化过程中出现重复错误，从而提高处理效率。

##### 工具检索

CellAgent 集成了多个单细胞分析工具，以确保操作的稳定性。该集成主要通过工具检索模块（标记为 $\mathcal{T}$）来实现。集成的工具在 CellAgent 框架内进行注册，使得工具选择器能够检测它们的存在，并在每个子任务开始时为代码程序员提供潜在有用工具的列表。此外，在我们的实现中，工具类配备了标准化文档，称为 Python 中的 docstrings。此功能使得执行器能够访问所选工具的文档，从而提高代码生成的准确性。

##### 代码沙盒

为确保代码执行的安全性和可靠性，CellAgent 实现了代码沙箱，将由大语言模型（LLMs）生成的代码与执行过程隔离。具体而言，这通过 Jupyter Notebook 转换（`nbconvert`）实现，其中数据加载和由 LLMs 生成的每个步骤的代码都在一个综合的 Jupyter notebook 中执行。这种实现方式解耦了 CellAgent 框架的运行与单细胞数据分析中代码的执行，从而增强了执行生成代码的安全性。此外，它还便于单细胞任务分析的结果管理和可重复性。

### 4.2 自迭代优化通过自我评估

#### 4.2.1 预处理

CellAgent 首先通过去除低质量的细胞和基因来处理数据，随后识别高度变异的基因。接着，评估器检查这些预处理步骤的有效性，优化预处理代码以更好地适应数据集的特征，并向用户展示适当的可视化图表。除非用户另有指定，CellAgent 默认执行一次自我优化的单次迭代用于预处理步骤。

#### 4.2.2 批次校正

CellAgent 采用迭代优化方法，通过 UMAP 图评估不同方法的结果，从而产生最佳的批次校正结果。具体而言，集成了 GPT-4v 的评估器会评估这些图，并对这些方法进行排名。默认情况下，迭代次数设置为三次，这意味着自优化过程将生成三种不同的批次校正结果。经过评估后，得分最高的方法将被选定为最终输出。

#### 4.2.3 细胞类型注释

CellAgent 采用基于数据库的注释工具，如 CellMarker 2.0 和 ACT，以及基于基因表达的工具，包括 CellTypist、SCSA、ScType 和 GPT-4 来进行细胞类型注释。通过自迭代优化机制，CellAgent 自动选择执行不同的方法并获取结果。由 GPT-4 驱动的评估器随后汇总这些注释，并确定每个簇的最终细胞类型。

#### 4.2.4 轨迹推断

在 CellAgent 中，轨迹推断的评估类似于批次校正的评估。由于轨迹信息难以通过文字直接描述，评估器对不同轨迹推断算法的可视化结果进行评分。然后，在自迭代优化过程中，选择得分最高的可视化结果作为最终输出。这些可视化结果通常包括展示轨迹的 UMAP 图，整合了细胞信息和轨迹信息，以丰富内容。此外，专家编写的关于轨迹推断任务的知识将以提示的形式提供给评估器。

### 4.3 评估指标

#### 4.3.1 细胞类型注释指标

为了评估细胞类型注释的准确性并便于不同方法之间的比较，我们采用了平均评分指标。对于预测标签和实际标签，当它们在注释术语或CL细胞本体名称上完全一致时，给予“完全匹配”评分；当标签共享一般细胞类型类别（例如成纤维细胞和基质细胞），但在具体注释或本体名称上有所不同时，给予“部分匹配”评分。相反，当广义细胞类型类别、注释或本体名称存在差异时，给予“不匹配”评分。我们为“完全匹配”、“部分匹配”和“不匹配”分别分配一致性评分1、0.5和0[[28](https://arxiv.org/html/2407.09811v1#bib.bib28)]。这些评分随后会在每个数据集中的所有细胞类型之间进行平均。这个平均分数作为注释准确性的量化指标，可以实现不同注释方法和数据集之间的可靠比较。这种系统性的评分方法确保研究人员能够客观评估细胞类型分类的精确度，并在需要时改进其分析方法或数据解释。

#### 4.3.2 批次修正指标

为了评估去除批次效应的能力，同时保持生物学变异性，我们使用了十种不同的指标。这些指标被分为两类[[18](https://arxiv.org/html/2407.09811v1#bib.bib18)]。表示批次效应去除效果的指标包括图连接性（Graph Connectivity）、PCR比较[[18](https://arxiv.org/html/2407.09811v1#bib.bib18)]、$iLISI_{Graph}$[[26](https://arxiv.org/html/2407.09811v1#bib.bib26)]、kBET[[25](https://arxiv.org/html/2407.09811v1#bib.bib25)]和$ASW_{batch}$[[40](https://arxiv.org/html/2407.09811v1#bib.bib40)]。表示生物学保守性的指标包括孤立标签（Isolated Labels）[[18](https://arxiv.org/html/2407.09811v1#bib.bib18)]、ARI[[27](https://arxiv.org/html/2407.09811v1#bib.bib27)]、NMI[[41](https://arxiv.org/html/2407.09811v1#bib.bib41)]、$ASW_{cell}$[[40](https://arxiv.org/html/2407.09811v1#bib.bib40)]和$cLISI_{Graph}$[[18](https://arxiv.org/html/2407.09811v1#bib.bib18)]。这些指标的权重分别代表批次修正和生物学保留的评分。总体评分是这些指标加权平均后的最终指示值。

#### 4.3.3 轨迹推断指标

使用了四种不同类型的度量来评估轨迹的不同方面。这些度量包括$\mathit{edgeflip}$，用于衡量两个拓扑结构之间的相似性，$\mathit{F1}_{\textit{branches}}$和$\mathit{F1}_{\textit{milestones}}$，用于评估细胞分配到分支的相似性，$\mathit{cor}_{\textrm{dist}}$，用于评估两个轨迹之间细胞位置的相似性[[34](https://arxiv.org/html/2407.09811v1#bib.bib34)]。此外，像$\mathit{wcor}_{\textrm{features}}$和$\mathit{cor}_{\textrm{features}}$这样的度量，量化了已知轨迹和预测轨迹中差异表达特征之间的一致性。总体评分作为$\mathit{edgeflip}$、$\mathit{F1}_{\textit{branches}}$、$\mathit{cor}_{\textrm{dist}}$和$\mathit{cor}_{\textrm{features}}$的几何平均值，作为综合轨迹评估度量[[34](https://arxiv.org/html/2407.09811v1#bib.bib34)]。

## 5 数据可用性

对于批量校正任务，我们使用了来自不同人类器官图谱的九个数据集，包括血液、骨髓、心脏、肠道、肾脏、肝脏、肺、胰腺和骨骼肌。这些数据集可以在[https://www.celltypist.org/organs](https://www.celltypist.org/organs) 获取。我们在细胞类型注释任务中使用了以下数据集：PBMC 数据集 ([https://www.10xgenomics.com/resources/datasets/8-k-pbm-cs-from-a-healthy-donor-2-standard-2-1-0](https://www.10xgenomics.com/resources/datasets/8-k-pbm-cs-from-a-healthy-donor-2-standard-2-1-0))；包含 Smart-seq2、10X Chromium 和 Drop-seq 的胰腺数据集 ([https://hemberg-lab.github.io/scRNA.seq.datasets/human/pancreas/](https://hemberg-lab.github.io/scRNA.seq.datasets/human/pancreas/))；多个 PBMC 数据集可以从单细胞门户网站获得，访问代码为 SCP424；多组学数据集 ([https://figshare.com/projects/Tabula_Muris_Transcriptomic_characterization_of_20_organs_and_tissues_from_Mus_musculus_at_single_cell_resolution/27733](https://figshare.com/projects/Tabula_Muris_Transcriptomic_characterization_of_20_organs_and_tissues_from_Mus_musculus_at_single_cell_resolution/27733))；来自不同供体的免疫数据集 ([https://figshare.com/ndownloader/files/25717328](https://figshare.com/ndownloader/files/25717328))；来自脑区的小鼠数据集 ([https://portal.brain-map.org/atlases-and-data/rnaseq/mouse-whole-cortex-and-hippocampus-10x](https://portal.brain-map.org/atlases-and-data/rnaseq/mouse-whole-cortex-and-hippocampus-10x))。我们在轨迹推断任务中使用了以下数据集：衰老 HSC-老年 Kowalczyk 数据集和衰老 HSC-年轻 Kowalczyk 数据集，这些数据集包含了人类造血干细胞的发育轨迹；人类胚胎 Petropoulos 数据集，包含了人类胚胎细胞的发育轨迹；来自胸腺的 NKT 分化 Engel 数据集；胰腺 α 细胞成熟和 cellbench-SC1、胚系-人类、细胞周期和刺激性树突状细胞-PIC 数据集。这些数据集存储在 Zenodo 上 ([https://doi.org/10.5281/zenodo.1443566](https://doi.org/10.5281/zenodo.1443566))。

## 参考文献

+   \bibcommenthead

+   Heumos 等人 [2023] Heumos, L., Schaar, A.C., Lance, C., Litinetskaya, A., Drost, F., Zappia, L., Lücken, M.D., Strobl, D.C., Henao, J., Curion, F., 等人：跨模态单细胞分析的最佳实践。《自然遗传学评论》24(8), 550–572 (2023)

+   Zappia 和 Theis [2021] Zappia, L., Theis, F.J.：超过 1000 个工具揭示了单细胞 RNA 测序分析领域的趋势。《基因组生物学》22, 1–18 (2021)

+   Amezquita 等人 [2020] Amezquita, R.A., Lun, A.T., Becht, E., Carey, V.J., Carpp, L.N., Geistlinger, L., Marini, F., Rue-Albrecht, K., Risso, D., Soneson, C., 等人：通过 Bioconductor 协调单细胞分析。《自然方法》17(2), 137–145 (2020)

+   Hao等人[2021] Hao, Y., Hao, S., Andersen-Nissen, E., Mauck, W.M., Zheng, S., Butler, A., Lee, M.J., Wilk, A.J., Darby, C., Zager, M., 等人：多模态单细胞数据的综合分析。Cell 184(13), 3573–3587 (2021)

+   Wolf等人[2018] Wolf, F.A., Angerer, P., Theis, F.J.：Scanpy：大规模单细胞基因表达数据分析。Genome biology 19, 1–5 (2018)

+   Luecken和Theis [2019] Luecken, M.D., Theis, F.J.：单细胞RNA-seq分析中的当前最佳实践：教程。Molecular systems biology 15(6), 8746 (2019)

+   Heumos等人[2023] Heumos, L., Schaar, A.C., Lance, C., Litinetskaya, A., Drost, F., Zappia, L., Lücken, M.D., Strobl, D.C., Henao, J., Curion, F., 等人：跨模式单细胞分析的最佳实践。Nature Reviews Genetics, 1–23 (2023)

+   Eraslan等人[2022] Eraslan, G., Drokhlyansky, E., Anand, S., Fiskin, E., Subramanian, A., Slyper, M., Wang, J., Van Wittenberghe, N., Rouhana, J.M., Waldman, J., 等人：跨组织单核分子参考图谱以理解疾病基因功能。Science 376(6594), 4290 (2022)

+   Sikkema等人[2023] Sikkema, L., Ramírez-Suástegui, C., Strobl, D.C., Gillett, T.E., Zappia, L., Madissoon, E., Markov, N.S., Zaragosi, L.-E., Ji, Y., Ansari, M., 等人：健康与疾病中肺的整合细胞图谱。Nature Medicine, 1–15 (2023)

+   OpenAI [2023] OpenAI：GPT-4技术报告。CoRR abs/2303.08774 (2023)

+   Anil等人[2023] Anil, R., Dai, A.M., Firat, O., Johnson, M., Lepikhin, D., Passos, A., Shakeri, S., Taropa, E., Bailey, P., Chen, Z., Chu, E., Clark, J.H., Shafey, L.E., Huang, Y., Meier-Hellstern, K., Mishra, G., Moreira, E., Omernick, M., Robinson, K., Ruder, S., Tay, Y., Xiao, K., Xu, Y., Zhang, Y., Ábrego, G.H., Ahn, J., Austin, J., Barham, P., Botha, J.A., Bradbury, J., Brahma, S., Brooks, K., Catasta, M., Cheng, Y., Cherry, C., Choquette-Choo, C.A., Chowdhery, A., Crepy, C., Dave, S., Dehghani, M., Dev, S., Devlin, J., Díaz, M., Du, N., Dyer, E., Feinberg, V., Feng, F., Fienber, V., Freitag, M., Garcia, X., Gehrmann, S., Gonzalez, L., 等人：Palm 2技术报告。CoRR abs/2305.10403 (2023)

+   Touvron 等人 [2023] Touvron, H., Martin, L., Stone, K., Albert, P., Almahairi, A., Babaei, Y., Bashlykov, N., Batra, S., Bhargava, P., Bhosale, S., Bikel, D., Blecher, L., Canton-Ferrer, C., Chen, M., Cucurull, G., Esiobu, D., Fernandes, J., Fu, J., Fu, W., Fuller, B., Gao, C., Goswami, V., Goyal, N., Hartshorn, A., Hosseini, S., Hou, R., Inan, H., Kardas, M., Kerkez, V., Khabsa, M., Kloumann, I., Korenev, A., Koura, P.S., Lachaux, M., Lavril, T., Lee, J., Liskovich, D., Lu, Y., Mao, Y., Martinet, X., Mihaylov, T., Mishra, P., Molybog, I., Nie, Y., Poulton, A., Reizenstein, J., Rungta, R., Saladi, K., Schelten, A., Silva, R., Smith, E.M., Subramanian, R., Tan, X.E., Tang, B., Taylor, R., Williams, A., Kuan, J.X., Xu, P., Yan, Z., Zarov, I., Zhang, Y., Fan, A., Kambadur, M., Narang, S., Rodriguez, A., Stojnic, R., Edunov, S., Scialom, T.：Llama 2：开放的基础和微调的聊天模型。CoRR abs/2307.09288 (2023)

+   洪等人 [2023] 洪胜, 祝格, 陈瑾, 郑晓, 程义, 张驰, 王建, 王志, 邱思琪, 林志, 周亮, 冉晨, 肖丽, 吴春, 施密杜柏, J.：MetaGPT：多代理协作框架的元编程 (2023)

+   Spataro [2023] Spataro, J.：介绍 Microsoft 365 Copilot —— 你的工作副驾驶。[https://blogs.microsoft.com/blog/2023/03/16/introducing-microsoft-365-copilot-your-copilot-for-work/](https://blogs.microsoft.com/blog/2023/03/16/introducing-microsoft-365-copilot-your-copilot-for-work/)。访问时间：2023年3月16日 (2023)

+   周等人 [2023] 周伟, 姜艺恩, 李琳, 吴佳, 王腾, 邱珊, 张俊, 陈瑾, 吴瑞, 王珊, 朱松, 陈劲, 张伟, 张宁, 陈辉, 崔鹏, Sachan, M.：代理：一个开放源代码的自主语言代理框架 (2023)

+   西等人 [2023] 西泽, 陈伟, 郭轩, 何伟, 丁彧, 洪博, 张敏, 王健, 金晟, 周恩, 郑锐, 范晓, 王翔, 熊磊, 周扬, 王伟, 姜晨, 邹毅, 刘鑫, 尹泽, 杜森, 翁瑞, 程文, 张强, 秦伟, 郑颖, 邱轩, 环旭, 桂涛：基于大语言模型的代理崛起与潜力：一项调查。CoRR abs/2309.07864 (2023) [https://doi.org/10.48550/ARXIV.2309.07864](https://doi.org/10.48550/ARXIV.2309.07864) [2309.07864](https://arxiv.org/abs/2309.07864)

+   王等人 [2023] 王立, 马超, 冯欣, 张震, 杨浩, 张俊, 陈泽, 唐洁, 陈曦, 林洋, 赵伟翔, 魏泽, 温建中：基于大语言模型的自主代理调查。CoRR abs/2308.11432 (2023) [https://doi.org/10.48550/ARXIV.2308.11432](https://doi.org/10.48550/ARXIV.2308.11432) [2308.11432](https://arxiv.org/abs/2308.11432)

+   Luecken 等人 [2022] Luecken, M.D., Büttner, M., Chaichoompu, K., Danese, A., Interlandi, M., Müller, M.F., Strobl, D.C., Zappia, L., Dugas, M., Colomé-Tatché, M. 等：单细胞基因组学中图谱级数据整合的基准测试。Nature methods 19(1), 41–50 (2022)

+   Domínguez Conde 等人 [2022] Domínguez Conde, C., Xu, C., Jarvis, L., Rainbow, D., Wells, S., Gomes, T., Howlett, S., Suchanek, O., Polanski, K., King, H., 等人：跨组织免疫细胞分析揭示了人类组织特异性的特征。科学 376(6594), 5197 (2022)

+   Lopez 等人 [2018] Lopez, R., Regier, J., Cole, M.B., Jordan, M.I., Yosef, N.：用于单细胞转录组学的深度生成模型。自然方法 15(12), 1053–1058 (2018)

+   Liu 等人 [2020] Liu, J., Gao, C., Sodicoff, J., Kozareva, V., Macosko, E.Z., Welch, J.D.：使用 Liger 联合定义来自多个单细胞数据集的细胞类型。自然协议 15(11), 3632–3662 (2020)

+   Hie 等人 [2019] Hie, B., Bryson, B., Berger, B.：使用 Scanorama 高效整合异质的单细胞转录组数据。自然生物技术 37(6), 685–691 (2019)

+   Korsunsky 等人 [2019] Korsunsky, I., Millard, N., Fan, J., Slowikowski, K., Zhang, F., Wei, K., Baglaenko, Y., Brenner, M., Loh, P.-r., Raychaudhuri, S.：快速、灵敏且准确的单细胞数据整合方法 Harmony。自然方法 16(12), 1289–1296 (2019)

+   Johnson 等人 [2007] Johnson, W.E., Li, C., Rabinovic, A.：使用经验贝叶斯方法调整微阵列表达数据中的批次效应。生物统计学 8(1), 118–127 (2007)

+   Büttner 等人 [2019] Büttner, M., Miao, Z., Wolf, F.A., Teichmann, S.A., Theis, F.J.：用于评估单细胞 RNA-seq 批次校正的测试度量。自然方法 16(1), 43–49 (2019)

+   Korsunsky 等人 [2019] Korsunsky, I., Millard, N., Fan, J., Slowikowski, K., Zhang, F., Wei, K., Baglaenko, Y., Brenner, M., Loh, P.-r., Raychaudhuri, S.：快速、灵敏且准确的单细胞数据整合方法 Harmony。自然方法 16(12), 1289–1296 (2019)

+   Hubert 和 Arabie [1985] Hubert, L., Arabie, P.：比较分区。分类学杂志 2, 193–218 (1985)

+   Hou 和 Ji [2024] Hou, W., Ji, Z.：评估 GPT-4 在单细胞 RNA-seq 分析中进行细胞类型注释的表现。自然方法 1–4 (2024)

+   Zheng 等人 [2017] Zheng, G.X., Terry, J.M., Belgrader, P., Ryvkin, P., Bent, Z.W., Wilson, R., Ziraldo, S.B., Wheeler, T.D., McDermott, G.P., Zhu, J., 等人：单细胞的高通量数字转录组分析。自然通讯 8(1), 14049 (2017)

+   van Galen 等人 [2019] Galen, P., Hovestadt, V., Wadsworth II, M.H., Hughes, T.K., Griffin, G.K., Battaglia, S., Verga, J.A., Stephansky, J., Pastika, T.J., Story, J.L., 等人：单细胞 RNA-seq 揭示了与 AML 进展和免疫相关的分层。细胞 176(6), 1265–1281 (2019)

+   Luecken 等人 [2022] Luecken, M.D., Büttner, M., Chaichoompu, K., Danese, A., Interlandi, M., Müller, M.F., Strobl, D.C., Zappia, L., Dugas, M., Colomé-Tatché, M., 等人：单细胞基因组学中图谱级数据整合的基准测试。自然方法 19(1), 41–50 (2022)

+   Muraro et al. [2016] Muraro, M.J., Dharmadhikari, G., Grün, D., Groen, N., Dielen, T., Jansen, E., Van Gurp, L., Engelse, M.A., Carlotti, F., De Koning, E.J., et al.: 人类胰腺的单细胞转录组图谱。细胞系统 3(4), 385–394 (2016)

+   Macosko et al. [2015] Macosko, E.Z., Basu, A., Satija, R., Nemesh, J., Shekhar, K., Goldman, M., Tirosh, I., Bialas, A.R., Kamitaki, N., Martersteck, E.M., et al.: 使用纳升液滴对单个细胞进行高并行基因组范围的表达谱分析。细胞 161(5), 1202–1214 (2015)

+   Saelens et al. [2019] Saelens, W., Cannoodt, R., Todorov, H., Saeys, Y.: 单细胞轨迹推断方法的比较。自然生物技术 37(5), 547–554 (2019)

+   Todorov et al. [2020] Todorov, H., Cannoodt, R., Saelens, W., Saeys, Y.: Tinga: 使用生长神经气体进行快速灵活的轨迹推断。生物信息学 36(补充_1), 66–74 (2020)

+   Grün et al. [2016] Grün, D., Muraro, M.J., Boisset, J.-C., Wiebrands, K., Lyubimova, A., Dharmadhikari, G., Born, M., Van Es, J., Jansen, E., Clevers, H., et al.: 使用单细胞转录组数据对干细胞身份的全新预测。细胞干细胞 19(2), 266–277 (2016)

+   Cannoodt et al. [2016] Cannoodt, R., Saelens, W., Sichien, D., Tavernier, S., Janssens, S., Guilliams, M., Lambrecht, B., Preter, K.D., Saeys, Y.: Scorpius改进了轨迹推断并识别了树突状细胞发育中的新模块。Biorxiv, 079509 (2016)

+   Wolf et al. [2019] Wolf, F.A., Hamey, F.K., Plass, M., Solana, J., Dahlin, J.S., Göttgens, B., Rajewsky, N., Simon, L., Theis, F.J.: Paga: 图形抽象通过单细胞的拓扑保持映射调和聚类与轨迹推断。基因组生物学 20, 1–9 (2019)

+   Street et al. [2018] Street, K., Risso, D., Fletcher, R.B., Das, D., Ngai, J., Yosef, N., Purdom, E., Dudoit, S.: Slingshot: 单细胞转录组学中的细胞谱系和伪时间推断。BMC基因组学 19, 1–16 (2018)

+   Büttner et al. [2019] Büttner, M., Miao, Z., Wolf, F.A., Teichmann, S.A., Theis, F.J.: 一种评估单细胞RNA-seq批次校正的测试度量。自然方法 16(1), 43–49 (2019)

+   Pedregosa et al. [2011] Pedregosa, F., Varoquaux, G., Gramfort, A., Michel, V., Thirion, B., Grisel, O., Blondel, M., Prettenhofer, P., Weiss, R., Dubourg, V., et al.: Scikit-learn: Python中的机器学习。机器学习研究杂志 12, 2825–2830 (2011)
