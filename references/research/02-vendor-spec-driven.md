# 厂商侧 AI 编码交付与规格驱动开发（SDD）调研

**调研时间** 2026-11-03｜**来源** 15 项（一手 12、二手 3，约 80% 一手）｜**标注**〔官〕厂商官方文本、〔社〕社区/从业者反馈、〔推〕我的推断。正文用 [S#] 引注，URL 见文末。Codex 官方文档站（developers.openai.com）本次抓取返回 403，Codex 侧仅核实到仓库 README [S9] 与 AGENTS.md 开放格式站 [S8]，"任务委派"原文未取到——此为覆盖缺口，不作推断补足。

## 1 流程与工件：谁写、谁读、谁验收

| 方案 | 工件链 | 谁写 | 谁验收 |
|---|---|---|---|
| Spec Kit v1.0 [S5][S6] | `constitution.md` → `spec.md` → `plan.md` → `tasks.md` → 代码；可选质量门 `clarify` / `checklist` / `analyze`；`converge` 收尾 | agent 依据模板生成，人提供意图与约束 | 人逐阶段复核；`analyze` 只读不改；`converge` 只追加任务 [S6] |
| Kiro [S7] | `requirements.md`（用户故事+EARS 验收）→ `design.md`（数据流、TS 接口、schema、图）→ `tasks.md`（依赖排序、回链需求号） | agent 从一句提示生成三件套 | 人逐任务触发、看 diff 与执行历史 [S7] |
| Claude Code [S1] | 无强制工件链；`CLAUDE.md`（<200 行）+ `.claude/rules/`（按路径）+ skills + `SPEC.md`（可选） | 人写规则，或让 agent 访谈后写 `SPEC.md` | 独立子代理对 diff 评审；`Stop` hook 强制检查 [S1] |
| AGENTS.md [S8] | 单文件 Markdown，可嵌套；就近文件优先，但"聊天中的显式指令覆盖一切" | 人（"活文档"） | 人评审合并；agent 先自跑清单内检查 [S8] |

三条官方自我限缩，值得直接引用：

- Spec Kit 虽称命令"按顺序设计"，但同时声明"只有 `/speckit.specify` 在 `/speckit.plan` 前是严格必需的"，clarify/checklist/analyze 是"为真正有歧义的东西才加的质量门"；bug 修复与想法评估是"独立入口，不是强制阶段"〔官 S6〕。相比 2025 年的五命令线性链，这是明显的去流程化。
- `converge` 被设计成 append-only：不改进代码、唯一写入是往 `tasks.md` 追加任务，循环到输出 `Converged` 为止〔官 S6〕。这是"规格→实现"闭环里唯一有机器判据的收尾。
- Kiro 官方自认："specs 与演进中的代码库保持同步……这解决了开发者实现时停止更新原始工件、导致文档错配的常见问题"〔官 S7, 2025-07-14〕。即厂商承认文档腐化是默认结局。

## 2 上下文工程：官方立场本身在移动

- 2025-09 立场：上下文是"有限且边际收益递减"的资源，因 context rot（token 越多、召回越差）与 n² 注意力摊薄，目标是"最小的高信号 token 集"；手段是 compaction（最轻的是清工具结果）、结构化笔记（NOTES.md）、子代理隔离（子代理烧数万 token，只回 1,000–2,000 token 摘要）〔官 S3〕。同文判断："等更大窗口"不可靠——所有尺寸的窗口都受污染与相关性影响〔官 S3〕。
- 2026 立场反转：该文档把"上下文窗口是主要约束，要激进管理"列入**可以停止做**的旧建议，理由是"当前模型多为一兆窗口，压缩自动发生，跑满很罕见"；"用子代理保护上下文"同样被列为过时，子代理改为服务并行、隔离噪声输出、获得未受污染的评审者〔官 S1，访问 2026-11-03〕。
- 同站另一页仍写"子代理通过把探索与实现挡在主对话之外来**保留上下文**"〔官 S2〕。两页并存，官方未调和。
- 官方当下的真实主张是**确定性 vs 非确定性**二分，而非"多写文档"：prompt/`CLAUDE.md`/skill 属模型可自主裁量的非确定性指令；hook、deny 规则、CI、分支保护属 harness 必执行的确定性约束。判据很实用——"反复强调仍被忽略的 `CLAUDE.md` 行，说明它该变成 hook"〔官 S1〕。
- 渐进披露是各家的共识机制：skill 只在会话注入名称与描述，命中才读全文〔官 S1〕；Gemini CLI 的 skill 生命周期为发现→激活→**用户同意**→注入→执行，激活前弹确认并只授予该目录读权〔官 S10〕；Cursor 规则分四种生效方式（always / 按 glob / 按描述智能选用 / 手动 @），并明确"引用文件而非复制内容"以防规则随代码腐化〔官 S11〕。

## 3 验证闭环与人工介入点

- 官方把验证拆成"agent 可自跑的检查"与"harness 强制的门"两级。Claude Code 的梯度：提示里写命令 → `/goal`（独立模型每轮判定）→ `Stop` hook（退出码 2 阻止结束，连续阻止 8 次后放行）→ 由没写过代码的子代理跑 `/code-review`〔官 S1〕。
- 证据要求是显式的："向 Claude 要**证据**：跑的命令与输出，或截图。'测试都通过'本身几乎不说明问题"〔官 S1〕。无人值守前要回答五问：done 的定义、能否自检、可触碰范围、什么能叫停、结果交给谁签收〔官 S1〕。
- 评审要限定范围：官方给出的评审提示包含 `skip style`，理由是"被要求找问题的评审者总能找到东西，全修完就多出抽象层和不可能场景的测试"〔官 S1〕。
- 人工介入被制度化的位置：Spec Kit 自定义 checklist"属评审者所有，实现不得静默自批"，`implement` 只读勾选状态、遇未勾选先询问〔官 S6〕；Cursor 明说"AI 指导不应是你唯一的安全控制"〔官 S11〕；Anthropic 早前结论仍是"自动化测试验证功能，人工评审确保方案符合更大系统要求"〔官 S4, 2024-12〕。
- 多代理编排：Claude Code 默认委派（Explore/Plan 只读、新窗口、跳过 `CLAUDE.md`），并设并发 20、嵌套 3 层上限，子代理回报还会被扫描以防提示注入〔官 S2〕。Spec Kit 与 Kiro 均**不做**主代理编排，只产出任务图交人逐条触发〔官 S6][S7〕。

## 4 关键差异轴

1. **重工件链 vs 轻对话**：Kiro/Spec Kit 把"需求→设计→任务"固化成文件与命令；Anthropic 的官方建议反而更轻——不知道要什么时让 agent 访谈你直到能写出 spec，写进 `SPEC.md`，然后**开新会话**实现，并用它当评审基准〔官 S1〕；Cursor/AGENTS.md 一派只保留一份规则文件〔官 S8][S11〕。
2. **规格是长期资产还是耗材**：Kiro 与 Spec Kit 追求"规格与代码同步演进"（`converge` 循环、spec 更新）〔官 S6][S7]；AGENTS.md 自称活文档但只承载项目常识〔官 S8〕。
3. **单代理 vs 多代理**：厂商侧只有 Anthropic 把多代理做成默认（子代理、worktree、`/batch` 每单元一 PR）〔官 S1][S2]；SDD 工具把并行留给"任务图 + 人"。
4. **规格的分层**：Böckeler 把 SDD 分为 spec-first / spec-anchored / spec-as-source 三层，并据其 2025-09 观察指出 Spec Kit 为每个 spec 建分支，实际仍是 spec-first（生命周期=一次变更），与其"活工件"叙事不符〔社 S12, 2025-10-15〕。v1.0 是否改掉此点本次未核实〔推〕。

