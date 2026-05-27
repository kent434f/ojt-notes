# OJT Notes

Documentation hub for planning, architecture, and flow diagrams.

## Quick Overview

This repository stores design and behavior notes as Markdown + Mermaid diagrams.

- `ganbare_app/`: product behavior and lifecycle documentation for the Ganbare app.
- `amara_college/`: narrative game reference diagrams for *Collegio de Amara: The Pale Warden*.

## Project Index

<details>
  <summary><strong>ganbare_app</strong> - Ganbare app design and behavior docs</summary>

  **Summary:** status: **active** | owner: `team` | documents: **20** | last updated: **2026-05-27**

  | File | Type | What it covers |
  |---|---|---|
  | [`README.md`](./ganbare_app/README.md) | Project index | Catalog of Ganbare design references and how to navigate them |
  | [`01-system-architecture.md`](./ganbare_app/01-system-architecture.md) | Architecture diagram | High-level system architecture overview |
  | [`02-data-model.md`](./ganbare_app/02-data-model.md) | Data model | Entity relationships and storage design |
  | [`03-user-lifecycle.md`](./ganbare_app/03-user-lifecycle.md) | Sequence flow | User onboarding and account state transitions |
  | [`04-exam-attempt-states.md`](./ganbare_app/04-exam-attempt-states.md) | State diagram | Exam attempt state transitions |
  | [`05-study-mode-question-states.md`](./ganbare_app/05-study-mode-question-states.md) | State diagram | Study mode question progression |
  | [`06-registration-and-email-verification.md`](./ganbare_app/06-registration-and-email-verification.md) | Sequence flow | Registration + email verification |
  | [`07-login-flow.md`](./ganbare_app/07-login-flow.md) | Sequence flow | Login and authentication flow |
  | [`08-password-reset.md`](./ganbare_app/08-password-reset.md) | Sequence flow | Password reset and recovery flow |
  | [`09-starting-an-exam.md`](./ganbare_app/09-starting-an-exam.md) | Sequence flow | Exam startup flow and guard checks |
  | [`10-save-and-pause.md`](./ganbare_app/10-save-and-pause.md) | Sequence flow | Save/pause behavior and resume logic |
  | [`11-submit-and-grading.md`](./ganbare_app/11-submit-and-grading.md) | Sequence flow | Exam submission and grading |
  | [`12-exit-guard-decision.md`](./ganbare_app/12-exit-guard-decision.md) | Decision flow | Exit guard logic and branching paths |
  | [`13-ai-chat-request-flow.md`](./ganbare_app/13-ai-chat-request-flow.md) | Sequence flow | AI chat request lifecycle |
  | [`14-shuffled-exam-construction.md`](./ganbare_app/14-shuffled-exam-construction.md) | Process flow | Shuffled exam construction workflow |
  | [`15-password-protected-exam-unlock.md`](./ganbare_app/15-password-protected-exam-unlock.md) | Sequence flow | Password-protected exam unlock flow |
  | [`16-role-based-access-control.md`](./ganbare_app/16-role-based-access-control.md) | Access-control flow | RBAC permission request path |
  | [`17-classroom-roles.md`](./ganbare_app/17-classroom-roles.md) | Role model | Role matrix and responsibilities |
  | [`18-stats-aggregation.md`](./ganbare_app/18-stats-aggregation.md) | Data flow | Statistics aggregation and reporting flow |
  | [`19-server-vs-client-component-decision.md`](./ganbare_app/19-server-vs-client-component-decision.md) | Decision table | Server vs client rendering choice mapping |
</details>

<details>
  <summary><strong>amara_college</strong> - Narrative game reference diagrams</summary>

  **Summary:** status: **active** | owner: `team` | documents: **8** | last updated: **2026-05-27**

  | File | Type | What it covers |
  |---|---|---|
  | [`README.md`](./amara_college/README.md) | Project index | Mermaid reference pack guidance and usage notes |
  | [`01-story-flow.md`](./amara_college/01-story-flow.md) | Flowchart | Four-act narrative structure and branching path |
  | [`02-clues-and-case-strength.md`](./amara_college/02-clues-and-case-strength.md) | State/decision flow | Clue propagation and case strength logic |
  | [`03-clue-reconstruction-puzzle.md`](./amara_college/03-clue-reconstruction-puzzle.md) | State machine | Puzzle card filtering and completion flow |
  | [`04-confrontation-and-endings.md`](./amara_college/04-confrontation-and-endings.md) | Decision flow | Confrontation flow and ending branches |
  | [`05-build-and-ci.md`](./amara_college/05-build-and-ci.md) | Process flow | Build, CI, and deployment checks |
  | [`06-architecture.md`](./amara_college/06-architecture.md) | Architecture | Core component and state ownership layout |
  | [`07-how-to-decision-trees.md`](./amara_college/07-how-to-decision-trees.md) | How-to flow | Common maintenance and contributor decision trees |
</details>

<details>
  <summary><strong>Upcoming projects</strong> - Add here as new folders grow</summary>

  **Summary:** status: **planning** | owner: `TBD` | documents: **0** | last updated: **N/A**

  - `project_name/` (placeholder for next docs bundle)
  - `project_name/` (placeholder for next docs bundle)
  - `project_name/` (placeholder for next docs bundle)
</details>
