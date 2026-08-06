# NOCTURNE Glossary — the proper nouns of this project

Read at boot (PLAN §0 STEP 1). Every term below is used in law in exactly
this sense. Maintained by the human gate; agents PROPOSE entries in their
handoff reports when they coin or encounter an undefined term. When law
and this file disagree, law wins — then this file gets fixed.

## The product and its parts

- **NOCTURNE** — the product entire: harness + spine + Memory Palace +
  visualization, under one name.
- **the Memory Palace / the Palace** — the memory product: the cloud
  database, its curation algorithms, and (M3) the curator agents.
- **the spine** — the always-on cloud backbone (Cloud Run + Cloud SQL)
  housing the Palace and connective modules; clients dial into it.
- **the Harness** — the local product: daemon, agent loop, and web UI on
  the user's machine.
- **the Garden** — the governance repo (SPEC, PLAN, BOARD, FLAGS,
  AMENDMENTS, notes) and the place the organism is grown.
- **the relay** — the build methodology: one agent per session, one
  packet per session, everything handed forward in files.
- **the Cube (Escher view)** — the 3D command center; one object (the
  work) with faces: FARM, ROOTS, TIPS, DECK, LEDGER, PALACE, SCORE.
- **the Deck** — the card rail of drafts/decisions; THE ONLY SURFACE
  THAT MAY DEMAND attention.

## The memory loop

- **memory unit** — one atomic fact, ≤128 tokens, with keywords,
  embedding, stamps, stats, and per-memory bias.
- **injection** — scoring + inserting the top memories into a thread's
  context at its first prompt.
- **the gate** — the per-thread approval surface where injected memories
  are shown as cards; keeps/removals/add-backs are typed training signal.
- **near-miss** — the next 3 memories below the injection line, shown in
  the gate; can be added back (strong positive signal) or vetoed
  ("never", v2.16).
- **injection_event** — the append-only log of every injection decision
  with full feature vectors; the training table for all learning.
- **pin** — always injected, exempt from scoring and the token budget;
  only humans and (M3) promotion flip it.
- **quarantine** — the bench: a memory removed from candidacy after 3
  "never" kills; reversible, never deleted.
- **tombstone** — soft deletion; nothing in the Palace is ever
  hard-deleted.
- **reinforcement** — a hard-duplicate save coalesces into a stats bump
  on the existing unit: independent re-derivation IS importance signal.
- **the approval queue** — the palace's front porch: candidates and
  proposals from machine producers wait here; invisible to injection
  until approved. Renders in two places by BIRTHPLACE: thread/run-born
  cards ride their origin's deck flashcard; corpus-born proposals live
  in the Palace queue module.
