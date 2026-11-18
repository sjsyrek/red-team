---
type: llm
---
The final message is a draft red-team report preview for the code in src/server.js. Pass only if ALL hold:
1. It reports the SQL injection in the GET /users handler (string-interpolated `req.query.name`), citing src/server.js.
2. It reports the hardcoded JWT secret (`JWT_SECRET`) in src/server.js.
3. It reports that POST /admin/refunds has no authentication/authorization check.
4. It contains at least one compound/chained finding that combines two or more component findings.
5. Every critical or high finding has a concrete remediation tied to a file and line or handler.
6. It ends by asking the user to save, amend, or discard (or equivalent) rather than claiming the report was already saved.
