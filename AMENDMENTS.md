# AMENDMENTS.md — enacted contract completions (append-only)

Decide-and-declare log per PLAN.md §2 and SPEC 1.4 (COMPLETION class).
Entries here are LAW, equal to docs/SPEC.md, binding on every later agent
and the judge. The human audits new entries at the between-session review;
an amendment stands unless a veto is appended beneath it (a FIXER packet
then reverts it). Only qualifying completions belong here — PLAN §2 defines
the fence; everything past it is a FLAG. Entry format: PLAN §4 template.

Historical note: F001–F005 predate this mechanism and were resolved by the
human directly in SPEC v1.5 (FLAGS.md resolution, 2026-07-17). Numbering
starts at A-001.

---

[A-001] [S1] [SPEC C.2] [P1.3]
gap: C.2 requires every successful CAS update to write `memory_revision`, but
     does not identify which state that row records or how `updated_at` moves.
law: In C.2's final Rules paragraph, after "all in one transaction", add:
     "A successful cloud-head CAS from revision n to n+1 sets
     `memory_unit.updated_at = now()` and appends exactly one
     `memory_revision` with `revision = n+1`, `parent_uid` equal to the prior
     cloud-head revision's `rev_uid`, and `body` / `label` equal to the
     resulting `memory_unit` values. A failed CAS changes neither table."
why: This makes C.2's stated append-only history and atomic CAS rule executable
     without adding a new field, behavior family, or persistence constraint.

[A-002] [S2] [SPEC C.4] [P1.4]
gap: C.4 requires RFC7807 for errors while also prescribing incompatible exact
     JSON bodies for the memory endpoints' domain-specific 409 responses.
law: Replace C.4's opening error sentence with: "Unless a route specifies an
     exact error body below, errors use RFC7807 JSON. The `label_conflict`,
     `duplicate_of`, and `conflict` 409 bodies specified by POST/PATCH
     `/v1/memories` are exact `application/json` responses and are the only
     C.4 exceptions."
why: This preserves both explicit requirements, including the exact 409 schemas
     already frozen by P0, by narrowing the blanket rule only where it conflicts.

[A-003] [S2] [SPEC C.4 POST /v1/memories] [P1.4]
gap: C.4 defines dedup bands but not deterministic selection, similar-list
     membership, ordering, or the exact threshold boundaries.
law: After POST `/v1/memories`' dedup behavior, add: "Dedup comparisons include
     only ACTIVE units with the same principal. `duplicate_of` is the unit with
     greatest cosine similarity, breaking equal-score ties by `memory_id` ASC.
     `similar` contains every such unit whose score satisfies
     `dedup_sim <= score < dedup_dup`, ordered by score DESC then `memory_id`
     ASC; M1 applies no additional result cap. The configured thresholds are
     inclusive at `dedup_sim` and `dedup_dup` as those inequalities state."
why: This completes the existing two-band behavior with stable, testable output
     and introduces no new filter, score, or caller option.

[A-004] [S2] [SPEC C.4 PATCH /v1/memories/{id}] [P1.1]
gap: C.4 does not say how PATCH maintains the stored embedding or handles null,
     no-op, missing-ID, stale-versus-label, and reactivation cases.
law: After PATCH `/v1/memories/{id}`' behavior, add: "A mutable property whose
     JSON value is null is treated as omitted. With zero remaining mutable
     properties, return RFC7807 422; for an absent memory ID, return RFC7807
     404. A supplied non-null `body` is embedded before the CAS, and a successful
     CAS writes its `body`, `embedding`, and `embedding_model` atomically. The CAS
     condition is evaluated before active-label uniqueness: a stale revision
     returns `{conflict: MemoryUnit}`. Any successful-revision write whose
     resulting status and label would collide with another ACTIVE unit of that
     principal returns `{label_conflict}`, including reactivation without a
     label change."
why: These rules carry C.2's atomic head/embedding and active-label invariants
     through the already-promised PATCH surface without adding mutation fields.

[A-005] [S2] [SPEC C.4 GET /v1/memories] [P1.1]
gap: C.4 names list filters and paging fields but does not define their matching,
     composition, count boundary, or lower bounds.
law: After GET `/v1/memories`' behavior, add: "Supplied filters are ANDed;
     omitted `project_key` and `status` apply no filter. Trim `q`; blank applies
     no filter, otherwise it is a case-insensitive literal substring match over
     `label` or `body`. `total` is the filtered count before paging. Require
     `1 <= limit <= 200` and `offset >= 0`."
why: This gives the existing panel-list parameters their minimal conventional
     meaning while preserving its stated stable order and response shape.

[A-006] [S2] [SPEC C.2, C.4, C.5] [P1.4]
gap: C.2 and C.5 cap labels by characters and bodies by tokens but do not define
     the M1 tokenizer or the API behavior when either limit is exceeded.
law: After POST/PATCH `/v1/memories`' behavior, add: "For M1, label length is
     measured in Unicode code points and body length is measured with the
     `cl100k_base` tokenizer. On create, and for each non-null replacement value
     supplied to PATCH, require `len(label) <= cfg.label_max` and
     `tokens(body) <= cfg.memory_max_tokens`; a violation returns RFC7807 422
     before embedding or any database write."
why: This makes the existing 64-character/128-token atomic-memory limits
     deterministic across providers without changing either configured bound.

[A-007] [S3] [SPEC C.3] [P1.2]
gap: C.3 leaves the keyword tokenizer, decay clock, budget units and rounding,
     tie breaks, pin ordering, rank meaning, and budget-skipped near misses
     undefined even though they change the exact prepare result.
law: After C.3's scorer rules, add: "For scorer v0, `tokens` are maximal runs
     of Unicode alphanumeric characters after lowercase conversion. `kw`
     removes the exact stopword set `{a, an, and, are, as, at, be, by, for,
     from, has, he, in, is, it, its, of, on, that, the, to, was, were, will,
     with}`; each stored keyword is tokenized by the same rule before union.
     Time and human-edit ages are measured from `thread.snapshot_ts`, in
     elapsed seconds divided by 86400, with negative ages clamped to zero; a
     human edit is a `memory_revision` whose editor is exactly `user`. Semantic
     cosine is clamped to `[0,1]`. A project match requires a non-null thread
     project equal to the memory project; a null memory project scores 0.5.
     Eligible `pin=true` units are fetched outside the non-pinned vector pool
     and top-k limit, scored only to populate the explainability fields, ordered
     by `memory_id` ASC, and injected first regardless of threshold or budget.
     Their `cl100k_base` body-token costs reduce the regular budget to no less
     than zero; pins alone may exceed it. The regular budget is
     `min(budget_tokens, floor(budget_pct * model_context_tokens))`, and each
     regular card costs its body's `cl100k_base` tokens. The non-pinned vector
     pool orders cosine DESC then `memory_id` ASC; scored regular candidates
     order score DESC then `memory_id` ASC. Greedy selection scans that order,
     accepts at most `top_k`, and after an over-budget card continues to the
     next. Near misses are the first `near_miss_k` unselected regular candidates
     in score order, including candidates excluded by threshold, budget, or the
     top-k cap. Rank is one-based in the combined complete order: pins first,
     then all regular candidates in score order; returned lists retain those
     ranks even when intervening candidates are not returned. `kind='pinned'`
     without `pin=true` does not bypass scoring."
why: These are deterministic mechanics for the scorer C.3 already specifies;
     they add no feature, signal, or learning behavior.

[A-008] [S3] [SPEC C.4 POST /v1/inject/prepare] [P1.2]
gap: C.4 promises a frozen revision-backed read although C.2 cannot reconstruct
     historical scoring state, while M1 permits only one prepare per thread;
     it also leaves event membership and injection-counter membership unstated.
law: After POST `/v1/inject/prepare`'s behavior, add: "M1 accepts exactly one
     successful prepare per thread, as required by C.6's one-injection flow. A
     thread row with non-null `snapshot_ts` returns RFC7807 409 on another
     prepare. An existing unstamped row may be stamped only when its principal,
     agent, machine, and project fields exactly match the request; mismatch
     returns RFC7807 409. Prompt embedding completes before the atomic database
     phase. That phase uses one repeatable-read transaction, stamps
     `snapshot_ts` from the database clock, reads and scores the heads visible
     at that boundary, writes events, and updates statistics; a conflict rolls
     back the entire phase. Each returned card writes exactly one
     `injection_event`: injected `pin=true` cards use `shown_as='pinned'`, other
     injected cards use `shown_as='injected'`, and near misses use
     `shown_as='near_miss'`; stored score, six features, and rank equal the
     response and outcome is null. To preserve the frozen card for replay and
     commit without changing C.2's DDL, the event's features JSON additionally
     contains `_memory: {label, body, pin, updated_at}` from the scored
     snapshot; the wire `features` object remains the exact six-field C.4
     shape. `stats.injections` increments once and `stats.last_injected_at` is
     set to `snapshot_ts` for each card in `injected`, including pins, and never
     for a near miss. Those server writes use C.2 CAS, append revisions with
     editor `system:inject`, origin machine from the request, and reason
     `inject/prepare`."
why: The one-shot transaction is the smallest implementation of M1's explicit
     one-injection-per-thread law; event snapshots keep the log replayable, and
     the counter rule follows rather than weakens the standing CAS invariant.

[A-009] [S4] [SPEC C.4 POST /v1/inject/commit] [P1.2.1a–c]
gap: The commit body does not define batch membership, invalid choices, outcome
     transitions and retries, add-back statistics, eventless prepares, or the
     transaction and attribution boundary needed to apply its behavior.
law: After POST `/v1/inject/commit`'s behavior, add: "The event batch is every
     `injection_event` row with the requested `injection_id`. Each `removed`
     memory ID must be distinct and name a batch row whose `shown_as` is
     `injected` or `pinned`; each `added_back` ID must be distinct and name a
     `near_miss` row; the lists must be disjoint. Duplicate, foreign, or
     wrong-class choices return RFC7807 422 with no write. No batch rows plus
     nonempty choices returns RFC7807 404. No batch rows plus both lists empty
     succeeds with the canonical zero-member C.6 block and `wrong_removed=[]`;
     because A-008 persists no batch row for a zero-card prepare, this also
     accepts any unknown injection UUID with empty choices in M1.

     The desired commit outcome is `removed:<reason>` for each removed row,
     `kept` for every untouched injected or pinned row, and `added_back` for
     each selected near miss; an unselected near miss remains NULL. Outcomes
     transition from NULL exactly once. Repeating the same desired outcome is
     idempotent. For retry comparison, `cited` or `mid_thread_removed` is a
     descendant of `kept` when `shown_as` is injected/pinned and of
     `added_back` when `shown_as` is near_miss. Any different non-NULL outcome
     returns RFC7807 409 with no write. An all-near-miss commit with no add-back
     is necessarily a durable no-op under the frozen C.2 schema.

     Only a new NULL-to-outcome transition changes head statistics. Each newly
     removed row increments `stats.removals` once. A new `removed:never` also
     increments `stats.never_kills`, adds that event's scorer-version
     `never_bias_step` to `bias`, and changes an ACTIVE head to quarantined when
     the resulting kill count is at least that version's `quarantine_kills`;
     an already quarantined or tombstoned status is preserved. Each newly
     `added_back` row increments `stats.injections` once and sets
     `stats.last_injected_at` to the commit transaction's database clock. Load
     the two never parameters from the event's `scorer_version` row. Apply all
     new event outcomes, head changes, and revisions atomically; update each
     affected head once through C.2 CAS in memory-ID order with editor
     `system:inject`, the event's `machine_id`, and reason
     `inject/commit:<outcome>`. `wrong_removed` contains the post-stat CURRENT
     MemoryUnit for every requested `removed:wrong`, ordered by event rank.

     Final members are rows whose outcome is `kept`, `added_back`, or `cited`,
     ordered by rank ASC then memory_id ASC. Render them only from the frozen
     event `_memory` payload and `memory_kind`, never from a current head."
