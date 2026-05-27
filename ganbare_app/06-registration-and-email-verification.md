# 06 - Registration and Email Verification

The full path from "I'm new here" to "I have a verified account." Spans `app/register/page.tsx`, `app/api/register/route.ts`, `lib/tokens.ts`, `lib/email.ts`, and `app/verify-email/page.tsx`.

## Diagram

```mermaid
sequenceDiagram
  actor U as Student
  participant B as Browser
  participant S as Next.js Server
  participant CHA as ALTCHA lib
  participant DB as Postgres
  participant R as Resend

  U->>B: Fill register form
  B->>CHA: Solve PoW puzzle (client-side)
  CHA-->>B: Solution payload
  U->>B: Click "Create Account"

  B->>S: POST /api/register<br/>{name, email, password, altcha}
  S->>CHA: verifySolution(altcha)
  CHA-->>S: ✓ valid
  S->>S: validate email, password strength
  S->>DB: User.findUnique({email})
  DB-->>S: null (new user)
  S->>S: bcrypt.hash(password, 12)
  S->>DB: User.create({...})
  DB-->>S: user
  S->>S: createToken(email, "EMAIL_VERIFY")
  S->>DB: VerificationToken.create<br/>(stores SHA-256 hash only)
  S->>R: sendVerificationEmail(email, rawToken)
  R-->>U: 📧 Email with link<br/>/verify-email?token=...

  Note over S,DB: Registration response<br/>doesn't wait for email send

  S-->>B: 201 Created
  B->>B: Auto-login via signIn("credentials")
  B->>U: Redirect to /dashboard<br/>(banner shown: please verify)

  U->>B: Click email link (much later)
  B->>S: POST /api/auth/verify-email<br/>{token}
  S->>DB: VerificationToken.findFirst<br/>(by SHA-256 hash)
  DB-->>S: record
  S->>S: Check expiry
  S->>DB: User.update<br/>{emailVerified: now()}
  S->>DB: VerificationToken.delete
  S-->>B: 200 OK
  B->>B: session.update() to refresh JWT
  B->>U: ✓ Banner disappears
```

## Notes

- **The DB stores `SHA-256(token)`**, never the raw token. If the DB leaks, the leaked records can't be used as valid tokens.
- **Resend failure does NOT fail registration.** The registration handler wraps the email-send in try/catch and logs on failure — the user can always request a resend later.
- **Auto-login** happens client-side via NextAuth's `signIn("credentials", { redirect: false })` immediately after the registration POST succeeds.
- **The banner polls on `visibilitychange`** — when the user returns to the tab after clicking the email link, `update()` re-reads `emailVerified` from the DB.
- **Tokens expire after 24 hours** for `EMAIL_VERIFY`, 1 hour for `PASSWORD_RESET`.
