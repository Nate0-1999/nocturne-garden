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
| D3 | Deploy & onboarding command    | J                                 | TODO        |                              |         |
| I1 | Integration & AC dry run       | S1-S6, H1-H9                      | DONE        | codex / 2026-07-31 / 7f26 | report 030; Harness b64cc82; /model + J1/J2 builder repair complete |
| J  | Judge                          | I1; H5 hold cleared 2026-07-30 (owner) | DONE | claude-code / 2026-07-31 / f648 | report 031; superseding verdict PASS (J0 re-audited, J1/J2 re-executed live, J3–J8 stand); ADR normalizations ENACTED (D.2 071, v2.28) |

Statuses: `TODO` · `IN_PROGRESS` · `DONE` · `BLOCKED` · `FAILED_JUDGMENT`.

### Active stop-line

- None. The superseding J verdict is PASS (report 031;
  harness/verification/m1/VERDICT.md, claude-code / 2026-07-31 / f648).
  Per PLAN §7 the owner now reads the verdict beside its screenshots —
  only then is M1 done and M2 planning opens (M3 re-plans too). The
  verdict's ADR status-normalization proposals await that same gate. D3 is
  claimable once the owner closes the gate.

### Exclusions (symmetric; see PLAN §1)

- H6 ⊗ H8 — both edit the harness web SPA (gate/chat surfaces).

### Gate scout notes

- H5 — SCOUT DONE — codex / 2026-07-27 / 86af — report 020; FAIL; HUMAN USE HOLD remains
- H5 — FIXER BLOCKED — codex / 2026-07-27 / 86af — report 021; F006–F010
  pass locally; deployed Spine remains pre-fix; F011; HUMAN USE HOLD remains
- H5 — FIXER DONE — codex / 2026-07-28 / 86af — report 022; F011 deployed
  and remote F007 passed; HUMAN USE HOLD remains and gates J only
- H5 — OWNER HOLD CLEARED — Nate / 2026-07-30 — explicit verdict:
  "H5 human-use hold cleared. Proceed to J."
