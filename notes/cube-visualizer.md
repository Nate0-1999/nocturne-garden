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

## v2 direction (human walkthrough, 2026-07-20)

Videogame-grade rendering; the viz IS the app (ComfyUI pattern): a large
CENTER STAGE (the cube's current face) with satellite panels, navigation by
clicking into the visualization. Aesthetic: sci-fi HUD — X-wing pilot
display / Nostromo terminal: near-black phosphor ground, wireframe
structures, mono readouts, angular clipped panels, restrained glow.

Layout law (from the three scenarios):
- LEFT RAIL = the deck, always present, ordered by completion/wait time;
  selecting an agent on stage tops its card; cards carry a DRAFT response
  (conductor-prepared, clearly machine-labeled) the human tweaks and
  fires; after firing, the rail auto-advances to the longest-waiting.
- CENTER STAGE = the cube face, large: FARM (projects + agents as ants in
  wireframe directory trees) ⇄ rotate ⇄ ROOTS (selected agent's subagents
  fanned in full color; other subtrees low-opacity ghosts).
- RIGHT RAIL (on selection) = inspector: context-window visualizer,
  injected memories with one-tap pop-off (mid_thread_removed), and
  SUGGESTIONS to add — near-misses surfaced live (both already in the
  event log; zero new data needed).
- BOTTOM-RIGHT overlay = Palace minimap: curator agents drifting through
  the memory field (M3), LIVE·CLOUD connection status.

Answers to OQ-C1/C2 implied: stage+rails replaces "default face" (deck is
a rail, not a face); cube is GLOBAL (fleet-wide; scenario 3 hops projects).
OQ-C3 (EV: opacity-floor vs node ramp) still open.
New concept to codify with ADRs: DRAFT RESPONSES on judge-released cards —
attention spent editing, not composing. Guard: drafts are labeled machine
text; a card is never auto-fired.
