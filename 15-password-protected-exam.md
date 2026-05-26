# 15 — Password-protected exam unlock

For exams with `exam.password` set, the student has to enter it before the take page will render. The unlock is cookie-based and lasts 4 hours.

## Diagram

```mermaid
sequenceDiagram
  actor U as Student
  participant B as Browser
  participant Info as /exam/:id<br/>(info page)
  participant Verify as POST /api/exam/<br/>:id/verify-password
  participant Take as /exam/:id/take<br/>(take page, RSC)
  participant DB as Postgres

  U->>B: Open /exam/:id
  B->>Info: GET
  Info-->>U: Show info + "🔒 Password required"

  U->>B: Click "Timed Exam",<br/>enter password
  B->>Verify: POST {password}
  Verify->>DB: exam.findUnique({id})
  DB-->>Verify: { password: "secret" }

  alt password matches
    Verify->>Verify: Read exam_access cookie<br/>(JSON array of examIds)
    Verify->>Verify: Push examId to list<br/>if not present
    Verify-->>B: Set-Cookie exam_access<br/>(httpOnly, sameSite: lax,<br/>maxAge: 4h)
    B->>Take: GET /exam/:id/take
    Take->>Take: auth()
    Take->>Take: Read exam_access cookie
    Take->>Take: Check examId in list
    Take-->>U: ✓ Render exam
  else password wrong
    Verify-->>B: 403 "Incorrect password"
    B-->>U: Show error
  end
```

## Cookie shape

```mermaid
flowchart LR
  C["exam_access cookie<br/>(HTTP-only, signed by Next.js)"]
  C --> J["JSON: ['examId1', 'examId2', ...]"]
  J --> Note["Pushed-only — never reset.<br/>Expires after 4h of no activity."]

  classDef c fill:#1E2761,color:#FFFFFF
  classDef d fill:#EEF2FE,color:#1A1A2E,stroke:#1E2761
  class C c
  class J,Note d
```

## Notes

- **The take page also checks the cookie**, not just the verify endpoint. Defense in depth — even if someone manually navigates to `/exam/:id/take`, the cookie is required.
- **Admins bypass the password gate** in both the take and study pages. They need to preview locked exams.
- **The cookie is `httpOnly`** so JavaScript can't read or forge it. The same-site `lax` setting allows top-level navigations to carry it.
- **4-hour expiry** is intentional — long enough to span a single exam session (most are < 2 hours) plus a re-take or two, short enough that a shared computer doesn't leak access.
- **Multiple exams can be unlocked at once.** The cookie holds an array, not a single ID.