why: This completes the promised gate decision on A-008's existing rows, keeps
     retries and counters exactly-once without a forbidden schema rewrite, and
     makes the unavoidable zero-card ambiguity harmless and explicit.

[A-010] [S4] [SPEC C.4 POST /v1/feedback] [P1.2.1d]
gap: The feedback endpoint names two signals but does not define membership,
     legal outcome transitions, retries, or their M1 statistic effects.
law: After POST `/v1/feedback`'s response, add: "Feedback targets the single
     `injection_event` matching both IDs; no match returns RFC7807 404. A new
     signal may transition only `kept` or `added_back` to the signal's literal
     outcome. Repeating that same signal is idempotent `{ok:true}` with no new
     write. Outcome NULL, any `removed:*` outcome, or the other feedback signal
     returns RFC7807 409. `mid_thread_removed` also increments the current
     head's `stats.removals` exactly once through C.2 CAS; its event transition,
     head update, and revision are one transaction using editor
     `system:feedback`, the event's `machine_id`, and reason
     `feedback/mid_thread_removed`. It changes no bias, status, citation, or
     other statistic. In M1, `cited` is event-log-only: it writes the outcome
     but does not change `stats.citations`, write a memory revision, or affect
     scorer v0, whose citation feature remains inert."
why: The existing outcome vocabulary can log both signals exactly once while
     preserving C.3's explicit M1 citation inertness and C.2's CAS invariant.

[A-011] [S4] [SPEC C.6 final_block] [P1.2.1c]
gap: The exact block template does not define member ordering, escaping,
     structural newlines, or the zero-member serialization.
law: After C.6's rendered block template, add: "Join structural lines with LF,
     with no blank separator lines and no terminal LF. Emit attributes in
     `label`, `kind`, `updated` order. In attributes escape `&`, `<`, `>`, and
     `"` as `&amp;`, `&lt;`, `&gt;`, and `&quot;`, and encode tab, LF, and CR as
     `&#9;`, `&#10;`, and `&#13;`. In body text escape only `&`, `<`, and `>`;
     preserve every other character and existing newline. Use frozen event
     values verbatim apart from that escaping. With zero members, return these
     four lines exactly: `<memory_system>`, the two literal preamble lines from
     the template, and `</memory_system>`."
why: Deterministic safe serialization completes the already-fixed XML-shaped
     wire block without adding a field, renderer option, or new behavior family.

[A-012] [S6] [SPEC C.4 POST /v1/search] [P1.1]
gap: C.4 supplies `k=10` and optional `project_key`, while the S6 charge promises
     top-k cosine search, but neither bounds k nor defines omitted-versus-null
     project behavior or the equal-cosine cutoff order.
law: After POST `/v1/search`, add: "Require `1 <= k <= 50`; a violation returns
     RFC7807 422 before embedding. An omitted or JSON-null `project_key` applies
     no project filter. A non-null `project_key` admits only ACTIVE units whose
     `project_key` is NULL or exactly equal to the request value. Results are the
     first k ordered by raw cosine similarity DESC then `memory_id` ASC. Search
     applies no scorer threshold, weights, bias, pin priority, or candidate
     re-ranking; `score` is that raw cosine and `features` / `rank` are null."
why: This completes the existing field, filter, and result contract using C.3's
     M1 pool bound and the established cosine/UUID tie convention, without
     adding a search signal or scorer behavior.

[A-013] [H1] [SPEC C.7] [P3]
gap: C.7 defines the WebSocket envelope but not its frame encoding or the
     deterministic malformed-message rejection that H1 is charged to test.
law: After C.7's M1 type list, add: "Each browser-to-daemon message occupies
     exactly one JSON text frame whose top-level value validates as the C.7
     envelope. A binary frame, invalid JSON, non-object JSON value, or object
     that fails envelope validation is malformed. On the first malformed
     message, the daemon invokes no type handler, closes that `/ws` connection
     with WebSocket code 1008 and reason `invalid C.7 envelope`, and performs no
     further processing on that connection."
why: This makes H1's existing rejection requirement observable and closes the
     binary-frame crash without defining any per-type payload or business flow.

[A-014] [H2] [SPEC C.4 POST /v1/inject/prepare] [P1.1]
gap: C.4 names `model_context_tokens` but does not define its valid domain even
     though the implemented prepare budget requires it to be positive.
law: After POST `/v1/inject/prepare`'s request body, add: "Require
     `model_context_tokens > 0`; a violation returns RFC7807 422 before
     embedding or any database write."
why: This makes the existing implemented/OpenAPI boundary explicit without
     changing prepare behavior, its budget formula, or any completed packet.

[A-015] [H3] [SPEC C.6 save_memory] [P1.4]
gap: C.6 tells the model to call save_memory again with force=true after a
     similar response, but the exact tool signature exposes no force input.
law: Replace `save_memory(label, body, kind, keywords?, project_scoped: bool)`
     with `save_memory(label, body, kind, keywords?, project_scoped: bool,
     force: bool = false)`. Forward force unchanged to C.4 POST /v1/memories.
     The tool never enables or retries force automatically. `true` skips only
     the near-similar band and does not override label or hard-duplicate 409s.
why: This makes C.6's already-required retry executable using C.4's enacted
     force field without adding another behavior family.

[A-016] [H7] [SPEC C.7 v1.12] [P3]
gap: C.7 names the loop messages but does not define their executable payloads,
     acknowledgement/cancellation correlation, queue and snapshot ordering, or
     how an M1 connection identifies the active thread; it also calls
     thread.snapshot D→C while requiring a client request.
law: Complete C.7 v1.12 with the following rules. `run_id` is a
     daemon-generated ULID allocated once when prompt.submit is accepted,
     including when it is queued; `prompt_id` is that inbound envelope's `id`.
     prompt.submit has a non-blank string `prompt` and requires outer
     `thread_id`. These payload members are required: run.started
     `{run_id,prompt_id}`; prompt.queued `{run_id,prompt_id}`; run.cancel
     `{run_id}`; run.delta text/thinking
     `{run_id,kind:"text"|"thinking",text:string}` or event
     `{run_id,kind:"event",event:object}`; run.usage
     `{run_id,requests,input_tokens,output_tokens}`; run.done
     `{run_id,stop_reason,partial}`; gate.open
     `{run_id,kind:"memory_gate",...}`; gate.commit `{run_id,...}`; and
     gate.dismiss `{run_id,...}`. Usage fields are non-negative integer
     cumulative totals for that run and never decrease; C.7's "incremental"
     means that updated cumulative snapshots may be emitted while the run
     advances. `stop_reason` is
     `end_turn|cancelled|error|budget_exceeded`; `partial` is false exactly for
     end_turn and true otherwise. These are minimum object members: additional
     JSON members are allowed and preserved so later gate and event contracts
     can extend them.

     thread.snapshot is bidirectional. A C→D request requires outer
     `thread_id` and payload `{request:true}`. Its D→C response carries
     `{messages,open_gate,active_run}`: `messages` is the daemon's ordered array
     of JSON message objects, including queued prompts and already-produced
     partial work; `open_gate` is null or the current gate.open payload; and
     `active_run` is null or `{run_id,prompt_id,state,usage,queued}`, with state
     `running|waiting_gate|cancelling`, usage in the run.usage shape without a
     second run_id, and queued an ordered array of
     `{run_id,prompt_id,prompt}`. Additional members are allowed and preserved.
     For M1 the daemon keeps one process-local active thread, selected by the
     latest valid thread.snapshot request or prompt.submit. On every WS connect
     it sends exactly one snapshot before live events when that thread exists;
     when none exists it sends none until a request or prompt selects one.
     Requesting an unknown thread returns the empty snapshot. Thread/run state
     survives a socket disconnect for the daemon process lifetime; reconnect
     sends the snapshot only and never replays old deltas or other prior
     events. This local selection is not a session or authorization boundary.

     A prompt received while its thread has a live run is appended once to a
     process-local FIFO and immediately acknowledged by prompt.queued. After
     every terminal run.done, the oldest queued prompt starts once: the prior
     run.done is emitted before its run.started, and the reserved run_id is
     reused. Cancellation applies only to the matching active run. The daemon
     first requests cancellation, awaits model/tool-batch termination and
     records terminal cancelled tool results, preserves all prior
     messages/output, emits gate.dismiss before run.done when a gate was open,
     and emits exactly one run.done(cancelled); no run.delta or run.usage for
     that run follows it. A duplicate cancellation while cancellation is
     pending shares that one confirmation. A stale, unknown, or already-
     terminal run_id produces error `{code:"run_not_active",run_id}` and
     cancels nothing. Queued prompts survive every terminal reason.

     Every daemon-created envelope has a fresh outer ULID `id` and timestamp;
     prompt_id supplies acknowledgement correlation. `type` accepts any
     non-blank string. The required payload validation above applies to known
     M1 behavior types. Reserved names and all other unknown types retain
     arbitrary JSON payloads and pass outer-envelope validation; an endpoint
     with a forward target forwards them unchanged, and an M1 daemon with none
     ignores them without emitting the not-implemented error. A-013 malformed-
     frame handling remains unchanged.
why: This fills only the wire/state details H7 must implement, repairs the
     snapshot request direction in the direction C.7 already promises, and
     uses daemon-lifetime local state rather than inventing sessions,
     authorization, M3 controls, or H4/H5-specific behavior.

[A-017] [H4] [SPEC C.7 thread.snapshot / thread.create] [P2, P3]
gap: H4 must render authoritative snapshot messages and a thread list, while
     C.7 leaves each message object's shape, thread creation/listing, the
     browser's required outer machine_id, and the trigger for recovering a
     backpressured live subscription undefined.
