# Changelog

All notable changes to the red-team plugin are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] — 2026-04-27

Beads tracker integration, mirroring the design-council pattern. Auto-detected and fails-safe — projects without beads behave identically to 0.1.0.

### Added

- **Phase 1 tracker detection** (`test -d .beads || command -v bd`). When detected, `bd memories` (security-keyword filtered), `bd ready`, and `bd show <id>` for any user-referenced bead are inlined into a new "Known issues from tracker" section of the shared `recon.md`. Every attacker reads the recon, so every attacker sees the tracker context — at zero per-spawn token cost beyond the prompt-cache miss on first read.
- **Tracker-awareness rule for attackers.** When the recon contains "Known issues from tracker", attackers are instructed to skip findings that match an existing bead — or report them as `chain_potential` extensions referencing the bead's id, never as fresh findings. Single biggest token-cost win on repeat-audited codebases.
- **Phase 1 skill self-audit.** Reads `bd memories` entries naming red-team and `~/.claude/red-team/*/report.md` `Appendix — Emergent insights` sections. Contradictions with `SKILL.md` are flagged in `recon.md` and **memory wins** — written after a real failure beats prose written to prevent the next failure.
- **Phase 4b finding filing.** After the user replies `save` on the report preview, Critical/High/Medium findings file as `bd create --type=bug --priority=<severity-mapped>` (critical=0, high=1, medium=2). Compound chains file as their own `--priority=0` parent bead with `bd dep add <chain> <component>` linkage to every component. Chain blocks-on its components, mirroring how a real attack requires the components to remain fixable.
- **Phase 4c silent-promise guard.** Before report finalization, every Critical and High finding must have a non-empty `**Tracker ID**` when beads is detected. Missing ids are filed now or the finding is demoted with rationale. Prevents the silent-promise rot of a C/H landing only in markdown outside the repo.
- **Phase 4d emergent-insights write-back.** Structural lessons surfaced during a run (recon-brief gaps, repeated-refile patterns, contradictions with SKILL.md) persist via `bd remember "<lesson>"`. The next red-team run picks them up at Phase 1 self-audit. The skill compounds across sessions.
- **`references/tracker-integration.md`** (new). Beads detection idiom, severity → priority map, finding-type map, compound-chain handling, two-list discipline, silent-promise guard, no-tracker fallback, and adapter contract for future trackers (`gh`, `glab`, Linear, Jira).
- **Report frontmatter**: `tracker: beads | none`, `primary-tracker-ids: []` (filed by this run), `linked-tracker-ids: []` (existing beads referenced by findings).
- **Per-finding `**Tracker ID**`** field on Critical, High, and Medium findings and on Compound Risk Chains. Chains additionally list `**Component beads**: [bd-AAA, bd-BBB]`.
- **Severity → priority mapping** documented in both `tracker-integration.md` and `severity-guide.md` (tight: critical=P0, high=P1, medium=P2; calibration drift requires a council or memory entry to surface).
- **Report Appendix — Emergent insights** section. Optional. Captures structural lessons that the run surfaced about the skill itself.

### Changed

- **`recon-template.md`**: new "Known issues from tracker" section between "Out of scope" and "Opt-in seats", with sub-sections for `bd memories`, `bd ready`, user-referenced beads, and self-audit contradictions.
- **`report-template.md`**: tracker frontmatter; per-finding tracker IDs; new `Appendix — Emergent insights` section; finalization notes referencing `tracker-integration.md`.
- **`SKILL.md`**: Phase 1 step 0 (tracker detection + self-audit), Phase 2 attacker tracker-awareness rule, Phase 4 split into 4a–4d (preview / file / guard / write-back), Stop-early skips filing with explicit user notice, new Notes-for-Commander entry on tracker integration, `references/tracker-integration.md` listed alongside other reference files.
- **`README.md`**: new "Tracker integration" section between "Roster" and "Slash command" detailing all five hooks; new benefit bullet linking to it.

### Notes

- **Stop-early** ("stop the red team") halts before any filing. Partial reports (`status: halted`) never file beads. The Commander tells you explicitly so the silence isn't ambiguous.
- **Low and info findings are not auto-filed** in 0.2.0. Clean checks remain explicit prose entries in the Clean Checks section. A future `--file-low` opt-in is on the roadmap.
- **No-tracker projects behave identically to 0.1.0.** Frontmatter shows `tracker: none`, IDs stay empty, the report markdown is the only handle. The Commander does not invent commands for a tracker that isn't there.
- **Filing is Commander-only.** Attackers never call `bd` themselves — they read the recon during the attack phase and emit findings; the Commander files them at Phase 4b after the user `save`s. Two-list discipline is simpler here than in design-council because red-team has no `TeamCreate`.

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
