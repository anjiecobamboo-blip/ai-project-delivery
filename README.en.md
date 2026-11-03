# AI Project Delivery

> A cross-harness methodology skill for shipping software with AI coding agents, including a non-technical user mode.

This repository is an **Agent Skill** (`SKILL.md` at the root) for DeepSeek Harness, Claude Code, Codex, Cursor and other Agent Skills compatible runtimes. It turns "deliver a project with an AI agent" into staged workflows, mental models and decision rules, while keeping genuine disagreements between schools visible instead of flattening them.

- **What it is**: a methodology skill plus three traceable research notes (55 sources, 80%+ primary)
- **What it is not**: another spec-driven development tool, a vendor-locked process, or a persona role-play
- **Who it is for**: people building with coding agents, including **non-technical users** (a dedicated mode is built in)

## Why this exists: a decision layer, not another tool

Most AI development frameworks answer "how". This repository answers "**which one, and when**".

| Dimension | Common approach | This repository |
|---|---|---|
| **Stance** | Sells one process (SDD, role-play, fully autonomous) | Keeps the disagreements, states each school's conditions and failure boundaries |
| **Audience** | Assumes you can read code and judge designs | Built-in non-technical user mode: at most 3 questions, demo-based acceptance, safety nets |
| **Carrier** | Locked to one IDE or vendor | One source across DeepSeek Harness, Claude Code and Codex, with equivalent mappings |
| **Evidence** | Slogans and assertions | 55 sources with URLs and dates; estimates marked as such; claims traceable and falsifiable |