## 5 批评与失败案例

- **大锤砸坚果**：Kiro 修一个小 bug，被展开成 4 个用户故事、16 条验收标准，含"作为开发者，我希望转换函数优雅处理边界情况"这类空话；Spec Kit 处理一个 3–5 点故事，产出大量重复、冗长、需人评审的 Markdown，"同样的时间我本可以用普通 AI 辅助编码做完，而且更感觉可控"〔社 S12〕。
- **假控制感与指令失效**：Spec Kit 的研究步骤把既有类的说明当成新规格，重新生成一遍造成重复代码；同时又会因过度遵守 constitution 某条而做过头〔社 S12〕。"有那么多的文件、模板、提示、工作流与 checklist，我仍频繁看到 agent 最终没遵守全部指令"〔社 S12〕。
- **构建退化为写文档**：一个"Create User Profile"故事膨胀到 19 条验收标准、1,070 行 Markdown，混入库表字段、组件映射、文件清单与测试清单，形成对后续实现的技术预偏见；"厚规格 ≠ 共享理解"（引 Jeff Patton："共享文档不是共享理解"）〔社 S13, 2026-04-08〕。
- **隐性知识与迭代经济学**：默会知识无法前置成文；当下一版可测原型只需几分钟时，"前置规格的经济学优势消失"，作者改为短 spec + 临时 plan + ADR + 把护栏下沉到 harness（测试、lint、架构规则、评审）〔社 S13〕。
- **社区原话**：spec-kit 讨论区存在题为"SpecKit creates the illusion of work, generating a bunch of text"的帖子〔社 S14，日期未核实，正文未取到〕。
- 可对证的失败模式收敛为四类：工件膨胀、评审负担转移给人、agent 不遵守自产工件、spec 与代码失同步（后者被厂商自己承认，见 §1）。

