# From Prompt to Agent：DA 团队如何构建可落地的分析智能体

> 时长：约 45 分钟（正文约 42–43 分钟，留 2–3 分钟机动）  
> 副标题：Prompt · Skill · Tool · MCP · Taxonomy · Harness · Validation · Dashboard Agent  
> 演讲定位：概念讲清楚、方法可复用、案例能落地。  
> 使用方式：`【屏幕】`是建议停留的 HTML 区块；`【略讲】`控制在 20–40 秒；`【重点】`建议放慢并配合页面图示。

---

## 时间分配

| 时间 | 内容 | 讲法 |
|---|---|---|
| 0:00–2:00 | 开场与今天讲什么 | 建立问题，给全景地图 |
| 2:00–5:00 | Chat / Copilot / Workflow / Agent | 略讲，统一概念 |
| 5:00–8:00 | Agent 的完整组成 | 略讲，用“新分析师”类比 |
| 8:00–15:00 | Prompt / Skill / Tool / MCP / Taxonomy / Harness | 重点，六个概念一次讲清 |
| 15:00–20:00 | DA 最适合做哪些 Agent | 分类讲，补未来展望 |
| 20:00–25:00 | 怎么建好一个 Agent | 重点，强调先 Workflow 后 Prompt |
| 25:00–29:00 | Prompt Contract | 重点，用好坏 Prompt 对比 |
| 29:00–34:00 | Harness / Taxonomy / Validation | 重点但紧凑 |
| 34:00–43:00 | Dashboard Design Agent 重点案例 | 全场核心，可延长到 45 分钟 |
| 43:00–45:00 | 成熟度、落地路径与收束 | 给下一步行动 |

---

## 0:00–2:00｜开场：今天不是一堂名词课

【屏幕：00 封面 → 01 今天讲什么】

大家好，今天我想跟大家聊一个最近很热、也很容易被讲乱的话题：Agent。

我在 DA 里面经常看到两种情况。第一种是，我们已经在用 AI 写 SQL、写 Python、总结报告，效率确实提高了；第二种是，我们把这些能力都叫 Agent，但再往下问——这个 Agent 的输入是什么、用什么工具、出错怎么办、怎么验证、换个人还能不能稳定复现——答案往往还没有完全想清楚。

所以今天我不想把它讲成一堂“技术名词培训”。我想围绕一条主线：

> 我们怎样从“让 LLM 帮我写一段 SQL”，走到“定义一个可以稳定完成分析任务的 Agent”，最后再让多个 Agent 组成一套可治理的分析工作流。

今天主要讲九件事。前面先把 Agent 是什么、由什么组成讲清楚；中间重点区分 Prompt、Skill、Tool、MCP、Taxonomy 和 Harness；然后讲 DA 最适合做哪些 Agent、怎么建好一个 Agent、怎么写 Prompt Contract，以及怎么验证。最后用比较多的时间，拆一个我们身边真实的问题：Dashboard Design Agent 到底怎么做。

这个案例不是“让 AI 再做一张漂亮图”。我们现在的 MI 困境是：每个人手上都有很多 Reports，可能重复，也可能只重复一部分；大家再用 AI Coding 各自做 Dashboard，页面、交互和聊天入口都会慢慢分叉。我们真正想讨论的是：能不能做一个大家共用的 Dashboard Design Agent？以后其他人只要给它一张输入表，再通过配置说明“哪个模块展示什么、使用哪个字段”，它就自动生成一个统一的 WebApp Dashboard。

今天所有抽象概念，我都会尽量回到同一个例子：Retail Banking Monthly Business Review。比如这个月 AUM 为什么下降，Agent 去哪里取数，按什么路径下钻，怎样证明结论是真的，以及怎样把它放进 Dashboard。

---

## 2:00–5:00｜从“回答问题”到“完成任务”

【屏幕：02 从 Copilot 到 Agent】

先统一四个概念：Chat、Copilot、Workflow 和 Agent。

Chat 最简单，就是回答一次。比如我问“AUM 是什么”，它给我一个定义，任务就结束了。

