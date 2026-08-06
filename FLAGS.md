[F001] [P0] [P1.4] — C.2 promises principal-scoped active-label uniqueness in the `label` comment, but the authoritative DDL defines no such partial unique index. P0 cannot choose whether prose or executable DDL wins; minimally add the exact index DDL or remove the promise, disturbing the C.2 persistence contract.
[F002] [P0] [P1.4] — C.4 says a caller may retry memory creation with `force=true`, but the exact request body and route define no location for `force`. P0 cannot invent a cross-repository field; minimally add it to the exact body or route query and define its default, disturbing the C.4 API contract.
[F003] [P0] [P1.3] — C.2 requires `origin_machine_id` on every memory revision, while the C.4 create and PATCH bodies provide no origin and no normative derivation rule. P0 cannot choose an attribution source; minimally add the field or state an exact server-side source rule, disturbing C.2 and C.4.
[F004] [P0] [P1.2.1b] — C.4 says committing a `wrong` removal additionally returns the memory unit, but the exact commit response contains only `final_block`. P0 cannot manufacture a response shape; minimally add the unit field and schema or remove the return promise, disturbing C.4 and the C.6 edit flow.
[F005] [P0] [P1.4] — C.4 calls `GET /v1/memories` paged but defines neither page inputs nor a response envelope; its PATCH success/conflict unit shapes are also undefined. P0 cannot make matching Spine and Harness contracts from that text; minimally specify pagination and the exact unit/409 bodies, disturbing C.4.

---
[RESOLUTION 2026-07-17 — human gate] F001–F005 resolved by SPEC v1.5 (D.2
entry 028), amended in the workspace master `garden_v1/harness-memory-spec.md`.
F001 → C.2 gains `memory_unit_active_label` partial unique index
(principal_id, label) WHERE status='active'; create/PATCH label collision →
409 {label_conflict}; tombstone/quarantine frees the label. F002 →
`force?: bool = false` on POST /v1/memories; overrides ONLY the 0.80–0.92
similar band, never the ≥0.92 duplicate 409. F003 → `machine_id` added to
create and PATCH bodies; spine writes it verbatim to
memory_revision.origin_machine_id. F004 → commit response gains
`wrong_removed: [MemoryUnit]`. F005 → shared MemoryUnit shape defined (C.2
row minus embedding); GET list takes limit/offset and returns
{items,total,limit,offset} ordered updated_at DESC, memory_id ASC; PATCH →
200 MemoryUnit | 409 {conflict: MemoryUnit}. Also: create returns
{created: MemoryUnit}; MemoryCard.score = cosine similarity in dedup and
/v1/search responses. P0 reset to TODO — the next P0 session must refresh
the frozen docs/SPEC.md copies in both product repos, update the v1.4
references in CLAUDE.md/AGENTS.md to v1.5, align the C.4 501-stub routes and
harness client stubs with the amended bodies, and re-verify before DONE
(see reports/000-P0.md notes).

[F006] [H5 SCOUT] [P1.2.1a–b] — The live gate committed `removed:not_relevant`, `removed:wrong`, and `removed:never`, yet the same turn's answer still used the unique chartreuse body removed at the gate; visible reasoning said it would search memory, although the captured surface did not include the tool-result body and therefore does not prove the internal leak path. Harness also discarded C.4's `wrong_removed` result instead of opening the required edit/expire flow. A completion cannot safely choose between a turn-scoped exclusion set, a search contract change, or another prompt/tool boundary fix. Minimally make every committed removal bind all memory supplied to that turn, add a trace-complete regression, and route `wrong_removed` into the edit/expire flow, disturbing the H3/H5 capability boundary and C.6 lifecycle.

[F007] [H5 SCOUT] [P1.2.1b, P1.4] — The required third `never` cannot be expressed in the shipped gate: two kills moved the fixture from scores 0.721 → 0.571 → 0.421, below the injected threshold, and near-miss rows expose add-back only. Before exact Scout cleanup, the unit therefore remained active at `never_kills=2`, contradicting C.3 and C.8 AC4. Minimally provide a contract-valid path to apply the third kill without weakening event-class validation, which disturbs gate actions and/or scorer eligibility rather than filling silence.

[F008] [H5 SCOUT] [P1.1, P1.2.1] — A project-scoped save correctly found no active project context, then the agent retried globally despite the user's explicit “do not save it globally” instruction and persisted the unit. Full project UX may be later work, but silently broadening persistence scope is not. Minimally forbid project-to-global fallback without explicit user confirmation and surface the missing-context result, disturbing H3 tool orchestration but requiring no H6 panel or M3 movement feature.

