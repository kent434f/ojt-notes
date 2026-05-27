# 08 - Password Reset

Forgot-password flow end to end. Spans `app/forgot-password/page.tsx`, `app/api/auth/forgot-password/route.ts`, `app/api/auth/reset-password/route.ts`, and `app/reset-password/page.tsx`.

## Diagram

```mermaid
sequenceDiagram
  actor U as User
  participant B as Browser
  participant S as Next.js Server
  participant DB as Postgres
  participant R as Resend

  Note over U,R: Step 1 — Request reset

  U->>B: Open /forgot-password,<br/>enter email, solve ALTCHA
  B->>S: POST /api/auth/forgot-password
  S->>S: verifySolution(altcha)
  S->>DB: User.findUnique({email})

  alt user exists
    S->>DB: VerificationToken.findFirst<br/>(within last 60s)
    alt no recent token
      S->>S: createToken(email, "PASSWORD_RESET")
      S->>DB: VerificationToken.create
      S->>R: sendPasswordResetEmail
    else rate-limited
      Note over S: Silently skip<br/>(don't reveal anything)
    end
  end

  Note over S,B: Always returns same message<br/>(prevents email enumeration)

  S-->>B: 200 "If account exists, we sent a link"
  B->>U: Show success message

  Note over U,R: Step 2 — Use reset link (might be hours later)

  U->>B: Click email link<br/>/reset-password?token=...
  B->>U: Show form
  U->>B: Enter new password,<br/>confirm, solve ALTCHA

  B->>S: POST /api/auth/reset-password<br/>{token, password, altcha}
  S->>S: verifySolution(altcha)
  S->>S: isPasswordStrong(password)
  S->>DB: validateToken(token, "PASSWORD_RESET")
  DB-->>S: token record (or null if expired)

  alt valid token
    S->>S: bcrypt.hash(password, 12)
    S->>DB: User.update<br/>{passwordHash, emailVerified: now()}
    S->>DB: consumeToken(token)
    S-->>B: 200 OK
    B->>U: Show success, link to /login
  else invalid/expired
    S-->>B: 400 "Invalid or expired token"
  end
```

## Notes

- **Email enumeration protection** — the response is identical whether the email exists or not. The 60-second rate limit also doesn't leak existence.
- **Successful reset also verifies the email.** Clicking the link is proof of inbox access, so we don't make them re-verify separately.
- **Tokens are single-use** — `consumeToken` deletes the row inside the same flow. Even if the user clicks the link twice, the second click fails.
- **No session is invalidated on password reset.** This is a known gap — if an attacker is currently logged in with the old password, they stay logged in. We accept this because JWT revocation is expensive and the threat model assumes the attacker doesn't have the JWT.
