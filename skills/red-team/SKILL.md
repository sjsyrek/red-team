---
name: red-team
description: Adversarial security and robustness review. Spawn independent specialist attacker agents to find every possible way to break an application — security vulnerabilities, logic flaws, race conditions, dependency risks, infrastructure misconfigs, and more. Use when the user says "red team this", "try to break it", "find vulnerabilities", "security audit", "attack surface", "what could go wrong", "pen test", or "stress test". Also triggers proactively (with plan-card pause for approval) when shipping code that handles authentication, payments, user-supplied input, external APIs, or sensitive data — even if the user didn't ask. Attack surface doesn't correlate with lines of code; invoke for small apps too. Do NOT skip when the user says "it's probably fine" — that's exactly when to run.
version: 0.2.0
---

# red-team

A multi-agent adversarial review skill. Independent specialist agents attack the application from distinct angles with **no shared context during the attack phase**; the Incident Commander cross-correlates findings afterward to surface compound risks no single agent would catch.

## What the user sees

1. **Plan card.** Before any agent spawns, the Commander shows a one-screen card: scope, roster (always-on plus opt-in seats with rationale for each opt-in), per-seat model, rough token/wall-clock budget. Reply `go`, `add llm-attacker`, `drop performance-attacker`, `narrow scope to src/auth`, or `abort`.
2. **Handshake status.** After spawn: `HANDSHAKE: 8/8 ok | started=[...] | failed=[]`. Spawn health visible, not inferred.
3. **Silent attack phase.** Agents run in parallel with no inter-agent communication. The Commander surfaces per-agent progress notifications.
4. **Report preview.** Before any file is written, the Commander posts the draft report to chat. Reply `save`, `amend <note>`, or `discard`.
5. **Output.** Final report at `~/.claude/red-team/<yyyy-mm-dd>-<slug>/report.md` (outside the repo under review).
6. **Stop early.** Say "stop the red team" at any phase — Commander cancels in-flight agents, collects whatever findings are already in, writes a partial report with `status: halted` and `cancelled_at: <phase>`.

## When to invoke

Trigger phrases: "red team this", "try to break it", "find vulnerabilities", "security audit", "pen test", "attack surface", "what could go wrong", "stress test".

**Proactive triggers** (pause for plan-card approval, do not run unprompted):
- Code that handles auth tokens, sessions, or passwords
- Payment or billing flows
- User-supplied input reaching a database, shell, or template engine
- External API integrations with secrets in scope
- New public endpoints

Attack surface doesn't correlate with lines of code; invoke for small apps too. Do **not** skip when the user says "it's probably fine" — that's exactly when to run.

**Do NOT invoke** for code-style review, doc-only changes, bug fixes that don't touch trust boundaries, or pure refactors. The token cost isn't earned.

## Phase 0 — Plan card

Before any spawn, post the following to chat. Wait for the user's reply.

```
RED TEAM PLAN
=============
Scope:      [repo root | path/to/subtree]
Always-on:  input-attacker, auth-attacker, state-attacker,
            dependency-attacker, logic-attacker, infra-attacker,
            data-attacker, performance-attacker
Opt-in:     [list with one-line rationale per seat]
Models:     [per-seat — Opus default for synthesis-heavy, Sonnet for analytical]
Budget:     ~Nk tokens, ~M minutes wall-clock
Output:     ~/.claude/red-team/<date>-<slug>/report.md

Reply: go | add <seat> | drop <seat> | narrow scope to <path> | abort
```

User commands:
- `go` — proceed to Phase 1
- `add <seat>` / `drop <seat>` — adjust roster
- `narrow scope to <path>` — re-recon with tighter scope
- `abort` — stop, no spawn, no artifact

## Phase 1 — Recon (Commander, single context)

Build the recon brief. This is the only shared context across all attackers — it must be precise and bounded.

Steps:

0. **Tracker detection (fails-safe).** Run `test -d .beads || command -v bd` from the project's git root. If either succeeds, beads is this run's tracker — gather context per `references/tracker-integration.md` (Phase 1 recon enrichment + skill self-audit) and inline into a `## Known issues from tracker` section of `recon.md`. If neither succeeds, skip silently — no tracker commands are invented. **When self-audit detects a memory entry that contradicts a prescription in this file, memory wins**; flag the contradiction in `recon.md` so every attacker sees it, follow memory, and record the drift in the report's `Appendix — Emergent insights`.
1. Read `CLAUDE.md` if present — extract architecture constraints, known risks, out-of-scope areas.
2. Scan project structure:
   - Tech stack (languages, frameworks, ORMs, auth libraries)
   - Entry points (HTTP routes, CLI args, message consumers, cron jobs, webhooks)
   - Data stores (databases, caches, file systems, external APIs)
   - Trust boundaries (user-controlled vs internal-only)
3. Check for existing security documentation (threat models, prior audit reports, post-incident write-ups).
4. Identify which opt-in seats apply.

