# 18 - Stats Aggregation

How `getUserStats(userId)` in `lib/stats/get-user-stats.ts` builds the payload that powers `/my-stats`. It's data-heavy but kept fast by issuing queries in parallel and chaining dependent ones off their prerequisites.

## Diagram

```mermaid
flowchart TD
  Call([getUserStats userId]) --> Kick[Kick off 4 base queries<br/>in parallel]

  Kick --> Q1[userAttemptsPromise<br/>completed attempts + exam]
  Kick --> Q2[allExamsPromise<br/>all exams in DB]
  Kick --> Q3[userResponsesPromise<br/>all responses with question.category]
  Kick --> Q4[classroomMembershipsPromise<br/>which classrooms user is in]

  Q1 --> Chain1[".then(attempts =>"]
  Chain1 --> Q5[scopedScoresPromise<br/>all attempts for exams user has taken<br/>by all users for comparison]

  Q4 --> Chain2[".then(memberships =>"]
  Chain2 --> Q6[classroomResolutionPromise<br/>all member userIds in user's classrooms<br/>for classroom-scoped comparison]

  Q1 & Q2 & Q3 & Q5 & Q6 --> Wait[await Promise.all]

  Wait --> Compute[Compute in JS]

  Compute --> E1[examComparisons<br/>per-exam: userBest, classAvg,<br/>percentile, distribution buckets,<br/>leaderboards]
  Compute --> E2[Global leaderboards<br/>from full allAttemptScores]
  Compute --> E3[categoryDataMap<br/>per-exam + 'all'<br/>using EXAM_CATEGORIES constant<br/>to backfill empty topics]
  Compute --> E4[comparisonLabel<br/>'Classroom Avg' / 'Classrooms Avg' /<br/>'All Students']

  E1 & E2 & E3 & E4 --> Return([Return UserStatsData])

  classDef start fill:#1E2761,color:#FFFFFF
  classDef query fill:#EEF2FE,color:#1A1A2E,stroke:#1E2761
  classDef chain fill:#FEF3C7,color:#1A1A2E,stroke:#EAB308
  classDef compute fill:#FEE2E2,color:#1A1A2E,stroke:#F96167
  classDef terminal fill:#DCFCE7,color:#166534,stroke:#16A34A

  class Call start
  class Q1,Q2,Q3,Q4,Q5,Q6,Wait query
  class Chain1,Chain2,Kick chain
  class Compute,E1,E2,E3,E4 compute
  class Return terminal
```

## Why this shape

| Approach | Why we chose it |
|---|---|
| Parallel queries | Avoids serial waterfalls. 4 base queries fire together; dependent ones fire as soon as their prerequisite resolves. |
| Compute aggregates in JS, not SQL | The per-user dataset is small (typically < 200 attempts). Easier to read and modify than complex window-function SQL. |
| Backfill canonical categories | Showing only attempted topics hides the gaps. `EXAM_CATEGORIES` from `lib/exam-categories.ts` has the official syllabus; we use it to seed empty categories with `correctRate: -1`. |
| Compute classroom vs global separately | We can't filter on the way down because we want the global leaderboard regardless of classroom membership. |

## Notes

- **`-1` is the "no data" sentinel** for category correctRate. The chart component (`CategoryChart`) renders these as "No data" rather than 0%, so unattempted topics don't look like terrible performance.
- **The classroom-scoped view is automatic.** No toggle; if you're in a classroom, your comparison defaults to classroom.
- **Categories are sorted by chapter-section prefix** (e.g. `"8-2"` parses to `[8, 2]`), so the chart matches the official syllabus order rather than alphabetical.
- **No caching.** The function runs on every `/my-stats` render. Acceptable because page renders are infrequent and the queries are indexed.
