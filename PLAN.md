# The Garden Plan — loop-enabled M1 execution

Companion to docs/SPEC.md v1.4. This is OPERATIONS, not constitution: it may
change freely; the spec may not. This document is written to be executed by a
relay of agents, generation 0 through n, each arriving with no memory of the
last. Everything an agent needs is in files; nothing lives in anyone's head.

**Workspace layout (created by Agent Zero, P0):**
```
garden/            # ops repo: THIS file (PLAN.md), BOARD.md, FLAGS.md, reports/
spine/             # product repo (SPEC C.1)
harness/           # product repo (SPEC C.1)
```
All three are sibling git repos in one workspace. Sessions launch at the
workspace root.

---

## 0. THE BOOT SEQUENCE — every agent, every session, in this order

You are one runner in a relay. The runners before you left everything you
need in files; the runners after you will have only what you leave in files.

**STEP 1 — Law.** Read `garden/PLAN.md` (this file) top to bottom. Read the
`CLAUDE.md` / `AGENTS.md` ground rules in any repo you will touch. Do NOT
read the whole spec yet — your packet will name its sections.

**STEP 2 — Ground truth.** Read `garden/BOARD.md`. Read the LAST handoff
report in `garden/reports/` (at most the last three — respect your context;
older history is in git if truly needed). Then VERIFY inherited ground: run
the test suites of both product repos. If anything is red, broken, or
contradicts the last report's claims, your role is FIXER (Section 3) — the
board's next packet waits. **Never build on unverified ground.**

**STEP 3 — Dispatch.** Determine your role from the board using Section 3's
rules. The situation assigns your role; the human does not need to.

**STEP 4 — Claim.** Edit BOARD.md: set your packet IN_PROGRESS with a session
tag (tool + date + short id) and commit this claim to `garden` BEFORE any
other work. The claim-commit is your baton pickup.

**STEP 5 — Focused law.** Now read exactly the spec sections your packet
charge names — fully, at full attention, nothing more.

**STEP 6 — Work.** Within CONTRACTS (SPEC 1.4). Journal every non-dictated
decision in the repo's DECISIONS.md citing a Problem Tree node. Defects you
discover: Blight Protocol (SPEC 2.1) — fix at the deepest containing node or
flag. Anything requiring a FORBIDDEN feature or contract change: STOP, write
a FLAG (Section 4), set your packet BLOCKED, end your session.

**STEP 7 — Handoff ritual.** In order: packet exit criteria met → both test
suites green → DECISIONS.md entries committed → handoff report written
(template, Section 4) → BOARD.md updated (DONE, or honestly back to TODO /
BLOCKED with the report explaining why) → all commits pushed with your packet
id in messages. **A clean unfinished handoff beats a dirty finished one** —
returning a half-done packet with an honest report is success, not failure.

**STEP 8 — Stop.** One packet per session. Do not begin the next packet, no
matter how much context or energy remains. Fresh ground needs fresh eyes.

---

## 1. BOARD.md — the state ledger

Agent Zero instantiates it from Section 5's packet list. One row per packet:

```
| id | packet             | deps      | status       | claim              | verdict |
|----|--------------------|-----------|--------------|--------------------|---------|
| P0 | Agent Zero         | —         | TODO         |                    |         |
| S1 | DB layer & CAS     | P0        | TODO         |                    |         |
...
```

**Statuses:** TODO · IN_PROGRESS · DONE · BLOCKED (see FLAGS.md) ·
FAILED_JUDGMENT (see verdict path).

**Rules:**
- Claim only a TODO packet whose deps are all DONE. If several qualify,
  take the lowest id (determinism beats optimization).
- **Stale claim recovery:** an IN_PROGRESS claim from a session that left no
  handoff report is stale after inspection confirms the session is dead.
  Inspect its branch: salvage what passes tests into your work if trivial,
  otherwise reset the packet to TODO, journal what you found, and note it in
  your report. Never silently adopt unverified work.
- Only the claiming session edits its row (plus the human, who may edit
  anything).

## 2. FLAGS.md — the stop-the-line file

