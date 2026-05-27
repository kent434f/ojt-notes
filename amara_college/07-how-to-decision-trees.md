# 07 - How-To Decision Trees

Five charts for the most common contributor tasks. Each one starts with the
question you're trying to answer and ends with concrete files to touch.

## 1. Adding a new clue

Covers the "I want the player to find a new piece of evidence" case. Decide
upfront whether it's just a story flag or also a puzzle card — the answers
fan out from there.

```mermaid
flowchart TD
    Start[I want to add a new clue] --> Q1{Where will the player<br/>encounter it?}
    Q1 --> Scene[Pick a scene in script.rpy]
    Scene --> Q2{Just a story flag,<br/>or also a puzzle card?}

    Q2 -- Just a flag --> F1[Pick a snake_case key name]
    Q2 -- Both --> F1

    F1 --> F2["In script.rpy:<br/>$ clue_flags.add('your_key')"]
    F2 --> Q3{Affects case strength?}

    Q3 -- Yes --> CS["In act4_intro python block:<br/>add new_ok = 'your_key' in clue_flags"]
    CS --> CSR["In act4_confrontation:<br/>add gated rebuttal using new_ok"]
    CSR --> P

    Q3 -- No --> P
    P{Puzzle card too?}

    P -- Yes --> P1["In puzzle.rpy _pz_card_defs:<br/>add (key, text, (requirement,))"]
    P1 --> P2["In verify-dynamic-puzzle-evidence.py:<br/>add an assert covering the new combo"]
    P2 --> Doc

    P -- No --> Doc[Update docs/story-flow.md<br/>with the new flag and where it's set]
    Doc --> Test[Run scripts/verify-* and renpy-lint]
    Test --> Play[Play through both with and without the flag]

    classDef start fill:#1A2030,stroke:#5B9BD5,color:#C8E8F0
    classDef done fill:#1a2a3a,stroke:#C8E8F0,color:#C8E8F0
    class Start start
    class Play done
```

## 2. Adding a new ending

Bigger change than a clue — touches Act 4 dispatch, gating, and docs.

```mermaid
flowchart TD
    Start[I want to add a new ending] --> Q1{Which case band<br/>reaches it?}

    Q1 -- Strong --> S1[Edit act4_choice strong menu:<br/>add menu option]
    Q1 -- Partial --> P1[Edit act4_choice partial menu:<br/>add menu option]
    Q1 -- Weak --> W1[Replace or branch from act4_path_stalled]

    S1 --> Path
    P1 --> Path
    W1 --> Path
    Path[Create act4_path_yourname label]
    Path --> Scene[Write the rooftop scene]
    Scene --> Exit[Call _ending_gate_exit<br/>for the 4-line exterior outro]
    Exit --> Card[Add ending_yourname title-card label]
    Card --> Doc[Update docs/story-flow.md<br/>ending table + flow diagram]
    Doc --> Test[Validators + lint + play all bands<br/>to confirm reachability + non-regression]

    classDef start fill:#1A2030,stroke:#5B9BD5,color:#C8E8F0
    classDef done fill:#1a2a3a,stroke:#C8E8F0,color:#C8E8F0
    class Start start
    class Test done
```

## 3. Adding a new puzzle card

Most contained of the three. Mostly a `puzzle.rpy` change plus one verifier
assert. The story doesn't need to know.

```mermaid
flowchart TD
    Start[Add a card to the reconstruction] --> Slot[Decide its position<br/>in canonical order]
    Slot --> Req[Decide what clue_flag tuple<br/>it requires to appear]
    Req --> Def["In _pz_card_defs:<br/>(key, text, (requirement,))"]
    Def --> Test1[Add an assert in<br/>verify-dynamic-puzzle-evidence.py]
    Test1 --> Run[Run that verifier]
    Run --> Pass{Passes?}
    Pass -- No --> Fix[Fix requirement or card order]
    Fix --> Run
    Pass -- Yes --> Doc[Update docs/story-flow.md<br/>canonical card list]
    Doc --> Play[Play with and without the unlock clue<br/>to confirm card appears/disappears]

    classDef start fill:#1A2030,stroke:#5B9BD5,color:#C8E8F0
    classDef done fill:#1a2a3a,stroke:#C8E8F0,color:#C8E8F0
    class Start start
    class Play done
```

## 4. Diagnosing a CI failure

When CI goes red, this is the fastest path from "which script failed?" to
"what's the actual problem?"

```mermaid
flowchart TD
    Red[CI is red] --> Which{Which job?}

    Which -- Lint --> L1{Which script in output?}
    L1 -- verify-renpy-assets.sh --> A1[Missing image/audio file<br/>OR filename has uppercase/spaces<br/>OR missing web build artifact<br/>check: filename in script.rpy vs disk]
    L1 -- verify-puzzle-html5.sh --> A2[Someone added drag: or draggable<br/>OR mutated puzzle_solved from screen<br/>fix: revert to tap-only + return value]
    L1 -- verify-dynamic-puzzle-evidence.py --> A3[Card filtering changed unexpectedly<br/>check: _pz_card_defs requirement tuples]
    L1 -- verify-vercel-workflow.sh --> A4[Workflow YAML lost --prebuilt<br/>OR lost the curl --fail smoke test<br/>fix: restore in .github/workflows/]
    L1 -- "renpy-lint.sh" --> A5[Generic Ren'Py error<br/>read the lint output carefully<br/>usually a label/jump typo or bad image ref]

    Which -- Deploy --> D1{Stage?}
    D1 -- Build --> B1[renpy-web-build.sh failed<br/>likely: Ren'Py web support not installed<br/>OR launcher couldn't find sources]
    D1 -- Vercel push --> B2[Check VERCEL_TOKEN secret is set<br/>OR network/auth issue]
    D1 -- Smoke test --> B3[Public URL returned non-200<br/>check Vercel deploy logs for 404s<br/>OR a renamed asset path]

    classDef red fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    classDef fix fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    class Red red
    class A1,A2,A3,A4,A5,B1,B2,B3 fix
```

## 5. Which file do I edit?

Quick router when you know what you want to change but aren't sure where the
levers are.

```mermaid
flowchart TD
    Q[I want to change...] --> T{What kind of change?}

    T -- Story text or scene flow --> SC["game/script.rpy"]
    T -- A new persistent variable --> V["game/variables.rpy<br/>(always default-ed)"]
    T -- Character sprite/expression/color --> C["game/characters.rpy"]
    T -- Sprite file itself --> CF["game/images/<br/>1386x776 PNG transparent"]
    T -- Puzzle mechanics --> P["game/puzzle.rpy<br/>(remember the html5 validator)"]
    T -- Background or CG --> BG["game/images/<br/>1280x720 PNG or JPG"]
    T -- Audio file --> AU["game/audio/<br/>MP3 for web compatibility"]
    T -- Theme colors / fonts / spacing --> G["game/gui.rpy"]
    T -- Screen layout or component --> S["game/screens.rpy"]
    T -- Game config / resolution --> O["game/options.rpy"]
    T -- Contributor docs --> D["docs/*.md<br/>(and this folder)"]
    T -- CI behavior / new validator --> SCR["scripts/ and .github/workflows/"]

    classDef target fill:#1a2a3a,stroke:#C8E8F0,color:#C8E8F0
    class SC,V,C,CF,P,BG,AU,G,S,O,D,SCR target
```
