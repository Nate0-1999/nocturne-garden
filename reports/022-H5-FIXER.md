packet:            H5 — The gate (FIXER F011)
session:           codex / 2026-07-28 / 86af
status:            DONE
what exists now:   Cloud Run revision n8-memory-palace-spine-00004-vs2 serves
                   100% of default traffic from immutable image index
                   sha256:dfe9fd5465038e9ac82ca61a49fd93f872afd041dae60b992a5b625fcb694cbb,
                   built from repaired Spine commit d41b286. Live typed Harness
                   smoke and the remote F007 quarantine path pass.
deviations:        none. The single F011 image update followed D.2 064; no
                   product contract, migration, resource, IAM, secret, SQL,
                   traffic posture, billing, or breaker change was made.
evidence:          Harness commit 33e5f46:
                   verification/h5/f011-2026-07-28/RESULTS.md,
                   remote-verification.json, transport-summary.json.
                   Spine commit ce76417:
                   verification/h5/f011-2026-07-28/deployment.json.
                   Three Never outcomes descended scores 0.7966666222 →
                   0.6466666460 → 0.4966666400; the third, in the near_miss
                   lane, set never_kills=3 and quarantined the fixture. A
                   fourth prepare returned no fixture and cleanup tombstoned
                   it at revision 7. Protected state matched before/after.
                   Harness: 258 passed, 2 deselected; Spine: 160 passed; Ruff,
                   lock checks, repository hooks, web lint/build, and diff
                   checks passed.
notes to the next agent:  F011's single-use grant is consumed; any further
                   cloud mutation requires fresh owner authority. H5 is DONE,
                   but only Nate can clear the HUMAN USE HOLD, which now gates
                   J only. H6 is the next deterministic eligible packet; H8
                   remains excluded while H6 is IN_PROGRESS. The deployed
                   digest is immutable, but the historical Dockerfile's
                   unpinned base name and unlocked pip ranges mean a rebuild
                   from Git alone is not guaranteed bit-identical. No glossary
                   entry is proposed; this packet coined no project term.
