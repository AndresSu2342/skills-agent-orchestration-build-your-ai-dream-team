# Agent team for Mona's Project Pulse dashboard

I will use a custom multi-agent team under `.github/agents/` and orchestrate the work with GitHub Copilot CLI in a Codespace.

- Orchestrator (`.github/agents/orchestrator.agent.md`) — Model: Claude Opus 4.7 (copilot). Responsible for coordinating the overall build, breaking the work into phases, assigning file ownership, running tasks in parallel where safe, and checking that the final dashboard integrates cleanly.
- Planner (`.github/agents/planner.agent.md`) — Model: Claude Opus 4.7 (copilot). Responsible for researching the repo, reviewing libraries and documentation, identifying risks and edge cases, and producing the implementation roadmap for Project Pulse.
- Coder (`.github/agents/coder.agent.md`) — Model: GPT-5.5 (copilot). Responsible for implementing application code, fixing logic issues, and creating any necessary runnable app support files needed for the dashboard.
- Designer (`.github/agents/designer.agent.md`) — Model: Gemini 3.1 Pro (copilot). Responsible for dashboard UX, accessibility, information hierarchy, interaction flow, and styling decisions so the interface feels like a polished Project Pulse product.

How the team will work together:

1. The Planner researches the repository and maps the work into clear phases and file responsibilities.
2. The Orchestrator turns that plan into tasks, assigns work to the Designer and Coder, and coordinates sequencing so overlapping files do not conflict.
3. The Designer focuses on the front-end experience, layout, visual polish, and usability of the dashboard.
4. The Coder implements the application logic and any supporting configuration needed to run the dashboard locally.
5. The Orchestrator verifies that the pieces fit together and reports progress before the next phase begins.

This setup keeps planning, coordination, product design, and implementation clearly separated while allowing the team to build Project Pulse collaboratively inside the Codespace workflow.
