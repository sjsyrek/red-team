# CWE quick reference

Common CWE IDs mapped to attack surface for fast tagging in the finding `cwe` field. This is a working subset, not exhaustive — when none fits, set `cwe: null` rather than forcing a marginal match.

## Input handling

| CWE | Title | Attacker |
|---|---|---|
| CWE-20 | Improper Input Validation | input |
| CWE-79 | Cross-site Scripting (XSS) | input |
| CWE-89 | SQL Injection | input |
| CWE-77 | Command Injection (generic) | input |
| CWE-78 | OS Command Injection | input |
| CWE-94 | Code Injection | input |
| CWE-95 | Eval Injection | input |
| CWE-1336 | SSTI (Server-Side Template Injection) | input |
| CWE-22 | Path Traversal | input |
| CWE-502 | Deserialization of Untrusted Data | input |
| CWE-1321 | Prototype Pollution | input |
| CWE-190 | Integer Overflow | input |
| CWE-176 | Improper Handling of Unicode Encoding | input |

## Authentication / authorization

| CWE | Title | Attacker |
|---|---|---|
| CWE-287 | Improper Authentication | auth |
| CWE-306 | Missing Authentication for Critical Function | auth |
| CWE-862 | Missing Authorization | auth |
| CWE-863 | Incorrect Authorization | auth |
| CWE-639 | Authorization Bypass via User-Controlled Key (IDOR) | auth |
| CWE-384 | Session Fixation | auth |
| CWE-613 | Insufficient Session Expiration | auth |
| CWE-269 | Improper Privilege Management | auth |
| CWE-345 | Insufficient Verification of Data Authenticity | auth |
| CWE-347 | Improper Verification of Cryptographic Signature (JWT alg:none, etc.) | auth |
| CWE-640 | Weak Password Recovery Mechanism | auth |

## State / concurrency

| CWE | Title | Attacker |
|---|---|---|
| CWE-362 | Race Condition | state |
| CWE-367 | TOCTOU | state |
| CWE-413 | Improper Resource Locking | state |
| CWE-841 | Improper Enforcement of Behavioral Workflow | state |

## Dependency / supply chain

| CWE | Title | Attacker |
|---|---|---|
| CWE-1395 | Dependency on Vulnerable Third-Party Component | dependency |
| CWE-829 | Inclusion of Functionality from Untrusted Control Sphere | dependency |
| CWE-494 | Download of Code Without Integrity Check | dependency |
| CWE-1357 | Reliance on Insufficiently Trustworthy Component | dependency |

## Business logic

| CWE | Title | Attacker |
|---|---|---|
| CWE-840 | Business Logic Errors (parent) | logic |
| CWE-799 | Improper Control of Interaction Frequency | logic |
| CWE-841 | Improper Enforcement of Behavioral Workflow | logic |
| CWE-915 | Improperly Controlled Modification of Dynamically-Determined Object Attributes (mass assignment) | logic / data |

## Infrastructure / configuration

| CWE | Title | Attacker |
|---|---|---|
| CWE-16 | Configuration | infra |
| CWE-200 | Information Exposure (parent) | infra |
| CWE-209 | Information Exposure Through Error Messages | infra |
| CWE-489 | Active Debug Code | infra |
| CWE-942 | Permissive Cross-Domain Policy (CORS) | infra |
| CWE-1004 | Sensitive Cookie Without HttpOnly | infra |
| CWE-693 | Protection Mechanism Failure | infra |
| CWE-732 | Incorrect Permission Assignment | infra |
| CWE-798 | Use of Hard-coded Credentials | infra |
| CWE-256 | Plaintext Storage of a Password | infra |
| CWE-295 | Improper Certificate Validation | infra |
| CWE-326 | Inadequate Encryption Strength | infra |

## Data shape

| CWE | Title | Attacker |
|---|---|---|
| CWE-915 | Mass Assignment | data |
| CWE-707 | Improper Neutralization (parent) | data |
| CWE-704 | Incorrect Type Conversion | data |
| CWE-228 | Improper Handling of Syntactically Invalid Structure | data |
| CWE-770 | Allocation of Resources Without Limits or Throttling | data / performance |

## Performance / availability

| CWE | Title | Attacker |
|---|---|---|
| CWE-400 | Uncontrolled Resource Consumption | performance |
| CWE-770 | Allocation of Resources Without Limits or Throttling | performance |
| CWE-1333 | Inefficient Regular Expression Complexity (ReDoS) | performance |
| CWE-409 | Improper Handling of Highly Compressed Data (zip bombs) | performance |
| CWE-834 | Excessive Iteration | performance |
| CWE-307 | Improper Restriction of Excessive Authentication Attempts | performance / auth |

## API / web service

| CWE | Title | Attacker |
|---|---|---|
| CWE-918 | Server-Side Request Forgery (SSRF) | api |
| CWE-352 | Cross-Site Request Forgery (CSRF) | api |
| CWE-444 | HTTP Request/Response Smuggling | api |
| CWE-444 | Inconsistent Interpretation of HTTP Requests (verb tampering, content-type confusion) | api |

## LLM-specific (no formal CWE yet — use OWASP LLM Top 10 IDs)

| ID | Title | Attacker |
|---|---|---|
| LLM01 | Prompt Injection | llm |
| LLM02 | Insecure Output Handling | llm |
| LLM03 | Training Data Poisoning | llm |
| LLM04 | Model Denial of Service | llm |
| LLM05 | Supply Chain Vulnerabilities | llm |
| LLM06 | Sensitive Information Disclosure | llm |
| LLM07 | Insecure Plugin Design | llm |
| LLM08 | Excessive Agency | llm |
| LLM09 | Overreliance | llm |
| LLM10 | Model Theft | llm |

## Mobile-specific

| CWE | Title | Attacker |
|---|---|---|
| CWE-927 | Use of Implicit Intent for Sensitive Communication (Android) | mobile |
| CWE-919 | Weaknesses in Mobile Applications (parent) | mobile |
| CWE-925 | Improper Verification of Intent by Broadcast Receiver | mobile |
| CWE-200 | Information Exposure (clipboard, screenshots, logs) | mobile |
