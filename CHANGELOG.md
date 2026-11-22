# MSP Toolbox — Change Log

Both **Claude** (Cowork) and **Codex** must append an entry here **every time they change any
file** in this repo. Put the newest entry at the top. Format:

```
## YYYY-MM-DD HH:MM — <Claude|Codex> — short one-line summary
- detail bullet 1
- detail bullet 2  (up to 3 bullets)
```

---

## 2026-11-23 — Claude — Add specific Windows Server versions to the VM Workload Group OS list

- `OS_OPTS` only had a single generic "Windows Server" entry — no way to record which version a VM is actually running. Added Windows Server 2025 / 2022 / 2019 / 2016 (2016 flagged "EOL Jan 2027", matching the date already cited in the Licensing Rules card) as distinct selectable options
- Kept the old `windows-server` / `windows-server-sql` values as "(other / unspecified)" entries rather than removing them — existing saved VM groups (old exports, the pre-populated sample data) still resolve and display correctly with no import/migration needed; no schema version bump required since `os` was already a free-form string field, just with more valid values now
- Verified via real dropdown selection in-browser: version-specific values still count correctly as Windows Server VMs in `licPullFromEnvironment()` (per-host attribution unaffected), render with the correct icon/label in the VM Table, and round-trip through TXT/XLSX export without errors; brace balance 1618/1618

---

## 2026-11-23 — Claude — Fix: new/renamed/removed host groups never refreshed the VM "Runs On" selector

