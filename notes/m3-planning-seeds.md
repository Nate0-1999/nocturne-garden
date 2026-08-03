# M3 planning seeds (started 2026-08-02)

Gate-collected inputs for M3 planning, gathered while M2's wave runs.

1. TOOL-PARITY SURVEY (owner directive, gate-day + 2026-08-02 "the basic
   stuff all main harnesses have"): field research on what Codex / Claude
   Code / leading harnesses ship vs what pydantic-ai + MCP give us free
   (function tools + MCP client already ride our ADR-013 seam via
   pydantic_ai_adapter) vs what needs porting (Codex browser extension).
   List: terminal/shell, filesystem, browser use, Agent Skills, MCP,
   CodeMode, background tasks, scheduled runs, sub-agent archetypes,
   compaction batteries, cost introspection. HANDS AND WALLS SHIP
   TOGETHER: every tool lands behind ADR-015 sandbox-first boundaries.
   IMAGE INPUT is committed same wave (v2.38).
2. THE DEVICE CHAIN (vision §19): skill/tool visualizer as an Ableton
   device chain — loaded, fired, per-skill cost via ledger purpose lanes,
   enable/disable toggles = a registry-bound CONTROL plugin. Design with
   the tools, not after them.
3. Already-queued M3 law awaiting packets: curators (ADR-022), staging +
   judge memory gate (ADR-021), movement law + f_loc (ADR-010), session
   tables/tree/forests (ADR-016 + journal backfill), role model policies
   (A-021), spend walls (ADR-024), the Cube + public plugin SDK +
   contributor skill (ADR-018/023), Ant Farm, per-message f_phase/f_sess
   candidates via replay.

4. THE MAINTENANCE CYCLE + COMPOSED REPORT (gate pitch, owner-corrected
   2026-08-03; SPECULATIVE — owner not yet sold on the bundle): all
   offline organs (learner re-fit, curator pass, retro-extraction over
   the journal, broker reconciliation, health report) consolidate into
   ONE cycle triggered by ACCUMULATED WORK (N dispositions / writes /
   messages since last pass — owner doctrine, never temporal crons; an
   idle agent runs nothing). On cycle completion the system COMPOSES a
   short prose report from the logs — memories learned, proposals
   awaiting taps, spend story — waiting as the Deck's first card on next
   arrival. "A nocturne is what it writes, not when it runs."
5. MEMORIES AS INDEXES / fetch_episode (owner: LOVED — "short memories
   act as indexes"): every unit already carries thread_origin (M1
   metadata, preserved deliberately). Build the follow path: M3 session
   serving + a fetch_episode tool/UI affordance so agents and the owner
   open the full source conversation when the index card isn't enough —
   10%-principle-compliant deep context (pay for the book only on
   demand). Sibling (secondary): similar-past-threads retrieval over
   thread embeddings for episodes no memory points at.
6. THE AUDITION (ghost gate, de-jargoned): before accepting a PROPOSED
   scorer version, run it as a visible shadow for a day — incumbent's
   picks happen, challenger's picks render faintly alongside ("would
   also have shown X / left out Y"). Accept after WATCHING it work, not
   after trusting a number. Cheap: challenger exists, prepare scores
   twice, ghost renders at low opacity.

## Empirical tool census (this machine's logs, 2026-08-03 — the survey's
## first dataset; aggregate counts only)

CLAUDE CODE (gate sessions): Bash 685 ≫ Edit 432 > Read 177 > browser
36 > Write 32 > WebFetch 18 = AskUserQuestion 18 > Artifact 16 >
WebSearch 12 > sub-Agent 8 > Skill 6.
CODEX (builder sessions): exec/shell overwhelmingly dominant (~10k+ real
calls); browser used in SOP packets; the rest is session plumbing.

FINDINGS: (1) SHELL IS THE KING by 10x — git, tests, everything; (2)
EDIT beats WRITE 13:1 — surgical patching is the real writing tool; (3)
browser use is essential but episodic (verification, not building); (4)
skills/MCP are GARNISH in practice on this machine (6 skill calls total)
— the "basic stuff" = shell, edit, read, browser, web fetch/search, and
structured questions to the human.

PINNING ANCHORS (owner directive: "static but with versioning to some
open source version" — this IS ADR-013 adopt-wrap-pin, applied):
- shell/fs/read/edit → own thin tools or pydantic-ai natives (versioned
  by our own wheel; no external dep to pin);
- browser → Playwright, version-pinned (the boring OSS anchor; enables
  rule-8 SOPs to run IN-harness; Codex-extension port stays optional);
- search → ripgrep, pinned binary;
- web fetch → httpx (already a dep), pinned;
- MCP client → official modelcontextprotocol Python SDK, pinned;
- skills → the Agent Skills open standard (SKILL.md; already ADR-007).

## N1 — THE OUROBOROS (proposed M3 north-star acceptance test)

"NOCTURNE completes one relay packet on its own repositories, end to
end": boot sequence → read law → run both suites → claim → edit → test
→ commit → handoff report, under walls. Requirements = shell+git, file
read/edit, the work protocol, boundary walls; browser only for UI
packets. The harness building itself is the parity wave's honest
finish line. Rough distance: wave-2 remainder → M2 judge → M3 parity
packets (fs/shell/edit first) → first attempt on a small spine-side
packet. Order of ~10 packets from 2026-08-03.

7. MOVEMENT COUPLING (owner core-check, 2026-08-03; PINNED): the FIRST
   M3 parity packet ships agent LOCATION + the ADR-010 movement law +
   origin_path activation + f_loc + movement-refresh WITH fs/shell —
   never as a later flourish. An agent gets hands and a place to stand
   in the same breath. The Context Bars omission (v2.46) is the
   cautionary tale.
