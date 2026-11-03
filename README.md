# AI Project Delivery · AI 项目交付方法论

> 一个跨 harness 的 AI 编码交付框架：从需求对齐到上线验收，并附带非技术用户模式。

这份仓库是一份 **Agent Skill**（根部的 `SKILL.md`），适用于 DeepSeek Harness、Claude Code、Codex、Cursor 等支持 Agent Skills 的运行时。它把「用 AI 代理交付软件项目」这件事拆成可执行的分阶段流程、心智模型与决策规则，并明确保留不同流派之间的分歧。

- **它是什么**：一份方法论技能 + 三份可追溯的调研笔记（55 个来源，一手占比 80% 以上）
- **它不是什么**：不是又一个 Spec-Driven Development 工具，不绑定单一厂商流程，不扮演任何人物
- **适用对象**：正在用编码 agent 做项目的人，以及**不懂代码、只会用 AI 的普通人**（内置「非技术用户模式」）

## 快速开始

### 方式一：通用技能安装器

```bash
npx skills@latest add anjiecobamboo-blip/ai-project-delivery
```

安装器会按提示选择运行时（Claude Code、Codex、Cursor 等）。

### 方式二：DeepSeek Harness

把仓库复制或克隆到 DSH 的用户技能目录：

```powershell
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$HOME\.dsh\skills\ai-project-delivery"
```

DSH 会自动发现目录根部的 `SKILL.md`，无需重启（热刷新）。

### 方式三：Codex / Claude Code / Cursor

```powershell
# Codex
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$env:CODEX_HOME\skills\ai-project-delivery"

# Claude Code
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$HOME\.claude\skills\ai-project-delivery"
```

### 方式四：作为 DSH 插件

本仓库同时是一个 DSH 插件包（`package.json` + `cordis.patch.yml`）：`cordis.patch.yml` 复用了 DSH 自带的 `@deepseek-ai/dsh-skill-filesystem`，把包目录注册为技能根目录。将它作为依赖安装到 DSH profile 后即可由 loader 加载。

## 什么时候会触发

**会自动或按需介入**（讨论「怎么做项目」时）：

- 要不要写 spec、怎么拆工单、会话怎么切
- 多 agent 能不能并行、能不能让 agent 自动跑（AFK）
- AI 代码怎么 review、Spec Kit / Kiro / 纯对话怎么选
- 新库或老库怎么接入、软件工厂怎么搭

**不会触发**（避免拖慢日常小任务）：

- 单次编码任务 → 用具体工程技能
- 单次排错 → 用 `diagnosing-bugs` 类技能
- 单次 diff / PR 评审 → 用 `code-review` 类技能
- 单次并行任务分派

## 内容概览

| 模块 | 内容 |
|---|---|
| 四个失效模式 | 没对齐、语言不共享、代码不工作、变成泥球 |
| 四层诊断表 | 对齐层 / 上下文层 / 验证层 / 架构层，含判定问句 |
| 8 阶段主流程 | 准备 → 对齐 → 原型/研究 → 规格 → 工单 → 实现 → 评审 → 交付 → 复盘 |
| 6 个心智模型 | 验证带宽定律、上下文预算、对齐是资产、深模块可导航性、油门与刹车、冷上下文评审 |
| 10 条决策启发式 | 会话切分、工单粒度、并行条件、停手信号、规则落在哪 |
| 10 类反模式 | vibe coding、大 PR、自写自评、无刹车自动化等 |
| 流派分歧表 | 规格地位、上下文管理、多 agent 并行、AFK 边界、TDD、review 位置、组织结构、安全 |
| 非技术用户模式 | 用户不懂代码时的切换规则、阶段适配、验收模板、安全网与固定话术 |
| 跨 harness 映射 | Claude Code / Codex / DSH 的等价操作 |

## 与现有框架的关系

| 项目 | 定位 | 与本仓库的差异 |
|---|---|---|
| [github/spec-kit](https://github.com/github/spec-kit) | 官方 SDD 工具包 | 本仓库把 SDD 作为「一种流派」而非默认答案 |
| [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) | 面向 AI 助手的 SDD | 同上，并给出「规格是输入还是压缩」的判断条件 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 角色化敏捷 AI 开发流程 | 本仓库不要求角色扮演，按任务规模裁剪流程 |
| [obra/superpowers](https://github.com/obra/superpowers) | 技能框架 + 开发方法论 | 本仓库是单主题方法论文档，可与其技能组合使用 |
| [mattpocock/skills](https://github.com/mattpocock/skills) | 工程技能集合（grill/spec/tickets/implement） | 本仓库吸收了其流水线并补充其它流派与反例 |

## 非技术用户模式

当用户表示自己不懂代码、只会用 AI 时，技能会切换为：

- 一次最多问 3 个问题，能用默认值就用默认值
- 不展示代码；技术概念先用一句类比解释
- 每次交付必须能演示（截图/录屏/可打开的产物）+ 不超过 3 条用户自己能判定的验收清单
- 检查点停下来让用户确认：需求摘要、规格、第一版切片、每轮验收
- 涉及支付、账号、删除数据、对外发布时先说明风险并取得同意

## 目录结构

```
.
├── SKILL.md                      # 技能主体（安装后在运行时自动发现）
├── agents/openai.yaml           # Codex 展示信息（display_name / short_description）
├── references/research/         # 调研笔记（来源、日期、一手/二手标注）
│   ├── 01-pocock-pipeline.md
│   ├── 02-vendor-spec-driven.md
│   └── 03-practitioner-schools.md
├── cordis.patch.yml             # DSH 插件入口
├── package.json
├── README.md / README.en.md
├── CHANGELOG.md
└── LICENSE
```

## 来源与致谢

本技能由「女娲 · Skill造人术」流程生成，综合以下公开材料（完整 55 个来源与日期见 `references/research/`）：

- Matt Pocock / AI Hero：grill → spec → 工单 DAG → implement → 独立评审 → retro，smart zone、deep modules、software-factory
- GitHub Spec Kit、AWS Kiro、Anthropic 官方文档：规格驱动、上下文工程、确定性检查
- Simon Willison、Cognition、Addy Osmani、HumanLayer、Kent Beck、Steve Yegge、Böckeler：验证带宽、多代理边界、AFK 反例、12-factor agents

感谢以上作者与维护者。本仓库与其无隶属或背书关系，所有引用均标注出处。

## 已知局限

- **时效**：调研时间 2026-11-03；该领域厂商立场一年内出现过多次反转
- **证据等级**：主体是实践者经验与厂商自述，几乎没有对照实验；技能内的经验阈值标有〔推断〕
- **工具差异**：多 agent、hook、自动压缩等能力在不同 harness 上不同，技能提供映射但不等价
- **规模偏差**：小团队、绿地的证据最强；大团队协作、合规、成本模型基本缺位

## 贡献

欢迎提交 issue 或 PR 修正事实、补充来源、改进流程判据。请保持两点：

1. 新增主张附来源（URL + 日期），区分一手/二手；
2. 不抹平分歧——相互冲突的证据应并列呈现并说明适用条件。

## License

[MIT](./LICENSE)

---

> 本仓库由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 流程生成，作者：`anjiecobamboo-blip`。
> 若你 redistribution，请保留来源与许可声明。
