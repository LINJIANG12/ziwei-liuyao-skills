# 知识库存放规则与检索规则（RAG 协议）

本文档规范两套技能（紫微 `ziwei` / 六爻 `liuyao`）的知识库存放与检索：

- **存放规则**：知识库文件放哪里、如何增减、如何让新增/修改生效（知识库会经常变化，这里给出唯一权威流程）。
- **检索规则**：skill 脚本 `retrieve` 子命令的协议——agent 一律通过调用脚本检索，**禁止直接翻阅知识库 JSON 文件**；脚本自动声明完整版/简易模式；未命中的内容禁止编造。

## 一、知识库存放规则

### 1.1 位置（Node 环境）

知识库根目录：仓库（或运行时工作目录）下 `assets-private/knowledge/`，内含：

```
assets-private/knowledge/
├── manifest.json              # 唯一开关：files 清单（必须与磁盘一致）
├── ziwei/
│   ├── meta.json              # 模块元数据（计数/来源）
│   ├── default/core.json      # 常驻注入（35 条默认核心规则）
│   ├── conditional/           # 条件注入（star-palace / star-combinations）
│   └── retrieval/rules/*.json # 检索资产（按领域分类：pattern/career/love/…）
└── liuyao/
    ├── meta.json
    ├── default/               # foundation / habits（常驻）
    ├── conditional/           # use-god / stage-guides（条件注入）
    └── retrieval/chunks/*.json # 检索资产（definition/condition/procedure/…）
```

Web 环境对应位置：`packages/web/public/assets-private/knowledge/`（目录结构与上相同）。

### 1.2 manifest.json 是唯一开关

- 新增/删除/修改任何知识库文件后，**必须同步更新 `manifest.json` 的 `files` 清单**（相对 `knowledge/` 的路径，如 `"ziwei/retrieval/rules/love.json"`）。
- 读取方（skill 脚本 `retrieve`、MCP `applyPrivateAssets`、Web 加载器）一律**以 manifest 为准**：清单里没有的文件不会被加载，清单里有但磁盘缺失的文件会被跳过。
- 检测知识库是否存在的标准：`assets-private/knowledge/manifest.json` 是否存在且可解析。注意 `assets-private/` 被 .gitignore 排除，glob 检索可能误报缺失——判断时用 `Test-Path`（Windows）/`test -f`，或直接 Read 该文件，至少两种方式独立确认。

### 1.3 文件格式约定

- 每个检索资产 JSON 顶层：`{ "module": "<ziwei|liuyao>", "layer": "retrieval", "category": "<领域名>", "count": N, "items": [...] }`。
- 条目字段（紫微）：`text`（正文，必须）、`section`/`subject`（定位，可选）、`categories`（领域数组，用于 `--category` 过滤）、`specificity`（0-1，可微调排序权重）。
- 条目字段（六爻）：`content` 或 `text`（正文，必须）、`title`/`section`（可选）、`intents`/`aliases`（命中词，可选，权重更高）。
- 条目内的 `ruleId`/`chapter`/`lineStart`/`lineEnd`/`sources` 等字段仅供内部追踪，**不会出现在 `retrieve` 输出中**；检索输出只投影展示字段（text/section/subject/categories）。

### 1.4 知识库变更流程（新增/修改/删除）

1. 编辑对应 JSON（或原始知识源再导出）。
2. 同步更新 `manifest.json` 的 `files`。
3. 重启使用方即可生效，**无需重建 skill bundle**（脚本按 manifest 运行时读取；MCP/Web 启动时注入）。
4. 验证：`node <技能>/scripts/ziwei-calc.mjs retrieve --query <新内容关键词>` 能命中即为生效。
5. 技能被复制到其他 agent 目录（`~/.codex/skills/` 等）后，知识库不随技能分发——复制后技能处于简易模式（见下）。

## 二、检索规则（retrieve 子命令协议）

### 2.1 何时用

- **解读/核对任何取义时**：先 `retrieve --query` 检索相关规则，引用命中的正文。
- **开工前声明模式**：任意一次 `retrieve` 调用都会返回 `mode: "complete" | "basic"`，以此向用户声明本次使用完整版还是简易模式。
- **禁止**：不调用检索而直接翻读 `assets-private/knowledge/**` JSON 文件；不凭记忆或教材引用未检索到的规则。

### 2.2 参数

| 参数 | 必填 | 说明 |
|---|---|---|
| `--query` | 是 | 检索词（自然语言或关键词均可，中文直接整句输入） |
| `--category` | 否 | 逗号分隔的领域过滤（紫微如 `pattern,sihua`；六爻如 `definition,condition`） |
| `--limit` | 否 | 返回条数上限，默认 6，最大 20 |
| `--knowledge-dir` | 否 | 覆盖知识库根目录（默认探测：参数 > 环境变量 `STARS_KNOWLEDGE_DIR` > `cwd/assets-private`） |

### 2.3 输出

```json
{
  "command": "retrieve",
  "module": "ziwei",
  "mode": "complete",
  "query": "破军化权官禄宫",
  "category": "pattern",
  "limit": 6,
  "status": "ok",
  "matchedRuleCount": 3,
  "hits": [
    { "text": "…规则正文…", "section": "2.3 财富财帛规则", "categories": ["pattern", "wealth"] }
  ],
  "filesLoaded": 15,
  "discipline": "检索结果只作取义参考…"
}
```

- `mode`：`complete`（命中/无命中都说明知识库可用）或 `basic`（未检测到知识库，只能用 references 兜底）。
- `status`：`ok`（有命中）、`no_match`（知识库存在但未命中）、`no_assets`（知识库缺失）。
- `hits[].text`：规则正文；`section`/`subject`/`categories` 为可选定位信息。
- `discipline`：随结果返回的反编造纪律（与 04-discipline.md 一致）。

### 2.4 使用纪律（反编造）

1. 只能引用 `hits[].text` 的正文或 skill `references/` 的公开简易知识；`no_match` 时**不得编造规则内容**，应如实说明未检索到并换更具体的关键词重试（如加星曜+宫位+四化词）。
2. 检索命中的规则是**取义参考**，不是盘面事实；与脚本返回的程序事实（排盘/四化/流年/旺衰）冲突时以程序事实为准。
3. 简易模式（`mode: "basic"`）下不得宣称使用了完整知识库，解读只引 references 简易知识。
4. 同一检索需求用一次 `retrieve` 取回即可，不重复调用；跨领域问题可分多次查询（如一次查格局、一次查领域规则）。

### 2.5 评分机制（如需调整排序）

- 对查询串做子串切分（长度 ≥2 的子串为词元）。
- 打分：正文 `text`/`content` 包含词元 +1；`section`/`subject`/`title` 包含 +2；`intents`/`aliases` 命中 +2；显式 `--category` 过滤时命中类目 +3。
- 紫微额外以 `specificity` 微调（+0.1 × specificity）。
- 按分数降序取前 `--limit` 条。想提高某条命中率：提高它在文本中关键词的覆盖度，或加 `aliases`（六爻）。

## 三、完整版与简易模式

| 场景 | 模式 | 依据 |
|---|---|---|
| `assets-private/knowledge/manifest.json` 存在且含本模块检索文件 | `complete` | 完整知识库 |
| manifest 缺失 / 无本模块检索文件 | `basic` | 内置公开简易知识（skill `references/`） |

- 公开仓库**不含** `assets-private/`（被 .gitignore），因此默认跑在简易模式；需要完整效果时按仓库根 `AGENTS.md` 的说明放置私有资产。
- 简易模式不影响功能链路：排盘/反推/起卦/旺衰全部可用，只是取义知识为基础级。
