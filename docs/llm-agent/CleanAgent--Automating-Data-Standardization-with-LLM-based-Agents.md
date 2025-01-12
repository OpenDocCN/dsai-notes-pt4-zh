<!--yml

分类: 未分类

日期：2025-01-11 12:46:46

-->

# CleanAgent：基于 LLM 的代理自动化数据标准化

> 来源：[https://arxiv.org/html/2403.08291/](https://arxiv.org/html/2403.08291/)

Danrui Qi, Jiannan Wang 西蒙弗雷泽大学

{dqi, jnwang}@sfu.ca

###### 摘要。

数据标准化是数据科学生命周期中的一个关键部分。尽管像 Pandas 这样的工具提供了强大的功能，但它们的复杂性以及需要为不同列类型定制代码的手动工作量带来了显著挑战。尽管像 ChatGPT 这样的巨大语言模型（LLMs）已经显示出通过自然语言理解和代码生成自动化这个过程的潜力，但它仍然需要专家级的编程知识和持续的交互来优化提示。为了解决这些挑战，我们的关键想法是提出一个 Python 库，提供声明式的统一 API，用于标准化不同列类型，简化 LLM 的代码生成，通过简洁的 API 调用实现。我们首先提出了 Dataprep.Clean，它作为 Dataprep 库的一个组件，通过使得标准化特定列类型只需一行代码，大大减少了复杂性。接着我们介绍了 CleanAgent 框架，将 Dataprep.Clean 和基于 LLM 的代理整合在一起，自动化数据标准化过程。使用 CleanAgent，数据科学家只需要提供一次需求，就可以实现免手动干预的自动化标准化过程。为了展示 CleanAgent 的实际效用，我们将其开发为一个用户友好的 Web 应用程序，允许 VLDB 参会者使用真实世界的数据集与其互动。

## 1\. 引言

数据标准化在数据科学领域中至关重要，旨在将同一列中的异构数据格式转换为统一的数据格式。这个关键的数据预处理步骤对于实现有效的数据集成、数据分析和决策制定至关重要。

示例 1. 我们在图[1](https://arxiv.org/html/2403.08291v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中展示了数据标准化任务。给定一个需要标准化的输入表格$T$，显然“Admission Date”列和“Address”列的数据格式不同。“Admission Date”列中的三个单元格数据包含两种不同的日期格式。数据标准化的目标是统一$T$中每列数据的格式，以满足数据科学家的要求，并得到标准化后的表格$T^{\prime}$。在图[1](https://arxiv.org/html/2403.08291v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中，数据科学家输入他们的要求，将“Admission Date”标准化为“MM/DD/YYYY HH:MM:SS”格式。在$T^{\prime}$中，“Admission Date”列中的三个单元格数据仅遵循一种日期格式，即“MM/DD/YYYY HH:MM:SS”格式。

传统上，数据科学家在进行数据标准化任务时，通常依赖于如Pandas（McKinney等，[2024](https://arxiv.org/html/2403.08291v2#bib.bib3)）这样的库。尽管Pandas是一个强大的工具，实现数据标准化通常需要编写数百或数千行代码。单列的标准化过程涉及识别列类型，应用诸如正则表达式等复杂方法对该列中的每个单元格进行验证，并将每个单元格转换为所需的格式。此外，一个表格可能包含多列，每列可能有不同的类型，需为每种列类型编写定制的标准化代码。

示例 2. 仍然考虑图[1](https://arxiv.org/html/2403.08291v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中的数据标准化任务。为了标准化“Admission Date”和“Address”，数据科学家需要编写用于“Admission Date”的日期时间标准化代码，以及用于“Address”的地址标准化代码，通常采用正则表达式（regex）。以下是“Address”标准化的示例代码。

[⬇](data:text/plain;base64,ZGVmIHN0YW5kYXJkaXplX2FkZHJlc3MoYWRkcik6CiAgICAjIEV4dHJhY3Qgc3RyZWV0IG51bWJlciBhbmQgc3RyZWV0IG5hbWUKICAgIHN0cmVldCA9IHBkLlNlcmllcyhhZGRyKS5zdHIuZXh0cmFjdChyJyhcZCsgW14sXSspJykuc3F1ZWV6ZSgpCiAgICAjIEV4dHJhY3Qgc3RhdGUgbmFtZQogICAgc3RhdGUgPSAiTEEiCiAgICAjIEV4dHJhY3QgemlwY29rZQogICAgemlwY29rZSA9IHBkLlNlcmllcyhhZGRyKS5zdHIuZXh0cmFjdChyJyhcZHs1fSknKS5zcXVlZXplKCkKICAgICMgT3V0cHV0IHN0YW5kYXJkaXplZCBhZGRyZXNzCiAgICByZXR1cm4gZiJ7c3RyZWV0fSwge3N0YXRlfSwge3ppcGNvZGV9Ig==)1def  standardize_address(addr):2  #  提取街道编号和街道名称3  street  =  pd.Series(addr).str.extract(r’(\d+  [^,]+)’).squeeze()4  #  提取州名5  state  =  "LA"6  #  提取邮政编码7  zipcode  =  pd.Series(addr).str.extract(r’(\d{5})’).squeeze()8  #  输出标准化地址9  return  f"{street},  {state},  {zipcode}"

如果输入表格$T$包含其他列类型，如电子邮件和IP地址，数据科学家仍然需要为电子邮件类型和IP地址类型编写专门的标准化代码，这非常耗时。

最近，LLM（大语言模型）的出现，特别是ChatGPT，展现了在革新这一过程中的潜力。通过利用其自然语言理解和代码生成能力，这些模型可以通过自主生成标准化代码来显著帮助数据科学家响应对话式提示。然而，这种方法仍然需要详细的提示设计，并且通常涉及多轮对话（Chen等人，[2023](https://arxiv.org/html/2403.08291v2#bib.bib2)），逐一标准化表格$T$中的列类型，这限制了效率和实用性。

图1：CleanAgent自动化数据标准化过程的示例。

为了克服这些局限，我们的核心思想是引入一个Python库，包含专门为标准化不同列类型而设计的声明性和统一API。这个想法简化了LLM的任务，即将自然语言（NL）指令转化为简洁的声明性API调用。这样的做法简化了LLM生成数据标准化代码的过程，仅需几行代码。

然而，以上的想法引入了两个主要挑战。第一个挑战（C1）是为数据标准化设计声明性和统一的API，确保能够有效减少标准化特定列类型时的复杂性（理想情况下每个列类型一行代码）。第二个挑战（C2）则集中在优化数据科学家与LLM之间的交互。我们的目标是最小化人工干预，理想情况下让数据科学家在一次输入中提交标准化需求，从而实现自主、无需干预的数据标准化过程。

为了解决C1问题，我们在[Dataprep库](https://github.com/sfu-db/dataprep)中提出了类型特定的Clean模块，名为[Dataprep.Clean](https://github.com/sfu-db/dataprep/tree/develop/dataprep/clean)。通过观察特定列类型数据标准化的常见步骤，我们设计了统一的API `clean_type(df, column_name, target_format)`，其中type表示期望的标准化类型，如日期、地址、电话等。这些统一的API相比原始的Pandas代码，具有更强的表达能力，减少了标准化特定列类型的复杂性。使用Dataprep.Clean，数据科学家只需一行代码即可标准化某一列类型。

为了解决C2问题，我们提出了CleanAgent框架，它通过Dataprep.Clean和基于LLM的Agent（Xi等，[2023](https://arxiv.org/html/2403.08291v2#bib.bib5)；Wu等，[2023](https://arxiv.org/html/2403.08291v2#bib.bib4)）自动化数据标准化过程。一旦用户输入最终目标，基于LLM的Agent可以解放用户双手，自动生成思路并执行特定任务。CleanAgent结合了Dataprep.Clean和基于LLM的Agent的能力。数据科学家只需输入需要标准化的表格及其要求，CleanAgent将自动完成数据标准化过程，分为三个步骤：注释每一列的类型、生成简洁的标准化Python代码并执行生成的Python代码。

示例 3. 继续示例1。给定需要标准化的输入表格$T$和数据科学家的要求，CleanAgent首先识别出“Admission Date”列属于日期类型，而“Address”列属于地址类型。根据列类型注释结果，CleanAgent通过调用“clean_date”和“clean_address”函数生成并执行标准化的Python代码，然后返回标准化后的表格$T^{\prime}$。

CleanAgent作为一个Web应用构建。我们允许VLDB与会者选择样本数据并与CleanAgent进行标准化交互。我们提供了演示视频，视频可在[Youtube](https://youtu.be/fSYXVM6qeqM)上找到。

总结来说，我们做出了以下贡献：(1) 我们提出了Dataprep.Clean，一个开源库，展示了通过类型特定的标准化函数简化实现特定列类型标准化的复杂度。(2) 我们提出了CleanAgent，它通过结合Dataprep.Clean和基于LLM的Agent的优点，自动化了数据标准化过程。(3) 我们将CleanAgent部署为具有用户友好界面的Web应用，并展示了其实用性。我们还将CleanAgent的实现开源，代码可以在[Github](https://github.com/sfu-db/CleanAgent)上找到。

## 2. 类型特定标准化API设计

在本节中，我们首先描述数据标准化的常见步骤。然后，我们介绍Dataprep.Clean的特定类型API设计。

数据标准化的常见步骤。受人类标准化数据单元格步骤的启发，我们确定了数据标准化的三个常见步骤。我们以日期时间列类型为例来说明这些步骤。

假设一位数据科学家正在处理一个包含两个记录的日期时间列：“Thu Sep 25 10:36:28 2003”和“1996.07.10 AD at 15:08:56”。数据科学家希望将这个混乱的列统一成目标格式“YYYY-MM-DD hh:mm:ss”。

(1) 分割。开始时，数据科学家需要将日期时间字符串分割成几个包含特定信息的单独部分。在我们的例子中，数据科学家可以通过使用空格和冒号作为分隔符，从第一个记录中提取出几个标记{’Thu’,’Sep’,’25’,’10’,’36’,’28’,’2003’}。不同类型有其自己的分割策略，分割不一定总是产生标记。例如，数据科学家会将电子邮件字符串分割成用户名部分和域名部分。

(2) 验证。标准化只能对有效的输入执行。因此，第二步应该是验证。例如，如果字符串“little cat”是日期时间列的一个实例，则该字符串无效，数据科学家将其转换为默认值如NaN。直观地说，一个字符串有效意味着分割后的每个部分都是有效的。因此，验证每个分割部分是很重要的。通常，数据科学家会通过他们的领域知识、一些语料库或规则来识别和验证每个部分。如果每个分割部分都是有效的，那么这个字符串也是有效的。例如，标记“Sep”可以被识别为有效的月份表示，标记“2003”可以被识别为有效的年份。

(3) 转换。标准化的最后一步是将每个分割的部分转换并重新组合成目标格式。在我们的例子中，因为目标格式是“YYYY-MM-DD hh:mm:ss”，所以将“Sep”转换为数字“09”，并与其他部分重新组合为目标“2003-09-25 10:36:28”。

图2\. 基于LLM的Agent基本结构。

统一API的设计。我们API设计的目标是让数据科学家通过一次函数调用完成一个列的所有常见数据标准化步骤。简洁性和一致性被视为API设计的原则。对数据标准化常见步骤的观察带来了特定类型API设计的思想。因此，我们将API设计成以下形式：

|  | clean_type(df, column_name, target_format) |  |
| --- | --- | --- |

图3\. CleanAgent的工作流程。

其中，clean_type 是函数名称，type 代表当前列的类型。第一个参数 df 代表输入的 DataFrame，第二个参数 column_name 是需要标准化的列，第三个参数 target_type 是用户指定的目标标准化格式。我们的 API 设计灵活且可扩展，便于用户添加新的标准化函数来处理新的数据类型。目前，我们在 Dataprep.Clean 中有 142 个标准化函数，代表了 142 种数据类型。

## 3\. CleanAgent 工作流

本节首先介绍基于 LLM 的代理的基本结构。然后，我们描述了由四个代理构成的 CleanAgent 工作流。通过这四个代理的协作，可以完成自动化的数据标准化过程。

基于 LLM 的代理的基本结构。根据之前关于基于 LLM 的代理的调查研究（Xi 等人，[2023](https://arxiv.org/html/2403.08291v2#bib.bib5)），我们总结了图 [2](https://arxiv.org/html/2403.08291v2#S2.F2 "Figure 2 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中所示的基于 LLM 的代理的基本结构。一个基于 LLM 的代理包括四个主要组成部分：（1）一个用于生成对输入信息回复的 LLM，（2）一个用于存储历史对话信息的内存，（3）一个定义代理角色的系统消息，（4）一组可以被基于 LLM 的代理调用来完成特定任务的外部工具，如网页搜索、代码运行等。

详细工作流。CleanAgent 的详细工作流如图 [3](https://arxiv.org/html/2403.08291v2#S2.F3 "Figure 3 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")所示。CleanAgent 由四个代理组成，包括聊天管理器、列类型注释器、Python 程序员和代码执行器。它们可以相互通信，并通过协作自动完成数据标准化过程。如图 [2](https://arxiv.org/html/2403.08291v2#S2.F2 "Figure 2 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")所示，每个代理都有自己的内存，用于存储与其他代理之间的历史对话信息。值得强调的是，聊天管理器的内存是独一无二的、全面的，涵盖了 CleanAgent 系统中所有代理的完整历史对话信息。这种广泛的内存使得 CleanAgent 中的每个代理都能够基于完整的历史对话信息生成响应。

图 4\. CleanAgent 用户界面

CleanAgent的输入包括需要标准化的表格$T$。数据科学家还可以输入额外的要求，比如“日期类型列的格式应该是MM/DD/YYYY”。通过接收输入表格和数据科学家的额外要求，CleanAgent将这些信息存储到聊天管理器的内存中，并开始完成数据标准化过程。聊天管理器将内存中的消息传递给列类型标注器（图[3](https://arxiv.org/html/2403.08291v2#S2.F3 "Figure 3 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中的①）。然后，列类型标注器接收表格信息，并利用LLM对输入表格的每一列进行类型标注。如果列类型标注器无法确定某一列的具体类型，它将输出“我不知道”。标注结果返回给聊天管理器并存储在聊天管理器的内存中（图[3](https://arxiv.org/html/2403.08291v2#S2.F3 "Figure 3 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中的②）。第三，Python程序员从聊天管理器接收历史消息，包括列类型标注结果（图[3](https://arxiv.org/html/2403.08291v2#S2.F3 "Figure 3 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中的③），选择相应的清洗函数并生成用于数据标准化过程的Python代码。生成的Python代码也返回给聊天管理器并存储在聊天管理器的内存中（图[3](https://arxiv.org/html/2403.08291v2#S2.F3 "Figure 3 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中的④）。最后，代码执行器接收来自聊天管理器的历史消息，包括列类型标注结果和生成的Python代码（图[3](https://arxiv.org/html/2403.08291v2#S2.F3 "Figure 3 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中的⑤），然后执行生成的Python代码。如果生成的代码执行没有错误，则返回标准化后的表格$T^{\prime}$。如果生成的代码执行时出现错误，则错误信息返回给聊天管理器并存储在其内存中（图[3](https://arxiv.org/html/2403.08291v2#S2.F3 "Figure 3 ‣ 2\. Type-Specific Standardization API Design ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")中的⑥）。然后，CleanAgent将重试整个工作流程，直到成功完成数据标准化过程。

## 4\. 演示场景

演示设置包括一个需要标准化的表格和一台笔记本电脑。笔记本电脑必须连接到互联网，以便访客可以顺利使用CleanAgent，并访问OpenAI的GPT服务。如果会议的互联网连接出现故障，也可以使用手机建立的移动热点来运行CleanAgent。

图[4](https://arxiv.org/html/2403.08291v2#S3.F4 "Figure 4 ‣ 3\. CleanAgent Workflow ‣ CleanAgent: Automating Data Standardization with LLM-based Agents")展示了CleanAgent的用户界面。如区域①所示，用户必须首先上传一个需要清理的CSV文件。然后，CleanAgent接收并显示上传文件的基本信息（行数和列数）。如果用户希望CleanAgent启动数据标准化过程，只需点击“开始标准化”按钮。

点击“开始标准化”按钮后，如区域②所示，User_Proxy生成三个详细步骤以完成数据标准化任务。首先，列类型注释器接收来自聊天管理器的消息，注释并输出每列的类型，如区域③所示。然后，Python程序员根据每列的类型，从Dataprep.Clean中选择标准化函数，并编写适当的Python代码以标准化输入表格中的列，如区域④所示。第三，代码执行器执行Python程序员生成的Python代码，并收集执行消息，如区域⑤所示。如果代码执行器在执行生成的Python代码时遇到错误消息，错误消息将发送到聊天管理器，并成为下一次尝试的提示的一部分。如果代码执行器收到成功执行的消息，CleanAgent将报告数据标准化已完成，如区域⑥所示。此外，用户可以点击“显示已清理表格”按钮，以检查标准化后的表格是否符合用户要求。如果符合，用户可以直接下载标准化后的表格。否则，用户可以用自然语言输入额外要求，CleanAgent将根据用户的输入启动新的数据标准化过程。

## 5\. 结论

在本文演示中，我们提出了CleanAgent，以通过Dataprep.Clean和基于LLM的代理自动化数据标准化过程。我们将CleanAgent实现为一个Web服务，用于可视化代理之间的对话。数据科学生命周期中的其他任务，如数据清理和数据可视化，也可以由基于LLM的代理完成（Xue等，[2023](https://arxiv.org/html/2403.08291v2#bib.bib6)）。

## 参考文献

+   (1)

+   Chen等（2023）Sibei Chen, Hanbing Liu, Weiting Jin, Xiangyu Sun, Xiaoyao Feng, Ju Fan, Xiaoyong Du, and Nan Tang. 2023. ChatPipe: Orchestrating Data Preparation Program by Optimizing Human-ChatGPT Interactions. *CoRR* abs/2304.03540 (2023). [https://doi.org/10.48550/ARXIV.2304.03540](https://doi.org/10.48550/ARXIV.2304.03540) arXiv:2304.03540

+   McKinney 等人（2024）Wes McKinney 等人 2024. pandas: 强大的 Python 数据分析工具包。 [https://pandas.pydata.org/](https://pandas.pydata.org/) 访问时间：2024-01-25。

+   Wu 等人（2023）Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, 和 Chi Wang. 2023. AutoGen: 通过多代理对话框架支持下一代 LLM 应用。 *CoRR* abs/2308.08155 (2023). [https://doi.org/10.48550/ARXIV.2308.08155](https://doi.org/10.48550/ARXIV.2308.08155) arXiv:2308.08155

+   Xi 等人（2023）Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, Rui Zheng, Xiaoran Fan, Xiao Wang, Limao Xiong, Yuhao Zhou, Weiran Wang, Changhao Jiang, Yicheng Zou, Xiangyang Liu, Zhangyue Yin, Shihan Dou, Rongxiang Weng, Wensen Cheng, Qi Zhang, Wenjuan Qin, Yongyan Zheng, Xipeng Qiu, Xuanjing Huan, 和 Tao Gui. 2023. 基于大语言模型代理的崛起与潜力：一项调查。 *CoRR* abs/2309.07864 (2023). [https://doi.org/10.48550/ARXIV.2309.07864](https://doi.org/10.48550/ARXIV.2309.07864) arXiv:2309.07864

+   Xue 等人（2023）Siqiao Xue, Caigao Jiang, Wenhui Shi, Fangyin Cheng, Keting Chen, Hongjun Yang, Zhiping Zhang, Jianshan He, Hongyang Zhang, Ganglin Wei 等人. 2023. Db-gpt: 通过私有大语言模型赋能数据库交互。 *arXiv 预印本 arXiv:2312.17449* (2023)。
