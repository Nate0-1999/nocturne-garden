# The Cube — unified work visualizer (design notes, pending approval)

Status: CONCEPT — mock built for human approval 2026-07-20; not yet law.
Owner: human + assistant. Relay agents: read-only context, build nothing
from this file until it lands in SPEC (future ADR-009/016 amendment).

## Concept

One underlying object — the work (session tree + agents + memory + budget +
time) — projected onto SIX faces of a rotatable cube. Rotation = changing
projection, never changing data. Selection is SHARED: picking a thread,
branch, or agent on any face highlights it on every face and can open it
(navigation), honoring Invariant 14 — the Cube is a map you go to; it never
demands. Cards arrive only in the Deck.

## The six faces

1. DECK   — the flashcard deck (attention surface; judge-released cards).
2. FARM   — the Ant Farm: agents in SPACE — colored ants at their current
            directories in the workspace tree (movement law makes it live).
3. ROOTS  — the search tree (MCTS view): branches from the root problem;
            EXPAND/FORK/GRAFT/PRUNE visible; worktrees = branches.
4. LEDGER — the capital alluvial: budget flowing through stratagems (time).
5. PALACE — memory: injections everywhere; selecting anything traces its
            injection_events (memories, features, scores, outcomes).
6. SCORE  — time/context: per-agent context bars, run/gate/judge timeline.

## Encodings (ROOTS face)

- Edge THICKNESS = total tokens/cost expended pursuing the branch.
- Branch OPACITY = current expected value — WITH A FLOOR (~0.25): pruned
  and low-EV branches must stay visible for audit/lessons. Alternative to
  evaluate in mock review: EV as node-fill on a sequential ramp instead of
  opacity (opacity fights dark/light themes and overlap legibility).
- Pruned = dashed + stub node; Graft = two edges merging (brass).
- Fleet color = stratagem/agent identity, consistent across all faces.

## Memory-injection traceability (hard requirement)

The PALACE face and all trace views are pure CONSUMERS of the
injection_event log — the same append-only table the learning loop reads.
Zero contact with the scoring path; zero schema change needed (thread_id,
memory_id, features, score, outcome already logged per event). Rendering
can never perturb injection optimization.

## Devil's advocate (raised at concept stage)

- Literal 3D cube = demo-candy risk. Compromise: cube as navigation
  metaphor (face transitions, corner cube widget) but each face renders
  FLAT when focused. 2.5D, not WebGL.
- Opacity-as-EV hides exactly the branches audits need; floor + ramp
  alternative noted above.
- Six equal faces would violate least-attention; resolved: only DECK
  demands, the rest are watchable.
- Scope: nothing here touches M1; H4 ships the minimal thread list + chat
  as charged. Cube is M2/M3 viz law after approval.

## Open questions for the human

- OQ-C1: default face on open — Deck (attention-first) or Roots (map-first)?
- OQ-C2: is the cube per-project (one cube per score) or global (fleet-wide
  with project filtering)?
- OQ-C3: EV encoding — opacity-with-floor vs node-color ramp?
