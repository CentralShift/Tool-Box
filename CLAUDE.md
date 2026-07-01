# Central Shift Tool-Box — Project Instructions

## What This Is
A public-facing collection of MSP tools for Central Shift clients and candidates.
Each tool is self-contained — candidates and clients visit this site to use tools
like the migration calculator, without needing to log in to anything.

- **Repo:** github.com/CentralShift/Tool-Box
- **Currently live:** centralshift.github.io/Tool-Box/
- **Planned:** tools.centralshift.com.au (Netlify + custom subdomain, not yet configured)

## Stack
- Plain HTML, CSS, JavaScript — no framework, no build step
- Hosted on GitHub Pages (auto-deploys from `main`)
- Future: migrate to Netlify with custom subdomain `tools.centralshift.com.au`

## Tools
- Migration tool (live)
- More to be added — each tool gets its own page/folder

## Deployment Workflow
- **Never push to `main` or GitHub** — that is the user's decision
- Work on a local feature branch: `git checkout -b feature/<short-description>`
- User reviews locally, then pushes `main` → GitHub Pages auto-deploys
- When migrated to Netlify: push `main` → Netlify deploys to tools.centralshift.com.au

## Rules
- No frameworks, no build tooling unless explicitly agreed
- No secrets or API keys in code — tools must be fully client-side or use public APIs
- Each tool should be self-contained in its own folder or clearly separated section
- Tools must work on mobile as well as desktop — these are public-facing
- Read `CHANGELOG.md` at the start of every session

## Project Ecosystem
All three Central Shift repos are on this machine. Agents may need to cross-reference them.

| Project | Local path | GitHub | Live |
|---|---|---|---|
| **Tool-Box (this repo)** | `Claude\Projects\Centralshift ToolBox` | `CentralShift/Tool-Box` | GitHub Pages → tools.centralshift.com.au (planned) |
| **Website** | `GitHub\Website` | `CentralShift/Website` | Netlify — centralshift.com.au |
| **CRM** | `Claude\Projects\CRM  ATS` | `CentralShift/CRM` | Vercel — internal only |

The website will have an MSP Toolbox page that links users here.