Copilot 是人主导、AI 辅助。比如分析师已经决定要查哪张表，Copilot 帮我补 SQL、解释报错，但下一步做什么仍然由人决定。

Workflow 是固定流程自动跑。比如每天 9 点刷新数据、生成 PDF、发邮件。它很有价值，而且很多任务用 Workflow 就够了，不需要硬做成 Agent。

Agent 的差别是：它围绕目标，根据中间结果决定下一步。比如目标是“找出 AUM 下跌主因”。Agent 先读 KPI，发现总量下降，再判断应该先按客户层级还是产品类型下钻；查完发现 72% 的变化来自 Affluent 客群的 TD 到期，于是继续看续存率；最后它还要检查分项能不能加回总量，并给出证据。

所以 Agent 的核心不是“用了 LLM”，而是这几个东西能形成闭环：Goal、Context、Reasoning、Tools、State、Action、Validation。

什么时候值得做 Agent？通常是步骤多、会分支、要跨系统、而且有重复频率的任务。例如每月都要做 MBR，但每个月异常指标不同、下钻路径也不同。

什么时候不值得？一次性翻译不需要；每天搬一个 CSV 用脚本更稳；“帮我提升整个 MI”这种边界都没定义的任务，也不能急着做 Agent。先把它收敛成“检查 12 个 KPI 的环比异常”，才有可能设计和验收。

一句话记住：如果任务需要根据环境反馈持续决定下一步，它更像 Agent；如果 A、B、C 永远固定，Workflow 往往更简单、更可靠。

---

## 5:00–8:00｜一个完整 Agent 的组成

【屏幕：03 Agent Anatomy】

我们可以把 Agent 想成一个刚入职、速度很快，但还不了解我们业务的分析师。

首先它要有大脑，也就是模型、Prompt 和推理策略。比如看到 NNM 下降，它要知道先区分“流入减少”还是“流出增加”，而不是直接开始写一段故事。

第二，它要有手，也就是 Tools 和 MCP。比如调用 SQL 查数、用 Python 做贡献度分析、读取指标定义、把结果写进 Dashboard。没有手的 Agent 只能聊天，不能完成任务。

第三，它要有知识。比如它必须知道 CASA 包括 Current Account 和 Savings Account，但不包括 TD；也要知道这一次报告只看 HK Retail，不包括 SME。

第四，它要有工作法。Skill 告诉它“月报分析通常怎么做”，Harness 把这些步骤真正组织起来。例如必须先通过数据质量检查，才允许生成管理层结论。

第五，它要有治理。比如不允许读取客户级 PII；金额对不上总账必须停止；“活动导致增长”这种因果判断必须交给分析师确认；同时每一步调用了什么工具、用了哪个版本，都要能回看。

所以这里我会用一个很简单的记法：

> Agent = Brain + Hands + Knowledge + Workflow + Governance。

少了任何一块，都可能做出一个看起来很聪明、但不适合进生产的 Demo。

---

## 8:00–15:00｜六个核心概念到底有什么区别

【屏幕：04 六个核心概念辨析】

【重点】这一节是今天最重要的概念部分。我用同一个任务来讲：做一份 Retail Banking Monthly Business Review，解释本月 AUM 变化。

### Prompt：这一次怎么做

Prompt 是给模型的执行指令和约束。它告诉 Agent：比较 MoM、YoY 和 Target；变化超过什么阈值才下钻；每条结论必须带什么证据；不知道时应该怎么办。

比如 Prompt 可以写：“只输出影响最大的三项；所有金额必须来自 KPI Tool；相关性不能写成因果；证据不足就标记待验证。”

所以 Prompt 像任务单或者岗位说明书。但只有岗位说明书，不代表员工已经有数据库权限，也不代表系统会自动重试。

### Skill：这一类任务通常怎么做

Skill 是一套可以复用的专业做法。比如“月报分析 Skill”规定：先做数据 QA，再看总量，再比较 MoM、YoY、Target，然后按 Product、Segment、Channel 分解，最后按影响大小排序并写成管理层语言。

