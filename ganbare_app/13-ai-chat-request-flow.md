# 13 - AI Chat Request Flow

A single student message in Study Mode's AI tab, from typing to streamed response. Code: `components/study-chatbot.tsx` and `app/api/chat/route.ts`.

## Diagram

```mermaid
sequenceDiagram
  actor U as Student
  participant C as StudyChatbot (React)
  participant SDK as useChat hook<br/>(@ai-sdk/react)
  participant S as /api/chat
  participant DB as Postgres
  participant A as Akash<br/>(Qwen3-30B)

  U->>C: Type "Why isn't B correct?",<br/>press Enter
  C->>C: Check turn count<br/>(must be < 10)
  C->>SDK: sendMessage({text})

  SDK->>S: POST /api/chat<br/>{messages, questionContext, examContext}

  S->>S: auth() — must be verified
  S->>S: Check user turn count ≤ 10
  S->>DB: chatMessage.create<br/>(user message, for audit log)

  S->>S: buildSystemPrompt(<br/>question, exam)<br/>— bakes in correct answer,<br/>rationale, hints, safety rules
  S->>S: convertToModelMessages(messages)

  S->>A: streamText({<br/>model: Qwen3-30B,<br/>system, messages,<br/>maxOutputTokens: 1024})

  loop For each token
    A-->>S: text chunk
    S-->>SDK: SSE chunk
    SDK-->>C: append to message
    C-->>U: tokens appear<br/>(streaming)
  end

  A-->>S: stream complete
  S->>DB: chatMessage.create<br/>(assistant message,<br/>via onFinish callback)

  S-->>SDK: stream end
  SDK->>C: status: "ready"
  C->>U: Show full reply
```

## Notes

- **The system prompt is rebuilt every request** with the full question context. The model never has to "remember" the question across turns — it's always re-injected.
- **Both user and assistant messages are logged** to `ChatMessage`. Disclosed to users in the UI: "💡 Chat messages are logged…"
- **Turn limit is enforced on BOTH ends.** Client disables input at 10; server rejects with 429 if a manipulated client tries to send more. Defense in depth.
- **Akash streams via SSE** through Vercel AI SDK. The `result.toUIMessageStreamResponse()` does all the protocol work.
- **`useMemo` on `transport`** is keyed on `(question.id, exam.id)`. Changing questions creates a new transport, effectively resetting the conversation.
- **The "loading bubble"** is shown until the first text token arrives — not when `status` becomes `streaming`. This avoids a visual flash where the bubble vanishes before the text appears.
