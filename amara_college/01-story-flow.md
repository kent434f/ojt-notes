# 01 - Story Flow

Five charts: the top-level four-act spine, then each act with its branches,
choices, clue-setters, and placeholder labels.

## 1. Top-level four-act spine

The skeleton every playthrough follows. The four endings are reachable from
Act 4 based on case strength (see `02-clues-and-case-strength.md`).

```mermaid
flowchart TD
    Start([start]) --> Cold[ph_study_session<br/>PLACEHOLDER cold open]
    Cold --> A1[Act 1: The Scene]
    A1 --> A2[Act 2: The Investigation]
    A2 --> A3[Act 3: The Unraveling]
    A3 --> A4[Act 4: The Confrontation]
    A4 --> E1([ending_truth])
    A4 --> E2([ending_escape])
    A4 --> E3([ending_sacrifice])
    A4 --> E4([ending_stalled])

    classDef placeholder fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef ending fill:#1A2030,stroke:#5B9BD5,color:#C8E8F0
    class Cold placeholder
    class E1,E2,E3,E4 ending
```

## 2. Act 1 - The Scene

Cold open is a placeholder. After the scream, the player gets two
choice points; both lead to the nurse, then a placeholder atmosphere beat.

```mermaid
flowchart TD
    Cold[ph_study_session<br/>PLACEHOLDER]
    Cold --> Intro[act1_intro<br/>Maribel hears the scream]
    Intro --> Lab[act1_lab_scene<br/>Clarita on the floor]
    Lab --> C1{First move?}
    C1 -- Examine symbols --> Sym["act1_examine_symbols<br/>sets lab_symbols"]
    C1 -- Approach Hazel --> Hazel[act1_approach_hazel]
    Sym --> Hazel
    Hazel --> C2{Ask for notebook?}
    C2 -- Ask --> NB["act1_notebook_clue<br/>sets notebook"]
    C2 -- Let it go --> Nurse[act1_nurse_arrives]
    NB --> Nurse
    Nurse --> Int1[ph_warden_interlude_1<br/>PLACEHOLDER]
    Int1 --> Act2([Act 2])

    classDef placeholder fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef setsflag fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    class Cold,Int1 placeholder
    class Sym,NB setsflag
```

## 3. Act 2 - The Investigation

This is where the missable evidence lives. The "Head back" option is what
makes the weak case band reachable — without it, players would always pick
up at least one optional clue.

```mermaid
flowchart TD
    A2I[act2_intro] --> Flash[ph_maribel_flashback<br/>PLACEHOLDER]
    Flash --> Rom["act2_question_rommel<br/>sets alibi_sus"]
    Rom --> PA{Push the alibi?}
    PA -- Push now --> Push[act2_push_alibi]
    PA -- Move on --> Locs
    Push --> Locs[act2_investigate_locations]
    Locs --> Loc{Choose location}
    Loc -- "Check the lab" --> Lab["act2_check_lab<br/>maybe sets lab_symbols<br/>sets financial"]
    Loc -- "Faculty room" --> Fac["act2_check_faculty<br/>sets financial"]
    Loc -- "Talk to Hazel" --> NB["act2_check_notebook<br/>sets notebook"]
    Loc -- "Head back" --> End[act2_end]
    Lab --> Locs
    Fac --> Locs
    NB --> End
    End --> Act3([Act 3])

    classDef placeholder fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef setsflag fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    classDef escape fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    class Flash placeholder
    class Rom,Lab,Fac,NB setsflag
    class End escape
```

## 4. Act 3 - The Unraveling

Recording and confession are always picked up (spine). The puzzle is what the
investigation work pays off into — and the puzzle's behavior depends on how
many cards the player earned (see `03-clue-reconstruction-puzzle.md`).

```mermaid
flowchart TD
    A3I[act3_intro] --> Dan[ph_danilo_moment<br/>PLACEHOLDER]
    Dan --> Rec["act3_recording<br/>sets recording"]
    Rec --> Name{Ask Danilo to name them?}
    Name -- Now --> NameNow[act3_danilo_names]
    Name -- Hold --> Hold[act3_hold_off]
    NameNow --> LM[ph_lourdes_moment<br/>PLACEHOLDER]
    Hold --> LM
    LM --> Conf["act3_lourdes_confession<br/>sets confession"]
    Conf --> PI[act3_puzzle_intro<br/>build earned puzzle_cards]
    PI --> Can{5 or more cards?}
    Can -- No --> Skip[act3_puzzle_insufficient]
    Can -- Yes --> Run[call screen clue_puzzle]
    Run --> Result{result}
    Result -- full + correct --> Succ[act3_puzzle_success<br/>puzzle_solved=True]
    Result -- 5-card + correct --> Part[act3_puzzle_partial<br/>puzzle_partial=True]
    Result -- wrong --> Fail[act3_puzzle_fail]
    Succ --> Ally
    Part --> Ally
    Skip --> Ally
    Fail --> Ally[act3_ally_castillo<br/>maybe sets ally_teacher_b]
    Ally --> AE[act3_end]
    AE --> Int2[ph_warden_interlude_2<br/>PLACEHOLDER]
    Int2 --> Act4([Act 4])

    classDef placeholder fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef setsflag fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    classDef puzzle fill:#2a1a3a,stroke:#C48EC4,color:#E8E8E8
    class Dan,LM,Int2 placeholder
    class Rec,Conf setsflag
    class Succ,Part,Skip,Fail puzzle
```

## 5. Act 4 - The Confrontation

Case strength is computed at the top of `act4_intro`. The confrontation runs
all gated rebuttals, then the choice menu the player gets depends on which
band they're in. See `04-confrontation-and-endings.md` for the rebuttal
gating in detail.

```mermaid
flowchart TD
    A4I[act4_intro<br/>compute case strength]
    A4I --> Conf[act4_confrontation<br/>evidence-gated rebuttals]
    Conf --> Band{case band?}
    Band -- strong --> SM{Truth or Sacrifice?}
    Band -- partial --> PM{Sacrifice or Escape?}
    Band -- weak --> Stalled[act4_path_stalled]
    SM -- Step forward --> Truth[act4_path_truth]
    SM -- Make it personal --> Sac
    PM -- Make it personal --> Sac[act4_path_sacrifice]
    PM -- Hold your ground --> Esc[act4_path_escape]
    Sac --> SacC{Promise Dalisay protection?}
    SacC -- Accept --> SAcc[act4_sacrifice_accept]
    SacC -- Refuse --> SRef[act4_sacrifice_refuse]
    SAcc --> SScene[ending_sacrifice_scene]
    SRef --> SScene
    Truth --> E1([ending_truth])
    Esc --> E2([ending_escape])
    SScene --> E3([ending_sacrifice])
    Stalled --> E4([ending_stalled])

    classDef ending fill:#1A2030,stroke:#5B9BD5,color:#C8E8F0
    classDef weak fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    class E1,E2,E3,E4 ending
    class Stalled weak
```