举个通俗的例子：Tool 像 Excel，Skill 是“如何用 Excel 做一套标准预算分析”。Skill 不是一个按钮，而是专业方法、步骤、模板和检查清单的组合。

### Tool：具体能做什么

Tool 是 Agent 可以调用的具体动作，例如执行 SQL、运行 Python、搜索报告、渲染图表、提交评论、发送邮件。

比如 `run_sql` 这个 Tool 接收一段受控查询，返回按产品分组的 AUM 变化。它能精确执行动作，但它自己不知道什么时候应该按产品查，什么时候应该按客户层级查。

所以“有工具”和“会正确使用工具”是两回事。锤子不会告诉我们哪里应该钉钉子。

### MCP：外部能力怎么统一接进来

MCP 可以理解成 Agent 世界里的通用插座。数仓、Data Dictionary、报告目录、文件系统，本来都有不同接口；通过 MCP，它们可以用相对统一的方式向 Agent 暴露 Tools、Resources 和上下文。

例如 Agent 先发现有一个 `get_metric_definition` Tool，再调用它读取 AUM 的批准口径。换一个数据平台时，只要仍然通过标准接口暴露能力，上层 Agent 不必把所有连接逻辑重写一遍。

但 MCP 不是数据库，也不是业务逻辑。插座负责连接，不会替我们决定要接咖啡机还是投影仪。

### Taxonomy：大家说的是不是同一个东西

Taxonomy 是业务对象的统一分类、命名和关系。比如 Deposit 下面有 CASA、TD、SD；Investment 下面有 MF、QDII、Banca；Flow 下面有 External In、External Out、Internal Transfer。

为什么重要？如果 Agent A 把 SD 归 Deposit，Agent B 把 SD 归 Investment，两个 Agent 各自看起来都能解释，合在一起却永远对不上。再比如 TD 转到 MF 是内部产品迁移，不应该算成整个客户层面的 External Out。

Taxonomy 就是 Agent 之间共享的业务语言。语义层会再往前一步：除了“它是什么”，还定义“怎么算、从哪取、按什么方式聚合、谁负责、当前版本是什么”。

### Harness：整套东西怎样真正跑起来

Harness 是运行和治理框架。它负责加载上下文、选择 Skill、路由 Tool、保存状态、处理超时和重试、运行验证、插入人工审批、记录模型与 Prompt 版本。

例如查询第 7 个 KPI 时超时，Harness 只重跑这一节点，而不是把整份月报重新做一遍；如果分项合计对不上总变化，它不让 Report Writer 继续“发挥”，而是停止并通知 Data Owner。

这里有一句我建议大家记住：

> Prompt 决定 Agent 会不会做；Harness 决定 Agent 能不能长期稳定地做。

最后把六个词连起来：Prompt 是任务单，Skill 是 SOP，Tool 是工具，MCP 是通用插座，Taxonomy 是共同语言，Harness 是整条生产线。Agent 是这些组件组合以后，围绕目标持续运行的系统。Prompt 不是 Agent，Tool 不是 Agent，MCP 也不是 Agent。

---

## 15:00–20:00｜DA 最适合做哪些 Agent

【屏幕：05 DA 最适合做哪些 Agents】

DA 的 Agent 可以沿着一条链来看：Data、Analysis、Insight、Report、Action。

在 Data 这一层，我最看好三个方向。

第一个是 Data Discovery Agent。比如有人问“有没有客户流失数据”，它不是只返回一个表名，而是返回三张候选表、它们的口径差异、Owner、freshness、已有 Dashboard 和推荐优先级。未来它会从关键词搜索走向语义、血缘和使用历史一起判断可信度。

第二个是 SQL / Data Preparation Agent。例如我们先确认 CASA 口径，再让 Agent 生成 Segment × Month 的汇总查询和测试。未来它如果和语义层结合，就能减少每个人重复写相似 SQL。

第三个是 Data Quality Agent。比如 TD 本月突然少 18%，它先检查源表是不是迟到，而不是马上写“客户资金流失”。未来它还可以顺着血缘自动定位上游节点和受影响的 Reports。