Append-only. A FLAG is how an agent halts safely: contract ambiguity, spec
bug, FORBIDDEN-feature need, or a Blight-Protocol escalation (fix would
disturb an ancestor node). Format:
`[flag-id] [packet] [tree node] — what was found, why it cannot be resolved
locally, the minimal proposed change, what it disturbs.`
A BLOCKED packet stays blocked until the human resolves the flag (usually by
amending the spec) and resets the row to TODO.

## 3. Role dispatch (derived from the board, in priority order)

1. **FIXER** — Step 2 found broken ground, or a packet row says
   FAILED_JUDGMENT. Claim that repair: your charge is the last report or the
   verdict file; fix per the Blight Protocol; do not add features.
2. **JUDGE** — all build packets (S*, H*, I1) are DONE and J is TODO.
   **Independence check: if your context contains ANY build work from this
   milestone, REFUSE the role** — end the session instructing the human to
   launch a fresh one (different model preferred). A judge with build
   context is a rubber stamp.
3. **BUILDER** — otherwise: claim the next eligible packet (Section 1 rules)
   and follow its charge (Section 5).
4. **NOBODY** — board shows all DONE and J passed: M1 is complete. Write a
   closing report; instruct the human that M2 planning opens (SPEC B.1:
   M3 re-plans then too). Build nothing.

## 4. Templates

**Handoff report** (`garden/reports/NNN-<packet>.md`):
```
packet:            S3 — Scorer v0 + inject/prepare
session:           claude-code / 2026-07-.. / a1b2
status:            DONE | RETURNED_TODO | BLOCKED
what exists now:   (2-6 lines, concrete: files, endpoints, behaviors)
deviations:        (DECISIONS.md entry ids + tree nodes, or "none")
evidence:          (test names, verification/ paths)
notes to the next agent:  <- the generational memory; write what you wish
                   you had been told: surprises, fragile spots, half-truths
                   in your own work, the fastest path into the code.
```

**FLAG:** see Section 2 format.

## 5. Packet charges

Every charge implicitly begins with the Boot Sequence. "Sections" = the only
spec reading required.

**P0 — Agent Zero (groundskeeper).** Charge: SPEC C.10 verbatim, PLUS:
create the `garden` repo with PLAN.md (this file), BOARD.md instantiated
from this section, empty FLAGS.md, reports/ with your own report as 000;
create CLAUDE.md / AGENTS.md in both product repos from the template in
Section 6. Human reviews your journal entries and report for cultural fit
before the relay continues.

**Spine track**
- **S1 — DB layer & CAS rules.** Sections: C.2, ADR-004. Deliver: models
  matching the DDL exactly; transactional CAS-update + revision-write
  helper; tombstone helper; tests incl. forced 409 and parent_uid lineage.
  Nodes: P1.3.
- **S2 — Memory CRUD + dedup bands.** Sections: C.4 (/v1/memories, PATCH,
  GET), C.5. Deliver: create with 0.92/0.80 bands (409 duplicate /
  similar[] / insert + force), PATCH with CAS, paged list; embedding
  provider behind the C.1 interface with a deterministic fake for tests.
  Nodes: P1.4, P1.1.
- **S3 — Scorer v0 + inject/prepare.** Sections: C.3, C.4 (prepare),
  ADR-005. Deliver: six features exactly per formulas; pgvector candidate
  pool; budget-greedy selection; near-misses; pinned bypass; snapshot_ts
  semantics; injection_event writes with full features. Golden tests:
  hand-computed scores on fixed fixtures — computed from the SPEC, not from
  the code. Nodes: P1.2.
- **S4 — Commit, feedback, quarantine.** Sections: C.4 (commit, feedback),
  C.3 (never rule), C.6 (final_block format). Deliver: outcome writes;
  never → bias → quarantine at 3; "wrong" returns unit; exact final_block
  renderer; stats updates. Nodes: P1.2.1a–d.

**Harness track**
- **H1 — Envelope + daemon WS.** Sections: C.7, C.1. Deliver: envelope
  models, WS server, type routing, malformed-envelope rejection tests.
  Nodes: P3. (Eligible after P0.)
- **H2 — spine_client + contract tests.** Sections: C.4. Deliver: typed
  client mirroring every body; contract-test job against a live spine
  container covering S1–S2. Nodes: P1.1. (Deps: S2.)