## 6 与 Pocock 流派的分歧

Pocock 侧文本以其 `to-spec` 技能文档为准〔一手 S15，无日期，v1.1+〕。三点是硬分歧，不在措辞：

| 议题 | 厂商 SDD | Pocock |
|---|---|---|
| spec 何时产生 | 工作开始时由 agent 生成，再驱动 plan/tasks〔S6][S7] | 对话**之后**综合，"不再提问"，"spec 里出现你没说过的话就是缺陷"〔S15〕 |
| 何时才需要 spec | 默认流程，规格是主线 | 仅当工作跨多个会话；单会话"跳过 spec，直接 implement"〔S15〕 |
| spec 的生命周期 | 与代码同步演进，`converge` 收敛〔S6] | "没有任何东西让它保持同步"，发版后可弃；持久知识进 `GLOSSARY.md` 与 ADR〔S15〕 |
| 上下文纪律 | 一兆窗口＋自动压缩，不必激进管理〔S1] | smart zone 约 125K–150K 起就变钝，"与窗口上限无关"（据同目录 `01-pocock-pipeline.md` 所引其演讲，非 S15 文本；与 S3 的 context rot 论证同向） |
| 评审读什么 | 读整套 Markdown 工件链 | 只读 seams 与 out-of-scope 两节，"其余留给 agent"〔S15〕 |

共同点也别忽略：两者都要求先定测试接缝再写规格、都要求评审者拥有独立上下文、都把"标准"放在评审侧而非实现侧。分歧的实质不是"要不要规格"，而是**规格是流程的输入还是对话的压缩**，以及**谁承担工件维护成本**。

## 7 未调和的矛盾（保留，不合并）

1. 上下文稀缺（S3, 2025-09）vs 上下文充裕（S1, 2026）：同一厂商、同一站点，结论相反，且 S2 仍保留旧表述。
2. 规格是资产（Kiro/Spec Kit 的同步叙事）vs 规格是耗材（Pocock"发版即弃"）；厂商自己的 `converge` 循环反而是"承认规格与现实会偏离"的证据。
3. 流程越重越可控（SDD 前提）vs 观察到的"假控制感"（S12）与"文档即负担"（S13）；但 S13 作者仍承认 SDD 在**思考阶段**有效——批评的是一路重到实现阶段。
4. checklist 由人拥有、agent 不得自批（S6）vs checklist 由 agent 生成、由 agent 逐轮评估（同一文档的机制设计）。

## 来源

| # | 类型 | 日期 | URL |
|---|---|---|---|
| S1 | 官 | 无标注（访问 2026-11-03） | https://code.claude.com/docs/en/best-practices |
| S2 | 官 | 无标注 | https://code.claude.com/docs/en/sub-agents |
| S3 | 官 | 2025-09（页面未标，据正文 Sonnet 4.5 发布推断） | https://www.anthropic.com/research/effective-context-engineering-for-ai-agents |
| S4 | 官 | 2024-12 | https://www.anthropic.com/research/building-effective-agents |
| S5 | 官 | v1.0（含 2026-10-16 动态） | https://github.com/github/spec-kit |
| S6 | 官 | 无标注 | https://github.github.io/spec-kit/reference/agentic-sdd.html |
| S7 | 官 | 2025-07-14 | https://kiro.dev/blog/introducing-kiro/ |
| S8 | 官/开放格式 | 无标注 | https://agents.md/ |
| S9 | 官 | 无标注 | https://github.com/openai/codex |
| S10 | 官 | 无标注 | https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/skills.md |
| S11 | 官 | 无标注 | https://cursor.com/docs/rules |
| S12 | 社 | 2025-10-15 | https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html |
| S13 | 社 | 2026-04-08 | https://www.innoq.com/en/blog/2026/04/versteckte-kosten-spec-driven-development/ |
| S14 | 社 | 未核实 | https://github.com/github/spec-kit/discussions/1784 |
| S15 | 一手实践 | 无标注（v1.1+） | https://github.com/mattpocock/skills/blob/main/docs/engineering/to-spec.md |
