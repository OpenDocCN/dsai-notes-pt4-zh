<!--yml

类别：未分类

日期：2025-01-11 12:00:35

-->

# CRMArena：了解LLM智能体在现实环境中执行专业CRM任务的能力

> 来源：[https://arxiv.org/html/2411.02305/](https://arxiv.org/html/2411.02305/)

黄功翔   阿克莎拉·普拉巴卡尔   希达特·达万   毛怡鑫

王焕   西尔维奥·萨瓦雷塞   肖凯铭   菲利普·拉班   吴建生

Salesforce AI研究

{kh.huang, akshara.prabhakar, sidharth, y.mao,

huan.wang, ssavarese, cxiong, wu.jason}@salesforce.com

###### 摘要

客户关系管理（CRM）系统对现代企业至关重要，为管理客户互动和数据提供了基础。将AI智能体集成到CRM系统中可以自动化常规流程并增强个性化服务。然而，由于缺乏能够反映真实世界CRM任务复杂性的现实基准，部署和评估这些智能体面临挑战。为了解决这个问题，我们推出了CRMArena，这是一个新型的基准，旨在评估AI智能体在基于专业工作环境的现实任务中的表现。我们与CRM专家合作设计了九个客户服务任务，分布在三种角色之间：服务代理、分析师和经理。我们合成了一个大规模的模拟组织，填充了16个常用的工业对象（例如账户、订单、知识文章、案例），这些对象之间具有高度的互联性，并将其上传到一个真实的Salesforce CRM组织中。系统将通过UI和API访问CRM，尝试完成CRMArena中的任务。实验结果显示，即便使用ReAct提示，最先进的LLM智能体在不到40%的任务中成功完成，而即使提供手工制作的函数调用工具，也只有不到55%的任务成功完成。我们的研究结果突显了在真实工作环境中部署智能体时，在函数调用和规则遵循方面需要增强智能体的能力。CRMArena是向社区提出的一个公开挑战：能够可靠完成任务的系统展示了在流行工作环境中直接的商业价值。¹¹1我们的代码和基准已发布在[https://github.com/SalesforceAIResearch/CRMArena](https://github.com/SalesforceAIResearch/CRMArena)。

![[无标题图片]](img/710dbe9d6d6452ad36067f9d4a8dd503.png)

CRMArena：了解LLM智能体的能力

在现实环境中执行专业CRM任务

黄功翔   阿克莎拉·普拉巴卡尔   希达特·达万   毛怡鑫 王焕   西尔维奥·萨瓦雷塞   肖凯铭   菲利普·拉班   吴建生 Salesforce AI研究 {kh.huang, akshara.prabhakar, sidharth, y.mao, huan.wang, ssavarese, cxiong, wu.jason}@salesforce.com

## 1 引言

客户关系管理（CRM）系统在现代企业中至关重要，是管理与现有客户和潜在客户互动的核心工具 Winer ([2001](https://arxiv.org/html/2411.02305v1#bib.bib17))；Payne和Frow ([2005](https://arxiv.org/html/2411.02305v1#bib.bib12))。基于大型语言模型（LLM）的智能代理与CRM系统的结合，承诺能够自动化日常任务、提升运营效率并彻底改变客户体验。然而，在现实专业环境中评估LLM代理仍然是一项挑战，原因在于缺乏能够真实反映现实CRM环境中任务复杂性的强大基准测试，主要由于企业内的数据隐私问题。

之前关于评估LLM代理在工作任务中的表现的基准测试，如WorkArena Drouin et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib3))，Workbench Styles et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib16))，以及Tau Yao et al. ([2024](https://arxiv.org/html/2411.02305v1#bib.bib20))，往往侧重于基本功能，且在两个关键领域有所不足。首先，这些对象（例如，数据库中的表格）和对象之间的依赖关系（例如，外键）的复杂性通常过于简单，缺乏现实场景中的复杂性。其次，基准测试中包含的任务，如浏览网页和过滤列表，通常过于简单，并未代表现实工作任务。

为了应对这些局限性，我们提出了CRMArena，一个综合性的基准测试，旨在评估LLM代理在现实工作环境中执行真实CRM任务的能力。CRMArena提供了一个现实的沙盒环境，模仿了Salesforce的架构，并使用由LLM驱动的可扩展数据生成管道进行开发（见[图1](https://arxiv.org/html/2411.02305v1#S1.F1 "在1 引言 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")的左上角）。具体来说，该管道解决了两个关键挑战：（1）对象连接性：通过镜像Salesforce的Service Cloud架构²²2[https://architect.salesforce.com/diagrams/data-models/service-cloud/service-cloud-overview](https://architect.salesforce.com/diagrams/data-models/service-cloud/service-cloud-overview)，反映了数据对象之间的复杂关系（例如，账户与案件和订单的关联）。 （2）引入潜在变量，以更好地模拟现实数据动态，例如影响案件归档行为并建模偏离公司指南的情况。

此外，CRMArena 根据实际的客户服务角色定义任务。通过咨询经验丰富的 Salesforce CRM 专家，我们确定了九个代表 CRM 用例的任务（见 [第 2.1 节](https://arxiv.org/html/2411.02305v1#S2.SS1 "2.1 任务 ‣ 2 CRMArena ‣ CRMArena：了解 LLM 智能体在现实环境中执行专业 CRM 任务的能力")）。这些任务涵盖了三个角色：服务经理、服务代理和服务分析师。例如，服务经理关注智能体的表现和战略资源分配。[表 1](https://arxiv.org/html/2411.02305v1#S2.T1 "在 2.1 任务 ‣ 2 CRMArena ‣ CRMArena：了解 LLM 智能体在现实环境中执行专业 CRM 任务的能力") 比较了 CRMArena 与以往的数据集。

![参见标题](img/37291f416a0acdb6932efd297cfdf477.png)

图 1：本研究工作的贡献概述。我们首先根据 Salesforce Service Cloud 模式生成真实的 CRM 数据，通过严格的验证过程确保数据的质量和多样性。然后将这些验证过的数据存储在本地，并上传到 Salesforce 组织（Org）。与领域专家进行的专家研究验证了数据的真实性。以该组织作为沙盒环境，我们创建查询实例，并在不同的智能体框架中对不同的 LLM 进行基准测试。

CRMArena 无缝集成 Salesforce³³3[https://www.salesforce.com/crm/](https://www.salesforce.com/crm/)，通过用户界面和 API 访问实现交互（见 [图 1](https://arxiv.org/html/2411.02305v1#S1.F1 "1 引言 ‣ CRMArena：了解 LLM 智能体在现实环境中执行专业 CRM 任务的能力")）。此集成促成了一项与 CRM 专业人员的专家研究，用于评估我们合成组织的质量（见 [第 2.5 节](https://arxiv.org/html/2411.02305v1#S2.SS5 "2.5 专家研究 ‣ 2 CRMArena ‣ CRMArena：了解 LLM 智能体在现实环境中执行专业 CRM 任务的能力")）。研究结果表明，90% 的领域专家认为测试环境符合实际或更好，强调了基准测试与现实世界 CRM 场景的高度一致性。在验证了 CRMArena 的真实性后，我们通过 API 访问评估了不同的智能体系统。我们开发了两组工具：通用工具与任务特定工具，并将其与三种智能体框架和不同的 LLM 结合使用。研究发现，当使用通用工具时，所有 LLM 智能体都难以可靠地完成任务，表现最好的系统完成的任务不到 40%。加入手动设计的工具能够提升性能，最强的 LLM 智能体能解决最多 55% 的任务。然而，我们发现较弱的 LLM 由于其较差的函数调用能力，往往无法从手动设计的工具中受益。

总结来说，我们的主要贡献是：

+   •

    介绍 CRMArena，这是一个具有领域专家验证任务的现实 CRM 代理基准，用于评估 LLM 代理在真实世界商业场景中的表现。

+   •

    开发基于现实世界 CRM 模式的数据生成策略，结合潜在变量、去重和严格的数据验证，以确保数据的多样性和质量。

+   •

    通过实验展示，即使是最先进的 LLM 代理也无法可靠地完成 CRMArena 任务，强调了该基准的价值和挑战。

## 2 CRMArena

受到 CRM 角色（如服务经理、服务代理和服务分析师）常见任务的启发，CRMArena 包含九个任务，反映了现实世界中的 CRM 场景。这些任务经过领域专家验证，被认为是 CRM 中的常见任务。有关这些任务的概述，请参见 [图 2](https://arxiv.org/html/2411.02305v1#S2.F2 "在最佳区域识别 (BRI) ‣ 2.1 任务 ‣ 2 CRMArena ‣ CRMArena: 理解 LLM 代理在现实环境中执行专业 CRM 任务的能力")。以下是每个任务的详细说明。

### 2.1 任务

CRMArena 中的任务旨在准确反映现实世界 CRM 环境中遇到的各种挑战。它们涵盖了三种关键角色的职责：服务经理，专注于战略资源分配；服务代理，负责处理客户咨询；以及服务分析师，分析数据趋势和绩效指标以改善服务运营。

数据集 # 对象 # 依赖关系/对象 真实环境 现实工作任务 专家验证 工作台风格等 ([2024](https://arxiv.org/html/2411.02305v1#bib.bib16)) 5 0 ✗ ✗ ✗ Tau-Bench Yao 等人 ([2024](https://arxiv.org/html/2411.02305v1#bib.bib20)) 3 0.67 ✗ ✗ ✗ WorkArena Drouin 等人 ([2024](https://arxiv.org/html/2411.02305v1#bib.bib3)) 7 0.86 ✓ ✗ ✗ CRMArena（我们的方法） 16 1.31 ✓ ✓ ✓

表格 1：我们的基准与先前数据集的比较。CRMArena 是最复杂的基准，涉及最多的对象和对象依赖关系。此外，CRMArena 是唯一经过专家验证的基准，涵盖了真实世界环境和现实工作任务。

#### 新案件路由（NCR）

该任务的目标是将最合适的人工代理分配给一个即将到来的案件，旨在优化各种绩效指标。输入包括案件主题和描述，输出为推荐的人工代理的 ID。该任务评估 LLM 代理根据案件历史记录以及这些代理的技能和可用性，将案件分配给最合适的人工代理的能力。

#### 处理时间理解（HTU）

该任务涉及识别具有最短/最长平均处理时间的代理。根据案例历史数据，目标是确定处理前一个案件最快/最慢的人工代理。

#### 转移次数理解（TCU）

在这个任务中，LLM代理必须找出在给定的案例历史期间，哪个人工代理转交案件的次数最少或最多。HTU和TCU都评估LLM代理基于预定义指标准确分析性能的能力。

#### 命名实体消歧（NED）

LLM代理必须消除与客户交易相关的命名实体的歧义。在这里，我们重点处理消除产品名称的歧义。根据[图2](https://arxiv.org/html/2411.02305v1#S2.F2 "在最佳区域识别（BRI） ‣ 2.1 任务 ‣ 2 CRMArena ‣ CRMArena：了解LLM代理在现实环境中执行专业CRM任务的能力")中显示的查询，代理需要在给定时间范围内识别出与提到的客户购买的跑步鞋相对应的具体订单。这考察了对产品名称和客户订单历史的理解。

#### 政策违规识别（PVI）

在这个任务中，LLM代理会收到一个包含客户与代理互动的案例，并需要确定是否有公司政策被违反。这需要分析案例细节，并将其与知识文章中列出的政策规则进行对比，以识别是否有违规行为。

#### 知识问答（KQA）

这里的目标是让LLM代理基于知识文章回答特定问题。这项任务评估代理从CRM知识库中查找准确和相关信息的能力。

#### 顶级问题识别（TII）

这个任务要求LLM代理识别特定产品中报告最多的问题。在给定的案例历史中，代理需要确定哪个问题的出现频率最高。这个任务考察的是分析问题报告进行趋势分析的能力。

#### 月度趋势分析（MTA）

LLM代理必须确定在给定产品和时间段内，哪个月份的案件数量最多。通过分析在特定时间段内的案例历史，LLM代理能够识别出案件最多的月份，展示其识别趋势和模式的能力。

#### 最佳区域识别（BRI）

在这个任务中，LLM代理的目标是识别案件处理速度最快的区域。代理需要分析各区域的案件关闭时间，并指出表现最好的区域。

![参见说明文字](img/b14e204838d4d8f446b9fa3675c424e1.png)

图2：CRMArena中引入的九个独立任务概览。

### 2.2 沙盒环境

为CRMArena创建一个沙箱环境面临独特的挑战，特别是与隐私问题相关，并且需要在不使用真实企业数据的情况下提供逼真的数据。为此，我们开发了一种可扩展的数据生成管道，能够在多个领域生成多样且逼真的CRM数据。在我们的设置中，我们建模了16个业务对象。对象及其描述的完整列表可以在[附录D](https://arxiv.org/html/2411.02305v1#A4 "附录D 沙箱环境详情 ‣ CRMArena：了解LLM代理在真实环境中执行专业CRM任务的能力")中找到。构建这样的管道有两个主要挑战：对象连接性和隐性因果关系。在以下小节中，我们将说明如何解决这些挑战。

#### 对象连接性

真实世界的公司数据通常表现出对象之间复杂的相互连接，例如Case和Account对象。我们基于Salesforce的Service Cloud架构的数据生成方法，确保了高连接性。例如，Case对象与Account、Contact和User等对象相连接。[图7](https://arxiv.org/html/2411.02305v1#A4.F7 "在附录D 沙箱环境详情 ‣ CRMArena：了解LLM代理在真实环境中执行专业CRM任务的能力")展示了这些相互依赖关系，创造了有意义的数据环境。[表1](https://arxiv.org/html/2411.02305v1#S2.T1 "在2.1任务 ‣ 2 CRMArena ‣ CRMArena：了解LLM代理在真实环境中执行专业CRM任务的能力")突出了我们的基准在对象连接性方面远远超过现有工作。

#### 隐性因果关系

复制现实世界数据中隐含的因果关系是一项重大挑战。为了解决这个问题，我们引入了潜变量来模拟各种潜在因素，从而生成能够反映真实客户关系管理（CRM）数据库中微妙依赖关系和模式的数据。这些潜变量对于促进某些任务和生成所需场景至关重要。如[图3](https://arxiv.org/html/2411.02305v1#S2.F3 "在质量与多样性保障 ‣ 2.2 沙盒环境 ‣ 2 CRMArena ‣ CRMArena：理解LLM代理在真实环境中执行专业CRM任务的能力")中的示例所示，ShoppingHabit变量使我们能够更真实地模拟客户在不同时间段或假期季节的购买模式。类似地，User（代理）对象的Skill潜变量使我们的EmailMessage和LiveChatTranscript模拟能够包括代理无法解决问题并必须转交给其他代理的情况。如果没有这个潜变量，我们就无法为Transfer Count Understanding任务提供必要的场景。[图8](https://arxiv.org/html/2411.02305v1#A4.F8 "在附录D沙盒环境细节 ‣ CRMArena：理解LLM代理在真实环境中执行专业CRM任务的能力")展示了完整的数据生成流程。

#### 质量与多样性保障

我们以JSON格式生成数据，每个JSON对象代表一个对象的条目，以确保更高的可控性（Huang et al. [2024](https://arxiv.org/html/2411.02305v1#bib.bib5)；Laban et al. [2024](https://arxiv.org/html/2411.02305v1#bib.bib7)）。由于对象数量庞大（例如，500个产品条目与40多个Pricebook条目配对，导致超过20,000个PricebookEntry条目）以及LLM的最大输出令牌有限，直接提示LLM生成所有对象的条目不可行。为了解决这个问题，我们采用了批次小提示方法，批次大小为10。然而，这种方法可能导致批次之间的内容重复或高度相似。为缓解这一问题，我们实施了两阶段去重策略。首先，对于所有对象，我们在迷你批次提示过程中将所有先前生成的条目包含在提示中，并指示LLM不要生成相同的内容。数据生成后，我们通过字符串精确匹配去除对某些任务至关重要的字段和对象的重复条目（例如，User的电子邮件）。

此外，我们对数据进行了严格的质量保证过程，涉及双层验证。格式验证器通过检查生成的小批次中每个条目是否包含对象所需的所有字段，确保所有数据条目符合预定义的架构。未通过验证的小批次将被丢弃并重新生成。内容验证器则检查任务的可行性，重点关注对特定任务至关重要的对象。例如，在命名实体歧义消解任务中，我们验证（1）改写后的模糊产品名称与其原始名称的偏差不应过大，且（2）与客户购买的其他产品的相似度不应过高。在这种情况下，内容验证器向LLM提供客户已购买的产品列表和改写后的产品名称。如果LLM正确识别出产品，我们会保留该条目；否则，该条目将被丢弃。⁴⁴4我们使用gpt-4o作为LLM进行数据合成和内容验证，因为其具有成本效益。

![参见说明](img/b90cbf66118c8adaf7af0e3bf0db8062.png)

(a)

![参见说明](img/7c9be7e9c5903325638ca557b92fe798.png)

(b)

图3：潜在变量（灰色）影响对象（蓝色）生成的示例。（a）购物习惯变量影响客户何时购买以及购买何种类型的产品。（b）技能变量决定代理是否能够处理案件或需要转交案件。

#### 上传到 Org

一旦数据生成，我们将其填充到一个干净的Simple Demo Org (SDO)⁵⁵5[https://partners.salesforce.com/s/education/general/Salesforce_Orgs](https://partners.salesforce.com/s/education/general/Salesforce_Orgs)中，这个Org位于Salesforce上，并且不包含潜在变量。此排除操作有两个目的：一是它模拟了典型的公司无法访问此类信息的场景，从而提供了一个更具现实性的测试环境；二是与在生成的数据库上进行测试相比，它增加了额外的挑战。此外，利用Salesforce的SDO作为沙箱环境消除了本地环境设置的必要性和复杂性，这在许多相关工作中是必需的，参见Styles等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib16)）；Drouin等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib3)）；Yao等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib20)）；Zhou等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib24)）。这种方法不仅促进了测试的进行，还鼓励了对我们基准测试的科学严谨性和未来研究。沙箱环境的详细信息可以在[附录D](https://arxiv.org/html/2411.02305v1#A4 "附录D 沙箱环境详细信息 ‣ CRMArena: 理解LLM代理在真实环境中执行专业CRM任务的能力")中找到。

#### 环境规范

我们的数据生成管道的输入包括公司名称、公司描述、数据库架构以及对象的规模（例如，案例和产品的数量）。我们选择为一家虚构的鞋业公司创建一个组织，因为鞋类行业通常具有多样的产品种类和客户服务场景。我们生成的数据规模旨在反映一家中型企业，涵盖了数千个订单和数百个产品与支持案例，时间跨度为4年。每个对象的总条目数见[附录D](https://arxiv.org/html/2411.02305v1#A4 "附录D 沙盒环境详情 ‣ CRMArena：了解LLM代理在真实环境中执行专业CRM任务的能力")。

#### 可扩展性

我们的数据生成管道旨在提供灵活性，并且可以通过更改用户指定的输入参数轻松适应其他行业。例如，通过在公司描述中指定行业，我们的管道可以自动生成适用于特定行业的现实CRM数据，例如金融行业。此外，为了适应客户服务以外的其他使用场景，例如销售，用户只需要提供相应的架构（例如，用于销售的Salesforce Sales Cloud架构）。这种灵活性确保了管道可以扩展以满足广泛的业务需求和LLM代理基准测试的目的。

请注意，我们当前的设置反映了CRM场景的简化版本，其中每个案例都与一个问题和一个产品相关联。这种简化有助于管理像“主要问题识别”这样复杂的任务，否则这需要LLM代理逐个分析每个案例，这对于当前状态下的LLM来说是不可行的。随着LLM能力的提升，我们的基准测试可以通过去除这些依赖来创建更复杂的设置。

### 2.3 查询实例生成

在创建沙盒环境之后，我们生成自然语言查询实例及其真实答案，以便对我们的任务进行基准测试。对于知识问答任务，查询可以通过简单地提示LLM每篇知识文章来生成问题和答案对（Laban等，[2022](https://arxiv.org/html/2411.02305v1#bib.bib8); Huang等，[2024](https://arxiv.org/html/2411.02305v1#bib.bib5)）。对于其余的任务，我们通过四个步骤构建查询实例：（1）种子查询构建，（2）真实答案计算，（3）ID映射，以及（4）查询重述。

我们手动创建了14个种子查询，每个查询中包含相应变量的占位符，如时间段或产品名称。这有助于开发可以利用仅在生成的数据库中可见的潜在变量来计算真实答案的函数。例如，代理在实时聊天中违反政策的行为只有在生成的数据库中才能验证。获取答案后，我们将生成的数据库中的ID映射到Salesforce Org中的对应ID，从而在沙箱环境中为我们的查询建立真实答案。最后，为了确保测试查询的多样性，我们使用LLM对种子查询进行改写，从而增强我们的基准测试过程的稳健性和多样性。这个过程的示例如[图1](https://arxiv.org/html/2411.02305v1#S1.F1 "在1介绍 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")的右上角所示。

此外，为了模拟现实世界中一些问题可能无法回答的场景，我们构建了不可回答的查询。受到Brahman等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib1)）提出的不可回答问题模式的启发，我们重点关注在CRM环境中最相关的虚假前提查询。例如，一个查询可能要求识别在某一特定时期内转移最多案件的代理，尽管在该时期内并没有代理转移案件。我们在五个任务中包含了不可回答的查询：转移时间理解、处理时间理解、主要问题识别、命名实体消歧义和政策违规识别。对于这些情况，我们期望模型输出“None”。总的来说，不可回答的查询占每个任务总查询的30%。总体上，我们每个任务生成130个查询实例，总计生成1,170个查询用于CRMArena。详细信息和种子查询请参见[附录B](https://arxiv.org/html/2411.02305v1#A2 "附录B 查询生成细节 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")。

### 2.4 工具：API和函数

Salesforce Orgs自然支持多种通用API，如Apex API、REST API和Tooling API，这些API旨在覆盖Salesforce生态系统中的广泛功能。对于我们的任务范围及其与Python环境的集成，我们选择使用SOQL和SOSL查询⁶⁶6[https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql_sosl_intro.htm](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql_sosl_intro.htm)。SOQL查询用于获取特定子集的对象，通过精确匹配或过滤条件，通常格式为"SELECT Id ..."，而SOSL查询则支持在如知识文章和产品名称等对象中进行模糊搜索，格式为"FIND ..."。这两种类型的查询理论上可以支持广泛的查询实例，消除了为函数调用手动设计动作的必要性。

除了通用API外，我们还开发了任务特定工具，采用Python包装函数的形式，以便于评估函数调用代理。这些函数通过提供结构化和逻辑化的操作，直接映射到典型的CRM任务，从而优化任务性能。我们在SOQL和SOSL之上手动定义了27个这样的Python包装函数（完整列表见[附录 C](https://arxiv.org/html/2411.02305v1#A3 "附录 C 动作空间详情 ‣ CRMArena: 理解LLM代理在现实环境中执行专业CRM任务的能力")），以简化函数调用，并专注于每个任务所需的关键组件。这些任务特定函数旨在最大化跨任务的可重用性，减少任务特定定制的需求。

![请参阅标题](img/55183801e75f3102d7419d02797f710c.png)

图 4：专家研究结果。图表展示了领域专家对整体Org结构（上图）和我们生成的单个对象（下图）的真实性评分。

### 2.5 专家研究

为了确保我们开发的沙箱环境的真实性和实用性，我们进行了一个用户研究，参与者包括十位具有多样化专业背景的专家，他们每天都在使用Salesforce Orgs。这些专家通过User Interviews平台⁷⁷7[https://www.userinterviews.com/](https://www.userinterviews.com/)招募。专家研究的详细信息可以在[附录 F](https://arxiv.org/html/2411.02305v1#A6 "附录 F 专家研究详情 ‣ CRMArena: 理解LLM代理在现实环境中执行专业CRM任务的能力")中找到。

每次专家研究的会话都分为三个部分。首先，我们向专家介绍了我们的沙箱环境，重点介绍了关键对象，如“案件”和“联系人”，并通过相关的URL为他们提供访问权限。这一初步的介绍旨在让专家熟悉该组织。其次，我们为他们分配了五个来自CRMArena的查询实例，每个实例代表一个不同的任务，要求他们完成。这个任务完成阶段的目的是评估沙箱在执行现实世界CRM任务时的实用性和操作一致性。最后，专家们评估了我们的组织环境与他们习惯的现实世界系统相比的真实感。他们还提供了详细的评分依据，深入探讨了我们的环境与实际CRM场景的一致性。

我们专家研究的结果展示在[图4](https://arxiv.org/html/2411.02305v1#S2.F4 "在2.4工具：API和函数 ‣ 2 CRMArena ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中。研究结果非常鼓舞人心：90%的专家将我们构建的组织评为“现实”或“非常现实”。这一积极评价也延伸到了组织中的个别对象，超过77%的专家给予这些对象类似的高评分，认为其具有高度现实感。这些结果强烈表明，我们的沙箱环境与现实世界的CRM系统高度相似。我们在[表14](https://arxiv.org/html/2411.02305v1#A6.T14 "在F.3定性反馈 ‣ 附录F专家研究详情 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中提供了专家访谈的定性反馈和评价依据。

## 3 基准测试实验

| 模型 | NCR | HTU | TCU | NED | PVI | KQA | TII | MTA | BRI | ALL |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Act |
| gpt-4o | 43.1 | 10.0 | 17.7 | 30.8 | 28.5 | 29.3 | 68.5 | 29.2 | 7.7 | 29.4 |
| gpt-4o-mini | 0.8 | 38.5 | 23.8 | 9.2 | 0.0 | 43.1 | 26.9 | 3.8 | 3.8 | 16.7 |
| claude-3.5-sonnet | 78.5 | 24.6 | 15.4 | 51.5 | 28.5 | 44.7 | 45.4 | 20.8 | 26.9 | 37.4 |
| claude-3-sonnet | 9.2 | 26.9 | 24.6 | 30.8 | 23.8 | 16.6 | 16.2 | 1.5 | 0.0 | 16.6 |
| llama3.1-405b | 46.2 | 17.7 | 17.7 | 13.9 | 30.0 | 47.0 | 15.4 | 5.4 | 6.9 | 22.2 |
| llama3.1-70b | 28.5 | 20.0 | 24.6 | 6.2 | 30.0 | 47.9 | 8.5 | 0.0 | 1.5 | 18.6 |
| ReAct |
| gpt-4o | 70.0 | 39.2 | 22.3 | 30.8 | 35.4 | 50.2 | 64.6 | 20.9 | 10.8 | 38.2 |
| gpt-4o-mini | 40.8 | 36.9 | 25.4 | 31.5 | 24.6 | 52.8 | 30.0 | 6.2 | 6.2 | 28.3 |
| claude-3.5-sonnet | 62.9 | 20.0 | 11.5 | 52.3 | 30.0 | 45.0 | 43.9 | 20.8 | 21.5 | 34.3 |
| claude-3-sonnet | 7.7 | 24.6 | 26.9 | 29.2 | 28.5 | 16.0 | 22.3 | 0.8 | 0.0 | 17.3 |
| llama3.1-405b | 81.5 | 22.3 | 15.4 | 33.9 | 34.6 | 55.3 | 34.6 | 13.9 | 13.1 | 33.8 |
| llama3.1-70b | 48.5 | 20.0 | 13.9 | 33.1 | 37.7 | 48.7 | 23.9 | 13.9 | 10.8 | 27.8 |
| Function Calling |
| gpt-4o | 60.0 | 47.7 | 81.5 | 46.2 | 39.2 | 30.4 | 97.7 | 27.7 | 59.2 | 54.4 |
| gpt-4o-mini | 0.8 | 10.8 | 10.8 | 17.7 | 13.8 | 39.7 | 60.0 | 0.0 | 21.5 | 19.5 |
| claude-3.5-sonnet | 4.6 | 33.1 | 82.3 | 52.3 | 30.0 | 40.5 | 69.2 | 26.9 | 36.9 | 41.8 |
| claude-3-sonnet | 0.8 | 1.5 | 30.0 | 25.4 | 41.5 | 23.2 | 12.3 | 1.5 | 0.0 | 15.1 |
| llama3.1-405b (提示) | 16.2 | 31.5 | 64.6 | 50.0 | 26.9 | 47.6 | 95.4 | 86.9 | 42.3 | 51.3 |
| llama3.1-70b (提示) | 1.5 | 23.1 | 44.6 | 53.8 | 37.4 | 42.4 | 93.8 | 43.8 | 29.2 | 41.1 |

表格 2：在不同代理框架下，各种LLM在CRMArena上的整体表现（%）。评估指标为知识问答（KQA）任务的F1得分，其他任务的精确匹配。ALL表示所有任务的平均表现。

### 3.1 实验设置

#### 模型

我们评估了最先进的专有和开源LLM，包括gpt模型（gpt-4o和gpt-3.5-turbo）；claude模型（claude-3.5-sonnet和claude-3-sonnet），以及llama模型（llama-3.1-405b和llama-3.1-70B）。⁸⁸8我们观察到原生的函数调用模式表现不佳，因此报告了Llama 3.1模型的提示模式表现。使用这些模型，我们测试了三种常见的代理框架：Act、ReAct（Yao等人，([2023](https://arxiv.org/html/2411.02305v1#bib.bib21))）和函数调用（FC）。ReAct是一种基于提示的方法，每个步骤都包括思考和行动过程，而Act则是去除思考成分后的ReAct。有关这些设置的详细信息，请参见以下段落和[附录G](https://arxiv.org/html/2411.02305v1#A7 "附录G 实施细节 ‣ CRMArena：了解LLM代理在现实环境中执行专业CRM任务的能力")。

#### 行动空间

每个任务都可以表述为一个部分可观测的马尔可夫决策过程（POMDP）$(\mathcal{U},\mathcal{S},\mathcal{A},\mathcal{O},\mathcal{T},\mathcal{R})$，其中包括指令空间$\mathcal{U}$、状态空间$\mathcal{S}$、动作空间$\mathcal{A}$、观察空间$\mathcal{O}$、转移函数$\mathcal{T}:\mathcal{S}\times\mathcal{A}\to\mathcal{S}$和奖励函数$\mathcal{R}:\mathcal{S}\times\mathcal{A}\to\{0,1\}$。在Act和ReAct设置中，动作空间是丰富的，即$\mathcal{A}=\{\texttt{execute <query>},\texttt{submit <result>}\}$。给定一个用户查询$u\in\mathcal{U}$，代理可以执行<query>来发出SOQL或SOSL查询，与实例进行交互，接收执行查询后的观察结果$o_{t}\in\mathcal{O}$。这一过程会一直持续，直到代理选择提交并接收一个二值奖励$r=\mathcal{R}(s_{T},\texttt{submit})\in\{0,1\}$。在Function Calling设置中，代理通过作为Python函数实现的API工具与环境进行交互。在这种情况下，代理不会直接暴露于Salesforce环境中，且对象依赖关系保持隐藏。在内部，API与环境的交互由我们定义的控制方式进行。一个动作$a$的形式为tool_call{**kwargs}。这些三种设置的系统提示详见[附录 E](https://arxiv.org/html/2411.02305v1#A5 "Appendix E Prompts ‣ CRMArena: Understanding the Capacity of LLM Agents to Perform Professional CRM Tasks in Realistic Environments")。

#### 观察空间

动作通过Simple Salesforce包在沙盒环境中执行⁹⁹9[https://github.com/simple-salesforce/simple-salesforce](https://github.com/simple-salesforce/simple-salesforce)。如果动作成功，环境将返回在CRM系统中查询的数据；否则，将返回错误信息，例如函数调用参数不正确。

#### 评估指标

对于知识问答任务，由于它是一个开放式文本生成任务，我们使用F1分数。对于其余任务，我们只需要比较预测的和真实的对象ID；因此，使用精确匹配。

### 3.2 结果

主要结果总结见[表2](https://arxiv.org/html/2411.02305v1#S3.T2 "在 3 基准实验 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")。我们做出了以下观察。首先，现实世界的CRM任务对顶级LLM代理仍然具有挑战性。在ReAct框架下，最佳模型（gpt-4o）的整体得分仅为38.2%。即使配备了人工设计的函数，整体表现仍然只有54.4%。这些发现突显了我们CRMArena的挑战。其次，强大和较弱的LLM在不同的代理框架下表现出相反的趋势。特别是，像gpt-4o和claude-3.5-sonnet这样的模型在FC设置中得分更高，而它们较弱的对手在配备函数调用能力时表现更差。这表明，人工定义的函数可能并不总是能帮助LLM代理，因为较弱的模型可能无法正确使用这些函数，从而导致性能下降。最后，开源模型正在追赶专有的LLM。在三种设置中，我们看到llama模型的得分与gpt和claude模型相似，有时甚至更高。这表明开源和封闭源模型之间的差距正在缩小。从[图6](https://arxiv.org/html/2411.02305v1#A4.F6 "在附录D 沙盒环境详情 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中，我们观察到llama模型在基于执行反馈的错误恢复范围上往往高于封闭源模型。

模型 # 完成标记 # 回合 数量 成本（$） ReAct gpt-4o 48568.73 5.4 0.182 claude-3.5-sonnet 70814.75 6.9 0.228 llama3.1-405b 35647.29 7.3 0.125 FC gpt-4o 78305.38 6.8 0.305 claude-3.5-sonnet 105248.43 8.1 0.371

表3：顶级代理在查询和任务中的平均成本。

### 3.3 讨论

#### 什么是最具成本效益的解决方案？

在三分之二的代理框架中，gpt-4o表现最佳。gpt-4o的高效性也体现在[表3](https://arxiv.org/html/2411.02305v1#S3.T3 "在 3.2 结果 ‣ 3 基准实验 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中，表明gpt-4o每次实例的成本最低，并且完成查询所需的回合数最少。因此，最具成本效益的解决方案是在函数调用设置下使用gpt-4o。

#### 函数类型如何影响模型性能？

在[表2](https://arxiv.org/html/2411.02305v1#S3.T2 "在 3 个基准实验中 ‣ CRMArena：理解 LLM 代理在现实环境中执行专业 CRM 任务的能力")中，我们观察到，为LLM代理配备功能调用能力并不一定会导致性能提升。为了更好地理解这一现象，我们根据两个维度对功能进行了分类：功能性和功能依赖性。功能性是指该功能是否仅通过SOSL或SOQL（查询）查询CRM系统，或者它是否包含其他操作，如数学计算或聚合（计算）。另一方面，功能依赖性将功能分为依赖于其他功能输出的（依赖）和独立的（独立）。这一点至关重要，因为我们的基准要求LLM代理执行一系列调用，每个调用都依赖于前一个调用的输出（Qin 等人，[2023](https://arxiv.org/html/2411.02305v1#bib.bib13)；Lu 等人，[2024](https://arxiv.org/html/2411.02305v1#bib.bib10)）。[表15](https://arxiv.org/html/2411.02305v1#A7.T15 "在附录G 实现细节中 ‣ CRMArena：理解 LLM 代理在现实环境中执行专业 CRM 任务的能力")列出了我们测试的功能和任务列表。

我们从每个类别中抽取了四对功能-任务对，以评估在特定功能从工具集中移除时，gpt-4o、gpt-4o-mini和claude-3-sonnet的表现，并用两个通用功能execute_soql和execute_sosl来执行任意查询。总结的结果在[表4](https://arxiv.org/html/2411.02305v1#S3.T4 "功能类型如何影响模型性能？ ‣ 3.3 讨论 ‣ 3 个基准实验 ‣ CRMArena：理解 LLM 代理在现实环境中执行专业 CRM 任务的能力")中指出，尽管所有功能类型都提高了gpt-4o的性能，但它们对gpt-4o-mini或claude-3-sonnet的影响不同。这表明，较强的模型更擅长有效利用人工设计的功能，而较弱的模型可能会遇到困难。有趣的是，假设有助于数学运算较弱的LLM的计算功能，实际上可能会由于其有限的功能调用能力而降低较弱模型的表现。

功能依赖性 gpt-4o gpt-4o-mini claude-3-sonnet 查询独立 -6.6 -6.9 2.3 查询依赖 -2.9 3.0 7.5 计算独立 -9.4 4.6 -3.3 计算依赖 -26.7 4.0 3.3

表4：移除每类功能后的性能差异（%）。较低的数字表示这些功能对LLM代理更有用。

![参考说明](img/6895e7fd173fc4b275e044e59882d68a.png)

图5：使用不同代理框架的gpt-4o的一致性。

#### 这些代理在多次试验中的一致性如何？

对于LLM代理，一致性非常重要，尤其是在工作环境中部署时。我们通过多次提示实验评估LLM代理的一致性。在这里，我们采用了Yao等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib20)）提出的pass^k度量。pass^k计算的是所有k个独立且分布相同的任务尝试成功的概率，并对所有任务取平均值。除了KQA外，我们在CRMArena的所有任务上进行了十次试验，因为KQA的奖励不是二元的。结果见[图5](https://arxiv.org/html/2411.02305v1#S3.F5 "函数类型如何影响模型性能？ ‣ 3.3 讨论 ‣ 3 基准测试实验 ‣ CRMArena：在真实环境中理解LLM代理执行专业CRM任务的能力")，我们发现令人惊讶的是，所有我们测试的三个代理框架的pass^k随着k的增加几乎以相同的速度下降。这表明这三个框架的一致性相似，而且表现最好的LLM不能可靠地通过我们评估的任何一个代理框架来解决任务。

## 4 相关工作

#### 代理基准测试

为了评估基于LLM的代理，已经开发了多个基准测试，Yao等人（[2022](https://arxiv.org/html/2411.02305v1#bib.bib19)）；Liu等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib9)）；Jimenez等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib6)）。最近，主要的研究工作专注于网络代理，挑战LLM在网站上导航并执行操作。这些网站通常涉及日常场景，如电子商务和社交讨论形式，Deng等人（[2023](https://arxiv.org/html/2411.02305v1#bib.bib2)）；He等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib4)）；Zhou等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib24)）；Lù等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib11)）；Yoran等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib22)）。另一项研究工作集中在评估部署代理的安全性，Ruan等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib14)）；Yuan等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib23)）。

#### 面向工作的数据集

一些研究已经开发了专门用于工作任务的数据集。CRM基准Salesforce（[2024](https://arxiv.org/html/2411.02305v1#bib.bib15)）旨在评估大语言模型（LLMs）在商业应用中的文本生成和摘要能力。WorkBench Styles 等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib16)）包含五个数据库，旨在评估LLM代理在简单工作任务中的表现，如发送电子邮件、创建日历邀请和计算网站流量来源。$\tau$-Bench Yao 等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib20)）创建了需要与用户互动以获取相关信息和授权的任务，这通过使用LLM模拟用户来实现。WorkArena Drouin 等人（[2024](https://arxiv.org/html/2411.02305v1#bib.bib3)）构建了一个基于Web的工作环境，允许测试具有视觉能力的代理。

## 5 结论

本工作介绍了CRMArena，这是一种新颖的基准，用于评估LLM代理在专业工作环境中执行现实CRM任务的能力。通过结合专家验证的任务和模拟CRM系统中典型复杂数据关系，CRMArena为LLM代理提供了一个全面且现实的挑战。我们的实验表明，即便是最先进的LLM，也在这些现实任务中遇到困难，即使具备了调用功能的能力，成功率仍然有限。这些发现突显了当前LLM能力与现实世界CRM场景需求之间的差距。CRMArena为在现实工作环境中更精细化评估LLM代理迈出了基础性的一步。

## 6 伦理考虑

本工作介绍了一种评估LLM代理在客户关系管理（CRM）系统中表现的基准。虽然使用的数据是合成生成的，但它是基于真实世界的CRM数据结构和任务模型。因此，考虑此项工作的伦理影响，特别是在数据偏见和隐私问题方面，尤为重要。

#### 数据偏见

尽管数据是合成的，但这些数据是通过在真实世界数据上训练的模型生成的，而真实数据中可能包含固有的偏见。这些偏见与客户人口统计、购买行为或案件解决方式有关，可能会在生成的数据中不经意地反映出来，进而可能延续刻板印象或歧视性做法。幸运的是，在对生成数据进行彻底的人工检查以识别潜在偏见之后，我们没有观察到这种模式。

#### 隐私问题

虽然我们的基准测试没有使用任何真实的客户数据，因此无法访问个人信息，但CRM数据本身的结构和性质可能会引发隐私问题。我们基准中的任务涉及访问敏感的客户信息，尽管是合成数据。为了确保即使是合成数据也能负责任地处理，我们进行了彻底的人工检查，以验证不存在任何个人可识别信息，并确认这些数据无法用来推断有关个人的私人信息。这个细致的审核过程加强了我们对道德数据处理的承诺，并减少了潜在的隐私风险。

## 7 限制

CRMArena包含九项任务，全面评估了LLM代理在现实环境中执行通常与三种主要角色相关的职责的能力：服务经理、服务代理和服务分析师。然而，本研究未涵盖CRM中其他常见角色，如销售代表。我们计划在未来的研究中纳入这些额外的角色。

## 参考文献

+   Brahman 等（2024）Faeze Brahman, Sachin Kumar, Vidhisha Balachandran, Pradeep Dasigi, Valentina Pyatkin, Abhilasha Ravichander, Sarah Wiegreffe, Nouha Dziri, Khyathi Chandu, Jack Hessel 等. 2024. 说“不”的艺术：语言模型中的情境性不合规。 *arXiv 预印本 arXiv:2407.12043*。

+   Deng 等（2023）Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samuel Stevens, Boshi Wang, Huan Sun, 和 Yu Su. 2023. [Mind2web: 面向通用代理的网络应用](https://openreview.net/forum?id=kiYqbO3wqw)。载于 *第37届神经信息处理系统会议 数据集和基准轨道*。

+   Drouin 等（2024）Alexandre Drouin, Maxime Gasse, Massimo Caccia, Issam H. Laradji, Manuel Del Verme, Tom Marty, David Vazquez, Nicolas Chapados, 和 Alexandre Lacoste. 2024. [Workarena: Web 代理在解决常见知识工作任务方面有多强大？](https://openreview.net/forum?id=BRfqYrikdo) 载于 *第41届国际机器学习会议*。

+   He 等（2024）Hongliang He, Wenlin Yao, Kaixin Ma, Wenhao Yu, Yong Dai, Hongming Zhang, Zhenzhong Lan, 和 Dong Yu. 2024. [WebVoyager: 使用大型多模态模型构建端到端的网络代理](https://doi.org/10.18653/v1/2024.acl-long.371)。载于 *第62届计算语言学协会年会（第1卷：长篇论文）会议论文集*，第6864–6890页，泰国曼谷。计算语言学协会。

+   Huang 等 (2024) Kung-Hsiang Huang, Philippe Laban, Alexander Fabbri, Prafulla Kumar Choubey, Shafiq Joty, Caiming Xiong, 和 Chien-Sheng Wu. 2024. [拥抱分歧以获得更丰富的见解：一个多文档摘要基准及其在新闻文章中总结多样化信息的案例研究](https://doi.org/10.18653/v1/2024.naacl-long.32). 发表在 *2024年北美计算语言学协会会议：人类语言技术（卷1：长篇论文）*，第570-593页，墨西哥城，墨西哥。计算语言学会。

+   Jimenez 等 (2024) Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, 和 Karthik R Narasimhan. 2024. [SWE-bench: 语言模型能解决现实世界的 GitHub 问题吗？](https://openreview.net/forum?id=VTF8yNQM66) 在 *第十二届国际学习表征会议* 上。

+   Laban 等 (2024) Philippe Laban, Alexander R Fabbri, Caiming Xiong, 和 Chien-Sheng Wu. 2024. 一个 haystack 的总结：对长上下文 LLMs 和 RAG 系统的挑战。发表于 *2024年自然语言处理经验方法会议论文集*。计算语言学会。

+   Laban 等 (2022) Philippe Laban, Chien-Sheng Wu, Lidiya Murakhovs’ka, Xiang’Anthony’ Chen, 和 Caiming Xiong. 2022. Discord 问题：新闻报道多样性分析的计算方法。*arXiv 预印本 arXiv:2211.05007*。

+   Liu 等 (2024) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, 和 Jie Tang. 2024. [Agentbench: 评估 LLMs 作为代理](https://openreview.net/forum?id=zAdUB0aCTQ). 在 *第十二届国际学习表征会议* 上。

+   Lu 等 (2024) Jiarui Lu, Thomas Holleis, Yizhe Zhang, Bernhard Aumayer, Feng Nan, Felix Bai, Shuang Ma, Shen Ma, Mengyu Li, Guoli Yin, Zirui Wang, 和 Ruoming Pang. 2024. [Toolsandbox: 一种有状态的、对话式的、交互式评估基准，用于 LLM 工具使用能力](https://arxiv.org/abs/2408.04682). *预印本*，arXiv:2408.04682。

+   Lù 等 (2024) Xing Han Lù, Zdeněk Kasner, 和 Siva Reddy. 2024. Weblinx: 具有多轮对话的现实世界网站导航。*arXiv 预印本 arXiv:2402.05930*。

+   Payne 和 Frow (2005) Adrian Payne 和 Pennie Frow. 2005. 客户关系管理的战略框架。*市场营销杂志*，69(4):167–176。

+   Qin 等 (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, Sihan Zhao, Lauren Hong, Runchu Tian, Ruobing Xie, Jie Zhou, Mark Gerstein, Dahai Li, Zhiyuan Liu, 和 Maosong Sun. 2023. [Toolllm: 帮助大型语言模型掌握超过 16000 种现实世界的 API](https://arxiv.org/abs/2307.16789). *预印本*，arXiv:2307.16789。

+   Ruan 等人（2024）Yangjun Ruan, Honghua Dong, Andrew Wang, Silviu Pitis, Yongchao Zhou, Jimmy Ba, Yann Dubois, Chris J Maddison 和 Tatsunori Hashimoto. 2024. 通过 LM 模拟沙盒识别 LM 代理的风险。发表于 *第十二届国际学习表征会议*。

+   Salesforce（2024）Salesforce. 2024. [Salesforce 宣布全球首个针对 CRM 的 LLM 基准测试](https://www.salesforce.com/uk/news/stories/crm-benchmark/)。

+   Styles 等人（2024）Olly Styles, Sam Miller, Patricio Cerda-Mardini, Tanaya Guha, Victor Sanchez 和 Bertie Vidgen. 2024. [Workbench: 一个用于代理在现实工作环境中应用的基准数据集](https://openreview.net/forum?id=4HNAwZFDcH)。发表于 *第一次语言建模会议*。

+   Winer（2001）Russell S Winer. 2001. 客户关系管理框架。*加州管理评论*，43(4):89–105.

+   Yang 等人（2024）John Yang, Akshara Prabhakar, Karthik Narasimhan 和 Shunyu Yao. 2024. Intercode: 标准化和基准测试带执行反馈的交互式编码。*神经信息处理系统进展*，第 36 卷。

+   Yao 等人（2022）Shunyu Yao, Howard Chen, John Yang 和 Karthik Narasimhan. 2022. [Webshop: 朝着具有基础语言代理的可扩展真实世界网页交互迈进](https://proceedings.neurips.cc/paper_files/paper/2022/file/82ad13ec01f9fe44c01cb91814fd7b8c-Paper-Conference.pdf)。发表于 *神经信息处理系统进展*，第 35 卷，第 20744–20757 页，Curran Associates, Inc.

+   Yao 等人（2024）Shunyu Yao, Noah Shinn, Pedram Razavi 和 Karthik Narasimhan. 2024. Tau-bench: 用于现实领域工具-代理-用户交互的基准测试。*arXiv 预印本 arXiv:2406.12045*.

+   Yao 等人（2023）Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan 和 Yuan Cao. 2023. [React: 在语言模型中协同推理与行动](https://openreview.net/forum?id=WE_vluYUL-X)。发表于 *第十一届国际学习表征会议*。

+   Yoran 等人（2024）Ori Yoran, Samuel Joseph Amouyal, Chaitanya Malaviya, Ben Bogin, Ofir Press 和 Jonathan Berant. 2024. [Assistantbench: 网络代理能解决现实且耗时的任务吗？](https://arxiv.org/abs/2407.15711) *预印本*，arXiv:2407.15711.

+   Yuan 等人（2024）Tongxin Yuan, Zhiwei He, Lingzhong Dong, Yiming Wang, Ruijie Zhao, Tian Xia, Lizhen Xu, Binglin Zhou, Fangqi Li, Zhuosheng Zhang 等人. 2024. R-judge: 针对 LLM 代理的安全风险意识基准测试。*arXiv 预印本 arXiv:2401.10019*.

+   Zhou 等人（2024）Shuyan Zhou, Frank F. Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, Uri Alon 和 Graham Neubig. 2024. [Webarena: 用于构建自主代理的真实网页环境](https://openreview.net/forum?id=oKn9c6ytLx)。发表于 *第十二届国际学习表征会议*。

## 附录 A 进一步讨论

#### 奖励与回合数的关系

在[图 6](https://arxiv.org/html/2411.02305v1#A4.F6 "附录 D 沙盒环境详情 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力")中，我们展示了代理成功完成用户查询所需的回合数分布。

#### 无法回答的查询分析

在[表 5](https://arxiv.org/html/2411.02305v1#A3.T5 "附录 C 动作空间详情 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力")中，我们展示了每个 LLM 代理的性能。总体而言，与标准查询相比，LLM 代理在处理此类查询时表现优秀。有趣的是，在[表 2](https://arxiv.org/html/2411.02305v1#S3.T2 "3 基准实验 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力")中观察到的趋势，在本实验中也得到了体现：函数调用只对更强的 LLM 有益，而较弱的 LLM，如 claude-3-sonnet 和 gpt-4o，在启用函数调用功能时表现较差。

## 附录 B 查询生成详情

[表 6](https://arxiv.org/html/2411.02305v1#A3.T6 "附录 C 动作空间详情 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力")展示了我们实验中使用的所有种子查询的完整列表。关于最终查询如何构建的更多示例，详见[表 9](https://arxiv.org/html/2411.02305v1#A4.T9 "附录 D 沙盒环境详情 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力")。

## 附录 C 动作空间详情

在基于文本的代理设置中（即 ReAct 和 Act），行动包括 (1) 执行 SOSL 查询，(2) 执行 SOQL 查询，以及 (3) 提交答案。在函数调用设置中，行动是一系列精心设计的函数，详见[表 7](https://arxiv.org/html/2411.02305v1#A3.T7 "附录 C 动作空间详情 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力")。

| 模型 | HTU | TCU | NED | TII | PVI |
| --- | --- | --- | --- | --- | --- |
| Act |
| gpt-4o | 15.4 | 48.7 | 94.9 | 87.2 | 92.3 |
| gpt-4o-mini | 94.9 | 79.5 | 30.8 | 79.5 | 74.4 |
| claude-3.5-sonnet | 25.6 | 28.2 | 82.1 | 33.3 | 84.6 |
| claude-3-sonnet | 84.6 | 79.5 | 100.0 | 51.3 | 74.4 |
| llama3.1-405b | 56.4 | 51.3 | 46.2 | 38.5 | 0.0 |
| llama3.1-70b | 46.2 | 76.9 | 20.5 | 20.5 | 100.0 |
| ReAct |
| gpt-4o | 64.1 | 48.7 | 100.0 | 84.6 | 74.4 |
| gpt-4o-mini | 97.4 | 82.1 | 97.4 | 61.5 | 71.8 |
| claude-3.5-sonnet | 12.8 | 7.7 | 87.2 | 30.8 | 82.1 |
| claude-3-sonnet | 79.5 | 84.6 | 94.9 | 69.2 | 94.9 |
| llama3.1-405b | 53.8 | 38.5 | 97.4 | 41.0 | 64.1 |
| llama3.1-70b | 64.1 | 41.0 | 97.4 | 17.9 | 17.9 |
| 函数调用 |
| gpt-4o | 59.0 | 84.6 | 74.4 | 100.0 | 35.9 |
| gpt-4o-mini | 15.4 | 7.7 | 0.0 | 0.0 | 0.0 |
| claude-3.5-sonnet | 52.6 | 74.4 | 100.0 | 100.0 | 100.0 |
| claude-3-sonnet | 2.6 | 15.4 | 59.0 | 38.5 | 56.4 |

表 5：在CRMArena上，针对不可回答查询，各种LLM在不同代理框架下的表现（%）。

| 1\. 在[YEAR] [MONTH/QUARTER/SEASON]，确定处理超过[NCASES]案件并且平均处理时间为[EXTREMA]的代理。 |
| --- |
| 2\. 在过去的[TIMEPERIOD]，在处理超过[NCASES]案件的代理中，找到平均处理时间为[EXTREMA]的代理。 |
| 3\. 在过去的[TIMEPERIOD]，在处理超过[NCASES]案件的代理中，哪个代理的平均处理时间为[EXTREMA]？ |
| 4\. 在[YEAR] [MONTH/QUARTER/SEASON]，确定处理超过[NCASES]案件并且转接次数为[EXTREMA]的代理。 |
| 5\. 在过去的[TIMEPERIOD]，在处理超过[NCASES]案件的代理中，找到转接次数为[EXTREMA]的代理。 |
| 6\. 在过去的[TIMEPERIOD]，在处理超过[NCASES]案件的代理中，哪个代理的平均转接次数为[EXTREMA]？ |
| 7\. 代理违反了哪篇知识文章的政策？ |
| 8\. 今天是[TODAY]。在过去的[TIMEPERIOD]中，是否有某个月我们收到的[PRODUCT]案件明显多于其他月份？ |
| 9\. 今天是[TODAY]。在过去的[TIMEPERIOD]中，[PRODUCT]最常见的问题是什么？ |
| 10\. 今天是[TODAY]。在[YEAR] [MONTH/QUARTER/SEASON]，[PRODUCT]最常见的问题是什么？ |
| 11\. 今天是[TODAY]。在过去的[TIMEPERIOD]中，我们在哪些州关闭案件的速度最快？ |
| 12\. 今天是[TODAY]。在[YEAR] [MONTH/QUARTER/SEASON]，我们在哪些州关闭案件的速度最快？ |
| 13\. 为这个案件分配哪个代理最好？ |
| 14\. 今天是[TODAY]。请展示我在[PERIOD]前订购的[PRODUCT]。 |

表 6：用于查询生成的完整种子查询集。

函数描述  

表 7：功能调用设置的完整函数列表

## 附录 D 沙盒环境详情  

我们在[图7](https://arxiv.org/html/2411.02305v1#A4.F7 "附录D 沙箱环境详情 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中展示了对象及其依赖关系。这些对象，除了Knowledge__kav外，都紧密连接，反映了现实工作环境的复杂性。每个对象的条目总数显示在[表8](https://arxiv.org/html/2411.02305v1#A4.T8 "附录D 沙箱环境详情 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中。我们的数据生成流程显示在[图8](https://arxiv.org/html/2411.02305v1#A4.F8 "附录D 沙箱环境详情 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中。

对象条目数量 用户 100 联系人 196 产品类别 12 产品 500 订单项 71,00 价格表 44 价格表条目 22,000 案例 977 订单 2,071 电子邮件消息 3,234 实时聊天记录 387 知识 45

表8：每个对象的条目数量。

![参见说明](img/09daffdeae98842ecda2578e410f5638.png)

图6：在ReAct下，代理达到目标所需的回合数分布。

![参见说明](img/f791714bf22011a07ea63a3934d19ffb.png)

图7：我们沙箱环境中的对象及其依赖关系。

![参见说明](img/b9c2f56e9a80a553dae0436ac6ce273d.png)

图8：数据生成概览。每个对象的生成都依赖于之前生成的对象，箭头指向它们。蓝色框表示标准对象，而灰色框表示未上传至Salesforce Org的潜在变量。

<svg class="ltx_picture" height="249" id="A4.T9.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,249) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="221.44" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| 处理时间理解种子查询：在[年份] [月份/季度/季节]，识别出处理超过[N个案例]并且具有[极值]处理时间的代理。填充后的查询：在2021年2月，识别出处理超过2个案件并且具有最长处理时间的代理。转述查询：在2021年2月，确定处理超过2个案件并且具有最长处理时间的代理。 |

| 顶级问题识别种子查询：在[年份] [月份/季度/季节]，[产品]最常见的问题是什么？填充后的查询：在2023年第二季度，Flex瑜伽垫最常见的问题是什么？转述查询：2021年第二季度，Flex瑜伽垫最常见的问题是什么？ |

表9：查询生成过程的示例。

### D.1 对象详情

以下，我们描述了每个对象的详细信息。

+   •

    ProductCategory：代表组织产品的类别。

+   •

    Product2：代表贵公司销售的产品。

+   •

    ProductCategoryProduct：保存产品与产品类别之间的关系，用于将产品分配到某个类别。

+   •

    Pricebook2：代表包含产品列表的价格表。

+   •

    Pricebook Entry：代表价格表中的产品条目（即 Pricebook2 和 Product2 之间的关联）。

+   •

    订单：代表与合同或账户相关的订单。

+   •

    订单项：代表公司销售的订单产品。

+   •

    知识：供用户或客户访问的文档或信息文章。

+   •

    联系人：指与账户相关的个人或方。

+   •

    问题：代表客户提出的一类问题。

+   •

    Account：贵公司与之进行业务往来的实体、公司或个人。在 B2C 环境中，账户代表客户。

+   •

    用户（代理）：系统用户，通常代表客户支持代理。

+   •

    案例：描述客户咨询或问题的记录。

+   •

    CaseHistory：记录案例随时间变化的变更和更新。

+   •

    EmailMessage：与案例或客户咨询相关的电子邮件沟通，通常发生在代理和客户之间。

+   •

    LiveChatTranscript：代理和客户之间实时聊天会话的对话记录。

## 附录 E 提示

在本节中，我们展示了我们实验中使用的提示。 [Table 10](https://arxiv.org/html/2411.02305v1#A5.T10 "在附录 E 提示 ‣ CRMArena：理解 LLM 代理在现实环境中执行专业 CRM 任务的能力")，[Table 11](https://arxiv.org/html/2411.02305v1#A5.T11 "在附录 E 提示 ‣ CRMArena：理解 LLM 代理在现实环境中执行专业 CRM 任务的能力")，[Table 12](https://arxiv.org/html/2411.02305v1#A5.T12 "在附录 E 提示 ‣ CRMArena：理解 LLM 代理在现实环境中执行专业 CRM 任务的能力") 显示了 Act、ReAct 和 Function Calling 设置的系统提示。

<svg class="ltx_picture" height="429.24" id="A5.T10.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,429.24) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="401.68" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| 你是 Salesforce 的专家，并且可以访问 Salesforce 实例。 |

| 指令 - 你将得到一个问题、系统描述和相关的任务背景。 - 与Salesforce实例交互，适当构建Salesforce对象查询语言（SOQL）或Salesforce对象搜索语言（SOSL）查询，以帮助你回答问题。 - 可以使用Salesforce对象搜索语言（SOSL）构建基于文本的搜索查询，查询搜索索引。 - 你的生成应该始终是一个Action命令，仅此而已。仅生成一个Action命令。 - 不要生成任何系统观察结果，你将根据Action命令收到这些信息。 - 如果没有找到符合要求的记录，直接返回‘None’。以下是如何使用这些命令的说明： Action - 可以是’execute’或’submit’。 - execute，用于执行SOQL/SOSL查询，这将返回从Salesforce实例运行查询后的观察结果。 - submit，用于将任务的最终答案返回给用户。 - 格式：<execute> 一个有效的SOQL/SOSL查询 </execute> 或 <submit> 用户的回答 </submit> 指南 - 执行SOQL/SOSL查询，以了解Salesforce实例，这将帮助你找到问题的答案。 - 当你对答案有信心时，提交它。 - 总是以一个submit命令结束，其中仅包含答案，不要有完整的句子或任何解释。 示例 1 问题：机会的总数是多少？ 输出：<execute> SELECT COUNT() FROM Opportunity </execute>（如果Salesforce实例的观察结果是100，你的下一步可以是） <submit> 100 </submit> 而不是 <submit> 机会的总数是100 </submit> 示例 2 [… 隐藏详细信息以节省空间… ] Salesforce实例描述 在Salesforce实例中可用的对象有：用户、账户、联系人、案例、Knowledge__kav、产品类别、Product2等。 每个对象的字段及其描述和依赖关系如下： 用户 - FirstName：代理的名字 - LastName：代理的姓氏 - Email：代理的电子邮件地址 [… 隐藏详细信息以节省空间… ] 额外的任务背景 处理/转移时间政策 [… 隐藏详细信息以节省空间… ] |</foreignobject></g></g></svg>

表格 10：在 Act 设置中使用的系统提示。

<svg class="ltx_picture" height="516.53" id="A5.T11.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,516.53) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="488.97" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| 你是一个Salesforce专家，并且可以访问Salesforce实例。 |

| 指令 - 您将获得一个问题、系统描述和相关任务上下文。 - 请一步步思考，并与 Salesforce 实例互动，适当构建 Salesforce 对象查询语言（SOQL）或 Salesforce 对象搜索语言（SOSL）查询，以帮助您回答问题。 - 可以使用 Salesforce 对象搜索语言（SOSL）构建基于文本的搜索查询，以查询搜索索引。 - 生成内容应始终是一个思考，后跟一个行动命令，且**仅此**。只生成一个思考和一个行动命令。 - 不要生成任何系统观察，您将根据您的行动命令收到此信息。 - 如果没有符合要求的记录，请返回 'None'。以下是如何使用这些命令的说明： 思考 - 一行推理，用于处理上下文并告知决策。不要包括额外的行。 - 格式：<thought> 您的思考 </thought> 行动 - 可以是 'execute' 或 'submit'。 - execute，执行 SOQL/SOSL 查询，从 Salesforce 实例中运行查询并返回观察结果。 - submit，返回任务的最终答案给用户。 - 格式：<execute> 有效的 SOQL/SOSL 查询 </execute> 或 <submit> 回复给用户 </submit> 指导原则 - 总是从思考开始，然后进行行动。 - 每次只生成一个思考和一个行动命令。 - 执行 SOQL/SOSL 查询以理解 Salesforce 实例，帮助您找到问题的答案。 - 当您对答案有信心时，提交它。 - 始终以包含**仅有答案**的提交操作结束，**不包含完整句子或任何解释**。 示例 1 问题：机会的总数是多少？ 输出：<thought> 我需要找到系统中机会的总数。 </thought> <execute> SELECT COUNT() FROM Opportunity </execute> （如果从 Salesforce 实例中获得的观察结果是 100，您的下一步可以是） <thought> 我已找到机会的总数。 </thought> <submit> 100 </submit> 不是 <submit> 机会的总数是 100 </submit> 示例 2 [… 为节省空间隐藏详情… ] Salesforce 实例描述 Salesforce 实例中可用的对象有：User, Account, Contact, Case, Knowledge__kav, ProductCategory, Product2, … 可用的字段以及其描述和依赖关系： User - FirstName: 代理的名字 - LastName: 代理的姓氏 - Email: 代理的电子邮件地址 [… 为节省空间隐藏详情… ] 附加任务上下文 处理/转移时间政策 [… 为节省空间隐藏详情… ] |

表格 11：在 ReAct 设置中使用的系统提示。

<svg class="ltx_picture" height="361.3" id="A5.T12.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,361.3) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="333.74" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">| Instructions - You are an expert in Salesforce and you have access to a Salesforce instance. - You will be provided a question, the system description, and relevant task context. - Interact with the Salesforce instance using the tools provided to help you answer the question. - You should ALWAYS make ONLY ONE tool call at a time. If you want to submit your final answer, use the ’submit’ tool. If not, you should call some other tool. But ALWAYS make a tool call. - Always end by calling ’submit’ tool containing ONLY the answer, NO full sentence or any explanation. - If your answer is empty that is there are no records found matching the requirements mentioned, just return ’None’ to the ’submit’ tool. Additional task context Case Routing Policy The case routing policy determines the best agent to assign the given new case based on the following criteria - Issue Expertise: The agent who has closed the most cases associated with the issue most similar to the given case. - Product Expertise: If there is a tie in issue expertise, the best agent is the one who has solved the most cases associated with the product most relevant to the given case. - Workload: If there is still a tie, the best agent is the one that has least cases with Status not ’Closed’. Domain Details Quarters of the Year - Q1: January 1 to March 31 (both inclusive). - Q2: April 1 to June 30 (both inclusive). - Q3: July 1 to September 30 (both inclusive). - Q4: October 1 to December 31 (both inclusive). Seasons - Winter: December 1 to February 28/29 (both inclusive). - Spring: March 1 to May 31 (both inclusive). - Summer: June 1 to August 31 (both inclusive). - Autumn/Fall: September 1 to November 30 (both inclusive). Time Periods - Past 2 quarters: This refers to any timeframe spanning two quarters back from a specified ‘end_date‘. That translates to a six-month period retrospectively from the ‘end_date‘. - Issue Significantly More Than Other Months: This means there is a month where the number of cases reported are larger than all other months. |</foreignobject></g></g></svg>

表格 12：在函数调用设置中使用的系统提示。

## 附录 F 专家研究详情

职业 性别 年龄 客户服务助理 女 23 客户服务助理 女 25 客户服务代理 男 39 客户服务助理 男 29 客户服务顾问 男 49 客户服务经理 男 39 客户执行 女 25 技术支持 女 38 客户服务顾问 女 25 客户服务代理 女 35

表13：我们专家研究中参与者的背景。

如[表13](https://arxiv.org/html/2411.02305v1#A6.T13 "在附录F专家研究详情 ‣ CRMArena: 理解LLM代理在真实环境中执行专业CRM任务的能力")中详细说明，我们为研究招募了来自不同领域的专家。参与者在年龄、性别和职业背景上各不相同。

### F.1 招募标准

使用User Interviews平台，我们设置了职位筛选条件，确保参与者的职位头衔符合以下之一：

+   •

    客户经理

+   •

    技术支持工程师

+   •

    支持工程师

+   •

    技术支持专家

+   •

    技术支持经理

+   •

    技术支持技术员

+   •

    技术支持代理

+   •

    技术支持专家

+   •

    客户经理/代理

+   •

    客户经理/分析师

+   •

    客户服务顾问/客户服务助理

+   •

    客户服务助理

+   •

    客户服务代表

此外，我们还创建了一个筛选调查问卷。调查中的最重要问题是“您多久使用一次Salesforce CRM？”有效的候选人必须选择“每天几次”选项，才能参与我们的研究。

### F.2 研究

我们使用Google表单进行专家研究，因为它使用方便。研究分为三部分：

+   •

    第1部分：熟悉组织 [5分钟]。这是对该组织中一些对象的总体概述。

+   •

    第2部分：任务完成 [45分钟]。在这一阶段，参与者将被分配与客户服务相关的任务。他们应尽可能在45分钟内完成尽可能多的任务。

+   •

    第3部分：质量评分 [10分钟]。根据他们在研究前两部分中的经验，评估组织及其对象的质量。

以下，我们将展示每个部分是如何执行的。

#### 第1部分

在这一部分，我们向受访者提供登录凭证，供其访问我们创建的组织（沙箱环境）。登录后，他们被指示花5分钟阅读与他们后续将完成的任务相关的组织中的对象。该部分的说明和界面如[图9](https://arxiv.org/html/2411.02305v1#A6.F9 "第1部分 ‣ F.2 研究 ‣ 附录F专家研究详情 ‣ CRMArena: 理解LLM代理在真实环境中执行专业CRM任务的能力")所示。

![参见说明](img/8c795b44ab2b22a074a901cf579d90a8.png)

图9：我们专家研究第1部分的说明和界面。

#### 第2部分

在熟悉了我们创建的组织后，参与者被要求完成任务。他们需要完成来自 CRMArena 的 5 个查询实例。一个查询示例如图 [图10](https://arxiv.org/html/2411.02305v1#A6.F10 "在第二部分 ‣ F.2 研究 ‣ 附录 F 专家研究细节 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力") 所示。

![参见说明](img/a3cc157243a9f69abccad783a245ef68.png)

图10：专家研究第二部分的一个查询示例。

#### 第三部分

在完成专家研究的前两部分后，进入最后阶段，参与者被要求对我们的组织和数据的现实性进行评分。除了提供评分，他们还需要为其评分提供理由。一个示例问题见图 [图11](https://arxiv.org/html/2411.02305v1#A6.F11 "在第三部分 ‣ F.2 研究 ‣ 附录 F 专家研究细节 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力")。

下面，我们提供了参与者可以选择的评分和描述。

组织评分：

+   •

    非常不现实：该组织结构和设置感觉高度人造，与典型的 Salesforce 设置毫无相似之处。

+   •

    不现实：该组织有一些熟悉的元素，但重要部分的结构不够令人信服。

+   •

    中立：该组织感觉有些现实，融合了可信和不可信的元素。

+   •

    现实：该组织在很大程度上与现实世界中的 Salesforce 设置相似，只有少量不一致之处。

+   •

    非常现实：该组织感觉完全真实，极其接近现实世界中的 Salesforce 配置。

对象评分：

+   •

    我不知道/我不熟悉该对象。

+   •

    非常不现实：这些物体似乎在本质上存在缺陷，或者被发明出来时几乎没有考虑典型的 Salesforce 对象。

+   •

    不现实：这些物体具有可识别的特征，但通常并不代表实际的 Salesforce 对象。

+   •

    中立：这些物体具有适度的现实感，结合了现实和不现实特征的元素。

+   •

    现实：这些物体大多是现实的，并且与 Salesforce 中使用的典型对象很好地对接，只有少量问题。

+   •

    非常现实：这些物体感觉完全真实，与现实世界中的 Salesforce 对象完全匹配。

![参见说明](img/757b98b0f3e939b30c5788071e902d02.png)

图11：我们专家研究第三部分的一个示例问题。

### F.3 定性反馈

在 [表14](https://arxiv.org/html/2411.02305v1#A6.T14 "在 F.3 定性反馈 ‣ 附录 F 专家研究细节 ‣ CRMArena：理解 LLM 代理在真实环境中执行专业 CRM 任务的能力") 中，我们展示了受访专家的定性反馈和理由，他们评估我们的组织和对象是否被认为是现实或不现实的。

| 评分实例 | 评分 | 理由 |
| --- | --- | --- |
| 组织 | 现实 | 1. 这与一个正常的Salesforce实例非常相似（即我们公司使用的那个）。然而，在某些页面上存在一些缺失的细节，比如点击进入联系人或账户时。 |
| 2. 它看起来像我当前工作的Salesforce仪表板，我基本上可以了解模拟的整体导航。 |
| 3. 这就是我用Salesforce找到案例号码和客户识别的每个案例信息的方式。 |
| 4. 我对组织一无所知，但我还是能够摸索并找到我需要的内容。 |
| 不现实 | 1. 缺乏客户数据/信息填写其余字段。系统缺少“工作过”的痕迹，一切都显得非常空荡和混乱，界面没有填充内容。 |
| 对象 | 现实 | 1. 案例管理、客户互动、知识库和记录是使其真实的因素。 |
| 2. 我认为电子邮件通信并不完美，但感觉相当真实。 |
| 3. 我觉得案例和客户问题是真实的，因此我认为它们是现实的。 |
| 4. 它们具有与典型Salesforce部署类似的细节和结构（至少在我的公司是这样）。许多元素有相同的字段，且字段位于预期的位置（例如，右侧的详细信息和附加上下文）。 |
| 不现实 | 1. 不现实的是找到代理信息。这是不现实的，因为我应该能够筛选并找到每个代理转移并处理与客户的时间。 |

表14：领域专家提供的关于我们的沙箱环境真实感的评分理由示例。

## 附录G 实现细节

我们使用OpenAI API进行gpt模型，使用Amazon Bedrock API进行claude模型，使用Together API进行llama3.1模型。以下是我们测试的模型版本：

+   •

    gpt-4o: gpt-4o-2024-08-06

+   •

    gpt-3.5-turbo: gpt-3.5-turbo-0125

+   •

    claude-3.5-sonnet: anthropic.claude-3-5-sonnet-20240620-v1:0

+   •

    claude-3-sonnet: anthropic.claude-3-sonnet-20240229-v1:0

+   •

    llama3.1-405b: meta-llama/Meta-Llama-3.1-405B-Instruct-Turbo

+   •

    llama3.1-70b: meta-llama/Meta-Llama-3.1-70B-Instruct-Turbo

我们选择了ReAct设置，而不是基于计划的方法，因为后者将任务分解为更易管理的步骤，之前的研究表明，在基于SQL的数据库查询任务中，计划策略对于根据执行反馈调整计划的灵活性较差（Yang et al.，[2024](https://arxiv.org/html/2411.02305v1#bib.bib18)）。我们将每个实例的最大操作次数设置为20，温度设置为0，top_p设置为1，适用于所有实验。

功能依赖功能任务查询独立获取订单项ID按产品(product_id) MTA 获取订单项ID按产品(product_id) NCR 搜索产品(search_term) NED 获取账户ID按联系人ID(contact_id) NED 查询依赖获取未转移案件ID(start_date, end_date) HTU 获取案件(start_date, end_date, agent_ids, case_ids) NCR 获取案件(start_date, end_date, agent_ids, case_ids) BRI 获取案件(start_date, end_date, agent_ids, case_ids) HTU 计算独立获取开始日期(end_date, period, interval_count) TCU 获取开始日期(end_date, period, interval_count) BRI 获取开始日期(end_date, period, interval_count) TII 获取期间(period_name, year) TCU 计算依赖计算区域平均关闭时间(cases) BRI 获取符合条件的代理ID按案件数量(agent_handled_cases, n_cases) TCU 计算平均处理时间(cases) HTU 获取处理案件最多的代理(subset_cases) NCR

表 15：在[表 4](https://arxiv.org/html/2411.02305v1#S3.T4 "如何函数类型影响模型性能？ ‣ 3.3 讨论 ‣ 3 基准测试实验 ‣ CRMArena：理解LLM代理在现实环境中执行专业CRM任务的能力")中测试的功能和任务列表。