在 Analysis 这一层，可以做 KPI Analysis、Root Cause / Driver、Forecast / Scenario Agent。

KPI Agent 的例子是：AUM 环比 -2.1%，比 Target 低 1.4 个百分点，然后按客户层级给出贡献。Driver Agent 可以说“72% 的下降来自 Affluent 客群 TD 到期”，但不能因为同时发生利率变化就写“利率导致下降”；它应该把这句话标成待验证假设。Scenario Agent 则可以模拟：如果 TD 续存率提高 3 个百分点，季度末 Deposit 会发生什么变化。

在 Insight 和 Report 这一层，可以做 Dashboard Insight、Monthly Report、Executive Summary、Campaign / Journey 和 Challenger Agent。

Dashboard Insight Agent 不是把所有红点念一遍，而是找出对总 AUM 真正重要的三项。Executive Summary Agent 也不只是把原文缩短，而是针对管理层保留“发生什么、影响多大、建议做什么”。Challenger Agent 专门找漏洞，例如质疑“活动带来增长”，因为同期季节性还没有排除。

还有一个常被低估的基础 Agent：Metadata / Dictionary Agent。比如有人问“Active Client 是 30 天还是 90 天口径”，它应该给出批准版本、Owner 和使用范围，而不是从历史聊天里猜。

如果要排优先级，我建议 DA 先做 Data Discovery、Data Quality、Dashboard Insight / Monthly Report。它们频率高、边界清楚、容易准备 Golden Set。不要一开始做一个“懂所有数据、会所有分析”的万能 Agent。

---

## 20:00–25:00｜怎么建好一个 Agent：不要先写 Prompt

【屏幕：06 怎么建好一个 Agent】

【重点】很多人做 Agent 的第一步，是打开聊天框开始写 Prompt。我建议顺序反过来：先把它当成一个分析产品来设计。

第一步，定义 Business Goal。不要写“我要做一个月报 Agent”，要写业务结果。例如：“把 MBR 初稿准备时间从两天降到半天，同时不降低数字准确率。”这决定了我们是在追求速度、准确、覆盖率，还是别的东西。

第二步，定义 Input Contract。输入从哪里来、什么粒度、什么版本、什么时候算 ready。比如只允许使用已关账的月度快照；freshness 超过 36 小时就不生成正式结论。

第三步，定义 Output Contract。比如每条 Insight 必须包含 KPI、变化值、最大贡献项、证据链接、置信度和 Owner。这样输出才能被 Dashboard、报告和人稳定消费。

第四步，拆任务。把“分析一下”拆成：QA、KPI 计算、异常检测、下钻、贡献分解、洞察排序、叙事生成、人工审核。每个中间结果都应该可检查。

第五步，决定 Tool、LLM 和人的分工。精确计算交给 SQL 或 Python；对模糊问题的理解、信息组织和表达交给 LLM；高风险判断交给人。比如环比不能让模型心算；“TD 到期导致客户流失”这种因果结论要人工批准。

第六步，准备 Knowledge 和 Taxonomy。比如 AUM 等于 Deposit 加 Investment；Wealth 是展示别名，不允许另算一套口径；Internal Transfer 不影响总体 NNM。

第七步，在开发之前定义 Validation。比如 12 个 KPI 的数字必须 100% 对齐；重大问题召回率达到 90%；没有证据的结论必须为 0。

第八步，设计失败。找不到 Target 怎么办？正确做法不是猜，而是写“Target unavailable”，继续完成 MoM 分析，同时把缺口记录下来。SQL 超时可以重试一次；对账失败必须停止；权限不足要转人工。

最后才是把这些设计写成 Prompt，做测试、迭代和版本管理。

所以这一节只需要记住一句：

> 先设计 Workflow，再写 Prompt。

没有输入、输出、验证和失败路径，再长的 Prompt 也只是一次性 Demo。

---

## 25:00–29:00｜Prompt Contract 怎么写

【屏幕：07 Prompt Contract】

一个好的 Agent Prompt，不是写得越长越好，而是像一份执行契约，尽量减少关键歧义。

