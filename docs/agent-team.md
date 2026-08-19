# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent custom team defined under `.github/agents/`, orchestrated with GitHub Copilot CLI in a Codespace.

- **Orchestrator** — Model: Claude Opus 4.7 (copilot). Coordinates the Planner, Coder, and Designer agents: breaks the request into phases, assigns non-overlapping file scopes, decides what can run in parallel vs. sequentially, and reports the final integrated outcome. Defined in `.github/agents/orchestrator.agent.md`.
- **Planner** — Model: Claude Opus 4.7 (copilot). Researches the repository and requirements, then produces an implementation plan (ordered steps, file assignments, dependencies, parallelizable vs. sequential work, edge cases, and validation expectations) without writing code. Defined in `.github/agents/planner.agent.md`.
- **Coder** — Model: GPT-5.5 (copilot). Implements the dashboard logic and code within its assigned file scope, including support files like `.vscode/launch.json` (configured to run from the `app` folder and open `index.html`) for Project Pulse. Defined in `.github/agents/coder.agent.md`.
- **Designer** — Model: Gemini 3.1 Pro (copilot). Owns UI/UX, accessibility, and visual design for the dashboard — project cards, status badges, responsive layout, and deterministic CSS hooks such as `.dashboard` and `.project-card`. Defined in `.github/agents/designer.agent.md`.

I'm using GitHub Copilot CLI in a Codespace to orchestrate this team: the Orchestrator agent delegates to Planner, Coder, and Designer, and all git operations (staging, committing, pushing) remain under my control rather than the agents'.