[F009] [H5 SCOUT] [P0] — Closing-checklist Tier 3.12 requires daemon + browser restart to preserve the thread list and history because “spine is source of truth,” but live restart preserved only the browser-local catalog and lost the selected transcript, exactly matching C.7's explicit M1 law that the daemon provides no thread persistence. The owner must choose whether the checklist is ahead of its milestone or C.7 is wrong; minimally align the closing expectation and milestone ownership without mislabeling the conforming H5 implementation as broken, disturbing accepted session durability scope.

[F010] [H5 SCOUT] [P1.2.1c] — At a live Chrome DevTools phone viewport of 390×844 at 100%, the first-turn gate retained a desktop-width inner layout and overflowed horizontally; only a narrow left slice of the cards was readable. B.6 rule 5 and C.9 J8 require the gate to be readable with full bodies visible and chat usable at that viewport. Minimally constrain the dialog, cards, and action area to the viewport, eliminate page-level horizontal overflow, and retain internal vertical scrolling/sticky actions; this is a local H5 rendering defect and must not alter scorer or memory contracts.

RESOLUTIONS (owner consult, 2026-07-27, via the human gate — SPEC v2.16, D.2 059):
- F006 — REPAIR AS FLAGGED: committed removals bind ALL memory supplied to
  that turn (turn-scoped exclusion + trace-complete regression); wrong_removed
  routes into the edit/expire flow. No law change needed; C.6 already promises
  both.
- F007 — RESOLVED BY AMENDMENT (v2.16): near-miss rows gain "never show
  this", identical semantics/kill counter to removed:never. Implement in the
  gate; C.8 AC4 is now satisfiable.
- F008 — RESOLVED BY AMENDMENT (v2.16): no silent scope broadening — surface
  missing-context, require explicit user confirmation before any global
  fallback.
- F009 — CHECKLIST CORRECTED (owner: nothing existed to destroy; C.7
  conformance confirmed). No product change; durable sessions remain on the
  M2 planning agenda.
- F010 — REPAIR AS FLAGGED: constrain gate dialog/cards/actions to the
  390×844 viewport, no page-level horizontal overflow, internal scrolling +
  sticky actions. Local rendering fix only.
FIXER inherits all five; re-run only the affected SOP slices (1, 2, 8, 12,
responsive repeat) before the owner's personal gate session.

[F011] [H5 FIXER] [P1.2.1b, P4] — F006–F010 now pass against the repaired
Harness and Spine source, but live Cloud Run still serves pre-fix Spine image
`e0cf50d50283cd2c4f800272b832b8166e299cab`. PLAN §5 says H5 and later Harness
work use deployed Spine, while the only current grant to mutate that service is
D1's packet-specific authority; H5 cannot manufacture wider cloud authority as
a COMPLETION. Minimally, the owner either performs the deployment or grants a
fresh relay one forward-only update of the existing
`n8-memory-palace-spine` service from immutable Spine commit `d41b286`, followed
by remote F007 and Harness smoke verification. The resolution must preserve
the existing project, region, service, SQL attachment, runtime identity,
secrets, IAM, max scale, traffic posture, and billing breaker, with no database
migration, deletes, broad IAM, secret rotation, billing change, or new cloud
resource. This disturbs only the D1 cloud-mutation boundary and live runtime;
the repaired product contracts and local evidence do not need redesign.

RESOLUTION F011 (owner, 2026-07-28, via the human gate — D.2 064): GRANTED,
single-use and forward-only: a fresh relay session may perform ONE update of
the existing `n8-memory-palace-spine` Cloud Run service to immutable Spine
commit `d41b286`, preserving the existing project, region, service, SQL
attachment, runtime identity, secrets, IAM, max scale, traffic posture, and
billing breaker — no database migration, deletes, broad IAM, secret
rotation, billing change, or new cloud resource. Required after deploy:
remote verification of the F007 near-miss Never path + Harness smoke, then
the handoff updates H5. The grant is consumed by that session, success or
failure; any further cloud mutation needs a fresh owner grant.