我一般建议至少考虑十二个字段。

Role：你是谁。例如“你是 Retail MI 分析 Copilot，为 DA 分析师准备可审阅初稿”。

Objective：最终目标。例如“识别本月对 AUM 影响最大的三个变化”。

Business Context：范围、受众和决策。例如只看 HK Retail，受众是月度业务评审会。

Input Contract：允许使用什么。例如只使用批准 KPI 快照和 Report Catalog，不读取客户 PII。

Definitions：口径和阈值。例如变化超过 1%，并且绝对金额超过 10M，才算 Material。

Task Steps：必须按什么顺序。例如先 QA，再比较，再下钻；QA 未通过，不得写结论。

Tool Policy：何时调用什么。例如所有金额只允许由 KPI Tool 返回，模型不得自己计算。

Reasoning Rules：判断原则。例如贡献可以陈述，因果必须有额外证据。

Guardrails：不能做什么。例如不展示少于 10 人的小客群，不写生产表。

Validation：输出前检查什么。例如分项贡献与总变化误差不能超过 0.1%。

Output Schema：固定输出结构。例如 metric、delta、driver、evidence、confidence。

Abstention 和 Examples：不知道时怎么办，并给正反例。例如证据不足就写“待验证假设”，再给下一步查询，而不是编一个听起来合理的原因。

对比一下。差 Prompt 是：“帮我分析这个月 Dashboard，有什么 insight？”它没有范围、基线、阈值、证据、结构，也没有失败规则。

好 Prompt 会明确：比较 MoM、YoY、Target；只看超过阈值的变化；数值只能来自 Tool；相关不能写成因果；每条结论都要有 evidence ID；如果信息缺失就明确 unavailable。

专业性不在于 Prompt 写得很复杂，而在于我们把“什么叫完成、什么叫正确、什么情况下应该停”说清楚了。

---

## 29:00–34:00｜Harness、Taxonomy 和 Validation：三块地基

【屏幕：08 Harness → 09 Taxonomy → 10 Validation】

### Harness

Harness 管的是整套运行过程：步骤编排、上下文加载、Tool 路由、状态、重试、验证、人工审核、日志、版本、成本和延迟。

举例来说，QA Agent 没有通过，Harness 就不启动 Insight Agent；Retail 任务只加载 Retail 的口径，避免混入 SME；第 7 个 KPI 超时，只重跑该节点；Prompt v4 误报增加，可以回滚 v3；三个月后还能根据 Run Manifest 复现当时使用的数据快照、Prompt 版本和审批人。

这就是 Demo 和 Production 的差别。Demo 只需要这一次跑通；Production 要求它长期可控地跑。

### Taxonomy 与 Semantic Layer

Taxonomy 统一分类和关系，Semantic Layer 统一计算和来源。

比如 Deposit 下面有 CASA 和 TD；Investment 下面有 MF 和 Banca；External In 减 External Out 得到 NNM，而 Internal Transfer 只是产品结构变化。

在语义层里，一个 Metric 还应该包含定义、公式、数据来源、可用维度、Owner 和版本。例如 `metric.casa.balance.v3` 的数据来自 `deposit_monthly_snapshot`，按批准的 CASA 产品集合求和，Owner 是 Deposit MI。

这样不同 Dashboard 可以长得不一样，但引用同一个 Metric ID，事实不会分叉。

### Validation

Agent 的验证不能只问“回答看起来是不是不错”。至少要分层看。

Data correctness：数据完整、及时、对账了吗？例如 12 个月是不是齐全。

Calculation correctness：公式和聚合对吗？例如分项贡献能不能加回总变化。

Business correctness：口径范围对吗？例如 CASA 没有把 TD 算进去。

Reasoning quality：证据真的支持结论吗？例如相关性有没有被写成因果。

Usefulness：是不是只挑重要、可行动的内容？例如给 Top 3，而不是把 30 个红点都复制出来。

Reproducibility：数据、Prompt 和 Tool 版本是否可追溯。

