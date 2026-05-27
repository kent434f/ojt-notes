# ojt-notes

Documentation hub for planning, architecture, and flow diagrams.

## Quick Overview

This repository stores design and behavior notes as Markdown + Mermaid diagrams.

- `ganbare_app/`: product behavior and lifecycle documentation for the Ganbare app.
- `amara_college/`: Mermaid reference pack for *Collegio de Amara: The Pale Warden*.

## Project Index

<details>
  <summary><strong>ganbare_app</strong> — Ganbare app design and behavior docs</summary>

  **Summary:** status: **active** | owner: `team` | documents: **20** | last updated: **2026-05-27**

  | File | Type | What it covers |
  |---|---|---|
  | [`00-INDEX.md`](./ganbare_app/00-INDEX.md) | Diagram index | Catalog of Ganbare design references and how to navigate them |
  | [`01-system-architecture.md`](./ganbare_app/01-system-architecture.md) | Architecture diagram | High-level system architecture overview |
  | [`02-data-model.md`](./ganbare_app/02-data-model.md) | Data model | Entity relationships and storage design |
  | [`03-user-lifecycle.md`](./ganbare_app/03-user-lifecycle.md) | Sequence flow | User onboarding and account state transitions |
  | [`04-exam-attempt-states.md`](./ganbare_app/04-exam-attempt-states.md) | State diagram | Exam attempt state transitions |
  | [`05-study-question-states.md`](./ganbare_app/05-study-question-states.md) | State diagram | Study mode question progression |
  | [`06-registration-sequence.md`](./ganbare_app/06-registration-sequence.md) | Sequence flow | Registration + email verification |
  | [`07-login-sequence.md`](./ganbare_app/07-login-sequence.md) | Sequence flow | Login and authentication flow |
  | [`08-password-reset-sequence.md`](./ganbare_app/08-password-reset-sequence.md) | Sequence flow | Password reset and recovery flow |
  | [`09-exam-start-flow.md`](./ganbare_app/09-exam-start-flow.md) | Sequence flow | Exam startup flow and guard checks |
  | [`10-save-pause-sequence.md`](./ganbare_app/10-save-pause-sequence.md) | Sequence flow | Save/pause behavior and resume logic |
  | [`11-exam-submit-sequence.md`](./ganbare_app/11-exam-submit-sequence.md) | Sequence flow | Exam submission and grading |
  | [`12-exit-guard-flowchart.md`](./ganbare_app/12-exit-guard-flowchart.md) | Decision flow | Exit guard logic and branching paths |
  | [`13-ai-chat-sequence.md`](./ganbare_app/13-ai-chat-sequence.md) | Sequence flow | AI chat request lifecycle |
  | [`14-shuffle-creation.md`](./ganbare_app/14-shuffle-creation.md) | Process flow | Shuffled exam construction workflow |
  | [`15-password-protected-exam.md`](./ganbare_app/15-password-protected-exam.md) | Sequence flow | Password-protected exam unlock flow |
  | [`16-rbac-request.md`](./ganbare_app/16-rbac-request.md) | Access-control flow | RBAC permission request path |
  | [`17-classroom-roles.md`](./ganbare_app/17-classroom-roles.md) | Role model | Role matrix and responsibilities |
  | [`18-stats-data-flow.md`](./ganbare_app/18-stats-data-flow.md) | Data flow | Statistics aggregation and reporting flow |
  | [`19-rendering-decision.md`](./ganbare_app/19-rendering-decision.md) | Decision table | Server vs client rendering choice mapping |
</details>

<details>
  <summary><strong>amara_college</strong> — Narrative game reference diagrams</summary>

  **Summary:** status: **active** | owner: `team` | documents: **8** | last updated: **2026-05-27**

  | File | Type | What it covers |
  |---|---|---|
  | [`README.md`](./amara_college/README.md) | Index | Mermaid reference pack guidance and usage notes |
  | [`01-story-flow.md`](./amara_college/01-story-flow.md) | Flowchart | Four-act narrative structure and branching path |
  | [`02-clues-and-case-strength.md`](./amara_college/02-clues-and-case-strength.md) | State/decision flow | Clue propagation and case strength logic |
  | [`03-puzzle.md`](./amara_college/03-puzzle.md) | State machine | Puzzle card filtering and completion flow |
  | [`04-confrontation-and-endings.md`](./amara_college/04-confrontation-and-endings.md) | Decision flow | Confrontation flow and ending branches |
  | [`05-build-and-ci.md`](./amara_college/05-build-and-ci.md) | Process flow | Build, CI, and deployment checks |
  | [`06-architecture.md`](./amara_college/06-architecture.md) | Architecture | Core component and state ownership layout |
  | [`07-how-to.md`](./amara_college/07-how-to.md) | How-to flow | Common maintenance and contributor decision trees |
</details>

<details>
  <summary><strong>Upcoming projects</strong> — add here as new folders grow</summary>

  **Summary:** status: **planning** | owner: `TBD` | documents: **0** | last updated: **N/A**

  - `project_name/` (placeholder for next docs bundle)
  - `project_name/` (placeholder for next docs bundle)
  - `project_name/` (placeholder for next docs bundle)
</details>
