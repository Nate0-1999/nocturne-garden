# Tool evidence dossier (2026-08-03) — provenance, lists, sources

The owner asked for evidence, not summaries. Everything here is
reproducible; run the commands yourself.

## 1. LOG PROVENANCE — exactly what was mined

- CLAUDE CODE: ~/.claude/projects/*/[session].jsonl — 5 session files,
  42MB. Tool calls are `"type":"tool_use"` entries. Repro:
    cat ~/.claude/projects/*/*.jsonl | grep -o
    '"type":"tool_use","id":"[^"]*","name":"[^"]*"' |
    grep -o '"name":"[^"]*"' | sort | uniq -c | sort -rn
- CODEX: ~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl (90 files) +
  ~/.codex/archived_sessions/ (174 files); 343MB total. Caveat: uniform
  ~2373 counts across many names are per-session tool SCHEMAS, not
  calls; only counts far above that baseline are real invocations.
- Aggregate metadata only; no conversation content was read.

## 2. THE LISTS (as measured)

CLAUDE CODE tool calls: Bash 685 · Edit 432 · Read 177 · browser
(computer) 36 · Write 32 · WebFetch 18 · AskUserQuestion 18 · Artifact
16 · TaskUpdate 15 · browser-navigate 14 · WebSearch 12 · TaskCreate 8 ·
Agent 8 · ToolSearch 7 · Skill 6.
SKILLS invoked (total 6): dataviz ×2, artifact-design ×2, xlsx ×2.
INSIDE the Bash calls (first tokens; cd-chains undercount payloads):
cd 350 · grep 53 · sed 35 · uv 33 · gcloud 23 · ls 22 · python3 21 ·
git 20 · find 9 · cat 9 · docker 8.
CODEX: exec ≈10k+ real calls (shell is everything); builtin browser in
SOP packets; no external MCP.
MCP SERVERS user-configured on this machine: NONE (all MCP usage is
app-builtin browser tooling).

## 3. EXISTING PLANS — where each already lives

- ADR-013 (LAW): adopt / wrap / PIN — never rebuild commodities. The
  owner's "static but versioned to open source" directive verbatim.
- ADR-007 (LAW): the tool-parity index.
- ADR-015 (LAW): walls — sandbox-first; hands and walls ship together.
- v2.38 (LAW): image input committed with the parity wave.
- notes/gate-day-feedback.md: the owner's original shopping list
  (Codex browser-extension port, Agent Skills, MCP, CodeMode,
  background tasks, compaction batteries, sub-agent archetypes, cost
  introspection).
- notes/m3-planning-seeds.md: items 1 (survey + anchors), 7 (movement
  coupling), census layers 1-2, N1 Ouroboros.

## 4. OPEN-SOURCE SOURCES — honest status: NAMED, NOT YET READ

The anchors were chosen from knowledge; cloning + reading the repos is
the M3 survey's first act (gate can start early on request).

| capability | source repo | license | pin form | what we take |
|---|---|---|---|---|
| tool framework + MCP client | pydantic/pydantic-ai | MIT | ALREADY PINNED: pydantic-ai==2.12.0 (harness pyproject) | function tools, MCP client |
| shell/fs/read/edit tools | (our own thin tools) | — | versioned by our wheel | policy + walls are the work, not the plumbing |
| browser use | microsoft/playwright-python | Apache-2.0 | exact pin | driver + page control; enables in-harness rule-8 SOPs |
| browser alt | openai/codex (chrome extension) | Apache-2.0 | port, not dep | owner-named option; evaluate vs Playwright at survey |
| search | BurntSushi/ripgrep | MIT/Unlicense | pinned binary | rg |
| web fetch | encode/httpx | BSD | already a dep (>=0.27,<1 → tighten at M3) | fetch |
| MCP | modelcontextprotocol/python-sdk | MIT | exact pin | client, door-open only (zero-MCP finding) |
| skills | Agent Skills standard (SKILL.md) | open spec | spec, not dep | loader per ADR-007 |
| durable execution | dbos-inc/dbos-transact-py | MIT | exact pin (M3) | checkpointing (Code Puppy prior art) |
| NOT copied ever | our envelope, gate, palace, scorer, queue | — | — | differentiators are never adopted |

## 5. Delta note: NOCTURNE ≠ Codex ≠ Claude Code

Our setup differs on purpose: memory-first (gate/palace wrap every
tool), walls-not-questions (no per-action approvals), the ledger prices
every call, movement law gives agents locations, and the rack renders
what other harnesses bury in logs. The census tells us WHAT verbs to
support; our law dictates HOW they're held.
