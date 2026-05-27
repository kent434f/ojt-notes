# Architecture

Three charts: file dependency graph (what reads what), the save/load state
lifecycle, and the character + sprite + position system.

## 1. File dependency graph

Solid arrows are "this file reads from / depends on that file." Most logic
flows out of `script.rpy`; everything else is supporting infrastructure.

```mermaid
flowchart LR
    subgraph Story[Story layer]
        SCRIPT[script.rpy]
        VARS[variables.rpy]
        CHARS[characters.rpy]
        PUZ[puzzle.rpy]
    end
    subgraph UI[UI layer]
        SCREENS[screens.rpy]
        GUI[gui.rpy]
        OPTS[options.rpy]
    end
    subgraph Assets[Assets on disk]
        IMG[game/images/]
        AUDIO[game/audio/]
        GUIIMG[game/gui/]
    end
    subgraph Tooling[Validators + CI]
        VAS[verify-renpy-assets.sh]
        VPH[verify-puzzle-html5.sh]
        VDP[verify-dynamic-puzzle-evidence.py]
        VVW[verify-vercel-workflow.sh]
    end

    SCRIPT --> CHARS
    SCRIPT --> VARS
    SCRIPT --> PUZ
    SCRIPT --> IMG
    SCRIPT --> AUDIO
    PUZ --> VARS
    CHARS --> IMG
    SCREENS --> GUI
    SCREENS --> GUIIMG

    VAS -.checks.-> SCRIPT
    VAS -.checks.-> CHARS
    VAS -.checks.-> IMG
    VAS -.checks.-> AUDIO
    VPH -.greps.-> PUZ
    VPH -.greps.-> SCRIPT
    VDP -.loads.-> PUZ
    VDP -.greps.-> SCRIPT
    VVW -.greps.-> Tooling

    classDef story fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    classDef ui fill:#2a1a3a,stroke:#C48EC4,color:#E8E8E8
    classDef asset fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef tool fill:#2a2a2a,stroke:#9AA5B8,color:#E8E8E8
    class SCRIPT,VARS,CHARS,PUZ story
    class SCREENS,GUI,OPTS ui
    class IMG,AUDIO,GUIIMG asset
    class VAS,VPH,VDP,VVW tool
```

## 2. Save / load / rollback lifecycle

Every variable the story reads must be declared with `default` in
`variables.rpy`. This is what makes saves, quick-loads, and rollback
all work without "variable does not exist" crashes.

```mermaid
sequenceDiagram
    participant Player
    participant Script as script.rpy
    participant State as variables.rpy state
    participant Disk as save file

    Note over State: At init: every `default` decl<br/>creates a known-good baseline
    Player->>Script: make a choice
    Script->>State: $ clue_flags.add('notebook')
    Script->>State: $ puzzle_solved = True
    Player->>Disk: F5 quick save
    Disk->>State: serialize all default-ed vars + current label
    Note over Disk: Even unused vars are stored — rollback safety
    Player->>Disk: F9 quick load
    Disk->>State: restore vars
    Disk->>Script: resume at saved label
    Player->>Player: rollback (Page Up)
    Script->>State: revert state to previous decision point
    Note over Script,State: Rollback only works because every var<br/>has a default — otherwise revert points<br/>can reference vars that didn't exist yet
```

## 3. Character + sprite + position system

Three layers of indirection let an artist rename a file without touching
`script.rpy`. Worth understanding because it's the single most reused
pattern in the codebase.

```mermaid
flowchart LR
    subgraph CharLayer[characters.rpy - character definitions]
        DEF["define inv = Character('Maribel', color='#5B9BD5', image='maribel')"]
    end
    subgraph SpriteLayer[characters.rpy - sprite aliases]
        A1["image maribel neutral = 'maribel_neutral.png'"]
        A2["image maribel serious = 'maribel_serious.png'"]
        A3["image maribel thinking = 'maribel_thinking.png'"]
    end
    subgraph PosLayer[characters.rpy - position transforms]
        P1["transform char_left<br/>xanchor 0.5, xpos 0.22, ypos 1.0"]
        P2["transform char_center"]
        P3["transform char_right"]
    end
    subgraph Files[On disk]
        F1[game/images/maribel_neutral.png]
        F2[game/images/maribel_serious.png]
        F3[game/images/maribel_thinking.png]
    end
    subgraph Use[script.rpy usage]
        U1[show maribel neutral at char_left]
    end

    A1 -.resolves to.-> F1
    A2 -.resolves to.-> F2
    A3 -.resolves to.-> F3
    A1 -.used by.-> U1
    P1 -.used by.-> U1
    DEF -.provides name+color for.-> U1

    classDef def fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    classDef alias fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef pos fill:#2a1a3a,stroke:#C48EC4,color:#E8E8E8
    classDef file fill:#2a2a2a,stroke:#9AA5B8,color:#E8E8E8
    class DEF def
    class A1,A2,A3 alias
    class P1,P2,P3 pos
    class F1,F2,F3 file
```