- Confirmed via reproduction: clicking "+ Standalone group" or "+ Cluster" added the group to `hostGroups[]` correctly, but `addHostGroup()`/`updateHostGroup()`/`removeHostGroup()` only called `renderHostGroups()` + `renderTopology()` — never `renderGroups()` — so every VM Workload Group's "Runs On" `<select>` kept showing only "Primary Host Group" until something else happened to trigger a re-render; made it look like additional host groups "fell under" the primary group when really they just weren't offered as assignable targets yet
- Added `renderGroups()` calls to `addHostGroup`, `updateHostGroup`, `removeHostGroup`, and `primaryTopologyChanged` (the last one keeps the "Runs On" label in sync when the primary's cluster name changes); a VM group assigned to a host group that's later removed gracefully falls back to displaying/computing against the primary (confirmed: orphaned `hostGroupId` doesn't crash anything, `licPullFromEnvironment()` already had the fallback from the previous fix)
- Verified entirely via real clicks/selects in the browser (add host group → option appears instantly in "Runs On"; rename → label updates instantly; assign via dropdown → data updates; remove the assigned group → dropdown falls back to Primary and licensing re-pull reroutes correctly); brace balance 1614/1614, no export errors

---

## 2026-11-23 — Claude — Fix: host groups couldn't have a different hypervisor; Windows licensing billed the wrong host group

- Confirmed via reproduction: a VMware primary host (Linux VMs only) + an additional Hyper-V cluster (the actual Windows VMs) — "Pull from Environment" billed all 5 Windows Server VMs to the VMware host and 0 to the Hyper-V cluster that needed them, because `licPullFromEnvironment()` dumped the total Windows VM count onto whichever host group happened to be first in the list, every other group got 0, regardless of where the VMs actually ran
- Root cause of the "can't choose a different OS" complaint: additional host groups (`renderHostGroups()`) never rendered a Hypervisor `<select>` at all, even though the data model (`hypervisor` field, defaulted to `'vmware'`) already supported it — added the missing selector (`HYPERVISOR_OPTS`, mirrors the primary group's options)
- Added a "Runs On" field to each VM Workload Group (`hostGroupId`, defaults to the primary host group) so each group of VMs can be assigned to the host group it actually runs on; `licPullFromEnvironment()` now sums Windows Server VMs **per assigned host group** instead of dumping everything on index 0, with safe fallback to primary for unassigned/invalid/excluded assignments; Licensing rows now show a hypervisor badge for visual confirmation, and a new reference card note explains licensing is per-host-core, not per-hypervisor
- Schema v9→v10 (`vmGroups[].hostGroupId`, `hostGroups[].hypervisor` now user-editable); fully backward compatible — v9 and older imports default every VM group to the primary host group (same effective behavior as before this fix), verified in-browser; brace balance 1614/1614, reproduced the bug and confirmed the fix via the real UI (clicks/selects, not just JS calls) both before and after

---

## 2026-11-23 — Claude — Fix: "Existing Version" is now a dropdown, not free text

- Replaced the free-text "Existing Version" input with a `<select>` (`LIC_EXIST_VERSIONS`: 2012 / 2012 R2 / 2016 / 2019 / 2022) so it can't be mistyped; existing rows created before this field existed (or carrying an invalid value) self-heal to the latest pre-2025 option on render, so the visible selection always matches what's actually stored on the row
- Caught by clicking through the real UI (toggle checkbox → dropdown → cores-owned input → Top Up As override) rather than only calling the calc functions directly — the dropdown had been silently showing "2012" (just the first `<option>`) while the row's actual `existVersion` was `undefined`; confirmed fixed and the full flow (gap panel + Cost Summary line items) updates live through real clicks/typing, not just JS calls

---

## 2026-11-23 — Claude — Windows Server Licensing: existing-licensing gap analysis + downgrade-rights top-up

- Each Windows host group row can now record existing licensing already owned (edition, version label, core licenses owned/host) via an "Already have licensing" toggle; gap is computed against the OWNED edition's requirement (not the auto-cheapest tier), independent of `licWinCompute`'s from-scratch sizing — flags Under-licensed / Over-licensed / Exactly licensed with numeric core gap, plus a mismatch note when the owned edition isn't the cheaper fit for the host's VM count
- When short, shows two top-up purchase paths — buy more of the **same version** vs buy **Windows Server 2025 + downgrade rights** (run the existing OS now, no repurchase at upgrade time) — cheapest flagged, user can force either via a "Top Up As" selector; reuses the existing `licBestMix` 16-core/2-core-pack optimizer and new editable "Existing/Legacy" price fields (default ≈ WS2022 list price)
- Cost Summary / TXT-PDF / XLSX now bill only the shortfall top-up cost for rows with existing licensing entered (not the full from-scratch cost); schema v8→v9 (exist* fields on Windows rows + 4 legacy price keys), fully backward compatible — v8 and older imports default cleanly (verified in-browser); brace balance 1588/1588, node --check passes, Node logic tests cover short/exact/surplus/mismatch/multi-host/forced-top-up, browser + XLSX + TXT export verified in both themes and mobile

---

## 2026-07-15 — Claude — Licensing tab: add Windows Server + RDS CAL calculator

- New "Client Access Licenses (CALs)" card on the Licensing tab: Windows Server CALs (every user/device accessing a server) and RDS CALs (Remote Desktop users/devices, on top of a Windows CAL) — user-vs-device comparison with auto-cheapest selection, aimed at shared-terminal sites (POS / gaming floor / kiosks, e.g. RSLs) where Device CALs usually win
- Four new editable prices (WS User/Device CAL, RDS User/Device CAL), CAL lines + subtotal in the Cost Summary shopping list, new CALs rules ref-box (mixing allowed, version-specific, not needed for anonymous web)
- Export schema v7→v8 (`lic.cals` block; v7/v6/v5 imports keep defaults — verified in-browser); CAL section added to TXT/PDF export and the XLSX Licensing sheet; brace balance 1522/1522, node --check passes, 7 Node logic tests + browser round-trip/XLSX tests pass

---

## 2026-07-15 — Claude — Add Licensing tab: Windows Server 2025 + SQL Server 2022 calculators

- New tab between Migration and Network: Windows Server 2025 per-core calculator (min 8 cores/proc, 16/host; Standard stacks full core-sets at 2 VMs/set vs Datacenter unlimited; per-host pack maths since licenses can't span hosts; VM-mobility toggle; auto/forced edition per row with break-even note) and SQL Server 2022 per-VM calculator (min 4 cores/VM, 2-core packs only, Standard 24-core cap warning, Express-eligibility hint); every requirement shown as cheapest 16-core + 2-core mix AND 2-core-packs-only equivalent
- Editable pricing card (defaults ≈ Microsoft USD list, reset button, currency label), live Cost Summary shopping list, licensing rules reference card with 2016/SQL EOL dates, "Pull from Environment" buttons import host groups + SQL-flagged VM groups; Environment ref cards now link to the tab
- Export/import schema v6→v7 (adds `lic` block; v6/v5 imports keep defaults — verified in-browser); LICENSING section added to TXT/PDF export and a Licensing sheet to XLSX; brace balance 1488/1488, node --check passes, 13 Node logic tests + browser round-trip/export tests pass

- New full-width card below the hardware/capacity content: optional proposed server(s) (hosts, sockets, cores, RAM/host, local storage/host, hyperthreading, cluster + N+1 HA reserve — same reserve logic as existing host groups) and optional proposed SAN/NAS usable TB; either can be entered alone or together
- Validates the proposed build's logical cores / RAM / storage against the *current workload demand* (VM Workload Groups totals, not existing hardware); reuses the exact Sizing Guidelines thresholds (CPU 6:1/10:1, RAM 1:1/1.1:1, storage +30%/+15% buffers) via `computeProposedValidation()`, a single calc function shared by the live panel, TXT/PDF export, and XLSX export; live-updates on any input change (both proposed-build fields and VM group edits)
- Export/import schema bumped v5→v6 (adds `pbv` block); v5 imports still load correctly with proposed-build defaults; brace balance 1320/1320, Node logic check confirmed optimal/fit/insufficient bands and verdict text for sample builds

---

## 2026-06-25 — Claude — Fix: light theme tokens now override Codex's later :root block (specificity bump)

- Root cause: `[data-theme="light"]` and `:root` both have specificity (0,1,0); Codex's dark graphite `:root` block appears later in the file so it wins over the light token block in all cases — light mode tokens were completely ignored
- Fix: changed the token-defining block selector from `[data-theme="light"]` to `:root[data-theme="light"]` (specificity 0,2,0) so it beats both `:root` blocks regardless of source order; descendant rules (`[data-theme="light"] body`, etc.) already exceed `:root` specificity and were not changed; no `[data-theme="dark"]` block exists so dark mode is unaffected
- In light mode `--bg-card` now resolves to `#FFFFFF`, `--text-primary` to `#0F172A`; brace balance 1193/1193

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
