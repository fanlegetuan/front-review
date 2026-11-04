---
name: front-review
description: >-
  Read-only frontend code & UI reviewer for PRs, diffs, modules, or AI-generated
  pages against project engineering specs, Preferred stack (shadcn/Nova, Lucide,
  Motion, Zustand, TanStack Router/Query), Radix UI Colors, UI copy/layout norms,
  state/URL/cache sync, permission resilience, and performance. Do not trigger
  for pure backend/infra repos. Triggers: front-review, frontend review, UI PR
  review, component PR review, Radix colors review, UI样式规范, 前端代码审查,
  代码审查.
---
# front-review

Review frontend diffs, PRs, or target modules against project engineering standards and UI/design norms without modifying source files unless asked. Catch stack drift, UI-style drift, state/URL/cache desync, permission holes, serial request waterfalls, oversized route files, and "build green" cheats.

## Inputs

| Input | Required | If missing |
| --- | --- | --- |
| Diff / PR / target path / inline PR summary | Yes | 🔴 STOP — ask for diff, PR URL, target module path, or inline PR summary; invent nothing |
| `package.json` | Repo mode: Yes / Inline mode: Optional | Repo mode: check root then `apps/*/package.json` & `packages/*/package.json`; unreadable → Fail Verification + Request changes. Inline mode: fall back to `Preferred stack` defaults |
| Project specs (`docs/开发规范.md`, `AGENTS.md`, `CLAUDE.md`) | Preferred | Read if present; project rules override default thresholds (`300` lines, `100KB` payload, `200` items). If absent, note `Spec: default` in `Orient` |
| Shared components / tokens / dirs | Preferred | Orient from `src/components/`, `tailwind.config.*`, `src/styles/*.css`; re-implementing existing `src/components/` component = `Warning` |

## Review flow

### 1 Orient
- **In:** target scope (diff, directory, or inline snippet), `package.json`, project spec docs (`docs/开发规范.md`, `AGENTS.md`), shared UI paths, color/token setup
- **Out:** one-line stack & spec note (`Context: repo | inline-snippet`, framework, router/query/state, pkg mgr + lockfile rule, UI/style, colors, file-size limit, shared components)
- Project spec rules take precedence over default Preferred stack when explicitly documented; flag undocumented stack conflicts.
- Perform read-only review; no unrelated refactors.

🔴 **CHECKPOINT:** In **Repo mode**, if `package.json` (root and workspace packages) or primary style/token file is unreadable → STOP and report missing paths. In **Inline snippet/summary mode** (no local repo), set `Context: inline-snippet (Preferred stack defaults)` and proceed without inventing file paths.

### 2 Audit & Score
- **In:** diff/files/snippet + Orient note
- **Out:** `Pass | Warn | Fail` for each of the 7 checklist dimensions with exact `path:line` evidence (or `<scope>:[inline]` when reviewing line-less inline summaries)
- **Directory / Module Audit Order (when input is a folder/repo rather than a single PR diff):** scan in priority order `src/routes/**` → `src/components/**` → `src/lib/** & src/hooks/**` → `tests/**`.
- **Deterministic Severity → Dimension Score Mapping:**
  - `Fail`: dimension has `>=1 Blocker`
  - `Warn`: dimension has `>=1 Warning` and `0 Blockers`
  - `Pass`: dimension has `0 Blockers` and `0 Warnings` (`Nit`s allowed)
- **Notes Completeness:** each row's `Notes` cell in `Checklist scores` MUST list all matched issues in that dimension (both `Blocker`s and `Warning`s, e.g., recording a `1px` border `Warning` alongside UI Style `Blocker`s).

### 3 Verdict
- **In:** 7 dimension scores + findings list
- **Out:** `Approve | Request changes | Block` + Findings grouped by severity (`Blockers` / `Warnings` / `Nits`) + 7-row Checklist table
- **Deterministic Verdict Rule:**
  - `Block`: any `Blocker` in Verification (green-build cheat), Dependencies/Lockfile, Permission Resilience, State/URL/Cache Sync, missing core UI states, emoji icons, or hardcoded non-Radix hex/rgb colors
  - `Request changes`: `0 Blockers`, but `>=2 Warnings` across engineering/UI dimensions or undocumented stack exception
  - `Approve`: `0 Blockers` and `<=1 Warning` (with `Nit`s listed for polish)

