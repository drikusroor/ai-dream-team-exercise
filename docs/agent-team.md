# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate four custom agents defined in `.github/agents/`:

| Agent | Model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Breaks the Project Pulse request into phases, delegates work with explicit file scopes, coordinates dependencies, and verifies the integrated result. It coordinates the team rather than implementing files itself. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 | Researches the repository and produces a practical implementation plan with ordered steps, file ownership, dependencies, parallel work, edge cases, and validation expectations. It plans but does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 | Implements the static dashboard and assigned support files using clear, deterministic, testable code. It validates its changes and reports remaining risks. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash | Defines the dashboard's UI/UX, accessibility, information hierarchy, responsive behavior, and polished visual treatment, including project cards and status indicators. | `.github/agents/designer.agent.md` |

## How the team will build Project Pulse

The Orchestrator will first ask the Planner to turn Mona's requirements into implementation phases with clear file assignments and dependencies. It will then delegate the dashboard experience and styling decisions to the Designer and the static application implementation to the Coder. Design and coding tasks can run in parallel only when their file scopes do not overlap; dependent or overlapping work will run sequentially. Finally, the Orchestrator will verify that the combined HTML, CSS, JSON data, and launch configuration form a coherent, runnable Project Pulse dashboard and will summarize each specialist's contribution.
