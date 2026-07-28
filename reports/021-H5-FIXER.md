packet:            H5 — The gate (FIXER F006–F010)
session:           codex / 2026-07-27 / 86af
status:            BLOCKED
what exists now:   Harness commit 4eb40a9 binds committed removals to every
                   memory path in the current model run, adds the typed
                   Wrong edit/expire hard pause, blocks same-run project-to-
                   global save fallback, and contains the gate at 390×844.
                   Spine commit d41b286 lets a near-miss Never use the existing
                   kill/quarantine transition. Both are pushed to main.
                   F006–F010 pass against an isolated local Spine; append-only
                   Chrome evidence is under verification/h5/fixer-2026-07-27/.
deviations:        Harness DECISIONS 013 [P1.2.1a, P1.4] and Spine DECISIONS
                   019 [P1.2.1b], implementing Garden A-022/A-023. F009 needed
                   no product change after the owner aligned the checklist to
                   C.7. F011 records the cloud-authority boundary; no deployed
                   resource was mutated.
evidence:          Harness: 258 passed, 2 deselected; both 11-record canonical
                   traces passed; Ruff, lock, pre-commit, web lint/build, and
                   diff checks passed. Spine: 160 passed; Ruff, lock,
                   pre-commit, and diff checks passed. Chrome performed the
                   actual desktop/mobile clicks and typing. Three visible
                   near-miss Never decisions quarantined one isolated fixture;
                   a fourth gate omitted it. Separate real-model probes showed
                   a removed unique memory was neither answered from nor
                   refetched, and missing project context did not retry a
                   global save. Exact-ID cleanup left no active fixture rows
                   by operator-read local query. See Harness
                   verification/h5/fixer-2026-07-27/RESULTS.md for evidence
                   boundaries and links.
notes to the next agent:  Do not mark H5 DONE, claim H6, or clear the HUMAN USE
                   HOLD. Read-only audit found Cloud Run revision
                   n8-memory-palace-spine-00003-pjh still serves pre-fix image
                   e0cf50d50283cd2c4f800272b832b8166e299cab. PLAN grants cloud
                   mutations to D1's named packet and separately requires H5
                   to use deployed Spine, so F011 needs Nate's explicit narrow
                   resolution and a reset to TODO. After that resolution, a
                   fresh FIXER should deploy immutable Spine d41b286 without
                   changing IAM/SQL/secrets/billing/breaker posture, verify the
                   remote near-miss Never path plus Harness smoke, then update
                   the handoff. Physical long-press/thumb feel still belongs
                   to Nate's personal gate. The web dependency install also
                   reported one unclassified high-severity advisory; no npm
                   audit metadata was sent without explicit authorization.
