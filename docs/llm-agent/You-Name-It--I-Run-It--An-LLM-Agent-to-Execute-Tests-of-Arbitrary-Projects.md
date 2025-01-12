<!--yml

类别：未分类

日期：2025-01-11 11:47:58

-->

# 你说了算，我来做：一个执行任意项目测试的LLM代理

> 来源：[https://arxiv.org/html/2412.10133/](https://arxiv.org/html/2412.10133/)

Islem Bouzenia [0000-0002-3920-3839](https://orcid.org/0000-0002-3920-3839 "ORCID identifier") 斯图加特大学，德国 [fi_bouzenia@esi.dz](mailto:fi_bouzenia@esi.dz) 和 迈克尔·普拉德尔 [0000-0003-1623-498X](https://orcid.org/0000-0003-1623-498X "ORCID identifier") 斯图加特大学，德国 [michael@binaervarianz.de](mailto:michael@binaervarianz.de)

###### 摘要。

在许多场景中，执行项目的测试套件是必不可少的，例如评估代码质量和代码覆盖率，验证开发人员或自动化工具所做的代码更改，以及确保与依赖项的兼容性。尽管这项工作非常重要，实际操作中执行项目的测试套件可能会遇到挑战，因为不同的项目使用不同的编程语言、软件生态系统、构建系统、测试框架和其他工具。这些挑战使得创建一个可靠的、适用于不同项目的通用测试执行方法变得非常困难。本文介绍了ExecutionAgent，这是一种自动化技术，它可以安装任意项目，配置它们以运行测试用例，并生成项目特定的脚本来重现该设置。我们的方案灵感来自于人类开发者处理该任务的方式，采用基于大型语言模型的代理，能够自主执行命令并与主机系统互动。该代理使用元提示（meta-prompting）来收集与给定项目相关的最新技术指南，并根据前一步骤的反馈迭代优化其过程。我们的评估将ExecutionAgent应用于50个开源项目，这些项目使用了14种不同的编程语言和多种不同的构建和测试工具。该方法成功地执行了33/55个项目的测试套件，并且与真实测试套件执行的测试结果相比，偏差仅为7.5%。这些结果比之前最好的技术提高了6.6倍。该方法带来的成本是合理的，每个项目的平均执行时间为74分钟，LLM成本为0.16美元。我们预见ExecutionAgent将成为开发人员、自动化编程工具和研究人员执行各种项目测试的宝贵工具。

^†^†版权：无

## 1. 引言

执行软件项目的测试套件是软件开发和软件工程研究过程中各种活动中的一个关键步骤。对于贡献开源项目的开发者而言，在提交拉取请求之前运行测试可以确保他们的更改不会引入回归错误。同样，越来越多的大型语言模型（LLM）代理可以自主编辑项目的代码（Bouzenia 等，[2024a](https://arxiv.org/html/2412.10133v1#bib.bib4)；Zhang 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib45)；Liu 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib21)；Tao 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib33)；Yang 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib42))，这为验证修改提供了巨大的需求（Spiess 等，[2025](https://arxiv.org/html/2412.10133v1#bib.bib32)），而执行测试套件便提供了这样一个反馈机制。最后，研究人员也依赖于运行测试，例如评估动态分析的有效性（Eghbali 和 Pradel，[2022](https://arxiv.org/html/2412.10133v1#bib.bib9)）或创建涉及测试执行的基准，例如 Defects4J（Just 等，[2014](https://arxiv.org/html/2412.10133v1#bib.bib17)），SWE-Bench（Jimenez 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib15)）和 DyPyBench（Bouzenia 等，[2024b](https://arxiv.org/html/2412.10133v1#bib.bib5)）。

不幸的是，实际上执行任意项目的测试远非简单。项目使用不同的编程语言和生态系统进行开发，每种生态系统都有其独特的工具、依赖关系和约定。复杂的依赖关系可能带来显著的挑战，尤其是在需要特定版本的库或工具时。文档往往不完整、不一致，或者完全缺失，迫使开发者推测所需的步骤。此外，项目可能对环境有隐含假设，如操作系统的特定要求或所需的系统配置，这些假设没有明确说明。测试框架的多样性进一步使得这一过程复杂化，因为每个框架都有其独特的设置和执行程序。

目前，有三种主要方法用于执行给定项目的测试，每种方法都有显著的局限性。第一种方法是手动遵循项目文档，并通过反复试错解决任何问题。这种方法耗时且随着项目数量的增加，难以扩展。第二种方法是重用项目维护者使用的现有持续集成/持续部署（CI/CD）工作流。然而，并非所有项目都有这样的工作流，即使存在，它们的执行通常依赖于特定的CI/CD平台，如GitHub Actions，而这些平台并非完全对公众开放。直接使用CI/CD工作流进一步复杂化，因为存在多个流行的平台，每个平台都有自己独特的配置脚本和技术栈。第三种方法是实现一个自动化的启发式脚本，旨在覆盖常见情况。这类脚本通常集中在单一编程语言和生态系统上，缺乏处理任意项目的灵活性，正如研究所指出的那样，这些脚本在多样化环境中的局限性。例如，Flapy（Gruber和Fraser，[2023](https://arxiv.org/html/2412.10133v1#bib.bib13)）仅适用于在PyPy上可用的Python项目，甚至在Python生态系统内，也无法成功运行许多测试套件。另一个例子是用于创建GitBug-Java的管道（Silva等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib31)），从66,042个仓库开始，最终仅成功执行了228个仓库的测试。

一个有效的解决方案需要应对多个挑战。首先，该方法应该了解针对一系列流行编程语言的最新技术和工具。其次，该技术应能够理解不完整和部分过时的文档，这是实践中常常遇到的情况。第三，该方法需要能够与系统交互，例如执行命令、监控输出和处理错误。最后，该方法必须能够评估设置过程是否成功，如果没有成功，则解决任何问题，直到测试套件成功运行。根据我们的了解，目前没有现有的技术能够解决这些挑战。这个空白为开发人员、自动化编码技术和需要可靠且可扩展解决方案的研究人员带来了显著的障碍，尤其是在各种项目中运行测试。受这些挑战的启发，本研究所探讨的问题是：*我们如何能够自动安装并运行任意项目的测试套件？*

本文提出了ExecutionAgent，这是第一个基于LLM的自主代理，用于自动设置任意项目并执行其测试套件。该方法解决了上述四个挑战，具体如下：针对第一个挑战，我们提出了“元提示（meta-prompting）”这一新概念，要求最近训练的LLM自动生成最新的、技术特定的以及编程语言特定的指导方针，而不是手动设计和硬编码提示。我们通过使用LLM解析并理解项目文档以及与项目相关的网络资源，解决了第二个挑战。为了解决第三个挑战，我们将代理与一套工具（如终端）连接，用于执行命令、监控输出并与系统互动。最后，为了应对第四个挑战，我们使代理能够基于先前步骤的反馈迭代地优化其过程，类似于人类开发人员的工作方式。

为了评估ExecutionAgent，我们将其应用于50个开源项目，这些项目使用了14种不同的编程语言和多种不同的构建与测试工具。该方法成功地设置并执行了33/50个项目的测试套件。与几种基准方法进行比较，如手动设计的脚本、针对特定语言的LLM生成脚本，以及通用LLM代理，我们发现ExecutionAgent比现有最好的技术高效6.6倍。为了验证测试执行的准确性，我们将其与手动建立的基准真值进行比较，发现测试结果（包括通过、失败和跳过的测试）与基准真值非常接近，平均偏差为7.5%。在研究该方法的成本时，我们发现平均每个项目的执行时间为74分钟，LLM的平均成本为每个项目0.16美元。总体来说，这些结果表明，ExecutionAgent有潜力成为开发人员、自动化编程工具和需要在各种项目中执行测试的研究人员的重要工具。

总结来说，本文贡献了以下内容：

1.  (1)

    第一个基于大型语言模型（LLM）的自主代理，用于自动设置任意项目并执行其测试套件。

1.  (2)

    “元提示（meta-prompting）”这一新概念，允许代理查询LLM以获取与安装和运行测试套件相关的最新指导方针和技术。

1.  (3)

    关于设计决策的若干技术见解，这些决策使得代理能够有效地与系统互动、执行命令、监控输出并处理错误。

1.  (4)

    实证证据表明，该方法能够成功执行各种项目的测试套件，明显优于现有技术。

## 2. 方法

以下介绍了我们执行任意项目测试的方法。我们首先定义我们要解决的问题（第[2.1](https://arxiv.org/html/2412.10133v1#S2.SS1 "2.1\. Problem Statement ‣ 2\. Approach ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")节），然后提供我们方法的概述（第[2.2](https://arxiv.org/html/2412.10133v1#S2.SS2 "2.2\. Overview of ExecutionAgent ‣ 2\. Approach ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")节），最后详细描述我们方法的各个组件（第[2.3](https://arxiv.org/html/2412.10133v1#S2.SS3 "2.3\. Preparation Phase ‣ 2\. Approach ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")节和第[2.4](https://arxiv.org/html/2412.10133v1#S2.SS4 "2.4\. Feedback Loop ‣ 2\. Approach ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")节）。

### 2.1\. 问题陈述

我们要解决的问题如下：给定一个软件项目，例如通过 git 仓库的 URL 或本地磁盘上的路径来标识，我们希望自动生成脚本来设置并运行该项目的测试。具体来说，期望的输出由两个脚本组成：一个用于创建隔离环境，如容器，另一个用于在隔离环境中设置项目并运行其测试。通过运行测试，我们的意思是执行项目的测试套件并收集结果，例如运行的测试数量、通过的测试和失败的测试。

我们的目标是以一种提供两项重要特性的方式来解决这个问题。首先，方法应该是与技术无关的，即它应该支持使用不同编程语言编写的项目，使用不同构建系统的项目，以及使用不同测试框架的项目。这个特性至关重要，因为它使得该方法能够广泛应用于各种项目。其次，方法应该是完全自动化的，即它不应要求任何手动干预或项目本身以外的额外信息。这个特性对于确保该方法能够在大规模使用且无需人工干预的情况下运作是至关重要的。

根据我们所知，已有的研究尚未解决该问题。特别是，现有的方法要么专注于特定的编程语言或生态系统（Gruber 和 Fraser，[2023](https://arxiv.org/html/2412.10133v1#bib.bib13)；Silva 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib31)），要么依赖于大量手动干预（Bouzenia 等，[2024b](https://arxiv.org/html/2412.10133v1#bib.bib5)）。

### 2.2\. ExecutionAgent 概述

我们通过一种方法解决了执行任意项目测试套件的问题，该方法利用了LLM和LLM代理的概念。我们所说的*LLM代理*是指一个使用LLM来自主地与一组工具进行交互，以实现特定目标的系统。我们使用的工具类似于人类在设置项目时可能使用的工具，例如通过终端可用的命令。这个方法的直觉是，LLM可以“理解”各种信息源，如项目文档、现有脚本和工具输出，并利用这种理解来决定执行给定项目测试所需的步骤。如图[1](https://arxiv.org/html/2412.10133v1#S2.F1 "图1 ‣ 2.2\. ExecutionAgent概览 ‣ 2\. 方法 ‣ 你命名，我执行：一个LLM代理用于执行任意项目的测试")所示，该方法包括两个阶段，我们在以下内容中简要描述这两个阶段。

![参考说明文字](img/3c4cbfe6b6cfb0a792a4fc5f2f6ebea9.png)

图1. ExecutionAgent概览。

##### 阶段1：准备阶段

给定一个项目代码库，该阶段收集构建代理初始提示所需的信息，这些信息被称为*提示成分*。一个关键挑战是以技术无关的方式创建这些提示成分，因为项目可能是用任何编程语言编写的，并且使用任何测试框架。我们的方法通过*元提示*这一新概念解决了这一挑战，元提示允许代理查询LLM以获取有关安装和运行测试套件的最新指南和技术。具体而言，ExecutionAgent利用元提示生成特定语言的指南、关于最新容器化技术的指南以及常用CI/CD脚本的可能位置。此外，该方法还查询网络，以收集有关如何安装给定项目的提示。收集到的信息随后作为提示成分传递到第二阶段。

##### 阶段2：反馈循环

在这一阶段，执行代理根据大型语言模型（LLM）的指导反复调用工具，以便安装项目并执行其测试套件。具体而言，执行代理反复执行三个步骤。步骤 <svg class="ltx_picture" height="15.74" id="S2.SS2.SSS0.Px2.p1.1.pic1" overflow="visible" version="1.1" width="15.74"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,15.74) matrix(1 0 0 -1 0 0) translate(7.87,0) translate(0,7.87)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -3.46 -4.46)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="6.92">1</foreignobject></g></g></svg> 查询LLM以获取下一个要执行的命令，使用动态更新的*命令提示符*，该提示符包含第一阶段的提示成分以及已调用命令的总结。步骤 <svg class="ltx_picture" height="15.74" id="S2.SS2.SSS0.Px2.p1.2.pic2" overflow="visible" version="1.1" width="15.74"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,15.74) matrix(1 0 0 -1 0 0) translate(7.87,0) translate(0,7.87)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -3.46 -4.46)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="6.92">2</foreignobject></g></g></svg> 通过调用其中一个工具执行LLM建议的命令。由于工具的输出可能冗长且包含无关信息，步骤 <svg class="ltx_picture" height="15.74" id="S2.SS2.SSS0.Px2.p1.3.pic3" overflow="visible" version="1.1" width="15.74"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,15.74) matrix(1 0 0 -1 0 0) translate(7.87,0) translate(0,7.87)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -3.46 -4.46)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="6.92">3</foreignobject></g></g></svg> 请求LLM对输出进行总结并提取最相关的信息。总结后的输出将用于更新命令提示符，三个步骤会重复进行，直到代理确定测试套件已成功执行。整个过程由我们称之为控制中心的组件进行管理。

执行代理采取的步骤在算法[1](https://arxiv.org/html/2412.10133v1#alg1 "算法 1 ‣ 阶段 2: 反馈循环 ‣ 2.2 执行代理概述 ‣ 2 方法 ‣ 你说，我做：一个执行任意项目测试的LLM代理")中有更详细的描述，我们将在接下来的章节中进行解释。

算法 1 执行代理的高级算法

0:  仓库 URL $u$0:  设置和运行测试的脚本1:  // 阶段 1：准备2:  $\mathit{lang\_guidelines}\leftarrow$ LLM("提供语言特定的指南", get_languages($u$))3:  $\mathit{container\_guidelines}\leftarrow$ LLM("提供关于最近容器化技术的指南")4:  $\mathit{ci\_paths}\leftarrow$ LLM("提供仓库中常见的 CI 脚本位置")5:  $\mathit{ci\_hints}\leftarrow$ read_ci_scripts_and_summarize($u$, $\mathit{ci\_paths}$)6:  $\mathit{install\_hints}\leftarrow$ search_web($u$)7:  8:  // 阶段 2：反馈循环9:  $\mathit{attempts\_left}\leftarrow 3$10:  $\mathit{p\_attempt\_lessons}\leftarrow$ ””11:  while $\mathit{attempts\_left}$ do12:     $\mathit{p\_cmd}\leftarrow$ create_command_prompt($\mathit{lang\_guidelines}$, $\mathit{container\_guidelines}$, $\mathit{ci\_hints}$, $\mathit{install\_hints}$, $\mathit{p\_attempts\_lessons}$),13:     $\mathit{budget\_left}\leftarrow$ 4014:     while $\mathit{budget\_left}$ do15:        // 步骤 1：获取下一个命令16:        $\mathit{thought},\mathit{cmd\leftarrow}$ LLM($\mathit{p\_cmd}$)17:        // 步骤 2：执行命令18:        $\mathit{cmd\_result}\leftarrow$ execute($\mathit{cmd}$)19:        if $\mathit{cmd}$ == "task_done" and valid($\mathit{cmd\_result}$) then20:           $\mathit{scripts}\leftarrow$ read_target_files()21:           return  $\mathit{scripts}$22:        end if23:        // 步骤 3：总结命令输出24:        $\mathit{p\_summarize}\leftarrow$ create_prompt($\mathit{p\_cmd}$, $\mathit{cmd\_result}$)25:        $\mathit{cmd\_result\_summary}\leftarrow$ LLM($\mathit{p\_summarize}$)26:        $\mathit{p\_cmd}\leftarrow$ update_prompt($\mathit{p\_cmd}$, $\mathit{thought}$, $\mathit{cmd}$, $\mathit{cmd\_result\_summary}$)27:        $\mathit{budget\_left}\leftarrow\mathit{budget\_left}-1$28:     end while29:     $\mathit{attempts\_left}\leftarrow\mathit{attempts\_left}-1$30:     $\mathit{p\_attempt\_lessons}\leftarrow analyze\_commands\_and\_thoughts(cmd\_% list,thoughts\_list)$31:  end while

### 2.3\. 准备阶段

准备阶段的动机有两个目标：(1) 收集与项目相关的有助于安装给定项目并运行其测试的信息；(2) 获取有助于LLM代理在第2阶段反馈循环中调用正确工具和命令的指南。为了实现这些目标，我们按照算法[1](https://arxiv.org/html/2412.10133v1#alg1 "算法1 ‣ 阶段2：反馈循环 ‣ 2.2\. ExecutionAgent概述 ‣ 2\. 方法 ‣ 你指定项目，我执行测试")中详细描述的一系列步骤进行操作，步骤位于第[1](https://arxiv.org/html/2412.10133v1#alg1.l1 "在算法1 ‣ 阶段2：反馈循环 ‣ 2.2\. ExecutionAgent概述 ‣ 2\. 方法 ‣ 你指定项目，我执行测试")行和第[6](https://arxiv.org/html/2412.10133v1#alg1.l6 "在算法1 ‣ 阶段2：反馈循环 ‣ 2.2\. ExecutionAgent概述 ‣ 2\. 方法 ‣ 你指定项目，我执行测试")行之间，我们将在下文中详细介绍。由于其中几个步骤使用了元提示的概念，我们首先解释这一概念。

#### 2.3.1\. 元提示

ExecutionAgent面临的一个关键挑战是，安装软件项目和运行测试套件的技术和最佳实践在不断发展。解决这一挑战的一种方式是花费大量时间为LLM代理构建合适的提示。这些提示应该提供如何使用常见的构建工具、包管理器、测试框架等的指南，适用于代理可能遇到的所有编程语言、软件生态系统和工具。然而，这种方法有几个缺点：它非常耗时，可能无法覆盖所有相关技术，而且可能很快过时。

与其手动构建这些提示，我们利用大型语言模型（LLM）强大的知识生成具有最新指南的提示，这些提示针对给定的项目。由于LLM通常是在大量文档上训练的，包括描述最新技术发展的文档，它们提供了一个内置机制，可以适应不断变化的技术环境。我们通过使用一种高级提示，称为*元提示*，来查询LLM获取特定技术的指南和其他信息，然后我们利用这些信息来构建代理的命令提示。具体来说，ExecutionAgent使用元提示生成三种提示成分，接下来我们将描述这些内容。

#### 2.3.2\. 语言特定指南

在最初的实验中，我们观察到 LLM 代理从描述如何典型地安装和运行用特定编程语言编写的项目的测试的指南中受益。通过采纳元提示（meta-prompting）的概念，ExecutionAgent 通过查询一个最新的 LLM 来生成一个语言特定指南的列表。为此，我们通过启发式方法确定项目的主要编程语言（使用 GitHub 的 Linguist 库¹¹1[https://github.com/github-linguist/linguist](https://github.com/github-linguist/linguist)），并利用这些信息来查询 LLM。图 [2](https://arxiv.org/html/2412.10133v1#S2.F2 "Figure 2 ‣ 2.3.2\. Language-Specific Guidelines ‣ 2.3\. Preparation Phase ‣ 2\. Approach ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects") 说明了一个 Java 项目的元提示示例以及 LLM 的响应。

<svg class="ltx_picture ltx_centering ltx_figure_panel" height="56.22" id="S2.F2.pic1" overflow="visible" version="1.1" width="603.46"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,56.22) matrix(1 0 0 -1 0 0) translate(3.46,0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 40.87)"><foreignobject color="#FFFFFF" height="11.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">指南元提示（适用于Java）</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 4.63)"><foreignobject color="#000000" height="27.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">我想安装并运行一个Java项目的测试。请列出我应遵循的10条指南，以便完成我的任务（例如，确定安装步骤、依赖关系、如何运行测试），同时考虑最新的实践和技术。</foreignobject></g></g></svg><svg class="ltx_picture ltx_centering ltx_figure_panel" height="137.89" id="S2.F2.pic2" overflow="visible" version="1.1" width="603.46"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,137.89) matrix(1 0 0 -1 0 0) translate(3.46,0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 123.89)"><foreignobject color="#FFFFFF" height="9.72" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">LLM 响应</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 4.63)"><foreignobject color="#000000" height="110.7" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">1\. 审查项目文档和README 为什么：文档通常包含安装步骤、依赖关系和设置说明。 应做什么：查看README.md或其他文档文件，了解安装和测试的先决条件和具体说明。 2\. 验证Java版本兼容性 为什么：Java项目可能针对特定版本 (…) 应做什么：通过检查pom.xml（Maven）或build.gradle（Gradle）文件，确认所需的Java版本 (…) 3\. 检查依赖管理工具 (…) 10\. 生成测试报告以供结果分析和调试 为什么：测试报告提供详细的日志和总结，帮助调试。 应做什么：Maven项目可以使用mvn surefire-report:report生成报告。Gradle通常默认生成HTML报告（build/reports/tests/test/index.html）。</foreignobject></g></g></svg>

图2. 元提示以获取Java项目的指南。为简洁起见，响应已缩短。

#### 2.3.3\. 容器指南

为了确保项目的测试在隔离的环境中执行，ExecutionAgent的目标是将项目安装并测试在容器内。类似于语言特定的指南，我们使用元提示（meta-prompting）向LLM查询有关最新和最常用的容器化技术的指南。例如，在写作时，GPT-4o模型通过响应我们的元提示，指出Docker是设置隔离环境的最佳容器化技术，并提供了使用Docker的建议。

#### 2.3.4\. 现有CI/CD脚本

<svg class="ltx_picture ltx_centering" height="1188.53" id="S2.F3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,1188.53) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.97 1173.84)"><foreignobject color="#000000" height="9.72" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.05">Summarization response</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.97 7.74)"><foreignobject color="#000000" height="1157.19" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.05">[⬇](data:text/plain;base64,eyAnc3VtbWFyeSc6ICdUaGUga2FybWEuY29uZi5qcyBmaWxlIGNvbmZpZ3VyZXMgdGhlIHRlc3RpbmcgZnJhbWV3b3JrIGZvciBCb290c3RyYXAsIHV0aWxpemluZyBKYXNtaW5lIGZvciB1bml0IHRlc3RpbmcgYW5kIFJvbGx1cCBmb3IgcHJlcHJvY2Vzc2luZy4gSXQgc3VwcG9ydHMgaGVhZGxlc3MgdGVzdGluZyB3aXRoIENocm9tZSBhbmQgRmlyZWZveCwgYW5kIGluY2x1ZGVzIGNvbmZpZ3VyYXRpb25zIGZvciBjb3ZlcmFnZSByZXBvcnRpbmcgYW5kIEJyb3dzZXJTdGFjayBpbnRlZ3JhdGlvbi4nLAogICdleHRyYWN0ZWQgZGVwZW5kZW5jaWVzJzogWydqYXNtaW5lJywgJ2thcm1hJywgJ2thcm1hLWphc21pbmUnLCdrYXJtYS1yb2xsdXAtcHJlcHJvY2Vzc29yJywgJ2thcm1hLWNocm9tZS1sYXVuY2hlcicsICdrYXJtYS1maXJlZm94LWxhdW5jaGVyJywgJ2thcm1hLWRldGVjdC1icm93c2VycycsICdrYXJtYS1jb3ZlcmFnZS1pc3RhbmJ1bC1yZXBvcnRlciddLAogICdpbXBvcnRhbnQgY29tbWFuZHMnOiBbJ25wbSB0ZXN0J10sCiAgJ2ltcG9ydGFudCBmaWxlcy9saW5rcy9oeXBlcmxpbmtzJzogW10gfQ==) {  ’summary’:  ’The  karma.conf.js  file  configures  the  testing  framework  for  Bootstrap,  utilizing  Jasmine  for  unit  testing  and  Rollup  for  preprocessing.  It  supports  headless  testing  with  Chrome  and  Firefox,  and  includes  configurations  for  coverage  reporting  and  BrowserStack  integration.’, ’extracted  dependencies’:  [’jasmine’,  ’karma’,  ’karma-jasmine’,’karma-rollup-preprocessor’,  ’karma-chrome-launcher’,  ’karma-firefox-launcher’,  ’karma-detect-browsers’,  ’karma-coverage-istanbul-reporter’], ’important  commands’:  [’npm  test’], ’important  files/links/hyperlinks’:  []  }</foreignobject></g></g></svg>

图3. LLM生成的总结示例。

一些项目包含CI/CD脚本，这些脚本提供有关安装步骤和项目依赖项的提示。为了从这些脚本中受益，ExecutionAgent会在代码库中搜索此类文件。尽管这些脚本有时可能已经过时或不完整，但现有的脚本仍然可以提供有关总体流程和非标准步骤的线索。ExecutionAgent不使用硬编码的特定文件路径，如.github/workflows/main.yml，而是通过元提示查询LLM有关代码库中CI/CD脚本的常见位置、文件名和文件扩展名。根据LLM的响应，该方法会在项目中搜索这些文件。由于识别出的文件可能很长且包含不相关的信息，我们尝试提取脚本中最相关的部分。为此，ExecutionAgent会将原始文件传给LLM，并请求模型提取四类信息：脚本的简短自然语言总结、项目的依赖项列表、设置和测试项目时的重要命令以及相关文件或链接的列表。我们称这四类信息为*总结*。图[3](https://arxiv.org/html/2412.10133v1#S2.F3 "Figure 3 ‣ 2.3.4\. Existing CI/CD Scripts ‣ 2.3\. Preparation Phase ‣ 2\. Approach ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")展示了LLM对总结提示的响应示例。

#### 2.3.5\. 网络搜索

对于流行的项目，网络上通常会有一些可以帮助安装过程的信息，这些信息超出了仓库本身提供的信息。为了获取这些信息，ExecutionAgent 会使用以下查询在搜索引擎中查询：“如何从源代码安装¡LANGUAGE¿项目’¡PROJECT¿’？”，其中¡LANGUAGE¿和¡PROJECT¿分别被替换为主要编程语言和项目名称。该方法接着从前五个搜索结果中提取所有文本，并要求LLM将其总结为与图 [3](https://arxiv.org/html/2412.10133v1#S2.F3 "Figure 3 ‣ 2.3.4\. Existing CI/CD Scripts ‣ 2.3\. Preparation Phase ‣ 2\. Approach ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects") 中显示的结构相同。如果某个搜索结果没有包含任何相关的安装提示，则模型会被指示返回“无关网页”。

### 2.4\. 反馈循环

基于在准备阶段获得的提示成分，我们方法的第二阶段是一个反馈循环，迭代地调用工具来安装项目并执行其测试套件。如算法[1](https://arxiv.org/html/2412.10133v1#alg1 "算法 1 ‣ 阶段 2: 反馈循环 ‣ 2.2\. 执行代理概述 ‣ 2\. 方法 ‣ 你指定名称，我执行它: 一个 LLM 代理执行任意项目的测试")中第[8](https://arxiv.org/html/2412.10133v1#alg1.l8 "在算法 1 ‣ 阶段 2: 反馈循环 ‣ 2.2\. 执行代理概述 ‣ 2\. 方法 ‣ 你指定名称，我执行它: 一个 LLM 代理执行任意项目的测试")行到第[31](https://arxiv.org/html/2412.10133v1#alg1.l31 "在算法 1 ‣ 阶段 2: 反馈循环 ‣ 2.2\. 执行代理概述 ‣ 2\. 方法 ‣ 你指定名称，我执行它: 一个 LLM 代理执行任意项目的测试")行所示，反馈循环由两个嵌套的循环组成。内层循环在 LLM 的引导下运行一系列命令，直到测试套件成功完成或达到可配置的命令限制（默认：40）。外层循环允许多次尝试安装和执行测试套件，且可配置最大尝试次数（默认：3）。这种设计允许在安装过程中遇到错误、导致测试无法运行的偶发失败时进行恢复。通过允许多次尝试，系统可以调整方法以克服此类错误。在每次外层循环迭代的开始，系统将从上次尝试中获得的经验教训（如果有的话）纳入命令提示中。这些经验教训是通过提示 LLM 分析前一次思考和命令序列，识别遇到的挑战，并为下一次迭代提出调整建议来生成的（第[30](https://arxiv.org/html/2412.10133v1#alg1.l30 "在算法 1 ‣ 阶段 2: 反馈循环 ‣ 2.2\. 执行代理概述 ‣ 2\. 方法 ‣ 你指定名称，我执行它: 一个 LLM 代理执行任意项目的测试")行）。内层循环的每次迭代包括三个步骤，如图[1](https://arxiv.org/html/2412.10133v1#S2.F1 "图 1 ‣ 2.2\. 执行代理概述 ‣ 2\. 方法 ‣ 你指定名称，我执行它: 一个 LLM 代理执行任意项目的测试")所示，并在下文中详细说明。

#### 2.4.1\. 步骤 1：LLM 代理选择下一个命令

我们方法的核心是一个大语言模型（LLM），它根据当前安装过程的状态选择下一个要执行的命令。为了指导LLM选择下一个命令，该方法构建了一个*命令提示符*，该提示符包含在准备阶段获得的提示成分，以及到目前为止执行的命令的摘要。命令提示符由静态部分组成，这些部分在每次使用ExecutionAgent时都是相同的，以及动态部分，这些部分要么是基于准备阶段创建的（第[12行](https://arxiv.org/html/2412.10133v1#alg1.l12 "在算法1 ‣ 第2阶段：反馈环 ‣ 2.2．ExecutionAgent概述 ‣ 2．方法 ‣ 您命名，我运行：一个执行任意项目测试的LLM代理")）要么在每次内循环迭代结束时更新（第[26行](https://arxiv.org/html/2412.10133v1#alg1.l26 "在算法1 ‣ 第2阶段：反馈环 ‣ 2.2．ExecutionAgent概述 ‣ 2．方法 ‣ 您命名，我运行：一个执行任意项目测试的LLM代理")）。具体来说，提示符包含以下部分：

1.  (1)

    代理角色（静态）：定义代理的主要任务和成功标准。具体来说，代理的角色是安装给定的项目并确保测试用例能够正确执行。

1.  (2)

    目标（静态）：概述代理需要完成的具体目标，即：

    +   •

        收集与安装相关的信息和需求。

    +   •

        编写脚本以安装项目并执行其测试。

    +   •

        运行生成的脚本并在必要时进行改进。

    +   •

        分析测试执行结果，并通过提供已执行、通过和失败的测试数量进行总结。

1.  (3)

    工具（静态）：列出代理可用的工具。我们在第[2.4.2节](https://arxiv.org/html/2412.10133v1#S2.SS4.SSS2 "2.4.2．步骤2：调用工具 ‣ 2.4．反馈环 ‣ 2．方法 ‣ 您命名，我运行：一个执行任意项目测试的LLM代理")中详细描述了这些工具。

1.  (4)

    指南（动态）：包含在准备阶段获得的编程语言特定指南和容器化指南。

1.  (5)

    安装提示（动态）：包含从现有的CI/CD脚本和网络搜索中获得的安装提示。它还包含之前尝试的经验教训（外部反馈环）。

1.  (6)

    工具调用历史（动态）：总结到目前为止调用的工具以及产生的输出。最初，这一部分是空的。

1.  (7)

    创建下一个命令提示的指令（静态）：指定 LLM 应根据提示中提供的上下文提供下一个要执行的命令。我们在一个 TypeScript 接口中定义了 LLM 响应的预期格式，如图[4](https://arxiv.org/html/2412.10133v1#S2.F4 "图 4 ‣ 2.4.1\. 第 1 步：LLM 代理选择下一个命令 ‣ 2.4\. 反馈循环 ‣ 2\. 方法 ‣ 你命名，我执行：一个用于执行任意项目测试的 LLM 代理")所示，这样 LLM 就可以以与给定接口兼容的 JSON 格式回答。预期格式包括代理的思考过程，以及要调用的工具名称和传递给工具的参数。要求 LLM 描述其思考过程的原因有两个：首先，已有实验证明，这可以提高 LLM 响应的质量（Wei 等人，[2022](https://arxiv.org/html/2412.10133v1#bib.bib38)）。其次，这有助于更容易地调试和理解 LLM 的决策。

<svg class="ltx_picture ltx_centering" height="1957.72" id="S2.F4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,1957.72) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.28 1943.59)"><foreignobject color="#000000" height="9.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="591.43">Expected format for selecting the next command</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.28 4.28)"><foreignobject color="#000000" height="1931.09" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="591.43">[⬇](data:text/plain;base64,aW50ZXJmYWNlIFJlc3BvbnNlIHsvLyBFeHByZXNzIHlvdXIgdGhvdWdodHMgYmFzZWQgb24gdGhlIGluZm9ybWF0aW9uIHRoYXQgeW91IGhhdmUgY29sbGVjdGVkIHNvIGZhciwgdGhlIHBvc3NpYmxlIHN0ZXBzIHRoYXQgeW91IGNvdWxkIGRvIG5leHQsIGFuZCBhbHNvIHlvdXIgcmVhc29uaW5nLgogIHRob3VnaHRzOiBzdHJpbmc7CiAgdG9vbDoge25hbWU6IHN0cmluZzsgYXJnczogUmVjb3JkPHN0cmluZywgYW55PjsgfTsgfQoKSGVyZSBpcyBhbiBleGFtcGxlIG9mIGEgY29tbWFuZCBjYWxsIHRoYXQgeW91IGNhbiBvdXRwdXQ6CnsgInRob3VnaHRzIjogIkkgbmVlZCB0byBjaGVjayB0aGUgZmlsZXMgYW5kIGZvbGRlcnMgYXZhaWxhYmxlIHdpdGhpbiB0aGlzIHJlcG9zaXRvcnkuIiwKICAidG9vbCI6IHsibmFtZSI6ICJsaW51eF90ZXJtaW5hbCIsICJhcmdzIjogeyJjbWQiOiAibHMifSB9IH0=) interface  Response  {//  Express  your  thoughts  based  on  the  information  that  you  have  collected  so  far,  the  possible  steps  that  you  could  do  next,  and  also  your  reasoning. thoughts:  string; tool:  {name:  string;  args:  Record<string,  any>;  };  } Here  is  an  example  of  a  command  call  that  you  can  output: {  "thoughts":  "I  need  to  check  the  files  and  folders  available  within  this  repository.", "tool":  {"name":  "linux_terminal",  "args":  {"cmd":  "ls"}  }  }</foreignobject></g></g></svg>

图 4. TypeScript 接口，用于指定 LLM 对命令提示的响应的 JSON 格式。

#### 2.4.2\. 第 2 步：调用工具

为了使我们的方法能够采取必要的步骤来安装项目并运行其测试，我们向代理提供了四个工具。ExecutionAgent 根据 LLM 对命令提示的响应调用这些工具（第[17](https://arxiv.org/html/2412.10133v1#alg1.l17 "在算法 1 ‣ 第 2 阶段：反馈循环 ‣ 2.2\. ExecutionAgent 概述 ‣ 2\. 方法 ‣ 你命名，我执行：一个用于执行任意项目测试的 LLM 代理")行至[22](https://arxiv.org/html/2412.10133v1#alg1.l22 "在算法 1 ‣ 第 2 阶段：反馈循环 ‣ 2.2\. ExecutionAgent 概述 ‣ 2\. 方法 ‣ 你命名，我执行：一个用于执行任意项目测试的 LLM 代理")行）。

##### 终端

类似于人类开发人员，访问终端对于成功设置项目和运行其测试至关重要，例如，安装依赖项、配置环境或列出可用文件。我们通过 terminal 工具为代理提供执行任何 Linux 终端中可用命令的能力。该工具接受要执行的命令作为输入，并返回命令的输出。

##### 文件 I/O 工具

虽然终端原则上足以执行系统上的任何操作，但我们提供了两个额外的工具来与文件系统进行交互。这些工具旨在模拟开发人员与文本编辑器的交互，在其中他们打开文件，读取部分内容，并有时通过写入文件进行更改。read_file 工具接受文件路径作为输入，并返回文件的内容。write_file 工具接受文件路径和内容作为输入，并将内容写入文件。后者的工具对于编写作为 ExecutionAgent 输出的脚本尤为有用，例如创建容器的 Dockerfile 和安装项目并运行测试的 install.sh 脚本。

##### 任务结束

除了上述描述的工具，我们还提供了一个名为task_done的特殊工具。该工具用于向代理发出信号，表示代理已成功完成任务，即项目已安装并执行了测试。该工具接受一个自然语言描述作为参数，解释代理为什么完成了任务。

#### 2.4.3\. 第三步：总结与提取

工具的输出可能会非常冗长，并包含大量不相关的信息。例如，假设代理执行一个命令来列出目录中的所有文件，或者读取一个文件的内容，那么仅仅将工具的输出返回给LLM进行下一个命令会迅速填满可用的提示空间。即使是提供大提示空间的LLM，减少提示中的信息量也是有益的，这样可以让代理专注于最重要的信息，并减少方法的整体成本。

为了减少由工具调用产生的文本量，当输出超过一定的令牌数（默认为200）时，ExecutionAgent会缩短输出。该方法要求另一个LLM对输出进行总结，并提取最相关的信息（行[24](https://arxiv.org/html/2412.10133v1#alg1.l24 "算法1 ‣ 阶段2：反馈循环 ‣ 2.2\. ExecutionAgent概述 ‣ 2\. 方法 ‣ 你命名，我执行：一个LLM代理用于执行任意项目的测试")至[25](https://arxiv.org/html/2412.10133v1#alg1.l25 "算法1 ‣ 阶段2：反馈循环 ‣ 2.2\. ExecutionAgent概述 ‣ 2\. 方法 ‣ 你命名，我执行：一个LLM代理用于执行任意项目的测试")）。摘要的预期格式与图[3](https://arxiv.org/html/2412.10133v1#S2.F3 "图3 ‣ 2.3.4\. 现有的CI/CD脚本 ‣ 2.3\. 准备阶段 ‣ 2\. 方法 ‣ 你命名，我执行：一个LLM代理用于执行任意项目的测试")中的示例相同。一旦最新执行的命令的输出被总结，方法会将该命令及其输出摘要附加到命令提示中的工具调用历史部分（行[26](https://arxiv.org/html/2412.10133v1#alg1.l26 "算法1 ‣ 阶段2：反馈循环 ‣ 2.2\. ExecutionAgent概述 ‣ 2\. 方法 ‣ 你命名，我执行：一个LLM代理用于执行任意项目的测试")）。

#### 2.4.4\. 控制中心

控制中心结合上述三个步骤，并管理LLM与工具之间的交互。特别地，它执行以下任务：

+   •

    解析LLM输出并验证其是否符合指定的输出格式。我们通过经验观察，几乎所有情况下LLM的输出语法都是正确的。

+   •

    按照算法[1](https://arxiv.org/html/2412.10133v1#alg1 "算法1 ‣ 第2阶段：反馈循环 ‣ 2.2. ExecutionAgent概述 ‣ 2. 方法 ‣ 你命名，我执行：一个LLM代理来执行任意项目的测试")中指定的下一步，调用特定命令。

+   •

    对于任何工具调用，检查命令是否在可配置的超时时间内返回（默认：五分钟）。如果命令未在此超时时间内终止，控制中心将向LLM代理提供迄今为止的输出，并要求在以下三种选项中做出决定：再等待五分钟、为正在运行的工具提供一些输入，或者终止命令。为已运行的工具提供输入对于需要用户输入的交互式工具非常有用，例如，用户通过输入“y”来确认是否安装某个软件包。

+   •

    清理工具的输出，移除终端颜色字符和其他特殊字符，例如，用于显示进度条的字符。

当代理选择 `task_done` 命令时，表示代理认为项目已经成功安装并且其测试套件已执行完毕。在终止ExecutionAgent之前，控制中心会验证代理是否确实达成了所有目标。这包括检查是否存在三个文件：一个用于创建容器的文件（例如Dockerfile）、一个在容器内执行的安装脚本，以及一个包含测试结果的文件（即通过、失败和跳过的测试数量）。如果所有目标都已达成，ExecutionAgent会将安装和测试脚本返回给用户（参见第[21](https://arxiv.org/html/2412.10133v1#alg1.l21 "算法1 ‣ 第2阶段：反馈循环 ‣ 2.2. ExecutionAgent概述 ‣ 2. 方法 ‣ 你命名，我执行：一个LLM代理来执行任意项目的测试")行）。否则，控制中心会向LLM代理提供反馈，指出缺少的具体内容，并要求提供新命令。

图[5](https://arxiv.org/html/2412.10133v1#S2.F5 "图5 ‣ 2.4.4. 控制中心 ‣ 2.4. 反馈循环 ‣ 2. 方法 ‣ 你命名，我执行：一个LLM代理来执行任意项目的测试")展示了ExecutionAgent为OpenVPN项目输出的脚本示例。这两个脚本显示了ExecutionAgent选择的配置、安装的依赖项，以及安装和执行该项目测试的步骤。

<svg class="ltx_picture" height="4830.71" id="S2.F5.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,4830.71) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.97 4818.05)"><foreignobject color="#000000" height="7.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.05">Dockerfile</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.97 4.97)"><foreignobject color="#000000" height="4804.16" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.05">[⬇](data:text/plain;base64,RlJPTSB1YnVudHU6MjAuMDQKIyBTZXQgdGhlIHRpbWV6b25lIHRvIGF2b2lkIGludGVyYWN0aXZlIHByb21wdHMKRU5WIFRaPVhYWC9ZWVkgIyBvbWl0dGVkIGZvciBhbm9ueW1pdHkKUlUNO25uIC1zbmYgL3Vzci9zaGFyZS96b25laW5mby8kVFogL2V0Yy9sb2NhbHRpbWUgJiYgZWNobyAkVFogPiAvZXRjL3RpbWV6b25lCiMgVXBkYXRlIHBhY2thZ2UgbGlzdHMgYW5kIGluc3RhbGwgbmVjZXNzYXJ5IGJ1aWxkIHRvb2xzIGFuZCBkZXBlbmRlbmNpZXMKUlUNO2FwdC1nZXQgdXBkYXRlICYmIFwKYXB0LWdldCBpbnN0YWxsIC15IC0tbm8taW5zdGFsbC1yZWNvbW1lbmRzIFwKc3VkbyBcCmJ1aWxkLWVzc2VudGlhbCBcCmxpYmx6bzItZGV2IFwKbGlicGFtMGctZGV2IFwKbGlibHo0LWRldiBcCmxpYmNhcC1uZy1kZXYgXApsaWJubC1nZW5sLTMtZGV2IFwKbGludXgtbGliYy1kZXYgXAphdXRvY29uZiBcCmF1dG9tYWtlIFwKbGlidG9vbCBcCmNtb2NrYS1kZXYgXApweXRob24zLWRvY3V0aWxzIFwKZ2l0IFwKJiYgYXB0LWdldCBjbGVhbiAmJiBybSAtcmYgL3Zhci9saWIvYXB0L2xpc3RzLyogfHwgZXhpdCAwCldPUktESVIgL2FwcAojIENsb25lIHRoZSBPcGVuVlBOIHJlcG9zaXRvcnkKUlUNO2dpdCBjbG9uZSBodHRwczovL2dpdGh1Yi5jb20vT3BlblZQTi9vcGVudnBuLmdpdCB8fCBleGl0IDAKV09SS0RJUiAvYXBwL29wZW52cG4KQ01EIFsiYmFzaCJd) FROM  ubuntu:20.04 #  Set  the  timezone  to  avoid  interactive  prompts ENV  TZ=XXX/YYY  #  omitted  for  anonymity RUN  ln  -snf  /usr/share/zoneinfo/$TZ  /etc/localtime  &&  echo  $TZ  >  /etc/timezone #  Update  package  lists  and  install  necessary  build  tools  and  dependencies RUN  apt-get  update  &&  \ apt-get  install  -y  --no-install-recommends  \ sudo  \ build-essential  \ liblzo2-dev  \ libpam0g-dev  \ liblz4-dev  \ libcap-ng-dev  \ libnl-genl-3-dev  \ linux-libc-dev  \ autoconf  \ automake  \ libtool  \ cmocka-dev  \ python3-docutils  \ git  \ &&  apt-get  clean  &&  rm  -rf  /var/lib/apt/lists/*  ||  exit  0 WORKDIR  /app #  Clone  the  OpenVPN  repository RUN  git  clone  https://github.com/OpenVPN/openvpn.git  ||  exit  0 WORKDIR  /app/openvpn CMD  ["bash"]</foreignobject></g></g></svg><svg class="ltx_picture" height="1056.45" id="S2.F5.2.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,1056.45) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.97 1041.51)"><foreignobject color="#000000" height="9.96" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.05">install_and_run.sh</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.97 7.74)"><foreignobject color="#000000" height="1024.86" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.05">[⬇](data:text/plain;base64,IyEvYmluL2Jhc2gKIyBSdW4gYXV0b2NvbmYgdG8gcHJlcGFyZSB0aGUgYnVpbGQgc3lzdGVtCmF1dG9yZWNvbmYgLWZ2aQojIENvbmZpZ3VyZSB0aGUgYnVpbGQgc3lzdGVtCi4vY29uZmlndXJlIC0tZW5hYmxlLXdlcnJvcgojIENvbXBpbGUgdGhlIHByb2plY3QKbWFrZSAtaiQobnByb2MpCiMgUnVuIHRlc3RzCm1ha2UgY2hlY2sgVkVSQk9TRT0xCmVjaG8gIlNldHVwIGFuZCB0ZXN0aW5nIGNvbXBsZXRlLiI=) #!/bin/bash #  Run  autoconf  to  prepare  the  build  system autoreconf  -fvi #  Configure  the  build  system ./configure  --enable-werror #  Compile  the  project make  -j$(nproc) #  Run  tests make  check  VERBOSE=1 echo  "Setup␣and␣testing␣complete."</foreignobject></g></g></svg>

图 5. ExecutionAgent 为 OpenVPN 项目生成的脚本。

## 3\. 评估

我们的评估旨在回答以下研究问题：

+   •

    RQ1（有效性）：ExecutionAgent 在正确设置项目和运行其测试用例方面的有效性如何？

+   •

    RQ2（成本）：在与 LLM 交互时，ExecutionAgent 在执行时间和令牌使用方面的成本是多少？

+   •

    RQ3（消融研究）：ExecutionAgent 不同组件和配置的影响如何？

+   •

    RQ4（轨迹）：ExecutionAgent 如何与工具进行交互？它采取了哪些轨迹来达到目标？

### 3.1\. 实验设置

#### 3.1.1\. 实现与模型

ExecutionAgent 是用 Python 和 bash 实现的。为了实现 ExecutionAgent 本身的隔离执行，我们使用了 Docker 容器。该代理由 OpenAI 的 GPT-4o-mini 模型提供支持，我们通过其 Python API 进行访问。

#### 3.1.2\. 指标

ExecutionAgent 执行的任务包括两个子任务：(i) 构建和/或安装项目，以及 (ii) 运行测试套件。我们通过衡量*成功构建率*（即成功构建和安装项目的项目比例）和*成功测试率*（即成功运行测试套件的项目比例）来评估该方法在执行这些任务时的有效性。为了确定这些比率，我们手动检查生成的脚本和测试结果。

为了更好地理解 ExecutionAgent 的输出，我们测量了生成脚本的*脚本大小*，即定义为命令的数量，排除所有注释、空白和“echo”命令。我们将复合命令拆分并单独计算其组件。例如，make . && make test 这一行被计为两个命令：make . 和 make test。

为了验证测试套件的执行情况，我们将 ExecutionAgent 产生的测试执行结果与手动建立的基准结果进行对比。基准结果是通过搜索目标项目的 CI/CD 日志获得的，并提取通过、失败和跳过的测试数量。我们将 ExecutionAgent 的结果与基准结果进行比较，主要比较这三项数字。具体而言，我们计算与基准结果相比的*相对偏差*，其公式为 $\mathit{deviation}=|\frac{(nb_{EA}-nb_{GT})}{nb_{GT}}|*100$，其中 $nb_{EA}$ 是 ExecutionAgent 产生的测试数量，$nb_{GT}$ 是基准结果中的测试数量。例如，如果 ExecutionAgent 执行的测试中有 95 个通过，而基准结果中有 100 个通过测试，那么偏差为 5%。我们分别计算通过、失败和跳过的测试数量的偏差，然后取这些偏差的平均值来得到整体偏差。

#### 3.1.3\. 数据集

我们从 GitHub 上收集了 50 个开源项目，要求这些项目具有三个特征：(i) 这些项目应该覆盖不同的编程语言。为此，我们从以下语言中抽取了每种语言十个项目，这些项目的主语言或第二主语言为该语言：Python、Java、C、C++ 和 JavaScript。由于许多项目包含多种语言的代码，因此整个数据集覆盖了 14 种编程语言。

(ii) 存在一个测试执行结果的“真实值”，我们可以从 CI/CD 平台的日志中提取这些结果。由于不同的项目使用不同的平台，我们收集了来自 GitHub Actions、CircleCI、Jenkins、CirrusCI，偶尔还会收集项目网站（例如，tensorflow）中的真实值数据。(iii) 每个项目至少有 100 次启动和 100 次提交，并且在收集前的过去 6 个月内至少有一次提交。表格 [1](https://arxiv.org/html/2412.10133v1#S3.T1 "Table 1 ‣ 3.1.3\. Dataset ‣ 3.1\. Experimental Setup ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects") 提供了数据集中项目的概览。我们数据集中的项目具有中位数 10K 次提交、45K 次星标和 1.4K 个测试用例。

表 1. 用于评估的项目。编程语言：J=Java，P=Python，JS=JavaScript，TS=TypeScript，KT=Kotlin，ASM=Assembly，SH=Shell。符号：+ 表示成功构建但未执行测试，- 表示构建失败。

| 项目 | 编程语言 | 真实值 | 执行代理 |
| --- | --- | --- | --- |
|  |  | 测试 | 通过 | 失败 | 跳过 | 测试 | 通过 | 失败 | 跳过 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Activiti | J | 3269 | 3266 | 0 | 3 | 2392 | 2389 | 1 | 2 |
| ansible | P | 1717 | 1703 | 0 | 14 | + | + | + | + |
| axios | JS, TS | 203 | 203 | 0 | 0 | 203 | 195 | 8 | 0 |
| bootstrap | JS, Html/css | 808 | 808 | 0 | 0 | 808 | 893 | 15 | 0 |
| ccache | C, SH | 47 | 36 | 0 | 11 | 47 | 36 | 0 | 11 |
| Chart.js | JS, TS | 3298 | 3298 | 0 | 0 | + | + | + | + |
| commons-csv | J | 856 | 845 | 0 | 11 | 856 | 845 | 0 | 11 |
| cpython | P, C, C++ | 478 | 462 | 0 | 16 | 478 | 447 | 0 | 31 |
| deno | JS, TS, Rust | 1458 | 1446 | 0 | 12 | 511 | 474 | 36 | 1 |
| distcc | C, P | 56 | 56 | 0 | 0 | 56 | 56 | 0 | 0 |
| django | P | 17604 | 16151 | 5 | 1448 | 17605 | 16148 | 6 | 1451 |
| dubbo | J | 14020 | 13678 | 0 | 342 | 14020 | 13678 | 2 | 342 |
| express | JS | 1228 | 1228 | 0 | 0 | 1228 | 1228 | 0 | 0 |
| flask | P | 484 | 484 | 0 | 3 | 484 | 483 | 1 | 3 |
| flink | J, Scala, P | 105242 | 100644 | 0 | 4598 | + | + | + | + |
| folly | C++, P | 3089 | 3088 | 0 | 0 | - | - | - | - |
| FreeRTOS | C, ASM | 215 | 215 | 0 | 0 | - | - | - | - |
| git | C, SH, Perl | 31715 | 31715 | 0 | 0 | 30264 | 29203 | 265 | 796 |
| guava | J | 857221 | 857221 | 0 | 0 | 857221 | 857221 | 0 | 0 |
| imgui | C++, C | 348 | 348 | 0 | 0 | - | - | - | - |
| json | C++ | 93 | 93 | 0 | 0 | 93 | 93 | 0 | 0 |
| json-c | C | 25 | 25 | 0 | 0 | 25 | 25 | 0 | 0 |
| keras | P | 12110 | 11860 | 0 | 250 | 12452 | 12445 | 7 | 0 |
| langchain | P | 1358 | 887 | 0 | 471 | + | + | + | + |
| libevent | C | 68 | 68 | 0 | 0 | 68 | 68 | 0 | 0 |
| mermaid | JS, TS | 3106 | 3104 | 0 | 2 | 3287 | 3276 | 0 | 11 |
| mpv-player | C | 14 | 14 | 0 | 0 | - | - | - | - |
| msgpack-c | C | 41 | 41 | 0 | 0 | - | - | - | - |
| mybatis | J | 3702 | 3664 | 0 | 38 | 3702 | 3664 | 0 | 38 |
| nest | JS | 1655 | 1655 | 0 | 0 | 1655 | 1655 | 0 | 0 |
| node | JS, C++, P | 4218 | 4218 | 0 | 0 | - | - | - | - |
| numpy | P, C, C++ | 44861 | 43197 | 0 | 1629 | - | - | - | - |
| opencv | C++, C, P | 232 | 216 | 0 | 16 | 232 | 216 | 0 | 16 |
| openvpn | C | 96 | 95 | 0 | 1 | 91 | 85 | 2 | 4 |
| pandas | P, C | 199448 | 172973 | 1025 | 25486 | 183384 | 171565 | 2233 | 9586 |
| pytest | P | 3762 | 3640 | 11 | 111 | 3762 | 3757 | 5 | 0 |
| react | JS, TS, Rust | 5509 | 5509 | 0 | 0 | 9736 | 0 | 0 | 0 |
| react-native | C++, J, JS | 158 | 158 | 0 | 0 | 158 | 158 | 0 | 0 |
| rocketmq | J | 2336 | 2324 | 0 | 12 | 2336 | 2324 | 0 | 12 |
| RxJava | J | 11863 | 9404 | 0 | 2459 | 11863 | 9395 | 9 | 2459 |
| scikit-learn | P, C, C++ | 37064 | 31900 | 125 | 5039 | 37729 | 32546 | 128 | 5055 |
| scipy | P, C, C++ | 52672 | 49745 | 0 | 2767 | + | + | + | + |
| spring | J, KT | 117 | 74 | 0 | 43 | 117 | 74 | 0 | 43 |
| tensorflow | C++, P | 3148 | 0 | 0 | 0 | + | + | + | + |
| TypeScript | JS | 96598 | 96598 | 0 | 0 | 96598 | 96598 | 0 | 0 |
| vue | TS, JS | 1449 | 1449 | 0 | 0 | 1449 | 1449 | 0 | 0 |
| webpack | JS | 28935 | 28815 | 0 | 120 | - | - | - | - |
| webview | C++, C, P | 12 | 12 | 0 | 0 | - | - | - | - |
| xgboost | C++, P, R | 208 | 207 | 0 | 1 | + | + | + | + |
| xrdp | C, C++ | 206 | 206 | 0 | 0 | + | + | + | + |

#### 3.1.4\. 基线

据我们所知，目前没有任何技术能够解决与ExecutionAgent相同的问题。然而，我们将我们的方法与三个相关的基线方法进行了比较，这些基线代表了该领域的最新技术。

##### LLM脚本

鉴于LLMs已经接触到大量的数据，包括如何安装项目的文档，我们利用这一特点，要求LLM生成一个脚本，用于安装并运行一个特定编程语言的任意项目的测试用例。我们使用的提示语如图[6](https://arxiv.org/html/2412.10133v1#S3.F6 "Figure 6 ‣ Flapy ‣ 3.1.4\. Baselines ‣ 3.1\. Experimental Setup ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")所示。由于我们的数据集聚焦于五种主要语言，我们为这五种语言中的每一种生成了一个专门的脚本。接下来，尝试使用与项目主要语言对应的脚本来安装每个项目。

##### AutoGPT

与其为自动设置任意项目的任务创建一个专门的代理，不如使用一个通用的基于 LLM 的代理。我们与这样的代理 AutoGPT²²2[https://github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) 进行比较（Yang 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib41)），该代理在给定任务描述后，能够自主推理任务，制定计划，执行并在多个迭代中更新计划。与 ExecutionAgent 类似，AutoGPT 也可以调用工具，如网页搜索、读写文件和执行 Python 代码。作为任务描述，我们提供了图 7 中所示的输入[7](https://arxiv.org/html/2412.10133v1#S3.F7 "Figure 7 ‣ Flapy ‣ 3.1.4\. Baselines ‣ 3.1\. Experimental Setup ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")。我们为 AutoGPT 提供了与 ExecutionAgent 相同的模型和预算（最大迭代次数）。

##### Flapy

该基准是一个人工编写的脚本，用于自动设置任意 Python 项目并运行其测试用例。³³3[https://github.com/se2p/FlaPy](https://github.com/se2p/FlaPy) 该脚本最初作为测试不稳定性的研究的一部分而开发（Gruber 和 Fraser，[2023](https://arxiv.org/html/2412.10133v1#bib.bib13)），并且按设计仅限于 Python 项目。

<svg class="ltx_picture ltx_centering" height="70.37" id="S3.F6.pic1" overflow="visible" version="1.1" width="603.46"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,70.37) matrix(1 0 0 -1 0 0) translate(3.46,0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 56.24)"><foreignobject color="#FFFFFF" height="9.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">Prompt for LLM scripts baseline</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 4.63)"><foreignobject color="#000000" height="43.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">Create a script that automatically installs a ¡LANGUAGE¿ project (on an Ubuntu Linux machine) from source code and runs test cases. The script should account for differences between projects, test frameworks, and dependencies installation. The script should be as general as possible, but should also handle special cases that you are aware of.</foreignobject></g></g></svg>

图 6. 用于生成通用安装脚本的提示（“LLM 脚本”基准）。

<svg class="ltx_picture ltx_centering" height="138.01" id="S3.F7.pic1" overflow="visible" version="1.1" width="603.46"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,138.01) matrix(1 0 0 -1 0 0) translate(3.46,0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 123.89)"><foreignobject color="#FFFFFF" height="9.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">Input to AutoGPT baseline</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 4.63 4.63)"><foreignobject color="#000000" height="110.7" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="590.74">You are an AI assistant specialized in automatically setting up a given project and running its test cases. For your task, you must fulfill the following goals: 1\. Gather installation-related information and requirements for the project ¡GITHUB URL¿ 2\. Write a bash script (.sh) that allows to install dependencies, prepare the environment, and launch test case execution. 3\. Refine the script if necessary: If an error happens or the output is not expected, refine the script. 4\. Once the script launches the test suite successfully, analyze the results of running the test suite to further check whether there are any major problems (for example, some test cases would fail because the project or environment is not well configured, which would mean that the previous goals were not achieved).</foreignobject></g></g></svg>

图 7. 给 AutoGPT 基准提供的输入

### 3.2. 效果

##### ExecutionAgent

应用 ExecutionAgent 于 50 个项目的结果见表格 [1](https://arxiv.org/html/2412.10133v1#S3.T1 "Table 1 ‣ 3.1.3\. Dataset ‣ 3.1\. Experimental Setup ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")，并在表格 [2](https://arxiv.org/html/2412.10133v1#S3.T2 "Table 2 ‣ Comparison with Flapy ‣ 3.2\. Effectiveness ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects") 中进行了总结。ExecutionAgent 成功安装并测试了 50 个项目中的 33 个。在这 33 个成功测试的项目中，ExecutionAgent 在 17/33 项目的结果与真实值一致，而剩余的 16/33 项目的平均偏差为 15.4%。在表格 [2](https://arxiv.org/html/2412.10133v1#S3.T2 "Table 2 ‣ Comparison with Flapy ‣ 3.2\. Effectiveness ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects") 中，如果平均偏差小于 10%，我们认为 ExecutionAgent 的结果与真实值*接近*。总体而言，这些结果表明，ExecutionAgent 在执行大量项目的测试用例方面是有效的，并且在大多数情况下，其结果接近或等于真实值。

为了更好地理解结果，我们根据主要编程语言对项目进行分析（图[9](https://arxiv.org/html/2412.10133v1#S3.F9 "图 9 ‣ 3.3\. 成本 ‣ 3\. 评估 ‣ 你说名字，我来执行：一个 LLM 代理来执行任意项目的测试")）。对于每种语言，我们展示了构建失败、成功构建并且测试套件执行的项目数量。结果表明，ExecutionAgent 对 Java 最为有效，在该语言中，它能够成功地构建并测试所有项目。最难处理的两种语言是 C 和 C++，我们认为这是由于这些语言的构建和测试过程缺乏标准化，通常需要重新编译包以使其与当前系统依赖和项目要求兼容。

##### 与 LLM 脚本的比较

在表[2](https://arxiv.org/html/2412.10133v1#S3.T2 "表 2 ‣ 与 Flapy 的比较 ‣ 3.2\. 效果 ‣ 3\. 评估 ‣ 你说名字，我来执行：一个 LLM 代理来执行任意项目的测试")中，我们将 ExecutionAgent 与三个基准进行比较。通用的 LLM 生成脚本能够成功构建许多项目（29/50），但通常无法执行测试套件（仅有 5/29 成功）。通过检查结果，我们发现 LLM 脚本通常没有考虑到普通用户与开发环境之间所需设置的差异。开发环境通常涉及额外的步骤，比如安装额外的软件（例如，编译器、glibc-32 或测试框架）、重新编译某些依赖项或修改配置文件。相比之下，使用环境设置要简单得多，通常使用一个已经完成的需求/设置文件，例如 Python 的 setup.py 或 Java 的 pom.xml。由于 LLM 脚本没有考虑到这些差异，它们通常无法执行测试套件，而 ExecutionAgent 则能够迭代地修复意外的错误。

##### 与 AutoGPT 的比较

即使 AutoGPT 拥有与 ExecutionAgent 相同的预算，它也只能构建 9/50 个项目，并成功运行其中 4 个项目的测试套件。结果表明，运行任意项目的测试任务并非 trivial，专门的代理，例如 ExecutionAgent，在这类任务中更为有效。从根本上讲，ExecutionAgent 与 AutoGPT 的不同之处在于，它使用了元提示（meta-prompting），更有效地管理已经执行命令的内存，并且使用了更复杂和更强大的工具。

##### 与 Flapy 的比较

表[2](https://arxiv.org/html/2412.10133v1#S3.T2 "表 2 ‣ 与Flapy的比较 ‣ 3.2\. 有效性 ‣ 3\. 评估 ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")最右列与Flapy基线进行了比较。由于该基线仅针对Python，我们仅将其应用于数据集中10个主要使用Python的项目。结果表明，Flapy在构建项目方面与LLM生成的通用脚本结果相似。然而，像LLM脚本一样，它在执行测试套件时也存在困难，并且无法成功完成这10个项目的任何测试。对于这10个Python项目，ExecutionAgent成功构建并测试了其中6个，所有测试结果均与真实值一致。

表 2. ExecutionAgent的有效性及与基线的比较。

|  | ExecutionAgent | LLM脚本 | AutoGPT | Flapy |
| --- | --- | --- | --- | --- |
| 已构建/已安装 | 41 / 50 | 29 / 50 | 9 / 50 | 6 / 10 |
| 执行的测试 | 33 / 50 | 5 / 50 | 4 / 50 | 0 / 10 |
| 接近真实值的结果 | 29 / 50 | 4 / 50 | 2 / 50 | 0 / 10 |

### 3.3\. 成本

我们从执行时间和令牌使用量的角度评估了ExecutionAgent所产生的成本。ExecutionAgent处理一个项目所需的平均时间为74分钟（在一台256GB内存的Xeon(R) Silver 4214机器上，并行处理五个项目）。考虑到最可靠的替代方法是手动设置并测试这些项目，我们认为ExecutionAgent的时间成本是可以接受的。

ExecutionAgent依赖于大型语言模型，这会产生基于令牌使用量计算的成本。图[9](https://arxiv.org/html/2412.10133v1#S3.F9 "图 9 ‣ 3.3\. 成本 ‣ 3\. 评估 ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")显示了因使用LLM而产生的货币成本。根据当前定价，处理一个项目的平均成本为0.16美元。成本的差异主要取决于ExecutionAgent是否成功构建和测试项目。对于构建或测试失败的项目，成本较高，因为ExecutionAgent会尝试修复任何问题，直到耗尽预算，从而导致平均成本为0.34美元。相反，对于成功的项目，成本较低，平均仅为0.10美元。总体而言，考虑到该方法的好处，我们认为当前的成本是可以接受的，并且随着LLM的效率提高和成本降低，未来成本有望进一步下降。

![参见说明](img/4225420564e2fea57c67305a51ce8ab1.png)

图 8. 按主要语言划分的ExecutionAgent有效性。

![参见说明](img/3aee5d21d1ed80dbb1252d6eb0ef40e5.png)

图 9. 由于LLM使用产生的货币成本。

### 3.4\. 消融实验

为了理解ExecutionAgent两个阶段的重要性，我们进行了一个消融研究，研究了我们方法的两种变体：（1）没有准备阶段的ExecutionAgent：在该变体中，该方法直接开始反馈循环，即没有通过元提示和网络搜索获得的信息。（2）没有反馈循环的ExecutionAgent：该变体用一个单一查询替代了反馈循环，查询提供了准备阶段收集的信息，并要求LLM生成一个完整的脚本来安装给定的项目并运行其测试。由于预算限制，我们在十个项目上随机子样本运行这些变体，同时确保每种主要语言至少包括两个项目。

这两种变体的结果，以及完整方法的结果，都在表格[3](https://arxiv.org/html/2412.10133v1#S3.T3 "表 3 ‣ 3.4\. 消融研究 ‣ 3\. 评估 ‣ 你说名称，我执行：一个执行任意项目测试的LLM代理")中呈现。我们发现，完整方法是最有效的，构建成功率为8/10，测试执行成功率为7/10。相比之下，省略任何一个阶段都会显著降低效果，特别是没有反馈变体仅成功执行了1个测试套件。由于这些变体整体执行的工作较少，它们的成本也较低。然而，适中的每个项目成本，以及完整方法的明显优势，展示了ExecutionAgent的价值。

省略准备阶段导致代理没有具体的计划，并且忽略了最佳实践。没有反馈循环的情况下，代理与LLM脚本基线类似，但在提示中包含了更多的信息和上下文。

表3. 使用ExecutionAgent变体的消融研究。

|  | ExecutionAgent | 无准备阶段 | 无反馈阶段 |
| --- | --- | --- | --- |
| 构建/安装 | 8 / 10 | 1 / 10 | 7 / 10 |
| 执行的测试 | 7 / 10 | 0 / 10 | 1 / 10 |
| 与真实结果相似的结果 | 7 / 10 | 0 / 10 | 1 / 10 |
| 每个项目的成本（美元） | 0.12 | 0.10 | 0.01 |

### 3.5\. 分析与讨论

#### 3.5.1\. 工具使用情况

为了理解ExecutionAgent的行为，我们首先对其工具使用情况进行了定量分析。总体来看，该方法发起的调用中，74.8%涉及工具终端，16.0%和8.2%分别涉及write_file和read_file工具。“任务结束”工具占所有工具调用的1%。这项高层次的分析表明，ExecutionAgent使用了所有可用的工具，且终端似乎是不可或缺的。

为了进一步揭示通过终端执行的命令，我们统计了所有调用命令的频率。图 [10](https://arxiv.org/html/2412.10133v1#S3.F10 "Figure 10 ‣ 3.5.1\. Tools Usage ‣ 3.5\. Analysis and Discussion ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects") 显示了结果，其中“其他”表示所有未明确显示的命令。我们发现，像 ls、cd 和 pwd 这样的目录和文件探索命令是常见的，占总数的 33%。另一个流行的命令子集（21%）是安装和构建命令，如用于系统包的 apt install、用于 Python 的 pip 和用于 JavaScript 的 npm。最后，“其他”命令的高频率表明 ExecutionAgent 使用了广泛的命令，这表明硬编码一组固定命令是无效的。

![参见说明文字](img/dec8092270cf0af8a2fa41e147fac094.png)

图 10. 最常用终端命令的分布（./file 表示调用本地脚本）。

#### 3.5.2\. 轨迹分析

我们还对代理的轨迹进行了定性分析，其中“轨迹”指的是 ExecutionAgent 在处理特定项目时所采取的步骤序列。以下描述了在此分析中得出的三项观察结果。

##### 反复出现的阶段：设置、依赖、测试

许多项目的轨迹，尤其是像 react-native、commons-csv 和 pytest 等软件库，通常包括三个明显可识别的阶段：（1）*设置*：代理通常从准备命令开始，如 ls，然后读取必要的文件（例如 README.md、pom.xml 或 package.json）以了解项目结构。（2）*依赖管理*：创建 Dockerfile 或设置脚本等命令（如 write_to_file）很常见。例如，在创建 Dockerfile 时，该方法指示使用的镜像，如 FROM node:20 或 FROM python:3.10-slim，然后通过调用额外的命令来使用 apt-get 和 npm 或 yarn 等包管理器安装依赖项。（3）*执行测试*：随后，该方法会发出命令以运行测试（例如 npm test、pytest），然后确认测试是否按预期执行。成功的测试执行通常会跟随日志记录结果，并创建 ExecutionAgent 负责生成的脚本。

##### 一致的 Docker 使用

许多轨迹涉及使用 Docker 来为应用程序创建隔离的环境（例如，docker build，docker run）。我们观察到 Docker 的两个具体用途：（1）*构建和运行容器*：对于提供自己 Dockerfile 的项目，代理通常使用该 Dockerfile 来创建环境并安装依赖项。（2）*调试和验证环境*：ExecutionAgent 使用像 docker images 和 docker ps 这样的命令来检查容器镜像是否已创建，并检查正在运行的容器，强调确保运行时环境已正确配置。

##### 错误处理和弹性

ExecutionAgent 展示了有效处理错误和意外结果的能力。特别是，我们观察到以下策略：（1）*回退操作*：生成的脚本中的许多命令包含回退逻辑，例如使用 || 指定命令失败时的备用操作。（2）*清理命令*：命令有时会与清理操作结合，以确保环境保持一致的状态，并且在发生错误时命令不会积累不必要的文件。

#### 3.5.3\. ExecutionAgent 生成脚本的复杂性

我们通过测量脚本的大小（见第[3.1.2节](https://arxiv.org/html/2412.10133v1#S3.SS1.SSS2 "3.1.2\. Metrics ‣ 3.1\. Experimental Setup ‣ 3\. Evaluation ‣ You Name It, I Run It: An LLM Agent to Execute Tests of Arbitrary Projects")）来评估 ExecutionAgent 生成脚本的复杂性。在50个项目中，平均脚本大小为16，最小为7，最大为31。也就是说，最终的设置需要运行16个命令，平均而言。虽然这个数字看起来不算特别高，但这些脚本非常简洁，因为代理一旦找到了成功的命令序列，就会生成它们。

#### 3.5.4\. 限制

通过手动轨迹分析，我们识别出两个常见的行为，这些行为往往导致未能成功执行测试。首先，代理通常会重复同样的错误，导致命令浪费。一个常见的模式是当 sudo 不可用时尝试使用它，结果是代理在出错后进行自我纠正，但在后续命令中仍然重复同样的错误。其次，ExecutionAgent 经常未能跟进某些命令。例如，在安装 Node 或 gcc 的新版本时，除非明确更改，否则旧版本通常仍然是默认版本。这种疏忽会导致反复检查版本、安装新版本和未能将其设置为默认版本，从而造成错误和浪费资源。

## 4\. 有效性威胁

我们的结果可能面临一些有效性威胁。首先，ExecutionAgent 通常在单一配置下运行测试（例如，某个语言版本、浏览器或操作系统），这可能无法捕捉到不同环境之间的差异，例如多个浏览器或操作系统版本。这一限制可以通过允许用户修改提示语来指定所需的配置来解决。其次，该方法是围绕现代技术设计的（例如，元提示中要求的技术），这可能会限制其在需要旧版依赖的项目中的有效性。然而，这一限制通过该技术的迭代方法得到缓解，使其在必要时能够检测并适应遗留依赖。最后，该方法主要在流行项目中进行了测试，这些项目通常拥有相对较好的文档。在不太知名或文档较差的项目中，结果可能会有所不同。

## 5. 相关工作

##### 软件工程中的大型语言模型

软件工程领域近年来见证了LLM使用的快速增长。为给定函数级注释生成代码已成为评估LLM能力的标准任务（Chen等人，[2021](https://arxiv.org/html/2412.10133v1#bib.bib6)；Liu等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib20)）。为了使代码生成能够在真实的软件开发中实际应用，研究人员提出了通过增强提示语来加入仓库级上下文的技术（Ding等人，[2022](https://arxiv.org/html/2412.10133v1#bib.bib8)；Zhang等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib44)；Shrivastava等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib30)；Eghbali和Pradel，[2024](https://arxiv.org/html/2412.10133v1#bib.bib10)）。除了生成应用程序代码，LLM还被用于生成单元测试（Lemieux等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib19)；Schäfer等人，[2024](https://arxiv.org/html/2412.10133v1#bib.bib28)；Ryan等人，[2024](https://arxiv.org/html/2412.10133v1#bib.bib27)；Kang等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib18)；Feng和Chen，[2024](https://arxiv.org/html/2412.10133v1#bib.bib11)；Pizzorno和Berger，[2024](https://arxiv.org/html/2412.10133v1#bib.bib24)） ，用于将代码从一种语言翻译成另一种语言（Rozière等人，[2020](https://arxiv.org/html/2412.10133v1#bib.bib26)），以及对接受代码作为输入的程序进行模糊测试，如编译器（Xia等人，[2024](https://arxiv.org/html/2412.10133v1#bib.bib40)）。这些方法展示了LLM在自动化需要理解和生成代码的软件开发任务中的潜力。除了从零开始创建新代码，LLM还被用于修改现有代码。一个研究方向集中于基于先前执行的编辑预测代码编辑（Wei等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib37)；Gupta等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib14)）。其他研究则尝试自动化常见的代码更改，如重构（Dilhara等人，[2024](https://arxiv.org/html/2412.10133v1#bib.bib7)）。Meta的一个团队展示了如何使用LLM来增强现有的人工编写测试（Alshahwan等人，[2024](https://arxiv.org/html/2412.10133v1#bib.bib2)）。最后，还有关于多步代码编辑的研究（Bairi等人，[2023](https://arxiv.org/html/2412.10133v1#bib.bib3)），在此过程中，LLM首先规划多个编辑步骤，然后依次执行它们。

我们的工作与上述所有工作不同，重点并不直接放在代码上，而是着眼于设置和运行软件项目的测试套件这一任务。我们设想基于LLM的代码生成和代码编辑技术将从我们的工作中受益，通过使用可执行的测试套件作为反馈信号来评估生成代码的正确性。

##### 基于LLM的代理

最近的研究开始探索基于 LLM 的代理在软件工程任务中的潜力。最著名的代理集中在自动化程序修复上，例如在 RepairAgent（Bouzenia 等，[2024a](https://arxiv.org/html/2412.10133v1#bib.bib4)）中，以及自动处理描述代码库错误、缺失特性和其他改进的问题，例如在 SWE-Agent（Yang 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib42)）、MarsCode Agent（Liu 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib21)）、Magis（Tao 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib33)）和 AutoCodeRover（Zhang 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib45)）中。另一类工作探索了一个代理尝试描述软件故障的根本原因（Roy 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib25)）。我们参考一项最新的调查，以获得有关近期工作的更全面概述（Jin 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib16)）。据我们所知，我们的工作是首个探索使用基于 LLM 的代理来设置和运行软件项目测试套件的研究。

自主 LLM 代理的概念，当然，不仅限于软件工程领域。在 2022 年，研究人员提出让 LLM 生成并执行代码来回答给定的问题（Gao 等，[2022](https://arxiv.org/html/2412.10133v1#bib.bib12)）。在此基础上，其他人提出通过 API 调用其他工具来增强 LLM（Schick 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib29)；Patil 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib23)）。ReAct 工作（Yao 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib43)）表明，LLM 代理在多种任务中能够超越单独的 LLM，例如，问答、事实验证以及交互式决策任务，如网页导航。Copra 是一种基于代理的方法，用于形式化定理证明（Thakur 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib34)）。两项最新的调查给出了这种“增强型 LLMs”（Mialon 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib22)）和 LLM 代理（Wang 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib36)）的全面概述。我们的评估与通用的基于 LLM 的代理 AutoGPT 进行比较，显示 ExecutionAgent 在设置和运行软件项目测试套件的任务中优于 AutoGPT。

##### 依赖于测试套件执行的基准测试

除了帮助开发人员，执行测试套件的能力对于研究人员也至关重要。特别是，多个流行的基准测试依赖于测试套件执行，如 Defects4J（Just 等， [2014](https://arxiv.org/html/2412.10133v1#bib.bib17)）、BugsInPy（Widyasari 等，[2020](https://arxiv.org/html/2412.10133v1#bib.bib39)）、SWE-bench（Jimenez 等，[2023](https://arxiv.org/html/2412.10133v1#bib.bib15)）和 DyPyBench（Bouzenia 等，[2024b](https://arxiv.org/html/2412.10133v1#bib.bib5)）。这些基准测试用于各种目的，例如评估故障定位、自动程序修复和动态分析。ExecutionAgent 可以通过减少设置和运行测试套件所需的手动工作，帮助自动化创建这些基准测试。

##### 测试套件执行的自动化流水线

一些研究项目探索了在更大规模上执行测试套件的自动化流水线，既用于创建基准测试，例如在 BugSwarm（Tomassi 等，[2019](https://arxiv.org/html/2412.10133v1#bib.bib35)）和 GitBug-Java（Silva 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib31)）中，或者作为实证研究的一部分（Gruber 和 Fraser，[2023](https://arxiv.org/html/2412.10133v1#bib.bib13)）。这些方法各自针对一种编程语言，例如 Java（Tomassi 等，[2019](https://arxiv.org/html/2412.10133v1#bib.bib35)；Silva 等，[2024](https://arxiv.org/html/2412.10133v1#bib.bib31)）和 Python（Gruber 和 Fraser，[2023](https://arxiv.org/html/2412.10133v1#bib.bib13)），有时还依赖于使用特定 CI/CD 平台的项目，例如 TravisCI（Tomassi 等，[2019](https://arxiv.org/html/2412.10133v1#bib.bib35)）。我们的评估通过与语言特定的基准进行实证对比，表明 ExecutionAgent 更加高效，并且支持多种语言。

## 6. 结论

在本文中，我们介绍了 ExecutionAgent，这是一种自动化技术，旨在解决跨不同软件项目执行测试套件的复杂性。通过利用基于大语言模型的代理，ExecutionAgent 可以自主安装、配置并运行任意项目上的测试，生成定制化的脚本以简化未来的测试执行。我们的方法模拟了人类开发者的决策过程，通过元提示和迭代优化适应项目特定的依赖关系和配置。对 50 个开源项目和 14 种语言的评估表明，ExecutionAgent 成功执行了大多数测试套件，并且与实际结果高度吻合，相较现有方法有了明显的改进。在合理的计算和财务成本下，ExecutionAgent 作为一种有效的技术，在开发人员、自动化编码系统以及需要可靠测试执行能力的研究人员中具有强大的潜力，能够支持多种软件生态系统。

## 7. 数据可用性

ExecutionAgent 可通过 [https://github.com/sola-st/ExecutionAgent](https://github.com/sola-st/ExecutionAgent) 获取。

## 参考文献

+   (1)

+   Alshahwan 等人 (2024) 纳迪亚·阿尔沙万 Nadia Alshahwan、朱宾·切达 Jubin Chheda、阿纳斯塔西娅·芬根诺娃 Anastasia Finegenova、贝利兹·戈凯亚 Beliz Gokkaya、马克·哈曼 Mark Harman、伊娜·哈珀 Inna Harper、亚历山德鲁·马尔吉内安 Alexandru Marginean、舒博·森古普塔 Shubho Sengupta 和 艾迪·王 Eddy Wang. 2024. 使用大型语言模型在 Meta 进行自动单元测试改进. 发表在 *FSE* 上，Vol. abs/2402.09171. [https://doi.org/10.48550/ARXIV.2402.09171](https://doi.org/10.48550/ARXIV.2402.09171) arXiv:2402.09171

+   Bairi 等人 (2023) 拉马克里希纳·拜里 Ramakrishna Bairi、阿塔尔夫·松瓦内 Atharv Sonwane、阿迪提亚·卡纳德 Aditya Kanade、瓦吉什·D·C Vageesh D C、阿伦·艾耶尔 Arun Iyer、苏雷什·帕萨萨拉蒂 Suresh Parthasarathy、斯里拉姆·拉贾马尼 Sriram Rajamani、B·阿肖克 B. Ashok 和 沙尚克·谢特 Shashank Shet. 2023. CodePlan: 使用 LLM 和规划进行仓库级编程. arXiv:cs.SE/2309.12499

+   Bouzenia 等人 (2024a) 伊斯勒姆·布兹尼亚 Islem Bouzenia、普雷姆库马尔·德万布 Premkumar Devanbu 和 迈克尔·普拉德尔 Michael Pradel. 2024a. RepairAgent: 基于 LLM 的自主程序修复代理. 预印本. arXiv:cs.SE/2403.17134

+   Bouzenia 等人 (2024b) 伊斯勒姆·布兹尼亚 Islem Bouzenia、巴贾吉·皮尤什·克里申 Bajaj Piyush Krishan 和 迈克尔·普拉德尔 Michael Pradel. 2024b. DyPyBench: 可执行 Python 软件的基准测试. 发表在 *ACM 软件工程基础国际会议 (FSE)* 上。

+   Chen 等人 (2021) 马克·陈 Mark Chen、杰里·特沃雷克 Jerry Tworek、许伍·俊 Heewoo Jun、齐名·袁 Qiming Yuan、亨里克·庞德·德·奥利维拉·平托 Henrique Ponde de Oliveira Pinto、贾里德·卡普兰 Jared Kaplan、哈里森·爱德华兹 Harrison Edwards、尤里·布尔达 Yuri Burda、尼古拉斯·约瑟夫 Nicholas Joseph、格雷格·布罗克曼 Greg Brockman、亚历克斯·雷 Alex Ray、劳尔·普里 Raul Puri、格雷琴·克鲁格 Gretchen Krueger、迈克尔·彼得罗夫 Michael Petrov、海迪·赫拉夫 Heidy Khlaaf、吉里什·萨斯特里 Girish Sastry、帕梅拉·米什金 Pamela Mishkin、布鲁克·陈 Brooke Chan、斯科特·格雷 Scott Gray、尼克·赖德 Nick Ryder、米哈伊尔·帕夫洛夫 Mikhail Pavlov、阿莱西娅·鲍尔 Alethea Power、卢卡斯·凯泽 Lukasz Kaiser、穆罕默德·巴瓦里安 Mohammad Bavarian、克莱门斯·温特 Clemens Winter、菲利普·蒂莱 Philippe Tillet、费利佩·佩特罗斯基·苏奇 Felipe Petroski Such、戴夫·卡明斯 Dave Cummings、马蒂亚斯·普拉佩尔 Matthias Plappert、福托斯·钱特齐斯 Fotios Chantzis、伊丽莎白·巴恩斯 Elizabeth Barnes、阿里尔·赫伯特-沃斯 Ariel Herbert-Voss、威廉·赫本·古斯 William Hebgen Guss、亚历克斯·尼科尔 Alex Nichol、亚历克斯·帕伊诺 Alex Paino、尼科拉斯·特扎克 Nikolas Tezak、唐杰 Jie Tang、伊戈尔·巴布什金 Igor Babuschkin、苏奇尔·巴拉吉 Suchir Balaji、山田·贾因 Shantanu Jain、威廉·桑德斯 William Saunders、克里斯托弗·赫塞 Christopher Hesse、安德鲁·N·卡尔 Andrew N. Carr、简·莱克 Jan Leike、乔舒亚·阿基姆 Joshua Achiam、维丹特·米斯拉 Vedant Misra、埃文·莫里卡瓦 Evan Morikawa、亚历克·拉德福德 Alec Radford、马修·奈特 Matthew Knight、迈尔斯·布伦迪奇 Miles Brundage、米拉·穆拉提 Mira Murati、凯蒂·梅耶 Katie Mayer、彼得·韦林德 Peter Welinder、鲍勃·麦格鲁 Bob McGrew、达里奥·阿莫代伊 Dario Amodei、山姆·麦坎德利什 Sam McCandlish、伊利亚·苏茨科夫 Ilya Sutskever 和 Wojciech Zaremba. 2021. 评估训练在代码上的大型语言模型. *CoRR* abs/2107.03374 (2021). arXiv:2107.03374 [https://arxiv.org/abs/2107.03374](https://arxiv.org/abs/2107.03374)

+   Dilhara 等人 (2024) 马琳达·迪拉哈 Malinda Dilhara、阿比拉姆·贝尔鲁尔 Abhiram Bellur、蒂莫菲·布雷克辛 Timofey Bryksin 和 丹尼·迪格 Danny Dig. 2024. 前所未有的代码变更自动化：LLM 与通过示例的转换融合. 发表在 *FSE* 上. [https://doi.org/10.48550/arXiv.2402.07138](https://doi.org/10.48550/arXiv.2402.07138)

+   Ding 等人 (2022) 杨瑞博 Ding、王子剑 Zijian Wang、瓦西·乌丁·阿赫迈德 Wasi Uddin Ahmad、穆拉利·克里希纳·拉马纳坦 Murali Krishna Ramanathan、拉梅什·纳拉帕蒂 Ramesh Nallapati、帕尔敏德·巴蒂亚 Parminder Bhatia、丹·罗斯 Dan Roth 和 Bing Xiang. 2022. CoCoMIC: 通过联合建模文件内和跨文件上下文进行代码补全. *arXiv 预印本 arXiv:2212.10007* (2022).

+   Eghbali 和 Pradel（2022）Aryaz Eghbali 和 Michael Pradel。2022年。DynaPyt：Python 动态分析框架。在 *ESEC/FSE ’22：第30届ACM欧洲软件工程联合会议暨软件工程基础研讨会*。ACM。

+   Eghbali 和 Pradel（2024）Aryaz Eghbali 和 Michael Pradel。2024年。De-Hallucinator：基于 LLM 的代码补全的迭代性基础。在 *CoRR* abs/2401.01701（2024）。[https://doi.org/10.48550/ARXIV.2401.01701](https://doi.org/10.48550/ARXIV.2401.01701) arXiv:2401.01701

+   Feng 和 Chen（2024）Sidong Feng 和 Chunyang Chen。2024年。提示即是你所需要的一切：使用大型语言模型进行自动化的 Android Bug 重放。在 *ICSE*。

+   Gao 等人（2022）Luyu Gao、Aman Madaan、Shuyan Zhou、Uri Alon、Pengfei Liu、Yiming Yang、Jamie Callan 和 Graham Neubig。2022年。PAL：程序辅助语言模型。*CoRR* abs/2211.10435（2022）。[https://doi.org/10.48550/arXiv.2211.10435](https://doi.org/10.48550/arXiv.2211.10435) arXiv:2211.10435

+   Gruber 和 Fraser（2023）Martin Gruber 和 Gordon Fraser。2023年。FlaPy：大规模挖掘易变的 Python 测试。在 *第45届IEEE/ACM国际软件工程大会：ICSE 2023伴随论文集，2023年5月14日至20日，澳大利亚墨尔本*。IEEE，127–131。[https://doi.org/10.1109/ICSE-COMPANION58688.2023.00039](https://doi.org/10.1109/ICSE-COMPANION58688.2023.00039)

+   Gupta 等人（2023）Priyanshu Gupta、Avishree Khare、Yasharth Bajpai、Saikat Chakraborty、Sumit Gulwani、Aditya Kanade、Arjun Radhakrishna、Gustavo Soares 和 Ashish Tiwari。2023年。Grace：语言模型与代码编辑相遇。在 *第31届ACM欧洲软件工程联合会议暨软件工程基础研讨会，ESEC/FSE 2023，2023年12月3日至9日，旧金山，美国*，Satish Chandra、Kelly Blincoe 和 Paolo Tonella（编）。ACM，1483–1495。 [https://doi.org/10.1145/3611643.3616253](https://doi.org/10.1145/3611643.3616253)

+   Jimenez 等人（2023）Carlos E. Jimenez、John Yang、Alexander Wettig、Shunyu Yao、Kexin Pei、Ofir Press 和 Karthik Narasimhan。2023年。SWE-bench：语言模型能解决现实世界中的 GitHub 问题吗？arXiv:cs.CL/2310.06770

+   Jin 等人（2024）Haolin Jin、Linghan Huang、Haipeng Cai、Jun Yan、Bo Li 和 Huaming Chen。2024年。从 LLM 到基于 LLM 的软件工程代理：当前挑战和未来的调查。*arXiv 预印本 arXiv:2408.02479*（2024）。

+   Just 等人（2014）René Just、Darioush Jalali 和 Michael D. Ernst。2014年。Defects4J：一个现有故障数据库，用于支持 Java 程序的控制测试研究。在 *ISSTA*，437–440。

+   Kang 等人（2023）Sungmin Kang、Juyeon Yoon 和 Shin Yoo。2023年。大型语言模型是少量示例测试者：探索基于 LLM 的通用 bug 重现。在 *第45届IEEE/ACM国际软件工程大会，ICSE*，2312–2323。[https://doi.org/10.1109/ICSE48619.2023.00194](https://doi.org/10.1109/ICSE48619.2023.00194)

+   Lemieux等人（2023）Caroline Lemieux、Jeevana Priya Inala、Shuvendu K Lahiri和Siddhartha Sen。2023年。《CODAMOSA：通过预训练的大型语言模型突破测试生成中的覆盖率瓶颈》。见于*《第45届国际软件工程会议，ICSE》*。

+   Liu等人（2023）刘家伟、夏春秋、王雨瑶和张令鸣。2023年。《你的代码是由ChatGPT生成的吗？对大规模语言模型在代码生成中的严格评估》。见于*《神经信息处理系统进展 36：2023年神经信息处理系统年会，NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10日至16日》*，Alice Oh、Tristan Naumann、Amir Globerson、Kate Saenko、Moritz Hardt和Sergey Levine（主编）。[http://papers.nips.cc/paper_files/paper/2023/hash/43e9d647ccd3e4b7b5baab53f0368686-Abstract-Conference.html](http://papers.nips.cc/paper_files/paper/2023/hash/43e9d647ccd3e4b7b5baab53f0368686-Abstract-Conference.html)

+   Liu等人（2024）刘一洲、高鹏飞、王新辰、彭超和张赵。2024年。《MarsCode Agent：AI原生自动化错误修复》。*arXiv预印本 arXiv:2409.00899*（2024）。

+   Mialon等人（2023）Grégoire Mialon、Roberto Dessì、Maria Lomeli、Christoforos Nalmpantis、Ramakanth Pasunuru、Roberta Raileanu、Baptiste Rozière、Timo Schick、Jane Dwivedi-Yu、Asli Celikyilmaz、Edouard Grave、Yann LeCun和Thomas Scialom。2023年。《增强型语言模型：综述》。*CoRR* abs/2302.07842（2023）。[https://doi.org/10.48550/arXiv.2302.07842](https://doi.org/10.48550/arXiv.2302.07842) arXiv:2302.07842

+   Patil等人（2023）Shishir G. Patil、张天俊、王欣和Joseph E. Gonzalez。2023年。《Gorilla：与海量API连接的大型语言模型》。*CoRR* abs/2305.15334（2023）。[https://doi.org/10.48550/ARXIV.2305.15334](https://doi.org/10.48550/ARXIV.2305.15334) arXiv:2305.15334

+   Pizzorno和Berger（2024）Juan Altmayer Pizzorno和Emery D. Berger。2024年。《CoverUp：基于覆盖率引导的LLM测试生成》。arXiv:cs.SE/2403.16218

+   Roy等人（2024）Devjeet Roy、张旭超、Rashi Bhave、Chetan Bansal、Pedro Henrique B. Las-Casas、Rodrigo Fonseca和Saravan Rajmohan。2024年。《探索基于LLM的智能体在根本原因分析中的应用》。见于*《第32届ACM国际软件工程基础会议同行论文集，FSE 2024，巴西波尔图德加林哈，2024年7月15日至19日》*，Marcelo d’Amorim（主编）。ACM，208–219。[https://doi.org/10.1145/3663529.3663841](https://doi.org/10.1145/3663529.3663841)

+   Rozière 等人（2020）Baptiste Rozière、Marie-Anne Lachaux、Lowik Chanussot 和 Guillaume Lample. 2020. 无监督的编程语言翻译. 发表在 *神经信息处理系统进展 33：2020年神经信息处理系统年会 NeurIPS 2020，2020年12月6-12日，虚拟会议*，Hugo Larochelle、Marc'Aurelio Ranzato、Raia Hadsell、Maria-Florina Balcan 和 Hsuan-Tien Lin（编）。 [https://proceedings.neurips.cc/paper/2020/hash/ed23fbf18c2cd35f8c7f8de44f85c08d-Abstract.html](https://proceedings.neurips.cc/paper/2020/hash/ed23fbf18c2cd35f8c7f8de44f85c08d-Abstract.html)

+   Ryan 等人（2024）Gabriel Ryan、Siddhartha Jain、Mingyue Shang、Shiqi Wang、Xiaofei Ma、Murali Krishna Ramanathan 和 Baishakhi Ray. 2024. 代码感知提示：在回归设置中使用大语言模型进行覆盖引导的测试生成研究. 发表在 *FSE*.

+   Schäfer 等人（2024）Max Schäfer、Sarah Nadi、Aryaz Eghbali 和 Frank Tip. 2024. 使用大语言模型进行自动化单元测试生成的实证评估. *IEEE 软件工程学报* 50, 1 (2024), 85-105页. [https://doi.org/10.1109/TSE.2023.3334955](https://doi.org/10.1109/TSE.2023.3334955)

+   Schick 等人（2023）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Luke Zettlemoyer、Nicola Cancedda 和 Thomas Scialom. 2023. Toolformer: 语言模型可以自我学习使用工具. *CoRR* abs/2302.04761 (2023). [https://doi.org/10.48550/arXiv.2302.04761](https://doi.org/10.48550/arXiv.2302.04761) arXiv:2302.04761

+   Shrivastava 等人（2023）Disha Shrivastava、Hugo Larochelle 和 Daniel Tarlow. 2023. 面向代码的大语言模型的仓库级提示生成. 发表在 *国际机器学习会议*，PMLR，31693-31715页.

+   Silva 等人（2024）André Silva、Nuno Saavedra 和 Martin Monperrus. 2024. GitBug-Java: 一种可复现的近期 Java 错误基准. 发表在 *2024年 IEEE/ACM 第21届国际软件仓库挖掘会议（MSR）*，IEEE，118-122页.

+   Spiess 等人（2025）Claudio Spiess、David Gros、Kunal Suresh Pai、Michael Pradel、Md Rafiqul Islam Rabin、Amin Alipour、Susmit Jha、Premkumar Devanbu 和 Toufique Ahmed. 2025. 编程语言模型的校准与正确性. 发表在 *国际软件工程会议（ICSE）*.

+   Tao 等人（2024）Wei Tao、Yucheng Zhou、Wenqiang Zhang 和 Yu Cheng. 2024. MAGIS: 基于大语言模型的 GitHub 问题解决多代理框架. *arXiv 预印本 arXiv:2403.17927* (2024).

+   Thakur 等人（2024）Amitayush Thakur、George Tsoukalas、Yeming Wen、Jimmy Xin 和 Swarat Chaudhuri. 2024. 用于形式化定理证明的上下文学习代理. arXiv:cs.LG/2310.04353 [https://arxiv.org/abs/2310.04353](https://arxiv.org/abs/2310.04353)

+   Tomassi et al. (2019) David A Tomassi, Naji Dmeiri, Yichen Wang, Antara Bhowmick, Yen-Chuan Liu, Premkumar T Devanbu, Bogdan Vasilescu, and Cindy Rubio-González. 2019. Bugswarm: 挖掘并持续增长可重复失败和修复的数据集。*2019 IEEE/ACM 第41届国际软件工程会议（ICSE）*。IEEE，339–349.

+   Wang et al. (2023) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, Wayne Xin Zhao, Zhewei Wei, and Ji-Rong Wen. 2023. 基于大型语言模型的自主智能体调查。arXiv:cs.AI/2308.11432

+   Wei et al. (2023) Jiayi Wei, Greg Durrett, and Isil Dillig. 2023. Coeditor: 利用上下文变化进行多轮代码自动编辑。*arXiv预印本arXiv:2305.18584* (2023).

+   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. 连锁思维提示法促使大型语言模型进行推理。*神经信息处理系统进展* 35 (2022)，24824–24837.

+   Widyasari et al. (2020) Ratnadira Widyasari, Sheng Qin Sim, Camellia Lok, Haodi Qi, Jack Phan, Qijin Tay, Constance Tan, Fiona Wee, Jodie Ethelda Tan, Yuheng Yieh, et al. 2020. Bugsinpy: 一个存储Python程序现有BUG的数据库，用于支持控制性测试和调试研究。*第28届ACM联合欧洲软件工程会议暨软件工程基础研讨会论文集*。1556–1560.

+   Xia et al. (2024) Chunqiu Steven Xia, Matteo Paltenghi, Jia Le Tian, Michael Pradel, and Lingming Zhang. 2024. Fuzz4All: 使用大型语言模型进行通用模糊测试。*第46届IEEE/ACM国际软件工程会议论文集，ICSE 2024，葡萄牙里斯本，2024年4月14日至20日*。ACM，126:1–126:13. [https://doi.org/10.1145/3597503.3639121](https://doi.org/10.1145/3597503.3639121)

+   Yang et al. (2023) Hui Yang, Sifu Yue, and Yunzhong He. 2023. Auto-gpt用于在线决策：基准与附加观点。*arXiv预印本arXiv:2306.02224* (2023).

+   Yang et al. (2024) John Yang, Carlos E. Jimenez, Kilian Lieret, Shunyu Yao, Alexander Wettig, Karthik Narasimhan, and Ofir Press. 2024. SWE-Agent: 代理-计算机接口推动自动化软件工程。 (2024).

+   Yao et al. (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R. Narasimhan, and Yuan Cao. 2023. ReAct: 在语言模型中协同推理与行动。*第十一届国际学习表征会议，ICLR 2023，卢旺达基加利，2023年5月1日至5日*。OpenReview.net. [https://openreview.net/forum?id=WE_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X)

+   Zhang et al. (2023) Fengji Zhang, Bei Chen, Yue Zhang, Jin Liu, Daoguang Zan, Yi Mao, Jian-Guang Lou, and Weizhu Chen. 2023. RepoCoder: 通过迭代检索和生成进行库级代码补全。arXiv:cs.CL/2303.12570

+   张等人（2024）张云同、阮海峰、范智宇、阿比克·罗伊乔杜里。2024年。AutoCodeRover：自主程序改进。arXiv:cs.SE/2404.05427
