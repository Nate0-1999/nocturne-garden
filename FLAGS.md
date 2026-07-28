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