law: In M1, daemon-authored `thread.snapshot.messages` use these minimum JSON
     objects. A user message is `{message_id,run_id,role:"user",content,state}`
     where `message_id` is the run's prompt_id, both IDs are ULIDs, content is
     a string, and state is one of `queued`, `running`, `end_turn`,
     `cancelled`, `error`, or `budget_exceeded`. An assistant message is
     `{message_id,run_id,role:"assistant",content,thinking,events,partial}`
     where `message_id` equals `run_id`, both are ULIDs, content and thinking
     are strings, events is an array of JSON objects, and partial is boolean.
     Additional JSON members are allowed and ignored by clients that do not
     understand them. A matching snapshot replaces the browser's transcript,
     open gate, active run, and usage for that thread; it is never merged with
     cached messages or prior deltas.

     The M1 browser owns a per-browser navigation catalog of
     `{thread_id,title,created_at,updated_at}` in local storage. `thread_id` is
     a browser-generated UUID, timestamps are ISO 8601 strings, and title is
     `New thread` until the first submitted prompt, then the first 48 Unicode
     code points of that prompt after collapsing whitespace, with `…` appended
     exactly when the normalized prompt exceeds 48 code points. `created_at`
     is fixed at catalog insertion; `updated_at` changes on each prompt submit.
     Creating or selecting a catalog entry sends a `thread.snapshot` request
     with payload `{request:true}` for that UUID; the first prompt creates
     daemon state through the existing prompt.submit path. The catalog is only
     local navigation metadata: snapshot remains transcript authority, the
     daemon provides no M1 thread enumeration or persistence, and
     `thread.create` gains no M1 behavior.

     A browser-authored direct-link envelope uses the literal machine_id
     `direct` until it has observed a daemon envelope, then echoes the latest
     non-blank daemon machine_id. The M1 direct daemon does not consult this
     field for identity or authority; future relay targeting supersedes the
     direct-link sentinel. Every browser-authored envelope still receives a
     fresh ULID and timestamp.

     When bounded live delivery or a connection outbox drops a subscriber, the
     daemon closes that WebSocket with code 1013 and reason
     `snapshot resync required`. The browser reconnects, requests its selected
     thread snapshot, and replaces local thread state before consuming later
     live events; it does not poll or replay buffered deltas.
why: This supplies only the render and navigation facts H4 already requires,
     formalizes H7's existing snapshot rows, and avoids inventing server-side
     thread storage, a title model call, sessions, or an authority boundary.

---
[FOLD MARKER 2026-07-20 — human gate, SPEC v2.0] A-001 through A-017 are
folded verbatim into SPEC Part C (each site carries a "[folded from
A-0NN]" marker). This file remains the append-only historical record and
the FIRST home of every future completion: agents keep enacting here;
folds into the spec happen only at human-gate editor passes. Numbering
continues at A-018.

[A-018] [H4] [SPEC C.7 thread.snapshot] [P2, P3]
gap: A-017 makes each matching snapshot a wholesale browser-state replacement,
     but H7 may send both its automatic attach snapshot and the browser's
     requested snapshot. A successfully sent prompt between those responses
     can otherwise lose its only local text before run.started or
     prompt.queued supplies correlation.
law: The M1 browser keeps every successfully sent but unacknowledged
     prompt.submit in a volatile outbox keyed by thread_id and prompt_id, with
     its prompt text. This outbox is separate from the authoritative
     transcript and is not persisted. A matching thread.snapshot still
     replaces the transcript, open gate, active run, and usage wholesale. For
     display only, the browser projects an outbox item whose prompt_id is
     absent from the replacement transcript as one submitting user row.
     run.started or prompt.queued updates an existing matching user row or
     materializes it from the outbox, then removes that item. A snapshot that
     already contains the prompt also removes its outbox item. Reload recovery
     remains snapshot-only; the browser never resubmits an outbox item.
why: This preserves A-017 snapshot authority while preventing an overlapping
     pre-ack snapshot from erasing the only copy of an accepted prompt. It adds
     no replay, persistence, server correlation field, or new behavior family.

[A-019] [H5] [SPEC C.6 steps 1–4; SPEC C.7 gate.*] [P1.2.1a–c]
gap: H5 must block on a concrete browser decision and carry C.4 prepare/commit
     data, but C.6 does not source model_context_tokens or reconcile its
     first-prompt rule with /remember, and A-016 deliberately leaves gate.*
     extension fields, normal resume, rejection, and memory-failure behavior
     undefined.
