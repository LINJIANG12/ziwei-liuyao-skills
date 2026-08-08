---
name: ziwei-agent
description: 紫微斗数排盘与时辰反推智能体。用户提到紫微、排盘、命盘、时辰、反推、定盘、生辰、八字、流年、大限、宫位、主星、四化等词时使用。可完成：真太阳时校准排盘、时辰未知时 15 候选反推定盘、正式解读、流年/流月/大限/小限核对。
compatibility: 需要 Node 18+；脚本零外部依赖（node scripts/ziwei-calc.mjs 直接运行）
---

# 紫微斗数智能体（ziwei-agent）

## 这是什么

紫微斗数排盘与时辰反推的完整智能体：**确定性计算由脚本完成**（真太阳时校准、排盘、15 候选、流年事实），**判断与解读由你完成**（按本 skill 的方法论与参考文档）。你不需要自己推算任何盘面数据——所有事实必须来自脚本输出。

## 何时使用

- 用户给出出生资料（性别/公历日期/时间/地点），要求排盘或看命盘
- 用户时辰未知，需要反推时辰（定盘）：通过人生经历核对 15 个候选时辰
- 用户要求看流年、流月、大限、小限应期

## 能力概览

1. **排盘**（`init` + `chart`）：真太阳时校准 → 完整命盘（十二宫/主星/四化/五行局/大限序列）
2. **反推时辰**（15 候选）：多轮对话核对用户经历 → 排除/锁定候选
3. **时间层事实**（`year-facts`/`year-comparison`）：大限/流年/流月/小限核对
4. **维度对比**（`dimension-compare`）：家庭/感情/学业/财富等宫位批量核对
5. **正式解读**：基于确认盘做十二宫解读

## 项目文件说明（先读这里，按需打开）

| 文件 | 是什么 | 什么时候读 |
|---|---|---|
| `references/01-chart.md` | 排盘原理：真太阳时校准、命盘结构（十二宫/主星/四化/五行局）、已知时辰起盘 | 起盘、核对盘面结构时 |
| `references/02-candidates.md` | 15 候选体系：23:00 切日、无早/晚子时、clockWindow 钟表窗口、候选键格式 | 时辰未知进入反推时（必读） |
| `references/03-inference-flow.md` | 反推完整流程：init → candidates → 逐事件核对 → record-finding → 排除/锁定 | 反推会话的每一轮（必读） |
| `references/04-discipline.md` | 判断纪律：多维度核对、证据分层、时段先确认再排除、强烈矛盾直接排除、排除即出局、锁定须确认；**读盘纪律（每轮读盘/解答前必读）**：断语三条件、吉凶加权、借星降权、应期映射（主题≠事件）、边界声明 | 每次做出核对/排除/锁定判断前，及每次读盘/解答选择题前（必读） |
| `references/05-time-layers.md` | 时间层取义与调用规则：大限/流年/流月/小限/流日流时、solarMonth 覆盖流月、连续年份一次取回 | 核对「某年某月发生某事」时 |
| `references/06-tool-protocol.md` | ziwei-calc.mjs 全部子命令的参数/输出格式 + 状态协议（含 `retrieve` 检索约定） | 需要计算时（对照参数） |
| `references/07-basic-knowledge.md` | 简易知识：十二宫主事、十天干四化表、大限/流年/小限规则、14 主星基础取义 | 简易模式解读兜底（完整知识库检索见「快速路径」） |

## 快速路径

- **第一件事**：知识库模式检测 + 向用户声明。直接调 `retrieve --query <主题词>`（如 `retrieve --query "命宫紫微贪狼"`），输出自带 `mode: complete|basic`：
  - `complete` → 完整知识库可用，解读取义一律通过 `retrieve` 检索，**不得直接翻阅 assets-private 知识库文件，不得凭记忆引用**
  - `basic`（或 `no_assets`）→ 简易模式，用 `references/07-basic-knowledge.md` 兜底，不得宣称使用了完整知识库
  - 知识库存放/变更/检索协议详见 `docs/knowledge-retrieval.md`
- **第二件事**：读 `references/02-candidates.md`（了解候选与切日）与 `references/04-discipline.md`（判断纪律 + 读盘纪律）
- **解答选择题/判断具体事件**（出身、样貌、学历、婚恋、某年事件等）：先读 `references/04-discipline.md` 的「读盘纪律（每轮读盘/解答前必读）」小节，再开始检索与推演——主题引动不得直接断言为事件，强断言须三条件齐备
- **用户要排盘（时辰已知）**：`node <技能绝对路径>/scripts/ziwei-calc.mjs init --gender <male|female> --date <YYYY-MM-DD> --time HH:MM` → 输出 `recommendedCandidateKey`（与钟表时间匹配的候选），直接 `chart --candidate-key <该key>` 一步取盘，无需手工挑候选
- **用户要排盘（时辰未知）**：`init --gender <male|female> --date <YYYY-MM-DD>` → 15 候选，按 `03-inference-flow.md` 流程反推
- **反推中每轮**：按 `03-inference-flow.md` 流程执行；需要计算时对照 `06-tool-protocol.md` 调脚本
- **解读**：先用 `retrieve --query` 检索相关取义规则（如星曜+宫位+四化组合词），再结合脚本返回的实盘事实；未命中时用 `references/07-basic-knowledge.md` 或如实说明，不编造规则

