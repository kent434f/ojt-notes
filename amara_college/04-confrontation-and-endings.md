# 04 - Confrontation and Endings

Three charts: which evidence breaks which of Gideon's defenses on the rooftop,
which choice menu the player gets per case band, and the full ending dispatch.

## 1. Rooftop Rebuttals - What Each Defense Requires

The confrontation is structured as four defenses. Each one is broken only by
the matching evidence. The tape always lands (it's a spine clue); everything
else is conditional.

```mermaid
flowchart TD
    Conf[act4_confrontation start] --> D1[Defense 1<br/>'A coerced confession won't hold']
    D1 --> R1[BROKEN by recording<br/>always present, so always breaks]
    R1 --> D2[Defense 2<br/>'It's all circumstantial']
    D2 --> M{money_ok?}
    M -- Yes --> R2[BROKEN by financial trail<br/>'Shadows don't sign wire transfers']
    M -- No --> NM[Maribel privately admits gap]
    R2 --> D3
    NM --> D3
    D3[Defense 3<br/>'Dalisay will never testify']
    D3 --> T{testimony_ok?}
    T -- Yes --> R3[BROKEN by Castillo's testimony]
    T -- No --> NT[Gideon: 'Name one']
    R3 --> D4
    NT --> D4
    D4[Defense 4<br/>premeditation]
    D4 --> P{premed_ok or puzzle_partial?}
    P -- premed_ok --> R4[BROKEN by full timeline<br/>'archive → Lourdes → Dalisay → floor']
    P -- puzzle_partial --> R4P[Weaker press<br/>'preparation, not panic']
    P -- neither --> Skip[Plan press skipped entirely]
    R4 --> End
    R4P --> End
    Skip --> End
    End[Reaction to Clarita beat]
    End --> CW{case_weak?}
    CW -- Yes --> EZ[Gideon RELAXES<br/>'ugly and proven are different rooms']
    CW -- No --> Cracks[gideon cracked sprite<br/>performance is over]

    classDef broken fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    classDef missing fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    classDef partial fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    class R1,R2,R3,R4,Cracks broken
    class NM,NT,Skip,EZ missing
    class R4P partial
```

## 2. Choice menu per case band

The menu the player sees at `act4_choice` is gated by the band. Weak case
gets no menu — the "choice" was already made by under-investigating in Act 2.

```mermaid
flowchart LR
    Choice[act4_choice] --> Band{case band}
    Band -- strong --> SM[Menu:<br/>Step forward / Make it personal]
    Band -- partial --> PM[Menu:<br/>Make it personal / Hold ground]
    Band -- weak --> NoMenu[No menu<br/>jump to act4_path_stalled]

    SM -- Step forward --> Truth[act4_path_truth]
    SM -- Make it personal --> Sac[act4_path_sacrifice]
    PM -- Make it personal --> Sac
    PM -- Hold ground --> Esc[act4_path_escape]
    NoMenu --> Stalled[act4_path_stalled]

    classDef strong fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    classDef partial fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    classDef weak fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    class SM,Truth strong
    class PM,Sac,Esc partial
    class NoMenu,Stalled weak
```

## 3. Full ending dispatch with shared exit helper

All four endings funnel through `_ending_gate_exit` for the four-line outro
over the exterior shot, then jump to their title-card label.

```mermaid
flowchart TD
    Truth[act4_path_truth] --> TExit[_ending_gate_exit<br/>4-line exterior outro]
    TExit --> ETruth([ending_truth<br/>'THE TRUTH'])

    Esc[act4_path_escape] --> EExit[_ending_gate_exit]
    EExit --> EEsc([ending_escape<br/>'THE LONG WAY'])

    Sac[act4_path_sacrifice] --> SC{Promise Dalisay?}
    SC -- Accept --> SAcc[act4_sacrifice_accept]
    SC -- Refuse --> SRef[act4_sacrifice_refuse]
    SAcc --> SScene[ending_sacrifice_scene]
    SRef --> SScene
    SScene --> SExit[_ending_gate_exit]
    SExit --> ESac([ending_sacrifice<br/>'THE COST'])

    Stalled[act4_path_stalled] --> StExit[_ending_gate_exit]
    StExit --> EStalled([ending_stalled<br/>'THE UNFINISHED'])

    classDef helper fill:#2a2a2a,stroke:#9AA5B8,color:#E8E8E8
    classDef title fill:#1A2030,stroke:#5B9BD5,color:#C8E8F0
    class TExit,EExit,SExit,StExit helper
    class ETruth,EEsc,ESac,EStalled title
```
