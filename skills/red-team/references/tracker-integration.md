# Tracker integration

Optional, auto-detected. The skill's core protocol runs identically with or without a tracker. Most users have no tracker configured — that's the default path, not the exceptional one.

## What's supported today

**First-class: [beads](https://github.com/gastownhall/beads)** — because it exposes a `bd` CLI and a memory system that compose cleanly with the Commander's tool surface and the parallel-attacker fan-out.

**Other trackers** (GitHub Issues via `gh`, GitLab via `glab`, Linear, Jira) can be wired by following the adapter contract at the bottom of this file. The skill does not ship adapters for them yet; contributions welcome.

## Beads detection

```
test -d .beads || command -v bd
```

Run from the invoking project's git root. Either succeeds → beads is this project's tracker.

## What the skill does when beads is detected

### Phase 1 — Recon enrichment

The Commander gathers tracker context **once** and inlines it into the shared `~/.claude/red-team/<slug>/recon.md`. Every attacker reads the recon, so every attacker sees the tracker context — at zero per-spawn token cost beyond the prompt-cache miss on first read.

```bash
# Security-relevant memories — keyword filter to stay bounded
bd memories | grep -iE 'security|vuln|auth|inject|secret|crypto|xss|csrf|idor|ssrf'

# Open beads — security-labeled if the project uses labels, else top open
bd list --status=open --label=security 2>/dev/null || bd ready

# Beads the user explicitly named in the trigger phrase or scope
bd show <id>     # for each id mentioned
```

Each result block lands under a new `## Known issues from tracker` section in the recon brief. **Attackers are instructed to check this section before reporting a finding** — if a finding matches an existing bead, the attacker either skips it or reports it as a `chain_potential` extension referencing the existing bead's id, never as a fresh finding.

### Phase 1 — Skill self-audit

```bash
# Memories that name the red-team skill itself
bd memories | grep -iE 'red-team|red team|incident commander|attacker|recon'

# Emergent-insights appendices from prior reports on this machine
ls -1 ~/.claude/red-team/*/report.md 2>/dev/null | while read f; do
  awk '/^## Appendix — Emergent insights/,0' "$f" | grep -v '^## '
done
```

If any surfaced entry directly contradicts a prescription in the current `SKILL.md` or this file, **memory is ground truth**. Memory is written after real sessions fail; the skill's text is written to prevent the next failure. When they disagree, the skill is stale.

The Commander's response to a detected contradiction:

1. Flag the contradiction in `recon.md` so every attacker sees it.
2. **Follow memory, not the skill's stale prescription.**
3. Record the contradiction in the report's `Appendix — Emergent insights` and write `bd remember "<lesson>"` so the next run picks it up.

### Phase 4 — Finding filing (after user `save`)

After the user replies `save` and the report file is written, the Commander files findings as beads. **Order matters**: components first so chains can link to them.

```bash
# 1. Per Critical / High / Medium finding — note that components are filed first
bd create \
  --title="<one-line finding title>" \
  --description="<repro + evidence + remediation, copied from the report>" \
  --type=bug \
  --priority=<severity-mapped>

# 2. Per Compound Risk Chain — file as a parent bead, then link components
bd create \
  --title="<chain title>" \
  --description="<attack narrative + component finding ids>" \
  --type=bug \
  --priority=0
bd dep add <chain-bead-id> <component-bead-id>   # repeat for each component
```

Filed bead ids are written back to the on-disk report:

- `primary-tracker-ids` frontmatter list — every bead this run filed.
- `linked-tracker-ids` frontmatter list — existing beads referenced by findings (from the recon enrichment).
- Per-finding `**Tracker ID**` field — set to the just-filed bead id.

### Phase 4 — Silent-promise guard

Before declaring the report finalized, the Commander greps the saved file. **Every Critical and High finding must have a non-empty `**Tracker ID**`** when beads is detected. If a Tracker ID is missing:

- File the bead now and update the report, **or**
- Demote the finding to Medium/Low (with the Commander's rationale recorded in the report) and re-run the guard.

Do not exit Phase 4 with unfiled C/H findings. A C/H finding without a tracker handle is exactly the silent-promise rot this guard exists to prevent.

### Phase 4 — Emergent-insights write-back

If the run surfaced a structural lesson — a SKILL.md prescription contradicted by reality, a recon brief that produced false trust boundaries, an attacker that consistently re-files known beads despite the recon enrichment — the Commander writes it via:

```bash
bd remember "<one-line lesson, ≤200 chars>"
```

The next red-team run reads this during Phase 1 self-audit. The skill compounds.

## Severity → bd priority map

| Report severity | `bd --priority` | Filed in 0.2.0? | Notes |
|---|---|---|---|
| critical | 0 | yes | Compound chains use 0 by default |
| high | 1 | yes | |
| medium | 2 | yes | |
| low | 3 | no | Future `--file-low` opt-in |
| info / clean check | — | no | Stays prose-only in the report |

## Finding type map

| Report element | `bd --type` |
|---|---|
| Vulnerability finding | `bug` |
| Compound risk chain | `bug` |
| Remediation work item from "Remediation Priority" section | `task` (only when explicitly distinct from a bug fix) |

## Compound chain handling

Chains are real beads, not just notation. Filing order:

1. File every component finding first → capture component bead ids.
2. File the chain as its own bead at priority one tier above the highest component (in practice, almost always priority 0).
3. `bd dep add <chain-bead> <component-bead>` for each component.

The chain bead now blocks-on every component bead. Closing a component does not close the chain — the chain stays open until every component is closed and the chain itself is reviewed for residual risk. This mirrors how a real attack requires the components to remain fixable: closing the SQLi but not the auth bypass leaves the chain partially live.

## Two-list discipline

red-team does **not** call `TeamCreate` (attackers run as `Agent(... run_in_background: true)` with no inter-agent communication). There is no session-scoped task list at `~/.claude/teams/<team>/` to confuse with persistent beads.

This simplifies the discipline compared to design-council:

- **Session-scoped state** lives in `~/.claude/red-team/<slug>/` (recon brief, report draft). Cleaned up or kept as artifact at user discretion.
- **Persistent state** lives in beads. Filed only at Phase 4 save, never during the attack phase.

Never file beads from inside an attacker. Filing is a Commander-only action, gated by the user's `save` reply on the report preview.

## When no tracker is detected (common case)

Tracker IDs stay null:

- `recon.md` has no "Known issues from tracker" section.
- Report frontmatter shows `tracker: none`, `primary-tracker-ids: []`, `linked-tracker-ids: []`.
- Per-finding `**Tracker ID**` reads `none`.
- The silent-promise guard is a no-op (nothing to file).
- The Commander does **not** invent commands for a tracker that isn't there.

Users with other trackers port the pattern manually (`gh issue create`, `glab issue create`, etc.) — the skill's durable contract is the report markdown's finding list; tracker integration is a convenience layer on top.

## Stop-early behavior

"Stop the red team" at any phase aborts before filing. Partial reports (`status: halted`) never file beads — the work is incomplete by definition. The Commander explicitly tells the user "no beads filed because the run was halted" so the silence isn't ambiguous.

## Adapter contract (for future trackers)

Each tracker adapter should expose five hooks:

1. **`detect()`** — returns true if this tracker is active in the current project.
2. **`fetch_for_recon()`** — returns text to inline in `recon.md` (security-relevant items, ready queue, referenced items).
3. **`fetch_for_self_audit()`** — returns memory entries (or equivalent) that name the red-team skill.
4. **`translate_finding(severity, title, description)`** — files the finding, returns the new tracker id.
5. **`link_chain(parent_id, component_ids)`** — links a chain bead to its components. No-ops if the tracker has no dependency model (the chain is then logged as a comment on the parent).

Implementations live in `references/tracker-adapters/<name>.md`. Today only beads is documented (in this file). PRs welcome for `gh`, `glab`, Linear, Jira.