Write the brief to `~/.claude/red-team/<slug>/recon.md`. Every attacker's spawn prompt points to this path so identical reads hit the prompt cache across parallel spawns.

Brief format: see `references/recon-template.md`. Tracker enrichment recipe: `references/tracker-integration.md`.

## Phase 2 — Attack fan-out (parallel)

Spawn all selected agents simultaneously in **one multi-tool-call message** via `Agent(... run_in_background: true)`. Each agent receives:

- Its **role brief** loaded from `references/agents/<slug>.md`
- A `Read` instruction for the recon brief at `~/.claude/red-team/<slug>/recon.md`
- The **finding schema** (below)
- Full tool access (file reads, bash, static analysis)
- A **tracker-awareness rule** (when the recon brief contains a `## Known issues from tracker` section): before reporting any finding, check the listed beads. If your finding matches an existing bead, either skip it or report it with `chain_potential: true` and the bead id in the `evidence` field — never refile as a fresh finding.

**Critical**: Agents do not communicate during this phase. Independence is the whole point. Shared context contaminates findings. Filing beads is a Commander-only action at Phase 4 save — attackers never call `bd`.

### Finding schema

Each agent emits one JSON block per finding:

```json
{
  "id": "<surface-slug>-<6char-hex>",
  "severity": "critical | high | medium | low | info",
  "surface": "the component or layer attacked",
  "finding": "one-sentence description of the vulnerability or flaw",
  "reproduction": "step-by-step path to trigger the issue",
  "evidence": "file path, line number, code snippet, or test output",
  "cwe": "CWE-XXX or null",
  "chain_potential": true
}
```

The `id` field is required — Phase 3 cross-correlation references findings by id when assembling chains. Use the surface name plus a short hex hash of the evidence (e.g., `auth-7f3a2b`).

If an attacker finds nothing in its domain, it must emit one `info`-severity entry naming what was checked:

```json
{
  "id": "auth-clean-001",
  "severity": "info",
  "surface": "authentication",
  "finding": "No findings. Reviewed JWT verification, session handling, and OAuth callback validation.",
  ...
}
```

A clean check is a real result. Omission is ambiguous — did no agent check it, or did they check and find nothing?

## Phase 2.5 — Handshake verify

After spawn, count incoming acknowledgements. Emit:

```
HANDSHAKE: N/N ok | started=[...] | failed=[...]
```

For any failed attacker:
- Retry once with the same role brief
- If second spawn fails, proceed and list the failed surface in the report's "Scope Gaps" section

## Phase 3 — Cross-correlation (Commander)

After all attackers report back, the Commander cross-correlates before writing the report. **This is the phase that catches compound risks.**

### Algorithm

1. Collect all findings into a single list, keyed by `id`.
2. For each finding where `chain_potential: true`, ask: which other findings does this combine with to produce a higher-severity outcome?
3. Common compound patterns:
   - Auth bypass + data exfiltration → **critical path to data breach**
   - Missing rate limit + timing oracle → **credential enumeration**
   - IDOR + privilege escalation → **full account takeover**
   - Unvalidated input + logging finding → **log injection + blind spot**
   - Secrets in env + SSRF → **cloud credential theft**
   - Race condition + financial logic flaw → **double-spend**
4. For each chain, create a compound finding with:
   - `severity`: one level above the highest component finding
   - `chain`: list of component finding `id`s
   - `attack_narrative`: prose description of how an attacker executes the chain end-to-end (3–5 sentences)

## Phase 4 — Report

Skeleton: `references/report-template.md`. Severity calibration: `references/severity-guide.md`. Tracker filing recipe: `references/tracker-integration.md`.

### 4a — Preview before write

Post the draft report to chat:

```
[draft report markdown]

Reply: save | amend <note> | discard
```

On `save`: write to `~/.claude/red-team/<yyyy-mm-dd>-<slug>/report.md` and proceed to 4b. On `amend`: revise per the note, re-preview. On `discard`: clean up artifacts, no file written, no beads filed.

The path is **per-user artifact space, outside the repo under review**. Attack reproductions should not land in git history by accident.

### 4b — File findings to tracker (when beads is detected)

After the report file is written, the Commander files findings as beads per `references/tracker-integration.md`:

