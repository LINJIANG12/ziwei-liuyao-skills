# Ziwei-Liuyao Skills（紫微六爻智能体）

排盘、起卦、旺衰、应期——全部本地确定性计算。AI 只负责解读，不负责算。

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-标准-1f6feb)](https://agentskills.org) [![Node](https://img.shields.io/badge/Node-18%2B-339933)](https://nodejs.org) [![零依赖](https://img.shields.io/badge/zero--dep-纯%20Node-3fb950)](https://nodejs.org) [![License](https://img.shields.io/badge/License-NonCommercial(原创)-red)](LICENSE) [![Engine](https://img.shields.io/badge/排盘引擎-MIT-9747ff)](https://github.com/Renhuai123/ziwei-doushu)

[简介](#简介) · [为什么是它](#为什么是它) · [快速开始](#快速开始) · [文档](#文档) · [安装](INSTALL.md) · [使用](#使用) · [知识库](#知识库) · [常见问题](#常见问题) · [贡献](#贡献) · [许可](#许可) · [English](README_EN.md)

## 简介

两套自包含的智能体技能，遵循 Agent Skills 开放标准，30+ 主流 agent 直接可用：

- **ziwei-agent**（紫微斗数）：真太阳时校准排盘、时辰未知时 15 候选反推定盘、流年/流月/大限/小限核对、正式解读（紫微排盘引擎基于 [王多鱼 AI 的开源排盘引擎](https://github.com/Renhuai123/ziwei-doushu) 二次开发）
- **liuyao-agent**（六爻）：三种方式起卦（三枚铜钱法真随机 1:3:3:1）、旺衰计算、应期判断、转六亲视角

作者学易八年，高强度实战。六爻有本人七八成的水平，紫薇则是经过反复测试具备一定准确率。

## 为什么是它

**别人的 AI 在"算"命，这里盘是算出来的。** 排盘、四化、旺衰、应期，100% 本地确定性计算，同一份出生资料永远得到同一张盘，并且内置工具可调取任意时间的精确排盘，不会因幻觉出现错误，将模式专注放在推断本身上。大幅度提高推理上限。
**时辰记不准？15 候选反推是独一份。** 老人报生辰只说得出"天刚黑""鸡叫的时候"。这里生成 15 个候选时辰，拿你记得住的人生经历逐条核对：模糊记忆先确认再排除，强烈矛盾直接排除，排除即出局，锁定必须命主点头。别家的排盘软件，只会让你猜。

**真太阳时校准，多数排盘软件都忽略了。** 命理讲的是出生地的当地时间，钟表走的是标准时。按出生地经度 + 均时差修正，经度差几度，时辰就差一个。这一步不做，后面全是白排。

**流派明确。** 六爻排盘以《增删卜易》《卜筮正宗》为核心，兼容部分《易隐》，流派冲突处按作者多年实战习惯取舍；

**AI 敢编，这里不敢。** 解读取义一律走知识库检索，检索不到就明说。证据分层、未取回的时间层不下结论、工具失败不外推——判断纪律写死在协议里，不是靠模型自觉。

## 快速开始

```bash
# 复制技能目录到你的 agent（以 Claude Code 全局安装为例）
mkdir -p ~/.claude/skills
cp -r skills/ziwei-agent skills/liuyao-agent ~/.claude/skills/

# 验证知识库模式（在发布包根目录运行）
node skills/ziwei-agent/scripts/ziwei-calc.mjs retrieve --query "命宫紫微贪狼"
```

`retrieve` 输出自带模式声明：`mode: "complete"` 表示完整知识库可用；`mode: "basic"` 表示简易模式。注：本包六爻检索知识库不开源，六爻 `retrieve` 恒为 `basic`（预期行为，见「知识库」）；紫微为 `complete`。各 agent 的详细安装、升级与卸载见 [INSTALL.md](INSTALL.md)。

## 文档

| 文档 | 内容 |
|---|---|
| [紫微斗数技能介绍](docs/ziwei-intro.md) | 我做了什么、优点、限制与未来改进计划 |
| [六爻技能介绍](docs/liuyao-intro.md) | 我做了什么、优点、限制与未来改进计划 |
| [命理心得](docs/insights.md) | 作者个人的命理学习与实践心得 |

## 使用

### ziwei-agent（紫微斗数）

| 场景 | 命令 |
|---|---|
| 排盘（时辰已知） | `init --gender <male\|female> --date <YYYY-MM-DD> --time HH:MM`，取输出 `recommendedCandidateKey` 后 `chart --candidate-key <key>` 一步取盘 |
| 排盘（时辰未知） | `init` 不带 `--time` → 15 候选，按 `references/03-inference-flow.md` 流程逐事件核对 |
| 流年/流月/大限/小限 | `year-facts --candidate-key <key> --year <YYYY> [--summary]` |
| 解读取义 | `retrieve --query <主题词>`（如"命宫紫微贪狼"），命中正文即取义参考 |

### liuyao-agent（六爻）

| 场景 | 命令 |
|---|---|
| 起卦 | `cast --question "..."`（随机算法）· `--lines <6位背数>`（手动摇卦）· `--name <本卦名> [--changed-name <变卦名>] --timestamp`（用户指明） |
| 旺衰 | `strength`（annual / monthly / five_day / intra_day 各时间单位） |
| 转六亲视角 | `perspective`（某爻为参照核心的局部关系） |
| 解读 | 取用神 → 看旺衰 → 看动变 → 断吉凶与应期 |

完整子命令参数、输出格式与状态协议（`--session` 会话隔离、`history` 历史卦、`visualize` 盘面可视化）见各技能内 `references/*-tool-protocol.md`。

## 知识库

- 本包内置公开知识库子集（`assets-private/knowledge/`）：**紫微完整检索规则** + 六爻基础注入规则
- **六爻检索知识库（206 条检索规则）为私有资产，不随本包分发**——六爻 `retrieve` 返回 `mode: basic` 属预期，基础取义见 `references/06-knowledge-injection.md`（两环节注入说明）与 `05-basic-knowledge.md`（简易知识）；紫微 `retrieve` 为 `mode: complete`
- 删除 `assets-private/` 目录即全部退回简易模式——功能链路完整，只是取义知识为基础级
- 替换为完整知识库：放入任意目录（结构见 [docs/knowledge-retrieval.md](docs/knowledge-retrieval.md)），用环境变量 `STARS_KNOWLEDGE_DIR` 或 `retrieve --knowledge-dir` 指定；探测顺序：`--knowledge-dir` > `STARS_KNOWLEDGE_DIR` > `cwd/assets-private` > 脚本上溯路径
- 知识库变更即时生效：脚本按 `manifest.json` 运行时读取，无需重建脚本

## 常见问题

| 问题 | 回答 |
|---|---|
| `retrieve` 返回 `basic`？ | 知识库目录未命中。确认 `assets-private/knowledge/manifest.json` 存在，或设置 `STARS_KNOWLEDGE_DIR` 指向知识库根目录 |
| 起卦结果想复现？ | 用 `--seed <整数>`，同一 seed 输出完全一致；不传 seed 时走三枚铜钱真实随机 |
| 状态文件在哪？ | 各技能 `.stars-state/`（会话、归档、可视化），可随时删除，不影响功能 |
| 多套占问/反推混在一起？ | 用 `--session <名称>` 隔离，或先 `reset` 清空 |

## 贡献

- 问题与建议：开 issue
- 文档与知识库：直接改 `skills/`、`docs/`、`assets-private/knowledge/`；修改知识库文件后同步更新 `manifest.json` 的 `files` 清单（见 docs/knowledge-retrieval.md）
- 计算脚本：发布包内为构建产物，源码与构建流程在上游仓库

## 许可

**双许可**：

- **作者（linjiang）原创部分**（六爻引擎、Agent 技能、知识库、文档）：[PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0)——个人随意使用，**严禁任何形式的商业使用**；商业使用须另行获得作者授权
- **紫微排盘引擎**：基于 [王多鱼 AI 的开源排盘引擎](https://github.com/Renhuai123/ziwei-doushu) 二次开发，遵循其 MIT 许可

详见 [LICENSE](LICENSE)。
