# Matt Pocock：AI 项目交付方法论调研

**调研时间** 2026-11-03｜**来源** 22 项（一手 18、二手 4；其中 3 项二手同出 ai-wiki 这一个 wiki，一手约 82%）｜**标注**〔他〕本人文本或演讲原话、〔二〕他人总结、〔推〕推断；来源编号见文末。他明确反对 GSD/BMAD/Spec-Kit 这类「接管流程」的框架：拿走控制权后，过程里的 bug 就难修〔他 S1〕。

## 1 主流水线（仓库 v1.3.2）

0）`/setup-matt-pocock-skills`：每 repo 一次，定工单系统、标签、文档位置〔他 S9〕。

1）`/grill-with-docs`（无 repo 用 `/grill-me`）：模糊想法 → 对齐的决策＋`GLOSSARY.md`＋ADR。判据：设计树每个分支都解决，按轮质询 frontier。理由：第一失效模式是错位，不是能力〔他 S1/S4〕。

2）可选 `/prototype`：必须跑起来才答得出的问题，借 `/handoff` 往返；原型当一手材料留在 `prototype/<name>` 分支〔他 S3〕。

3）`/to-spec`：把同一窗口的对话综合成 spec（问题、方案、用户故事、实现与测试决策、范围外），发布到工单系统，不做访谈；先确认测试缝，「理想数量是一条」。理由：「上下文窗口会结束」，spec 持久、工单一次性〔他 S5/S10〕。

4）`/to-tickets`：切 tracer-bullet 工单并声明阻塞边，形成 DAG。判据：纵切贯穿各层、单独可演示、塞进一个全新窗口；先 quiz 用户再发布。理由：顺序计划只能单代理跑，DAG 才可并行且依赖可见〔他 S6〕。

5）`/implement` 逐条（条间 clear）或 `/implement-spec` 整份（frontier 并行、集成分支）。判据：一次运行一条工单，实现者按缝走 `/tdd`。理由：一条工单一个干净窗口＝保住聪明区〔他 S7/S8〕。

6）`/code-review`：固定点以来的 diff 做双轴（Standards / Spec），各起子代理、绝不合并。理由：混合判定会让通过的一轴掩盖失败的一轴〔他 S11〕。

7）`/pr` 写 PR 体，`/retro` 收尾：机械问题转确定性检查，判断题进 `CODING_STANDARDS.md`。理由：标准属于 reviewer，不属于上下文压力最大的 implementer〔他 S12〕。

**上下文纪律**：1–3 步必须同一不断开的窗口，`/to-tickets` 前不 clear 不 compact；此后每条 `/implement` 从新窗口起〔他 S4〕。旁路：`/triage`（外部 issue）、`/diagnosing-bugs`（先建会红的反馈环）、`/wayfinder`（一窗装不下的雾状工程）。wayfinder 只产决策不产交付，清雾后必须交回 `/to-spec`；直接跳 `/implement` 会丢掉已链接的细节〔他 S4/S13〕。

## 2 上下文纪律

- smart zone 与窗口上限无关：「窗口还很空就可能已在 dumb zone，按 smart zone 规划，不是按窗口规划」〔他 S2〕。
- 阈值：前沿模型 125K–150K 起，他补一句「但这有争议」〔他 S2〕；CHANGELOG 记录从 ~120k 上调至 ~150k〔他 S3〕。
- 阶段边界五选项有顺序：Continue → `/clear` → `/handoff` → subagent → `/compact`。`/handoff` 只买可移植（换 harness、目录、同事、中途分叉），是窄工具不是通用桥〔他 S4〕。

## 3 架构主张：deep modules

- 深度＝接口处的杠杆，不是「实现行数/接口行数」〔他 S14〕。
- 对 AI 的意义：浅模块代码库＝稠密依赖图，代理逐边追踪会耗尽上下文；代理还会给每个小函数套测试边界，得到假覆盖。深模块给一个大测试边界，能抓集成 bug，并给出稀疏导航面〔二 S15；他 S1「代理同时加速软件熵增」〕。
- 规则：接口即测试面，「想测到接口之外说明模块形状不对」；「一个 adapter 是假缝，两个才是真缝」〔他 S14〕。`/to-spec` 把选缝提前到 spec 阶段确认〔他 S5〕。
- 维护：`/improve-codebase-architecture` 定期普查深化候选，但他限定「这是普查不是抢救，不会替你解开泥球」〔他 S1〕。