**In one line**: if you want to execute, install [Spec Kit](https://github.com/github/spec-kit) or [BMAD](https://github.com/bmad-code-org/BMAD-METHOD); if you want to know **when to use them and when not to**, use this.

**Good for**:

- People shipping full projects with coding agents who keep hitting rework, loss of control, or a codebase that gets harder to change
- Non-technical users who only work through AI (dedicated mode)
- Teams that want one consistent process across multiple agent clients

**Not for**:

- Those who want an out-of-the-box automation pipeline (this is methodology, it does not execute for you)
- A single coding task, a single debugging session, or a single diff review (use more specific skills)
- Organisations that need compliance and audit templates (none are included)

**How to combine it**: treat this as a routing layer. Use its criteria to choose a process and its granularity, then hand execution to Spec Kit, BMAD, superpowers, or the skills you already have.

## Install

```bash
# Generic skills installer
npx skills@latest add anjiecobamboo-blip/ai-project-delivery
```

```powershell
# DeepSeek Harness
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$HOME\.dsh\skills\ai-project-delivery"

# Codex
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$env:CODEX_HOME\skills\ai-project-delivery"

# Claude Code
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$HOME\.claude\skills\ai-project-delivery"
```

DeepSeek Harness discovers the root `SKILL.md` automatically and hot-reloads; Codex picks it up in a new session.

## When it triggers

**Triggers** (questions about how to run a project):

- Should we write a spec? How should tickets be sliced? When do we switch sessions?
- Can multiple agents write in parallel? Can an agent run overnight (AFK)?
- How should AI-written code be reviewed? Spec Kit vs Kiro vs plain conversation?
- How do we adopt this on a greenfield or a legacy codebase? How do we build a software factory?

**Does not trigger** (to avoid slowing down small tasks):

- A single coding task
- A single debugging session
- A single diff or PR review
- A single parallel task dispatch

## Contents

| Section | What it covers |
|---|---|
| Four failure modes | misalignment, missing shared language, broken feedback loops, ball-of-mud |
| Diagnosis table | alignment / context / verification / architecture layers |
| 8-stage flow | setup, align, prototype/research, spec, tickets, implement, review, ship, retro |
| 6 mental models | verification bandwidth, context budget, alignment-as-asset, deep modules, accelerators vs brakes, cold-context review |
| 10 heuristics | session splitting, ticket size, parallelism conditions, stop signals, where rules live |
| 10 anti-patterns | vibe coding, big PRs, self-review, automation without brakes, and more |
| Disagreement map | spec as input vs compression, context scarcity vs abundance, multi-agent writes, AFK, TDD, review placement, org design, security |
| Non-technical user mode | question limits, demo-based acceptance, safety nets, fixed phrasing |
| Harness mappings | DeepSeek Harness / Claude Code / Codex equivalents |

## Relationship to other frameworks

| Project | Positioning | Difference |
|---|---|---|
| [github/spec-kit](https://github.com/github/spec-kit) | Official SDD toolkit | Treats SDD as one school, not the default answer |
| [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) | SDD for AI assistants | Adds conditions for when spec is input vs. compression |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | Role-based agile AI development | No role-play required; flow is trimmed to task size |
| [obra/superpowers](https://github.com/obra/superpowers) | Skills framework and methodology | A single-topic methodology document that composes with its skills |
| [mattpocock/skills](https://github.com/mattpocock/skills) | Engineering skill collection | Absorbs its pipeline and adds other schools plus counterexamples |

## Non-technical user mode

When the user says they cannot code and only work through AI, the skill switches to:

- At most 3 questions at a time; defaults over interviews
- No code shown; every technical term explained with an analogy first
- Every delivery is demonstrable (screenshot, recording, or a runnable artifact) plus at most 3 acceptance checks the user can judge themselves
- Checkpoints pause for confirmation: requirement summary, spec, first slice, each round of acceptance
- Payments, accounts, data deletion and public deployment require an explicit risk explanation and consent

## Structure

```
.
├── SKILL.md                      # the skill body (auto-discovered after install)
├── agents/openai.yaml            # Codex display metadata
├── references/research/          # sourced research notes (URLs, dates, primary/secondary)
│   ├── 01-pocock-pipeline.md
│   ├── 02-vendor-spec-driven.md
│   └── 03-practitioner-schools.md
├── cordis.patch.yml              # DeepSeek Harness plugin entry
├── package.json
├── README.md / README.en.md
├── CHANGELOG.md
└── LICENSE
```

## Sources and acknowledgements

Built with the [Nuwa skill-distillation workflow](https://github.com/alchaincyf/nuwa-skill), synthesising public material from (full list of 55 sources with dates in `references/research/`):

- Matt Pocock / AI Hero: grill → spec → ticket DAG → implement → independent review → retro; smart zone, deep modules, software-factory
- GitHub Spec Kit, AWS Kiro, Anthropic docs: spec-driven development, context engineering, deterministic checks
- Simon Willison, Cognition, Addy Osmani, HumanLayer, Kent Beck, Steve Yegge, Böckeler: verification bandwidth, multi-agent boundaries, AFK counterexamples, 12-factor agents

Thanks to all authors and maintainers. This repository is not affiliated with or endorsed by them; every citation is attributed.

## Limitations

- **Freshness**: research date 2026-11-03; vendor positions in this field have reversed more than once within a year
- **Evidence level**: mostly practitioner experience and vendor self-report, with almost no controlled studies; skill-internal thresholds are marked as estimates
- **Harness differences**: multi-agent support, hooks and auto-compaction differ across harnesses; mappings are provided but not equivalent
- **Scale bias**: strongest evidence for small teams and greenfield work; large-team coordination, compliance and cost models are largely uncovered

## Contributing

Issues and PRs are welcome for factual corrections, new sources, and sharper decision criteria. Two rules:

1. Attach a source (URL + date) to new claims, and separate primary from secondary;
2. Do not flatten disagreements: conflicting evidence should be presented side by side with its conditions.

## License

[MIT](./LICENSE)

> Generated with the [Nuwa skill-distillation workflow](https://github.com/alchaincyf/nuwa-skill). Author: anjiecobamboo-blip.
> If you redistribute this work, keep the attribution and licence notice.
