# 02 - Clues and Case Strength

Four charts: which scene sets which flag, how case strength is computed, the
`financial → testimony` causal rewire, and the full input-to-outcome matrix.

## 1. Clue Flag Map - Where Each Flag Is Set

Three "spine" clues are always set on a normal playthrough. Three "missable"
clues depend on which choices the player made.

```mermaid
flowchart LR
    subgraph Spine["SPINE (always set)"]
        SA[alibi_sus]
        SR[recording]
        SC[confession]
    end
    subgraph Missable["MISSABLE (investigation)"]
        MN[notebook]
        ML[lab_symbols]
        MF[financial]
    end

    A1ES[act1_examine_symbols] --> ML
    A1NB[act1_notebook_clue] --> MN
    A2QR[act2_question_rommel] --> SA
    A2CL[act2_check_lab] --> ML
    A2CL --> MF
    A2CF[act2_check_faculty] --> MF
    A2CN[act2_check_notebook] --> MN
    A3RC[act3_recording] --> SR
    A3LC[act3_lourdes_confession] --> SC

    classDef spine fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    classDef miss fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    class SA,SR,SC spine
    class MN,ML,MF miss
```

## 2. Case strength computation

Four booleans are derived at the top of `act4_intro`, then collapsed into one
of three bands. Note that `puzzle_partial` blocks `case_weak` but does not
make a case strong.

```mermaid
flowchart TD
    F1["'financial' in clue_flags"] --> MOK[money_ok]
    F2["'lab_symbols' in clue_flags"] --> SOK[staging_ok]
    F3[puzzle_solved] --> POK[premed_ok]
    F4[ally_teacher_b] --> TOK[testimony_ok]
    F5[puzzle_partial] -.blocks weak only.-> Band

    MOK --> Band{Compute band}
    SOK --> Band
    POK --> Band
    TOK --> Band

    Band -- "money + premed + (staging or testimony)" --> Strong[case_strong]
    Band -- "no money, no premed, no partial, no staging" --> Weak[case_weak]
    Band -- "anything else" --> Partial[case_partial]

    classDef strong fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    classDef partial fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef weak fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    class Strong strong
    class Partial partial
    class Weak weak
```

## 3. Causal Rewire - `financial` to `Castillo` to `testimony`

`act3_ally_castillo` used to set `ally_teacher_b = True` unconditionally.
After the rewire, it only flips true when the player can show the money. This
is what makes a missed Act 2 location have real consequences three scenes
later.

```mermaid
flowchart LR
    F{financial in clue_flags?}
    F -- Yes --> Show[Show budget sheet to Castillo]
    Show --> Agree[castillo resolved sprite<br/>'Yes, all of it']
    Agree --> T1["ally_teacher_b = True"]
    F -- No --> NoShow["Only instinct, no paper"]
    NoShow --> Decline["castillo uneasy sprite<br/>'come back with more than a hunch'"]
    Decline --> T0["ally_teacher_b stays False"]

    T1 --> TOK[testimony_ok = True]
    T0 --> NTOK[testimony_ok = False]
    TOK --> Breaks[Breaks Gideon's<br/>'no one will testify' defense]
    NTOK --> NotBreak[Gideon's defense stands]

    classDef good fill:#1a2a3a,stroke:#C8E8F0,color:#C8E8F0
    classDef bad fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    class T1,TOK,Breaks good
    class T0,NTOK,NotBreak bad
```

## 4. Input Matrix - What Unlocks What

A compact view of how individual missable clues snowball through the system.

```mermaid
flowchart LR
    NB[notebook] --> Card1[Unlocks 'archive' puzzle card]
    Card1 --> Five[Gets to 5+ earned cards]
    LS[lab_symbols] --> Card2[Unlocks 'staging' puzzle card]
    Card2 --> Five
    LS --> SOK[staging_ok]
    Five --> CanPuzzle[Puzzle is playable]
    CanPuzzle --> Partial{correct order?}
    Partial -- yes, 5 cards --> PP[puzzle_partial]
    Partial -- yes, 6 cards --> PS[puzzle_solved]

    FIN[financial] --> MOK[money_ok]
    FIN --> Cast[Castillo agrees to testify]
    Cast --> TOK[testimony_ok]

    MOK --> Strong
    PS --> Strong
    SOK --> Strong
    TOK --> Strong
    Strong[case_strong reachable]

    classDef missable fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef state fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    classDef strong fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    class NB,LS,FIN missable
    class MOK,SOK,TOK,PS,PP state
    class Strong strong
```
