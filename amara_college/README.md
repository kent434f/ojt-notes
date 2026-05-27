# Mermaid Reference Pack — Collegio de Amara: The Pale Warden

A visual companion to `intern-explainer.md`. Every file in this folder is a set
of Mermaid diagrams grouped by topic. Use them as a lookup, not a read-through.

> **How to view:** any modern Markdown viewer renders Mermaid (GitHub, VS Code
> with the Markdown Preview Mermaid Support extension, Obsidian, Typora). If
> your viewer doesn't, paste the code block into <https://mermaid.live> to
> render it standalone.

## Files

| File | What's in it |
|---|---|
| [`01-story-flow.md`](01-story-flow.md) | The four-act spine and a per-act flow with all branches, choices, and placeholder labels. |
| [`02-clues-and-case-strength.md`](02-clues-and-case-strength.md) | Where each clue flag is set, how `case_strong / partial / weak` is computed, the `financial → Castillo` causal rewire. |
| [`03-puzzle.md`](03-puzzle.md) | The clue-reconstruction screen as a state machine; how the card list is filtered by earned clues; what the screen returns and who owns the result. |
| [`04-confrontation-and-endings.md`](04-confrontation-and-endings.md) | Which evidence breaks which of Gideon's defenses on the rooftop; which menu the player gets per case band; the four endings. |
| [`05-build-and-ci.md`](05-build-and-ci.md) | Pre-commit hook, CI lint job, web build, Vercel deploy with smoke test, and a coverage map of which validator protects what. |
| [`06-architecture.md`](06-architecture.md) | File dependency graph, save/load state lifecycle, the character + sprite + position system. |
| [`07-how-to.md`](07-how-to.md) | Decision-tree style flows for the most common contributor tasks: adding a clue, adding an ending, adding a puzzle card, diagnosing a CI failure, finding which file to edit. |

## A note on chart syntax

Charts here lean on a small set of Mermaid features for portability:

- `flowchart TD` (top-down) and `flowchart LR` (left-right)
- `stateDiagram-v2` for screen state
- `sequenceDiagram` for save/load and similar over-time flows
- Subgraphs for grouping
- `classDef` styling for highlighting (problems, fixes, hard limits)

If a diagram looks broken in your viewer, paste it into <https://mermaid.live>
to confirm whether it's the chart or the viewer.
