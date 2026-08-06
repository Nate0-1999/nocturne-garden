# The Garden Plan — loop-enabled M1 execution

Companion to docs/SPEC.md v2.6. This is OPERATIONS, not constitution: it may
change freely; the spec may not. This document is written to be executed by a
relay of agents, generation 0 through n, each arriving with no memory of the
last. Everything an agent needs is in files; nothing lives in anyone's head.

**Workspace layout (created by Agent Zero, P0):**
```
garden/            # ops repo: THIS file (PLAN.md), SPEC.md (the MASTER
                   #   constitution), BOARD.md, FLAGS.md, AMENDMENTS.md,
                   #   reports/          remote: Nate0-1999/nocturne-garden
spine/             # product repo (SPEC C.1)   remote: Nate0-1999/nocturne-spine
harness/           # product repo (SPEC C.1)   remote: Nate0-1999/nocturne-harness
```
All three are sibling git repos in one workspace. Sessions launch at the
workspace root. `garden/SPEC.md` is the constitution's master; each product
repo carries a frozen copy at `docs/SPEC.md`, refreshed only at human gates.
Amending the master is a human act — agents propose via FLAGS, or enact
qualifying COMPLETIONS via AMENDMENTS.md (Section 2), and never edit
SPEC.md themselves.

**Cloud footprint (D1 + D2; live-audited 2026-07-28):** GCP project
`n8-memory-palace` is ACTIVE and billed, with a $100/month
BILLING_ACCOUNT-scoped budget (`82be62a3-…`) alerting at 50/90/100%. The D2
billing circuit breaker is ARMED: that budget is wired (schema 1.0) to the
`billing-breaker` Pub/Sub topic, which feeds a private no-retry Cloud Run
function that detaches project billing at cost ≥ budget. Least-privilege
verified — topic accepts only Google's `billing-budget-alert@system`
publisher, the function is invokable only by its trigger SA, and detach
authority (`roles/billing.projectManager`) is held only by the
`billing-breaker-runtime` SA (spine DECISIONS 016–018; report 018). An
older project-scoped $100 budget (`304a09a7-…`) remains as alert-only
redundancy. Cloud SQL `n8-memory-palace-db` is RUNNABLE in `us-central1`
(Postgres 16, db-f1-micro), contains the `spine` database and built-in user,
has backups/PITR and deletion protection enabled, and is migrated to Alembic
`0002 (head)`. Three region-pinned secrets and the dedicated least-privilege
`spine-runtime` identity feed one Gen2 Cloud Run service from the immutable
regional `spine` Artifact Registry repository. Revision
`n8-memory-palace-spine-00004-vs2`, built from Spine commit `d41b286` and
requested by immutable image index `sha256:dfe9fd5465038e9ac82ca61a49fd93f872afd041dae60b992a5b625fcb694cbb`,
serves 100% of default traffic at
`https://n8-memory-palace-spine-713925718873.us-central1.run.app`. GCS was not
created: it is not an M1 dependency, and snapshot lifecycle remains M4 horizon.

D1 passed an application-bearer health check, real OpenRouter create → hard
duplicate → prepare → commit round trip, and typed Harness client smoke. Cloud
Run reserves some paths ending in `z`, so remote verification uses Spine's
authenticated, OpenAPI-hidden `/health` alias; the specified `/healthz` remains
the local `docker compose` C.8 acceptance path. The ignored `harness/.env` is
mode 0600 and points at the deployed service; deployed runtime credentials live
only in Secret Manager. OpenRouter is the default for chat and embeddings, so
no OpenAI credits are needed and the OPENAI line is a direct-provider-only
legacy slot. Every cloud command must still carry explicit project and region
flags; ambient gcloud configuration is never authority.

---

## 0. THE BOOT SEQUENCE — every agent, every session, in this order

You are one runner in a relay. The runners before you left everything you
need in files; the runners after you will have only what you leave in files.

**STEP 1 — Law.** Read `garden/PLAN.md` (this file) top to bottom. Read
`garden/AMENDMENTS.md` — enacted completions there are law, equal to the
spec. Read `garden/GLOSSARY.md` — the project's proper nouns; every term
in law is used in its glossary sense, and if you coin or meet an
undefined term, PROPOSE its entry in your handoff report. Read the
`CLAUDE.md` / `AGENTS.md` ground rules in any repo you will
touch. Do NOT read the whole spec yet — your packet will name its sections.

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
charge names — fully, at full attention, nothing more. If your packet
touches user-facing surface (UI, visualization, interaction), ALSO read
`garden/NATES_VISION.md` — the owner's extended vision (GUIDANCE per SPEC
1.4): deviate from it only with a journaled DECISIONS.md entry.

