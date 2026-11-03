# AI Project Delivery

> A cross-harness methodology skill for delivering software projects with AI coding agents, including a mode for non-technical users.

This repository is an **Agent Skill** (`SKILL.md` at the root) for DeepSeek Harness, Claude Code, Codex, Cursor and other Agent Skills compatible runtimes. It turns "ship a project with an AI agent" into staged workflows, mental models and decision rules, while keeping genuine disagreements between schools visible instead of flattening them.

- **What it is**: a methodology skill plus three traceable research notes (55 sources, 80%+ primary)
- **What it is not**: another spec-driven tool, a vendor-locked process, or a persona role-play
- **Who it is for**: anyone building with coding agents, including non-technical users (a dedicated non-coder mode is built in)

## Install

```bash
# Generic skills installer
npx skills@latest add anjiecobamboo-blip/ai-project-delivery

# DeepSeek Harness
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$HOME/.dsh/skills/ai-project-delivery"

# Codex
git clone https://github.com/anjiecobamboo-blip/ai-project-delivery.git "$env:CODEX_HOME/skills/ai-project-delivery"
```

The skill auto-triggers on delivery-flow questions (spec or not, ticket slicing, context switching, multi-agent parallelism, AFK runs, AI code review, tool choice). It deliberately does **not** trigger on a single coding task, a single debugging session, or a single diff review.

## Contents

| Section | What it covers |
|---|---|
| Four failure modes | misalignment, missing shared language, broken feedback loops, ball-of-mud |
| Diagnosis table | alignment / context / verification / architecture layers |
| 8-stage flow | setup, align, prototype, spec, tickets, implement, review, ship, retro |
| 6 mental models | verification bandwidth, context budget, alignment-as-asset, deep modules, accelerators vs brakes, cold-context review |
| 10 heuristics | session splitting, ticket size, parallelism conditions, stop signals, where rules live |
| 10 anti-patterns | vibe coding, big PRs, self-review, automation without brakes, and more |
| Disagreement map | spec as input vs compression, context scarcity vs abundance, multi-agent writes, AFK, TDD, review placement, org design, security |
| Non-technical user mode | question limits, demo-based acceptance, safety nets, fixed phrasing |

## Relationship to other frameworks

It absorbs GitHub Spec Kit, OpenSpec, BMAD-METHOD, Matt Pocock's pipeline and practitioner schools (Simon Willison, Cognition, Addy Osmani, Kent Beck, Steve Yegge, HumanLayer), and treats them as schools with conditions rather than answers.

## Structure

```
SKILL.md                  # the skill body
agents/openai.yaml        # Codex display metadata
references/research/      # sourced research notes
cordis.patch.yml          # DeepSeek Harness plugin entry
package.json
```

## Scope and limitations

Research date 2026-11-03. Evidence is mostly practitioner experience and vendor self-report; no controlled studies. Numbers are cited with dates, and this skill marks its own heuristics as estimates. Harness capabilities differ; mappings are provided but not equivalent.

## License

MIT. See [LICENSE](./LICENSE).

> Generated with the [Nuwa skill-distillation workflow](https://github.com/alchaincyf/nuwa-skill). Author: `anjiecobamboo-blip`.