- **passive approval** — deck-ridden queue items actually rendered in
  the viewport and passed without denial are approved when the card
  resolves (enter-unless-denied, the gate's grammar on the write door);
  logged as weak signal; contradictions and collapsed groups never
  passive-approve.
- **auto_exited** — a memory autonomously demoted by per-message
  re-scoring (never by a human hand); ungraded by the scoreboard — not a
  disposition (A-031).
- **candidate** — a memory unit's status while waiting in the queue.
- **the conversation journal** — the plain file on disk where the daemon
  writes every message as it happens (the owner's "commit after every
  chat" perpetuity directive, built at M2D). Restart-proof capture;
  serving/rewind arrives with M3's session tables.
- **seed ingestion** — uploading markdown that splitters break into
  atomic candidates (the cold open; ADR-019).
- **the scorer** — the transparent linear model ranking candidates:
  weighted sum of features (f_sem, f_kw, f_time, f_proj, f_freq, f_hist;
  f_loc at M3) plus per-memory bias **b_m**; threshold **τ**, top-k 8.
- **the Chrysopoeia** — the learning loop: tunes the ALGORITHM (weights,
  features) from logged gate signals. Asks "did we show the right
  memories?"
- **the curators** — M3 maintenance agents: tend the CONTENTS (dedup,
  contradiction, promotion, slop). Ask "is this memory in good
  condition?" Never touch a weight.
- **Palace Health Report** — the deterministic diagnostic (code, not
  LLM) every curator pass must read before intervening.
- **the Injection Console** — the four-layer scoring module (ADR-009
  item 4, v2.42): READ (score decomposition, exactly summing bars),
  CONTROL (versioned knob law), WHAT-IF (preview re-rank, never
  persisted), AUDITION (challenger counterfactuals overlaid on live
  gates; incumbent governs until the one-tap).
- **earned autonomy** — curator verdict classes graduate from queued to
  auto-execute only when the owner's approvals run near-unanimous;
  contradictions never graduate.
- **staging** — branch-scoped memory during multi-agent runs: an agent
  sees the corpus plus its OWN staged saves only; the judge promotes the
  winner's staging at run close.
- **origin_agent** — the materialized-path lineage id of a save
  (run/root.3.1 = premiere → attempt 3 → its sub-agent 1).
- **movement law** — agents must move to where the work is; location
  stamps memories and refreshes injection (ADR-010, M3).

## Work and orchestration

- **mode scale** — Solo / Duet (pair chat) / Ensemble / **Symphony**
  (parallel search: attempts in worktrees, EXPAND/FORK/GRAFT/PRUNE).
- **the judge** — the evaluating agent seat: triages boundary crossings,
  gates run completion, promotes staged memories, ranks digests.
- **the work protocol** — spec alignment → agent loop → judge; the human
  interjects only when the judge says done (ADR-012).
- **walls, not questions** — Invariant 14's posture: boundaries pause
  work silently for triage; nothing interrupts the human except the Deck.
- **boundary card** — a judge-triaged request to cross a wall, surfaced
  on the Deck once, at highest leverage.
- **agent tests** — the third tier: unit tests verify code, integration
  tests verify contracts, agent tests verify EXPERIENCE — a standing
  suite of human-style browser verifications agents re-execute at
  merges (smoke) and gates (full). Proposed 2026-08-05; law at M3.
- **the Ouroboros (N1)** — the self-hosting milestone test: NOCTURNE
  completes one relay packet on its own repositories, end to end, under
  walls. The snake bites its tail; the harness builds itself.
  "Ouroboros mode" is the standing capability that follows.
- **the 10% principle** — a build agent spends at most ~10% of thread
  tokens on memory; push injection, fire-and-forget saves.

- **the capability ladder** — NOCTURNE is one product with rungs, never
  forks: local palace → your palace in your own cloud (the MIRROR
  PRINCIPLE: deploy is a recipe anyone runs against their own supplier;
  self-operated, never SaaS by accident) → replicas + sync (blue-water,
  M5). Moving between rungs = explicit export/import until M5.

## Cost domain

- **the spend ledger / spend_event** — append-only, receipt-line-normal
  cost truth: one line per price class (product_type, quantity_type,
  unit_of_measure, quantity, basis, behavior, lineage). ADR-024.
- **basis** — how sure a cost number is: measured / allocated /
  estimated. Measured never masquerades as allocated.
- **Palace Vitals** — the read-only usage gauges (lifecycle rates, spend
  by category, counters); presented as the Ableton-style bottom strip.
- **the accuracy score** — the memory system's report card: each scorer
  generation's held-out agreement percentage (binary scoreboard as a %),
  tracked as a Vitals time series across the harness's life. Updates
  that wouldn't raise it are never proposed (v2.41).
- **spend walls** — prospective soft budgets (per run/day) enforced with
  the walls-not-questions posture; judge triages at the wall.
- **the D2 breaker** — the armed billing circuit breaker that detaches
  project billing at the account budget; the independent backstop.
- **token-cost policy (A-020/021)** — how a thread's model is chosen:
  pinned / max / elbow / slope:<λ> / floor over the broker's benchmark
  table; KV-cache-sticky (session_id = thread_id); never classifier
  routing.
- **the frontier / the lower hull** — of the broker's benchmark table:
  the non-dominated price-intelligence set, and the taut string
  stretched beneath it. Hull slopes only ever increase, so they are the
  true marginal prices of intelligence (A-021).
- **the elbow** — the frontier model that undercuts the frontier's
  average exchange rate of log-price for intelligence by the most: the
  point of diminishing returns, zero configuration. Reproduced the
  owner's independent "52 today" on first calibration; the recommended
  chat policy at enable time.
- **the commodity tier / scarcity premium** — the capability level
  competition has already priced to the bone, versus what lone
  top-of-frontier models charge for the last few index points. The
  owner's standing intent: buy the first, never pay the second.
- **the pocket** — the dense cluster of near-frontier models crowding
  just above the elbow; the market's echo of the commodity tier, never
  an independent selection signal; earmarked as M3 Symphony diversity
  shards.
- **resolution point** — the only moments a thread's model may change:
  thread start, a human `/model` command, or (M3, reserved) an algorithmic
  role boundary — each journaled (old→new, reason, sacrificed prefix) with
  a new stickiness epoch. Drift is never a resolution point (v2.26).
- **cache-sticky routing** — epoch zero uses session_id = thread_id; every
  later resolution point uses a distinct epoch-scoped session_id, re-pinning
  provider and model while preventing drift inside that epoch. A growing
  prefix can then re-bill at cache-read prices. The cheapest token is the one
  already cached (A-020; v2.26 resolution-point refinement).
- **the blast-radius principle** — price a role by the blast radius and
  reviewability of its errors, never by task difficulty: economic
  policies where volume is high and errors are judge-filtered (leaf
  search); pinned or max where errors compound (orchestrators, judges,
  curators).

## Visualization and plugins

- **the rack** — the dockable grid of interface modules; layouts are
  savable sets; the shipped UI is a factory preset ("version one of the
  default opinion").
- **plugin classes** — VISUALIZERS (read), CONTROLS (write via the
  registry), FACES (stage schemas), THEMES (style only).
- **the parameter registry** — typed descriptors of every controllable
  parameter; controls bind, never free-hand; every knob turn journaled.
- **the three surfaces** — the whole plugin read API: the event stream,
  the query surface (+as_of), the selection bus. No notify API exists.
- **as_of / the rollback bar** — time-scrubbing: every surface re-renders
  its state at any past moment (ADR-016).
- **procedural law** — all stage geometry is grown deterministically from
  work metadata; same data → same geometry; no hand-authored assets.
- **NEO-NOIR / COBALT-SERAPH** — the default and alternate themes.
- **Ant Farm / Context Bars / Memory Graph** — the named visualizations:
  fleet-by-directory, per-agent context breakdown, the memory force graph.
- **nocturne-plugin-contributor** — the shipped agent skill that teaches
  any agent to author and contribute plugins from zero repo knowledge.

## Governance

- **the human gate** — the owner (Nate) plus his pen: audits reports,
  enacts amendments, clears holds; the only authority over SPEC.md.
- **packet** — one bounded work charge on the BOARD; one per session.
- **the board (BOARD.md)** — the state ledger of packets, deps, claims.
- **claim commit** — the git commit that locks a packet to a session.
- **FLAGS / stop-the-line** — the escalation channel when law blocks work.
- **AMENDMENTS / COMPLETION** — decide-and-declare authority: agents fill
  qualifying contract silence themselves and log it as law.
- **DECISIONS.md** — each repo's journal of non-dictated choices, citing
  Problem Tree nodes.
- **the Blight Protocol** — defect handling: fix at the deepest problem
  node that contains it, or flag.
- **the Problem Tree (P-nodes)** — the motivation skeleton: every packet
  and decision anchors to a problem node.
- **scout (B.6 rule 9)** — the agent that pre-runs a human gate's
  checklist as a live browser SOP, burning down mechanics so the owner
  spends attention only on taste.
- **SOP walkthrough (B.6 rule 8)** — human-style operating procedures
  executed live by agents in a real browser, with screenshots and
  first-person prose.
- **HUMAN USE HOLD** — an owner gate: the relay may build, but the judge
  waits until the owner declares the hold cleared.
- **normativity classes** — CONTRACT (binding) / GUIDANCE (deviate with a
  journaled reason) / HORIZON (accepted direction, building forbidden).
- **the two-secrets rule** — onboarding needs exactly a GCP credential
  and an OpenRouter key; anything demanding a third secret is wrong.
- **exclusion set** — packets that must never be IN_PROGRESS together
  (file-surface conflicts); enables safe concurrent relay sessions.
- **ULID discipline** — every id is a ULID: time-sortable, the id IS the
  timestamp (run_id, event_uid, rev_uid, spend ids).
- **document the edges, don't police the interior** — the project rule:
  local freedom everywhere; documentation marks what's recommended to
  modify vs law-bound, with the why.