🛑 **STOP before Approve:** any `Fail` dimension (`>=1 Blocker`), silenced types/lint/tests, or features deleted only for CI → output `Block` or `Request changes`.

🔴 **CHECKPOINT:** Every `Fail` dimension MUST map to `>=1 Blocker` with `path:line` (or `<scope>:[inline]`) evidence; a dimension with only `Warning`s MUST be scored `Warn`, never `Fail`.

## Preferred stack (project-compatible)

Respect existing dependencies in `package.json` and project specs first. When adding new capabilities or reviewing greenfield UI, enforce:

| Concern | Prefer |
| --- | --- |
| UI primitives | shadcn/ui + Nova preset (or project's established `src/components/ui/`) |
| Icons | Lucide (`lucide-react`) |
| Animation | Motion (`motion`) |
| Routing & URL state | TanStack Router (`@tanstack/react-router`) with validated search params |
| Server state & cache | TanStack Query (`@tanstack/react-query`) with explicit query keys & invalidation |
| Global client state | Zustand (only when cross-component; local `useState`/`useReducer` first) |
| Resizable panes / DnD | `react-resizable-panels` / `dnd-kit` |
| Colors | [Radix UI Colors](https://www.radix-ui.com/colors) + semantic scale steps (bg / text / border / accent) |
| Styles | Project tokens + Tailwind mapped to Radix scales for color/font/space/radius/shadow |

## Checklist → evidence (7 Dimensions)

### 1. Dependencies & Stack
- **Blocker:** introducing a 2nd UI, icon, animation, router, or state library when one exists in `package.json`; duplicating an existing global Zustand store; committing a conflicting lockfile (e.g., `bun.lock` or `yarn.lock` in an `npm` + `package-lock.json` repo).
- **Warning:** re-implementing a component or hook that already exists in `src/components/` or `src/hooks/`; documented stack exception.

### 2. Design & States
- **Blocker:** emoji used as functional icons; mixing icon families; missing any required interactive state (`default | loading | empty | error | disabled | feedback`); interactive control missing keyboard focus/Escape handling, ARIA label on icon-only buttons, or text overflow handling (`truncate` / `break-words`).

### 3. UI Style (copy · color · layout)
- **Blocker:**
  1. Middle-dot `·` used as copy separator.
  2. Redundant filler helper prose that restates the label.
  3. Raw backend enum codes (e.g., `LOCAL`, `PENDING_SYNC`) leaked into localized UI when localized labels apply.
  4. Hardcoded hex/rgb/hsl colors outside Radix UI Colors or project semantic tokens.
  5. Decorative Eyebrow / Overline that repeats the title without adding distinct category context.
  6. `>4` font sizes in a single view or any text `<12px`.
- **Warning:**
  7. `1px` structural borders/dividers where `0.5px` hairline border is specified; non-concentric nested corner radii (`outer_radius != inner_radius + padding`).
  8. Decorative left/top card accent stripes (allowed only on semantic `error | warning | info | success` alerts or active nav indicators) or heavy ad-hoc box shadows.
  9. Animation duration `>300ms` or non-`ease-out` curve for standard UI transitions.
  10. `>1` primary CTA button in the same view/modal, or destructive action lacking confirmation / danger styling.

### 4. State, URL & Cache Sync
- **Blocker:**
  - Route `validateSearch` fails to clamp `page >= 1`, restrict `pageSize` to whitelist, or validate filter enums.
  - Controlled filter inputs fail to sync when URL search params change via browser back/forward or reset.
  - Pagination renders all page buttons without a sliding window when total pages `>7`.
  - `useMutation` omits `invalidateQueries` for affected sibling queries (e.g., updating user roles without invalidating both `['users']` and `['roles']`).
  - Editable form drawer/modal closes on backdrop/Escape/Cancel without checking dirty state (`DiscardChangesDialog`).

### 5. Auth & Permission Resilience
- **Blocker:**
  - Protected route or action missing `PermissionGate` / `usePermission` guard, or rendering a blank screen instead of explicit forbidden feedback.
  - Permission or bootstrap query caches transient network errors (`isError`) without allowing retry on subsequent navigation.

### 6. Performance & Architecture
- **Warning:**
  - Sequential `for...await` waterfall fetching multiple independent pages or endpoints instead of bounded `Promise.all`.
  - Un-memoized (`useMemo`) Base64 decoding, `JSON.parse`, regex syntax tokenization on payloads `>100KB`, or un-memoized filtering/sorting on lists `>200` items.
  - Route (`src/routes/**`) or component file exceeds `300` lines (or project spec limit in `docs/开发规范.md`) by mixing sub-tables, dialogs, or pure helpers in the route file.
- **Blocker:** infinite render/effect loop (`useEffect` updating its own dependency without guard) or memory leak (uncleaned timer/listener/WebSocket).

### 7. Verification
- **Blocker (🛑 Green-Build Cheat):** `@ts-ignore`, unjustified `@ts-expect-error`, `eslint-disable` masking real errors, deleted/skipped tests, or deleted product features to pass CI; failing `tsc --noEmit`, `eslint`, `vitest`, or build.

## Failure modes (trigger → first fix → still fails)

| Trigger | First fix | Still fails → |
| --- | --- | --- |
| No diff, PR, target path, or inline summary | Ask user for PR URL, diff, target module path, or snippet | 🔴 STOP; emit empty review |
| Root `package.json` missing or lacks frontend deps (Monorepo) | Scan `apps/*/package.json` and `packages/*/package.json` | In Repo mode: Fail Verification + Request changes; in Inline mode: use `Preferred stack` defaults |
| Inline PR summary/snippet lacks file line numbers | Set `Context: inline-snippet` in `Orient` and cite `<scope>:[inline]` | Never invent line numbers; if snippet lacks context to judge a rule, mark `Pass (not applicable in snippet)` |
| `docs/开发规范.md` or custom token file absent | Fall back to `tailwind.config.*` / `src/styles/*.css` and default thresholds (`300` lines, `100KB`, `200` items) | Note `Spec: default` in `Orient` and continue |
| Read-only sandbox or missing `node_modules` blocks running `tsc`/`vitest` | Statically audit diff for `@ts-ignore`, `eslint-disable`, broken types, or deleted tests | Note `Verification: static-only` in table; if cheat found → 🛑 Block: Fail Verification |
| Oversized module or truncated diff (`>20` files) | Audit in priority order: `src/routes/**` → `src/components/**` → `src/lib/** & src/hooks/**` → `tests/**` | State audited file count/scope explicitly in `Orient` |
| Overlapping new dependency or stray lockfile (`bun.lock`) | Cite Prefer table + single lockfile rule | Blocker: Fail 1. Dependencies & Stack |
| Emoji icons, missing loading/empty/error states, or missing a11y/overflow | List exact missing states/attributes with `path:line` | Blocker: Fail 2. Design & States |
| Non-Radix hex/rgb, `·` separator, raw enum leak, duplicate Eyebrow, or `<12px` font | Map to Radix tokens / fix copy / collapse to `<=4` sizes (`>=12px`) | Blocker: Fail 3. UI Style (also record any `1px` border/stripe `Warning` in Notes) |
| Unvalidated URL search params, missing `invalidateQueries`, or unguarded dirty drawer | Clamp `validateSearch`, add query invalidation, wire dirty close guard | Blocker: Fail 4. State, URL & Cache Sync |
| Missing `PermissionGate` or permanent lockout on transient permission API error | Add `PermissionGate` + retry recovery when `status === 'error'` | Blocker: Fail 5. Auth & Permission Resilience |
| Sequential `for...await` waterfall, un-memoized `>100KB` decode, or file `>300` lines | Use `Promise.all`, wrap in `useMemo`, extract subcomponents | Warning: Warn 6. Performance & Architecture |
| `@ts-ignore` / `eslint-disable` / deleted tests for CI green | Fix root-cause type, lint, or test error | 🛑 Block: Fail 7. Verification |

## Finding line examples (copy shape)

- Blocker: `src/settings/Drawer.tsx:12` (or `Drawer:[inline]`) — imports `react-icons` while project uses `lucide-react` — replace with Lucide icon
- Blocker: `src/routes/_app/users.tsx:258` — `handleDrawerOpenChange` closes drawer without checking `formDirty` — trigger `DiscardChangesDialog` before closing
- Blocker: `src/routes/_app/messages.index.tsx:45` — `validateSearch` accepts negative `page` and arbitrary `pageSize` — clamp `page >= 1` and restrict `pageSize` to whitelist
- Blocker: `src/lib/permission.ts:62` — permission query caches transient error for 5 minutes without retry on navigation — re-fetch when `query.state.status === 'error'`
- Blocker: `src/theme/colors.ts:19` — hardcoded `#FF5500` — map to Radix accent scale step
- Warning: `src/lib/failure-pulse.ts:88` — sequential `for...await` fetches pages 2..N — use `Promise.all` for concurrent fetching
- Warning: `src/components/cpi/json-view.tsx:74` — runs `JSON.parse` and regex tokenization on `>100KB` payload every render — cache with `useMemo` and cap tokenization size
- Warning: `src/routes/_app/monitor.tsx:1` — route file is 580 lines (exceeds 300-line limit) — extract section tables into `src/components/monitor/`
- Warning: `src/card/Item.tsx:14` — `1px` border + decorative left accent stripe — use `0.5px` border and remove stripe
- Nit: `src/pane/Split.tsx:40` — magic `w-[320px]` — map to spacing token

## Resources

| Path | Role |
| --- | --- |
| This `SKILL.md` | Complete review recipe |
| `test-prompts.json` (same folder) | Darwin evaluation & regression prompts |
| [Radix UI Colors](https://www.radix-ui.com/colors) | Color scales + semantic usage |

No external scripts required. Do not invent non-existent reference paths.

## Anti-patterns (do not)

- Modify repository files during a review without explicit user request (review is strictly read-only)
- Invent file paths, line numbers, or findings absent from the diff/repo (use `<scope>:[inline]` for line-less summaries)
- Approve while any Blocker or 🛑 STOP condition remains unresolved
- Score a dimension `Fail` when it only has `Warning`s, or score it `Warn` when it has a `Blocker`
- Omit same-dimension `Warning`s (such as `1px` borders) from `Findings` and `Notes` just because that dimension already has a `Blocker`
- Propose a second UI, icon, animation, router, or state library when one already exists in `package.json`
- Treat CI green as sufficient when types/lint are silenced or tests/features were deleted
- Ignore project-specific specs (`docs/开发规范.md`, `AGENTS.md`) in favor of generic guesses
- Miss state/URL/cache desynchronization, missing mutation invalidations, unguarded dirty drawers, `for...await` waterfalls, or un-memoized `>100KB` parsing
- Skip `Orient` or skip keyboard / a11y / long-text checks on interactive UI

## Output

```markdown
## Verdict
Approve | Request changes | Block

## Orient
One-line summary: Context (repo | inline-snippet), detected stack, project spec constraints, and audited scope.

## Summary
1–3 sentences on overall quality and merge/release fitness.

## Findings
### Blockers
- `path:line` (or `<scope>:[inline]`) — issue — required root-cause fix
### Warnings
- `path:line` (or `<scope>:[inline]`) — issue — suggested fix
### Nits
- `path:line` (or `<scope>:[inline]`) — polish only

## Checklist scores
| Dimension | Result | Notes |
| --- | --- | --- |
| 1. Dependencies & Stack | Pass/Warn/Fail | Summarize all Blockers & Warnings in this dimension |
| 2. Design & States | Pass/Warn/Fail | |
| 3. UI Style (copy · color · layout) | Pass/Warn/Fail | |
| 4. State, URL & Cache Sync | Pass/Warn/Fail | |
| 5. Auth & Permission Resilience | Pass/Warn/Fail | |
| 6. Performance & Architecture | Pass/Warn/Fail | |
| 7. Verification | Pass/Warn/Fail | |
```

Language: match requester (中文/English); keep Verdict labels and `path:line` references exact.

## Self-check before send
1. Every `Fail` dimension has `>=1 Blocker` in Findings; every dimension with only `Warning`s is scored `Warn` (never `Fail`).
2. All matched `Blocker`s and `Warning`s are reflected in both `Findings` and the corresponding row's `Notes` in `Checklist scores`.
3. Never output `Approve` if any 🛑 STOP condition was triggered.
