# 总控 Agent：任务入口与需求澄清设计规格

- **产品：** AstraWork
- **规格版本：** 1.0
- **状态：** 已确认
- **日期：** 2026-08-15
- **目标平台：** Windows 11 桌面端
- **参考项目：** [MadsGao/frakio-work](https://github.com/MadsGao/frakio-work)
- **关联原型：** `.superpowers/brainstorm/2047-1786785912/content/`

## 1. 摘要

本规格定义 AstraWork 总控 Agent 的首个功能：任务入口与需求澄清。

该功能接收新项目想法或现有项目开发任务，将自然语言、文档、图片、代码、外部链接和运行现场转化为可追溯的结构化需求。总控 Agent 每轮只提出一个影响范围或实施的重要问题。需求达到最低完备门槛后，系统生成版本化需求包，供用户逐章审阅并整体批准。

功能采用「自然对话 + 结构化需求账本」架构：

- 自然对话负责低门槛输入、解释和追问。
- 结构化需求账本负责事实、证据、冲突、假设、风险、完备度和版本。
- 聊天记录不是需求事实的权威来源。
- 批准后的需求包不可原地修改。
- 需求批准后才进入可编辑 DAG 规划；本功能不创建执行任务，不修改项目。

## 2. 目标与非目标

### 2.1 产品目标

1. 用户可以从一句话、规格文档、图片或参考项目开始，不必先填写固定表单。
2. 总控 Agent 不重复询问已经明确的信息。
3. 每个关键需求都能追溯到用户回答、项目事实或外部证据。
4. 系统能够解释还缺少什么、为什么要问，以及何时可以结束访谈。
5. 新项目和现有项目使用统一需求账本，同时保留不同的上下文采集流程。
6. 长访谈可以自动保存、恢复和创建分支。
7. 批准后的需求变化必须生成新版本和影响分析。
8. 项目扫描、附件解析和模型调用不得破坏前台交互流畅度。
9. 用户在模型请求前可以核对实际出站上下文和敏感信息扫描结果。

### 2.2 首版非目标

本功能首版不负责：

- 生成或执行 DAG。
- 修改项目文件、安装依赖或运行有副作用命令。
- 自动合并访谈分支中的冲突需求。
- 多人同时编辑需求包。
- 云端同步或跨设备恢复。
- 自动发布需求包到外部平台。
- 替代后续架构 Agent 的技术方案设计。
- 在用户未操作时自动展开右侧面板。

## 3. 用户与使用环境

### 3.1 核心用户

首版面向单人本地使用的软件开发者。用户管理自己的 Agent 团队，并在本地接入新项目或现有项目。

### 3.2 性能基线

验收设备基线：

- Windows 11
- 16 核及以上 CPU
- 32 GB 内存
- NVMe SSD
- 最大目标仓库：30 万文件、10 GB

后台任务遵循「前台流畅优先」策略。用户输入、滚动、切换栏位和发送请求的优先级高于扫描与索引吞吐量。

## 4. 术语

| 术语 | 定义 |
|---|---|
| Intake Session | 一次任务接入与需求澄清会话。 |
| Requirement Ledger | 结构化需求账本，保存需求事实与治理状态。 |
| Evidence | 用户回答、项目文件、运行现场或外部资料形成的可追溯证据。 |
| Project Profile | 现有项目的结构、技术栈、规则、Git 状态与安全画像。 |
| Requirement Package | 由账本编译出的可审阅需求包。 |
| Blocking Conflict | 影响范围、安全、架构或验收，必须在批准前解决的冲突。 |
| Non-blocking Conflict | 可延迟验证、不阻止当前规格批准的冲突。 |
| Completeness Gate | 生成需求包前必须满足的最低完备门槛。 |
| Interview Branch | 从某个历史问题创建的独立需求探索分支。 |
| Context Selection | 本次模型请求实际包含、固定、排除或裁剪的上下文清单。 |

## 5. 信息架构与窗口布局

### 5.1 Tauri 内置标题栏

桌面端隐藏系统外层标题框，由应用绘制 Windows 风格内置标题栏：

- 左侧显示 AstraWork Logo 与产品名。
- 中部是可拖拽区域，显示当前项目与会话路径。
- 右侧提供最小化、最大化/还原和关闭按钮。
- 双击可拖拽区域切换最大化/还原。
- 交互控件必须排除在拖拽区域之外。
- 窗口边缘继续使用原生缩放行为。
- 关闭按钮悬停使用 Windows 红色反馈。

### 5.2 左侧导航

左侧固定四个一级入口，顺序如下：

1. 新建任务
2. 自动化任务
3. 协作中心
4. 生态市场

下方区域保持为：

- 项目
- 对话
- 底部空间与设置入口

点击品牌 Logo 收起或展开左栏。左栏完全收起后，窗口左上保留小型 Logo 恢复按钮。系统按工作区保存左栏开关和展开宽度。

### 5.3 中间对话区

中栏是主要工作区，包含：

- 会话标题与总控 Agent 状态。
- 自然对话消息流。
- 单选、多选、自由文本、附件与「暂缓/未知」回答控件。
- 需求完备度摘要。
- 增强版 Composer。

中栏视觉参考 frakio-work：浅灰桌面外壳、白色圆角工作区、低对比边框、紧凑图标和悬浮式 Composer。

### 5.4 右侧面板

右栏默认收起。控制按钮始终位于对话栏右上方。

允许展开右栏的用户操作：

- 点击右栏按钮。
- 点击消息中的文件、证据、需求章节、任务、Diff 或其他右栏承载对象。

系统事件只能为按钮增加徽标或高亮，不得自动展开右栏。

右栏展开时占用布局宽度，中栏同步缩放；不覆盖中栏。右栏支持以下页签：

- 上下文
- 需求包
- 证据
- 历史

右栏状态遵循以下优先级：

1. 首次安装、新工作区、新建 Intake Session、切换会话和应用重启后，右栏一律收起。
2. 系统只保存右栏上次页签和展开宽度，不跨会话或重启恢复「已展开」状态。
3. 系统事件不得把 `rightPanelOpen` 从 `false` 改为 `true`。
4. 点击引用对象属于显式用户操作，可以展开右栏、切换到目标页签，并定位到对应对象。
5. 引用入口必须传递稳定的 `(tabId, objectId)`；右栏根据对象类型渲染对应内容，把目标滚动到可见区域、移动键盘焦点并短暂高亮。目标缺失时保留面板并显示可恢复错误，不得静默聚焦错误对象。
6. 重复点击已显示对象必须保持幂等，不重复创建面板、重复写入历史或累积高亮计时器。
7. 右栏控制器使用原生按钮并维护 `aria-expanded`、`aria-controls`；页签使用 `tablist`、`tab`、`tabpanel`、`aria-selected`。页签支持 Left/Right、Home、End，Escape 收起右栏并把焦点返回实际打开面板的引用控件；原控件已不可用时回退到右栏按钮。收起区域必须从键盘顺序和辅助技术树中移除。

Tauri 窗口最小尺寸为 1100 × 680 px。首版不提供低于 1100 px 的窄屏布局，也不允许右栏覆盖中栏；窗口达到最小宽度后继续缩放必须由系统阻止。三栏、左栏收起和右栏展开状态均需在最小尺寸下通过布局测试。

## 6. Composer 设计

### 6.1 输入与附件

Composer 支持：

- 自由文本。
- 图片、文件、目录和代码范围。
- 拖放文件。
- 粘贴截图、图片、文本或文件路径。
- Windows 资源管理器「发送到 AstraWork」。
- 从文件、浏览器、Review、任务和产物面板直接引用。

附件先在本地识别、解析、预览和索引。模型只接收与当前问题相关的片段，原始文件保留本机。

### 6.2 第一层常用控制

第一层始终显示：

- 模型选择。
- 供应商原生思考参数。
- 上下文占用与管理入口。
- 添加图片。
- 添加文件。
- 添加代码或目录。
- `@` 统一引用。
- `/` 命令菜单。
- 发送按钮。

模型切换默认只覆盖当前消息。发送后恢复当前 Agent 的默认模型。

思考控制沿用供应商原生语义，例如 reasoning effort 或 thinking budget。模型不支持该能力时，控件禁用并解释原因。

### 6.3 第二层运行设置

第二层默认折叠为四个状态胶囊：

- 工作模式
- 权限模式
- 运行目标
- 预算与用量

点击后展开完整设置。

以下变化必须突出显示，并要求用户在发送前确认：

- 权限升级。
- 运行目标变化。
- 工作模式开始产生副作用。
- 达到预算硬上限。

#### 工作模式

模式按当前上下文自适应显示：

- 需求访谈
- 普通对话
- 规划
- 执行
- 审查

不合法的模式必须禁用并显示原因。

#### 权限模式

采用四档风险模式：

1. 只读
2. 受控写入
3. 项目内自动
4. 完全访问

删除、外部发布、付费和授权属于硬边界操作，不得由普通档位自动放行。

本功能的访谈和项目画像阶段固定使用只读模式。

#### 运行目标

按层展示：

- 项目
- 分支或 Git worktree
- 执行环境
- 终端

默认继承会话状态。任何变化必须在发送前醒目标记。

#### 预算与用量

显示：

- 本次预计 Token。
- 本次预计费用。
- 本次预计耗时。
- 当前任务累计费用。
- 软上限。
- 硬上限。

达到软上限时提醒；达到硬上限时暂停并请求用户决策。

### 6.4 `@` 统一引用

`@` 菜单按类型分组搜索：

- 专业 Agent
- 文件、目录与代码符号
- 历史会话、需求章节、DAG 节点、运行结果与产物
- MCP、浏览器页面、知识库与项目画像

`@Agent` 默认是咨询，不移交主响应权。被咨询 Agent 返回结构化意见给总控 Agent，由总控汇总并面向用户回答。

只有明确使用移交命令时，指定 Agent 才成为本次主响应者。

### 6.5 `/` 命令

命令类别包括：

- 任务与模式
- 上下文与附件
- Agent 与模型
- 工具与工作流

命令来源包括系统内置命令和扩展注册命令。扩展命令必须声明：

- 命令名称与命名空间
- 来源
- 参数 Schema
- 所需权限
- 是否产生副作用
- 可用模式
- 结果类型

未知、声明不完整或权限不足的命令不得执行。

## 7. 新项目流程

### 7.1 多入口自适应采集

新项目允许从以下任意组合开始：

- 一句话想法
- 详细描述
- PRD 或规格文档
- 图片与界面截图
- 外部链接
- 参考项目

总控先提取已有信息并写入候选账本，再针对缺口提问。

### 7.2 访谈规则

- 每轮只提出一个关键问题。
- 优先使用带推荐项和影响说明的选择题。
- 允许自由文本、附件、代码选区和运行现场作为补充回答。
- 用户可以回答「暂缓」或「未知」。系统必须记录最晚决策节点。
- 不得重复询问已由高可信证据回答的问题。
- 次要未知项使用显式假设，不得暗中补齐。

### 7.3 最低完备门槛

以下 6 项必须明确后，系统才允许生成第一版需求包：

1. 目标用户
2. 核心问题
3. 首版范围
4. 关键流程
5. 约束
6. 可验证验收标准

该门槛由确定性服务检查。LLM 可以解释缺口，但不能覆盖检查结果。

每项门槛的机器判定必须满足：

1. 至少存在 1 条 `confirmed`、未删除、未被替代且关联有效 Evidence 的对应 `RequirementItem`。
2. 必填门槛不得仅由 `Assumption` 或模型推断满足。
3. 目标用户必须包含用户类型和主要使用场景。
4. 核心问题必须包含当前痛点和期望结果。
5. 首版范围必须同时包含范围内与明确不做事项。
6. 关键流程至少包含入口、主要成功路径和失败/退出路径。
7. 约束至少覆盖平台、数据/隐私和已知技术或业务限制；不适用项需有用户确认的 Decision。
8. 每条验收标准必须包含触发条件、可观察结果和判定方式。
9. 不存在影响该门槛的开放阻塞冲突。
10. 已到最晚决策节点的关键未知项必须先解决。

Completeness Evaluator 返回 `passed`、`evaluatedRevision`、`failedRules[]`、对应 Requirement ID 和 Evidence ID。账本 Revision 变化后，旧结果立即失效。

### 7.4 假设与风险

非关键未知项可以形成显式假设。每条假设必须包含：

- 假设内容
- 依据
- 潜在影响
- 验证方式
- 最晚决策节点
- 当前状态

## 8. 现有项目流程

### 8.1 第一阶段扫描

用户选择本地目录并授予只读扫描权限。第一阶段必须覆盖：

- 结构与技术栈
- 规范与文档
- Git 与工作区
- 安全与敏感项

隔离扫描器可以流式识别密钥和证书的类型与位置，但不得持久化、索引、记录、展示或发送密钥值。

### 8.2 项目画像确认

扫描完成后，总控展示：

- 技术栈
- 架构与模块边界
- 常用命令
- 项目规则
- Git 状态
- 风险
- 扫描范围与盲区

用户确认或修正后，项目画像才成为后续 Agent 的共享项目基线。用户修正作为高可信项目级事实保存。

### 8.3 第二阶段定向读取

总控围绕当前任务定向读取：

- 相关代码与符号
- 测试
- Git 历史与 Diff
- 运行日志和失败现场
- 外部文档与参考资料

不得为单个任务无条件重新读取整个仓库。

### 8.4 增量更新

使用文件事件与 Git 变化检测更新画像和索引：

- 普通代码变化静默增量索引。
- 技术栈、依赖、项目规则、构建命令和仓库边界变化触发用户复核。
- 项目目录移动或权限变化时，暂停增量更新并提示重新授权。

## 9. 冲突处理

证据来源可能包括：

- 用户当前指令
- 用户历史决策
- 项目代码
- 项目文档
- 运行现场
- 外部资料

冲突分为：

### 9.1 阻塞冲突

影响范围、安全、架构或验收。系统必须展示双方证据并请求用户决定。未解决前不得批准需求包。

### 9.2 非阻塞冲突

不影响当前范围或可以在后续节点验证。系统将其记录为风险，并绑定验证方式与最晚决策节点。

系统不得静默选择任一冲突来源。

## 10. 需求包与审阅

需求包必须包含：

1. 产品需求规格
2. 交互与页面说明
3. 技术约束摘要
4. 验收与待决清单

用户按章节执行：

- 评论
- 改写
- 接受
- 退回

所有阻塞项解决后，用户执行整体批准。

需求包批准后：

- 版本内容不可变。
- 保存批准人、时间、账本 Revision 与内容哈希。
- 后续变更创建新版本。
- 新版本必须生成对范围、DAG、预算、代码和验收的影响分析。
- 需求批准后才允许进入 DAG 草案生成。

## 11. 自动保存、恢复与分支

### 11.1 自动保存

以下操作必须本地持久化：

- 用户原始回答
- 结构化解析结果
- 证据
- 假设与风险
- 冲突与决策
- 当前问题
- 上下文选择
- 需求包草案
- UI 栏位状态

用户点击发送后，系统必须先将原始回答、附件引用、当前分支和输入账本 Revision 写入本地 durable journal，并完成 SQLite WAL 提交。只有收到 durable commit acknowledgement 后，UI 才显示「已提交」并启动模型请求。

本地保存不得等待模型请求完成。结构化解析可以异步执行，但解析结果必须引用原始回答 ID 和输入账本 Revision；Revision 已变化时，结果进入待复核队列，不得直接覆盖新状态。第 18 节的 100 ms 指标以 durable commit acknowledgement 为终点，不以内存入队为终点。

### 11.2 恢复

重启应用后，用户应恢复到：

- 原访谈分支
- 最近已提交回答
- 当前未回答问题
- 需求账本 Revision
- Composer 草稿与附件引用

### 11.3 访谈分支

用户可以从任意历史问题创建分支。分支必须保存父分支和分叉 Revision。

分支合并采用逐条需求差异审阅，不自动覆盖用户决策。

## 12. 内部组件

### 12.1 Intake Session

负责访谈生命周期、入口类型、当前分支、恢复点和状态转换。

### 12.2 Evidence Ingestion

负责统一接收、解析、去重、索引与标记证据。

### 12.3 Project Profiler

负责分阶段扫描、项目画像和增量更新。

### 12.4 Requirement Ledger

负责需求事实、来源、状态、作用域、置信度和版本。它是需求事实的权威来源。

### 12.5 Interview Policy

负责识别账本缺口并选择下一问题，执行一次一个关键问题的约束。

### 12.6 Conflict Service

负责冲突检测、分类、展示和解决记录。

### 12.7 Completeness Evaluator

负责确定性检查最低完备门槛。

### 12.8 Requirement Package Compiler

负责从账本编译需求包，并保留反向来源映射。

### 12.9 Revision & Impact Service

负责批准版本、不可变修订和影响分析。

## 13. 核心数据模型

### 13.1 `IntakeSession`

| 字段 | 类型 | 说明 |
|---|---|---|
| id | UUID | 会话 ID。 |
| type | enum | `new_project` 或 `existing_project`。 |
| projectId | UUID? | 现有项目 ID。 |
| activeBranchId | UUID | 当前访谈分支。 |
| status | enum | 当前生命周期状态。 |
| pausedFromStatus | enum? | 进入 `paused` 前的状态。 |
| conflictReturnStatus | enum? | 进入冲突处理前的返回状态。 |
| currentQuestionId | UUID? | 当前待回答问题。 |
| activePackageRevisionId | UUID? | 当前审阅中的需求包版本。 |
| ledgerRevision | integer | 当前账本 Revision。 |
| lastDurableJournalOffset | integer | 最近完成持久提交的 Journal Offset。 |
| createdAt | datetime | 创建时间。 |
| updatedAt | datetime | 更新时间。 |

### 13.2 `Evidence`

| 字段 | 类型 | 说明 |
|---|---|---|
| id | UUID | 证据 ID。 |
| sessionId | UUID | 所属 Intake Session。 |
| projectId | UUID? | 所属项目。 |
| branchId | UUID | 所属访谈分支。 |
| kind | enum | 用户回答、文件、代码、运行现场、网页或项目画像。 |
| sourceRef | string | 原始来源引用。 |
| contentHash | string | 内容哈希。 |
| sensitivity | enum | 普通、敏感或禁止出站。 |
| parseStatus | enum | 解析状态。 |
| metadata | JSON | 页码、行号、符号、URL 等。 |
| createdAt | datetime | 创建时间。 |

### 13.3 `RequirementItem`

| 字段 | 类型 | 说明 |
|---|---|---|
| id | UUID | 需求项 ID。 |
| sessionId | UUID | 所属 Intake Session。 |
| branchId | UUID | 所属访谈分支；跨分支合并通过新 Revision 和 Decision 表达，不迁移原记录。 |
| category | enum | 用户、问题、范围、流程、约束、验收或非功能需求。 |
| statement | string | 需求陈述。 |
| status | enum | 候选、已确认、被否决、已替代或已删除。 |
| confidence | number | 0–1。 |
| evidenceIds | UUID[] | 来源证据。 |
| scope | enum | 用户、项目或当前任务。 |
| introducedRevision | integer | 引入 Revision。 |
| supersededBy | UUID? | 替代项。 |
| preDeleteStatus | enum? | 逻辑删除前状态，仅在 `status=deleted` 时存在。 |
| deletedAt | datetime? | 逻辑删除时间。 |
| deletionDecisionId | UUID? | 删除原因和操作者对应的 Decision。 |
| restoredAt | datetime? | 最近一次恢复时间。 |
| restorationDecisionId | UUID? | 最近一次恢复对应的 Decision。 |
| restoredFromRequirementItemId | UUID? | 从锁定、已合并或已放弃分支复制恢复时引用原需求项。 |
| createdAt | datetime | 创建时间。 |
| updatedAt | datetime | 最近状态或内容变更时间。 |

### 13.4 `InterviewBranch`

| 字段 | 类型 | 说明 |
|---|---|---|
| id | UUID | 分支 ID。 |
| sessionId | UUID | 所属 Intake Session。 |
| name | string | 用户可见分支名称。 |
| normalizedName | string | 用于唯一约束的规范化名称。 |
| parentBranchId | UUID? | 父分支。 |
| forkedFromRevision | integer | 分叉时账本 Revision。 |
| headRevision | integer | 当前头 Revision。 |
| status | enum | `active`、`merged`、`abandoned`。 |
| creationDecisionId | UUID | 创建或分叉分支的 Decision。 |
| mergeDecisionIds | UUID[] | 合并时用户确认的逐条决策。 |
| abandonmentDecisionId | UUID? | 放弃分支的用户 Decision。 |
| createdAt | datetime | 创建时间。 |
| updatedAt | datetime | 最近变更时间。 |
| mergedAt | datetime? | 合并完成时间。 |
| abandonedAt | datetime? | 放弃时间。 |

`normalizedName` 使用应用锁定的 Unicode 版本和与 locale 无关的 `NFKC_Casefold` 生成：先移除首尾 Unicode `White_Space`，再执行 Unicode Default Case Folding 与 NFKC 规范化，并再次执行 NFKC；该算法版本随数据库 Schema Version 保存。数据库必须建立 `(sessionId, normalizedName)` 唯一约束。同一 Session 内分支名称唯一。测试向量至少覆盖 `ß/SS`、`Σ/σ/ς`、`I/İ/ı/i`、全角字符和组合字符。已合并或放弃的分支不可继续写入。分支创建、合并或放弃状态与对应 Decision、时间戳必须在同一事务写入。

### 13.5 `Question` 与 `Answer`

`Question` 必须包含 `id`、`sessionId`、`branchId`、`prompt`、`type`、`options`、`targetGapRules`、`selectedReason`、`ledgerRevision`、`status` 和时间戳。

`Answer` 必须包含 `id`、`questionId`、`rawContent`、`attachmentEvidenceIds`、`deferState`、`latestDecisionNode`、`inputRevision`、`durableCommittedAt` 和时间戳。`rawContent` 在 durable commit 后不可修改；更正通过新 Answer 和 Decision 表达。

### 13.6 `Conflict` 与 `Decision`

`Conflict` 必须包含 `id`、`sessionId`、`branchId`、双方 Requirement/Evidence 引用、`severity`、`affectedGateRules`、`status`、`deferUntil`、`resolutionDecisionId` 和时间戳。

`Decision` 必须包含 `id`、`sessionId`、`branchId`、`actor`、`kind`、`selectedRequirementIds`、`rejectedRequirementIds`、`rationale`、`evidenceIds`、`ledgerRevision` 和时间戳。分支创建、合并与放弃 Decision 均通过 `sessionId` 和 `branchId` 关联目标分支。

### 13.7 `RequirementPackageRevision`

| 字段 | 类型 | 说明 |
|---|---|---|
| id | UUID | 需求包版本 ID。 |
| sessionId | UUID | 所属会话。 |
| branchId | UUID | 来源分支。 |
| revisionRequestId | UUID? | 批准后修订对应的 Revision Request；首版为空。 |
| baseApprovedPackageRevisionId | UUID? | 批准后修订所基于的批准版本；首版为空。 |
| impactAnalysisId | UUID? | 对批准后修订生成的影响分析；首版为空。 |
| version | integer | 单调递增版本号。 |
| ledgerRevision | integer | 编译使用的账本 Revision。 |
| sourceSnapshotHash | string | 来源快照哈希。 |
| contentHash | string | 规范化内容哈希。 |
| status | enum | `draft`、`reviewing`、`stale`、`approved`、`superseded`。 |
| sectionReviewState | JSON | 各章节评论、接受和退回状态。 |
| createdAt | datetime | 创建时间。 |

同一 Session 的 `version` 唯一；`approved` 内容不可更新。Package Version 只在编译成功事务中分配，失败、取消、过期的编译 Attempt 不创建 Revision，也不消耗版本号。

### 13.8 `PackageCompileAttempt`

必须包含 `id`、`sessionId`、`branchId`、`inputLedgerRevision`、`inputSnapshotHash`、`revisionRequestId`、章节反馈引用、`status`（`queued`、`running`、`succeeded`、`failed`、`stale`、`cancelled`）、结构化错误、`outputPackageRevisionId` 和时间戳。Attempt 的成功提交必须在单一事务中重新校验输入 Revision 与快照哈希仍为当前值，然后分配下一个 Package Version 并插入完整 `RequirementPackageRevision`；迟到或陈旧结果只把 Attempt 标记为 `stale`。

### 13.9 `RequirementRevisionRequest` 与 `RevisionImpactAnalysis`

`RequirementRevisionRequest` 必须包含 `id`、`sessionId`、`branchId`、`baseApprovedPackageRevisionId`、`currentImpactAnalysisId`、`requestedAt`、`requestedBy`、`status`（`interviewing`、`package_ready`、`impact_pending`、`impact_running`、`impact_failed`、`impact_completed`、`approved`、`cancelled`）和时间戳。

每条 `RevisionImpactAnalysis` 本身代表一次可审计 Attempt，必须包含 `id`、`revisionRequestId`、`attemptNumber`、`baseApprovedPackageRevisionId`、`targetPackageRevisionId`、`inputLedgerRevision`、`baseContentHash`、`targetContentHash`、范围影响、DAG 影响、预算影响、代码影响、验收影响、`status`（`pending`、`running`、`completed`、`failed`、`stale`、`cancelled`）、结构化错误和时间戳。同一 Revision Request 的 `attemptNumber` 唯一；任何时刻最多一个 `pending` 或 `running` Attempt。

创建或重试影响分析时插入新的 `RevisionImpactAnalysis`，并原子更新 `RequirementRevisionRequest.currentImpactAnalysisId` 与 `RequirementPackageRevision.impactAnalysisId`；旧失败、过期或取消记录保持不可变。批准守卫只读取两个当前指针共同指向的分析。并发重试通过 Revision Request 行锁和活动状态唯一约束串行化。

批准后修订的 Package 进入整体批准前，当前影响分析必须为 `completed`，且输入 Ledger Revision、基准哈希和目标哈希均与当前值匹配。任一输入变化立即把旧分析标记为 `stale`、清空两个当前指针并重新生成。新版本批准事务同时把 Revision Request 置为 `approved`，并将此前批准版本置为 `superseded`；旧版本内容与 Approval Record 保持不可变。

### 13.10 `ContextSelection`

必须包含 `id`、`sessionId`、`messageId`、供应商、模型、原生思考参数、上下文编译器版本、条目列表、包含/固定/排除/裁剪状态、Token 估算、敏感扫描版本、`previewHash`、`requestHash` 和时间戳。

`previewHash` 与最终 `requestHash` 不一致时，请求不得发送。

### 13.11 `ApprovalRecord`

必须包含 `id`、`packageRevisionId`、`actorId`、`ledgerRevision`、`packageContentHash`、开放阻塞冲突计数、`approvedAt` 和审计签名。

批准操作必须在单一数据库事务中重新验证：Package Hash 未变化、Ledger Revision 匹配、开放阻塞冲突为 0、完备门槛通过；若为批准后修订，还必须验证匹配当前输入的影响分析已完成。验证和写入之间不得释放事务锁。

### 13.12 `ProjectProfile`、`Assumption`、`Risk` 与 `AcceptanceCriterion`

这些实体必须包含稳定 ID、所属 Session/Project/Branch、来源 Evidence、状态、引入 Revision、更新时间和删除标记。

`AcceptanceCriterion` 额外包含触发条件、可观察结果和判定方式。`Assumption` 与 `Risk` 额外包含影响、验证方式和最晚决策节点。

所有实体必须使用稳定 ID、时间戳和来源引用。删除采用可恢复逻辑删除；批准版本不可变。`IntakeSession` 的 `currentQuestionId`、`pausedFromStatus`、`conflictReturnStatus`、`activePackageRevisionId` 和 `lastDurableJournalOffset` 用于精确恢复。

## 14. 状态机

### 14.1 Intake Session 转移表

| 来源状态 | 事件与守卫条件 | 目标状态 | 原子副作用 |
|---|---|---|---|
| `created` | 新项目开始 | `collecting_context` | 创建主分支和空账本。 |
| `created` | 现有项目开始 | `collecting_context` | 创建主分支、账本和 Project Profile 草案。 |
| `collecting_context` | 新项目初始证据 durable commit 完成 | `interviewing` | 生成首个缺口评估。 |
| `collecting_context` | 现有项目目录授权与初始证据 durable commit 完成 | `profiling_project` | 创建扫描作业。 |
| `profiling_project` | 首份可操作画像生成 | `waiting_profile_review` | 固化 Profile 草案 Revision。 |
| `waiting_profile_review` | 用户确认或修正画像 | `interviewing` | 写入已确认 Profile Revision。 |
| `interviewing` | 存在开放阻塞冲突 | `waiting_conflict_resolution` | 保存冲突集合和 `conflictReturnStatus=interviewing`。 |
| `waiting_conflict_resolution` | 阻塞冲突全部解决 | `conflictReturnStatus` | 清空 `conflictReturnStatus` 并重新运行完备评估。 |
| `interviewing` | 完备门槛通过 | `ready_for_package` | 保存通过的规则结果和账本 Revision；批准后修订同时把 Revision Request 置为 `package_ready`。 |
| `ready_for_package` | 用户请求生成需求包，且 Revision 未变化 | `compiling_package` | 仅创建 `PackageCompileAttempt`，不预分配 Package Version。 |
| `compiling_package` | 首版编译成功，且输入 Revision/快照仍匹配 | `reviewing_package` | 原子分配下一个版本号、写入完整 Draft Package Revision、设置 `activePackageRevisionId` 并把 Attempt 置为 `succeeded`。 |
| `compiling_package` | 批准后修订编译成功，且输入 Revision/快照仍匹配 | `analyzing_revision_impact` | 原子分配下一个版本号、写入完整 Draft、设置 `activePackageRevisionId`、创建 `pending` 影响分析并更新两个当前指针、把 Revision Request 置为 `impact_pending`，再通过 Outbox 启动作业。 |
| `compiling_package` | 编译失败 | `package_compile_failed` | 把 Attempt 置为 `failed` 并保存结构化错误；不创建 Package Revision、不消耗版本号。 |
| `package_compile_failed` | 用户重试且来源 Revision 仍有效 | `compiling_package` | 创建新的编译 Attempt。 |
| `analyzing_revision_impact` | Worker 领取当前 `pending` 分析 | `analyzing_revision_impact` | 原子把分析置为 `running`，并把 Revision Request 置为 `impact_running`。 |
| `analyzing_revision_impact` | 影响分析完成，且全部输入与当前指针仍匹配 | `reviewing_package` | 原子保存五类影响结果、把当前分析置为 `completed`、把 Revision Request 置为 `impact_completed`；`activePackageRevisionId` 继续指向待审 Draft。 |
| `analyzing_revision_impact` | 影响分析失败 | `revision_impact_failed` | 把当前分析置为 `failed`，保存结构化错误并把 Revision Request 置为 `impact_failed`。 |
| `revision_impact_failed` | 用户重试且全部输入仍匹配 | `analyzing_revision_impact` | 插入新的 `pending` 分析，原子切换两个当前指针，把 Revision Request 置为 `impact_pending` 并通过 Outbox 启动作业。 |
| `reviewing_package` | 用户退回章节且只需文档重编译 | `compiling_package` | 保存章节反馈、把当前 Draft 与分析置为 `stale`、清空 `activePackageRevisionId` 和两个分析指针；批准后修订把 Request 置为 `package_ready`。创建新的 Compile Attempt，仅在成功后创建下一 Draft。 |
| `reviewing_package` | 用户退回章节且改变需求事实 | `interviewing` | 写入反馈、把当前 Draft 与分析置为 `stale`、清空活动指针、把 Revision Request 置为 `interviewing` 并重新评估缺口。 |
| `reviewing_package` | 用户批准，且事务守卫全部通过 | `approved` | 原子写入 Approval Record、锁定 Package Revision 并清空 `activePackageRevisionId`；批准后修订同时把 Request 置为 `approved`，并将旧批准版本置为 `superseded`。 |
| `approved` | 用户提出需求变更 | `interviewing` | 在同一 Session 创建新 Interview Branch、新 Ledger Revision 和状态为 `interviewing` 的 `RequirementRevisionRequest`；旧批准版本保持不可变，下一 Package Version 仅在后续编译成功时分配。 |
| `ready_for_package`、`compiling_package`、`package_compile_failed`、`reviewing_package`、`analyzing_revision_impact` 或 `revision_impact_failed` | Ledger Revision 或来源快照变化 | `interviewing` | 使完备结果、活动 Draft、当前分析和未完成 Attempt 失效；迟到结果不得提交；清空 `activePackageRevisionId` 与两个分析指针，把 Revision Request 置为 `interviewing` 并重新评估缺口。 |
| `compiling_package` | 用户暂停 | `paused` | 取消当前 Compile Attempt 与未投递 Outbox，保存 `pausedFromStatus=compiling_package`；不得覆盖 `conflictReturnStatus`。 |
| `analyzing_revision_impact` | 用户暂停 | `paused` | 把当前分析置为 `cancelled`、取消未投递 Outbox，清空两个分析指针并把 Request 置为 `impact_pending`；保存 `pausedFromStatus=analyzing_revision_impact`。 |
| 除 `paused`、`approved`、`cancelled`、`compiling_package`、`analyzing_revision_impact` 外的状态 | 用户暂停 | `paused` | 把来源状态写入 `pausedFromStatus`，不得覆盖 `conflictReturnStatus`；数据库约束禁止 `pausedFromStatus=paused`。 |
| `paused` | 用户重复暂停或暂停命令重放 | `paused` | 幂等成功，不修改 `pausedFromStatus`、`conflictReturnStatus` 或其他业务数据。 |
| `paused` | 用户恢复，且来源为 `compiling_package` | `compiling_package` | 创建新的 Compile Attempt 后清空 `pausedFromStatus`。 |
| `paused` | 用户恢复，且来源为 `analyzing_revision_impact` | `analyzing_revision_impact` | 插入新的 `pending` 分析、原子更新两个指针并通过 Outbox 启动作业，然后清空 `pausedFromStatus`。 |
| `paused` | 用户恢复，且来源为其他合法状态 | `pausedFromStatus` | 恢复后只清空 `pausedFromStatus`。 |
| 任意非终态 | 用户取消 | `cancelled` | 原子取消活动 Compile Attempt 与影响分析、把当前非终态 Revision Request 置为 `cancelled`、清空 `activePackageRevisionId` 和两个分析指针，并取消未投递 Outbox；保留全部只读审计数据。 |

任何未在表中声明的转移返回 `INVALID_INTAKE_TRANSITION`，不得产生部分副作用。状态与原子副作用在同一事务提交；外部作业通过事务 Outbox 启动。

### 14.2 需求项状态

```text
candidate → confirmed → superseded
candidate → rejected
confirmed → superseded
任意非 approved-package 快照内状态 → deleted（逻辑删除）
deleted → preDeleteStatus（用户执行恢复且所属分支仍可写）
```

逻辑删除与恢复各自创建 Decision，并在同一事务中推进 Ledger Revision。删除时保存 `preDeleteStatus`、`deletedAt` 和 `deletionDecisionId`；恢复时清空删除字段，写入 `restoredAt` 与 `restorationDecisionId`，不得复用旧 Revision 或改写历史 Decision。若所属分支已合并、放弃或来源批准快照已锁定，恢复必须在同一 Session 的新可写分支中创建具有新稳定 ID 的等价 Requirement Item，并通过 `restoredFromRequirementItemId` 引用原项；恢复 Decision 同时引用原项与新项。该自引用不得跨 Session、指向自身或形成环。已批准 Package 引用的历史 Requirement Item 不做物理删除。

### 14.3 冲突状态

```text
open → acknowledged → resolved
open → deferred（仅非阻塞冲突）
deferred → open       （到达验证节点或证据变化）
deferred → resolved   （验证完成并形成 Decision）
acknowledged → open   （相关 Requirement/Evidence 变化）
```

阻塞冲突不得进入 `deferred`。所有冲突状态变化必须引用 Decision 或触发 Evidence。

## 15. 上下文与隐私

上下文面板必须显示：

- 来源清单
- 包含、固定、排除和裁剪状态
- 各来源 Token 占用
- 总窗口比例
- 预计输入费用
- 实际供应商
- 敏感信息扫描结果
- 出站内容预览

用户可以逐项：

- 排除
- 恢复
- 固定
- 脱敏
- 阻止发送

模型请求使用的上下文必须与预览一致。发送前系统比较 `previewHash` 与规范化最终请求的 `requestHash`；不一致则阻止发送并重新生成预览。请求日志记录两个哈希、编译器版本和一致性结果，不记录禁止出站的原文。

## 16. 安全要求

1. 访谈和项目画像固定为只读。
2. 只读阶段无法调用写文件、安装、提交、发布和桌面副作用工具。
3. 隔离的秘密扫描器可以流式检查文件内容，但密钥值不得持久化、索引、记录、渲染或出站；扫描结果只保存秘密类型、规范化位置和不可逆指纹。
4. 每次打开文件前必须执行 canonical path 与授权根目录边界检查；Windows junction、mount point、符号链接及其他 NTFS reparse point 默认不跟随。检查与打开使用安全句柄或文件身份复核，防止 TOCTOU 替换逃逸。
5. 压缩文件限制为：原始文件 ≤ 1 GB、解压总量 ≤ 4 GB、文件数 ≤ 20,000、嵌套深度 ≤ 3、单文件 ≤ 512 MB、压缩比 ≤ 100:1、处理时限 ≤ 120 秒。任何成员路径不得越出临时解压根目录；触线后停止解压并保留结构化错误。
6. 网页与文档内容视为不可信证据，不能改变系统权限或执行策略。
7. 被咨询 Agent 只获得当前授权的上下文子集。
8. 扩展命令必须经过注册、Schema 校验与权限评估。
9. 本地凭据存入 Windows Credential Manager，数据库只保存引用。
10. 导出审计包前再次执行敏感信息扫描。

### 16.1 外部 URL 获取策略

- 默认只允许 `https:`；`http:` 需要用户逐次确认。
- 禁止 `file:`、`javascript:`、`data:` 和自定义协议。
- 默认阻止 loopback、RFC 1918 私网、链路本地、组播、保留地址和云元数据地址。
- DNS 解析前后以及每次重定向都重新执行地址策略，防止 DNS rebinding。
- 最多跟随 5 次重定向；响应体上限 50 MB；单请求总时限 30 秒。
- 只处理显式允许的 MIME 类型；未知二进制作为附件候选，不直接解析。
- 不自动携带浏览器 Cookie、系统代理凭据、桌面登录状态或 Authorization Header。
- 用户显式授权访问本机或局域网 URL 时，授权只作用于该精确 Origin 和当前 Intake Session。

## 17. 异常与恢复

| 异常 | 处理 |
|---|---|
| 附件解析失败 | 保留原附件和错误原因；允许重试、切换解析器或选择片段。 |
| 项目扫描中断 | 保存游标并增量继续。 |
| 模型调用失败 | 相同供应商、模型、思考参数和请求负载的传输级重试可复用原预览。供应商、模型、思考参数、上下文内容或裁剪结果任一变化时，必须重新编译 Context Selection、重新扫描敏感信息并更新预览；跨供应商降级需要用户再次确认。不得丢失访谈状态。 |
| 回答解析不确定 | 保存用户原文，不写入确定事实；后续澄清。 |
| 需求包编译失败 | 保留账本，仅重试编译。 |
| 分支合并冲突 | 展示逐条差异，由用户决定。 |
| 数据库写入失败 | 阻止新批准操作；允许只读查看与恢复导出。 |
| 磁盘空间不足 | 暂停索引和附件复制；保留已提交回答。 |
| 项目目录移动 | 暂停扫描并请求重新定位与授权。 |

## 18. 量化性能指标

所有指标在性能基线设备上验收。基准协议必须版本化并记录：设备 CPU 型号、逻辑核数、内存、SSD 型号、Windows Build、电源模式、应用 Build、数据集哈希、文件类型分布、冷/热缓存状态、并发后台作业、样本数量和测量起止点。

首版 CI 性能设备固定为 16 核 / 32 线程 CPU、32 GB 内存、PCIe 4.0 NVMe SSD、Windows 11 高性能电源模式。仓库基准数据集固定版本和哈希，包含 30 万文件、10 GB 数据，覆盖 TypeScript/JavaScript、Python、Rust、Markdown、JSON、图片和二进制资源。

延迟指标至少执行 100 次并报告 P50/P95；冷启动执行 30 次且每次清理应用进程与页缓存条件。CPU 百分比按整机逻辑处理器总容量计算。「用户活跃」负载定义为持续输入、滚动长会话及每 2 秒执行一次文件/符号搜索。「60 FPS」要求测试区间内 P95 帧时间 ≤ 16.7 ms，超过 50 ms 的长帧比例 < 0.1%。

「首份可操作项目画像」必须至少包含技术栈、仓库根、模块顶层、项目规则、构建/测试命令候选、Git 状态、安全风险计数和扫描盲区；它是部分画像，UI 必须显示后台深度扫描仍在继续。

| 指标 | 目标 |
|---|---|
| 冷启动到可输入 | P95 ≤ 3 秒 |
| Composer 输入反馈 | P95 ≤ 50 ms |
| 长对话滚动 | 目标 60 FPS |
| 左右栏动画 | 180–220 ms，不等待索引 |
| 回答与附件状态本地保存 | P95 ≤ 100 ms |
| 首份可操作项目画像 | P95 ≤ 10 秒 |
| 30 万文件元数据扫描 | P95 ≤ 60 秒 |
| 首批相关模块索引 | ≤ 30 秒 |
| 全量代码符号与文本索引 | ≤ 8 分钟 |
| 单文件变化进入增量索引 | P95 ≤ 2 秒 |
| 文件与符号搜索 | P95 ≤ 200 ms |
| 常规上下文预览生成 | P95 ≤ 500 ms |
| 用户活跃时后台 CPU | ≤ 总 CPU 的 25% |
| 应用空闲时后台 CPU | 可提升到总 CPU 的 75% |
| 索引进程内存软上限 | 4 GB |
| 索引进程内存硬上限 | 6 GB |
| UI 进程常态内存 | ≤ 700 MB |

达到索引内存软上限时降低并发或分片落盘。达到硬上限时暂停新索引批次并释放缓存，不得挤压 UI 进程。

性能面板显示 P50、P95、峰值和最近退化原因。

## 19. 可观测性

系统记录：

- 每个问题的选择原因、账本缺口和使用证据。
- 每次模型请求的供应商、模型、原生思考参数、上下文清单哈希、Token、费用、耗时和结果状态。
- 项目扫描覆盖范围、跳过原因、敏感项计数和画像版本。
- 需求账本 Revision、批准记录和变更影响。
- 解析、扫描、索引和编译失败的结构化错误。

用户可以导出需求访谈审计包。导出包不得包含凭据、密钥值或被标记为禁止出站的原文。

## 20. 测试矩阵

### 20.1 单元测试

- 需求账本增删改、来源追溯、逻辑删除、同分支恢复、锁定分支复制恢复与不可变批准版本；复制恢复验证新 ID、同 Session、可写 Branch、`restoredFromRequirementItemId` 和禁止引用环。
- 分支名称按锁定 Unicode 版本执行 `White_Space trim → NFKC_Casefold → NFKC`，使用 `ß/SS`、`Σ/σ/ς`、`I/İ/ı/i`、全角字符和组合字符确定性向量验证 `(sessionId, normalizedName)` 唯一约束。
- 分支创建、合并和放弃时，状态、`creationDecisionId`、`mergeDecisionIds`、`abandonmentDecisionId`、`createdAt`、`updatedAt`、`mergedAt`、`abandonedAt` 原子写入；Decision 的 Session/Branch 归属匹配，已合并或放弃分支禁止继续写入。
- 最低完备门槛。
- 问题选择、重复抑制与最晚决策节点。
- 冲突分类。
- Package Compile Attempt 失败、取消或过期不创建 Revision、不消耗版本号；并发成功事务只分配一个唯一递增版本。
- Draft 创建、进入影响分析、进入审阅、章节退回、Ledger 失效和批准时，`activePackageRevisionId` 按状态机原子赋值、保持或清空。
- `RequirementRevisionRequest` 的 `interviewing`、`package_ready`、`impact_pending`、`impact_running`、`impact_failed`、`impact_completed`、`approved`、`cancelled` 状态均有可达路径，并与 Session 状态一致；取消事务同时终止活动 Attempt、清空活动指针并保留审计记录。
- 批准后修订的五类影响分析、当前指针、输入哈希失效、失败/并发重试、Attempt 唯一序号、第二版批准与旧批准版 `superseded` 状态。
- 模型临时覆盖、原生思考参数和预算上限。

### 20.2 契约测试

- 多供应商模型适配。
- `@` 统一引用。
- `/` 内置与扩展命令。
- 附件解析器。
- Project Profiler。
- 上下文编译器。
- Tauri 窗口与 Windows Credential Manager。

### 20.3 集成测试

- 一句话新项目到需求包批准。
- 文档、截图和参考项目导入。
- 现有项目扫描、画像确认和定向读取。
- 阻塞冲突解决。
- 冲突等待期间暂停、强制终止、重启恢复、继续处理并解决冲突，验证 `pausedFromStatus` 与 `conflictReturnStatus` 相互独立。
- 重复暂停和暂停命令重放保持幂等，禁止 `pausedFromStatus=paused`；从每个可暂停状态恢复到原状态。
- 编译或影响分析运行中暂停时，当前 Attempt 被取消且迟到结果被拒绝；恢复后创建新 Attempt，不重复分配 Package Version 或复用已取消分析。
- 访谈中断、恢复、分支、合并和修订。
- 需求包编译失败后恢复。
- 在生成前、编译中、编译失败后、审阅中和影响分析中修改 Ledger，验证旧完备结果、Draft、分析和 Attempt 失效，迟到结果被拒绝并返回访谈。
- 模型失败后的降级链。
- 大附件解析失败、取消和重试。

### 20.4 UI 与可访问性测试

- Logo 控制左栏展开与收起。
- 首次安装、新工作区、新 Session、切换会话和重启后右栏默认收起，且收起内容不进入键盘顺序或辅助技术树。
- 右栏只因用户点击按钮或文件、证据、需求章节等引用对象而展开；展开后切换到对应页签，渲染正确对象，将其滚动可见、聚焦并短暂高亮。
- 重复打开同一 `(tabId, objectId)` 不复制内容、不重复写历史，并取消旧高亮计时器后重新使用同一目标。
- 页签支持 Left/Right、Home、End；Escape 收起右栏并把焦点返回打开面板的原引用；原引用被移除、禁用、隐藏、位于 inert 区域或聚焦失败时回退到右栏按钮；ARIA 展开、选中和控件关联状态同步更新。
- 系统新增风险、冲突和结果时只更新徽标，不展开右栏。
- 左栏开关与宽度持久化；右栏只持久化页签与宽度，不持久化展开状态。
- 在 1100 × 680 px 最小窗口下，右栏按钮仍可展开面板且不覆盖中栏。
- Tauri 内置标题栏拖动、最大化、窗口按钮和缩放。
- Composer 全部控件。
- 第二层运行设置默认折叠。
- 键盘导航、焦点、屏幕阅读器、缩放和减少动态效果。

### 20.5 安全测试

- 密钥值隔离。
- 敏感目录、符号链接、junction、mount point、其他 reparse point、路径替换竞态与根目录逃逸。
- 压缩包原始大小、解压总量、文件数、单文件大小、压缩比、嵌套深度、处理时限和 Zip Slip 边界。
- 外部 URL 的 SSRF、DNS rebinding、重定向、私网/元数据地址、超大响应和凭据隔离。
- 恶意文档和提示注入。
- 扩展命令、MCP 与 Agent 越权。
- 出站预览与实际请求一致。
- 只读阶段副作用阻断。

### 20.6 恢复测试

在扫描、索引、模型请求、需求包编译和数据库写入期间强制终止进程，验证：

- durable commit acknowledgement 之前强杀时，UI 不得显示回答已提交；重启后恢复 Composer 草稿。
- durable commit acknowledgement 之后、模型请求之前或期间强杀时，用户原始回答不得丢失且不得重复提交。
- 当前分支、`pausedFromStatus`、`conflictReturnStatus`、当前问题与账本 Revision 可恢复；覆盖「冲突处理中暂停、重启、恢复、解决冲突」路径。
- 批准版本保持完整。
- 失败步骤可单独重试。

### 20.7 性能测试

使用 30 万文件、10 GB 仓库和长会话基准，验证第 18 节所有指标及连续运行内存稳定性。

## 21. 首版验收场景

使用以下 4 类真实场景完成端到端验收：

1. 新项目：从一句话、PRD、截图和参考链接的组合开始，不进入 Project Profile Review。
2. 现有中型 Web 单仓库。
3. 现有大型 Monorepo。
4. 现有混合语言、包含文档与二进制资源的桌面项目。

新项目场景必须覆盖：

- 多入口证据导入
- 最低完备门槛
- 「暂缓/未知」与最晚决策节点
- 阻塞冲突
- 需求包章节退回和重新编译
- 整体批准
- 应用重启恢复
- 访谈分支
- 批准后的需求修订与影响分析

3 个现有项目场景除以上适用流程外，还必须完成项目接入、项目画像确认和增量更新。

## 22. 与后续功能的边界

本功能向 DAG 计划编辑功能输出：

- 已批准 Requirement Package Revision
- 已确认 Project Profile Revision
- Requirement Ledger Snapshot
- 未阻塞的假设与风险
- 验收标准
- 上下文来源引用
- 用户批准记录

DAG 规划不得直接读取未批准的聊天摘要作为需求权威。若规划阶段发现新需求冲突，必须创建需求修订并返回本功能处理。

## 23. 实施拆分约束

本规格是一个产品功能规格，但实施必须拆成 6 个有独立验收门的实现计划：

1. Tauri Shell、内置标题栏、左侧导航、右栏和增强 Composer。
2. Intake 状态机、Requirement Ledger、durable journal、持久化与恢复。
3. 访谈策略、冲突服务、完备门槛、需求包审阅与不可变批准。
4. Evidence Ingestion、Context Selection、多供应商模型调用、`@` 与 `/`。
5. Project Profiler、文件/符号索引和增量更新。
6. 安全加固、性能基准、故障注入和端到端验收。

每个计划只能依赖前序计划公开的稳定契约，不得通过共享超大模块绕过边界。第 2、3 项完成前，不得把聊天摘要当作需求事实权威；第 4、5 项完成前，UI 可使用契约一致的测试替身。

## 24. 参考项目取舍

从 frakio-work 借鉴：

- 浅灰桌面外壳与白色圆角工作区。
- 三栏信息架构。
- 对话为主、上下文面板为辅。
- 紧凑悬浮 Composer。
- 多 Agent 自然协作与模型选择。

不照搬：

- 超大前端单文件。
- 两套任务执行后端。
- Space、Workspace、Project 概念混用。
- 缺少稳定路由。
- 仅依赖 LLM 决定需求事实与 Agent 路由。
- 右栏默认常驻占用中栏空间。

## 25. 已确认决策清单

- Windows 本地优先，单人使用。
- 新项目与现有项目都支持。
- 多入口自适应需求采集。
- 一次一个关键问题。
- 最低完备门槛。
- 显式假设与风险。
- 现有项目分阶段扫描并确认画像。
- 自动保存并支持访谈分支。
- 需求包章节审阅、整体批准。
- 批准后新建版本并执行影响分析。
- 需求批准后才生成 DAG。
- 自然对话 + 结构化需求账本。
- 模型选择仅覆盖当前消息。
- 供应商原生思考参数。
- `@Agent` 默认咨询。
- `/` 支持内置与扩展命令。
- 本地解析附件并按需注入。
- 第二层运行设置默认折叠。
- Tauri 使用应用内置 Windows 标题栏。
- 左栏一级入口为新建任务、自动化任务、协作中心、生态市场。
- 左栏由品牌 Logo 控制。
- 右栏默认收起，由对话栏右上角按钮或用户点击引用对象展开。
- 系统事件只提示，不自动展开右栏。
- 高性能工作站基线，前台流畅优先。
- 使用完整测试矩阵。
