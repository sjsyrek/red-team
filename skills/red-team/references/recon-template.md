# Recon brief template

The Commander writes this once, before any agent spawns. Lands at `~/.claude/red-team/<slug>/recon.md`. Every spawn prompt points to this path so identical reads hit the prompt cache across parallel attackers.

## Format

```markdown
# Recon — <application name>

**Generated**: <ISO date>
**Scope**: <repo root | path/to/subtree>
**Trigger**: <user trigger phrase | proactive: <reason>>

## Tech stack
- Languages: <list>
- Frameworks: <list>
- Runtimes: <list with versions if pinned>
- ORMs / query builders: <list>
- Auth libraries: <list>
- Crypto / hashing: <list>

## Entry points
| Method | Path / surface | Auth required | Source file |
|---|---|---|---|
| POST | /api/users | none | src/routes/users.ts:14 |
| GET | /api/users/:id | bearer | src/routes/users.ts:42 |
| (CLI) | bin/import-csv | local | bin/import-csv.ts |
| (consumer) | queue: orders.pending | internal | src/workers/orders.ts |
| (cron) | nightly:cleanup | internal | infra/cron.yml:12 |

## Data stores
- Primary DB: <postgres / mysql / etc.> (<schema location>)
- Caches: <redis / memcached / in-process>
- File storage: <S3 / local / etc.>
- External APIs: <list with auth model>

## Trust boundaries
**User-controlled (untrusted)**:
- HTTP request body, query params, headers, cookies
- File uploads
- Webhook payloads from <providers>

**Internal-only (trusted by design)**:
- Cron triggers
- Internal RPC from <services>
- Database fixtures

## Key dependencies
Notable libraries with outsized blast radius (auth, crypto, parsing, deserialization):
- <pkg> @ <version> — <purpose>
- ...

## Out of scope
From CLAUDE.md / user instruction:
- <item> — <reason>

## Known issues from tracker

(Populated only when beads is detected — `test -d .beads || command -v bd`. Otherwise omit the entire section. Recipe: `references/tracker-integration.md`.)

### From `bd memories` (security-keyword filtered)
- <bd-XXX> — <one-line memory text>

### From `bd ready` (security-labeled or top open beads)
- <bd-XXX> [P<n>] — <title>

### Referenced by user
- <bd-XXX> — <summarized bd show output: title, description, current status>

### Self-audit contradictions (memory wins on conflict with SKILL.md)
- <one-line contradiction> — follow memory; record in report's emergent-insights appendix

## Opt-in seats
- `<seat>` — <one-line rationale based on detected context>
```

## Notes

- The recon must be **precise and bounded**. Vague entries leak into spawn prompts and produce sloppy findings.
- Trust boundaries are the single most important section. If you cannot draw a line between user-controlled and internal-only inputs, recon is incomplete — pause and ask the user before proceeding to fan-out.
- The "Out of scope" section is enforceable: agents are instructed to ignore findings entirely contained in those areas.
- The "Known issues from tracker" section is what prevents attackers from re-discovering already-filed bugs. When present, every attacker is instructed to skip findings that match a listed bead — or report them as `chain_potential` extensions referencing the existing bead's id, never as fresh findings.
