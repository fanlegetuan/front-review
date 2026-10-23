---
name: front-review
description: >-
  Use when reviewing frontend PRs, AI-generated UI, or component changes against
  project stack, Radix UI Colors, and UI copy/layout norms (shadcn/Nova, Lucide,
  Motion, Zustand). Triggers: front-review, frontend review, UI PR review,
  UI样式规范, 前端代码审查.
---
# front-review

Review frontend diffs against project standards. Catch stack drift, design inconsistency, UI-style drift, and “build green” cheats.

## Inputs

| Input | Required | If missing |
| --- | --- | --- |
| Diff / PR / files | Yes | 🔴 STOP — ask for diff/PR URL; invent nothing |
| `package.json` | Yes | Read repo root; unreadable → Fail Verification + Request changes |
| Shared components / tokens / dirs | Preferred | Orient from what exists; unknown reuse = Warning |

## Review flow

### 1 Orient
- **In:** layout, `package.json`, shared UI paths, color/token setup
- **Out:** one-line stack note (framework, pkg mgr, UI/style, colors, shared components)
- No unrelated refactors.

🔴 **CHECKPOINT:** If `package.json` or primary style/token file is unreadable → STOP; report missing paths; do not guess Preferred stack.

### 2 Score
- **In:** diff + Orient note
- **Out:** Pass/Warn/Fail per checklist row with `path:line` or quote evidence

### 3 Verdict
- **In:** scores + failure-table hits
- **Out:** Approve | Request changes | Block + Findings template
- Fix root cause; no workarounds-as-approval.

🛑 **STOP before Approve:** any Blocker, silenced types/lint/tests, or features deleted only for CI → Block or Request changes.

## Preferred stack (project-compatible)

Import only libraries the page needs. Flag conflicting imports unless the PR documents a forced exception.

