# front-review

Agent Skill for reviewing frontend PRs and AI-generated UI against a fixed stack, Radix UI Colors, and UI copy/layout norms.

**Darwin score:** 90.0 (`full_test`)

**Prefer stack:** shadcn/ui (Nova) / Lucide / Motion / react-resizable-panels / dnd-kit / Zustand / [Radix UI Colors](https://www.radix-ui.com/colors).

## Install

Copy `SKILL.md` (and optional `test-prompts.json`) into your agent skills folder, e.g. `.cursor/skills/front-review/`.

## When to use

Frontend / UI PR review, AI-generated page review, stack or UI-style drift checks. Triggers: `front-review`, frontend review, UI PR review, UI样式规范, 前端代码审查.

## What it enforces

- Reuse over overlapping UI / icon / animation / state libraries
- Radix UI Colors for all color usage
- Copy: no middle-dot separators, no filler helper text, no decorative Eyebrow/Overline
- Layout: optical alignment, concentric radii, `0.5px` borders, ≤4 type sizes (min 12px), restrained motion (`ease-out` ~250ms)
- No green-build cheats
- Structured output: Verdict / Findings / Checklist scores (includes **UI style**)

## Files

| File | Role |
| --- | --- |
| `SKILL.md` | Skill recipe |
| `test-prompts.json` | Optional Darwin / regression prompts |

## License

MIT