## 4 并行与自动化

- **Ralph loop**（2026-04 演讲）：先 `once.sh` 跑通再套 `while true`；prompt 硬编码优先级「关键 bug > 开发基建 > tracer bullet > 打磨」；工单打 AFK 标签，人离场＝夜班〔二 S15〕。
- **Sandcastle**：他的 TS 库，`sandcastle.run()`，沙箱可插拔（bubblewrap/Seatbelt、Docker、Daytona、E2B、Modal、Vercel），带分支策略与回合并〔他 S18〕。演讲版形态：planner → 每 issue 一个 worktree+Docker 实现者 → reviewer → merger〔二 S15〕。
- **`/implement-spec`**（1.3.0 转正）：harness 内编排，工单当任务图，frontier 上每条起一个 worktree 实现者，全部落一条集成分支，末尾跑一次 review；子代理之间只传上下文指针〔他 S8/S17〕。他明说这不替代 Sandcastle：真 AFK 的活用「确定性循环（Sandcastle、shell 脚本、CI job）更快更便宜更可靠，因为没有代理做编排决策」〔他 S17〕。
- **software-factory**（in-progress）：只给词汇不给工具——task/trigger/run/runner ＋五个服务槽位（工单系统、工单来源、git host、orchestrator、可观测性）。核心一句：「每个 trigger 都是油门，没有刹车的油门造出 slop cannon」；三种刹车＝自动检查、自动 review、人工 review；一次只加一个触发器、先节流、每轮别超几次就交人〔他 S16〕。

## 5 模型分工：Sonnet 实现 / Opus 评审

理由：评审才需要脑力，实现可以磨；评审者必须拿全新上下文，否则「实现已把聪明区烧完，review 发生在 dumb zone」；标准对实现者是拉取、对评审者是推送〔二 S15〕。但当前技能文本一次没提模型名，README 说「works with any model」〔他 S1〕——这是 Sandcastle 时代的配置，不是仓库教义〔推〕。

## 6 他承认的未解问题

1. AI 下的 code review 未解：Ralph 批量提交把 PR 推大，与「PR 要小」冲突，「我老实说还不知道答案」〔二 S15 原话〕。
2. spec/PRD 保留策略只是工作假设；被问「数据库迁移不也一样」时答「我不知道」〔二 S15〕。
3. `code-review` 与 Claude Code 内置同名，「被报告最多的问题，且未修复」；子代理会再调它裂变，「有报告到 50 多个代理」；其结论也只是假设，「没有收敛保证，别循环跑到干净」〔他 S11〕。
4. `/to-tickets` 过度拆分是「被报告最多的问题」；同一 checkout 跑多个 `/implement`「比不支持更糟」（amend 落到别人 commit、stash 消失），worktree 也不解决 stash 共享〔他 S6/S7〕。
5. `/implement-spec` 的 review-fix 循环曾跑 4 小时；GitHub 上被阻塞工单在 blocker 合并后仍不启动〔他 S17〕。

## 7 撤回/修正过的建议

- smart zone ~120k → ~150k〔S3〕。
- 「`/handoff` 被过度推销」：只是窄工具；逃逸阀改口为 `/compact`〔S3，1.2.0〕。
- `to-prd` → `to-spec` 改名，「PRD」一词从发布文本清除〔S3〕。
- grilling 从「一次一问」改为「按轮问 frontier」，13 问压到 3 轮〔S3〕；`grill-me` 今只剩 7 行，转调 `grilling`〔他 S19〕。
- 「为简洁牺牲语法」的 CLAUDE.md 指令弃用；「别把 PRD 当 AFK 优化对象」〔二 S15〕。
- 「一次性」不再等于「删掉」：原型改留分支〔S3〕。删掉 `resolving-merge-conflicts`（1.3.0）与 `diagnosing-bugs` 的事后交接（「实际很少触发」，1.1.0）〔S3〕。
- wayfinder 的 ticket 改称 decision ticket，因为「人们一直当成实现工单读」〔S3〕。

