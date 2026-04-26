# Changelog

All notable changes to the red-team plugin are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] — 2026-04-26

Initial release. Adversarial review skill with eight always-on attacker agents and three opt-in seats. Mirrors the design-council commander/specialist pattern, but adversarially framed: agents do not communicate during the attack phase; the Incident Commander cross-correlates findings afterward to surface compound risks no single agent would catch.

### Added

- **Eight always-on attacker agents**: `input-attacker`, `auth-attacker`, `state-attacker`, `dependency-attacker`, `logic-attacker`, `infra-attacker`, `data-attacker`, `performance-attacker`.
- **Three opt-in attacker seats** added by the Commander based on detected context: `llm-attacker` (when LLM/AI components present), `api-attacker` (REST/GraphQL APIs with external consumers), `mobile-attacker` (iOS/Android client in scope).
- **Phase 0 plan card** so the user reviews scope, roster, opt-in rationale, and budget before any agent spawns.
- **Phase 2.5 handshake status** confirming all attacker agents started.
- **Cross-correlation phase** (Phase 3) that synthesizes compound risk chains from independent findings — auth bypass + data exfiltration → critical breach path, IDOR + privilege escalation → account takeover, etc.
- **Phase 4 report preview** — Commander posts the draft report to chat for `save` / `amend` / `discard` before writing the file.
- **"Stop the red team" early-termination path** at any phase, with partial-report write and `status: halted`.
- **`/red-team [scope]` slash command** for explicit invocation with optional path scope.
- **Per-agent reference files** in `references/agents/` (loaded only when spawning that agent — token-efficient).
- **`references/severity-guide.md`** — exploitability-based severity rubric, not theoretical CVSS.
- **`references/cwe-quick-ref.md`** — CWE catalog mapped to attack surfaces.
- **`references/recon-template.md`** — Phase 1 brief skeleton.
- **`references/report-template.md`** — Phase 4 markdown skeleton.
- **Reports written to `~/.claude/red-team/<yyyy-mm-dd>-<slug>/report.md`** — per-user artifact space, outside the repo under review (attack reproductions should not land in git history by accident).
