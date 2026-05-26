# 09 — Starting an exam

What happens when a verified student clicks "Take Exam" on an exam they've never started before. Code lives in `app/exam/[examId]/take/page.tsx`.

## Diagram

```mermaid
sequenceDiagram
  actor U as Student
  participant B as Browser
  participant S as Next.js Server (RSC)
  participant DB as Postgres

  U->>B: Click "Take Exam"<br/>(optionally with ?shuffle=true)
  B->>S: GET /exam/:id/take

  S->>S: auth() — must be logged in
  S->>S: Must have emailVerified

  par fetch in parallel
    S->>DB: exam.findUnique<br/>(with questions)
    S->>DB: examAttempt.findFirst<br/>{userId, examId, IN_PROGRESS}
  end

  alt exam.password set AND not admin
    S->>S: Check exam_access cookie
    alt cookie missing examId
      S-->>B: redirect → /exam/:id
      Note over B: Password gate (see 15-password-protected-exam)
    end
  end

  alt existing IN_PROGRESS attempt found
    Note over S: RESUME path
    S->>S: Reorder questions per saved questionOrder
    S->>S: Build initialAnswers, initialFlagged,<br/>initialQuestionTimes from saved responses
    S-->>B: Render ExamClient with saved state
  else no existing attempt
    Note over S: FRESH START path
    S->>S: Maybe shuffle questions + choices
    S->>S: Maybe generate examCode
    S->>DB: examAttempt.create<br/>{IN_PROGRESS, remainingTimeSeconds,<br/>questionOrder, choiceOrders, examCode}
    S-->>B: Render ExamClient with fresh state
  end

  B->>U: Show first question,<br/>timer starts counting down
```

## Notes

- **`auth()` and `params`/`searchParams` are awaited in parallel** with `Promise.all` to avoid a serial waterfall. Saves 50–100ms.
- **The exam row is created server-side on render**, not on first answer. This means you'll see an `IN_PROGRESS` row even for a student who opens the page and leaves immediately. Abandon cleans these up.
- **The resume branch re-orders questions** to match the original shuffled order. Without this, a resumed shuffled attempt would suddenly show questions in a different order.
- **Admins bypass the password gate.** This is intentional — they need to be able to preview locked exams.
- **`displayNumber` is computed at render time** — for shuffled attempts it's the position (1-based), for natural-order it's `question.number`.
