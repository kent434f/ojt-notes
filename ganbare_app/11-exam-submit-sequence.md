# 11 — Submit & grading

What happens when the student hits "Submit Exam" (or the timer runs out and auto-submits). Code lives in `app/api/exam/submit/route.ts`.

## Diagram

```mermaid
sequenceDiagram
  actor U as Student
  participant C as ExamClient (React)
  participant S as /api/exam/submit
  participant DB as Postgres

  alt timer hits 0
    Note over C: CountdownTimer fires<br/>onTimeUp() callback
  else manual submit
    U->>C: Click "Review & Submit"<br/>then "Submit Exam"
  end

  C->>C: buildResponses()<br/>(with current question's elapsed time)
  C->>S: POST /api/exam/submit<br/>{examId, attemptId, startedAt, responses}

  S->>S: auth()
  S->>DB: question.findMany({examId})<br/>(select id, correctAnswer)
  DB-->>S: correct answer map

  S->>S: Grade each response<br/>(isCorrect = selectedAnswer === correct)
  S->>S: Sum score, totalTime

  alt attemptId provided (normal path)
    S->>DB: examAttempt.findUnique<br/>(verify ownership)
    S->>DB: prisma.$transaction([
    Note right of DB: 1. delete existing responses
    Note right of DB: 2. createMany graded responses
    Note right of DB: 3. update attempt:<br/>{score, status: COMPLETED,<br/>completedAt: now(),<br/>remainingTimeSeconds: null}
    S->>DB: ])
  else no attemptId (edge case — row was lost)
    S->>DB: examAttempt.create<br/>{COMPLETED, responses}
  end

  S-->>C: {attemptId, score}
  C->>C: router.push(<br/>/exam/:id/result/:attemptId)
  C->>U: Show score + per-question breakdown
```

## Notes

- **Grading happens server-side, never client-side.** The client doesn't know the correct answers — that data is filtered out of the take-page query (`select` only includes question metadata, not `correctAnswer`).
- **Auto-submit on timeout** is fired by the `CountdownTimer` component's `onTimeUp` callback. A `firedRef` ensures it only fires once even if the interval ticks again.
- **The fallback `examAttempt.create` path** is for the case where someone submits without ever creating an in-progress row. In practice this shouldn't happen — the take page always creates one — but the code defends against it.
- **`completedAt` is set on the server side** with `new Date()`, not from the client. We don't trust client clocks for scoring or analytics.
- **The result page is its own route** so refreshing it doesn't accidentally re-submit. Idempotent by design.
