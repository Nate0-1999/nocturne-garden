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
