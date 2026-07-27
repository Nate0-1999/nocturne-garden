packet:            H5 SCOUT — human-use closing SOP
session:           codex / 2026-07-27 / 86af
status:            DONE — SCOUT COMPLETE; OWNER HOLD REMAINS
what exists now:   A live Chrome run executed all thirteen Tier 1–3 checks
                   plus the B.6 responsive repeat against the deployed Spine
                   through a Harness daemon marked h5-sop-verification.
                   Harness verification/h5/SOP-SCOUT.md classifies every
                   check and its evidence; the dated evidence directory holds
                   the screenshots and redacted DB trace.
                   Natural-language dedup/edit, edit lineage, snapshot pinning,
                   token enforcement, Unicode, cold-open, alternate removal
                   input, and memory-offline recovery worked.
                   Four runtime failures and one owner-level contract/checklist
                   fork remain under F006–F010. H5 stays DONE and its HUMAN USE
                   HOLD remains in force.
deviations:        none. No product code or specification was changed. F006
                   [P1.2.1a–b], F007 [P1.2.1b, P1.4], F008 [P1.1, P1.2.1],
                   F009 [P0], and F010 [P1.2.1c] record the unresolved
                   findings.
evidence:          Harness verification/h5/SOP-SCOUT.md and
                   verification/h5/scout-2026-07-27/: screenshots 00–39 plus
                   db-trace-redacted.json and cleanup-result.json. The
                   read-only trace found all 14 fixture units and 13 injection
                   IDs across 50 event rows with full feature vectors,
                   contiguous same-UUID edit lineage for both Gamma and the
                   dedup fixture, and the expected two-kill stats preserved on
                   the cleanup-tombstoned head. Exact-ID CAS cleanup
                   tombstoned all 14 Scout fixtures while excluding both owner
                   memories. Boot ground: Harness 243 passed, 2 deselected,
                   with web, Ruff, lock, and pre-commit gates; Spine 160
                   passed with Ruff, lock, and pre-commit gates. The same
                   suites passed again at exit.
notes to the next agent:  Do not claim H6 or clear the hold. Owner consultation
                   must resolve F006–F010, then a FIXER should repair and
                   re-run only the affected SOP slices before Nate's personal
                   gate. Population relevance and Continue latency are M2
                   tuning/taste observations, not excuses to alter scorer v0.
                   The live 390×844 rendering objectively failed; physical
                   phone long-press still separately needs Nate's hand.
                   Restart lost daemon transcript state while preserving the
                   browser catalog; F009 records the checklist/C.7 fork rather
                   than blaming a conforming H5 implementation. The raw exact-
                   duplicate response is the checklist's explicitly named
                   pre-v2.14 known gap, not a new H5 defect. Exact Scout fixture
                   IDs and tombstone proof live in the SOP; never clean by
                   label or touch the two pre-existing owner memories.
