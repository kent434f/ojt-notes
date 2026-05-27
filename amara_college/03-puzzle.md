# Puzzle (Clue Reconstruction)

Three charts: the screen as a state machine, how the card list is filtered by
the earned flags, and the contract between the puzzle screen and the story.

## 1. Puzzle screen state machine

All UI state is screen-local (held in `default`-ed screen variables). The
screen never mutates story state — it returns `True` or `False`, and
`script.rpy` interprets that.

```mermaid
stateDiagram-v2
    [*] --> Init: enter screen
    Init --> Empty: shuffle + create empty slots
    Empty --> ClueSelected: tap a clue
    ClueSelected --> SlotFilled: tap an empty slot
    ClueSelected --> Empty: tap same clue (deselect)
    SlotFilled --> Empty: tap X on a slot (remove)
    SlotFilled --> ClueSelected: tap another clue
    SlotFilled --> AllFilled: last slot filled
    AllFilled --> Checked: CHECK ANSWER
    Checked --> Correct: order matches canonical
    Checked --> Incorrect: order does not match
    Incorrect --> SlotFilled: rearrange
    Incorrect --> AllFilled: 3 attempts reached
    Empty --> Empty: RESET
    SlotFilled --> Empty: RESET
    Correct --> [*]: CONTINUE returns True
    Incorrect --> [*]: CONTINUE returns False
    AllFilled --> [*]: CONTINUE returns False (after 3 wrong)
```

## 2. Card filtering by earned flags

The canonical card list has six entries. `_pz_cards_for_flags(clue_flags)`
filters them down to only what the player has earned — and the resulting
count decides what kind of puzzle (or no puzzle) runs.

```mermaid
flowchart TD
    Canon["_pz_card_defs (canonical 6)"] --> Filter["_pz_cards_for_flags<br/>(canon, clue_flags)"]
    Flags[clue_flags] --> Filter
    Filter --> Cards[puzzle_cards earned]
    Cards --> N{"len(puzzle_cards)?"}

    N -- 4 cards<br/>spine only --> Skip["Skip the puzzle<br/>act3_puzzle_insufficient"]
    N -- 5 cards<br/>spine + one optional --> Partial[Run puzzle]
    N -- 6 cards<br/>spine + both optional --> Full[Run puzzle]

    Partial --> P5{Correct order?}
    Full --> P6{Correct order?}

    P5 -- yes --> SP[puzzle_partial = True<br/>puzzle_solved = False]
    P5 -- no --> FP[both stay False]
    P6 -- yes --> SS[puzzle_solved = True<br/>puzzle_partial = False]
    P6 -- no --> FP

    Skip --> NF[both stay False]

    classDef skip fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    classDef partial fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef full fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    class Skip,NF skip
    class SP partial
    class SS full
```

## 3. Screen ↔ story contract

The puzzle screen is a UI component, not a story owner. This is what keeps
the puzzle replayable, rollback-safe, and inspectable from `script.rpy`.

```mermaid
sequenceDiagram
    participant Story as script.rpy
    participant Screen as clue_puzzle screen
    participant State as variables.rpy

    Story->>State: $ puzzle_cards = _pz_cards_for_flags(clue_flags)
    Story->>Screen: call screen clue_puzzle(puzzle_cards)
    Note over Screen: All state is screen-local default-ed vars:<br/>slots, order, selected, attempts, feedback, solved
    Screen->>Screen: player taps, rearranges, checks
    Screen-->>Story: Return(True or False)
    Story->>State: $ puzzle_correct = _return
    Story->>State: $ puzzle_solved = puzzle_correct and _pz_full_reconstruction(puzzle_cards)
    Story->>State: $ puzzle_partial = puzzle_correct and _pz_partial_reconstruction(puzzle_cards)
    Note over State: Only script.rpy writes to story state.<br/>The screen never touches puzzle_solved directly.
```
