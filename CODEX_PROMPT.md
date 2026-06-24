# Codex Onboarding Prompt

You are working on **MSP Toolbox**, a single-file web app (`index.html`) — a set of tools for MSP/IT work: a VM/server sizing calculator ("Environment"), RAID calculators, a workload migration estimator, and network tools (IP/subnet calculator, VLAN planner, DNS checker). It's deployed via GitHub Pages at https://centralshift.github.io/Tool-Box.

**Your working folder:** `C:\Users\Admin\Documents\Claude\Projects\Centralshift ToolBox`

**This is a shared repo.** You (Codex) and a second agent (Claude, in Cowork) both edit this codebase. Read `AGENTS.md` in the repo root first — it defines the workflow. Key rules:

1. **You edit files locally. You do NOT run git, commit, or push.** You have no GitHub access. Claude owns all commits, pushes, and deployment.
2. **Before editing,** read `CHANGELOG.md` to see what changed recently and why.
3. **After every change,** prepend a new entry to the top of `CHANGELOG.md`: `## YYYY-MM-DD HH:MM — Codex — <short summary>` plus 1–3 detail bullets. This is how Claude knows what you changed and picks it up.
4. **Keep it a single-file app** — all HTML/CSS/JS stay in `index.html` unless agreed otherwise. No build steps or frameworks.
5. **Don't reformat or refactor unrelated code.** Keep changes focused so diffs stay reviewable.
6. **Never put secrets** (tokens, keys) in any tracked file. The GitHub token lives only in local git config, which Claude manages.

When you finish, leave the change saved in the folder and logged in CHANGELOG.md — Claude will review, commit, and push it (Pages redeploys ~60s after).