**STEP 6 — Work.** Within CONTRACTS (SPEC 1.4). Journal every non-dictated
decision in the repo's DECISIONS.md citing a Problem Tree node. Defects you
discover: Blight Protocol (SPEC 2.1) — fix at the deepest containing node or
flag. A contract that is silent or self-inconsistent on a detail you need:
COMPLETION — enact it in AMENDMENTS.md and keep moving (Section 2 says what
qualifies). Anything requiring a FORBIDDEN feature or a true contract
change: STOP, write a FLAG (Section 4), set your packet BLOCKED, end your
session.

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
- EXCLUSIONS (owner, 2026-07-28; D.2 065): a packet may declare an
  exclusion set — packets it must never be IN_PROGRESS alongside
  (declared in the board's Exclusions note; always symmetric). Claim law
  extends: a packet is claimable only if no excluded packet is
  IN_PROGRESS. Exclusions express FILE-SURFACE CONFLICTS (same
  subsystem, likely merge collisions), never ordering — ordering is
  deps. Dormant under a serial relay (one session = vacuously
  satisfied); it is the safety that makes CONCURRENT sessions legal:
  any independent set of ready, mutually non-excluded packets may be
  claimed by parallel sessions, each still obeying one-packet-per-
  session and the full boot sequence. The claim-commit remains the
  lock; a push race means rebase and take the next eligible packet.
- HUMAN packets or substeps explicitly marked in Section 5 are never claimed
  or executed by an agent. The current D1 packet is relay-owned. D2's
  destructive billing-breaker deployment remains human-only even though its
  build packet is DONE.
- **Stale claim recovery:** an IN_PROGRESS claim from a session that left no
  handoff report is stale after inspection confirms the session is dead.
  Inspect its branch: salvage what passes tests into your work if trivial,
  otherwise reset the packet to TODO, journal what you found, and note it in
  your report. Never silently adopt unverified work.
- Only the claiming session edits its row (plus the human, who may edit
  anything).

## 2. AMENDMENTS.md & FLAGS.md — solve it yourself, or stop the line

Contract trouble has two exits. Take the first whenever it qualifies; the
second is for what the first cannot carry.

**AMENDMENTS.md — decide-and-declare (the default).** Where law is SILENT or
SELF-INCONSISTENT on a detail your packet needs — a missing field, an
undefined response shape, a promise the DDL forgot — do not halt: choose the
minimal completion that honors the spec's stated intent, append it to
`garden/AMENDMENTS.md` as an EXACT spec diff (template, Section 4), cite the
amendment id in DECISIONS.md, and build against it. Enacted amendments are
law for every later agent and the judge. The human audits them at the normal
between-session review and may veto; a veto becomes a FIXER charge.
A completion QUALIFIES only if it (a) touches no Invariant (SPEC 1.3), no
FORBIDDEN row (B.4), no auth or data-loss semantics; (b) reverses no ADR;
(c) changes nothing an already-DONE packet built; (d) contradicts no
explicit spec sentence — it fills silence or repairs an internal
contradiction in the direction the spec itself points. Rule of thumb: if a
wrong guess would rework committed work or violate standing law, FLAG; if it
can be completed compatibly and declared where every future agent will read
it, complete it and move on.

**FLAGS.md — stop-the-line.** Append-only. A FLAG is how an agent halts
safely when completion does not qualify: FORBIDDEN-feature need, Invariant
conflict, ADR reversal, a Blight-Protocol escalation (fix would disturb an
ancestor node), or a genuine design fork (two readings, materially different
products). Format:
`[flag-id] [packet] [tree node] — what was found, why COMPLETION does not
qualify, the minimal proposed change, what it disturbs.`
A BLOCKED packet stays blocked until the human resolves the flag (usually by
amending the spec) and resets the row to TODO.

## 3. Role dispatch (derived from the board, in priority order)

1. **FIXER** — Step 2 found broken ground, a packet row says
   FAILED_JUDGMENT, or the human vetoed an amendment (the veto note in
   AMENDMENTS.md is your charge). Claim that repair: your charge is the last
   report, the verdict file, or the veto; fix per the Blight Protocol; do
   not add features.
2. **JUDGE** — all build packets (S*, H*, I1) are DONE and J is TODO.
   **Independence check: if your context contains ANY build work from this
   milestone, REFUSE the role** — end the session instructing the human to
   launch a fresh one (different model preferred). A judge with build
   context is a rubber stamp.
3. **SCOUT** (SPEC B.6 rule 9) — milestone progress is gated on a HUMAN
   gate: a HUMAN packet row, or a HUMAN USE HOLD note on a DONE packet
   (e.g. H5's hold). Do NOT claim a build packet. Your claim: append a
   scout note (session tag) under the gated row and commit. Execute the
   gate's checklist — H5's is garden/notes/h5-closing-checklist.md, agent
   tiers — as a rule-8 SOP through live browser use, exactly as the owner
   would, under machine_id '<gate>-sop-verification' (events
   hygiene-excluded per SPEC v2.7); classify each item PASS / FAIL /
   NEEDS-TASTE; tombstone your fixtures (record ids); write
   verification/<gate>/SOP-SCOUT.md in the harness repo + a handoff
   report; FAILs → Blight Protocol / flag for owner consultation;
   NEEDS-TASTE → listed for the owner's personal session. You never clear
   the hold — only the owner does. If a scout report for this gate
   already exists and ground is unchanged, report so and stop (no
   re-scouting unchanged ground). A gated row with NO written checklist:
   derive one from the packet's charge and its named spec sections,
   commit it as garden/notes/<gate>-closing-checklist.md marked PROPOSED
   (owner blesses or amends it), then execute its agent-safe tiers —
   a missing file is work to do, not a reason to stop.
4. **BUILDER** — otherwise: claim the next eligible packet (Section 1 rules)
   and follow its charge (Section 5).
5. **NOBODY** — board shows all DONE and J passed: M1 is complete. Write a
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

**Amendment** (`garden/AMENDMENTS.md`, append-only):
```
[A-NNN] [packet] [spec section] [tree node]
gap: what the spec fails to say, or says twice differently (one sentence)
law: the exact contract text as it now stands — a precise diff or the full
     amended body/DDL/shape, complete enough that both repos and the judge
     implement identically from this entry alone
why: how this is the spec's own intent completed, not new design (one line)
```

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
  renderer; stats updates. Nodes: P1.2.1a–d. (Deps: S3, S5 — S5 first:
  S4's wrong_removed serializes MemoryUnit, which now carries origin_path.)
- **S5 — origin_path metadata.** Sections: C.2, C.4 (as amended),
  ADR-005 location-relevance note. Deliver: migration 0002 adding
  origin_path; create/PATCH/MemoryUnit surfaces carrying it; scoring
  UNTOUCHED (inert metadata — scorer v0 neither reads it nor may any later
  scorer penalize its absence, per the null rule); tests. Nodes: P1.3.
  (Deps: S3.)
- **S7 — Broker-routed embeddings.** Sections: C.5 (v2.3), C.1 embeddings
  interface. Deliver: `embed_base_url` setting wired through to the
  existing adapter's base_url parameter; embed_model default updated to
  the namespaced broker id; compose/env passthrough
  (SPINE_EMBED_BASE_URL); dimension validation unchanged (1536 stays
  law); tests: config default = OpenRouter, override = direct provider,
  both against the deterministic fake; OpenAPI freshness. Small packet —
  the adapter already accepts base_url; this exposes it. Nodes: P1.1.
  (Deps: S2.)
- **S6 — /v1/search.** Sections: C.4 (search, shared shapes), C.3
  (candidate rule). Deliver: POST /v1/search per the exact body — embed
  query; top-k by cosine over ACTIVE units of the principal, applying the
  C.3 project filter when project_key is supplied; results as MemoryCards
  with score = cosine similarity, features/rank null; tests on
  deterministic embeddings. The last 501 dies here. Nodes: P1.1.
  (Deps: S2.)

**Cloud gate**
- **D1 — GCP deploy & remote verification.** Sections: ADR-003, C.5, C.8.
  Starting truth is the live-audited cloud footprint at the top of this plan.
  In project `n8-memory-palace`, region `us-central1`, the claiming relay agent
  may perform only this packet's named mutations:
  1. tighten `harness/.env` to mode 0600 without printing values; create the
     built-in Cloud SQL user/database `spine` with a generated credential;
     enable automated backups/PITR and deletion protection; run production
     Alembic through Cloud SQL Auth Proxy and prove head `0002` (migrations
     never run at container boot);
  2. enable Secret Manager; create secrets for the complete database URL,
     static Spine token, and OpenRouter key; bind the latter to runtime env
     `SPINE_OPENAI_API_KEY`, the adapter's generic compatible-bearer slot;
  3. create dedicated runtime identity `spine-runtime`, granting only Cloud
     SQL Client at the project and Secret Accessor on those exact secrets;
     never deploy on the broad default compute identity;
  4. create exactly one Docker-format Artifact Registry repository, `spine`,
     in `us-central1`; use the active deployer locally (not Cloud Build or a
     new build identity) to build the current Spine Dockerfile for Linux/amd64.
     This Apple-Silicon workspace uses Homebrew `docker-buildx` for the required
     cross-build. If it is absent, D1 may install only that package and add
     `/opt/homebrew/lib/docker/cli-plugins` to Docker's `cliPluginsExtraDirs`,
     preserving all other Docker config. Prove Buildx,
     tag the image
     `us-central1-docker.pkg.dev/n8-memory-palace/spine/spine:<SPINE_SHA>`, and
     push it without any build-IAM mutation. Deploy exactly one service,
     `n8-memory-palace-spine`, from that immutable tag with the SQL attachment,
     gen2 execution, port 8000, max scale 1, the three secret env bindings, and
     explicit C.5 broker URL/model.
     Cloud Run transport must allow unauthenticated invocation because the
     frozen application boundary is the Spine static bearer; this does not
     make the API itself unauthenticated;
  5. verify authenticated `/health` on Cloud Run, then a real OpenRouter-backed
     create → hard-duplicate → prepare → commit round trip; update only the
     ignored local `harness/.env` `SPINE_URL`, preserve mode 0600, and prove a
     typed Harness client call against the deployed URL. Retain `/healthz` for
     local C.8 acceptance: Cloud Run reserves some paths ending in `z`, so that
     exact path can be intercepted before the container.

  Every command uses explicit `--project=n8-memory-palace` and the applicable
  `--region`/`--location=us-central1`. Never echo, commit, or place secret
  values in normal Cloud Run environment settings or reports. No deletes, GCS,
  billing/budget/D2 changes, broad IAM, default-runtime use, unrelated resource
  edits, or destructive replacement are authorized. Unexpected existing state,
  a non-forward migration, interactive reauthentication, or any need for wider
  authority returns the packet honestly to TODO/BLOCKED for the human.

  Exit: both product suites green; redacted evidence under
  `spine/verification/d1/` records project/region, DB and Alembic state, backup
  protection, service URL, revision, image digest, Spine SHA, IAM topology,
  and the remote round trip; the D1 handoff records the URL and BOARD is DONE.
  H5 and later Harness work use the deployed Spine. Local contract tests may
  still use their deterministic container. (Deps: S4, S7.) Rationale: B.3
  commits M1 to "spine on Cloud Run"; the gate prevents the relay from
  finishing M1 having only ever talked to localhost.
- **D2 — Billing circuit breaker (build: agent; deploy: HUMAN). DONE +
  ARMED 2026-07-21.** Pure infra (Google's "disable billing with
  notifications" pattern), in spine `infra/billing-breaker/`: a private
  no-retry Cloud Run function on a `billing-breaker` Pub/Sub topic that
  detaches project billing at cost ≥ budget; unit-tested on fixtures with
  no live calls; a human-only deploy script + runbook. Agents built and
  tested; the human armed it at the gate (see cloud footprint above;
  report 018; spine DECISIONS 016–018). The deploy preflight was hardened
  at the gate to run against a default-posture GCP project. Revisit when
  Google's Spend Caps (private preview 2026; covers Cloud Run but NOT
  Cloud SQL) reach GA. Nodes: P4. (Deps: P0.)

**Harness track**
- **D3 — The deploy & onboarding command (packaging wave).** Sections:
  ADR-019, ADR-013. Deliver: `nocturne` CLI (init/up/deploy/open) per the
  two-secrets rule; both wheels with bundled web assets and packaged
  migrations; `nocturne deploy` = the executed D1 runbook as idempotent
  code with --dry-run, honoring D.2 045's boundaries (no deletes, no
  broad IAM; the D2 breaker step prompts for HUMAN confirmation);
  quickstart README. Verification per ADR-019's clause: fresh machine +
  Docker + one OpenRouter key → working browser chat. Nodes: P4.
  Owner-resumed 2026-08-01 under SPEC v2.30 / D.2 073: publish the Harness
  distribution as `nocturne-ai`, retain console command `nocturne`, and run
  D3 concurrently with the independently claimed M1C/M2 relays. (Deps: J.)

- **H1 — Envelope + daemon WS.** Sections: C.7, C.1. Deliver: envelope
  models, WS server, type routing, malformed-envelope rejection tests.
  Nodes: P3. (Eligible after P0.)
- **H2 — spine_client + contract tests.** Sections: C.4. Deliver: typed
  client mirroring every body; contract-test job against a live spine
  container covering S1–S2. Nodes: P1.1. (Deps: S2.)
- **H3 — Agent + memory tools.** Sections: C.6 (tools, instructions,
  /remember), C.5, ADR-013. Deliver: pydantic-ai agent — chat + memory
  tools ONLY; docstrings per spec; similar/409 surfaced to the model;
  /remember with generated label. MemoryCapability is the ADR-013 seam's
  FIRST feature: define the minimal internal capability protocol and the
  single adapter module; ship it as a pydantic-ai v2 Capability subclass;
  nothing outside the adapter imports pydantic-ai capability machinery.
  Nodes: P1.2, P1.4. (Deps: H2.)
- **H4 — Web shell + chat.** Sections: C.1 web/, C.7 (as amended), ADR-009
  mobile law, B.6 rule 7. Deliver: responsive SPA — thread list, chat pane,
  run.delta streaming; sane at 390px. Verification per B.6 rule 7: drive
  the real UI with browser automation as a user would; screenshots of every
  acceptance state committed as evidence. Nodes: P2, P3. (Deps: H1, H7.)
- **H5 — The gate.** Sections: C.6 steps 1–4, ADR-005 gate UX, B.6 rule 7.
  Deliver: prepare-on-send; modal with full bodies + per-feature scores;
  one-tap ✕, modifier reasons, near-miss add-back; hard pause; commit →
  run. Verification per B.6 rules 7 AND 8 (scripted checks + an executed live
  SOP walkthrough with prose observations). Nodes: P1.2.1a–c. (Deps: D1, H4.) The human personally uses the result
  before the relay continues (Section 7). HOLD RESCOPED (owner,
  2026-07-28; D.2 064): the HUMAN USE HOLD gates J only — H6/H8/H9/I1
  build while the owner's real use continues in parallel (the deep
  population-feel verdict is already deferred to the real-work era per
  the closing checklist); only the owner clears the hold.
- **H6 — Memory panel.** Sections: C.6 (live panel), C.4 (feedback, PATCH),
  B.6 rule 7. Deliver: live list; ad-hoc remove → mid_thread_removed →
  re-render next call; edit flow with CAS conflict surfacing; manual pin
  toggle. Verification per B.6 rules 7 AND 8 (scripted checks + an executed live
  SOP walkthrough with prose observations). Nodes: P1.2.1d, P1.3. (Deps: H5.)
- **H8 — Gate-day polish: markdown + model visibility.** Sections: C.1
  web/, B.6 rules 7+8, NATES_VISION, garden/notes/gate-day-feedback.md +
  scorer-evolution.md (gate-day findings). Deliver: (1) MARKDOWN
  rendering of assistant content in the chat pane — sanitized subset
  (headings, bold/italic, lists, tables, fenced code with monospace
  styling; NO raw HTML passthrough — render as text; theme-token
  styled); user messages stay plain text; (2) the ACTIVE MODEL visible
  in the thread header/top bar — read from the thread's RESOLVED model
  (the A-020 floor-routed slug when MODEL_INTELLIGENCE_FLOOR is set,
  else the static config; per-thread selector remains M2) so H8's
  display stays truthful the day H9 lands; (3) KEYWORDS MANDATE (SPEC v2.7, D.2 050): update the C.6
  agent-instruction string (2-5 keywords per save) and make /remember
  generate label AND keywords in its one short completion. Verification
  per B.6 rules 7+8 including a rendering screenshot of a markdown-heavy
  reply (table + code block) at desktop and 390px, an SOP step confirming
  raw-HTML input renders inert, and a scripted check that a /remember
  memory lands with non-empty keywords.
  Nodes: P2, P3. (Deps: H5. H6 precedes by board order; the H5 human-use
  hold gates J only.)

- **H7 — Envelope v1.12 & loop controls.** Sections: C.7 (as amended), ADR-014
  (M1 subset). Deliver: daemon behavior + envelope models for run.started,
  run.cancel (confirmed abort, work preserved), prompt.queued
  (queue-to-turn-boundary), thread.snapshot (hydration on (re)connect),
  run.usage, gate.dismiss; stop_reason on run.done; typed run.delta union;
  gate.open kind field; reserved-type passthrough (forward/ignore unknown
  types). Tests: cancel mid-stream is confirmed by run.done(cancelled);
  reconnect hydrates from snapshot only; a queued prompt fires exactly once
  after turn end. Nodes: P3. (Deps: H1. Runs BEFORE H4 — the UI consumes
  every one of these.)

**Closing**
- **I1 — Integration & AC dry run.** (Also re-executes every UI packet's
  SOP per B.6 rule 8.) Sections: C.8. Deliver: compose
  end-to-end; walk all seven criteria as a BUILDER (not judge); fix gaps;
  seed demo memories; verification/README pointing a judge at everything.
  (Deps: all S*, H*.)
- **M1C — M1 closing report.** The PLAN §3 rule 5 NOBODY duty, made a
  packet so the board never dead-ends: write the M1 closing report
  (reports/, template §4) summarizing the milestone arc P0→J with the
  superseding verdict; update BOARD notes; build NOTHING. (Deps: J plus the
  recorded post-J owner gate; D3 is not an M1 closure dependency.)
- **M2A — Spend ledger core (wave 1).** Sections: ADR-024 entire (read
  its Motivation first), A-020(e)/A-021 usage path, C.2 migration
  discipline. Deliver: spine migration for `spend_event` (receipt-line
  normal form, ULID event_uid, all lineage columns); broker-seam
  synchronous llm.* line writing (chat, embeddings, /remember; purpose
  enum; one line per price class incl. cache read/write, grouped by ref);
  canonical materialized views v_spend_rate, v_thread_cost, v_run_cost,
  v_memory_cost, v_cache_efficiency (minute refresh; derived, never
  authoritative); the receipt-language glossary as SQL COMMENTs.
  Verification: B.6 r7 — double-run view determinism (same rows →
  byte-identical views); a sample row must read aloud as a sentence.
  GCP billing reconciliation + infra.* lines are WAVE 2 — do not build.
  (Deps: J.)
- **M2B — Rack refound + NEO-NOIR identity (wave 1).** Sections: ADR-023
  clauses 1-2 + resize law (read the Motivation), ADR-018 clause 7
  (themes), NATES_VISION §8 + §16-17. Deliver: the rack grid runtime
  (CSS grid, drag/resize/dock within manifest-declared grid-unit bounds,
  ResizeObserver events); every existing surface (thread list, chat
  pane, gate, memory panel, header) re-founded as a first-party rack
  module on the plugin read API (event stream, query+as_of stub,
  selection bus) — NO private paths (dogfooding law); NEO-NOIR as the
  default theme tokens; layouts savable/restorable. The plain shirt dies
  here — vision §17 is the acceptance mood. Verification: B.6 r7+r8,
  screenshots desktop + 390×844, an SOP prose pass on whether it FEELS
  like the mock's family. (Deps: J. Excludes M2C.)
- **M2D — Durable transcripts, capture-only (wave 1).** Sections: the
  m2-planning-agenda item-4 resolution, C.7 (M1 persistence law stands),
  ADR-016. Deliver: the daemon appends every message + event (including
  model_change) to a local per-thread append-only jsonl as it happens;
  restart-proof capture; explicitly NOT serving (thread reload behavior
  unchanged — serving arrives with the M3 session table, backfilled from
  these files). Transcripts never ride anything with a git remote.
  (Deps: J.)
- **M2E — Hybrid candidate retrieval (wave 1).** Sections: D.2 050(1),
  C.3 candidate pool, ADR-005. Deliver: Postgres FTS (tsvector over
  label+body+keywords, indexed); inject/prepare candidate pool becomes
  vector top-50 ∪ FTS top-N (N a journaled DECISIONS constant); scorer
  unchanged — it ranks the union; injection_event rows note pool
  provenance in features meta for future replay. Verification: a
  planted exact-keyword/weak-embedding memory MUST reach the gate.
  (Deps: J.)
- **M2C — Palace Vitals + the spend strip (wave 1).** Sections: ADR-009
  item 5, v2.17 presentation law (D.2 060), ADR-024 views, ADR-023.
  Deliver: the Ableton-style bottom strip as a first-party rack plugin —
  lanes (total, by purpose, by model) over v_spend_rate, hover scrub,
  lane click = selection focus, collapsible; Vitals counters (lifecycle
  rates, palace counts, queue depth placeholder). Dollar-true from
  ledger rows only. Verification: B.6 r7+r8; every lane traces to a
  canonical view. (Deps: M2A, M2B. Excludes M2B.)
- **M2G — Per-message re-scoring + disposition locks (wave 2).**
  Sections: ADR-005 per-message re-scoring block (v2.31/v2.33 — read the
  MOTIVATION: gate once, then autonomous; locks not boosts), C.3/C.5,
  A-016 ordering. Deliver: every post-first message re-scores against the
  live palace (prepare per message, as-of its own snapshot); binding
  selection = τ + budget (k is display config); human-confirmed/pinned
  never auto-demoted; human-removed thread-excluded until re-added; every
  autonomous entry/exit an injection_event (passive keeps per C5); panel
  updates ambiently — NO modals after the first gate. (Deps: J.)
- **M2F — The Chrysopoeia learner v1 (wave 2).** Sections: ADR-005
  learning scope + v2.32 scoreboard + v2.34 authority (read all three
  motivations), notes/m2-planning-agenda methodology. Deliver: spine-side
  batch exact re-fit (pairwise ranking over the hygiene-filtered,
  actor-classed log; weights ≥0 sum 1 as constraints; b_m shrinkage);
  time-split BINARY replay (fewest disagreements with recorded
  dispositions wins; wrong/stale excluded; cheaper-at-tie; margin
  config); minimum-dispositions floor (config) before first challenger;
  winner → PROPOSED scorer_config version, owner activates (early era);
  /retrain manual trigger + optional nightly schedule; every version
  reproducible from the log alone. (Deps: J.)
- **M2H — Extraction + approval queue + thread-end card (wave 2).**
  Sections: ADR-021 clause 4 with the v2.34 trigger, v2.25 unified queue
  law (birthplace routing, passive approval on literal visibility),
  ADR-022 kinships. Deliver: extraction on archive/close (+ idle
  fallback over M2D transcripts; ≤5 atomic keyworded candidates,
  verdicts-at-birth, queue-aware dedup); candidate status invisible to
  injection/search; the rack-module thread-end card (final post up top,
  approve/deny list beneath; viewport-visible passive approval;
  contradictions and collapsed groups never passive-approve); rejected →
  tombstoned-as-rejected + logged. (Deps: M2B, M2D. Excludes M2J, M2K,
  M2C.)
- **M2I — Seed ingestion (wave 2).** Sections: ADR-019 clause 4, v2.25
  queue law (grouped batches = explicit action), ADR-022 splitting law.
  Deliver: markdown upload → splitter pass → atomic candidates through
  the standard create/dedup pipeline → per-document batch in the Palace
  queue module. (Deps: M2H.)
- **M2J — Parameter registry + model device (wave 2).** Sections:
  ADR-023 clauses 3 AND 5 (read the motivations: every knob binds a real
  parameter; GLOBAL|CURRENT scope), C.5, v2.26 resolution points. Deliver: the typed parameter
  registry (descriptors, journaled A-016 change events, as_of-scrubbable);
  per-thread model params (temperature, top_p, top_k, max_tokens, effort)
  wired to broker requests; the MODEL DEVICE reference control plugin —
  resolved slug display + registry-bound knobs + the per-thread selector
  knob riding the resolve_named seam. (Deps: M2B. Excludes M2H, M2K, M2C.)
- **M2K — Memory Graph + hyperparameter console + contribution bars
  (wave 2).** Sections: ADR-009 items 3-4 (authoritative encodings),
  Invariant 6, ADR-023 incl. clause 5 scope law (console = the first
  law-bound control plugin; graph and bars ship with GLOBAL|CURRENT
  toggles, and the spend strip adopts its toggle here). The console
  ships the v2.42 READ+CONTROL layers incl. the accuracy series AND
  the v2.43 SCORE GRAPH (two series, annotated) with normalized knob
  display, docked COMPACT in the factory set; simulate + audition are
  M2P's, not yours.
  Deliver: the force-directed Memory Graph (encodings per law; CAS-safe
  edit on click); the console (τ, top_k, budget, half-lives, weights —
  every change INSERTs a version; displays PROPOSED learner versions for
  one-tap activation); per-feature contribution bars on every gate/panel
  card. (Deps: M2B. Excludes M2H, M2J, M2C.)
- **M2L — Citation heuristic v1 (wave 2).** Sections: ADR-005 citation
  law (n-gram v1 is decided; OQ-5 is only the later LLM-judge upgrade).
  Deliver: n-gram detection of injected-memory reuse in agent output;
  cited outcomes logged (increments f_freq numerator); per-message scope
  rides M2G's re-scoring loop. (Deps: M2G.)
- **M2M — Ledger self-audit vs broker (wave 2).** Sections: ADR-024
  sourcing (v2.34 scope). Deliver: scheduled job reconciling llm.* ledger
  sums against the broker's usage/credits API; drift alert into Vitals;
  GCP billing-export ingestion explicitly NOT built (deferred owner-gated
  setup). (Deps: M2A.)
- **M2N — Owner DB lifecycle hardening (wave 2).** Sections: ADR-019
  v0.1 positioning (v2.35 — read the motivation: the owner's data is
  real now; this is operational durability, NOT a multi-user refactor),
  ADR-003, C.2 migration discipline. Deliver: pin the local pgvector
  image to a digest (no floating pg16 tag); `nocturne backup` /
  `nocturne restore` (v2.49 INFORMED RESTORE: side-by-side into a fresh
  volume, verify there, former volume kept as a rollback generation;
  before switching, present the ROLLBACK MANIFEST — the named diff list
  of memories lost / edits reverted / pins undone — and switch only on
  explicit confirmation; a failed restore never touches the live
  palace) / `nocturne doctor`; an automatic BACKUP RECEIPT
  before every migration (local and owner-cloud); an Alembic advisory
  lock (no two concurrent migrations); historical upgrade-matrix tests
  (every supported revision → head, not just empty → head); a config-file
  upgrader (versioned config migrates, never merely errors); retain N
  backup generations (config). RESOURCE WATCH (owner meta-review,
  2026-08-02): `nocturne doctor` also checks free disk, journal size,
  and db size, and WARNS BEFORE the disk fills (the journal's hard-stop
  must only ever follow an ignored warning); Vitals gains a RESOURCES
  gauge (daemon RSS + uptime, disk free, db size — watchable, never
  demanding); add a rule-7 soak check: the daemon under scripted load
  for an extended run must hold memory within a stated bound.
  Explicitly NOT in scope: per-install
  cloud targets, fleet lifecycle, expand/contract discipline for
  strangers — see the positioning clause. (Deps: J.)
- **M2O — Fixture isolation + accounting fail-open (wave 2).** Sections:
  B.6 rules 10-11 (v2.38 — read the motivation: the owner transmitted to
  a leftover fixture daemon unknowingly, and a failed spend receipt
  killed a real turn). Deliver: (1) scenario/fixture servers bind a
  distinct non-default port, render a full-surface FIXTURE banner,
  terminate with their launching session, and all SOP tooling uses an
  isolated browser profile (fixture threads never enter the owner's
  catalog; provide a one-shot cleanup for already-polluted catalogs);
  (2) agent_runtime spend-receipt failure FAILS OPEN — turn completes,
  basis=estimated ledger line queued for reconciliation, loud drift
  surface in Vitals; regression: a turn with a dead ledger MUST still
  answer. (Deps: J.)
- **M2P — Injection Console: audition + what-if (wave 2).** Sections:
  ADR-009 item 4 v2.42 (read the four layers and their motivations),
  ADR-023 clauses 3+5, v2.34 learner authority. Deliver: SIMULATE
  (v2.43 rename) — INSTANT preview (knob turns re-rank the visible gate,
  unmistakably marked, never persisted) and DEEP (one-click backtest of
  arbitrary values via a new small spine endpoint over M2F's replay
  engine; returns the held-out %); FORCE with honest delta + journal
  (the exploration lever; INFORMED-FORCE v2.44: enabled only after deep
  simulate scores the exact set — knob changes stale the score and
  disarm force); 2D accuracy-vs-parameter slice curves (the
  Landscape's M3 precursor); and the AUDITION overlay (spine scores the same prepare under
  incumbent AND the PROPOSED version; overlay renders counterfactual
  picks faint/marked on live gates + panel; incumbent alone governs
  injection until the one-tap; pure consumer — verified by diffing
  actual context with overlay on/off). (Deps: M2K, M2F. Excludes the
  SPA cluster.)
- **M2Q — Test motivation sweep + law-coverage report (wave 2).**
  Sections: B.6 rule 12 (v2.45 — read the motivation: a test is law
  made executable and cites its statute). Deliver: (1) the deterministic
  pre-commit checker in BOTH repos (motivation docstring + citation
  token grammar; fails unmotivated tests; baseline file grandfathers
  the existing suite); (2) THE SWEEP: motivate every existing test
  (~144 spine + ~306 harness) by reading it against the law — a test
  that cannot be motivated is either unwritten law (propose the
  completion) or nothing (delete it, journaled); retire the baseline;
  (3) the LAW-COVERAGE REPORT generator: citations → per-section
  defender lists, zero-defender sections highlighted, emitted into
  verification/. May split the sweep across two sessions with an honest
  handoff if context demands. (Deps: J.)
- **M2R — Context Bars, + memory category (wave 2; dropped-ball
  recovery, v2.46).** Sections: ADR-009 item 1 (the Code Puppy port:
  category breakdown, compaction threshold line), P2.2 ("what is
  filling each context window?"), ADR-023 clauses 1+5. Deliver: the
  per-thread context-window visualizer as a rack module — category
  breakdown of the live context (system, history, MEMORY BLOCK as its
  own category, tools) against the model's true context length (the
  resolved route's, per A-026), threshold line included; GLOBAL|CURRENT
  scope; compact by default beside the spend strip. ALSO (v2.48): sweep
  ALL existing customer-facing surfaces for garden leaks — ADR/packet/
  amendment ids, build jargon in labels, errors, command output — and
  rephrase per THE GARDEN NEVER SHOWS + vision §18. (Deps: M2B.
  Excludes the SPA cluster.)
- **M2S — Rung-2 first-class startup (owner spin friction, 2026-08-05).**
  Sections: ADR-019 (four commands + capability ladder; read the
  motivation — the OWNER's rung has no product-grade startup: today it
  requires sourcing .env and knowing `uv run harness dev`), the
  garden-never-shows law. Deliver: `nocturne init --remote <spine-url>`
  (prompts for the bearer; OpenRouter key from env or prompt; writes the
  same ~/.nocturne config home with a REMOTE palace mode); `nocturne up`
  with a remote config starts DAEMON ONLY (no local containers) and
  opens the browser; `nocturne doctor` remote-aware (spine health +
  journal + disk; local-palace checks skipped with a plain sentence);
  migrate the owner's .env setup with one command or clear doc line.
  Acceptance: a cold terminal reaches the rack in TWO commands on
  either rung. (Deps: all M2 packets.)
- **M2T — Owner-cloud credential alignment + M2 deploy (grant D.2 094,
  single-use).** Sections: v2.51 (read the motivation: the deployer met
  a hand-built foundation and rightly refused custody it couldn't
  verify), M2N receipts, the deploy runbook. Deliver, in order: (1)
  Cloud SQL on-demand backup receipt, verified SUCCESSFUL; (2) the
  one-time credential reset — fresh database password, user updated,
  spine-database-url secret rewritten, values never printed or logged;
  (3) the packaged image built and pushed; (4) service rolled forward;
  (5) migrations 0002→0009 via the classifier; (6) authenticated remote
  verification (health + typed round trip + one M2-surface probe, e.g.
  /v1/vitals). ALSO: fix the cascade defect (migrations must not be
  declared incompatible before alembic_version is read) and apply the
  v2.52 language calibration: NOOP and standard ops vocabulary STAY;
  fix only the remedy-free refusal sentences (each states situation +
  next action plainly) and any invented/garden jargon. The
  grant is consumed by this packet, success or failure. (Deps: all M2
  packets.)
- **M2X — M2 gate day (HUMAN).** Never agent-claimed. The owner uses
  the assembled M2 product for real: re-scoring under every message,
  thread-end flashcards, seed ingestion, the strip, Context Bars, the
  Memory Graph, the Injection Console, /model, backup/restore. The
  SCOUT precedes (B.6 rule 9) and DERIVES the M2 closing checklist
  (none exists — per the missing-checklist clause), burning down
  mechanics so the owner's session is taste and authentic signal. The
  owner's dispositions during the spin count toward the learner's
  25-signal floor. Hold clears on the owner's word; gates M2J.
- **M2J — M2 judge.** Fresh CLAUDE CODE session (B.6 independence —
  every M2 builder was Codex). Judges the assembled wave against the
  M2 milestone line + all wave charges; re-executes SOPs per rule 8;
  proposes ADR status normalizations; verdict to
  harness/verification/m2/VERDICT.md. (Deps: M2X hold cleared.)
- **J — Judge.** Sections: B.6, C.8, C.9, plus garden/AMENDMENTS.md
  (enacted amendments are law) — nothing else. Fresh session, different
  model than the builders (Codex if built by Claude Code). Execute J0–J8
  with browser automation for screenshots; produce
  `verification/m1/VERDICT.md`. Any FAIL → set FAILED_JUDGMENT on the
  packets the verdict implicates (or I1 if diffuse); a FIXER inherits the
  verdict. ADDITIONAL DUTY (owner, 2026-07-22): ADR STATUS NORMALIZATION —
  propose (in the verdict, for the human gate to enact) status updates
  aligning each ADR's Status line with built-and-verified reality (e.g.
  ADR-004 PROPOSED though its unit/CAS law shipped in the S-packets);
  COMPLETION authority per AMENDMENTS.md, evidence required per B.6.

## 6. CLAUDE.md / AGENTS.md template (both product repos)

```
# Ground rules (read every session)
1. You are one runner in a relay governed by ../garden/PLAN.md — run its
   Boot Sequence before anything else.
2. The constitution is docs/SPEC.md (v2.6): sections 1 -> 2 -> B -> C; read
   fully the sections your packet names.
3. You are in Milestone M1 unless your charge says otherwise. Feature
   ledger (SPEC B.4) applies: FORBIDDEN means do not build, stub, or
   "prepare for" — flag instead (garden/FLAGS.md).
4. Every non-dictated decision -> DECISIONS.md, citing a Problem Tree node
   (SPEC §2). Features that cannot name their problem do not get built.
5. Defects -> Blight Protocol (SPEC 2.1): deepest containing node;
   escalate ancestors/contracts via FLAG.
6. Contracts are literal: DDL C.2, API C.4, envelope C.7, invariants 1.3.
   A contract gap or self-contradiction is a COMPLETION (PLAN §2): enact
   it in garden/AMENDMENTS.md and proceed — never guess silently, never
   stall on what qualifies.
7. Done means judged (SPEC B.6): leave experiential + traced evidence.
8. One packet per session. Handoff ritual, then stop.
9. Invariant 14 is the product's soul: LEAST ATTENTION. Walls, not
   questions. Nothing you build may demand the human's attention except
   judge-released returns and true boundary crossings; everything you
   build must be watchable without demanding. When a design choice trades
   your convenience against the human's attention, attention wins.
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
  - after D1 — review the relay handoff and its redacted cloud evidence.
    Agents have the narrow D1 grant above; billing, budget, D2 deployment,
    destructive recovery, and any authority expansion remain yours.
  - after H5 — use the gate yourself for a day of real prompts; the gate
    is the product's soul and no judge substitutes for your hands.
  - after J — read VERDICT.md beside its screenshots; only then is M1
    done, and M2 planning opens (SPEC B.1; M3 re-plans too).
- Otherwise your between-session review is: the diff, the DECISIONS.md
  entries, the handoff report, and any new AMENDMENTS.md entries — an
  amendment stands unless you veto it there (append the veto; a FIXER
  inherits it). Supervise the relay closely through S2; loosen as the
  reports earn it.
