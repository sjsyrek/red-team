# Report template

The Commander writes this in Phase 4. Preview to chat first, then save to `~/.claude/red-team/<yyyy-mm-dd>-<slug>/report.md` on user confirmation.

## Skeleton

```markdown
---
status: complete | halted
date: YYYY-MM-DD
scope: <repo root | path>
agents_run: [input-attacker, auth-attacker, ...]
agents_failed: []
proactive_trigger: <reason or null>
---

# Red Team Report: <Application Name>

**Date**: YYYY-MM-DD
**Scope**: <what was reviewed>
**Agents**: <list of seats that ran>

---

## Executive Summary

<2–4 sentences. What's the overall risk posture? What's the worst realistic attack scenario? What must be fixed before the next deploy?>

---

## Compound Risk Chains

<Only if cross-correlation found chains. For each:>

### CHAIN: <Title>
**Components**: <finding-id-A> + <finding-id-B>
**Combined Severity**: CRITICAL
**Attack Narrative**: <prose description of how an attacker chains these. 3–5 sentences.>

---

## Critical & High Findings

<For each critical or high finding:>

### [SEVERITY] [SURFACE]: <One-line title>
**ID**: `<finding-id>`
**CWE**: CWE-XXX
**Finding**: <1–2 sentence description>
**Reproduction**:
1. <Step one>
2. <Step two>
3. <Expected vs actual outcome>

**Evidence**: `path/to/file.ts:42` — <relevant snippet or output>
**Remediation**: <Concrete fix, not "add validation". E.g.: "Add parameterized queries using the ORM's `.where()` API instead of string interpolation on line 42.">

---

## Medium Findings

<Condensed format: id, title, one-line description, file reference, remediation sketch>

---

## Low / Informational Findings

<Bullet list: id — surface — finding — file reference>

---

## Clean Checks

<Bullet list of domains where agents found nothing. A clean check is a real result.>

- `auth-attacker`: No findings. Reviewed JWT verification, session handling, OAuth callback validation.
- `state-attacker`: No findings. Reviewed payment flow idempotency, queue consumer locking, and inventory write paths.
- ...

---

## Remediation Priority

1. <Most urgent — fix before deploy>
2. <Next sprint>
3. <Backlog — low risk, worth addressing>

---

## Scope Gaps

<What was NOT reviewed and why. Helps distinguish "safe" from "unchecked".>

- <surface> — <reason: out of scope, agent failed to spawn, requires runtime access, etc.>
```

## Notes

- The **Compound Risk Chains** section comes before individual findings because the chain narrative is the highest-signal output of the cross-correlation phase. A chain of two mediums combining into a critical is precisely the value-add over a single-context review.
- The **Clean Checks** section is mandatory. Omission is ambiguous; an explicit "we looked here and found nothing" gives the team confidence and helps future reviews skip already-cleared surfaces.
- The **Scope Gaps** section is the audit trail for "we never looked here." If an attacker failed to spawn or the user narrowed the scope, log it.