1. **Component findings first.** For each Critical / High / Medium finding: `bd create --type=bug --priority=<severity-mapped>` where critical=0, high=1, medium=2. Capture the new bead id.
2. **Compound chains second.** Each chain files as its own `--type=bug --priority=0` bead, then `bd dep add <chain-bead> <component-bead>` for every component in the chain. The chain blocks-on its components.
3. **Update the on-disk report.** Populate frontmatter `primary-tracker-ids` (this run's filings), `linked-tracker-ids` (existing beads referenced by findings), `tracker: beads`. Set each finding's `**Tracker ID**` to the just-filed bead id.

When beads is **not** detected: skip 4b entirely. Frontmatter stays as `tracker: none`, IDs stay null, the report markdown is the only handle.

### 4c — Silent-promise guard

Before declaring the report finalized, grep the saved file. **Every Critical and High finding must have a non-empty `**Tracker ID**`** when beads is detected. Missing IDs:

- File the bead now and update the report, **or**
- Demote the finding's severity (with rationale recorded in the report) and re-run the guard.

Do not exit Phase 4 with unfiled C/H findings. A C/H finding without a tracker handle is exactly the silent-promise rot this guard prevents.

### 4d — Emergent-insights write-back

If the run surfaced a structural lesson — a SKILL.md prescription contradicted by reality, a recon brief that produced false trust boundaries, an attacker that consistently re-reports known beads despite the recon enrichment — write it via `bd remember "<lesson, ≤200 chars>"` so the next run picks it up during Phase 1 self-audit. The lesson also lands in the report's `Appendix — Emergent insights`.

Skipped when beads is not detected; the appendix entry is the only handle.

## Stop early

User says "stop the red team" at any phase:

1. Broadcast cancel to all in-flight agents
2. Collect whatever findings are already in
3. Skip cross-correlation (or run abbreviated version on whatever is collected)
4. Write partial report with frontmatter `status: halted`, `cancelled_at: <phase>`, `cancellation_reason: user_request`
5. **Skip Phase 4b filing.** Partial reports never file beads — the work is incomplete by definition. Tell the user explicitly: "no beads filed because the run was halted." Silence here is ambiguous; an explicit notice isn't.
6. Clean up

The partial report is still useful — it tells you what was checked before you stopped, and what wasn't.

## Roster

### Always-on

| Slug | Mission |
|---|---|
| `input-attacker` | Subvert behavior via user-controlled input |
| `auth-attacker` | Bypass authentication / authorization |
| `state-attacker` | Exploit concurrent or sequential state inconsistency |
| `dependency-attacker` | Find CVEs / supply-chain risks in third-party code |
| `logic-attacker` | Find business-logic invariant violations |
| `infra-attacker` | Find deployment / runtime misconfigurations |
| `data-attacker` | Find data-shape and validation gaps |
| `performance-attacker` | Find resource-exhaustion / DoS paths |

### Opt-in (Commander adds during recon based on detected context)

| Slug | Add when |
|---|---|
| `llm-attacker` | LLM/AI components detected (OpenAI, Anthropic, local models, embeddings) |
| `api-attacker` | REST or GraphQL API with external consumers |
| `mobile-attacker` | Mobile client (iOS/Android) in scope |

Per-agent attack methodology lives in `references/agents/<slug>.md`. **Load only when writing that agent's spawn prompt** — keeps the always-loaded SKILL context lean.

## Notes for the Incident Commander

**On severity calibration.** Use real-world exploitability, not theoretical CVSS. A SQL injection behind admin auth that requires a compromised admin is HIGH, not CRITICAL. An IDOR exposing PII with no auth is CRITICAL regardless of how "simple" the app is. See `references/severity-guide.md`.

**On evidence standards.** Agents must cite specific file paths and line numbers. "There might be an injection risk in the query builder" is not a finding. "Line 142 of `src/db/users.ts` interpolates `req.query.id` directly into a SQL string" is a finding.

**On scope.** If the user provided scope boundaries (via plan card or `narrow scope to`), enforce them strictly. Do not report findings the team can't fix (vulnerabilities in third-party hosted services, etc.).

**On clean checks.** A domain with no findings goes in the "Clean Checks" section of the report, not omitted. Omission is ambiguous — did no agent check it, or did they check and find nothing?

**On remediation.** Every critical and high finding needs a concrete, actionable remediation. Not "validate inputs" but "use `zod.string().max(255).email()` on the `email` field at line 67 of `src/routes/auth.ts`." Low findings can be more general.

**On proactive triggering.** When triggered proactively (not from a user trigger phrase), the plan card includes the **proactive trigger reason** — e.g., "Detected new public endpoint at `routes/api/v2/payments.ts`". The user can `defer` (records as a known unaudited surface, will not re-trigger on this surface for 24h) or `go`.

**On tracker integration.** When beads is detected, Critical/High/Medium findings auto-file as `--type=bug` beads after the user `save`s the report; compound chains file as parent beads with `bd dep add` linkage. Existing security beads surface during recon so attackers don't re-report them. Full recipe in `references/tracker-integration.md`. Fails-safe — projects without beads behave identically to projects without any tracker awareness.

## Reference files

- `references/agents/<slug>.md` — Per-agent attack methodology + spawn brief (load when writing that agent's spawn prompt)
- `references/severity-guide.md` — Exploitability-based severity rubric
- `references/cwe-quick-ref.md` — Common CWE IDs mapped to attack surface for quick tagging
- `references/recon-template.md` — Phase 1 brief format
- `references/report-template.md` — Phase 4 markdown skeleton
- `references/tracker-integration.md` — Beads detection, severity → priority map, finding-filing recipe, silent-promise guard, adapter contract
