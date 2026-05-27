# 10 - Save and Pause

How an in-progress exam attempt persists its state. Code lives in `app/exam/[examId]/take/exam-client.tsx` (the client) and `app/api/exam/save/route.ts` (the API).

## Diagram

```mermaid
sequenceDiagram
  actor U as Student
  participant C as ExamClient (React)
  participant S as /api/exam/save
  participant DB as Postgres

  Note over U,DB: Triggered by "Save & Pause" button OR<br/>"Save & Exit" in the exit-guard dialog

  U->>C: Click "Save & Pause"
  C->>C: buildResponses()<br/>(combines saved + current<br/>question's elapsed time)

  C->>S: POST /api/exam/save<br/>{attemptId, remainingTimeSeconds,<br/>currentQuestionIndex, responses}

  S->>S: auth()
  S->>DB: examAttempt.findUnique<br/>(verify ownership)
  alt status != IN_PROGRESS
    S-->>C: 400 "Already completed"
    C-->>U: (handled silently or via error)
  end

  S->>DB: prisma.$transaction([
  Note right of DB: 1. delete all existing<br/>QuestionResponse for this attempt
  Note right of DB: 2. createMany new ones<br/>(isCorrect: false placeholder)
  Note right of DB: 3. update ExamAttempt<br/>{remainingTime, currentIdx, totalTime}
  S->>DB: ])
  DB-->>S: success

  S-->>C: 200 OK
  C->>C: router.push("/exam/:id")
  C->>U: Land on exam info page,<br/>showing "Resume Exam"
```

## Notes

- **Responses are replaced, not merged.** We delete then `createMany`. This is simpler and avoids tricky diff logic, and the response count is bounded (≤ totalQuestions).
- **`isCorrect` is set to `false` placeholder on save.** It gets recomputed on final submit. We don't grade until submission to avoid leaking correctness via timing or DB inspection.
- **`buildResponses` adds in the current question's unsaved elapsed time** without calling `recordQuestionTime` — that uses `setState` which is async and would give stale values.
- **The timer's `timeLeftRef`** is what gets sent as `remainingTimeSeconds`. We use a ref (not state) so the timer's per-second re-renders don't trigger a re-render of the whole exam component.
- **Submit reuses this same delete-then-createMany pattern**, but also sets `status: COMPLETED` and the real `isCorrect` values.