- **H3 — Agent + memory tools.** Sections: C.6 (tools, instructions,
  /remember), C.5. Deliver: pydantic-ai agent — chat + memory tools ONLY;
  docstrings per spec; similar/409 surfaced to the model; /remember with
  generated label. Nodes: P1.2, P1.4. (Deps: H2.)
- **H4 — Web shell + chat.** Sections: C.1 web/, C.7, ADR-009 mobile law.
  Deliver: responsive SPA — thread list, chat pane, run.delta streaming;
  sane at 390px. Nodes: P2, P3. (Deps: H1.)
- **H5 — The gate.** Sections: C.6 steps 1–4, ADR-005 gate UX. Deliver:
  prepare-on-send; modal with full bodies + per-feature scores; one-tap ✕,
  modifier reasons, near-miss add-back; hard pause; commit → run. Nodes:
  P1.2.1a–c. (Deps: S4, H4.) The human personally uses the result before
  the relay continues (Section 7).
- **H6 — Memory panel.** Sections: C.6 (live panel), C.4 (feedback, PATCH).
  Deliver: live list; ad-hoc remove → mid_thread_removed → re-render next
  call; edit flow with CAS conflict surfacing; manual pin toggle. Nodes:
  P1.2.1d, P1.3. (Deps: H5.)

**Closing**
- **I1 — Integration & AC dry run.** Sections: C.8. Deliver: compose
  end-to-end; walk all seven criteria as a BUILDER (not judge); fix gaps;
  seed demo memories; verification/README pointing a judge at everything.
  (Deps: all S*, H*.)
- **J — Judge.** Sections: B.6, C.8, C.9 ONLY. Fresh session, different
  model than the builders (Codex if built by Claude Code). Execute J0–J8
  with browser automation for screenshots; produce
  `verification/m1/VERDICT.md`. Any FAIL → set FAILED_JUDGMENT on the
  packets the verdict implicates (or I1 if diffuse); a FIXER inherits the
  verdict.

## 6. CLAUDE.md / AGENTS.md template (both product repos)

```
# Ground rules (read every session)
1. You are one runner in a relay governed by ../garden/PLAN.md — run its
   Boot Sequence before anything else.
2. The constitution is docs/SPEC.md (v1.4): sections 1 -> 2 -> B -> C; read
   fully the sections your packet names.
3. You are in Milestone M1 unless your charge says otherwise. Feature
   ledger (SPEC B.4) applies: FORBIDDEN means do not build, stub, or
   "prepare for" — flag instead (garden/FLAGS.md).
4. Every non-dictated decision -> DECISIONS.md, citing a Problem Tree node
   (SPEC §2). Features that cannot name their problem do not get built.
5. Defects -> Blight Protocol (SPEC 2.1): deepest containing node;
   escalate ancestors/contracts via FLAG.
6. Contracts are literal: DDL C.2, API C.4, envelope C.7, invariants 1.3.
7. Done means judged (SPEC B.6): leave experiential + traced evidence.
8. One packet per session. Handoff ritual, then stop.
```

## 7. Tool roles & human calibration gates

- **Claude Code:** primary builder; pre-commit hook greps FORBIDDEN
  patterns (weight updates, extraction, relay client, maintenance jobs,
  auth beyond bearer) and blocks with a pointer to SPEC B.4.
- **Codex:** the judge (fresh sessions, Playwright), and occasional
  second-opinion reviewer of PRs.
- **Code Puppy:** daily driver; after S4, pointing a Code Puppy experiment
  at the spine API is the first live test of ADR-002's bidirectional
  coupling.
- **Human gates (you, between sessions):**
  - after P0 — journal/report culture check; redo P0 if it reads like a
    contractor wrote it (cheapest possible recalibration).
  - after S3 — hand-verify one injection's scores against C.3 by
    calculator (catches golden tests written from code instead of spec).
  - after H5 — use the gate yourself for a day of real prompts; the gate
    is the product's soul and no judge substitutes for your hands.
  - after J — read VERDICT.md beside its screenshots; only then is M1
    done, and M2 planning opens (SPEC B.1; M3 re-plans too).
- Otherwise your between-session review is: the diff, the DECISIONS.md
  entries, the handoff report. Supervise the relay closely through S2;
  loosen as the reports earn it.
