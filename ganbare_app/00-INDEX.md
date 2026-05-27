# Ganbare! — Diagram Reference Library

A grab-bag of focused Mermaid diagrams covering Ganbare!'s architecture, data model, and core flows. Each file is standalone — pick the one you need, open it in any Markdown viewer that renders Mermaid (VS Code with **Markdown Preview Mermaid Support**, GitHub, GitLab, Obsidian, Notion, etc.), and you should see the diagram inline.

## How to use this

- **Onboarding** — read them in order, top to bottom.
- **Debugging** — search for the flow you're touching and open just that file.
- **Reviewing a PR** — find the relevant state machine or sequence diagram to verify the change matches the intended flow.

If a diagram drifts out of sync with the code, fix the diagram — the source of truth is always `prisma/schema.prisma` and the actual route handlers.

---

## Architecture & data

| # | File | What it shows |
|---|------|---------------|
| 01 | [System architecture](./01-system-architecture.md) | The whole stack — browser, Next.js, DB, external services |
| 02 | [Data model (ERD)](./02-data-model.md) | All Prisma tables and their relations |

## State machines

| # | File | What it shows |
|---|------|---------------|
| 03 | [User lifecycle](./03-user-lifecycle.md) | Anonymous → registered → verified → admin |
| 04 | [Exam attempt states](./04-exam-attempt-states.md) | `IN_PROGRESS` ↔ `COMPLETED` and the transitions between them |
| 05 | [Study mode question states](./05-study-question-states.md) | `unanswered` → `trying` → `correct` / `revealed` |

## Auth & user management

| # | File | What it shows |
|---|------|---------------|
| 06 | [Registration & email verification](./06-registration-sequence.md) | The full sign-up + verify flow |
| 07 | [Login flow](./07-login-sequence.md) | Credentials sign-in with ALTCHA |
| 08 | [Password reset](./08-password-reset-sequence.md) | Forgot-password → reset link → new password |
| 16 | [Role-based access control](./16-rbac-request.md) | How a single request gets authorized |

## Exam-taking

| # | File | What it shows |
|---|------|---------------|
| 09 | [Starting an exam](./09-exam-start-flow.md) | What happens when a student clicks "Take Exam" |
| 10 | [Save & pause](./10-save-pause-sequence.md) | How in-progress state is persisted |
| 11 | [Submit & grading](./11-exam-submit-sequence.md) | Final submit, scoring, redirect to results |
| 12 | [Exit guard decision](./12-exit-guard-flowchart.md) | The three triggers and the dialog choices |
| 14 | [Shuffled exam construction](./14-shuffle-creation.md) | How question + choice ordering is randomized |
| 15 | [Password-protected exam unlock](./15-password-protected-exam.md) | The cookie-based unlock mechanism |

## AI study companion

| # | File | What it shows |
|---|------|---------------|
| 13 | [Chat request flow](./13-ai-chat-sequence.md) | A student message → Akash → streamed reply |

## Classrooms & stats

| # | File | What it shows |
|---|------|---------------|
| 17 | [Classroom roles](./17-classroom-roles.md) | Admin, member, and the join paths |
| 18 | [Stats aggregation](./18-stats-data-flow.md) | How `getUserStats` builds the stats payload |

## Frontend patterns

| # | File | What it shows |
|---|------|---------------|
| 19 | [Server vs client component decision](./19-rendering-decision.md) | When to reach for a client component |