law: After C.6 step 4, add: "In M1, the one gate belongs to the first ordinary
     chat turn. The daemon-only /remember command neither opens nor consumes
     it. The attempt is claimed when that chat turn begins the injection flow;
     cancellation, model failure, or memory failure does not surprise the user
     with a later gate in the same daemon-lifetime thread. Harness config adds
     positive integer model_context_tokens (environment name
     MODEL_CONTEXT_TOKENS), default 1000000 for the default chat model, and
     sends it unchanged to inject/prepare; a model override must also override
     this value when its context window differs.

     An M1 memory gate.open requires
     {run_id,kind:'memory_gate',injection_id,snapshot_ts,scorer_version,
     injected,near_misses}; injection_id is a UUID, snapshot_ts is an ISO 8601
     timestamp, scorer_version is non-blank, and both card arrays contain the
     exact C.4 scored MemoryCard shape. The six feature values are the raw M1
     feature scores supplied by C.4. H5 renders them truthfully as feature
     scores; weighted contribution bars remain M2 per B.3.

     The matching browser gate.commit requires outer thread_id and
     {run_id,injection_id,removed,added_back}. removed is a unique array of
     exact C.4 {memory_id,reason} entries whose IDs occur in that gate's
     injected array; added_back is a unique array of IDs from that gate's
     near_misses; the two sets are disjoint. The daemon validates run, thread,
     injection, membership, and the single-submit boundary against its open
     gate before resolving the hard pause, and uses its server-held
     injection_id for C.4 commit. Any stale, duplicate, mismatched, or invalid
     decision changes nothing and returns error
     {code:'gate_not_committable',run_id}.

     A valid decision is accepted exactly once. The gate remains open and the
     active run remains waiting_gate while inject/commit is in flight. On a
     successful commit the daemon emits gate.dismiss, returns the active state
     to running, supplies final_block as system-adjacent instructions, and
     only then invokes the model. Cancellation retains A-016 ordering and
     never invokes the model for that run.

     Only a Spine client failure during prepare or commit degrades memory: the
     daemon emits error {code:'memory_unavailable',run_id,
     phase:'prepare'|'commit',message:'Memory is unavailable; continuing
     without injected context.'} and invokes the chat model without a
     final_block. Prepare failure opens no gate; commit failure emits that
     error and gate.dismiss before the memoryless model invocation. This is
     fail-open for chat, not a gate timeout. Validation and programming faults
     retain the ordinary run.done(error) path."
why: This is the smallest executable handshake already promised by C.6 and
     A-016, makes the configured default model's context explicit, preserves
     the hard human pause and typed training signal, and enforces Invariant 9
     without inventing retries, offline cache behavior, M2 contributions, or
     H6 editing UI.

[A-020] [H9] [SPEC C.5; C.6/A-019 model_context_tokens] [P4]
gap: C.5 makes chat model selection static: a cheap development default, a
     flagship override, per-thread manual choice. It is silent on HOW a model
     is chosen when the owner's operating policy is economic, and silent on
     provider routing — so the broker's price-weighted load balancing may
     move a mid-run conversation to a cold host, re-billing the entire
     accumulated prompt prefix at full input price. Nothing in current law
     lets the daemon choose "the cheapest model that is smart enough"
     without a human hand-picking slugs.
law: (a) FLAT INTELLIGENCE FLOOR. Harness config gains optional
     model_intelligence_floor (env MODEL_INTELLIGENCE_FLOOR, positive
     number, default unset). When unset, model selection is unchanged. When
     set, the daemon resolves each new thread's chat model at that thread's
     first run: among the broker's benchmark listing
     (GET /api/v1/benchmarks?source=artificial-analysis), the model with
     the lowest prompt price whose intelligence_index is >= the floor;
     ties break by completion price, then permaslug. The benchmark response
     may be cached for at most 24 hours. Selection is deterministic given
     the cached table, and the daemon logs the chosen slug, its
     intelligence_index, both prices, and the table's fetch timestamp. An
     explicit per-thread model override (existing C.5 law) beats the floor.
     Per-prompt classifier routing (openrouter/auto*) is REJECTED as law:
     model choice must be reproducible and auditable from a numeric table,
     never a third party's opaque per-prompt judgment.
     (b) THREAD-STABLE, CACHE-STICKY ROUTING. A thread's resolved model does
     not change within the daemon lifetime of that thread. Every OpenRouter
     chat request carries session_id equal to the C.7 thread_id, so the
     broker pins the serving provider (and model) from the first turn and
     KV-cache prefix reuse survives the whole run. Broker-side expiry of
     sticky state after inactivity is acceptable. Provider fallbacks remain
     enabled; the daemon never sets allow_fallbacks=false.
     (c) CHEAPEST HOST. Floor-selected requests set provider sort to price.
     Config gains optional provider_quantizations (default unset); when set
     it is forwarded verbatim as the broker's provider.quantizations filter,
     because a host serving an int4 quantization of a floor-qualified model
     silently defeats the floor. The default stays unset so an incomplete
     host table can never empty the provider pool.
     (d) CONTEXT WINDOW FOLLOWS THE MODEL. A-019 requires
     model_context_tokens to match the model in use. When the floor selects
     the model, the daemon sources that model's context length from the
     broker's model listing at selection time and sends it to
     inject/prepare for that thread, in place of the static default. The
     static MODEL_CONTEXT_TOKENS default continues to govern when the floor
     is unset.
     (e) TRUTHFUL ACCOUNTING, NO NEW SURFACE. Broker-reported usage,
     including cached-token and cache-write details, is recorded on the
     existing run usage path. ADR-014 run walls are unchanged. No cost UI
     is built in M1; Palace Vitals gauges remain M2 law.
     (f) FENCES. The floor never applies to embed_model — C.2 fixes the
     1536-dimension space and a changed embedding model corrupts it — and
     the /remember label agent simply follows the thread's resolved model
     per decision 008. Benchmark-endpoint failure, an empty eligible set,
     or a missing context length fails open to the static chat_model
     default (with its static context tokens) and a logged warning — never
     a blocked run and never a question to the human.
why: P4 — one human funds the workforce, and input-token spend on growing
     run histories is the dominant marginal cost, a collapse vector the D2
     circuit breaker can only stop, not economize. A flat benchmark floor
     makes "smart enough, then cheapest" a one-number, auditable policy —
     neither a hand-picked slug nor an unaccountable classifier. And the
     cheapest token is the one already cached: warm-prefix reads bill at
     roughly 0.1–0.5x input price, so within-run provider stickiness
     dominates nominal price differences between hosts; session_id makes
     stickiness deliberate instead of detect-then-stick luck. Verified
     against broker docs 2026-07-27: the benchmarks endpoint returns
     intelligence_index plus per-model pricing; session_id pins model and
     provider from turn one; usage with cached-token details is returned by
     default on every response.

[A-021] [H9] [SPEC C.5; supersedes A-020 clauses (a) and (c)] [P4]
gap: A-020 fixed one economic policy (a flat floor) for one role (chat), and
     a quantization filter no measurement supports — intelligence indices
     are published per model, never per quantization or endpoint, so clause
     (c) guarded against a difference nothing can observe. The relay's later
     roles (search-pool workers, orchestrators, curators, judges) must not
     inherit a bargain-hunting policy fit only for high-volume filtered
     work, and a flat floor cannot follow a drifting frontier.
law: (a-revised) TOKEN-COST POLICY TYPE. Model selection is a POLICY — a
     role-generic type with exactly five M1 values:
       pinned:<model> — the named model verbatim (existing chat_model
         syntax); no benchmark table consulted.
       max — the model with the highest intelligence_index in the cached
         benchmark table; ties break by lower prompt price, then permaslug.
       elbow — the point of diminishing returns on the cost-intelligence
         Pareto frontier, computed deterministically as follows. A model
         is DOMINATED when another model has intelligence_index >= its
         AND prompt price <= its, at least one strictly; the FRONTIER is
         the set of non-dominated models (duplicate price/index pairs
         keep the lexically first permaslug), sorted by
         intelligence_index ascending — along it, price rises with
         intelligence. Map each frontier model to
         x = (intelligence_index - min) / (max - min) and
         y = (log10 prompt_price - min) / (max - min), min and max taken
         over the frontier, so the cheapest, least-intelligent model
         sits at (0,0) and the priciest, most-intelligent at (1,1). The
         CHORD is the diagonal y = x; its slope is the frontier's own
         average exchange rate of log-price for intelligence. Each
         model's SIGNED OFFSET is x - y: positive means its log price
         sits below the chord — a better-than-average trade. The elbow
         is the model with the greatest positive offset, the deepest
         bargain relative to the frontier's average rate; ties break by
         lower prompt price, then permaslug; if no model has a positive
         offset, elbow falls back to max. This is a slope rule whose
         threshold the table computes for itself: the offset grows
         wherever intelligence is being bought below the frontier's
         average log-price rate and shrinks wherever it is bought above
         that rate, so there is no hand-set slope constant, and dense
         pockets of similar models — point spacing, not value — cannot
         move the answer the way they whipsaw a local-derivative rule.
         Worked example (index, $/M prompt): A(20, 0.10) B(35, 0.30)
         C(55, 1.00) D(60, 8.00), none dominated. log10 prices: -1,
         -0.523, 0, 0.903. x: 0, .375, .875, 1. y: 0, .251, .525, 1.
         Offsets x - y: 0, .124, .350, 0 → elbow = C. D buys five more
         index points at eight times the price; C is the last
         better-than-average trade.
         [Clause revised by the owner's pen 2026-07-28, pre-claim, in
         the audit window: terms defined for agents without the design
         session's context, and the original unsigned perpendicular-
         distance rule — which on a concave frontier could select a
         worse-than-average point — replaced by signed offset, an
         identical ranking on the intended side of the chord.]
       slope:<λ> — pay for intelligence until its marginal price
         exceeds λ, a positive number in prompt-dollars per M tokens
         per intelligence-index point (raw units, never normalized).
         Compute the frontier exactly as defined under elbow, sorted by
         intelligence_index ascending, in raw (intelligence_index,
         prompt price) space — no logs, no normalization. Take its
         LOWER CONVEX HULL: the chain of frontier models running from
         the cheapest to the most intelligent such that every frontier
         model lies on or above every chain segment — the taut string
         stretched beneath the points. Hull segment slopes
         (delta price / delta intelligence_index) are non-decreasing by
         construction, so the slope crosses any λ at most once; models
         the hull skips are strictly worse buys than the hull vertices
         bracketing them. The selection is the upper endpoint of the
         last hull segment whose slope is <= λ — the most intelligent
         model reachable while every marginal index point still costs
         <= λ. If the first segment's slope already exceeds λ, select
         the hull's cheapest vertex; if no segment's slope exceeds λ,
         select the hull's most intelligent vertex. Collinear frontier
         models on a hull segment count as hull vertices; remaining
         ties break by cheaper prompt price, then permaslug.
         Worked example: frontier (54, $0.90) (55, $1.00) (55.3, $1.60)
         (58, $1.70) (65, $9.00). Adjacent secant slopes 0.10, 2.00,
         0.04, 1.04 whipsaw around any threshold, but the lower hull is
         (54, $0.90)-(55, $1.00)-(58, $1.70)-(65, $9.00) with slopes
         0.10, 0.23, 1.04 — monotone — so at λ=0.5 the crossing is
         unique and the selection is (58, $1.70). Design intent for M3,
         non-binding, nothing built now: the frontier models a hull
         segment skips — the dense pocket around the selection — are
         natural diversity shards for Symphony search pools.
         [slope:<λ> added by the owner's pen 2026-07-28, pre-claim:
         the owner's ratified marginal-price rule; the hull is part of
         the law so the λ-crossing is unique on a discrete table.]
       [Calibration record, owner's pen 2026-07-28. On the owner's
         reference chart of the budget tier (frontier, prices read
         approximately: 44.4/$0.09, 51.2/$0.20, 55.0/$0.53, 57.1/$0.60,
         60.7/$1.15), elbow computes signed offsets 0, +.10, -.05,
         +.03, 0 and selects 51.2 — matching the owner's independently
         stated "52 today" with zero configuration. The owner's
         underlying intent, recorded for whoever tunes role policies:
         buy the COMMODITY TIER — the capability level competition has
         already priced to the bone — and do not pay the scarcity
         premium single models charge for the last few index points.
         The dense pocket of near-frontier models crowding just above
         the elbow is the market confirming that tier (the same skipped
         pocket noted under slope as M3 diversity shards); density is
         the market's echo of the elbow, not an independent selection
         signal, which is why no density policy exists. elbow is
         therefore the recommended chat policy value at enable time.
         slope:<λ> remains for expressing an absolute willingness-to-
         pay, with a measured caution: λ=0.5 is far too generous for
         the budget tier — on this chart it buys to the top; the
         owner's revealed rate there is nearer $0.03-0.05 per index
         point.]
       floor:<n> — A-020(a)'s rule unchanged: lowest prompt price with
         intelligence_index >= n; ties by completion price, then permaslug.
     Table sourcing, the 24-hour cache, per-thread resolution timing,
     logging of the choice (now including the policy value), the per-thread
     manual override, and the REJECTION of per-prompt classifier routing
     all stand as A-020 wrote them. A-020 clauses (b), (d), (e), (f) stand
     unchanged and apply to every policy; read A-020(d)'s "floor-selected"
     as "policy-selected (non-pinned)". Env MODEL_INTELLIGENCE_FLOOR is
     superseded by the policy grammar and is not built.
     (a2) ROLES. Config binds one policy per agent role. M1 has two roles:
     chat (env MODEL_POLICY_CHAT, default pinned:<chat_model> — unset
     preserves today's behavior) and the /remember label agent, which
     follows the chat thread's resolved model per harness decision 008 and
     receives no independent policy in M1. Later-milestone roles MUST adopt
     this same policy type when their milestones arrive; their config keys
     do not exist until then (B.5). Design intent recorded for those
     milestones: price a role by the blast radius and reviewability of its
     errors, not by task difficulty — high-volume judge-filtered leaf work
     runs economic policies (slope, elbow, floor); low-volume compounding roles
     (orchestrators, judges, curators) run pinned or max. A spend-rate
     policy (budget:$/hr) is deliberately absent until the M2 Vitals spend
     lanes (v2.17) supply measured burn.
     (c-revised) The quantization filter is struck, not built:
     Artificial Analysis publishes per-model indices and polices — rather
     than measures — endpoint precision, and the broker's benchmark rows
     carry no endpoint dimension. provider_quantizations does not exist.
     Non-binding future exploration: adopt endpoint-level intelligence data
     if a source ever publishes it. A-020(c)'s surviving sentence — sort
     providers by price — applies to every non-pinned policy.
     (g) DEGENERATE TABLES. elbow over a frontier of fewer than three
     points falls back to max over that frontier; all other degenerate or
     unavailable-table cases keep A-020(f)'s fail-open to the static
     chat_model default.
why: The owner ratified this grammar across the 2026-07-27/28 design
     sessions: five auditable policy points over one cached numeric table,
     assigned per role, replacing both a single magic floor and any
     classifier. The
     elbow's normalization is pinned in law because a knee is only
     auditable when its axis scaling is explicit — and log price because
     frontier prices span orders of magnitude. The role table exists so the
     economics live where errors are filtered and volume is high, never
     where errors compound into the palace.

[A-022] [H5 FIXER] [SPEC C.4 commit, C.6, ADR-005] [P1.2.1b]
gap: SPEC v2.16 requires a near-miss "never show this" veto with the same
     semantics as removed:never, while C.4 and A-019 still reject every
     removed decision whose row was shown_as near_miss.
law: In POST `/v1/inject/commit`, a `removed` item may name an injected or
     pinned row with any existing removal reason, or a near_miss row only
     when its reason is `never`. A near-miss ID in both `removed` and
     `added_back` remains an RFC7807 422 with no write. The desired outcome
     for a near-miss veto is `removed:never`; an untouched near miss remains
     NULL. The veto uses exactly the same event transition, scorer-version
     `never_bias_step`, removals and never_kills increments, quarantine
     threshold, CAS revision, retry/idempotency rule, and final-block
     exclusion as removed:never on an injected or pinned row. A-019 browser
     membership validation uses the same class/reason rule. C.6's gate renders
     mutually exclusive Add and Never controls for every near miss.
why: This makes v2.16's owner-enacted F007 resolution executable by reusing
     the existing removal wire and kill path, without a field, migration, or
     second outcome family.

[A-023] [H5 FIXER] [SPEC C.6 step 3; A-019 gate.*] [P1.2.1a]
gap: C.4 returns current `wrong_removed` units so the UI can open the promised
     edit/expire flow, but A-019 defines only the initial review decision and
     would dismiss the hard pause before that response can be acted on.
law: The memory gate has two typed stages under the same run and hard pause.
     The existing stage is `review` (the default when `stage` is omitted).
     After its successful inject/commit, if `wrong_removed` is nonempty, the
     daemon keeps the model stopped and, in event-rank order, replaces the
     open gate with a `gate.open` whose `stage` is `wrong_resolution`, whose
     existing injected and near_misses arrays are empty, and whose
     `wrong_removed` array contains exactly one current MemoryUnit returned by
     that commit. It may also carry a visible `resolution_error` from a prior
     failed attempt. The browser answers with `gate.commit` carrying empty
     removed and added_back arrays plus
     `wrong_resolution:{memory_id,expected_revision,action,body?}`, where
     action is `edit` or `expire`; edit requires a nonblank body and expire
     forbids one.

     The daemon validates the run, thread, injection, stage, exact memory ID,
     displayed revision, and one-submit boundary before accepting the
     resolution. Edit PATCHes that unit's body; expire PATCHes its status to
     tombstoned. Both use the returned current revision, editor `user`, the
     trusted run machine_id, and reason `gate/wrong:edit` or
     `gate/wrong:expire`. A CAS conflict replaces the stage with the conflict's
     current MemoryUnit and a visible error; any other Spine client failure
     keeps the same stage open with a visible error, so neither silently loses
     the requested correction nor invokes the model. Cancellation retains
     A-016 ordering. Once every wrong unit is resolved, the daemon emits the
     single final gate.dismiss and invokes the model with the already-committed
     final_block. A review with no wrong removals remains the one-stage flow.
why: This completes the edit/expire consequence ADR-005 and C.4 already
     promise, using the existing gate lifecycle and PATCH contract rather than
     prebuilding H6's memory panel or trusting stale browser state.

[A-024] [H6] [SPEC C.7 memory.panel.update; C.6 live panel] [P1.2.1d, P1.3]
gap: C.7 names `memory.panel.update` as an M1 type, but gives it no direction
     or payload, so H6 cannot expose C.4 list, feedback, and PATCH behavior
     without inventing an undisclosed browser authority boundary.
law: `memory.panel.update` is bidirectional and requires outer `thread_id`.
     Its C→D payload is a discriminated union with exactly these required
     members:
       `{action:"refresh"}`;
       `{action:"remove", memory_id}`;
       `{action:"edit", memory_id, expected_revision, body}`;
       `{action:"pin", memory_id, expected_revision, pin}`.
     Memory IDs are UUIDs, `expected_revision >= 1`, body is a string, and pin
     is boolean. The D→C payload is one of:
       `{action:"state", request_id,
         result:"refreshed"|"removed"|"edited"|"pin_changed",
         items:[{memory:MemoryUnit,in_context:bool}], total}`;
       `{action:"conflict", request_id, operation:"edit"|"pin",
         memory:MemoryUnit, message}`;
       `{action:"error", request_id,
         operation:"refresh"|"remove"|"edit"|"pin", code, message}`.
     `request_id` echoes the triggering C→D envelope id. `code` and `message`
     are safe display strings and never carry credentials, response bodies, or
     raw exception text.

     The daemon derives principal, machine, editor, PATCH reason, and current
     injection ID from trusted configuration and thread state; the browser
     supplies none of them. State contains every ACTIVE C.4 list unit for that
     configured principal, in C.4 stable order, and `total` is that filtered
     count. `in_context` is true only for a member still retained in the
     thread's successfully committed injection.

     Remove is valid only for an `in_context` unit. It submits C.4
     `mid_thread_removed` using the server-held injection ID. Only after
     `{ok:true}` does the daemon remove that unit from the thread's current
     rendered block and add its ID to the thread's memory-tool exclusions;
     every later model call receives exactly that updated block, with stale
     dynamic memory blocks removed from provider history. Failure changes
     neither context nor exclusions.

     Edit and pin call C.4 PATCH with the displayed expected revision,
     daemon-owned `editor="user"` and machine ID, and reason `panel/edit` or
     `panel/pin`. The daemon never retries a human CAS conflict: it returns the
     conflict's current MemoryUnit so the UI replaces the stale row, visibly
     explains the conflict, and requires a fresh action. A successful mutation
     returns a refreshed state. Pin affects future injection candidacy; neither
     edit nor pin rewrites the already-frozen current-thread injection.
why: This fills the one wire/state seam H6 needs using only already-promised
     C.4 operations, preserves server-held identity and injection authority,
     and makes mid-thought removal bind the next model call without adding
     polling, persistence, retry automation, or a new API behavior family.

[A-025] [H9] [SPEC C.5; A-021(a-revised elbow), (g)] [P4]
gap: A-021 requires `log10(prompt price)` for elbow selection but does not
     classify a zero-priced frontier row, which the broker schema permits.
law: For elbow only, every frontier prompt price must be strictly positive. If
     any frontier prompt price is zero, the elbow table is degenerate under
     A-021(g), and resolution fails open to the static `chat_model` and static
     context tokens. Other policies continue to evaluate non-negative prices
     under their existing rules.
why: This applies A-021(g)'s existing fail-open to undefined log arithmetic
     without inventing an epsilon, dropping a benchmark row, or changing any
     other policy.

[A-026] [H9] [SPEC C.5; A-020(d); A-021(a-revised), (g)] [P4]
gap: The broker benchmark identifies a model by permanent `model_permaslug`,
     while the model listing uses that value as `canonical_slug` and uses
     `id` as the request route. The listing is not one-to-one: explicit
     `:<variant>` routes such as `:free` and `:batch` may share a canonical
     slug while advertising different context lengths.
law: A non-pinned policy first selects its benchmark row unchanged, then joins
     that row's `model_permaslug` to model-list rows by exact
     `canonical_slug`. A row whose `id` has no `:<variant>` suffix is a
     standard route. If exactly one standard route exists, the daemon resolves
     the request model to that row's `id` and the context window to that same
     row's positive `context_length`; explicit variant rows do not participate.
     If no standard route exists and exactly one joined row exists, that sole
     route and its context are used. No joined row, multiple standard routes,
     or multiple variant-only routes is a missing/ambiguous context under
     A-021(g) and fails open to the static model and context pair.
why: The policy selects a benchmarked model, not an unmeasured routing variant.
     Binding model ID and context from one broker row makes the chosen model
     executable without silently opting into free, batch, or extended
     semantics, while retaining a sole available route and the existing safe
     fail-open for actual ambiguity.

[A-027] [M2A] [ADR-024 ledger and broker seam] [P4]
gap: ADR-024 requires Harness broker calls to synchronously write Spine-owned
     receipt lines, but defines neither their cross-process write contract nor
     how its "3-4 lines" shorthand behaves when a reported price class is zero
     or reasoning is separately billed.
law: Spine exposes bearer-protected POST `/v1/spend/events` with body
     `{events:[SpendEvent...]}` and response `{accepted:n}`. The nonempty batch
     contains unique client-minted ULID `event_uid` values and every ADR-024
     spend_event column. M2A accepts only `llm.request` and `llm.embedding`.
     The write is atomic and append-only. Replaying an event_uid with every
     normalized field equal is an idempotent acceptance; reusing one with any
     different field returns RFC7807 409 and inserts nothing. `accepted` is the
     number of supplied lines, including identical replays. The route uses the
     standing application bearer unchanged.

     Replace "an LLM request emits 3-4 lines" with: "A successful broker
     request emits one receipt line for each nonzero price class the provider
     reports, all sharing the broker response or generation id as `ref`.
     Unreported and zero-value classes emit no synthetic line. When reasoning
     tokens are reported as a distinct billed subset of output, subtract them
     from the ordinary output quantity and emit a `reasoning` line; otherwise
     they remain output and the raw provider detail stays in `meta`. Embeddings
     emit `llm.embedding` input_fresh lines. If a broker supplies only aggregate
     native USD, allocate that exact total across its lines, mark those lines
     `basis=allocated`, and record the allocation source in `meta`; a directly
     attributable broker amount is `measured`. Missing cost remains NULL and
     never changes a measured token quantity into an estimate."

     Harness mints chat receipt ids and submits the complete response batch
     before terminalizing the turn. Spine's production embedding adapter mints
     and appends its receipt before returning vectors to memory behavior.
     Receipt-write failure fails the enclosing operation visibly; no
     best-effort background queue is introduced. A missing broker generation id
     uses the provider request-id header, or finally the first event_uid, as
     `ref`, with that fallback named in `meta`.
why: This supplies the smallest replay-safe HTTP seam between the already
     separate Harness and Spine processes, preserves Spine as sole ledger owner,
     and makes every recorded line an honest purchase rather than padding each
     request with zero-quantity pseudo-receipts.

[A-028] [M2C] [ADR-009 item 5; ADR-023 query surface; ADR-024 views] [P2.4, P4.1]
gap: M2C requires a first-party Vitals plugin to read canonical spend lanes and
     lifecycle gauges, but law gives no read contract or window, and the current
     logs cannot reconstruct several named lifecycle transitions. Treating
     absent instrumentation as zero would violate Invariant 10.
law: Spine exposes bearer-protected GET `/v1/vitals` with no query parameters.
     It returns one live trailing-hour snapshot with this exact shape:
       `{as_of, window_minutes:60,
         spend:{source_view:"v_spend_rate", latest_minute,
           lanes:[{dimension:"total"|"purpose"|"model", key, label,
             points:[{minute, cost_usd, receipt_lines, unpriced_lines}]}]},
         lifecycle_rates:[{metric, status, per_hour, source}],
         palace_counts:[{metric, status, count, source}]}`.
     `as_of`, `minute`, and nullable `latest_minute` are offset-aware timestamps.
     A total lane has `key=null`; purpose/model keys are their exact canonical
     values, with a null model represented by the stable key `unreported` and
     human label "Model not reported". `cost_usd` is an exact non-negative
     decimal string or null. Counts are non-negative integers. Lane points are
     grouped only from rows of `v_spend_rate` whose minute is within
     `(as_of - 60 minutes, as_of]`, ordered by minute; lanes are ordered total,
     purpose key, then model key. Total, purpose, and model regroup the same
     canonical rows, so priced dollars, receipt lines, and unpriced lines
     conserve across all three dimensions. A mixed-price point carries the sum
     of known dollars plus its positive `unpriced_lines`; an all-unpriced point
     keeps `cost_usd=null`. Missing cost is never rendered or aggregated as
     free. The endpoint reads the materialized view at its ordinary cadence and
     never refreshes it on demand.

     Gauge `status` is exactly `measured`, `not_recorded`, or `placeholder`.
     A non-measured gauge has a null numeric value and null source; it is never
     encoded as zero. M2C measures only `created` per hour from
     `memory_unit.created_at`, plus current `active_units` and active
     `pinned_units` counts from `memory_unit`. It returns the named lifecycle
     metrics `reinforced`, `superseded`, `merged`, `quarantined`, `tombstoned`,
     and `add_backs` as `not_recorded` until a canonical transition timestamp
     exists. It returns `candidates_pending`, `edges`, and `staged_units` as
     `not_recorded`, and `queue_depth` as the packet's explicit `placeholder`.
     The UI says these signals are not recorded or not active yet; it does not
     parse revision reasons, borrow prepare timestamps for later decisions, or
     infer transitions from a head's current `updated_at`.

     The compiled first-party Vitals resident obtains this response only through
     ADR-023's public query surface with
     `{resource:"vitals", as_of:null|"now"}`. Other `as_of` values return the
     existing `historical_unavailable` result. Hover/touch scrub publishes the
     minute on the shared selection surface together with the currently focused
     spend lane; it does not claim historical Palace counters. The frame retains
     `connect-src 'none'` and never receives Spine credentials or a private
     transport. M2C implements PLAN's wave-one total/purpose/model lanes; no
     plugin loader, authoring SDK, parameter registry, or lifecycle writer is
     introduced.
why: This completes the read and honesty seams already required by the M2C
     packet, using the canonical M2A view and existing memory heads without
     changing an authoritative row or pretending future M2 machinery has
     emitted events. Typed unavailability is reversible as those logs arrive,
     while guessed zeroes or a retrofit event history would not be.

[A-029] [M2C] [A-028 model lane keys; ADR-024 views] [P2.4, P4.1]
gap: A-028 reserves model-lane key `unreported` for a null canonical model but
     also says non-null model keys are their exact canonical values. The spend
     contract permits a real model whose canonical value is literally
     `unreported`; merging those rows would corrupt totals, while emitting two
     identical keys would make lane selection ambiguous.
law: Reserve public model-lane key `unreported` solely for a null canonical
     model. A non-null canonical model normally remains its exact public key.
     If its value equals `unreported` or begins with `~`, prefix one `~` to form
     the public key; its human label remains the exact canonical value. Thus a
     real `unreported` model has key `~unreported`, a real `~unreported` model
     has key `~~unreported`, and neither collides with the null-model lane.
     Deterministic model-lane ordering uses these resulting public keys. This
     key escape affects only the Vitals read projection and selection identity;
     it never rewrites a canonical ledger row.
why: This is the smallest injective completion of A-028's self-collision. It
     preserves every spend row and the stable null sentinel without narrowing
     accepted ledger data, adding a schema field, or changing authority.

[A-030] [M2G] [SPEC C.2, C.4 POST /v1/inject/prepare and /v1/feedback; ADR-005] [P1.2, P1.4]
gap: ADR-005 requires an atomic prepare on every post-first message, binary
     human locks, actor-classed passive outcomes, and human re-addition, while
     C.2/C.4 expose only one prepare per thread, no actor class, and feedback
     that can remove but cannot re-add. They also do not define the exact
     autonomous transition contract needed to render the next model context.
law: Add non-null `actor_class TEXT NOT NULL DEFAULT 'human'` to
     `injection_event`, constrained for M2G writes to `human` or `passive`.
     Existing rows are `human`. Extend POST `/v1/inject/prepare` with optional
     `mode`, exactly `gate` (default) or `autonomous`, and optional UUID lists
     `current_memory_ids`, `confirmed_memory_ids`, and `excluded_memory_ids`,
     each defaulting empty. Extend its response with nullable `final_block`,
     which is null in gate mode and the canonical C.6 block in autonomous mode.

     Gate mode requires all three lists empty and retains the existing one-shot
     prepare/commit behavior unchanged. Autonomous mode requires an already
     prepared thread with identical principal, agent, machine, and project
     identity; lists contain no duplicates,
     `confirmed_memory_ids` is a subset of `current_memory_ids`, and
     `excluded_memory_ids` is disjoint from both. One repeatable-read
     transaction locks that thread, advances `thread.snapshot_ts` to its own
     database-clock snapshot, retrieves the ordinary live candidate union plus
     every active eligible current or confirmed unit, and excludes every listed
     excluded unit before scoring. Missing, inactive, wrong-principal, or
     project-ineligible current/confirmed IDs return RFC7807 409 rather than
     silently weakening a lock.

     Selection remains score-descending threshold plus token-budget selection;
     configured top-k remains only its standing safety/display cap. Active pins
     and confirmed units are binary forced members: pins order first by UUID,
     then non-pin confirmed units by score DESC and UUID ASC. Both classes
     bypass threshold and top-k, their token costs reduce the ordinary regular
     budget to no less than zero, and they alone may overflow it. Remaining
     regular candidates retain C.3 order and greedy selection. Each autonomous
     batch writes one event for every returned selected or near-miss unit and
     for every prior current unit that exits. Selected prior members have
     outcome `kept`; selected new members `auto_entered`; unselected prior
     members `auto_exited`; undispositioned near misses remain null. Those rows
     have actor_class `passive`; gate rows have actor_class `human`.
     `final_block` is rendered from exactly the selected events in rank order.

     Extend feedback signal with `mid_thread_added`. It is valid only as the
     exactly-once/idempotent transition from `mid_thread_removed` on the same
     event membership; a later `mid_thread_removed` may symmetrically replace
     `mid_thread_added` on that membership. A first `mid_thread_removed` may
     transition a passive `kept` or `auto_entered` membership as well as the
     standing human outcomes; otherwise the panel could not remove context
     that entered through autonomous scoring. Re-addition does not change
     corpus counters. The Harness keeps
     daemon-lifetime per-thread current, confirmed, excluded, and event-source
     sets as trusted state: first-gate survivors become confirmed; panel remove
     moves a member from confirmed/current to excluded after Spine accepts
     `mid_thread_removed`; panel add moves it from excluded to confirmed/current
     after Spine accepts `mid_thread_added`. Each post-first ordinary prompt
     supplies those sets to autonomous prepare, atomically replaces only the
     autonomous portion of current context from `final_block`, and emits an
     unprompted `memory.panel.update` state when membership changes, using a
     daemon-minted request_id and state result `rescored`. Remember
     commands retain their command path and do not invoke re-scoring. The first
     gate remains the only modal memory review. Memory-panel state items add
     boolean `thread_excluded`; only those non-context rows expose re-add.
why: This is the smallest contract extension that makes the already-decided
     gate-once/autonomous-afterward behavior executable and replayable. It
     preserves every DONE-packet gate call, keeps lock magnitudes binary, uses
     the existing event and renderer authorities, and deliberately stops at
     daemon-lifetime thread state until later session-serving work exists.

[A-031] [M2F] [ADR-005 learning scope and scoreboard] [P1.2]
gap: ADR-005 fixes the learner's math and authority but leaves the deterministic
     time split, actor discount, proposal encoding, and manual trigger response
     undefined, so two conforming learners could fit and promote different
     versions from the same log.
law: The M2F learner reads one repeatable-read snapshot of `injection_event`,
     ordered by gate time then injection UUID, and groups rows by injection_id.
     Hygiene excludes a group when any row's principal or machine identity is
     test/fixture/verification-class: exact or delimited `test`, `fixture`, or
     `verification`, including a machine value ending in `-verification`.
     Eligible positive dispositions are human `kept`, every `added_back`,
     `cited`, and `mid_thread_added`, plus passive `kept` and `auto_entered`;
     eligible negatives are every `removed:not_relevant`, `removed:never`, and
     `mid_thread_removed`. Explicit disposition/feedback outcomes have weight
     1 even when they replace a row originally created with actor_class
     `passive`; only passive `kept` and `auto_entered` positives receive the
     configurable discount, default 0.25. Null, `removed:wrong`, `auto_exited`,
     and every other outcome are ungraded.

     Sort eligible gates by `(max(ts), injection_id)`. The newest configurable
     fraction, default 0.20 rounded up to whole gates, is holdout; every older
     gate is training. At least one gate and one eligible disposition must
     remain on each side. Before the first challenger, require configurable
     minimum eligible dispositions, default 25. Training creates every
     positive-negative pair within each training gate and performs one
     deterministic whole-log convex re-fit using squared hinge loss,
     configurable pair margin default 0.05, non-negative global weights
     constrained to sum exactly 1, and L2-shrunk per-memory bias offsets with
     configurable coefficient default 1.0. A pair containing a passive example
     receives the smaller actor weight. The online head bias remains the
     immediate never-kill safety term; a version's learned offset is added to
     it, not substituted for it.

     Replay grades each eligible holdout disposition as one binary decision.
     A pin predicts injected. A regular row predicts injected exactly when its
     recomputed score meets the incumbent version's manual tau; budget and k
     remain manual and are not learner variables. This per-disposition replay
     is the v1 meaning of ADR-005's storage-complete claim: it uses the recorded
     feature tuple and frozen body, never fabricates candidates outside the
     logged gate. Each mismatch adds its actor weight; cheaper-at-tie sums
     `cl100k_base` body tokens for predicted injected dispositions. A challenger
     wins when its weighted mismatches improve on the incumbent's recorded
     decisions by the configurable absolute margin, default 1.0, or when
     weighted mismatches tie exactly and it injects fewer tokens.

     POST `/retrain` has no request body and returns
     `{status, incumbent_version, proposal_version, eligible_dispositions,
     training_dispositions, holdout_dispositions, training_pairs, incumbent,
     challenger, reason}`. Status is `insufficient_data`, `not_better`, or
     `proposed`; proposal_version and both score objects are nullable where no
     comparison exists. A score object is
     `{disagreements, weighted_disagreements, injected_tokens}` with the
     weighted value serialized as an exact decimal string. The trigger never
     activates a version.

     A winner inserts one inactive `scorer_config`. Its deterministic version
     derives from a canonical digest of the incumbent, ordered training and
     holdout examples, and learner settings. It copies every manual scorer
     parameter unchanged and stores under params `_learner`: status
     `proposed`, algorithm id, source digest and boundary, settings, fit
     diagnostics, replay scores, and the learned memory-id-to-bias-offset map.
     Repeating an identical fit returns the same proposal idempotently. Active
     scorer loading applies that map when such a version is later activated;
     old rows have an empty map. Optional positive
     `SPINE_LEARNER_SCHEDULE_HOURS` runs the same operation periodically;
     unset means manual-only. Concurrent triggers serialize on one database
     advisory transaction lock.
why: This makes the owner-resolved batch learner and binary referee exactly
     reproducible from the existing append-only evidence, preserves manual
     activation and manual selection parameters, and adds no online SGD,
     learner UI, automatic promotion, or hidden training state.

[A-032] [M2H] [ADR-021 clause 4 and unified queue; ADR-022 kinships] [P1.4, P1.5]
gap: M2H requires verdicts at birth and says approval enacts them, but the
     general curator tool set and health-report worker do not land until M3.
     The queue row, exact decision transition, archive/idle trigger, and
     viewport-visible passive-resolution contract are otherwise unspecified.
law: Add `candidate` to memory-unit status and add append-only
     `memory_edge` plus `approval_queue_item` and `approval_decision` rows.
     A queue item owns one candidate unit, one birthplace thread, ordered
     machine-fetched neighbor IDs, zero or more implicated target IDs, and one
     proposed verdict: `new`, `merge`, `supersede`, or `contradict`. Its state
     is `pending`, `approved`, or `rejected`. Candidate creation runs under the
     same principal advisory lock, embedding provider, body/label limits, and
     cosine thresholds as ordinary creation, but deduplicates against both
     ACTIVE and CANDIDATE heads. A hard duplicate creates no candidate or queue
     row; a similar match is admitted and appears in ordered neighbors.

     POST `/v1/extractions` accepts one thread-born batch of at most five
     atomic, nonblank, keyworded candidates and returns the admitted queue
     cards. Each candidate supplies its proposed verdict and implicated target
     IDs; targets must be same-principal ACTIVE units and must be present in
     the machine-fetched neighbor set. GET `/v1/approval-queue` reads pending
     cards, optionally narrowed to birthplace thread. POST
     `/v1/approval-queue/{item_uid}/decisions` accepts `approve` or `deny`,
     `approval_mode` explicit or passive, and actor class human or passive.
     Deny is always explicit/human, tombstones the candidate with revision
     reason `rejected`, and appends the decision. Approve activates the
     candidate and appends the decision. `new` changes nothing else; `merge`
     and `supersede` revision-tombstone their implicated targets and append
     respectively `merged_from` or `supersedes` edges from the new unit;
     `contradict` leaves targets active and appends `contradicts` edges.
     Contradiction rejects passive mode mechanically. A repeated identical
     decision is idempotent; a conflicting second decision is 409. These are
     the minimal deterministic verdict mechanics M2H needs, not the M3 curator
     agent, report, autonomy, generic tool SDK, or graph retrieval.

     Harness extraction reads the durable M2D transcript on explicit
     `POST /v1/threads/{thread_id}/archive`; a daemon idle sweep uses the same
     path after configurable positive `EXTRACTION_IDLE_HOURS`. The model emits
     working summary, open loops, and at most five candidate drafts. Archive is
     idempotent per transcript tail. Failed extraction leaves the thread
     unarchived and returns a visible service error; an idle failure is logged
     and retried without blocking chat. The thread-end rack card shows the
     final assistant post above its candidate list. An IntersectionObserver
     marks an individual uncollapsed, non-contradiction row seen only when its
     whole row intersects the literal viewport. Resolving the card explicitly
     denies selected rows and passively approves only those marked seen;
     unseen, collapsed, and contradiction rows remain pending for the Palace
     queue module. Passive sends actor class passive and approval_mode passive;
     taps send human/explicit. No timer resolves queue items and no queue event
     notifies the owner. The law-bound module declares CURRENT by default,
     follows the shared thread selection, and persists GLOBAL|CURRENT only as
     rack layout presentation state.
why: This supplies the smallest auditable bridge between extraction consent
     and M3 curator operations. It makes every approved verdict real without
     smuggling in the future diagnostic agent, and it defines literal passive
     visibility so scrolling past a card cannot silently approve hidden work.

[A-033] [M2I] [ADR-019 clause 4; ADR-021 unified queue; ADR-022 splitting] [P1.4, P1.5]
gap: M2I requires corpus-born, per-document seed batches, but M2H's queue
     contract records only a non-null thread birthplace and exposes only
     item-at-a-time decisions. The seed splitter also has no bounded request,
     retry identity, or executable source-lineage contract. Reusing a fake
     thread ID would corrupt birthplace routing, while activating children
     one at a time would violate the explicit grouped-batch action.
law: Harness exposes JSON POST `/v1/seeds` to its own rack surface with
     `{batch_uid,source_name,markdown}`. `batch_uid` is a client-minted UUID;
     `source_name` is the basename of a `.md` or `.markdown` file; markdown is
     nonblank valid UTF-8 and at most 24 KiB. The browser may submit several
     selected files, but each file is one independent request and one batch.
     Markdown is the only M2I format. The tools-free splitter receives the
     complete document and emits one or more independently comprehensible
     candidates, each with one claim, its own label, kind, and 2-5 keywords.
     It may emit at most 64 children, and every child must satisfy the standing
     128-token memory limit. If those bounds cannot preserve the source, the
     request fails visibly; it never truncates, mechanically chops, summarizes,
     or partially substitutes the document.

     Harness fetches the active neighborhood and proposes the standing
     new/merge/supersede/contradict verdict for every split child, then submits
     the whole result to bearer-protected Spine POST `/v1/seeds`. Spine verifies
     the SHA-256 digest of the supplied markdown, creates one queue-invisible
     tombstoned-as-split source head whose first revision retains the exact
     markdown, and runs every child through M2H's same candidate create/dedup
     path. Each admitted child's first revision has `parent_uid` equal to that
     source revision; every admitted sibling pair receives symmetric
     `relates_to` edges. Hard duplicates produce no child card and increment
     the batch duplicate count. Candidate children remain invisible to list,
     search, and injection.

     Extend each approval queue row/card with birthplace exactly `thread` or
     `seed`, nullable thread UUID, and for seed rows non-null `batch_uid`,
     `source_name`, and lowercase SHA-256 digest. Existing rows migrate as
     thread-born. Queue reads can filter birthplace; thread-end surfaces always
     request thread-born rows, and seed rows render only in the law-bound Palace
     queue module. Reusing a batch UID with the same principal, source name, and
     digest returns its existing batch without re-splitting; any mismatch is
     409. There is no stored upload blob beyond the tombstoned source revision
     and no filesystem copy.

     POST `/v1/approval-queue/batches/{batch_uid}/decisions` accepts only
     `{decision:approve|deny,approval_mode:explicit,actor_class:human,machine_id}`
     and decides every pending card in that seed batch in one transaction.
     Approval enacts each card's standing verdict; denial tombstones every
     candidate. An identical replay returns the existing decisions; a mixed or
     conflicting replay is 409 and changes nothing. Any stale implicated target
     makes the whole approval 409 and leaves the batch pending. Seed batches
     never passive-approve, expire, notify, or appear in a thread-end card.
why: This is the smallest honest extension of the queue M2H already shipped.
     One tombstoned source head makes ADR-022's existing revision lineage real
     without a document store, explicit batch decisions preserve the owner's
     attention boundary, and strict failure preserves information better than
     a convenient lossy splitter.

[A-034] [M2J] [ADR-023 clauses 3 and 5; C.5; v2.26 resolution points] [P2.5, P3, P4]
gap: M2J names a typed registry, control writes, history, and a scoped model
     device but does not define the parameter domains, public write/query
     boundary, refusal event, or the meaning of GLOBAL for thread-only controls.
law: The M2J Harness registry declares exactly these free-journaled thread
     descriptors: `model.slug` (nonblank OpenRouter model string),
     `model.temperature` (nullable number, 0..2), `model.top_p` (nullable
     number, 0..1), `model.top_k` (nullable integer, 0..500),
     `model.max_tokens` (nullable integer, 1..131072), and `model.effort`
     (nullable option: none|minimal|low|medium|high|xhigh). Null means inherit
     the provider default and is every model-parameter descriptor's default.
     The selector's displayed default is the thread's resolved slug. The
     superseded A-020 environment floor is not resurrected: A-021's policy
     grammar remains configuration, and PLAN M2J's delivered controls are the
     per-thread selector and five request parameters.

     Extend ADR-023's public rack query surface with resource `parameters`,
     requiring a thread UUID for CURRENT. A live result is
     `{thread_id,as_of,resolved_model,descriptors,values,changes}`; descriptors
     carry the clause-3 fields, values contains the six current values, and
     changes is ordered by timestamp then event id. `as_of=now` reads current
     state; an aware ISO-8601 timestamp replays daemon-lifetime registry events
     at or before that instant. A timestamp earlier than the retained process
     history is still an honest replay from descriptor defaults and the
     thread's first resolution; M2D's capture-only journal is not served back.

     The public control action submits
     `{module_id,thread_id,parameter_id,value}`. The host and daemon both require
     the module manifest to bind that exact descriptor. A successful write
     validates the descriptor, atomically changes thread state, and publishes
     one C.7 `parameter.change` event with event id, actor `human`, timestamp,
     old and new values, and scope/thread identity; transcript capture makes
     the event durable. Null resets a request parameter to provider inheritance.
     `model.slug` invokes the existing `resolve_named` seam, preserves the five
     request overrides, starts a new stickiness epoch, and emits the standing
     `model_change` event as well as `parameter.change`. Writes while that
     thread has a live run are refused so a request cannot change underneath
     execution.

     Any unknown, unbound, law-bound, invalid, or busy write changes nothing
     and publishes one C.7 `parameter.refused` event with the attempted id,
     module id, timestamp, and stable reason; values and credentials are not
     copied into refusal events. The MODEL DEVICE manifest is a control plugin
     bound only to the six descriptors above. CURRENT follows the shared thread
     selection and may write. GLOBAL shows the same registry descriptors and
     inherited defaults without a thread target and is read-only until a real
     global descriptor ships; the GLOBAL|CURRENT toggle remains saved rack
     presentation state and never enters the registry.
why: This supplies the smallest replayable, mechanically bound seam for M2J's
     real broker controls and selector while preserving A-021, M2D's explicit
     capture-only boundary, and v2.40's separation of module scope from
     parameter scope.

[A-035] [M2K] [ADR-009 items 3-4; ADR-023 clauses 3 and 5; Invariant 6] [P1.2.3, P2.3, P2.5]
gap: M2K names authoritative graph encodings, scorer-version controls,
     accuracy, contribution bars, and GLOBAL|CURRENT behavior but defines no
     read/write boundary, graph membership, descriptor domains, activation
     journal, or exact contribution arithmetic. The existing Vitals contract
     also cannot express CURRENT thread spend.
law: Spine exposes bearer-protected POST `/v1/memory-graph/query` with
     `{principal_id,memory_ids}` where `memory_ids` is null for GLOBAL or a
     unique UUID list for CURRENT. GLOBAL returns every non-candidate memory
     head of that principal; CURRENT returns only requested same-principal
     non-candidate heads and names any omitted IDs. The live result is
     `{as_of,graph_edge_sim,nodes,edges,omitted_memory_ids}`. Each node carries
     the complete current MemoryUnit, `in_current_context`, and its ordered
     revision trail `{rev_uid,parent_uid,revision,ts,reason}`. Node size derives
     only from `stats.injections`; color only from kind; brightness only from
     updated_at recency; pin alone supplies a halo; quarantine is ghosted;
     tombstone is visibly struck/faded; CURRENT membership alone supplies the
     live-thread border pulse.

     Graph edges join returned nodes only. `similarity` carries exact cosine
     similarity and exists iff it is at least configurable
     `SPINE_GRAPH_EDGE_SIM` (default 0.75), using the stored embedding space.
     `lineage` carries the exact stored memory_edge type. A head with more than
     one revision also receives one self-edge `edit_trail` carrying its
     revision count; the inspector renders the ordered revision trail. Node
     click publishes a memory selection only: the existing Memory Panel owns
     the C.4 CAS edit. The Memory Graph remains a visualizer and never writes.
     Historical graph reconstruction is `historical_unavailable` until head
     status/stats/pin have replay-complete logs; current heads never masquerade
     as past truth.

     Spine exposes bearer-protected POST `/v1/scorer-console/query` with
     `{principal_id,thread_id,as_of}`. Null thread is GLOBAL; a UUID thread is
     CURRENT. It returns the typed law-bound descriptors, configuration and
     activation history, active version, PROPOSED learner versions with their
     replay manifests, accuracy points, and scored-candidate histories. GLOBAL
     includes the principal's events; CURRENT includes that thread only.
     Candidate points retain event_uid, time, version, score, rank, shown_as,
     outcome, six raw features, and seven decimal-string contributions:
     `sem|kw|time|proj|freq|hist = stored feature * that event version's
     weight`; `bias = stored score - exact decimal sum of those six`.
     Consequently the seven decimal contributions sum exactly to the stored
     score. Gate and Memory Panel cards render these weighted contributions;
     a unit with no scored event says `Not scored yet` rather than inventing
     bars. Accuracy is
     `100 * (holdout_dispositions - disagreements) / holdout_dispositions`
     from the learner replay manifest, or null/`not_recorded` when that
     denominator is absent. M2K may add the denominator to future M2F proposal
     manifests; existing proposal content is never rewritten.

     The console declares exactly these GLOBAL law-bound descriptors:
     `scorer.tau` number 0..1; `scorer.top_k` integer 1..8;
     `scorer.budget_tokens` integer >=1; `scorer.half_life_time_days` and
     `scorer.half_life_hist_days` numbers >0; and six numbers 0..1 named
     `scorer.weight.sem|kw|time|proj|freq|hist`, whose enacted vector must sum
     to 1 within 1e-9. Step hints are .01, 1, 128, .5, and .01 respectively;
     a step is presentation, never rounding authority. The console manifest
     binds all eleven IDs. Its scope toggle changes the READ candidate set;
     the descriptors remain visibly Palace-wide in both scopes and never
     become thread parameters.

     POST `/v1/scorer-configs` accepts the base version, the complete eleven-
     value set, and daemon-owned human/machine attribution. It validates the
     base is active, copies every non-controlled parameter unchanged, INSERTs
     one new scorer_config version, deactivates the prior version, activates
     the new one, and appends one scorer_activation row in one transaction.
     POST `/v1/scorer-configs/{version}/activate` accepts only an inactive
     learner-PROPOSED version and atomically makes it active. Every activation
     appends `{event_uid,version,previous_version,actor_class,machine_id,
     reason,changes,ts}`; exact old/new controlled values are journaled and a
     matching `scorer.change` C.7 event is published. Replays of an event UID
     are idempotent only when identical. No update mutates a configuration's
     weights or params. WHAT-IF re-ranking and AUDITION remain entirely M2P.

     For the spend strip's CURRENT scope, Spine adds bearer-protected GET
     `/v1/vitals/threads/{thread_id}`. It returns the existing Vitals shape,
     but spend lanes group trailing-hour authoritative spend_event rows for
     that exact thread and label `source_view="spend_event"`; palace gauges
     remain explicitly Palace-wide. GET `/v1/vitals` and its canonical
     `v_spend_rate` projection are unchanged. Every graph, console, and Vitals
     module persists its GLOBAL|CURRENT toggle only in rack layout state and
     every CURRENT query follows the shared selected thread without a second
     selector.
why: This is the smallest mechanically honest contract for the M2K surfaces.
     It uses existing heads, revisions, events, configs, ledger rows, CAS edit,
     and selection bus; adds one append-only activation journal; and does not
     build M2P previews/auditions, M3 graph maintenance, bulk actions, private
     first-party data paths, or fabricated historical state.

[A-036] [M2L] [ADR-005 citation heuristic; C.3 f_freq; C.4 /v1/feedback] [P1.2]
gap: ADR-005 chooses an n-gram heuristic for M2 v1, but does not fix n, the
     text boundary, short-memory behavior, per-message event attribution, or
     the canonical statistic transition. Implementations could therefore
     produce different training labels from the same turn or keep f_freq inert.
law: Citation v1 tokenizes with C.3's maximal runs of Unicode alphanumeric
     characters after lowercase conversion. For each memory selected in the
     exact context of one ordinary model call, let n be the smaller of eight
     and that memory body's token count. A body with fewer than four tokens is
     not autonomously cited. Otherwise the memory is cited exactly when at
     least one of its contiguous n-token sequences occurs contiguously in the
     successful final assistant text. Labels, prompts, thinking, tool traffic,
     system instructions, and failed or cancelled outputs do not participate.
     Repeated matches in one output still produce one signal for that memory.

     Harness snapshots the selected bodies and each member's current
     injection-event source at the same model/feedback lock boundary M2G uses,
     then submits one existing `cited` feedback per detected member after the
     successful model call and before releasing that boundary. Thus every
     post-first message attributes reuse to its own autonomous scoring batch;
     the first gated message attributes it to the committed gate batch. A
     citation-feedback failure emits the existing safe memory-unavailable
     error with phase `citation` but does not retract or replace an assistant
     answer already produced. Each member is attempted independently; no
     background retry queue or cross-member batch endpoint is introduced.

     Spine permits `cited` from `kept`, `added_back`, or `auto_entered`.
     Its first successful transition atomically changes that event outcome to
     `cited`, increments the current head's `stats.citations` once through C.2
     CAS, and appends the ordinary revision with editor `system:feedback`, the
     event machine, and reason `feedback/cited`. An identical replay is an
     idempotent no-op. This is the f_freq numerator; no weight, bias, status,
     or other statistic changes. Existing outcome-conflict rules remain.
why: This makes the already-chosen cheap citation signal deterministic and
     replayable, activates the scorer feature ADR-005 reserved for M2, and
     rides M2G's per-message event source without adding semantic judgment,
     provider cost, another transport, or a UI surface.

[A-037] [M2M] [ADR-024 sourcing and self-audit] [P4.1]
gap: ADR-024 requires broker reconciliation and a Vitals drift alert but does
     not freeze the credential boundary, comparison baseline, tolerance,
     durable audit row, schedule, failure state, or Vitals projection.
law: M2M uses the existing OpenRouter bearer only; it never requests a
     management key. Spine reads cumulative current-key usage from official
     `GET /api/v1/key` field `data.usage`, a finite non-negative USD number.
     Reconciliation is active only when the configured embedding base URL is
     the OpenRouter API and `SPINE_OPENAI_API_KEY` is present. Direct-provider
     overrides remain honest `not_recorded`, never queried with the wrong key.

     Add append-only table `spend_reconciliation` with ULID `event_uid` primary
     key, aware database timestamp `ts`, provider `openrouter`, status exactly
     `baseline|balanced|drift|unavailable`, nullable cumulative
     `broker_usage_usd` and `ledger_cost_usd`, nullable
     `broker_since_baseline_usd`, `ledger_since_baseline_usd`, and signed
     `drift_usd`, non-negative `tolerance_usd`, non-negative
     `unpriced_lines`, and nullable stable `error_code`. Successful numeric
     fields use the ledger's 12-decimal USD precision. `unavailable` has null
     USD observations except tolerance, may retain the current ledger
     unpriced-line count, and carries only
     `broker_unavailable` or `invalid_broker_response`; raw errors, response
     bodies, headers, and credentials are never persisted.

     One advisory-locked reconciliation reads broker usage and, in the same
     run, sums every non-null `cost_usd` on `llm.request` and `llm.embedding`
     rows and counts their null costs. The first successful observation is a
     baseline. Every later success compares cumulative growth from the oldest
     successful baseline:
       `broker_since = broker_now - broker_baseline`
       `ledger_since = ledger_now - ledger_baseline`
       `drift = ledger_since - broker_since`.
     A negative cumulative growth is an invalid broker response. Status is
     `drift` exactly when `abs(drift) > tolerance`, otherwise `balanced`.
     Default `SPINE_RECONCILIATION_TOLERANCE_USD` is `0.000001`; default
     `SPINE_RECONCILIATION_HOURS` is 24, both positive. The owned Spine
     process runs once at startup and then at that cadence. Failures append an
     unavailable observation and are logged; they never block receipts, chat,
     embeddings, Vitals, or the next scheduled run. No GCP resource or billing
     export is read or created.

     Both Vitals scopes add Palace-wide `reconciliation` with status exactly
     `not_recorded|baseline|balanced|drift|unavailable`, nullable `checked_at`,
     the latest row's exact decimal-string observations, tolerance,
     `unpriced_lines`, source `openrouter:/api/v1/key` when configured, and a
     safe nullable error code. With no configured reconciler or no row, status
     is `not_recorded` and observations are null. The rack renders this as a
     compact watchable line: baseline, aligned, signed drift, temporarily
     unavailable, or not yet recorded. Drift uses the theme's sole danger
     color but creates no popup, notification, card, or new attention channel.
why: This compares the one existing broker key to the one authoritative ledger
     without a third secret, discounts spend predating M2A, preserves audit
     history, and makes disagreement visible without building the deferred GCP
     export or demanding owner attention.

[A-038] [M2O] [B.6 rules 10-11] [P3, P4.1]
gap: B.6 requires isolated fixtures, a queued estimated receipt, and loud
     Vitals drift, but does not define fixture identity enforcement, legacy
     catalog cleanup, queue durability/replay, or the immediate Vitals shape.
law: Every Harness fixture/scenario app has one nonblank identity ending in
     ` REGRESSION`, exposes it from GET `/__scenario__/identity` as
     `{fixture,deterministic:true}`, and redirects a top-level UI request that
     lacks the exact `fixture` query value to the same URL with that value.
     It refuses every request received on product port 8765 before serving UI
     or accepting a WebSocket. The rack renders the verified identity as a
     pointer-transparent, viewport-covering FIXTURE overlay; a query string
     alone never creates the overlay. Fixture launchers run in the foreground
     or own the child in a `finally` cleanup, reject port 8765, and browser
     automation creates a fresh temporary profile which it removes on exit.

     The one-time polluted-catalog cleanup is explicit and narrow. The thread
     rail offers `Remove fixture threads` only when the browser-local catalog
     contains a title produced from the first prompt of a pre-M2O H4, H5, H6,
     H8, or M2G deterministic scenario. It shows the count and removes only
     those exact catalog entries after the human activates it; it never clears
     all storage, guesses from substrings, deletes transcripts, or touches a
     catalog on a distinct fixture origin.

     A failed or incomplete POST `/v1/spend/events` never changes a completed
     model outcome. Harness changes each failed batch line's basis to
     `estimated`, retains its event_uid and broker timestamp, records the
     original basis plus queue time in safe meta, and stores one atomic
     mode-0600 JSON batch beneath `NOCTURNE_HOME/receipt-queue`. It retries
     queued batches oldest-first at daemon startup and before a later current
     receipt. A file is removed only after Spine accepts its complete batch;
     replay therefore rides A-027 idempotency. Queue or retry failure emits a
     safe `spend_pending` event but does not fail, retract, or replace the turn.
     If disk enqueue itself fails, the process retains the batch in memory and
     reports degraded accounting; accounting remains fail-open.

     Harness enriches only its public Rack Vitals result (the Spine A-028 body
     is unchanged) with `accounting:{status,pending_lines,oldest_queued_at,
     source}`. Status is `clear`, `pending`, or `degraded`; source is always
     `harness.receipt_queue`; counts include disk and memory batches; oldest is
     null exactly when no line is pending. `degraded` means at least one
     pending batch is not durable. Vitals renders pending/degraded in the
     theme's sole danger color as a persistent accounting-drift line, without
     an alert role, popup, notification, or card. Successful replay returns it
     to clear. Broker reconciliation remains the dollar authority and exposes
     any cumulative mismatch on its existing cadence.
why: This makes the two owner-observed failures mechanically visible and
     replay-safe at their existing boundaries, without adding a mock mode, a
     second ledger, a notification channel, or generic browser-data cleanup.

[A-039] [M2R] [ADR-009 item 1, ADR-023 clauses 1 and 5] [P2.2, P2.5]
gap: The law requires category bars against the resolved model's true context
     length, but OpenRouter reports only the request's aggregate input-token
     count. It does not provide a trustworthy system/history/memory/tools split,
     and M2 does not yet enact compaction policy.
law: Harness owns one daemon-lifetime context observation per thread, replaced
     only after a model request returns. The observation's `used_tokens` is the
     terminal broker response's per-request input-token count, never the run's
     cumulative usage. Its `context_tokens` is the exact context length on that
     request's immutable `ThreadModelResolution`. Before a completed request,
     no observation exists and the instrument says that it is waiting for the
     first model response.

     The four displayed categories are `system`, `history`, `memory`, and
     `tools`. Because the broker does not report this split, Harness labels it
     `estimated breakdown`: memory is the cl100k count of the exact injected
     MEMORY BLOCK; system and tools are cl100k counts of the owned capability
     instruction text and stable public tool definitions; history is the
     non-negative remainder of broker-reported input after those three values.
     If the three estimates exceed broker input, they are reduced in
     tools-then-system-then-memory order until the category sum equals the
     broker total. The aggregate used count and context limit remain measured
     facts even when category estimates are imperfect.

     The instrument draws a presentation-only threshold at 80% of the resolved
     context length, says `Compaction is not active`, and shows a compact table
     of category token counts. It never compacts, warns, blocks, predicts, or
     changes a run. CURRENT returns the selected thread's observation. GLOBAL
     returns every observed thread plus aggregate used and capacity totals; its
     bars and table show those category sums. Missing CURRENT selection or an
     unobserved thread is a truthful empty state. The first-party CONTEXT BARS
     module is compact by default beside Palace Vitals and uses only the public
     Rack query, selection, and scope surfaces.

     Customer-facing copy across the existing owner app must not expose Garden
     identifiers, packet names, amendment numbers, ADR references, scenario
     lineage, or implementation vocabulary as product explanation. Internal
     protocol values and developer-only diagnostics may remain stable; visible
     labels and recoverable errors are rephrased in owner language without
     changing their behavior.
why: The owner needs the real denominator and a useful map of what crowds it,
     but false precision would be worse than no map. One measured total plus an
     explicitly estimated split keeps provenance visible, while a passive 80%
     line previews the future pressure boundary without smuggling M3 compaction
     machinery into M2.
