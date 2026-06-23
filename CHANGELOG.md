# MSP Toolbox — Change Log

Both **Claude** (Cowork) and **Codex** must append an entry here **every time they change any
file** in this repo. Put the newest entry at the top. Format:

```
## YYYY-MM-DD HH:MM — <Claude|Codex> — short one-line summary
- detail bullet 1
- detail bullet 2  (up to 3 bullets)
```

---

## 2026-06-23 18:00 — Claude — Initial MSP Toolbox build (Phases 1–3)

- Single-file app (`index.html`) deployed to GitHub Pages at centralshift.github.io/Tool-Box
- Four tabs: **Environment** (server/VM hardware + VM-group sizing + recommendations), **Storage/RAID** (forward calc + reverse/target-capacity planner), **Migration** (workload transfer time estimator), **Network** (IP/Subnet Calculator, VLAN Planner, live DNS Checker via Cloudflare DoH)
- Sticky header with tab bar, light/dark theme toggle, and Import / Copy / .txt / PDF export buttons; export data block at v4 (includes VLAN rows); jsPDF CDN for client-side PDF generation
