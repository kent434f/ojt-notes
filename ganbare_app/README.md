# Ganbare App

## Overview

Focused Mermaid diagrams covering Ganbare app architecture, data modeling, user flows, exam behavior, classroom behavior, and frontend rendering decisions.

## How to use

- **Onboarding:** read the files in numeric order.
- **Debugging:** open the flow or state diagram closest to the feature being changed.
- **Reviewing:** compare code changes against the relevant state machine, sequence diagram, or decision table.

If a diagram drifts out of sync with the app, update the diagram. The implementation source of truth is the codebase, especially `prisma/schema.prisma` and route handlers.

## Files

| # | File | Type | What it covers |
|---|---|---|---|
| 01 | [`01-system-architecture.md`](./01-system-architecture.md) | Architecture diagram | Browser, Next.js, database, and external service boundaries |
| 02 | [`02-data-model.md`](./02-data-model.md) | Data model | Prisma tables, relationships, and persistence notes |
| 03 | [`03-user-lifecycle.md`](./03-user-lifecycle.md) | State flow | User onboarding and account state transitions |
| 04 | [`04-exam-attempt-states.md`](./04-exam-attempt-states.md) | State diagram | Exam attempt states and allowed transitions |
| 05 | [`05-study-mode-question-states.md`](./05-study-mode-question-states.md) | State diagram | Study mode answer states and UI mapping |
| 06 | [`06-registration-and-email-verification.md`](./06-registration-and-email-verification.md) | Sequence flow | Registration and email verification |
| 07 | [`07-login-flow.md`](./07-login-flow.md) | Sequence flow | Login and authentication flow |
| 08 | [`08-password-reset.md`](./08-password-reset.md) | Sequence flow | Password reset and recovery behavior |
| 09 | [`09-starting-an-exam.md`](./09-starting-an-exam.md) | Sequence flow | Exam startup flow and guard checks |
| 10 | [`10-save-and-pause.md`](./10-save-and-pause.md) | Sequence flow | Save, pause, and resume behavior |
| 11 | [`11-submit-and-grading.md`](./11-submit-and-grading.md) | Sequence flow | Final submit, grading, and result redirect |
| 12 | [`12-exit-guard-decision.md`](./12-exit-guard-decision.md) | Decision flow | Exit guard triggers and dialog outcomes |
| 13 | [`13-ai-chat-request-flow.md`](./13-ai-chat-request-flow.md) | Sequence flow | AI chat request lifecycle |
| 14 | [`14-shuffled-exam-construction.md`](./14-shuffled-exam-construction.md) | Process flow | Shuffled exam and choice ordering |
| 15 | [`15-password-protected-exam-unlock.md`](./15-password-protected-exam-unlock.md) | Sequence flow | Password-protected exam unlock behavior |
| 16 | [`16-role-based-access-control.md`](./16-role-based-access-control.md) | Access-control flow | Role checks, ownership checks, and route-level authorization |
| 17 | [`17-classroom-roles.md`](./17-classroom-roles.md) | Role model | Classroom roles, permissions, and join paths |
| 18 | [`18-stats-aggregation.md`](./18-stats-aggregation.md) | Data flow | Stats aggregation and reporting payloads |
| 19 | [`19-server-vs-client-component-decision.md`](./19-server-vs-client-component-decision.md) | Decision table | Server vs client component selection |

## Mermaid notes

Most files use Mermaid diagrams that render in GitHub Markdown, VS Code with Mermaid preview support, Obsidian, Typora, and similar Markdown tools.
