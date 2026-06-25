# MSP Toolbox — Change Log

Both **Claude** (Cowork) and **Codex** must append an entry here **every time they change any
file** in this repo. Put the newest entry at the top. Format:

```
## YYYY-MM-DD HH:MM — <Claude|Codex> — short one-line summary
- detail bullet 1
- detail bullet 2  (up to 3 bullets)
```

---

## 2026-06-25 — Claude — Fix: light theme now re-themes cards/panels/text, not just the page background

- Root cause: Codex's operations-console CSS hardcoded 13 dark hex values directly on element rules (`.card-header`, `.vm-group`, `.topology-strip`, `.host-group`, `.calc-row`, etc.) that bypass the CSS-variable token system — these never flip on theme toggle; `[data-theme="light"]` token overrides are correct but were invisible to these elements
- Fix part 1 — token block: updated `--text-code:#1D4ED8` → `#2C7A72` (dark teal for mono on white); added light-mode overrides for `--accent-blue-bright:#1A8A83`, `--accent-teal:#1B8080`, `--border-active:#1B8080`, `--focus-ring` so teal UI chrome is readable on light backgrounds
- Fix part 2 — element rules: replaced all 13 hardcoded dark colors (`#0D1317`, `#0C1216`, `#0E151A`, `#10171C`, `#0D1518`, `#0A1014`, `#226D68/#4FB6AC`, `#83D8CF`, `#8FD4CC`, `#A5E8E0`, `#BFDBFE`, `#657477`) with CSS variables (`var(--bg-elevated)`, `var(--bg-surface)`, `var(--bg-base)`, `var(--accent-blue)`, `var(--accent-teal)`, `var(--text-code)`, `var(--accent-blue-bright)`, `var(--text-muted)`) across the ops-console and dashboard-refresh blocks; dark mode unchanged; brace balance 1193/1193

---

## 2026-06-25 08:19 — Codex — Add operations-console UI and multi-host infrastructure modelling

- Restyled MSP Toolbox toward Option B with flatter graphite surfaces, restrained teal status colour, technical spacing, text-only navigation, and responsive overflow fixes
- Added primary/additional standalone and clustered host groups, roles, per-group hardware, N+1 HA reserves, topology rendering, and multiple SAN/NAS/HCI storage arrays
- Effective environment capacity now aggregates compute hosts, excludes backup-only CPU/RAM, counts shared storage once, and round-trips via v5 TXT/XLSX data; syntax, IDs, brace balance, and cluster/backup/SAN model tests pass

---

## 2026-06-25 07:35 — Codex — Save visual redesign concepts for persistent review

- Added three UI concept PNGs under `visual-concepts/` so the light workbench, dark operations console, and editorial planner options remain accessible across devices
- No application HTML, CSS, or JavaScript changed

---

## 2026-06-24 — Claude — Fix: reference cards no longer clip at bottom in two-column layout

- Root cause: `.right-col` is a flex column with `max-height`+`overflow-y:auto`; default `flex-shrink:1` on `.card` children lets the flex algorithm shrink tall cards below their content height, and `overflow:hidden` on `.card` clips the bottom — confirmed on RAID ("RAID 10 — High-I/O databases") and Migration ("Cold / Offline Copy") tabs
- Fix: added `.right-col > .card{flex-shrink:0;}` inside `@media(min-width:1101px)` so cards never shrink; the right-col scrollbar handles overflow as intended
- Affects all four tabs (Environment, RAID, Migration + reverse planner); brace balance 1011/1011

---

## 2026-06-24 — Claude — Fix: right-column reference cards no longer clip/cut off content

- Primary fix: `.log-pills` changed from `flex-wrap:nowrap` to `flex-wrap:wrap` — pills in Calculation Log entries now wrap instead of overflowing and being clipped by the card boundary
- Secondary fix: added `overflow-wrap:break-word;word-break:break-word` to `.card-body`, `.ref-box`, `.log-detail-val`, `.log-detail-sub` so long text and monospace values wrap rather than overflow
- Right-col sticky scroll container: added `overflow-x:hidden` (prevents horizontal bleed) and `padding-bottom:16px` (breathing room so last card content isn't flush against the scroll clip boundary); brace balance 1010/1010

---

## 2026-06-24 — Claude — Cross-tab sync: Environment totals feed Migration + Storage/RAID on Calculate

- Clicking "Calculate" on the Environment tab now propagates total VM count → `mig-vms`, total storage → `mig-size`/`mig-unit`, and total storage → `rv-target`/`rv-unit`; downstream calcs recompute automatically
- Synced fields show a small "↩ from Environment" badge and a blue input border; if the user manually edits a synced field the badge flips to "✎ manual override" — next Calculate re-syncs all fields
- Import is isolated: `raidApplyImport` calls `calculate(true, false)` so import-restored values are never overwritten by a stale sync; brace balance 664/664

---

## 2026-06-24 — Claude — Fix: import now restores VM workload groups

- Root cause: `calculate()` called during import fired `scrollIntoView` on `#results`, scrolling VM groups out of the viewport so they appeared to vanish even though they were correctly restored in the DOM
- Fix: added `skipScroll` param to `calculate()`; `raidApplyImport` passes `true` so results refresh without the scroll side-effect — groups remain visible after import
- No data-logic changes; brace balance confirmed 653/653; manual-Calculate button still scrolls as before

---

## 2026-06-24 — Claude — Added Excel (.xlsx) export/import

- `⬇ XLSX` button in header exports a workbook with four human-readable sheets (Environment, RAID, Migration, Network) plus a hidden `_data` sheet containing the v4 JSON blob for round-trip import
- Import button now accepts `.txt` and `.xlsx`; detects file type and routes to the correct handler; `.xlsx` import reads the `_data` sheet and calls `raidApplyImport()` to repopulate all calculators
- SheetJS 0.18.5 loaded via CDN; existing `.txt` and PDF export/import unchanged

---

## 2026-06-24 — Claude — Added CODEX_PROMPT.md

- Added `CODEX_PROMPT.md` onboarding prompt to repo root for Codex agent context and workflow rules

---

## 2026-06-24 10:03 — Codex — Refresh Environment as a live capacity dashboard

- Reworked the Environment layout into a compact server-profile and VM-workload workspace with clearer hierarchy, spacing, focus states, and responsive behaviour
- Added a sticky live Capacity Summary for CPU, memory, storage, fit status, and the optimal recommended host specification
- Preserved the existing calculation, export, import, licensing, RAID, migration, and network logic; JavaScript syntax and dashboard ID bindings validated

---

## 2026-06-24 — Claude — Pipeline test from new canonical folder

- Verified commit + push + GitHub Pages deploy works from `C:\Users\Admin\Documents\Claude\Projects\Centralshift ToolBox`

---

## 2026-06-23 18:00 — Claude — Initial MSP Toolbox build (Phases 1–3)

- Single-file app (`index.html`) deployed to GitHub Pages at centralshift.github.io/Tool-Box
- Four tabs: **Environment** (server/VM hardware + VM-group sizing + recommendations), **Storage/RAID** (forward calc + reverse/target-capacity planner), **Migration** (workload transfer time estimator), **Network** (IP/Subnet Calculator, VLAN Planner, live DNS Checker via Cloudflare DoH)
- Sticky header with tab bar, light/dark theme toggle, and Import / Copy / .txt / PDF export buttons; export data block at v4 (includes VLAN rows); jsPDF CDN for client-side PDF generation
