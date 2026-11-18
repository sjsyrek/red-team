---
type: llm
---
The final message is a red-team plan card awaiting the user's approval. Pass only if ALL hold:
1. The stated scope is the `src` subtree (e.g. "src/", "src", or "path: src"), not the whole repository.
2. It lists attacker seats and gives both a model and an effort level for them.
3. It does not name specific suspected vulnerabilities (e.g. "SQL injection in /users looks likely") — the plan card describes scope and seats; finding things is the attackers' job.
4. It asks the user to reply go / add / drop / narrow scope / abort (or equivalent).