具体方法可以是 Golden Set、Regression Test、Challenger Agent 加 Human Review。比如从过去 12 个月选出 20 类典型异常，由资深分析师保存正确的 KPI、下钻路径和结论；以后每次改 Prompt 或 Tool，都跑一遍回归。

---

## 34:00–43:00｜重点案例：Dashboard Design Agent 怎么做

【屏幕：11 Dashboard Design Agent；按五个 Tab 依次讲】

【重点，建议放慢】现在进入今天最核心、也最贴近我们日常工作的案例。

### 第一，为什么要做这个 Agent

我们现在的 MI 困境，不是大家不会做 Report，而是每个人手上都有很多 Reports。有些完全重复，有些只重复一部分。

比如 A 和 B 都有一张 AUM Report。A 的版本按 Segment 展示，B 的版本按 Product 展示；两个 Report 里可能有 70% 的内容是一样的。

接下来，每个人又需要用 AI Coding 把自己的 Report 做成 Dashboard。短期看，这样很快；但过一段时间就会发现，同一类 Dashboard 出现了好几套代码、好几种布局、好几种图表风格。A 的页面有筛选器，B 的没有；C 的聊天界面知道当前月份，D 的聊天界面却不知道用户正在看什么。

更麻烦的是，后面只是想改一个字段，也可能要重新和 AI 解释整套需求。比如本来趋势图展示 AUM，现在想换成 NNM，AI 可能重新生成一大段代码，而不是只改这一个模块。

所以我想做的不是一个特别复杂的“万能 Dashboard Agent”。我想做的是一个大家可以直接拿来用的 Dashboard Design Agent：

> 使用者提供一张输入表，再告诉 Agent“哪个模块展示什么、使用哪个字段”，Agent 自动生成一个统一的 WebApp Dashboard。

以后大家不需要各自从零写 React、图表组件和聊天框。大家共用同一个 Dashboard 底座，差异只放在输入表和配置里面。

### 第二，用户到底怎么使用

整个过程可以非常直观，只有六步。

第一步，上传输入表。可以先支持 CSV 或 Excel。例如表里有 Month、Segment、Product、AUM、NNM、Target。

第二步，Agent 读取字段。它会识别 Month 是日期，Segment 和 Product 是分类字段，AUM、NNM、Target 是数值字段。

第三步，使用者填写 Dashboard 配置。比如上面要三张 KPI Card，中间要一张 AUM 月度趋势，下面要一个按 Product 的柱状图和一个明细表。

第四步，Agent 自动检查配置。比如折线图的 x 轴应该是 Month，y 轴应该是 AUM；如果用户误把 Product 放到 y 轴，Agent 要给出人能看懂的错误提示。

第五步，Agent 生成 WebApp。页面、组件、数据绑定、颜色、金额格式、响应式布局和统一 Copilot 都自动带出来。

第六步，用户看预览。如果想修改，不需要重新生成整套网站，只改配置。例如说“趋势图按 Segment 分颜色”，Agent 只增加 `color: Segment`。

这里最重要的是：Agent 不是每次自由发挥写一个新网站，而是在我们已经做好的一套组件里进行组合。

### 第三，配置到底要填什么

这个配置其实可以很通俗。

如果我想展示“总 AUM”，就选择 KPI Card，value 字段选择 AUM，aggregation 选择 sum。

如果我想展示“AUM 月度趋势”，就选择 Line Chart，x 轴选择 Month，y 轴选择 AUM。

如果我想展示“各产品的 AUM”，就选择 Bar Chart，x 轴选择 Product，y 轴选择 AUM。

如果我想按客群筛选整个页面，就加一个 Filter，字段选择 Segment。

如果我想看原始明细，就加一个 Data Table，列选择 Month、Segment、Product 和 AUM。

所以配置本质上是在回答四个问题：这个模块是什么、它展示什么、使用哪个字段、怎样聚合。

使用者必须提供的是输入表、页面标题、筛选字段、模块类型和模块字段。Agent 自动完成的是布局、颜色、格式、数据绑定和前端代码。

举个最简单的配置：

