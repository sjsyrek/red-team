# Red Team

> Spawn independent specialist Claude agents to attack your application in parallel — they work without shared context, the Incident Commander cross-correlates their findings, you get a triage report with compound risk chains.

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
