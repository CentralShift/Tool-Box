# MSP Toolbox — Agent Collaboration Guide

## Project

**Single-file static web app** — `index.html` only (no build step, no framework, no backend).
Deployed automatically via **GitHub Pages** at `https://centralshift.github.io/Tool-Box`.
Push to `main` → live in ~60 seconds.

## Repository

- Canonical working folder: `C:\Users\Admin\Documents\Claude\Projects\Centralshift ToolBox`
- Remote: `https://github.com/CentralShift/Tool-Box` (public)
- Branch: `main`

## Agent Roles

| Agent | Responsibilities |
|---|---|
| **Codex** (OpenAI CLI) | Edits `index.html` (and other files) locally. **Must NOT run git, commit, or push** — it has no GitHub access. |
| **Claude** (Cowork) | Reviews Codex changes, commits, and pushes to GitHub. Also makes its own direct edits. |

Both agents **must** update `CHANGELOG.md` whenever they change any file.

## Workflow

1. **Before editing** — read `CHANGELOG.md` to see what the other agent last changed and avoid conflicts.
2. **Make your edits** to `index.html` (or other files as agreed).
3. **After editing** — prepend a new entry to the top of `CHANGELOG.md` (below the header block, above the previous entry):
   ```
   ## YYYY-MM-DD HH:MM — <Claude|Codex> — short summary
   - detail bullet
   ```
4. **Claude only** — stage, commit with explicit `-m`, and push:
   ```
   $env:GIT_TERMINAL_PROMPT=0
   git add <files>
   git commit -m "description"
   git push origin main
   ```

## Architecture Notes

- All CSS is inside the `<style>` block in `index.html`; all JS is inside the `<script>` block at the bottom.
- CSS custom properties (design tokens) live on `:root`; light-theme overrides on `[data-theme="light"]`.
- Tab switching: `switchTab(name)` — panels are `#tab-{name}`, buttons have `data-tab="{name}"` attribute.
- Export: `getExportText()` → `.txt` download, clipboard copy, and PDF via jsPDF (CDN). Import reads the `===CALC-DATA===` JSON block (current version: **v5**) and calls `raidApplyImport(d)`.
- Environment topology state: primary `#hw-*` host group plus module-level `hostGroups[]` and `sans[]`; `getInfrastructureModel()` applies cluster N+1 reserves and aggregates effective compute, RAM, local storage, and shared storage.
- VLAN state: module-level `vlanRows[]` array + `vlanNid` counter.
- VM group state: module-level `groups[]` array + `gid` counter.
- Migration constants: `MIG_LINE_MBPS`, `MIG_METHOD_LABELS`, `MIG_NET_LABELS`.

## Tabs and Key IDs

| Tab | Panel ID | Key inputs | Key output area |
|---|---|---|---|
| Environment | `#tab-environment` | `#hw-*`, VM group buttons | `#results`, `#stats-grid` |
| Storage / RAID | `#tab-storage` | `#r-*` (forward), `#rv-*` (reverse) | `#r-results`, `#rv-results` |
| Migration | `#tab-migration` | `#mig-*` | `#mig-result` |
| Network | `#tab-network` | `#sub-input`, `#dns-domain`, `#dns-type`, `#vlan-tbody` | `#sub-results`, `#dns-results` |

## Rules

- Keep it a **single file** (`index.html`) unless both agents explicitly agree to split.
- **Do not reformat unrelated code** — only touch lines relevant to your change.
- **Do not introduce build steps**, npm packages, or external framework dependencies.
- **Do not commit secrets** — the PAT lives only in `.git/config` (never in tracked files).
- **Do not push from the old folder** (`C:\Users\Admin\Claude\Projects\Calc`) — it is retired.
