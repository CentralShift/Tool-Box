# MSP Toolbox — Change Log

Both **Claude** (Cowork) and **Codex** must append an entry here **every time they change any
file** in this repo. Put the newest entry at the top. Format:

```
## YYYY-MM-DD HH:MM — <Claude|Codex> — short one-line summary
- detail bullet 1
- detail bullet 2  (up to 3 bullets)
```

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
