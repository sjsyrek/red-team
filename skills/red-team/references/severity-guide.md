# Severity calibration guide

Severity reflects **real-world exploitability**, not theoretical CVSS. The rubric below is the enforcement mechanism — when in doubt, calibrate against these examples, not against a numeric score.

## The four-axis test

For each finding, ask:

1. **Auth required to exploit?** None / user-level / admin-level / network-position / physical access
2. **User interaction required?** None / one click / multiple clicks / specific timing
3. **Blast radius?** Single record / single user / class of users / all users / system-wide
4. **Reversibility?** Trivially reversible / requires rollback / data loss / persistent compromise

Severity is the join of these axes, not a sum.

## Rubric

### CRITICAL

At least one of:
- **Unauthenticated** + **system-wide blast** + **persistent**: e.g., RCE on a public endpoint, unauth SSRF reaching cloud metadata, unauth auth-bypass to admin role.
- **Unauthenticated** + **PII exposure of all users**: e.g., IDOR on `/users/:id` with no auth, customer DB dump via unprotected admin API.
- **Auth required but trivial** + **persistent system compromise**: e.g., any authenticated user can become admin via known privilege-escalation step.

Examples:
- IDOR on `/users/:id/orders` exposing other users' order history with only a valid session — even though auth is required, the auth is trivially obtainable (any signup) and the blast radius is all users → CRITICAL.
- Unsanitized input flowing into a dynamic-code-execution sink (interpreter `eval`, dynamic `Function` constructor, `vm.runInThisContext`) reachable from a public webhook → CRITICAL.

### HIGH

- **Auth required (admin)** + **persistent compromise**: e.g., SQLi behind admin login (compromised admin → full DB).
- **Auth required (user)** + **single-class blast**: e.g., privilege escalation from `editor` to `admin` role.
- **Unauthenticated** + **single-user impact** + **persistent**: e.g., stored XSS reaching one user's session if they view a specific resource.
- **Secrets exposure** in source: e.g., production API key committed to repo.

Examples:
- SQLi behind admin auth — compromised admin account is a real but bounded prerequisite → HIGH (not CRITICAL).
- Stored XSS in user profile rendering for any other user who views the profile → HIGH.

### MEDIUM

- **Auth required + user interaction required + bounded blast**: e.g., reflected XSS that needs the user to click a crafted link.
- **Information disclosure** without direct exploitation: e.g., stack traces in 500 responses, version banners on public endpoints.
- **Rate-limit gaps** without a directly chainable exploit: e.g., login endpoint accepts unlimited attempts but the credential format makes brute force impractical.
- **Unsafe defaults** that require additional steps to exploit: e.g., CSRF protection disabled but no exploitable state-changing endpoint.

### LOW

- **Defense-in-depth gaps** with no current exploit path: e.g., missing CSP header on a JSON-only API, missing HSTS in dev environment.
- **Hardening recommendations** that aren't bugs: e.g., bcrypt cost factor 10 (modern recommendation is 12).
- **Best-practice deviations** without exploitability: e.g., using `crypto.randomUUID()` somewhere a sequential ID would suffice.

### INFO

- **Clean checks**: domain reviewed, no issues found. (See SKILL.md for why clean checks are real results.)
- **Observations** that aren't findings: notes for the team's awareness without action recommendation.

## Common calibration mistakes

- **Inflating severity by chaining hypothetical attacks.** Severity is the finding as observed. Chains live in the Compound Risk section with their own combined-severity field — don't pre-bake the chain into individual finding severity.
- **Deflating severity because "the auth is good".** If auth is trivially obtainable (any signup), treat it as effectively unauthenticated for blast-radius purposes.
- **Borrowing CVSS scores wholesale.** CVSS is a vendor framework optimized for unknown attacker contexts. Yours is a known context — use the four-axis test against the actual deployment.
- **Missing the data-classification axis.** Exposure of email addresses is different from exposure of payment data. When PII or PCI data is involved, push the severity up by one level.