EXECUTION F011 (codex / 2026-07-28 / 86af): CONSUMED — SUCCESS. The sole
authorized mutation updated `n8-memory-palace-spine` to revision
`n8-memory-palace-spine-00004-vs2`, requested by immutable image index
`sha256:dfe9fd5465038e9ac82ca61a49fd93f872afd041dae60b992a5b625fcb694cbb`
from Spine commit `d41b286`; it serves 100% of default traffic. Remote typed
Harness smoke and F007 passed: three Never decisions produced injected,
injected, then near_miss lanes, the third quarantined the fixture, a fourth
prepare omitted it, and exact-ID cleanup tombstoned it. The protected-state
comparison matched before and after, including SQL, identity, secrets, IAM,
scale, traffic/`d1v`, Artifact Registry immutability, and the billing breaker.
Evidence: Spine `verification/h5/f011-2026-07-28/deployment.json`; Harness
`verification/h5/f011-2026-07-28/`. Any further cloud mutation needs a fresh
owner grant. The HUMAN USE HOLD remains and gates J only.

[F012] [I1 FIXER] [P3, P4] — C.9 J1 requires one thread to send `hello` on
the default model and then "Switch the thread to an OpenRouter model string;
one exchange." The shipped M1 default is already OpenRouter; A-020(b)/A-021
require a thread's first model resolution to remain stable for that thread's
daemon lifetime; owner decision D.2 061 and PLAN H8 explicitly defer the
per-thread model selector to M2. Current C.7 has no model-set message and the
M1 UI only displays the resolved model. The named evidence therefore cannot
be produced honestly: two daemon starts or two threads do not prove a
same-thread switch; adding the selector pulls an M2 control and parameter-
registry behavior into M1; treating the existing default OpenRouter exchange
as both stages changes explicit judge law. COMPLETION cannot choose among
those outcomes without reversing standing law or changing an explicit
sentence. Minimally, the owner must either rewrite J1 to require a second
daemon/new-thread model-string proof, declare the visible default OpenRouter
exchange sufficient, or explicitly move the selector into M1 with an exact
wire/authority/UI contract. After that choice, the FIXER can retain the
hello-specific C.7 trace and J2's fresh-word 409/similar action coupling.

RESOLUTION F012 (owner, 2026-07-31, via the human gate — SPEC v2.26, D.2
069): OPTION THREE, refined — the owner rules that deliberate switching
must exist and drift must not: thread stability becomes changes-only-at-
explicit-journaled-RESOLUTION-POINTS; M1 gains the minimal `/model <slug>`
command (no M2 registry surface pulled forward — the /remember pattern);
C.9 J1 rewritten to test it as originally worded. The I1 repair FIXER
implements /model per C.6 v2.26, produces the J1/J2 evidence (J2 charge
unchanged: the fresh-word restatement must itself reach the 409/similar
path), then J resets to TODO for a FRESH CLAUDE CODE judge per B.6
independence. Rationale: the re-resolution seam is real future surface —
M2's selector knob and M3's cost-optimization role policies switch models
at this exact seam, so J1 tests product, not ceremony.

[F013] [D3] [P4] — ADR-019 fixes the public install contract as
`pipx install nocturne`, but PyPI's normalized `nocturne` distribution name is
already owned by an unrelated scalable-deep-learning project, with published
releases through 0.0.6 and a different maintainer. D3 cannot publish the two
NOCTURNE wheels under that exact install name, and silently choosing another
distribution name would reverse an explicit accepted ADR rather than complete
contract silence. Minimally, the owner must either secure a PyPI ownership
transfer for `nocturne` or amend ADR-019 to a new public distribution/install
name while retaining the `nocturne` console command. This disturbs the public
onboarding identity, quickstart, and both-wheel dependency metadata; it does
not require product-runtime redesign.

RESOLUTION F013 (SPEC v2.30, D.2 073; owner deferral 2026-07-31): current law
uses PyPI distribution `nocturne-ai` while the console command remains
`nocturne`. The owner has deferred D3 until M2 wave 1 is complete and may
supersede that distribution name before D3 is claimed. D3 is reset to TODO
behind M2C/M2D/M2E; the naming question no longer blocks unrelated packets.

OWNER SEQUENCING UPDATE (2026-08-01): the owner confirmed the v2.30 naming
resolution and resumed D3 now, concurrently with the independently claimable
M1C/M2 relays. The public install line is `pipx install nocturne-ai`; the
brand and console command remain `nocturne`. PLAN and BOARD restore D3's
direct dependency on the completed J packet.

