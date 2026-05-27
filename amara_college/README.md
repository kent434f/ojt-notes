# Amara College

## Overview

Mermaid reference pack for *Collegio de Amara: The Pale Warden*. These diagrams document story flow, clue logic, puzzle behavior, endings, build checks, and implementation architecture.

## How to use

- **Story review:** start with the story flow, clue logic, puzzle, and endings files.
- **Implementation review:** use the architecture and build/CI files to trace code ownership and validation paths.
- **Contributor work:** use the how-to decision trees before adding clues, endings, puzzle cards, or CI fixes.

## Files

| # | File | Type | What it covers |
|---|---|---|---|
| 01 | [`01-story-flow.md`](./01-story-flow.md) | Flowchart | Four-act narrative structure, branches, choices, and placeholder labels |
| 02 | [`02-clues-and-case-strength.md`](./02-clues-and-case-strength.md) | State/decision flow | Clue flags, case strength computation, and evidence dependencies |
| 03 | [`03-clue-reconstruction-puzzle.md`](./03-clue-reconstruction-puzzle.md) | State machine | Clue reconstruction screen, card filtering, and story contract |
| 04 | [`04-confrontation-and-endings.md`](./04-confrontation-and-endings.md) | Decision flow | Rooftop rebuttals, case-band menus, and ending dispatch |
| 05 | [`05-build-and-ci.md`](./05-build-and-ci.md) | Process flow | Pre-commit checks, CI linting, web build, deployment, and validation coverage |
| 06 | [`06-architecture.md`](./06-architecture.md) | Architecture | File dependencies, save/load lifecycle, and character rendering system |
| 07 | [`07-how-to-decision-trees.md`](./07-how-to-decision-trees.md) | How-to flow | Contributor decision trees for common maintenance tasks |

## Mermaid notes

Most files use portable Mermaid syntax:

- `flowchart TD` and `flowchart LR`
- `stateDiagram-v2`
- `sequenceDiagram`
- `subgraph`
- `classDef`

If a diagram does not render in a Markdown viewer, paste the diagram block into <https://mermaid.live> to check whether the issue is the chart or the viewer.
