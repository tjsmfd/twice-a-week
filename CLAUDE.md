# S ≠ L (Surviving Isn't Living)

Put this file in the root of the `twice-a-week` repo. Claude Code reads it automatically at the start of every session.

## What this is
A personal PWA for TJ that tracks activities he's been putting off (movies, hikes, new restaurants, game nights) and pushes him to do them regularly, currently 3 per week (the code default is `settings.goal: 3`). Core rule: **never skip twice**. Each activity gets a plan and a date.

- **Repo:** https://github.com/tjsmfd/twice-a-week (public, branch `main`)
- **Live app:** https://tjsmfd.github.io/twice-a-week/ (GitHub Pages, deploys from `main` root)
- **Formerly named:** "Twice a Week". Only the visible title changed to "S ≠ L". Internal code names (`twa_*` keys, repo name) were left as-is on purpose. Don't rename them.
- The weekly goal is 3, set by `settings.goal` in code. TJ may change or drop it later, so read the setting instead of hard-coding a number, and don't build features around "twice".

## Repo layout
| Path | What it is |
|---|---|
| `index.html` | The whole live app: one self-contained HTML file (HTML + CSS + JS inline, ~380 KB). |
| `preview/index.html` | Preview build for testing changes before they go live. Uses its own storage key (`twa_data_preview`, seeded from live data). |
| `previous/index.html` | Rollback copy of the prior live version. Its auto-update check is disabled so it doesn't jump to the new build. |
| `README.md` | Big "Open the latest version" link + a **Last updated** line. |

## How the code works (quick map)
- `STANDALONE = !(window.claude && window.claude.use)`. The same code also ran as a claude.ai artifact (using ArtifactData collections `acts`, `log`, `hist`, `config/settings`). On GitHub Pages it's standalone and saves to `localStorage` key `twa_data_v1`.
- **Auto-update:** a small script at the top of `index.html` holds a build number (`twa-build` meta). On load/foreground it re-fetches the page; if the build number differs, it reloads. **Bump the build number on every release** or phones won't pick up the change.
- **Google Calendar, two ways:**
  1. OAuth sign-in with TJ's Client ID (`GCID_DEFAULT`, baked in). Tokens expire hourly.
  2. **Always-on link (preferred):** a Google Apps Script TJ deploys in his own account. The app generates the script text (`gasScript()`); it exposes `doPost` actions `ping/list/create/update/delete/save/load` guarded by a per-user key. Stored as `twa_gas` (web app URL) and `twa_gkey`.
- **Cloud backup:** through that same Apps Script, saved as `S ≠ L backup.json` in TJ's Google Drive a few seconds after changes. A "link code" (`<script URL>#<key>`) restores everything on a new phone.
- Also: manual backup export/import, automatic local snapshots, History tab with undo/restore.
- Calendar events are color "Grape". Default reminder: 1 hour before. Failed calendar writes go into a retry queue; the app re-checks Calendar every 3 min while open.
- **If you change the Apps Script's actions**, the app must tell TJ to update his script (there's an existing "Copy updated script" flow). He then redeploys: Deploy → Manage deployments → pencil → Version: New version → Deploy.

## Features built so far (high level)
Weekly goal visual ("Horizon" sunrise, matches the opening animation) · category tracking with white line icons · "Don't skip twice" alerts + daily reminder check · Sunday planning flow · two-way Google Calendar sync · day-schedule preview (busy/open blocks) when picking dates · bedtime-aware scheduling · History tab with undo · backups (manual, snapshots, Drive) · sound effects · swipe/motion polish · Night Sky update (Sky tab panorama, constellations, star check-off, weekly quests/events/campaigns/chains, ideas star chart, starry background) · iOS 26 PWA bottom-gap fix · auto-update.

## Design decisions TJ has made (don't undo without asking)
- Dark card style, shared with his "Mission Budget" app.
- Palette: refined violet ("Ultraviolet"). He **rejected** all black-and-white and colored category tints.
- Categories: icon-only, white line icons.
- Progress: "Horizon". He rejected the circular "0/2" ring.
- Bottom nav: icon-only with underline tabs. The "+" button floats separately, **not** inside the nav bar.
- Long alert/info text: collapse into tap-to-expand sections, not dense paragraphs.
- Sounds: Warm chord = completion, "Horizon theme" = weekly goal hit, Slide / Step-back = move / undo.
- Week starts **Sunday** (all week-boundary logic).
- Text has been shrunk on request; keep things compact.

## How TJ wants to work
- **Concept first.** For visual changes, build a separate preview/concept page (or use `preview/`) and show it. **Don't apply to the live app until he says "proceed."**
- When he approves: push to `main` yourself, keep the prior live build in `previous/` when the change is big, and **update the README "Last updated" line** (date, time ET, one-sentence summary) every time.
- Test before shipping (Playwright screenshots worked well: load the page, add sample data, check for console errors).
- He works mostly from his **iPhone** (home-screen PWA). Also uses a Chromebook and a home desktop. Assume he's not deep on dev tooling: explain in plain words, keep steps short.
- Replies: direct, short, plain. No hype. Core answer first.

## Context that affects scheduling features
- Bedtime is ~12:30 AM. Warn when a plan would run past it.
- Works shifts at Dick's Sporting Goods. Shifts go on Google Calendar as green "DSG shift" events with a 2-hour reminder, so the app sees them as busy time.
- Weeks are planned Sunday–Saturday.

## Ship checklist
1. Edit `index.html` (or `preview/` first if he wants to see it).
2. Bump the `twa-build` number.
3. Test (no console errors, data survives reload).
4. If big change: copy current live `index.html` to `previous/index.html` first (and disable its update check).
5. Update README "Last updated: <Mon D, YYYY · H:MM AM/PM ET>. <what changed>".
6. Commit + push to `main`. GitHub Pages redeploys in about a minute.
7. **Confirm the deploy succeeded.** After every push, check that the GitHub Pages deploy run shows success (`gh run list --limit 3`, or the Actions tab). Don't call a release done until it does. If it failed, say so and fix it.

## Syncing with GitHub (safe version)
Never use `git reset --hard` to sync. It can silently destroy unpushed commits and uncommitted work.
1. `git status`: see if anything is uncommitted or unpushed.
2. `git fetch origin`: download remote changes without touching local files.
3. Compare: `git log --oneline HEAD..origin/main` (remote-only commits) and `git log --oneline origin/main..HEAD` (local unpushed commits), plus `git diff HEAD origin/main` if needed.
4. If there are no unpushed local commits, `git pull --ff-only` is safe. If there are, **stop and tell TJ** what differs. Merge or rebase only with his OK. Never reset over unpushed commits.

## Public repo: keep this file clean
The repo is public. Never put TJ's email address, calendar contents or details, tokens, keys, or link codes in this file or anywhere in the repo.

## Optional add-on: Morning paper / Fuller Dispatch
**This is NOT part of the core app.** TJ may remove it. Leave it alone unless he asks you to change, move, or remove it.

What it is: the app sends this week's goal, what's done, and what's planned to TJ's own Google Drive so his printed morning paper can show it. Nothing is sent until he saves a paper link in Settings ("Morning paper" block).
- **Where it lives:** the `// ---------- morning paper (The Fuller Dispatch) ----------` section of `index.html`, plus the `s-paper` block in Settings.
- **Storage keys:** `fd_paper`, `fd_paper_at`.
- **Ship-checklist step (only while the add-on exists):** before committing, check it is still in `index.html` (`grep -c "Fuller Dispatch" index.html` is non-zero, and `fd_paper` and `s-paper` still appear). If it has gone missing without TJ asking, restore it before shipping and tell him.
