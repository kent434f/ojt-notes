# 12 — Exit guard decision

The decision tree for `hooks/use-exam-exit-guard.ts`. This is the hook that prevents a student from accidentally throwing away 90 minutes of work.

## Diagram

```mermaid
flowchart TD
  Start([Student attempts to leave the exam page]) --> Q1{What was the trigger?}

  Q1 -->|Tab close / refresh| BU[beforeunload handler<br/>Browser shows<br/>native confirm dialog]
  BU --> Done1([Browser handles<br/>user's choice])

  Q1 -->|Click on a link &lt;a&gt;| L1[Capture-phase click listener<br/>checks href]
  L1 --> L2{href starts with<br/># or javascript:?}
  L2 -->|Yes| Skip([Allow click,<br/>do nothing])
  L2 -->|No| L3{href ==<br/>current URL?}
  L3 -->|Yes| Skip
  L3 -->|No| Capture[preventDefault +<br/>stopPropagation]
  Capture --> Dialog

  Q1 -->|Sign Out button| SO[Listener catches<br/>data-nav-action='signout']
  SO --> Capture

  Q1 -->|Browser Back button| Back[popstate handler]
  Back --> RePush[history.pushState dummy<br/>to stay on page]
  RePush --> Dialog

  Dialog[Show exit-guard dialog<br/>3 buttons]
  Dialog --> Choice{User picks…}

  Choice -->|Save & Exit| SaveExit[Call handleSaveAndPause<br/>then redirect home]
  Choice -->|Continue Exam| Cancel([Dismiss dialog,<br/>stay on page])
  Choice -->|Leave Without Saving| Leave[handleConfirmLeave]

  Leave --> What{Was the trigger…}
  What -->|Browser Back| HistGo["history.go(-2)<br/>past dummy + current"]
  What -->|Sign Out| SignOut["signOut callbackUrl: '/'"]
  What -->|Link click| Push["router.push pendingHref"]

  classDef start fill:#1E2761,color:#FFFFFF
  classDef decision fill:#FEF3C7,color:#1A1A2E,stroke:#EAB308
  classDef action fill:#EEF2FE,color:#1A1A2E,stroke:#1E2761
  classDef terminal fill:#DCFCE7,color:#166534,stroke:#16A34A
  classDef danger fill:#FEE2E2,color:#991B1B,stroke:#DC2626

  class Start start
  class Q1,L2,L3,Choice,What decision
  class L1,SO,Back,RePush,Capture,Dialog,SaveExit action
  class Skip,Cancel,Done1 terminal
  class BU,Leave,HistGo,SignOut,Push danger
```

## Notes

- **Capture-phase click listener** is critical. Bubble-phase would fire too late — Next.js's `Link` would have already started client-side navigation.
- **Pushing a dummy history state** lets us turn the Back button into "show dialog" without leaving. When the user confirms leaving, `history.go(-2)` skips past both the dummy AND the take-page entry.
- **The dialog can't be dismissed by clicking outside or pressing Escape.** Both are blocked in `AlertDialogContent` via `onPointerDownOutside`, `onEscapeKeyDown`, `onInteractOutside` preventDefault.
- **The hook is disabled when the page is already navigating away** (`enabled: !submitting && !saving`). Otherwise, programmatic navigation would trigger our own dialog.
- **Hash links (`#section`) and same-page links bypass the dialog.** Anchor clicks and self-links shouldn't trigger an exit warning.
