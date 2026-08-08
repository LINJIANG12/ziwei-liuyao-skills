# 安装指南

[README](README.md) · [English](README_EN.md)

## 环境要求

- Node.js 18 或更高版本
- 无其它任何依赖——脚本是零依赖单文件，不需要 `npm install`、不需要 API Key

## 安装
把 `skills/` 下的技能目录复制到你的 agent 技能目录。agent 会在相关任务出现时自动发现并加载（渐进式：先读 SKILL.md 索引，按需打开 references 与运行脚本）。

### Claude Code

```bash
# 全局安装（所有项目可用）
mkdir -p ~/.claude/skills
cp -r skills/ziwei-agent skills/liuyao-agent ~/.claude/skills/

# 或项目级安装（仅当前项目）
mkdir -p .claude/skills
cp -r skills/ziwei-agent skills/liuyao-agent .claude/skills/
```

### Codex

```bash
mkdir -p ~/.codex/skills
cp -r skills/ziwei-agent skills/liuyao-agent ~/.codex/skills/
```

或放入项目 `.codex/skills/` 目录（仅当前项目生效）。

### ZCode

- 全局：复制到 `~/.zcode/skills/`
- 或在 Settings → Skills 中导入

### GitHub Copilot

```bash
mkdir -p ~/.copilot/skills
cp -r skills/ziwei-agent skills/liuyao-agent ~/.copilot/skills/
```

或放入项目 `.github/skills/` 目录。

### 其它工具

支持 Agent Skills 开放标准的工具（Cursor、Gemini CLI 等），按各自官方文档将技能目录放入对应 skills 目录即可。

## 验证

安装后，在发布包根目录运行以下命令确认知识库模式：

```bash
node skills/ziwei-agent/scripts/ziwei-calc.mjs retrieve --query "命宫紫微贪狼"
node skills/liuyao-agent/scripts/liuyao-calc.mjs retrieve --query "用神"
```

输出含义：

| 输出 | 含义 |
|---|---|
| `mode: "complete"` | 完整知识库可用，解读取义走检索 |
| `mode: "basic"` | 简易模式，用各技能 `references/` 简易知识兜底 |

说明：本包六爻检索知识库不开源，六爻 `retrieve` 恒为 `basic`（预期行为），基础取义见 `references/06-knowledge-injection.md`。两种模式功能链路都完整（排盘/起卦/旺衰全部可用），区别只在取义知识的深度。

## 升级

重新下载发布包，将 `skills/` 下两个技能目录复制覆盖即可。会话状态在技能目录外的 `.stars-state/`，覆盖技能目录不影响已存档状态。

## 卸载

- 删除 agent 技能目录下的 `ziwei-agent` 与 `liuyao-agent`
- 如需清空状态，一并删除技能目录的 `.stars-state/` 子目录

## 常见问题

**retrieve 返回 basic？**
知识库目录未被探测到。检查发布包根目录 `assets-private/knowledge/manifest.json` 是否存在；或设置环境变量 `STARS_KNOWLEDGE_DIR` 指向知识库根目录（包含 `knowledge/` 的目录）。探测顺序：`--knowledge-dir` 参数 > `STARS_KNOWLEDGE_DIR` > `cwd/assets-private` > 脚本上溯路径。

**技能被复制到 agent 目录后，知识库还能用吗？**
可以。技能脚本会按上述顺序自动探测知识库；如果复制技能后 cwd 不再包含 `assets-private/`，用 `STARS_KNOWLEDGE_DIR` 显式指定即可，或将 `assets-private/` 一并复制到工作目录。

**起卦结果想复现？**
`cast` 时传 `--seed <整数>`（必须为整数），同一 seed 得到完全相同的卦象；不传 seed 时走三枚铜钱法真实随机（老阴/少阳/少阴/老阳 = 1/8、3/8、3/8、1/8）。

**Windows 用户注意**
脚本为纯 Node 实现，Windows/macOS/Linux 均可运行。路径含中文或空格时，用引号包裹路径，或先 `cd` 到发布包目录再运行。
