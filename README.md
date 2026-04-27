# Red Team

> Independent specialist Claude agents attack your app in parallel — the Incident Commander cross-correlates findings to surface compound risks no single review catches.

## What problem it solves

Single-context security reviews suffer from tunnel vision: whichever angle you start from biases everything that follows. Iterating with the same context doesn't fix this — each turn inherits the prior framing, and the model's attention has already collapsed onto the threats it surfaced first. Genuine adversarial review needs structurally independent vantage points: the SQL-injection mindset and the race-condition mindset don't share priors.

Red Team spawns each attacker as an **independent Claude agent with its own context**. The `auth-attacker` doesn't see the `state-attacker`'s working theories; the `dependency-attacker` isn't anchored on the `infra-attacker`'s findings. After the parallel attack phase, the Incident Commander cross-correlates the independent findings to find the compound risks — chains where two medium findings combine into a critical one.

## How it works

0. **Plan card (you confirm before anything spawns).** The Commander shows a one-screen card: scope, agent roster (always-on plus opt-in seats with rationale), rough token/wall-clock budget. You reply `go`, `add llm-attacker`, `drop performance-attacker`, `narrow scope to src/auth`, or `abort`.
1. **Recon (Commander, single context).** The Commander scans the project: tech stack, entry points, data stores, trust boundaries, key dependencies, opt-in seat triggers. The recon brief lands in `~/.claude/red-team/<slug>/recon.md`. Every spawn prompt points to that path so the recon hits the prompt cache across parallel attacker spawns.
2. **Attack fan-out (parallel).** In one multi-tool-call message, the Commander spawns every selected attacker via `Agent(... run_in_background: true)`. Each receives only its own role brief and the recon — **no inter-agent communication during the attack phase**. Independence is the whole point.
3. **Handshake verify.** The Commander emits one structured line — `HANDSHAKE: 8/8 ok | started=[...] | failed=[]` — so spawn health is visible, not inferred.
4. **Cross-correlation (Commander).** Findings flagged `chain_potential: true` are tested against each other. Auth bypass + data exfiltration → critical breach path. IDOR + privilege escalation → account takeover. Race condition + financial logic flaw → double-spend. Compound findings inherit a severity bump from the highest component finding.
5. **Report preview + write.** The Commander posts the draft report to chat for `save` / `amend` / `discard`. On save: `~/.claude/red-team/<yyyy-mm-dd>-<slug>/report.md` (outside the repo under review).

**Stop early** at any phase by saying "stop the red team" — Commander cancels in-flight agents, collects whatever findings are already in, writes a partial report with `status: halted`.

## Benefits

- **Genuine independence.** Each attacker is a separate Claude with its own context — no priming bleeds between offensive angles. You catch bugs the original author and a single-context reviewer would both miss.
- **Compound-risk discovery.** The cross-correlation phase finds chains that no individual finding would surface. Two mediums combining into a critical is the most valuable output of this pattern.
- **First-class beads tracker integration.** Findings file as `bd` beads with severity-mapped priority; existing security beads surface during recon so attackers don't refile known issues; chains link to component beads via `bd dep add`. See [Tracker integration](#tracker-integration) below.
- **Cheap fan-out via prompt cache.** Recon brief written once, referenced by every spawn — identical reads hit the 5-minute prompt cache, recovering meaningful tokens on multi-attacker reviews.
- **Clean checks are real results.** A domain with no findings is reported as a clean check, not omitted. You can distinguish "we looked here and it's solid" from "we never looked here."
- **Severity rooted in exploitability**, not theoretical CVSS. A SQLi behind admin auth requiring a compromised admin is HIGH, not CRITICAL. An IDOR exposing PII with no auth is CRITICAL regardless of how "simple" the app is.

## Tradeoffs

- **Token cost.** 8+ parallel contexts, each receiving role brief + recon brief + tool access for static analysis. Expect roughly **8–15×** the tokens of a single-context security review. The proactive triggers list is deliberately bounded so the cost is earned.
- **Wall-clock latency.** Even parallel, the phase ends only when the slowest attacker is in. Static-analysis-heavy seats (`dependency-attacker` running `npm audit`) often dominate.
- **Not a substitute for a real pen-test.** This is design-time / pre-deploy review against the codebase. It does not run the application, throw real packets, or test infrastructure live. Use it as the cheap layer before you pay for a black-box engagement.
- **False-positive surface area.** Eight attackers each looking for problems will sometimes flag non-issues. The exploitability-based severity rubric in `references/severity-guide.md` is the calibration tool — and the user-driven `amend` step in the report preview is the safety valve.

## When to invoke

Trigger phrases: "red team this", "try to break it", "find vulnerabilities", "security audit", "pen test", "attack surface", "what could go wrong", "stress test".

