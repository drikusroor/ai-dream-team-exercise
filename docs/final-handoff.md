# Project Pulse final handoff

Project Pulse is a static, no-build dashboard that shows Mona's team projects as cards. Each card shows the owner, status, recent activity, priority, and a short summary. The files live in `app/` and a local Python HTTP server runs them through a VS Code launch configuration.

## Team contributions

| Agent | Model | Contribution | Files |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Locked the shared contract (JSON schema, value-to-class rule, CSS hooks) from `docs/project-pulse-plan.md` §2. Ran Designer and Coder in parallel with separate file scopes, then ran the integration review and final validation. Did not edit app files. | `docs/final-handoff.md` (through Coder) |
| Planner | Claude Opus 5.5 | Wrote the plan: file ownership, phases, dependencies, edge cases, and validation expectations. | `docs/project-pulse-plan.md` |
| Designer | Gemini 3.7 Flash | Built the stylesheet:<br>• Colour tokens as CSS custom properties, with WCAG AA contrast documented.<br>• Gradient header with a pulse dot, plus a stats strip.<br>• Responsive grid `repeat(auto-fill, minmax(280px, 1fr))` that drops to 1 column at ≤640px.<br>• Cards with rounded corners, shadows, a top border coloured by status, and a lift on hover or focus-within.<br>• Status pills with shapes (circle, diamond, square) and priority tags with signal bars, so colour is not the only cue.<br>• Neutral badges for unknown values.<br>• Styles for loading, error, and empty messages.<br>• `:focus-visible`, `prefers-reduced-motion`, and `forced-colors` support. | `app/styles.css` |
| Coder | GPT-5.5 | **Data:** 6 projects, `lastUpdated` 2026-10-26. Statuses: On track 2, At risk 2, Blocked 1, Completed 1. Priorities: High, Medium, and Low, 2 each.<br>**Markup:** semantic HTML with one `h1`. Each card is a `ul`/`li` > `article` with an `h2`. An `aria-live` status region announces changes.<br>**Script (inline):** fetches `project-data.json`, checks `response.ok`, and checks that `projects` is an array. Builds the page with `textContent` only (no `innerHTML`). Counts projects by status for the stats strip. Shows "Unknown"/"Not set" for missing values and skips entries that are not objects. Opening the page as `file://` shows a hint that points to the launch config.<br>**Launch config:** "Run Project Pulse Dashboard". | `app/project-data.json`, `app/index.html`, `.vscode/launch.json` |

## How to run

1. Open the repository in VS Code.
2. Open **Run and Debug** and select **"Run Project Pulse Dashboard"**.
3. The config runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. When the server starts, the browser opens `http://localhost:5500/index.html`.
4. When you are done, stop the server (Stop button or Ctrl+C in the terminal).

Opening `app/index.html` directly as a `file://` page shows an error message on purpose. The browser blocks `fetch` from `file://` pages, so the message tells you to use the launch configuration instead.

## Dashboard validation

### Checked by reading the files

| Check | Result |
| --- | --- |
| `app/project-data.json` and `.vscode/launch.json` are valid JSON | PASS |
| All 6 projects have `name`, `owner`, `status`, `recentActivity`, and `priority`, all with allowed values | PASS |
| `app/index.html` contains Project Pulse, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | PASS |
| `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `prefers-reduced-motion`, `minmax(280px` | PASS |
| `.vscode/launch.json` contains "Run Project Pulse Dashboard", `${workspaceFolder}/app`, `index.html` | PASS |
| The class names in the HTML match the contract hooks used in the CSS | PASS |

### Runtime checks (run on 2026-10-26)

| Command / check | Result |
| --- | --- |
| `python3 -m json.tool app/project-data.json >/dev/null && python3 -m json.tool .vscode/launch.json >/dev/null && echo JSON_OK` | PASS: printed `JSON_OK` |
| `git status --porcelain .vscode/tasks.json` | PASS: empty output, file unchanged |
| `cd app && python3 -m http.server 5599`, then `curl` each file | PASS: `index.html` 200, `styles.css` 200, `project-data.json` 200 |
| Served `index.html` contains "Project Pulse" | PASS: 5 matching lines |
| Server banner matches the `serverReadyAction` pattern `Serving HTTP on .* port ([0-9]+)` | PASS |
| Browser render at 1280px (Playwright) | PASS: 6 cards in 3 columns, one `h1` and six `h2`, stats 6 / 2 / 2 / 1 / 1, loading message hidden, styles applied, no horizontal scroll |
| Browser render at 375px (Playwright) | PASS: 1 column, no horizontal scroll |
| Browser console | PASS: no app errors. The only entry is a `favicon.ico` 404, which the browser requests by default |
| Temporary server stopped | PASS: nothing listening on port 5599 afterwards |
| `bash scripts/validate-exercise.sh` | 110 PASS, 2 FAIL (details below) |

The 2 failures in `scripts/validate-exercise.sh` are template checks. Neither one is a dashboard defect:

- **"Learner answer files are tracked in the template"**: this check expects a clean template. In this learner repo, `.vscode/launch.json`, `app/*`, `docs/agent-team.md`, and `docs/project-pulse-plan.md` are tracked because they are the exercise answers.
- **"README explains Project Pulse story"**: `README.md` does not contain the phrase "Project Pulse". `README.md` was outside the scope of this task and was not changed.

All Step 3 and Step 4 workflow checks passed, including launch config name/JSON, dashboard CSS selector, card markup/styling, and handoff launch reference.

### Manual checks for the learner

- [ ] Run "Run Project Pulse Dashboard". The browser should open the dashboard, not a directory listing.
- [ ] Six project cards render. The stats strip shows On track 2, At risk 2, Blocked 1, Completed 1.
- [ ] At about 375px wide, cards stack in 1 column and nothing scrolls horizontally.
- [ ] At about 1280px wide, cards form a multi-column grid.
- [ ] The DevTools console shows no errors, and the network panel shows `styles.css` and `project-data.json` loading with status 200.
- [ ] Pressing Tab shows a visible focus outline.

## Known risks and open items

- **Port 5500 in use:** if another process (for example Live Server) is using port 5500, the launch fails. Stop that process and launch again.
- **Output buffering:** Python can hold back its output, so the "Serving HTTP on … port 5500" line may never appear. If so, `serverReadyAction` does not open the browser. Either open `http://localhost:5500/index.html` yourself, or change the command to `python3 -u -m http.server 5500`.
- **Codespaces / remote:** `openExternally` goes through port forwarding. Allow port 5500 to forward if VS Code asks.
- **`.vscode/tasks.json`** was left unchanged on purpose.
- **README:** the validation script wants "Project Pulse" in `README.md`. This was out of scope here; update it if your template requires it.

## Final handoff

The learner owns the next steps:

1. Review `docs/final-handoff.md` and the app files: `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`.
2. Run the manual checks above using "Run Project Pulse Dashboard".
3. Check the Step 4 workflow run that the push to `main` triggered.

The learner directed all git operations. `docs/final-handoff.md` was staged, committed, and pushed only at the learner's explicit request.
