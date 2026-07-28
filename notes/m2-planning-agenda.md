# M2 planning agenda — areas light on specificity (2026-07-27)

Status: NOTES. None of these block M1 (H-track, I1, J). They are the
named agenda for the M2 planning session that opens after J passes,
alongside the queued tool/skill-parity deep-dive (gate-day-feedback.md).
Ranked by load-bearing-ness vs current thinness.

1. CHRYSOPOEIA UPDATE MATH. ADR-005's signals table says "SGD step,
   small −, large −" — no learning rates, no magnitudes, no weight
   normalization discipline, no cadence (per-event vs batch), and no
   REPLAY ACCEPTANCE CRITERION (what metric makes a candidate scorer
   version "better" — add-back rate? removal rate? ranking metric on
   replayed gates?). C5's provenance-class weighting is "may weight" —
   unquantified. This is the product's differentiator and its thinnest
   spec. M2 planning must pin: step sizes, the replay win-metric,
   drift guardrails per era, class weights.
2. TOKEN-SPEND ATTRIBUTION. Palace Vitals promises spend-by-category/hr
   (building vs memory tools vs curation vs judge vs extraction) but no
   law says how a token gets its category label. run.usage is per-run;
   attribution rules (tag at tool-call boundary? by principal? by
   phase?) must be spec'd before the Vitals packet.
3. UNIFIED QUEUE LAW. Four producers now converge on "the approval
   queue" (extraction verdicts-at-birth, curator proposals, Symphony
   digests, seed batches) — spec'd in four places, no single section
   owning: ordering, batching, UI home, and the Invariant-14 posture
   (queue depth visible in Vitals; deliberately NO nudges? — decide
   once, apply to all producers). M2 builds the first queue; pin then.
4. DURABLE SESSIONS + PERPETUAL LOGS. The session_message/thread-lineage
   schema is a notes sketch (scorer-evolution.md), the perpetual-logs
   directive is notes-only, and the milestone split (what M2 builds vs
   M3) is unassigned. ADR-016 is direction; the table law needs a home.
5. JUDGE-AS-MEMORY-GATE EVIDENCE. ADR-021 R3 gives the judge promotion
   authority at run close but not the evidence standard (what it weighs,
   how negative-result solicitation is phrased, digest ranking criteria).
   Pairs with the ADR-012 judge-criteria work. M3-adjacent; sketch at M2.

Minor (packet-level, not planning-level): FTS union constants (pool
sizes, indexed fields, language config) for hybrid retrieval;
extraction anti-pattern SAMPLES (negative examples in the instruction,
per the samples principle); verification-principal enforcement becomes
real (not string-convention) at M4 identity.

## Chrysopoeia methodology sketch (owner riff, 2026-07-27)

Sharpens agenda item 1; refines decision 010's "SGD" wording (same phasing,
better algorithm for our scale). NOT yet law — planning input.