| Concern | Prefer |
| --- | --- |
| UI | shadcn/ui + Nova preset |
| Icons | Lucide |
| Animation | Motion |
| Resizable panes | react-resizable-panels |
| Drag & drop | dnd-kit |
| Global state | Zustand |
| Colors | [Radix UI Colors](https://www.radix-ui.com/colors) + their usage scale (bg/text/border steps) |
| Styles | Existing tokens; Tailwind maps to Radix scales for color/font/space/radius/shadow |

## Checklist → evidence

**Dependencies:** reuse mature/business components; justify new deps; reject overlapping UI/icon/animation/state libs; match framework/dirs/norms/pkg mgr; no drive-by refactors.

**Design:** tokens match product; extend shared components; no emoji-icons / mixed icon families; states = default|loading|empty|error|disabled|feedback; responsive + keyboard + a11y + long text.

**UI style (copy · color · layout):**
1. No middle-dot `·` as copy separator — remove if present
2. No redundant explanatory / helper prose that dirties the UI
3. Copy fits the audience: clear, not overly colloquial, length matches the control; no coined jargon
4. All colors from Radix UI Colors; follow scale roles (background / text / border / accent)
5. Optical alignment; nested corners use concentric radii
6. Borders / dividers at `0.5px` when lines are needed
7. Minimize motion; if required → `ease-out` ~`250ms` baseline
8. ≤4 type sizes (prefer 3); min `12px`; hierarchy via weight / color / spacing, not more sizes
9. Minimize judgment-required choices; put required choices where the user already decides; reorder by information priority
10. No casual radius / shadow / colored left-edge card stripes; if used, values must match the scene
11. No decorative Eyebrow / Overline that repeats the title; keep only when it adds category/context the title lacks; prefer type/weight/space for hierarchy

**Code quality:** single responsibility; local React state first, Zustand only if shared; mature libs for complex UX; no cosmetic-only heavy deps; no `@ts-ignore` / eslint-disable / deleted tests as green-build.

**Verification:** author checked package/components/style/dirs; planned structure/states/reuse; typecheck+lint+tests+build (or CI); smoke layout/interaction/responsive/console; root-cause fixes (not reinstall / one-off CSS overrides).

## Failure modes (trigger → first fix → still fails)

| Trigger | First fix | Still fails → |
| --- | --- | --- |
| No diff/PR | Ask URL/diff | 🔴 STOP; empty review |
| `package.json` unreadable | Retry repo-root path | Fail Verification; Request changes |
| Overlapping new dep (2nd UI/icon/motion/state) | Cite Prefer table + existing pkg | Blocker: drop or document exception |
| Emoji / mixed icons | Require Lucide (or project set) | Fail Design |
| Happy-path only | List missing states as Blockers | Block until states exist |
| `@ts-ignore` / eslint-disable / deleted tests for CI | Real type/lint/test fix | 🛑 Block — green-build cheat |
| Zustand for local-only UI | `useState`/`useReducer` | Request changes if duplicates existing store |
| Unrelated refactor in PR | Flag out-of-scope hunks | Request changes: split PR |
| Stack conflict + documented exception | Accept; Warning note | Undocumented → Request changes |
| Hardcoded non-Radix colors | Map to Radix scale steps | Fail UI style |
| Middle-dot `·` separators / redundant helper copy / decorative Eyebrow | Remove or rewrite | Fail UI style (copy) |
| >4 type sizes or text `<12px` | Collapse to ≤3–4 sizes; min 12px | Fail UI style (type) |
| Casual heavy shadow / left-edge color bar / random radius | Remove or match scene tokens | Warn or Fail UI style |
| Extra motion without need | Drop or set `ease-out` 250ms | Warn UI style |

## Finding line examples (copy shape)

- Blocker: `src/settings/Drawer.tsx:12` — imports `react-icons` while project uses Lucide — replace with `lucide-react`
- Blocker: `src/list/Page.tsx` — no loading/empty/error UI — add the three states before merge
- Blocker: `src/theme/colors.ts` — hardcoded `#FF5500` — map to Radix accent scale
- Warning: `src/page/Header.tsx:8` — Eyebrow `OVERVIEW` repeats title — remove
- Warning: `src/card/Item.tsx` — `1px` border + left accent stripe — use `0.5px` border; drop stripe unless required
- Nit: `src/pane/Split.tsx:40` — magic `w-[320px]` — map to spacing token

## Resources

| Path | Role |
| --- | --- |
| This `SKILL.md` | Sole required recipe |
| `test-prompts.json` (same folder) | Optional Darwin / regression prompts |
| [Radix UI Colors](https://www.radix-ui.com/colors) | Color scales + usage |

No scripts or assets. Do not invent external reference paths.

## Anti-patterns (do not)

- Invent paths/findings absent from the diff
- Approve while Blockers remain
- Propose a second UI/icon/animation/state library when one exists
- Treat CI green as enough if types/tests silenced or features deleted
- Swap formal icons for emoji
- Use middle-dot `·` separators or filler helper text
- Add decorative Eyebrow/Overline that only restates the title
- Mix ad-hoc hex/rgb outside Radix scales
- Drop in a second design system “for consistency”
- Rewrite unrelated modules “while here”
- Skip Orient and jump to nits

## Output

```markdown
## Verdict
Approve | Request changes | Block

## Summary
1–3 sentences on merge fitness.

## Findings
### Blockers
- `path:line` — issue — required fix
### Warnings
- `path:line` — issue — suggested fix
### Nits
- polish only

## Checklist scores
| Dimension | Result | Notes |
| --- | --- | --- |
| Dependencies | Pass/Warn/Fail | |
| Preferred stack | | |
| Design consistency | | |
| UI style | | |
| Code quality | | |
| Verification | | |
```

Language: match requester (中文/English); keep Verdict labels and `path:line` stable.

## Self-check before send
1. Every Fail has a Blocker or explicit Request-changes item
2. Preferred-stack and UI-style conflicts cited with evidence
3. No Approve if 🛑 STOP conditions hit
