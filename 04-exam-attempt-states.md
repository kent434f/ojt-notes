# 04 — Exam attempt states

How an `ExamAttempt` row moves through its lifecycle. Read this before touching anything in `/api/exam/*`.

## Diagram

```mermaid
stateDiagram-v2
  [*] --> NotStarted

  NotStarted --> InProgress : Student clicks "Take Exam"<br/>(GET /exam/:id/take creates the row)

  state InProgress {
    [*] --> Answering
    Answering --> Answering : Pick option,<br/>flag, navigate
    Answering --> Saved : "Save & Pause" or<br/>auto-save
    Saved --> Answering : Resume from dashboard
  }

  InProgress --> Completed : Submit or<br/>timer hits 0
  InProgress --> [*] : Abandon<br/>(POST /api/exam/abandon)

  Completed --> Completed : View results,<br/>retake (creates new row)

  Completed --> [*] : Admin deletes user
```

## Stored fields per state

```mermaid
flowchart LR
  IP["IN_PROGRESS"]
  CP["COMPLETED"]

  IP -.has.-> IPF["score = 0<br/>completedAt = null<br/>remainingTimeSeconds = N<br/>currentQuestionIndex = K<br/>questionOrder = [...]<br/>responses = (latest snapshot)"]

  CP -.has.-> CPF["score = final<br/>completedAt = now()<br/>remainingTimeSeconds = null<br/>currentQuestionIndex = (last value)<br/>responses = graded (isCorrect set)"]

  classDef state fill:#1E2761,color:#FFFFFF,stroke:#151B47
  classDef fields fill:#EEF2FE,color:#1A1A2E,stroke:#1E2761
  class IP,CP state
  class IPF,CPF fields
```

## Notes

- **There's no separate "saved" status** — saving just updates the existing `IN_PROGRESS` row. The `Saved` substate above is conceptual, not in the schema.
- **Abandon is a hard delete**, inside a transaction with `QuestionResponse.deleteMany`. We chose this over a soft `ABANDONED` status because abandoned attempts have zero analytical value and we don't want them polluting averages.
- **Submitting from an existing in-progress row** updates it in place. Submitting without an `attemptId` (rare, only if the row was lost) creates a new completed row directly.
- **Only one `IN_PROGRESS` row per (user, exam)** by convention. The take page looks for one with `findFirst({ status: IN_PROGRESS })` and resumes it; otherwise it creates one.
