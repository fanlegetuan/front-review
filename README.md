# front-review

Read-only Agent Skill for reviewing frontend PRs, diffs, modules, or AI-generated UI against project engineering specs, a Preferred stack, Radix UI Colors, UI copy/layout norms, state/URL/cache sync, permission resilience, and performance.

**Darwin score:** 97.2 (`full_test`, 3-0 paired ratchet)

**Preferred stack:** shadcn/ui (Nova) / Lucide / Motion / TanStack Router & Query / react-resizable-panels / dnd-kit / Zustand / [Radix UI Colors](https://www.radix-ui.com/colors).

## Install

Copy `SKILL.md` (and optional `test-prompts.json`) into your agent skills folder, e.g. `.agents/skills/front-review/`, `.cursor/skills/front-review/`, or `~/.gemini/config/skills/front-review/`.

## When to use

Frontend / UI PR review, module or repo-level frontend code audit, AI-generated page review, stack or UI-style drift checks. Triggers: `front-review`, frontend review, UI PR review, component PR review, Radix colors review, UI样式规范, 前端代码审查, 代码审查.

## What it enforces (7 Dimensions)

1. **Dependencies & Stack:** reuse over overlapping UI / icon / animation / router / state libraries; single package manager & lockfile hygiene
2. **Design & States:** all 6 UI states (`default | loading | empty | error | disabled | feedback`), keyboard / a11y / long-text handling, no emoji-as-icons
3. **UI Style (copy · color · layout):** Radix UI Colors for all color usage; no middle-dot `·` separators, no filler helper text, no raw backend enum leaks, no duplicate Eyebrow/Overline; optical alignment, concentric radii, `0.5px` borders, ≤4 type sizes (`>=12px`), restrained motion (`ease-out` ~`250ms`)
4. **State, URL & Cache Sync:** validated URL search params (`validateSearch` clamping & bidirectional filter sync), sliding pagination window (`>7` pages), complete `invalidateQueries` on mutations, dirty-form close guard (`DiscardChangesDialog`)
5. **Auth & Permission Resilience:** `PermissionGate` coverage, explicit forbidden feedback, transient permission query error retry recovery
6. **Performance & Architecture:** `Promise.all` over sequential `for...await` waterfalls, `useMemo` for `>100KB` payload decoding/tokenization and `>200` item lists, `300`-line route/component split threshold
7. **Verification:** zero green-build cheats (`@ts-ignore`, `eslint-disable`, deleted tests/features), dual-mode support (`Repo mode` & `Inline snippet mode`)

## Files

| File | Role |
| --- | --- |
| `SKILL.md` | Skill recipe |
| `test-prompts.json` | Darwin evaluation & regression prompts |

## License

MIT
