# M1 Relay Board

Packets are claimed and completed according to `PLAN.md`. Row order is the
deterministic packet order when more than one dependency-ready packet exists.

| id | packet                         | deps                              | status      | claim                        | verdict |
|----|--------------------------------|-----------------------------------|-------------|------------------------------|---------|
| P0 | Agent Zero                     | —                                 | DONE        | codex / 2026-07-17 / 70ed    |         |
| S1 | DB layer & CAS rules           | P0                                | DONE        | codex / 2026-07-17 / 5f1c    |         |
| S2 | Memory CRUD & dedup bands      | S1                                | DONE        | codex / 2026-07-17 / c842    |         |
| S3 | Scorer v0 + inject/prepare     | S2                                | DONE        | codex / 2026-07-19 / b73a    |         |
| S4 | Commit, feedback, quarantine   | S3, S5                            | DONE        | codex / 2026-07-19 / a4c9    |         |
| S5 | origin_path metadata           | S3                                | DONE        | codex / 2026-07-19 / e5a7    |         |
| S6 | /v1/search                     | S2                                | DONE        | codex / 2026-07-19 / f6a2    |         |
| D1 | GCP deploy & remote verify     | S4, S7                            | DONE        | codex / 2026-07-20 / b82e    | report 017 |
| S7 | Broker-routed embeddings       | S2                                | DONE        | codex / 2026-07-20 / c105    |         |
| D2 | Billing circuit breaker        | P0                                | DONE        | codex / 2026-07-26 / 86af    | report 019 |
| H1 | Envelope + daemon WS           | P0                                | DONE        | codex / 2026-07-20 / 25ff    |         |
| H2 | spine_client + contract tests  | S2                                | DONE        | codex / 2026-07-20 / db84    |         |
| H3 | Agent + memory tools           | H2, S6                            | DONE        | codex / 2026-07-20 / 7c3a    |         |
| H4 | Web shell + chat               | H1, H7                            | DONE        | codex / 2026-07-20 / c84d    |         |
| H5 | The gate                       | D1, H4                            | DONE        | codex / 2026-07-28 / 86af    | report 022; remote F007 PASS; HOLD CLEARED owner 2026-07-30 |
| H6 | Memory panel                   | H5                                | DONE        | codex / 2026-07-28 / 86af    | report 023 |
| H7 | Envelope v1.12 & loop controls | H1                                | DONE        | codex / 2026-07-20 / 9d2f    |         |
| H8 | Gate-day polish: md + model    | H5                                | DONE        | codex / 2026-07-30 / 86af    | report 024 |
| H9 | Model policy routing A-020/021 | H5                                | DONE        | codex / 2026-07-30 / 86af    | report 025 |
| I1 | Integration & AC dry run       | S1-S6, H1-H9                      | DONE        | codex / 2026-07-31 / 7f26 | report 030; Harness b64cc82; /model + J1/J2 builder repair complete |
| J  | Judge                          | I1; H5 hold cleared 2026-07-30 (owner) | DONE | claude-code / 2026-07-31 / f648 | report 031; superseding verdict PASS (J0 re-audited, J1/J2 re-executed live, J3–J8 stand); ADR normalizations ENACTED (D.2 071, v2.28); OWNER GATE CLEARED 2026-07-31 |
| M1C| M1 closing report              | J; owner gate cleared 2026-07-31  | DONE        | codex / 2026-08-01 / b5e2    | report 035; M1 CLOSED; M2 relay open |
| M2A| Spend ledger core              | J                                 | DONE        | codex / 2026-08-01 / a4d2    | report 036; Spine 371b698; Harness 4a59046; A-027 |
| M2B| Rack refound + NEO-NOIR identity | J                               | DONE        | codex / 2026-08-01 / c8e4    | report 037; Harness 05d4e98; B.6 r7+r8 |
| M2D| Durable transcripts (capture-only) | J                             | DONE        | codex / 2026-08-02 / fe58    | report 038; Harness e44bfaa; Harness Decision 025 |
| M2E| Hybrid retrieval (FTS ∪ vector) | J                                | DONE        | codex / 2026-08-02 / c2a4    | report 039; Spine 93ffd17; D.2 050(1) |
| M2C| Palace Vitals + spend strip    | M2A, M2B                          | DONE        | codex / 2026-08-02 / c2c0    | report 040; Spine ac332d2; Harness 3f2f6ed; B.6 r7+r8 |
| D3 | Deploy & onboarding command    | J                                 | BLOCKED     | codex / 2026-08-01 / d3a1    | report 034; F014 owner PyPI publisher setup; release code + installed-wheel evidence pushed |

