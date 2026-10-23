# front-review

Agent Skill for reviewing frontend PRs and AI-generated UI against a fixed stack and design bar.

**Score (Darwin triage):** 90.0 · Prefer stack: shadcn/ui (Nova) · Lucide · Motion · react-resizable-panels · dnd-kit · Zustand · design tokens.

## Install

Copy `SKILL.md` (and optional `test-prompts.json`) into your agent skills folder, e.g. `.cursor/skills/front-review/` or your runtime’s skills directory.

## When to use

Frontend / UI PR review, AI-generated page review, stack drift checks. Triggers: `front-review`, frontend review, UI PR review, 前端代码审查.

## What it enforces

- Reuse over new overlapping UI / icon / animation / state libraries
- Preferred stack table (when project-compatible)
- Required UI states: default, loading, empty, error, disabled, feedback
- No green-build cheats (`@ts-ignore`, silenced lint, deleted tests)
- Structured output: Verdict · Findings · Checklist scores

## Files

| File | Role |
| --- | --- |
| `SKILL.md` | Skill recipe |
| `test-prompts.json` | Optional Darwin / regression prompts |

## License

MIT
