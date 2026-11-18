---
type: llm
---
The final message is a red-team plan card that waits for the user's approval before any attacker runs. Pass only if ALL hold:
1. It names the scope (the repository root, or a path within it).
2. It lists the always-on attacker seats (at least input-attacker, auth-attacker and dependency-attacker by name).
3. It gives a model for the seats AND an effort level (per seat or as a stated default for all seats). A model with no effort level fails this item.
4. It gives a rough budget (tokens and/or wall-clock time).
5. It does not name specific suspected vulnerabilities (e.g. "SQL injection in /users looks likely") — the plan card describes scope and seats; finding things is the attackers' job.
6. It ends by asking the user to reply with something like go / add / drop / narrow scope / abort.
