packet:            D2 FIXER — Ruff format ground repair
session:           codex / 2026-07-26 / 86af
status:            DONE
what exists now:   Spine commit 86c868c restores the inherited Ruff format
                   gate in infra/billing-breaker/deployment_checks.py. The
                   _is_project_service_agent(...) condition was mechanically
                   collapsed to Ruff's canonical form; behavior and scope are
                   unchanged.
deviations:        none. No decision record was required for a format-only
                   repair.
evidence:          Post-fix Spine ground is green: 160 passed; Ruff lint and
                   format checks, uv lock, pre-commit, and git diff checks
                   passed. Harness sibling ground is green: 243 passed with
                   2 deselected; web lint and production build, Ruff lint and
                   format, uv lock, pre-commit, and git diff checks passed.
notes to the next agent:  Boot correctly dispatched this FIXER ahead of SCOUT
                   when inherited ground was red. H5's HUMAN USE HOLD remains
                   intact. Start a fresh relay; with green ground it should
                   dispatch SCOUT. Before the live Chrome SOP, ensure the
                   Harness daemon is restarted with
                   MACHINE_ID=h5-sop-verification, then follow the H5 closing
                   checklist and SPEC B.6 rule 9. Do not clear the hold.
