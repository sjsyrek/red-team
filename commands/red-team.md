---
description: Adversarial security review by independent specialist attacker agents
argument-hint: [scope-path]
---

Invoke the `red-team` skill on the current project.

**Scope**: $ARGUMENTS

If the scope above is empty, treat the entire repository at the current working directory as the review scope. If the scope is a path, narrow the recon and attack fan-out to that subtree only — but still load `CLAUDE.md` and dependency manifests from the repo root, since trust boundaries and supply-chain risks cross subtree boundaries.

Begin with Phase 0 (plan card) before spawning any agents. Do not skip the plan card even though the slash command was explicit; the user still needs to confirm roster and budget.