`data.file` 是 `mi_monthly.xlsx`；筛选器是 Month 和 Segment；第一个模块是 KPI Card，value 是 AUM；第二个模块是 Line Chart，x 是 Month、y 是 AUM；第三个模块是 Bar Chart，x 是 Product、y 是 AUM。

这份配置保存下来以后，换一个业务团队也可以继续使用。他们只需要把文件换成自己的输入表，再把 AUM、Product 这些字段改成自己表里的字段。

### 第四，怎么把所有模块通过 Dashboard 串起来

这里不需要讲得很复杂。核心是：所有模块使用同一份数据，也使用同一个筛选状态。

比如页面顶部有 Month 和 Segment 两个全局筛选器。用户选中 `2026-09` 和 `Affluent` 之后，AUM 卡片、月度趋势、产品分布和明细表要一起刷新。

Copilot 也要读取同一份状态。用户问“为什么 AUM 比较低”，Copilot 知道他现在看的是 2026 年 9 月、Affluent 客群，而不是全量数据。

所以所谓“串起来”，不是每个图表各做各的。Dashboard 有一个共同的数据层和筛选状态，每个模块只是用不同方式展示同一批数据。

### 第五，Copilot 应该怎么写

这个 Copilot 的主要职责，是帮助用户完成和修改 Dashboard 配置，而不是每次替用户重新写一套网站。

它需要知道四样东西：输入表的字段和类型；平台支持哪些组件；当前 Dashboard 配置是什么；当前页面选择了什么筛选条件。

它可以做四类事情。

第一，根据自然语言生成配置。比如用户说：“上面放 AUM、NNM、Target 三张卡，中间放 AUM 趋势，下面按 Product 展示 AUM。”Copilot 把这句话转换成模块配置。

第二，修改某个模块。比如“把 Product 柱状图改成横向，只显示 Top 10”。

第三，发现字段不匹配。比如用户说使用 Revenue，但输入表里只有 Income。Copilot 不能自己猜这两个是不是一回事，而要询问用户。

第四，解释当前 Dashboard。比如用户问“这张图展示什么”，Copilot 根据模块配置和当前筛选条件回答。

Copilot 的 Prompt 可以这样写：你是 Dashboard Design Copilot；输入是表结构、当前配置、组件库和用户请求。先检查字段是否存在、类型是否合理，再把需求映射到支持的组件，只修改必要的配置，运行校验，通过以后再生成 WebApp。

同时规定几条硬规则：不能创造输入表里不存在的字段；不能偷偷生成组件库以外的组件；数值字段必须说明 sum、average 还是 count；日期字段必须说明按日、月还是季度；不确定字段含义时先问用户。

它需要的 Tools 也很清楚：`inspect_table` 读取表结构，`list_components` 查看支持的组件，`validate_config` 检查配置，`generate_webapp` 生成页面，`preview_dashboard` 返回预览。

### 第六，真正需要完成什么

要让其他人直接拿来用，我们需要完成六块能力。

第一，Input Adapter，负责读取 CSV、Excel 或批准的数据视图，并识别字段类型。

第二，Config Schema，定义筛选器、模块、字段、聚合、布局和主题可以怎样填写。

第三，Component Library，先做好六个最常用组件：KPI Card、Line Chart、Bar Chart、Pie Chart、Data Table 和 Filter。

第四，WebApp Generator，把配置转换成页面、数据绑定、交互和可运行代码。

第五，Design Copilot，让用户通过自然语言新增或修改配置。

第六，Validator，检查字段是否存在、类型是否合理、每个模块能不能正常渲染。

第一个 POC 不需要做得很大。只支持一种标准输入表、六个组件、三类全局筛选器和一个统一主题。使用者可以上传表、填写配置、生成 WebApp，再通过 Copilot 修改一个模块，这个闭环跑通就已经很有价值。

第一版也要主动控制边界：不支持任意前端组件，不让 Agent 自由重写整套代码，不兼容所有历史 Report 格式，也先不把复杂的洞察分析混进来。

这个 Agent 第一阶段只做好一件事：