- BATCH EXACT RE-FIT over online SGD: at hundreds of events the whole
  hygiene-filtered log re-solves exactly (convex). Gate events become
  pairwise ranking constraints ("added-back should outscore removed in
  that context"); hinge loss + weights ≥0 summing to 1 = LP/QP —
  normalization becomes a CONSTRAINT, not a patch. Signal-class weights
  (C5) weight the constraints. b_m fitted with shrinkage (regularized;
  sparse memories can't earn wild biases); the instant never-kill bias
  stays online (safety response, not learning). Solver: scipy/sklearn/
  cvxpy-class batteries — NO Gurobi (license = a third onboarding secret;
  6-variable problem needs no cannon). Perfect fit with the existing
  invariant: "log is primary; weights recomputed from it, never synced."
- CHAMPION/CHALLENGER (owner): incumbent scorer serves every thread;
  challengers train OFFLINE with zero latency pressure; go-live only on
  TIME-SPLIT replay win (train ≤T, beat incumbent on events >T);
  scorer_version stamp; instant rollback. Training cost is irrelevant;
  interpretability is the only real constraint.
- INTERPRETABILITY AS CONTRACT, not model class: any scorer qualifies if
  it emits honest per-feature contributions for gate cards.
- MODEL-CLASS LADDER, each rung earned by data volume + replay win, human
  gate approves class jumps in early eras:
  1. linear combiner (now → ~1k events; contributions exact);
  2. learned feature shaping / GAM (learn per-feature response curves,
     e.g. per-kind recency half-lives; still additive, contributions
     still exact, inference lookup+sum);
  3. gradient-boosted trees / LambdaMART (interactions; SHAP
     contributions — slight explanation-fidelity cost, replay decides);
  4. learned-embedding rankers (JEPA-class) at shared-palace data scale.
- FEATURE GROWTH robustness: 6 live + f_loc (law, M3) + f_gen/f_phase/
  entity/importance (queued); ragged features via the null rule; trees
  handle missing natively. The scoreboard metric (replay win-metric)
  remains the open owner-taste decision.

## Chrysopoeia ops answers (owner Q&A, 2026-07-27)

- CADENCE, two decoupled clocks: RE-FIT generously (every ~25 hygiene-
  passing gate events + nightly cron — curator-trigger pattern); GO-LIVE
  rarely (only on time-split replay win; early era also passes the
  hyperparameter console for owner approval). Version changes never move
  ground under a running thread (prepare-time scoring + scorer_version
  stamps).
- COMPUTE: palace-anchored like the curators — a scheduled Cloud Run job
  beside the palace. Rungs 1-2 are sub-second single-CPU numeric solves
  (no GPU, no LLM calls; pennies/month, D2-compatible); rung 3 still
  CPU-seconds. First era may use a manual /retrain command from the
  harness.
- DISSEMINATION: none needed — clients never score locally; every
  harness calls spine inject/prepare and the SPINE scores with the
  active scorer_config row. Activation = one row flip; all machines
  serve the new version on their next thread; zero client sync, zero
  skew. Offline replicas RECOMPUTE weights from their log (never sync);
  cloud head's active version is authoritative on reconnect. Shared
  palaces carry their own scorer_config (per-palace learning).

## The spend ledger (owner design session, 2026-07-27)

Expands agenda item 2 from attribution rules into a full subsystem —
becomes DDL + law in the M2 instrument packet (with the audit instrument
and Vitals; the dashboard is one consumer of this, not the thing itself).

- TWO COST CLASSES, never silently mixed: METERED events (exact per-action
  price at source — broker requests return real USD) vs ALLOCATED flows
  (Cloud SQL instance-hours, storage GB-months, Cloud Run cpu-seconds —
  allocation is policy, not measurement). Honesty column `cost_basis:
  measured | allocated | estimated` — the accounting rhyme of ADR-017's
  MEASURED vs JUDGED.
- ATOMIC GRAIN: spend_event, append-only, perpetual, ULID PK (owner's
  "spend unit id" — ULID not UUID per id discipline; the id IS the
  timestamp). Columns: ts, dotted extensible kind (llm.request,
  llm.embedding, infra.compute, infra.storage, …), cost_usd (nullable —
  billing arrives late), cost_basis, tokens in/out/cached, model/provider/
  quantization (A-020 logging folds in), purpose enum (building/extraction/
  curation/judge/remember/embedding/scout), principal_id, machine_id,
  origin_agent path, thread_id/run_id/prompt_id lineage, memory_id
  (nullable), ref (broker generation id / GCP billing line id), meta jsonb.
- GRAINS ARE GROUP BYs, NEVER NEW TABLES: "a query" = rollup over
  prompt_id (embedding event + allocated db sliver); cost per thread /
  run / sub-agent subtree (origin_agent prefix) / model / memory / hour
  all derive from lineage columns. Rollups derived, never stored as truth
  (log primary).
- SOURCING: broker seam writes llm.* synchronously (A-020e path grown
  up); daily reconciliation job ingests GCP billing export → infra.*
  rows AND self-audits ledger-sum vs actual invoice (drift alerted). D2
  breaker remains the independent backstop.
- HORIZON PAYOFF: memory_id on the ledger enables cost-per-citation —
  "what has this memory cost vs earned" as a curator slop/promotion
  signal. The economics term rides the schema key.
