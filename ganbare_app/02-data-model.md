# 02 — Data model (ERD)

Every Prisma table and how they relate. The source of truth is `prisma/schema.prisma` — if this diagram drifts, fix the diagram.

## Diagram

```mermaid
erDiagram
  User ||--o{ ExamAttempt : "takes"
  User ||--o{ ClassroomMember : "joins via"
  User ||--o{ ChatMessage : "sends"
  User ||--o{ Classroom : "creates"

  Classroom ||--o{ ClassroomMember : "has"

  Exam ||--o{ Question : "contains"
  Exam ||--o{ ExamAttempt : "is attempted as"
  Exam ||--o{ ChatMessage : "scopes"

  Question ||--o{ QuestionResponse : "answered in"
  Question ||--o{ ChatMessage : "scopes"

  ExamAttempt ||--o{ QuestionResponse : "contains"

  User {
    string id PK
    string name
    string email UK
    string passwordHash
    DateTime emailVerified "nullable"
    Role role "STUDENT or ADMIN"
    DateTime createdAt
  }

  VerificationToken {
    string id PK
    string tokenHash "SHA-256, never raw"
    string type "EMAIL_VERIFY or PASSWORD_RESET"
    string email
    DateTime expiresAt
    DateTime createdAt
  }

  Classroom {
    string id PK
    string name
    string description "nullable"
    string inviteCode UK "6 chars, no ambiguous"
    string createdById FK
    DateTime createdAt
    DateTime updatedAt
  }

  ClassroomMember {
    string id PK
    string classroomId FK
    string userId FK
    DateTime joinedAt
  }

  Exam {
    string id PK
    string title
    string date
    int year
    string season "S or A"
    string examCategory "IP, FE, etc."
    string subject "nullable"
    int timeLimitMinutes
    int totalQuestions
    string password "nullable, exam lock"
  }

  Question {
    string id PK
    int number
    string category "nullable"
    string body
    string codeBlock "nullable"
    string table "nullable, GFM"
    string image "nullable, blob URL"
    string imageDescription "nullable"
    string questionType
    json options "OptionsMap"
    string correctAnswer "key into options"
    string explanation "legacy"
    string rationale "nullable, rich MD"
    stringArray rationaleImages
    string rationaleVideo "nullable"
    string examId FK
  }

  ExamAttempt {
    string id PK
    string userId FK
    string examId FK
    int score
    int totalTime "seconds"
    DateTime startedAt
    DateTime completedAt "nullable"
    AttemptStatus status "IN_PROGRESS or COMPLETED"
    int remainingTimeSeconds "nullable"
    int currentQuestionIndex
    stringArray questionOrder "for shuffle"
    json choiceOrders "nullable, for shuffle"
    string examCode UK "nullable, shuffle code"
  }

  QuestionResponse {
    string id PK
    string attemptId FK
    string questionId FK
    string selectedAnswer "nullable"
    boolean isCorrect
    int timeSpent "seconds"
    boolean flagged
  }

  ChatMessage {
    string id PK
    string userId FK
    string examId FK
    string questionId FK
    string role "user or assistant"
    string content
    DateTime createdAt
  }
```

## Notes

- **`VerificationToken` has no FK to `User`** — it identifies by email so password-reset works even if the user record's email gets updated mid-flow.
- **`options` is JSON, not separate columns** — the migration `20260215150000_options_json_column` consolidated `optionA..J` + hints into one `OptionsMap`. See `lib/question-options.ts`.
- **Per-question rows in `QuestionResponse`** make per-question analytics cheap. A single JSON blob per attempt would have made "which question is hardest?" much harder.
- **`questionOrder` and `choiceOrders`** are only populated for shuffled attempts. Empty → render in natural order.
- **`examCode` is unique** so a teacher can look up a specific shuffled variant.
