# 01 — System architecture

The whole stack on one page. Use this when you're trying to figure out where a piece of work lives, or which service is responsible for a given concern.

## Diagram

```mermaid
flowchart TD
  subgraph clients["Clients"]
    Student["👨‍🎓 Student<br/>(browser or mobile web)"]
    Admin["👩‍💼 Admin<br/>(browser)"]
  end

  subgraph edge["Vercel Edge"]
    Routes["Next.js App Router<br/>RSC + Route Handlers"]
  end

  subgraph data["Data layer"]
    Prisma["Prisma Client<br/>(via Neon adapter)"]
    DB[("PostgreSQL<br/>Neon serverless")]
  end

  subgraph external["External services"]
    Akash["Akash Network<br/>Qwen3-30B-A3B<br/>(chat streaming)"]
    Resend["Resend<br/>(transactional email)"]
    Blob["Vercel Blob<br/>(image uploads)"]
    Altcha["ALTCHA<br/>(proof-of-work CAPTCHA)"]
  end

  Student -->|HTTPS| Routes
  Admin -->|HTTPS| Routes

  Routes <-->|"queries / mutations"| Prisma
  Prisma <--> DB

  Routes -->|"streamText()"| Akash
  Routes -->|"sendEmail()"| Resend
  Routes -->|"handleUpload()"| Blob
  Student -.->|"client-side puzzle"| Altcha
  Routes -.->|"verify solution"| Altcha

  classDef client fill:#CADCFC,stroke:#1E2761,color:#1A1A2E
  classDef edgeNode fill:#1E2761,stroke:#151B47,color:#FFFFFF
  classDef dataNode fill:#EEF2FE,stroke:#1E2761,color:#1A1A2E
  classDef ext fill:#FEE2E2,stroke:#F96167,color:#1A1A2E

  class Student,Admin client
  class Routes edgeNode
  class Prisma,DB dataNode
  class Akash,Resend,Blob,Altcha ext
```

## Notes

- **Everything goes through Next.js.** There is no separate API service. Route handlers (`app/api/**/route.ts`) and Server Components share the same `lib/prisma.ts` client.
- **Neon's serverless Postgres scales to zero.** Cold starts can add ~100–300 ms but cost is essentially zero when idle.
- **Akash is the only AI dependency.** Switching to OpenAI or Anthropic later means changing one file (`lib/akash.ts`) and the model string.
- **ALTCHA does NOT call our server during the puzzle.** The puzzle is solved entirely client-side; we only verify the proof on submit.
