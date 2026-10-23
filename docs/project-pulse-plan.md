# Project Pulse implementation plan

## 1. Summary

Project Pulse is a small static dashboard with no build step. It shows Mona's team projects as cards, and each card lists the owner, status, recent activity, priority and a short summary. It needs four files:

- **app/index.html**: page markup plus an inline `<script>` that fetches `project-data.json` and renders one `.project-card` per project.
- **app/styles.css**: a polished, responsive card layout with status and priority badges.
- **app/project-data.json**: the project data, under a top-level `projects` array.
- **.vscode/launch.json**: a **Run Project Pulse Dashboard** configuration that runs `python3 -m http.server 5500` from `app/` and opens `http://localhost:%s/index.html`.

The Designer owns the visual layer. The Coder owns the markup, data, rendering logic and launch config. Both work from the shared contract in section 2, so they can run in parallel without touching the same files. The existing `.vscode/tasks.json` must not be changed.

## 2. File assignments

| File | Owning agent | Purpose |
| --- | --- | --- |
| `app/styles.css` | Designer | Layout, typography, card look (`border-radius`, `box-shadow`), status and priority badges, responsive grid, focus styles, loading/error/empty message styles |
| `app/index.html` | Coder | `<title>Project Pulse</title>`, a `<h1>`, link to `styles.css`, `fetch("project-data.json")`, renders the cards, handles errors |
| `app/project-data.json` | Coder | Valid JSON: `{ "projects": [ ... ] }` with 5–6 sample projects |
| `.vscode/launch.json` | Coder | Strict JSON with no comments. The "Run Project Pulse Dashboard" configuration |

**Shared contract (lock before Phase 1; changes go through the Orchestrator):**

- **JSON schema.** Each project has these required fields: `name`, `owner`, `status`, `recentActivity`, `priority`. Optional fields are `summary` (a contributor-friendly sentence) and `id`. A top-level `lastUpdated` (ISO date) is optional.
  - `status` must be one of: `"On track"`, `"At risk"`, `"Blocked"`, `"Completed"`.
  - `priority` must be one of: `"High"`, `"Medium"`, `"Low"`.
- **Value-to-class rule.** Lowercase the value and replace spaces with hyphens. For example, `"At risk"` becomes `status-badge--at-risk` and `"High"` becomes `priority-badge--high`.
- **CSS class names / hooks:**
  - Page structure: `.dashboard` (the `<main>` wrapper), `.dashboard-header`, `.dashboard-title`, `.dashboard-subtitle`, `.dashboard-stats` / `.stat`, `.project-grid` (a `<ul>`).
  - Inside each card: `.project-card` (an `<li>` containing an `<article>`), `.project-name`, `.project-owner`, `.project-summary`, `.project-activity`, `.project-meta`.
  - Badges: `.status-badge` plus `--on-track|--at-risk|--blocked|--completed`, and `.priority-badge` plus `--high|--medium|--low`.
  - Messages: `.dashboard-message` plus `--loading|--error|--empty`.

## 3. Designer responsibilities

- Write **app/styles.css** only. Use CSS custom properties for colours, spacing and radius.
- `.dashboard`: a centred, max-width layout with readable spacing. The header should make "Project Pulse" the clear focal point.
- `.project-grid`: CSS Grid using `repeat(auto-fill, minmax(280px, 1fr))`. That gives 1 column on mobile and 2–3 on desktop.
- `.project-card`: `border-radius`, `box-shadow`, padding, and a subtle hover/focus-within lift. Respect `prefers-reduced-motion`.
- Visual priority inside a card: name, then status badge, then owner, then summary, then recent activity, then priority.
- Badges must not rely on colour alone, because the text label is always rendered. Contrast must be at least 4.5:1 (WCAG AA).
- Give the base `.status-badge` and `.priority-badge` neutral styles, so unknown values still look fine.
- Style `.dashboard-message` states and visible `:focus-visible` outlines.
- When finished, report design decisions and any extra hooks needed, but do not edit HTML.

## 4. Coder responsibilities

- **app/project-data.json**: 5–6 realistic projects that cover every status and priority value, and that strictly follow the contract.
- **app/index.html**:
  - Semantic structure: `lang="en"`, a viewport meta tag, `<header>`, `<main class="dashboard">`, `<h1 class="dashboard-title">Project Pulse</h1>`, and an `aria-live="polite"` message region.
  - Inline script steps: `fetch("project-data.json")`, check `response.ok`, validate that `projects` is an array, then build elements with `textContent` (no `innerHTML` with data). Read `project.status`, `project.recentActivity` and `project.priority` explicitly. Also render a stats strip with counts by status.
