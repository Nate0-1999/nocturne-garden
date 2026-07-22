# Scorer evolution — owner's gate-day design inputs (2026-07-21)

Status: NOTES for M2 planning (the Chrysopoeia). Not law yet. These are
scorer-v1 FEATURE CANDIDATES + philosophy, to be adjudicated by offline
replay against the injection_event log per ADR-005's evaluation doctrine.

## Candidate: f_gen — the generality axis

Owner: a memory like "Motivation as root context" applies across all
projects — a GENERALITY score, deliberately in opposition to the
location/directory axis. Analysis: partially, not fully, covered today —
a global memory (project_key NULL) scores f_proj=0.5 everywhere (middling
by absence of stamps), but a truly universal principle can never reach
full strength outside a project match. Two variants to evaluate:
- DECLARED generality: agent/human estimates at save time (a unit field
  or kind); cheap, subjective, gameable.
- EMPIRICAL generality (elegant): derived from the log — a memory that
  is kept/cited across many DISTINCT project_keys/threads IS general.
  Measurable today from injection_event outcomes; no schema change; the
  curator can maintain it as a stat. Generality becomes something a
  memory EARNS.
Recommendation: prototype both in replay; empirical is philosophically
aligned (log-first, learned-not-asserted).

## Candidate: f_phase — conversation-phase relevance

Owner: memories differ in WHEN they matter within a conversation —
motivation-type memories matter throughout; "narrow commits,
problem→solution titles" matters only during implement/review. Proxy
ideas: human interactions since thread start / since last compaction; or
categorical work phases (spec / implement / review).
Analysis: moot in M1 (one injection per thread) — becomes meaningful
exactly when re-scoring exists: OQ-15 (per-prompt re-scoring, M2
decision) and the ADR-010 movement-refresh (M3). KEY INSIGHT: in conduct
mode the ADR-012 work protocol KNOWS its phase (spec alignment → loop →
judge) as explicit state — no classifier needed; Duet mode would need a
cheap heuristic/classifier. injection_event already logs prompt_text +
full context, so phase features are retro-evaluable by replay before any
live rollout. Not overfit — schedule with OQ-15.

## Philosophy confirmations (owner Q3)

- TUNING ALGORITHM: M1 deliberately tunes NOTHING (base functionality +
  signal collection first — confirmed as the reason). The committed path
  (decision 010): M2 = SGD on the GLOBAL linear weights from gate signals
  (removals/add-backs per the ADR-005 table), b_m absorbing per-memory
  correction; M3 = hierarchical per-project offsets, partial pooling.
  Rationale: sparse single-user signal (dozens–hundreds of events, not
  millions) starves big models; interpretability is law (the gate shows
  per-feature contributions — a linear model's explanation IS its
  weights); scorer_version + replay make every step cheap to evaluate and
  roll back. JEPA-class learned joint-embedding matching is the right
  instinct at the wrong LAYER/stage: it would augment RETRIEVAL (learned
  memory↔context affinity) rather than the scoring weights, and needs
  data volumes that arrive only with shared palaces / multi-user. Revisit
  at that scale, gated by replay.
- PLUGGABLE PARAMETERS: already law — the f_loc null rule established the
  pattern (omit a feature and renormalize remaining weights), and
  scorer_config rows version weights+params, so features can be added or
  removed per scorer version without migrations.
- PERMANENT BACKTEST TABLE: already exists BY DESIGN — injection_event is
  the append-only training log (every outcome: kept, removed:reason,
  added_back, cited, mid_thread_removed — with full six-feature vector,
  prompt_text, scorer_version, context tuple); memory_revision captures
  every human add/edit (/remember, panel) permanently. C.8 criterion 6
  makes replay-reconstruction an acceptance test. The owner's first
  add-back (2026-07-21) is training row one.

## Queued deep-dive

Tool/skill parity (browser use via Codex extension port, Agent Skills,
MCP, CodeMode, etc. — see gate-day-feedback.md): owner wants a dedicated
detailed session; plan a focused field survey + adoption charges when it
opens.