| M2G| Per-message re-scoring + locks | J                                 | DONE        | codex / 2026-08-03 / b803    | report 041; Spine aab87ab; Harness 5a8338a; A-030; B.6 r7+r8 |
| M2F| Chrysopoeia learner v1         | J                                 | DONE        | codex / 2026-08-03 / c6a1    | report 042; Spine 326de29; A-031 |
| M2H| Extraction + queue + thread-end card | M2B, M2D                    | DONE        | codex / 2026-08-03 / d4e2    | report 043; Spine e73d420; Harness c452499; A-032; B.6 r7+r8 |
| M2I| Seed ingestion                 | M2H                               | DONE        | codex / 2026-08-03 / 92ad    | report 044; Spine f3b3e53; Harness 456397f; A-033; B.6 r7+r8 |
| M2J| Parameter registry + model device | M2B                            | DONE        | codex / 2026-08-03 / 3d91    | report 045; Harness da5c220; A-034; B.6 r7+r8 |
| M2K| Memory Graph + console + contribution bars | M2B                   | DONE        | codex / 2026-08-03 / 7ac4    | report 046; Spine 8ffff28; Harness 1249fe2; A-035; B.6 r7+r8 |





| M2L| Citation heuristic v1 (n-gram) | M2G                               | DONE        | codex / 2026-08-03 / a83f    | report 047; Spine 2974d81; Harness 216888b; A-036 |
| M2M| Ledger self-audit vs broker    | M2A                               | DONE        | codex / 2026-08-03 / d91c    | report 048; Spine eaf5bc7; Harness 38720ae; A-037; B.6 r7 (gate audit 2026-08-04: r8 tag removed — no SOP artifact backs it; Vitals drift-alert surface gets its user pass at the M2 scout) |
| M2N| Owner DB lifecycle hardening   | J                                 | DONE        | codex / 2026-08-04 / 4d2c    | report 058; Harness 2b0c020; Spine 475513b; A-041/A-046; live cloud receipt verified |
| M2O| Fixture isolation + accounting fail-open | J                       | DONE        | codex / 2026-08-03 / c81e    | report 050; Harness 2d32540+5ab237f; A-038; B.6 r7 |
| M2R| Context Bars (+memory category) | M2B                              | DONE        | codex / 2026-08-04 / 8a4e    | report 051; Harness 1887756; A-039; B.6 r7+r8 |
| M2Q| Test motivation sweep + coverage report | J                        | DONE        | codex / 2026-08-04 / 91b7    | report 059; Harness c1bbd01; Spine 2bc3141; A-040 |
| M2P| Injection Console: audition + what-if | M2K, M2F                   | DONE        | codex / 2026-08-04 / 4f6a    | report 060; Spine ffb4142; Harness bfe9308; A-047/A-048; B.6 r7+r8 |
| M2X| M2 gate day (HUMAN)            | all M2 packets DONE               | TODO        |                              | HUMAN packet — never agent-claimed; scout precedes and derives the checklist (B.6 r9 + PLAN §3.3); HUMAN USE HOLD gates M2J |
| M2J| M2 judge                       | M2X hold cleared (owner)          | TODO        |                              | fresh CLAUDE CODE session per B.6 independence; judges the M2 wave |

| M2S| Rung-2 first-class startup     | all M2 packets DONE               | DONE        | codex / 2026-08-05 / c805    | report 061; Harness 20414bf; owner spin friction removed before M2X |


Statuses: `TODO` · `IN_PROGRESS` · `DONE` · `BLOCKED` · `FAILED_JUDGMENT`.

### Active stop-line

- D3 — F014: release remains owner-deferred; the two pending PyPI trusted
  publishers are required before a relay creates the `v0.1.0` releases.

### Milestone closure

- M1 CLOSED — report 035. The superseding J verdict passes J0–J8; the owner
  cleared the verdict gate on 2026-07-31, and D.2 071 enacted the verdict's ADR
  status normalization. M2A, M2B, M2C, M2D, M2E, M2F, M2G, and M2H are DONE;
  M2 wave one is complete and M2 wave two is open. M2I, M2J, and M2K are DONE;
  M2L, M2M, M2N, M2O, and M2R are DONE; M2N now includes its lifecycle
  foundation, verified local backup authority, informed side-by-side restore,
  doctor, startup warning, passive resource Vitals, bounded daemon soak, and a
  live-verified owner-cloud pre-migration receipt through report 058. Later
  independent rows remain dependency-ready. D3 is not an M1
  closure dependency and remains blocked only on F014.

### Exclusions (symmetric; see PLAN §1)

- H6 ⊗ H8 — both edit the harness web SPA (gate/chat surfaces).
- M2B ⊗ M2C — both edit the web SPA (rack surfaces).
- M2H ⊗ M2J ⊗ M2K ⊗ M2C ⊗ M2P ⊗ M2R — pairwise: all edit the web SPA (rack modules).

### Gate scout notes

- H5 — SCOUT DONE — codex / 2026-07-27 / 86af — report 020; FAIL; HUMAN USE HOLD remains
- H5 — FIXER BLOCKED — codex / 2026-07-27 / 86af — report 021; F006–F010
  pass locally; deployed Spine remains pre-fix; F011; HUMAN USE HOLD remains
- H5 — FIXER DONE — codex / 2026-07-28 / 86af — report 022; F011 deployed
  and remote F007 passed; HUMAN USE HOLD remains and gates J only
- H5 — OWNER HOLD CLEARED — Nate / 2026-07-30 — explicit verdict:
  "H5 human-use hold cleared. Proceed to J."
- J — OWNER VERDICT GATE CLEARED — Nate / 2026-07-31 — explicit verdict:
  "J verdict gate cleared. Proceed to D3."
