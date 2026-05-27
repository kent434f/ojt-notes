# 17 - Classroom Roles

How users, classrooms, and the join paths fit together. The schema has only two role enums (`STUDENT`, `ADMIN`), but classrooms add a per-classroom membership layer.

## Diagram

```mermaid
flowchart TD
  subgraph users["Global roles"]
    Admin["👩‍💼 ADMIN user"]
    Student["👨‍🎓 STUDENT user"]
  end

  subgraph classroom["A Classroom"]
    C[("Classroom<br/>{name, description,<br/>inviteCode}")]
    M1["ClassroomMember<br/>(student #1)"]
    M2["ClassroomMember<br/>(student #2)"]
    M3["..."]
  end

  Admin -->|creates<br/>POST /api/classrooms| C
  Admin -->|owns via<br/>createdById| C

  C -->|has| M1
  C -->|has| M2
  C -->|has| M3

  Student -->|"joins via invite code<br/>POST /api/classrooms/join"| M1
  Admin -->|"adds by email<br/>POST /api/classrooms/:id/members"| M2

  classDef admin fill:#FEE2E2,stroke:#F96167,color:#1A1A2E
  classDef student fill:#EEF2FE,stroke:#1E2761,color:#1A1A2E
  classDef classroom fill:#1E2761,color:#FFFFFF
  classDef member fill:#FEF3C7,stroke:#EAB308,color:#1A1A2E

  class Admin admin
  class Student student
  class C classroom
  class M1,M2,M3 member
```

## Permission matrix

```mermaid
flowchart LR
  subgraph capabilities["What you can do in a classroom"]
    direction TB
    P1["View roster"]
    P2["View group stats"]
    P3["Add / remove members"]
    P4["Delete classroom"]
    P5["See full member emails"]
  end

  M[Member] --> P1
  M --> P2

  CR[Creator admin] --> P1
  CR --> P2
  CR --> P3
  CR --> P4
  CR --> P5

  OA[Other admins] --> P1
  OA --> P2
  OA --> P3
  OA --> P4
  OA --> P5

  classDef cap fill:#EEF2FE,color:#1A1A2E,stroke:#1E2761
  classDef role fill:#1E2761,color:#FFFFFF
  class P1,P2,P3,P4,P5 cap
  class M,CR,OA role
```

## Notes

- **Members see *redacted* emails** of other members (e.g. `m•••••a@example.com`) — only their own email is shown in full. See `lib/email-redact.ts`.
- **There's no "co-admin" or "moderator" role per classroom.** It's binary: admin (global) or member.
- **Deleting a user is blocked if they created classrooms** — they'd orphan all the members. The admin has to reassign or delete the classrooms first.
- **Invite codes are 6 chars from a safe alphabet** (no ambiguous chars) — see `lib/invite-code.ts`. Collisions are retried up to 3 times on `P2002` unique-constraint errors.
- **A user can be in many classrooms.** The stats label adapts: "Classroom Avg" for one, "Classrooms Avg" for multiple, "All Students" for none.