> 把输入表和简单配置，稳定地变成一个统一、可修改、可复用的 WebApp Dashboard。

---

## 43:00–45:00｜成熟度、落地路径与收束

【屏幕：12 成熟度与落地路径】

最后用成熟度把今天的内容收起来。

L0 是 Prompt，一次性对话；L1 是 Prompt 加 Tool，能查数据；L2 是 Structured Workflow，步骤固定下来；L3 加 Harness、Validation、State 和 Logging，开始可生产；L4 是多个小 Agent 分工并加入治理；L5 才是跨业务域的 Production Agent System。

所以“我在 ChatGPT 里写了一段很长的 Prompt”，其实只是第一层，不是做得不好，而是还没有走到后面的工程化和治理层。

DA 团队可以从三个 POC 开始。

第一个是 Dashboard Design Agent：根据输入表和模块字段配置，自动生成统一的 WebApp Dashboard。成功标准可以是标准配置生成成功率 100%、字段映射错误为 0。

第二个是 Monthly Report Agent：把批准 KPI、图表和 Insight 组成可审阅初稿。成功标准可以是准备时间下降 50%，数字准确率 100%。

第三个是 Data Discovery / QA Agent：先解决“数据在哪里、能不能信、谁负责”。成功标准可以是常见数据问题首次命中率 85% 以上。

不管选哪一个，每个 POC 都必须带四样东西：一个明确任务、一套固定输入、一个标准输出、一套 Validation。

我今天最后想留下的一句话是：

> Agent 不是更长的 Prompt，而是一套可运行、可验证、可治理的分析工作系统。

我们不需要从“万能 Agent”开始。先把共同事实层和一个最小闭环做好，再把 Data QA、Analysis、Insight、Report、Review 一段一段串起来。这样做出来的东西，才不仅是 Demo，而是真正能进入 DA 日常工作的能力。

谢谢大家。

---

## 现场可能被问到的问题（备答，不计入 45 分钟）

### 1. 有了 Power BI / Tableau，为什么还要做 Dashboard Design Agent？

它不一定要替代现有 BI 工具。它解决的是“每个人都要重复设计页面、选择组件、绑定字段和做聊天入口”的问题。

例如一个团队只要说明“Month + AUM 做趋势，Product + AUM 做柱状图”，Agent 就能生成统一 WebApp；后续也可以把同一套配置思想适配到现有 BI 平台。

### 2. 这是不是普通 Dashboard 模板，不一定需要 Agent？

模板负责提供稳定组件，Agent 负责理解需求、映射字段、生成配置、检查错误和帮助修改。两者是配合关系。

例如没有 Agent 时，用户也可以手动改 YAML；有 Agent 后，用户可以直接说“增加一个按 Segment 的筛选器”，由 Agent 生成准确的 config patch。

### 3. 每个人的输入表结构都不一样怎么办？

第一版不要试图兼容所有格式，先定义一份标准输入表或一个字段映射步骤。

例如平台要求至少有 Date、Dimension、Metric 三类字段；如果用户的字段叫 Report_Month，上传时把它映射为 Month。先标准化，再生成页面。

### 4. Agent 能不能自动帮用户选择图表？

可以推荐，但推荐要基于简单、透明的规则，最终配置仍然可见、可改。

例如“日期 + 数值”优先推荐折线图，“类别 + 数值”优先推荐柱状图，“单个数值”推荐 KPI Card。Agent 不应该为了好看随意使用难以理解的图表。

### 5. Dashboard 里的 Copilot 怎样知道用户正在看什么？

Copilot 读取当前 Dashboard 配置、当前模块和全局筛选状态，而不是单独维护另一套上下文。

例如页面选择了 Month = 2026-09、Segment = Affluent，Copilot 的回答也只基于这一范围。

### 6. 怎么衡量 Dashboard Design Agent 的 ROI？

同时看效率、质量和复用。

例如：从输入表到可预览 Dashboard 的时间、标准配置生成成功率、字段映射错误数、组件复用比例、样式一致性、修改一个模块所需时间，以及非开发人员能否独立完成。