- **.vscode/launch.json**: write it as follows.
  - `"version": "0.2.0"`.
  - One configuration:
    - `"name": "Run Project Pulse Dashboard"`
    - `"type": "node-terminal"`
    - `"request": "launch"`
    - `"command": "python3 -m http.server 5500"`
    - `"cwd": "${workspaceFolder}/app"`
  - `serverReadyAction`:
    - `"pattern": "Serving HTTP on .* port ([0-9]+)"`
    - `"uriFormat": "http://localhost:%s/index.html"`
    - `"action": "openExternally"`
- Do not edit `.vscode/tasks.json` or `app/styles.css`.

## 5. Ordered implementation steps / phases

1. **Phase 0, contract (Orchestrator):** confirm the contract in section 2 and pass it to both agents word for word.
2. **Phase 1, build (parallel):**
   - **1a. Designer:** writes `app/styles.css`.
   - **1b. Coder:** works through its files in this order:
     1. `app/project-data.json`
     2. `app/index.html`
     3. `.vscode/launch.json`
3. **Phase 2, integration review (sequential):** the Designer looks at the rendered page and lists issues. Fixes to CSS go to the Designer. Fixes to markup or script go to the Coder. Never send both agents to the same file at the same time.
4. **Phase 3, validation (Orchestrator):** run the checks in section 9 and summarise what each agent contributed. Git operations stay with the learner.

## 6. Dependencies

- `app/index.html` depends on the JSON schema field names, so the data file is written first.
- `app/index.html` and `app/styles.css` both depend on the agreed CSS class names. The contract removes any runtime dependency between them.
- `.vscode/launch.json` depends only on `app/index.html` existing at its final path. It does not depend on its content.
- Phase 2 depends on Phase 1 being fully finished. Phase 3 depends on Phase 2.

## 7. Parallel work decisions

- **Parallel:** Designer (`app/styles.css`) and Coder (`app/index.html`, `app/project-data.json`, `.vscode/launch.json`) in Phase 1. Their file scopes don't overlap, and the locked contract is their only interface.
- **Sequential:** the Coder's own three files, because the HTML is written against the data. Phase 2 fixes, because review findings may touch either file and need a single owner per change. Final validation, because it needs the integrated result.
- **Rule:** if the Designer needs a new hook in the HTML, it asks the Orchestrator, and the Coder adds it in Phase 2. The Designer never edits `app/index.html`.

## 8. Edge cases

- **Fetch failure / non-OK response:** show the `.dashboard-message--error` message, including the status code, and log details with `console.error`.
- **file:// loading:** browsers block `fetch` on `file://`. The error message should tell the user to start "Run Project Pulse Dashboard". Do not duplicate the data inline, because the copies would drift apart.
- **Invalid JSON or missing `projects` array:** show the error state with a clear reason.
- **Empty `projects`:** show the empty state ("No projects yet") and zero counts.
- **Missing or unknown fields:** show "Not set" or "Unknown" and use the neutral badge. Skip non-object entries.
- **Long text:** use `overflow-wrap: anywhere` for long names and activity text.
- **Accessibility:**
  - One `h1`, with each card's name as an `h2`.
  - Card list built as `ul`/`li`.
  - Badge text that makes sense on its own, e.g. "Status: At risk".
  - `aria-live` for loading and error messages.
  - Visible focus outlines and AA contrast.
  - Respect `prefers-reduced-motion`.
- **Port 5500 already in use:** the server fails to start. Stop the other process first (see open questions).

## 9. Validation expectations

- **`app/index.html`:**
  - Contains "Project Pulse", `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
  - No console errors when served.
- **`app/styles.css`:**
  - Contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
  - Responsive at about 375px and 1280px wide.
- **`app/project-data.json`:**
  - `python3 -m json.tool app/project-data.json` succeeds.
  - Every project has `name`, `owner`, `status`, `recentActivity` and `priority`.
- **`.vscode/launch.json`:**
  - `python3 -m json.tool .vscode/launch.json` succeeds.
  - Contains "Run Project Pulse Dashboard", `${workspaceFolder}/app` and `index.html`.
- **Manual check:** in Run and Debug, start **Run Project Pulse Dashboard**. The browser should open `http://localhost:5500/index.html` and show the dashboard with cards, not a directory listing. Stop the server afterwards.
- **Regression:** `bash scripts/validate-exercise.sh` should still pass, apart from learner-file checks that only apply to the template.

## 10. Open questions

- Is the `node-terminal` launch type acceptable, or should it be a Python debug config? `node-terminal` is built into VS Code, needs no extension, and works in Codespaces.
- Python may buffer the "Serving HTTP" line so `serverReadyAction` never fires. If that happens, can the command use `python3 -u -m http.server 5500`? That differs from the exact command in the Step 3 prompt.
- In Codespaces, `openExternally` goes through port forwarding. Should `openExternally` be kept, or switched to the in-editor Simple Browser (`debugWithChrome` is not available)?
- Should the stats strip and `summary` field be required? They are optional in this plan.
- Is a fixed port 5500 fine, or is a fallback needed if it's taken?
