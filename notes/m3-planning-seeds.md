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
- shell/fs/read/edit → ADOPT an existing OSS toolset base, never from
  scratch (owner, v2.48; candidates: Code Puppy, OpenCode, Hermes,
  OpenHands, aider, pydantic-ai natives) — fork/pin, modify under our
  law; the walls and policy stay ours;
- browser → the OpenAI/Codex CHROME EXTENSION PORT is the DEFAULT
  (owner, v2.48); Playwright pinned as the headless fallback;
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

## Census layer 2 (2026-08-03): inside the shell + MCP inventory

WHAT THE SHELL CALLS ACTUALLY RUN (Claude Code, first tokens; cd-chains
undercount payloads): navigation (cd 350, ls 22), search (grep 53, find
9), text surgery (sed 35, cat 9), runtimes (uv 33 — the test runner —
python3 21), git 20, gcloud 23, docker 8, shell constructs (for/export).
THE INNER TOOLKIT for a self-building agent = navigate, search, read,
patch, run tests, commit — plus cloud/docker episodically. This is the
shell-tool policy + walls design input: what the sandbox must permit
freely (navigate/search/read/test) vs journal (git write ops) vs
boundary-card (gcloud, docker, network).
MCP INVENTORY: ZERO user-configured MCP servers on this machine — all
MCP usage is platform-builtin (browser). Finding: the real dependency
set is small and mostly NOT MCP; MCP client support (pinned SDK) is a
door to keep open, not a load-bearing wall to build first.

8. PACKET SIZING LESSON (gate, 2026-08-04): M2N consumed SIX sessions
   (five honest RETURNED_TODO handoffs + one close) — lawful under the
   clean-unfinished-handoff rule, but a sizing smell. M3 planning should
   split lifecycle-grade packets (backup/restore/locks/matrix/upgrader
   were five packets wearing one id). Also: r8 tags on the board REQUIRE
   SOP artifacts (M2M's unbacked tag corrected at the gate); the M2
   SCOUT is the assembled-product user pass that per-packet SOPs can't
   provide. Amendment fold debt now A-020..A-046 (~27 entries) — the
   pre-M3 editor pass is genuinely due after the M2 judge.

9. REMOTE-INIT GAP (found live, 2026-08-05, owner's M2 spin startup):
   `nocturne init` only mints a fresh LOCAL palace; there is no "init
   against an existing spine" (SPINE_URL + token), so the owner's cloud
   rung rides the pre-D3 .env path with the raw daemon entry point.
   The mirror-principle packet needs exactly this: init --remote
   <spine-url> (prompting for the bearer), making rung 2 a first-class
   CLI citizen and retiring the .env-sourcing ritual.

10. AGENT TESTS — the new tier (owner riff, 2026-08-05; refine after M2
    gate-day experience per owner): unit tests verify CODE, integration
    tests verify CONTRACTS, AGENT TESTS verify EXPERIENCE — a standing,
    versioned suite of human-style verifications (open/click/watch/
    judge, real browser, screenshots as evidence, PASS/FAIL/NEEDS-TASTE)
    re-executed at defined moments. We have the organs (r8 per-packet
    SOPs, r9 scout, r12 motivation, judge re-execution); the NEW element
    is PERMANENCE: each UI packet CONTRIBUTES its SOP into a living
    verification/agent-tests/ suite (motivation-cited per r12) instead
    of retiring it as one-shot evidence. Tiered like human regression:
    agent-smoke (core loop, minutes, at big-feature merges) vs full
    suite (gate days, judgments). The company-SOP analogy is the right
    frame; costs are real (every run is real browser + real tokens per
    rule 10) so the tiers matter. Candidate law: B.6 rule 13 at M3
    planning, seeded from the existing H5/M2 SOPs.
