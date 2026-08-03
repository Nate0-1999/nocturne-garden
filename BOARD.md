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
| M2D| Durable transcripts (capture-only) | J                             | DONE        | codex / 2026-08-02 / fe58    | report 038; Harness e44bfaa; ADR-025 |
| M2E| Hybrid retrieval (FTS ∪ vector) | J                                | DONE        | codex / 2026-08-02 / c2a4    | report 039; Spine 93ffd17; D.2 050(1) |
| M2C| Palace Vitals + spend strip    | M2A, M2B                          | IN_PROGRESS | codex / 2026-08-02 / c2c0    | ADR-009 item 5, v2.17; wave 1 |
| D3 | Deploy & onboarding command    | J                                 | BLOCKED     | codex / 2026-08-01 / d3a1    | report 034; F014 owner PyPI publisher setup; release code + installed-wheel evidence pushed |

| M2G| Per-message re-scoring + locks | J                                 | TODO        |                              | OQ-15 impl (v2.31/33); wave 2 headliner |
| M2F| Chrysopoeia learner v1         | J                                 | TODO        |                              | v2.32 scoreboard; PROPOSED-version authority; wave 2 |
| M2H| Extraction + queue + thread-end card | M2B, M2D                    | TODO        |                              | ADR-021 cl.4 (v2.34 trigger); v2.25 queue law; wave 2 |
| M2I| Seed ingestion                 | M2H                               | TODO        |                              | ADR-019 cl.4; wave 2 |
| M2J| Parameter registry + model device | M2B                            | TODO        |                              | ADR-023 cl.3; wave 2 |
| M2K| Memory Graph + console + contribution bars | M2B                   | TODO        |                              | ADR-009 items 3-4; Invariant 6 bars; wave 2 |
| M2L| Citation heuristic v1 (n-gram) | M2G                               | TODO        |                              | ADR-005 (OQ-5 v1 already decided); wave 2 |
| M2M| Ledger self-audit vs broker    | M2A                               | TODO        |                              | ADR-024 sourcing (v2.34 scope); wave 2 |

| M2N| Owner DB lifecycle hardening   | J                                 | TODO        |                              | v2.35/D.2 078; wave 2 |

| M2O| Fixture isolation + accounting fail-open | J                       | TODO        |                              | v2.38/D.2 081; wave 2; owner-hit incident |


Statuses: `TODO` · `IN_PROGRESS` · `DONE` · `BLOCKED` · `FAILED_JUDGMENT`.

### Active stop-line

- D3 only — F014: the owner must register the two pending PyPI trusted
  publishers before a relay creates the `v0.1.0` releases. M2C remains
  independently claimable by another relay session.

### Milestone closure

- M1 CLOSED — report 035. The superseding J verdict passes J0–J8; the owner
  cleared the verdict gate on 2026-07-31, and D.2 071 enacted the verdict's ADR
  status normalization. M2A, M2B, M2D, and M2E are DONE; M2C is next by
  deterministic board order. D3 is not an M1 closure dependency and remains
  blocked only on F014.

### Exclusions (symmetric; see PLAN §1)

- H6 ⊗ H8 — both edit the harness web SPA (gate/chat surfaces).
- M2B ⊗ M2C — both edit the web SPA (rack surfaces).
- M2H ⊗ M2J ⊗ M2K ⊗ M2C — pairwise: all edit the web SPA (rack modules).

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
