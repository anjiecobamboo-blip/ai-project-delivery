# 03 · 实践者流派：用 AI 代理交付软件项目的七种主张

**调研时间**：2026-11-03 ｜ **来源**：18 个 ｜ **一手 15（83%）/ 二手 3（17%）**；推断处均标〔推〕。
标注：〔一〕一手〔二〕二手〔推〕推断。**矛盾一律保留，不调和。**
体例：URL 内联于每条主张后（不同于 02 的 [S#] 尾注），便于逐条对照冲突双方原文。

## 0. 总结论

六家在"小步、可验证、人担责"上意外一致；在**瓶颈归因、要不要规格、能否 AFK、写操作能否并行**上互不兼容。分歧不是深浅之争，是**对同一现象给出相反因果**。

## 1. 七家画像

| 流派 | 瓶颈判断 | 核心做法 | 明令禁止 | 适用条件 |
|---|---|---|---|---|
| **Simon Willison** | 验证 + 安全边界 | 红/绿 TDD、先跑测试、只读 subagent、线性走查、把规则写进工具而非 prompt | 把未审代码甩给协作者（指南 anti-patterns 首条） | 懂行工程师、生产系统；vibe coding 仅限"坏了自己承担"的自用工具 |
| **Cognition（Devin）** | 上下文工程 | 单线程写者；clean-context reviewer 循环；smart friend；manager→child 的 map-reduce | 并行写者 swarm、无共享上下文的子代理 | 长时任务、企业客户；2026 自认只有"写单线程"可行 |
| **Anthropic** | token 预算与并行广度 | orchestrator-worker、把 effort 预算写进 prompt、20 条种子 eval、end-state 评估 | 用于"需共享上下文或强依赖"的任务 | 广度研究型任务；自认编码平行度低 |
| **Yegge** | 吞吐与人工调度 | Beads 依赖图、GUPP 钩子、merge queue、Mayor/Polecat/Witness 分工 | 用 markdown TODO 管任务；Stage 6 以下团队用 Gas Town | 10+ 代理的老手、超大遗留库 |
| **HumanLayer** | 控制流与产品化（"80% 质量墙"） | 12 factor：自持 prompt/上下文、统一状态、pause/resume、人类作为工具调用、小代理 | 抛弃 DAG 全交给循环；整包换框架重写 | 面向客户的产品型 agent |
| **Kent Beck** | 设计判断力 | TDD、小步、**显式约束**＋人格提示 | 接受被禁用的测试、未要求的特性 | 有可测边界的设计问题 |
| **Addy Osmani** | 验证带宽（back pressure） | 代码分区（绿/黄/红）、characterization test、先研究后计划、review 前置、~100 行 PR、skills 固化流程 | dark factory、并行先行、同一 session 既写测试又让测试通过 | 棕地老库、大团队 |

## 2. 分歧点（保留原样）

### 2.1 瓶颈：上下文 vs 验证
- Cognition：「context engineering … effectively the **#1 job** of engineers building AI agents」（cognition.com/blog/dont-build-multi-agents，2025-06-12）〔一〕
- Addy：「**The bottleneck was never generation**」；「you can only hand a loop as much autonomy as you can cheaply and reliably verify, and **not one inch more**」（addyosmani.com/blog/software-factories/，2026-07-20）〔一〕
- 深一层对立：Cognition 要"尽量共享上下文"，其 2026 文却主张给 reviewer **干净上下文**（context rot）；Addy 则要求 reviewer 干净、写者单线程。同一词（context）指向相反操作。

### 2.2 单代理 vs 多代理
- Cognition 2025：违反两原则的架构「should **by default rule out**」；最简解法是单线程线性 agent〔一〕。
- Anthropic：多代理在内部研究评测超单 agent **90.2%**，但同文写明「most coding tasks involve fewer truly parallelizable tasks than research, and LLM agents are **not yet great at coordinating and delegating**」，且耗 token 约 **15×**（anthropic.com/engineering/multi-agent-research-system，2025-06 据 Cognition 脚注为 06-13）〔一〕。
- Cognition 2026：反转一半——「multi-agent systems work best today when **writes stay single-threaded**」；unstructured swarm 是「mostly a **distraction**」，可行形态是 map-reduce-and-manage（cognition.com/blog/multi-agents-working，2026-04-22）〔一〕。
- Yegge：20–30 个 Claude Code 并行，polecat 群 + Refinery 合并队列；「Gas Town is an **industrialized coding factory** manned by superintelligent robot chimps」（steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04，2026-01-01）〔二〕。
- Addy：「**Parallelize last**」；「Parallelism **multiplies the bottleneck** you already have」（2026-09-14）〔一〕。
→ 真分歧不在"能否并行"，而在**写操作能否并行**：Cognition/Addy 说不能，Yegge 用 merge queue 与 GUPP 硬扛。

### 2.3 规格先行 vs 对话对齐
- Addy：`/spec` 先行，把审阅从 2000 行 diff 前移到 **200 行计划**；「Process over prose. Workflows over reference.」（addyosmani.com/blog/agent-skills/，2026-05-03）〔一〕
- Simon：不要规格仪式，用「deceptively short prompt」+ 测试当规格，主抓手是红/绿 TDD（simonwillison.net/guides/agentic-engineering-patterns/，2026）〔二：仅核到目录〕
- Yegge：规格 = 可执行的 Beads 图（epic/molecule/hook），「**Do not use markdown TODO lists** for task tracking」（github.com/steveyegge/beads README，2026）〔一〕
- Kent Beck：模糊人格提示不够——「I couldn't just say "be me"; I had to **tell it what I would do**」（newsletter.kentbeck.com，2026-01-20）〔一〕
→ 三种"规格"互不等价：散文 spec、可执行测试、依赖图。

### 2.4 TDD 是否必需
- Simon / Addy：必需，且必须"先写失败测试"；Addy 的抗合理化表直答「"I'll write tests later." → There is **no later**」〔一〕。
- Addy 棕地补丁：不能让同一 session 既写测试又让它们通过，否则「a green suite that **encodes the implementation you just invented**」（2026-09-14）〔一〕——与"agent 自跑 red/green 循环"直接冲突。
- Anthropic：不校验过程，主张 **end-state evaluation** 而非逐步校验（2025-06）〔一〕。
- Böckeler：最低限度要盯的是测试代码（martinfowler.com/articles/exploring-gen-ai/i-still-care-about-the-code.html，2025-07-09）〔一〕。

### 2.5 能否 AFK / review 在哪
- Yegge：AFK 是目标。GUPP：「**If there is work on your hook, YOU MUST RUN IT**」；用 NDI（不确定的幂等）替代确定性重放〔二〕。
- Addy/Horthy：dark factory 实跑约 **4 个月无人读码**，最终栽在 comprehension debt；结论是 back pressure（2026-07-20）〔一〕。
- Cognition：review 由 clean-context agent 先做，平均每 PR 抓 **2 个 bug、58% 严重**，「most bugs are already resolved by the time a human opens the PR」（2026-04-22）〔一〕。
- Anthropic Claude Code 团队：「"did a human read the code?" is an **imprecise proxy**」，review 由终点闸门变成持续动作（claude.dev/blog/solving-code-review-at-anthropic，2026-10-16，经 Simon 转引）〔二〕。
- Simon 自述：「I'm **not reviewing every line** of code that they write anymore, even for my production level stuff」，并自警这是 normalization of deviance（simonwillison.net/2026/May/6/…，2026-05-06）〔一〕。
→ review 位置四种答案：人前置（Addy）、agent 前置（Cognition）、持续化（Anthropic）、名义保留实际后撤（Simon）。

### 2.6 组织结构：谁在管代理
- Yegge：把代理当**有岗位的工人**——Mayor（接线）、Polecat（临时工）、Refinery（合并队列）、Witness（救卡住的人）、Deacon（巡检）。「You are a **Product Manager**, and Gas Town is an Idea Compiler」〔二〕。
- Addy：工厂「is not a bigger agent; it is an **org chart made of loops**」，人退到 outer loop 持有否决权（2026-07-20）〔一〕。
- Cognition：manager Devin 拆任务、经内部 MCP 协调 child，「managers trained on small-scoped delegation default to being **overly prescriptive**」（2026-04-22）〔一〕。
- HumanLayer：把"组织"降级为软件问题——人类只是**一个工具调用**（factor 7），控制流自己持有（factor 8）〔一〕。
→ Yegge 要"造组织"，HumanLayer 要"消灭组织"，Addy 要"保留组织的否决点"。三种组织观不可同时成立。

### 2.7 安全是否算交付问题
- Simon：lethal trifecta = 私有数据 + 不可信内容 + 对外通信；「**Guardrails won't protect you**」，唯一稳妥是整组避开（simonwillison.net/2025/Jun/16/the-lethal-trifecta/，2025-06-16）〔一〕。
- 其余六家基本不把 prompt injection 列为交付阻塞项〔推：据各家主线文本判断〕。

## 3. 可引用原话
1. 「Actions carry implicit decisions, and **conflicting decisions carry bad results**」— Cognition，2025-06-12〔一〕
2. 「**Guardrails won't protect you**」— Simon，2025-06-16〔一〕
3. 「The human is removed as the bottleneck for **90-99%** of the work」— Yegge，2025-03-22〔一〕
4. 「Agents skip those steps … because they're **invisible**」— Addy，2026-05-03〔一〕
5. 「most of the products … billing themselves as "AI Agents" are **not all that agentic** … mostly deterministic code, with LLM steps sprinkled in」— HumanLayer〔一〕
6. 「**LLMs are NOT compilers** … they are **inferrers**」「Hallucinations are the core feature」— Böckeler，2025-07-09〔一〕
7. 「Personas Drive **Micro-Behavior**；Constraints Drive **Macro-Architecture**」— Kent Beck，2026-01-20〔一〕
8. 「**The bottlenecks have shifted** … the entire SDLC was designed around a day per few hundred lines」— Simon，2026-05-06〔一〕

## 4. 适用条件速查
- **团队 <5 人 / 新库**：Simon、Kent Beck 路径成本最低；Yegge 明确"Stage 6 以下用不了"〔二〕。
- **棕地 10 年+ 库**：Addy 的分区 + characterization test 是唯一给出可操作前置条件的（2026-09-14）〔一〕〔推：唯一性为本文判断〕；Anthropic 自认编码平行度低，反而与此一致。
- **研究/广度任务**：只有 Anthropic 的结论可直接外推，且它自己划了"不适用编码"的界〔一〕。
- **产品型 agent（卖给客户）**：HumanLayer 的 12 factor 是唯一以"生产可用"为判据的清单〔一〕。

## 5. 未核实 / 缺口
- Gas Town 原文托管在 Medium，本次仅取得镜像转录〔二〕，架构细节（GUPP/NDI/Convoy）未逐字核。
- Yegge「death of the junior developer」的一手为 Sourcegraph 2025-03-22《Revenge of the Junior Developer》〔一〕；同名书本身未取到。
- Simon 红/绿 TDD 章节正文抓取多次超时，只有目录与相邻章节〔二〕；HumanLayer 12 factor 仅核 README 与 factor 3/7/10 标题。
- Addy Osmani 的 spec-driven development 专文未定位到，spec 立场依据 Agent Skills（2026-05-03）与棕地文〔推：两文合并判断〕。