[F014] [D3] [P4] — D3's release artifacts, OIDC publication workflows, and
public quickstart now exist, but neither `nocturne-spine` nor `nocturne-ai`
exists on PyPI (both project JSON endpoints returned 404 on 2026-08-01).
Trusted publication of a first release requires an owner-authenticated PyPI
pending-publisher registration; the relay has no PyPI authority and must not
invent or solicit a long-lived upload token. The minimal owner action is to
register these two pending publishers, in dependency order: owner
`Nate0-1999`, repository `nocturne-spine`, workflow `release.yml`, environment
`pypi`, PyPI project `nocturne-spine`; then owner `Nate0-1999`, repository
`nocturne-harness`, workflow `release.yml`, environment `pypi`, PyPI project
`nocturne-ai`. A fresh relay can then publish GitHub release `v0.1.0` for Spine,
wait for PyPI, publish Harness `v0.1.0`, and prove a clean public-index
`pipx install nocturne-ai==0.1.0`. This disturbs only first-release authority;
no product contract or code change is needed. PyPI documents this no-project-
yet flow at https://docs.pypi.org/trusted-publishers/creating-a-project-through-oidc/.

RESOLUTION F014 (owner, 2026-08-02): DEFERRED — publishing waits until the
product reaches a more complete state ("I don't think we are in a state yet
to publish"). The pending-publisher registration remains the owner's task
when he chooses to ship; the release workflows stay ready. Relay: do not
re-raise; D3's shipped artifacts stand.

[F015] [M2N] [P1.2.1b, P4] — M2N requires `nocturne restore`, but ADR-019,
ADR-003, C.2, and D.2 078 do not define whether restoring may replace the live
Palace database, must target an empty database, or restores beside the live
database and switches only after verification. Those choices have materially
different recovery behavior and data-loss risk. PLAN §2 explicitly forbids a
COMPLETION from touching data-loss semantics, so the relay cannot silently
choose one and cannot honestly claim a durable restore path while leaving the
choice to implementation accident. The recommended minimum is a
non-destructive side-by-side restore: create a fresh managed database volume,
restore and verify the artifact there, preserve the former volume as a
rollback generation, then switch the local Palace only after an explicit
owner confirmation; a failed restore never touches the live volume. The owner
may instead authorize an in-place replacement contract with an exact force /
confirmation boundary and mandatory pre-restore backup receipt. Resolution
must also say whether owner-cloud restore belongs to M2N or remains a human
Cloud SQL operation; D.2 078 requires owner-cloud pre-migration backups but
names only one unqualified `nocturne restore` command. This disturbs only the
restore and recovery boundary. Digest pinning, backup creation/retention,
doctor, pre-migration receipts, migration locking, historical upgrades,
config upgrades, Resources Vitals, and the soak proof remain mechanically
defined, but building them ahead of the recovery contract would leave one
supposed lifecycle as two incompatible designs.

RESOLUTION F015 (owner, 2026-08-04 — SPEC v2.49, D.2 092): SIDE-BY-SIDE,
with the ROLLBACK MANIFEST. Restore creates a fresh managed volume,
restores and verifies there, preserves the former volume as a rollback
generation, and switches ONLY after the owner confirms against a named
diff list (memories lost / edits reverted / pins undone / event counts —
computed by diffing the two live volumes). A failed restore never touches
the live palace. In-place replacement is NOT authorized. Owner-cloud
restore remains a human Cloud SQL operation; M2N's cloud duty stays
pre-migration backup receipts only.

[F016] [M2T] [P1.3, P4] — D.2 094's single-use credential-alignment grant was
consumed on failure after a verified on-demand Cloud SQL backup receipt was
persisted but before the database password or `spine-database-url` secret was
changed. `gcloud sql users set-password` rejected the private flags document
because its key was `password` rather than the required `--password`; Harness
fixes and tests that defect, and read-only inspection confirms secret version 1
is still the sole enabled version. M2T cannot retry because D.2 094 explicitly
says the grant is consumed on success or failure. Minimally issue a new exact
single-use grant for the corrected reset and state whether receipt
`01KZA98YYHDRZWTBTRQ7SNVZTS` (SUCCESSFUL on-demand backup 1785977929765) may
satisfy backup-first ordering or whether the retry must create a fresh backup.
This disturbs only owner-cloud mutation authority and M2T/M2X scheduling; no
credential value was printed or logged.

RESOLUTION F016 (owner, 2026-08-05 — SPEC v2.53, D.2 096): fresh single-use
grant issued for the CORRECTED reset, same scope/sequence as D.2 094. Bright
line: every mutation attempt takes its OWN fresh backup receipt — receipt
01KZA98YYHDRZWTBTRQ7SNVZTS does NOT carry forward (nothing wrong with it;
the rule is the rule so no future session has to weigh staleness). M2T
resumes: fresh receipt → corrected reset → image → migrations → verification.