## 8 局限与批评

- 生态绑定：Claude Code 是主分发渠道，`/clear`、`/compact`、token 状态栏都是该 harness 行为；同名冲突被他称为「Claude Code 的 harness bug」〔他 S11〕。`/implement-spec` 要求「后台子代理＋每代理一个 worktree」的 harness，「一次只能跑一个子代理的 harness 上，它只是更慢的 implement」〔他 S17〕；1.3.2 才补上 Codex fork 子代理的适配〔他 S3〕。
- 强依赖工单系统：没跑 setup 就让你去跑，本地 markdown 是降级方案〔他 S7/S8〕。
- 适用面：wayfinder 与 software-factory 面向雾状大工程与远程基础设施，绿地区域友好；遗留系统只给「普查不是抢救」的限定〔他 S1〕。多团队协作、合规、成本模型基本缺位〔推〕。
- 证据形态：成功判据是「It's working if」式定性清单，没有量化或对照实验〔他 S11/S17；推〕。

## 9 保留的矛盾（不调和）

1. **阈值**：wayfinder 按「一个 100K token 会话」定 ticket 尺寸〔S13〕，词典说 125–150K〔S2〕，`implement` 文档说「非平凡 ticket 超 100k 很正常」〔S7〕。
2. **clear vs compact**：演讲把 compact 当坏味道、强调 clear〔二 S15〕；仓库把 `/compact` 定为阶段边界默认，并把 Continue 列为最该先排除的选项〔他 S3/S4〕。
3. **spec 去留**：演讲主张实现后删掉防 doc rot〔二〕；`to-spec` 文档说「工单一次性，spec 不是，它留在原地记录推理」〔他 S10〕。
4. **TDD**：仓库把 `/tdd` 织进每步〔他 S7/S8〕；2026-09 播客里他说对 TDD 心情复杂，改为「让代理拿出能跑通的证据」，有无 TDD 都行〔二 S20〕。

## 来源

S1 `README.md`＋`CLAUDE.md`（本地 `data/mattpocock-skills-v1.3.2/`，v1.3.2）｜S2 https://www.aihero.dev/ai-coding-dictionary/smart-zone （2026-11-03 抓取）｜S3 同目录 `CHANGELOG.md`（1.1.0–1.3.2）｜S4 `skills/ask-matt\SKILL.md` ｜S5 `…\to-spec\SKILL.md` ｜S6 `…\to-tickets\SKILL.md`＋`docs/engineering/to-tickets.md` ｜S7 `…\implement\SKILL.md`＋`docs/engineering/implement.md` ｜S8 `…\implement-spec\SKILL.md` ｜S9 `skills/engineering/setup-matt-pocock-skills/SKILL.md` ｜S10 `docs/engineering/to-spec.md` ｜S11 `docs/engineering/code-review.md` ｜S12 `docs/engineering/retro.md` ｜S13 `…\wayfinder\SKILL.md` ｜S14 `skills/engineering/codebase-design/SKILL.md` ｜S15 https://raw.githubusercontent.com/tim-kaa-py/ai-wiki/…/summaries/2026-04-24_ai-engineer_workflow-for-ai-coding-matt-pocock.md （AI Engineer 2026-04-24 演讲，二手）｜S16 `skills/in-progress/software-factory/SKILL.md` ｜S17 https://www.aihero.dev/skills-implement-spec （2026-10-05 更新）｜S18 https://github.com/mattpocock/sandcastle README （2026-11-03 抓取）｜S19 `…\grill-me\SKILL.md` ｜S20 https://newsletter.pragmaticengineer.com/p/ai-skills-with-matt-pocock （2026-09-17，二手）｜S21 https://raw.githubusercontent.com/tim-kaa-py/ai-wiki/…/wiki/people/matt-pocock.md （2026-05-08，二手）｜S22 https://raw.githubusercontent.com/tim-kaa-py/ai-wiki/…/wiki/concepts/harness-engineering.md （2026-07-02，二手）