## 子代理评估工作流（多命例任务）

多个命例需要逐一判断时（如一批选择题/判断题），**不要**预生成工作簿或自己解析大规模 JSON。主代理直接派发子代理，每个子代理独立直调脚本，只读取数，输出判断即可。

1. **派发**：N 个命例 → N 个子代理，每个子代理分配唯一 `--session <命例名>`，互不干扰
2. **子代理固定三步（全部只读，不写任何文件）**：
   - `init --gender <male|female> --date <YYYY-MM-DD> --time HH:MM --timezone <地区> --longitude <经度> --latitude <纬度> --place <地点> --session <命例名>`
     → 输出自带 `trueSolar`（真太阳时校正分钟）、`recommendedCandidateKey`（已知钟表时间时的推荐候选键），不需要人工换算
   - `chart --candidate-key <key> --session <命例名>` → 取完整命盘
   - 题目涉及时点：`year-facts --candidate-key <key> --year <YYYY> --session <命例名> --summary` → 该年大限/流年/小限精简事实
3. **产出由任务决定**：本 skill 不预设固定答案格式。根据原问题的形式作答——选择题给出选项（可附一句星曜依据）、判断题为「结论 + 证据」、开放题为解读。只做判断，不调用 `exclude`/`lock`/`record-finding`/`reset`/`visualize` 等写入命令
4. **授权只读**：子代理不修改任务文件、不生成中间产物、不需要维护候选状态（脚本自动管理状态文件）

## 脚本调用说明

- 脚本路径用**绝对路径**或先 `cd` 到技能目录：`node /绝对/路径/skills/ziwei-agent/scripts/ziwei-calc.mjs <子命令> ...`。脚本内部通过自身位置解析状态目录（`<技能根>/.stars-state/`），与当前工作目录无关；但 shell 里写相对路径 `scripts/...` 时要求 cwd 是技能根目录
- **真太阳时**：默认启用（按出生地经度 + 均时差校准，输出含 `trueSolar` 标注校准量；`chart` 的 `trueSolarDateTime` 即校准后时间）。用户明确知道钟表时间无误时可加 `--no-true-solar` 关闭校准（按钟表标准时排盘）
- **时区解析**：显式 `--timezone` 优先；未指定时按出生地（`--place`）匹配城市库时区（含港澳台：香港 → Asia/Hong_Kong，1980 年后无夏令时）；未匹配到城市时默认 Asia/Shanghai 并在输出提示。注意：Asia/Shanghai 含中国 1986-1991 夏令时历史（夏季 +9），港澳台等非夏令时地区出生者请用 `--timezone Asia/Hong_Kong` 等显式指定，否则经度修正可能偏差约 60 分钟并改变时辰判定
- **会话残留**：同一 `--session` 重复 `init` 会覆盖旧反推状态，输出带 `previousState` 提示。多套独立反推用 `--session <名称>` 隔离，或先 `reset`
- **year-facts 精简**：输出过长或只核对流月时加 `--summary`（省略流年/小限十二宫星曜明细，流月不受影响）；流月输出带 `continuesFromPreviousMonth`/`continuesIntoNextMonth` 跨月标记——同一流月跨两个公历月会在两次展开中各出现一段，属同一流月，**不得重复计数**
- **排盘可视化**：`visualize [--no-open]` 生成十二宫盘面 HTML（命宫高亮、主星/四化/借星/大限、候选切换器）并**用本地浏览器打开**；页面与源数据存 `<技能根>/.stars-state/visuals/<session>-<时间戳>.html/.json`，**可追溯**。用户要求「看盘面/命盘图」时使用

## 命名对照（MCP 工具 ↔ 脚本子命令）

| MCP 工具名 | 脚本子命令 | 作用 |
|---|---|---|
| `get_ziwei_inference_candidates` | `candidates` | 候选状态列表 |
| `get_ziwei_inference_chart` | `chart` | 取候选完整命盘 |
| `get_ziwei_inference_year_facts` | `year-facts` | 大限/流年/流月/小限事实（`--summary` 精简） |
| `get_ziwei_inference_year_comparison` | `year-comparison` | 全部候选年度对比 |
| `get_ziwei_inference_dimension_comparison` | `dimension-compare` | 宫位维度对比 |
| `exclude_ziwei_inference_candidates` | `exclude` | 排除候选 |
| `lock_ziwei_inference_hour` | `lock` | 锁定建议 |
| `record_ziwei_inference_finding` | `record-finding` | 记录核对结论 |
| — | `init` | 建立出生资料并生成候选（MCP 由会话初始化完成） |
| — | `reset` | 清空会话状态 |

## 重要边界

- 所有盘面事实必须来自 `ziwei-calc.mjs` 输出，**不得自行推算**排盘/四化/流年
- 未取回的时间层不得下结论
- 锁定时辰必须用户确认；本 skill 只提出建议
