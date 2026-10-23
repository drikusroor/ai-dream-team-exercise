# Project Pulse implementation plan

## Goal

Build a lightweight static dashboard so contributors can quickly see project names, health/status, progress, owners, recent activity, priority or risk, and the next milestone with its due date. Present each project in a readable, responsive card with a text status badge; the dashboard must be accessible and open directly in the browser from VS Code.

## Roles and file assignments

| Owner | Assignment |
| --- | --- |
| Orchestrator | Coordinate the agents, agree on shared data and class names, resolve integration questions, and validate the finished dashboard. |
| Planner | Define the phases, dependencies, parallel work, and validation expectations in this plan. |
| Designer | Describe the information hierarchy, card layout, responsive behavior, and accessibility decisions before implementation; implement `app/styles.css`. |
| Coder | Implement `app/project-data.json`, `app/index.html`, and `.vscode/launch.json`; coordinate markup and data integration with the Designer. |

## Implementation phases

1. **Agree on the interface.** The Orchestrator asks the Designer to describe the dashboard hierarchy, project-card layout, responsive behavior, and accessibility decisions. The Orchestrator, Designer, and Coder agree on a top-level `projects` array and the required project fields: `name`, `owner`, `status` (project health), `progress` (0–100), `recentActivity`, `priority`, and `upcomingMilestone` (with a name and due date). Agree on the card and badge class names before implementation. Include a short contributor-friendly summary for each project.
2. **Build in parallel.** The Designer creates `app/styles.css`. The Coder creates `app/project-data.json` and `.vscode/launch.json`. The Coder may scaffold `app/index.html` at the same time, then completes its rendering once the shared field and class names are agreed. The page links to `styles.css`, loads `project-data.json`, and displays each project's health, numeric progress, owner, recent activity, and upcoming milestone.
3. **Integrate and review.** The Coder checks that the HTML, stylesheet, and data use the same field and class names. The Designer reviews readability, responsive behavior, and accessibility. Configure the launch entry to serve from `app/` and open `index.html`, not a directory listing.
4. **Validate and hand off.** The Orchestrator checks the app and launch configuration against the validation expectations below, then summarizes the result and any limitations in the final handoff.

## Dependencies and parallel work

- Agreeing on the data schema and CSS class names is a dependency for integrating `app/index.html` with `app/styles.css` and `app/project-data.json`.
- After those names are agreed, the Designer can work on `app/styles.css` in parallel with the Coder's `app/project-data.json` and `.vscode/launch.json`.
- The Coder can scaffold `app/index.html` in parallel, but final integration depends on the shared schema and class names.
- `.vscode/launch.json` depends on the `app/` directory and the `index.html` target. Final validation depends on all implementation files being complete.

## Validation expectations

- Confirm `app/project-data.json` and `.vscode/launch.json` are valid JSON.
- Confirm the project data has a top-level `projects` array and every project includes `name`, `owner`, `status`, `recentActivity`, and `priority`.
- Confirm every project includes `progress` from 0 to 100 and an `upcomingMilestone` with a name and due date.
- Confirm `app/index.html` links to `styles.css`, loads `project-data.json`, and renders project cards with health, numeric progress, owner, recent activity, priority, and upcoming milestone.
- Confirm `app/styles.css` styles the dashboard and project cards, including rounded corners and shadows, with readable responsive layout and status text that does not rely on color alone.
- Confirm `.vscode/launch.json` has a **Run Project Pulse Dashboard** configuration that serves the `app/` directory and opens `index.html`.
- Run the step 3 workflow checks, then launch the dashboard in VS Code and verify the browser shows the dashboard rather than a directory listing.