**Proactive triggers** (Commander pauses for plan-card approval, doesn't run unprompted):
- Code that handles auth tokens, sessions, or passwords
- Payment or billing flows
- User-supplied input that reaches a database, shell, or template engine
- External API integrations with secrets in scope
- New public endpoints

Attack surface doesn't correlate with lines of code; invoke for small apps too. Do **not** skip when the user says "it's probably fine" — that's exactly when to run.

**Do not invoke** for: code-style review, doc-only changes, bug fixes that don't touch trust boundaries, or pure refactors. The token cost isn't earned.

## Roster

**Always-on (8):** `input-attacker` · `auth-attacker` · `state-attacker` · `dependency-attacker` · `logic-attacker` · `infra-attacker` · `data-attacker` · `performance-attacker`

**Opt-in (Commander adds during recon):** `llm-attacker` · `api-attacker` · `mobile-attacker`

Per-agent attack methodology lives in [`skills/red-team/references/agents/`](./skills/red-team/references/agents/). Each file is loaded only when that attacker is spawned.

## Tracker integration

red-team integrates with **[beads](https://github.com/gastownhall/beads)** as a first-class tracker. Detection is fails-safe: projects without beads behave identically to projects without any tracker awareness — no commands invented, no errors raised.

When beads is detected at the start of a run (`test -d .beads || command -v bd`), four hooks fire:

1. **Recon enrichment.** `bd memories` (security-keyword filtered), `bd ready`, and `bd show <id>` for any bead the user named are inlined into the shared `recon.md`. **Every attacker reads the recon**, so every attacker sees the tracker context — at zero per-spawn token cost beyond the first read (the rest hit the prompt cache). Attackers are instructed to skip findings that match an existing bead, or report them as `chain_potential` extensions referencing the bead's id, never as fresh findings. This is the single biggest win: no more re-filing the auth bypass that's already filed as `bd-83a`.

2. **Skill self-audit.** The Commander reads `bd memories` for entries that name red-team itself plus the `Appendix — Emergent insights` from prior reports. **If a memory entry contradicts a prescription in `SKILL.md`, memory wins** — it was written after a real session failed; the skill text was written to prevent the next failure. The contradiction lands in `recon.md` and the report's emergent-insights appendix.

3. **Post-save filing.** After you reply `save` on the report preview, the Commander files findings as beads — components first so chains can link to them:

   | Severity | `bd --priority` | Filed |
   |---|---|---|
   | critical | 0 | yes |
   | high | 1 | yes |
   | medium | 2 | yes |
   | low | 3 | no (future opt-in) |
   | info / clean check | — | no |

   Compound chains file as `--type=bug --priority=0` parent beads, then `bd dep add <chain> <component>` for every component. The chain bead now blocks-on its components, mirroring how a real attack requires the components to remain fixable. Filed bead ids land in the report's `primary-tracker-ids` frontmatter and per-finding `**Tracker ID**` field.

4. **Silent-promise guard.** Before declaring the report finalized, the Commander greps the saved file: every Critical and High finding **must** have a non-empty `**Tracker ID**`. Missing ids are filed now or the finding is demoted with rationale. A C/H finding without a tracker handle is exactly the rot this guard prevents — markdown reports outside the repo don't show up in `bd ready` and don't get triaged.

A fifth hook closes the loop: when a structural lesson surfaces during the run (recon brief leaked false trust boundaries, an attacker consistently re-files known beads, a SKILL.md prescription contradicted by reality), the Commander writes it via `bd remember "<lesson>"` so the next run picks it up at self-audit. **The skill compounds.**

Stop-early ("stop the red team") halts before any filing — partial reports never file beads, and the Commander tells you so explicitly. No tracker? You still get the full report; only the bead-handle is absent.

Full recipe, severity → priority map, compound-chain semantics, and the adapter contract for future trackers (`gh`, `glab`, Linear, Jira) live in [`skills/red-team/references/tracker-integration.md`](./skills/red-team/references/tracker-integration.md).

## Slash command

```
/red-team [scope]
```

Optional `scope` argument narrows the review (e.g., `/red-team src/auth`). Without it, the entire repo is in scope.

## Output location

`~/.claude/red-team/<yyyy-mm-dd>-<slug>/report.md` — per-user artifact space, **not** plugin-owned, **not** in your repo. Reports often contain reproduction steps you don't want committed by accident.

## Installation

```
/plugin marketplace add sjsyrek/claude-plugins
/plugin install red-team@sjsyrek
```

The marketplace lives at [sjsyrek/claude-plugins](https://github.com/sjsyrek/claude-plugins) and points at this repo.

## Runtime requirements

- `Agent` with `run_in_background: true` — spawns each attacker as an independent context
- File-system write access to `~/.claude/red-team/`

Some package-audit tools (`npm audit`, `pip-audit`, `bundler-audit`, `govulncheck`) are invoked by `dependency-attacker` if available on the host — graceful degradation if absent.

## Version history

See [CHANGELOG.md](./CHANGELOG.md).

## License

MIT
