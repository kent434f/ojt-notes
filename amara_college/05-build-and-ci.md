# Build and CI

Four charts: local pre-commit, CI lint job, web build + Vercel deploy, and a
coverage map of which validator protects which silent-failure mode.

## 1. Local pre-commit flow

Optional, opt-in via `./scripts/setup-git-hooks.sh`. CI runs the same checks
regardless, but the hook catches issues before they ever reach the remote.

```mermaid
flowchart TD
    Edit[edit .rpy / asset / config] --> Stage[git add]
    Stage --> Commit[git commit]
    Commit --> Hook{pre-commit<br/>hook installed?}
    Hook -- No --> Accept[commit accepted]
    Hook -- Yes --> Assets[verify-renpy-assets.sh]
    Assets -- pass --> Lint[renpy-lint.sh]
    Assets -- fail --> Block[BLOCK commit]
    Lint -- pass --> Accept
    Lint -- fail --> Force{FORCE_RENPY_LINT<br/>env var set?}
    Force -- Yes --> Block
    Force -- No --> Accept

    classDef block fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    classDef pass fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    class Block block
    class Accept pass
```

## 2. CI lint workflow

Runs on every push and PR to `main` via `.github/workflows/renpy-lint.yml`.
Independent jobs run in parallel; any failure fails the whole workflow.

```mermaid
flowchart LR
    Push[git push] --> CI[GitHub Actions trigger]
    CI --> V1[verify-renpy-assets.sh]
    CI --> V2[verify-puzzle-html5.sh]
    CI --> V3[verify-dynamic-puzzle-evidence.py]
    CI --> V4[verify-vercel-workflow.sh]
    CI --> V5["renpy-lint.sh --error-code"]
    V1 --> Result{All pass?}
    V2 --> Result
    V3 --> Result
    V4 --> Result
    V5 --> Result
    Result -- Yes --> Green[CI green<br/>PR mergeable]
    Result -- No --> Red[CI red<br/>see 07-how-to for diagnosis]

    classDef green fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    classDef red fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    class Green green
    class Red red
```

## 3. Web build and Vercel deploy

Runs on push to `main` via `.github/workflows/renpy-vercel-deploy.yml`. The
smoke test is the last line of defense — it actually hits the deployed URL.

```mermaid
flowchart TD
    Push[push to main] --> WF[deploy workflow]
    WF --> Build[scripts/renpy-web-build.sh]
    Build --> Wrap[wraps launcher web_build]
    Wrap --> Out{output type?}
    Out -- "-web/ folder" --> Copy[copy to web-deploy/]
    Out -- "-web.zip" --> Unzip[unzip, then copy]
    Out -- source only --> Fail[FAIL: web support not installed]
    Copy --> Vercel
    Unzip --> Vercel
    Vercel["vercel deploy --prebuilt<br/>static, no remote build settings"]
    Vercel --> Wait[wait for deploy]
    Wait --> Smoke[curl --fail public production URL]
    Smoke -- 200 --> Done[Deploy green]
    Smoke -- non-200 --> SmokeFail[FAIL: see asset routing / 404s]

    classDef fail fill:#3a1a1a,stroke:#B85042,color:#E8E8E8
    classDef done fill:#1A2030,stroke:#C8E8F0,color:#C8E8F0
    class Fail,SmokeFail fail
    class Done done
```

## 4. Validator coverage — which script protects what

Each validator exists because some silent-failure mode would otherwise ship.
This is the design principle: every silent failure mode gets a loud validator.

```mermaid
flowchart LR
    subgraph V[Validators]
        VA[verify-renpy-assets.sh]
        VP[verify-puzzle-html5.sh]
        VE[verify-dynamic-puzzle-evidence.py]
        VV[verify-vercel-workflow.sh]
        VL[renpy-lint.sh]
    end
    subgraph F[Silent failure mode it catches]
        FA[Missing image/audio file<br/>Ren'Py runs anyway and renders nothing]
        FP[Drag-and-drop reintroduced<br/>Puzzle breaks on mobile/web]
        FE[Puzzle reveals unearned evidence<br/>Spoiler bug, not a crash]
        FV[Vercel workflow loses --prebuilt or smoke test<br/>Bad deploys ship green]
        FL[Generic Ren'Py syntax/import error<br/>Caught on player's machine, not yours]
    end
    VA --> FA
    VP --> FP
    VE --> FE
    VV --> FV
    VL --> FL

    classDef v fill:#1a2a3a,stroke:#5B9BD5,color:#C8E8F0
    classDef f fill:#3a2a1a,stroke:#E8B86E,color:#E8B86E
    class VA,VP,VE,VV,VL v
    class FA,FP,FE,FV,FL f
```
