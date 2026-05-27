# 14 - Shuffled Exam Construction

How the `?shuffle=true` query param turns into a uniquely-ordered attempt with a shareable exam code. Code lives at the bottom of `app/exam/[examId]/take/page.tsx`.

## Diagram

```mermaid
flowchart TD
  Start([Student clicks 'Start Fresh'<br/>with Shuffle toggle on]) --> Nav[/Navigate to/<br/>/exam/:id/take?shuffle=true/]
  Nav --> Server[Server component runs]
  Server --> Check{Existing<br/>IN_PROGRESS<br/>attempt?}

  Check -->|Yes| Resume[Resume — ignore shuffle param,<br/>use saved questionOrder]
  Resume --> Done([Render ExamClient<br/>with saved state])

  Check -->|No| IsShuffle{isShuffled?}
  IsShuffle -->|No| Plain[Natural order:<br/>questionOrder = empty,<br/>choiceOrders = undefined,<br/>examCode = undefined]
  Plain --> Create

  IsShuffle -->|Yes| ShuffleQ["shuffleArray(exam.questions)<br/>— Fisher-Yates"]
  ShuffleQ --> ShuffleC[For each question,<br/>shuffleArray of option keys]
  ShuffleC --> Code[generateExamCode<br/>— 8 random chars from<br/>unambiguous alphabet]
  Code --> Create

  Create[examAttempt.create<br/>with questionOrder,<br/>choiceOrders, examCode]
  Create --> Render([Render ExamClient<br/>with shuffled order,<br/>displayNumber = 1..N])

  classDef start fill:#1E2761,color:#FFFFFF
  classDef decision fill:#FEF3C7,color:#1A1A2E,stroke:#EAB308
  classDef plain fill:#EEF2FE,color:#1A1A2E,stroke:#1E2761
  classDef shuffle fill:#FEE2E2,color:#1A1A2E,stroke:#F96167
  classDef terminal fill:#DCFCE7,color:#166534,stroke:#16A34A

  class Start,Nav start
  class Check,IsShuffle decision
  class Server,Resume,Plain,Create plain
  class ShuffleQ,ShuffleC,Code shuffle
  class Done,Render terminal
```

## Result-page reconstruction

```mermaid
flowchart LR
  R[Result page renders] --> Has{questionOrder<br/>non-empty?}
  Has -->|Yes — shuffled| Sort[Sort responses by<br/>questionOrder index]
  Has -->|No — natural| Num[Sort responses by<br/>question.number]
  Sort --> Display[Display Q1, Q2, ...<br/>using choiceOrders<br/>to render options in seen order]
  Num --> Display

  classDef d fill:#FEF3C7,color:#1A1A2E,stroke:#EAB308
  classDef a fill:#EEF2FE,color:#1A1A2E,stroke:#1E2761
  class Has d
  class R,Sort,Num,Display a
```

## Notes

- **Exam code alphabet** excludes `0`, `O`, `1`, `I`, `L` — anything visually ambiguous when read aloud or written down.
- **Shuffle is per-attempt**, not per-user. Taking the same exam twice with shuffle gives you two different orderings.
- **`choiceOrders` is a `{ questionId: [originalKey, ...] }` map.** The array indexes display order; values are the original (`"a"`, `"b"`, ...) keys for looking up text and hints.
- **`displayNumber` is computed at render time** — the DB never stores "this question was shown as Q5." It's always derivable from `questionOrder.indexOf(questionId) + 1`.
- **Once an attempt is shuffled, it stays shuffled.** Resuming a shuffled attempt re-applies the saved order; you can't accidentally "unshuffle" mid-exam.
