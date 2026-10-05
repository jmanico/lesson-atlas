# LessonAtlas: Security Architecture and Secure Coding Standard

Version: 1.0\
Date: 2026-10-05\
Status: Implementation security baseline\
Companions: REQUIREMENTS.md 1.2, ARCHITECTURE.md 1.1, DESIGN.md 1.0

## 1. Authority, scope, and source material

This document defines enforceable security controls for the React web client, Node.js REST API and workers, identity federation, PostgreSQL, Redis, object storage, AI processing, and deployment. REQUIREMENTS.md remains authoritative for product scope and acceptance. The initial release has teacher accounts only; students and guardians are records without accounts or direct access (A-03). The student authentication options in Section 4.5 are a future design constraint, not permission to build a student portal or expose student records to students now.

The implementation must apply these local secure-coding references: `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge/content/js/nodejs/26/prompt.md`, `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge/content/js/react/19/prompt.md`, and their shared `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge/content/js/javascript/ES2026/prompt.md` baseline. This file translates those prompts into LessonAtlas controls and adds identity, data, and operational rules. Where a prompt covers a feature this app does not use, the rule is conditional rather than a reason to add that feature.

Node.js 26 is still a Current release on this document's date according to the [Node.js release schedule](https://nodejs.org/en/about/previous-releases). Production uses a supported LTS line, initially Node.js 24, until Node.js 26 reaches LTS and passes compatibility and security tests. Node.js 26-specific rules below become mandatory at that upgrade. React 19 and `react-dom` must use matching, patched versions. No security statement here is a claim of legal or standards certification.

## 2. Threat model and trust boundaries

| Boundary | Principal risk | Authoritative control |
| --- | --- | --- |
| Browser to CDN/API | Stolen session, CSRF, malicious input, injection, denial of service | TLS, secure session, CSRF/origin checks, schema validation, rate limits |
| School IdP to application | Forged or weak federation event, account takeover, incorrect tenant mapping | Pinned OIDC connection, token validation, assurance verification, explicit subject binding |
| API to PostgreSQL | Cross-workspace disclosure, stale writes, injection, tampered academic history | Server policy, scoped queries, constraints, transactions, parameterized SQL |
| API/worker to Redis and object storage | Cache or export leakage, replay, unauthorized object access | Workspace and region scoping, short TTL, private buckets, signed access, idempotency |
| Jobs to AI provider | Prompt injection, overbroad data sharing, provider retention, fabricated evidence | Minimized authorized retrieval, approved provider, bounded request, output validation, teacher acceptance |
| Operations to production | Secret leakage, privileged support access, unsafe deployment | Managed secrets, least privilege, isolated environments, image provenance, audit |

Assume client state, URLs, files, CSVs, notes, standards text, model responses, identity assertions, queue messages, and network responses can be attacker controlled or stale. A valid teacher session does not authorize another workspace or every operation within the teacher's own workspace. The server must decide each requested action against current state.

## 3. Security rules and ownership

Rules use stable `SECURITY.md` IDs for requirements and test traceability. The API team owns request, authorization, data, and session rules; the web team owns browser rules; the platform team owns deployment and operations; the privacy owner approves data-sharing and retention policy. A bypass needs a documented product/security decision and updated requirements if behavior changes.

### 3.1 Identity and teacher authentication

**SEC-AUTH-01 — Teacher authenticator assurance.** Every teacher sign-in and reauthentication must use a passkey with user verification or a stronger phishing-resistant cryptographic authenticator. A synced passkey with user verification is acceptable; a hardware-bound key may be required by a future institutional policy. Password alone, password plus OTP, email link, SMS, push approval, or an OIDC event whose method cannot be verified does not meet this application policy. This is a product requirement stricter than merely offering a phishing-resistant option. NIST describes password and OTP as non-phishing-resistant and recognizes properly configured WebAuthn authenticators as phishing-resistant in [SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b.html). Applies to ACCESS-04.

**SEC-AUTH-02 — Federation connection.** A school or organization may configure an approved OpenID Connect provider. Register an explicit issuer, client ID, redirect URI, allowed signing algorithms, key source, organization/workspace binding, assurance contract, and provisioning policy. Do not select an issuer from an untrusted email domain or request parameter. Each connection has an owner-approved lifecycle; disable it without deleting audit history. Federation does not create shared teaching or district-administrator access (ACCESS-05).

**SEC-AUTH-03 — OIDC transaction.** Use Authorization Code Flow with PKCE; validate `state`, `nonce`, exact issuer and audience, signature/key, expiry, issuance time, authorized party where relevant, and the token endpoint's TLS identity. Reject unsigned tokens, algorithm confusion, unknown issuers, unsolicited callbacks, nonce replay, and missing required claims. Bind local identity to stable `(issuer, sub)` rather than email; email is a changeable attribute and must not auto-link accounts. Perform explicit, audited account linking when needed. Use the [OpenID Connect Core specification](https://openid.net/specs/openid-connect-core-1_0.html) as the protocol authority.

**SEC-AUTH-04 — Federation assurance evidence.** The connection contract must specify an approved `acr` value and/or trustworthy `amr` evidence plus provider configuration proving user-verified passkey or stronger authentication for this event. The application validates the returned evidence and authentication time against that contract on every session establishment and required step-up. `amr` strings alone are not a universal passkey guarantee; providers use different vocabularies, so each mapping needs integration tests and security review. A provider unable to supply verifiable assurance cannot be used for teacher login. An already-established weak IdP session cannot silently satisfy the policy.

**SEC-AUTH-05 — Enrollment and recovery.** Passkey enrollment, additional authenticator binding, device replacement, and account recovery require a comparably strong verified ceremony, user notification, audit event, and revocation of compromised sessions. Help desk identity assertions or email ownership alone may start recovery but cannot finish it and issue a teacher session. Lock or hold an account when strong recovery cannot be completed; record an institution-approved manual process before production. Accessibility and backup-device plans must be reviewed so mandatory passkeys do not strand teachers.

**SEC-AUTH-06 — Session lifecycle.** After successful assurance verification, issue only an opaque, server-managed session in an `HttpOnly`, `Secure`, `SameSite` cookie, with session rotation at login and privilege changes. Enforce server-side idle and absolute expiry, session revocation, sign-out, and an authentication-time limit for sensitive operations such as exports or credential changes. A suggested starting policy is 1 hour idle and 24 hours absolute, subject to customer risk review; do not treat these values as a NIST certification claim. Provider logout and local logout are distinct: local revocation is immediate even if provider logout fails. No bearer token or refresh token goes into JavaScript-accessible storage (ACCESS-03).

**SEC-AUTH-07 — Machine identities.** API, workers, queue dispatchers, and maintenance jobs use separate short-lived service credentials and least-privilege roles. They cannot impersonate a teacher by setting a header. Each job carries authenticated initiating actor, workspace, purpose, and immutable source references, which the worker reauthorizes or validates against an approved system policy before processing.

### 3.2 Authorization and tenant isolation

**SEC-AZ-01 — Server policy.** Every protected REST operation, history read, search, export, object download, AI retrieval, and worker action requires server-side policy evaluation of principal, workspace, action, resource, and current state. Client routes, hidden buttons, Redux values, and feature flags are not authorization boundaries. Default deny on missing or ambiguous context. Treat opaque IDs as identifiers only (ACCESS-01–02, SEC-02).

**SEC-AZ-02 — Data scoping.** Every owned SQL row and relationship has workspace scope. Queries include it, foreign keys prevent cross-workspace joins, and object keys/cache keys include region and workspace. PostgreSQL row-level security may add defense in depth but cannot replace application policy. Tests must attempt cross-workspace access through IDs, filters, search, pagination, imports, exports, revisions, AI jobs, and stale queue messages. Denials must not reveal whether another workspace's record exists.

**SEC-AZ-03 — Change integrity.** All authoritative edits use version checks and transactional constraints. AI proposals are never authority; accepting selected changes rechecks permissions, exact source versions, field allowlists, and evidence references within the write transaction. Retry keys prevent duplicate imports and acceptances. History and corrections have the same policy as current records (HISTORY-01–06, AI-01–05).

### 3.3 React 19 and browser code

**SEC-JS-01 — Shared JavaScript boundary.** Validate untrusted values at runtime with allowlisted schemas that reject unknown security-sensitive fields. Check type, byte length, and encoding before normalization; validate the normalized value and use only that representation afterward. Treat parsed JSON as untyped, bound bytes/depth/collection sizes, and reject duplicate security-sensitive keys when parsers could disagree. Reject prototype-pollution keys before object merging; use `Map` or null-prototype dictionaries for attacker-controlled keys and `Object.hasOwn` for ownership decisions. Bound numeric values, regex input and complexity, and date formats. Never turn data into code through `eval`, `Function`, string timers, dynamic imports, revivers, or prototype reconstruction. Await security decisions and fail closed on rejection or timeout.

**SEC-WEB-01 — Safe rendering.** Render untrusted text as JSX text. Avoid `dangerouslySetInnerHTML`; if rich content is needed, sanitize with a maintained allowlist at a reviewed sink and never append raw fragments afterward. Keep untrusted content out of inline scripts/styles, raw SVG/MathML, event-handler strings, and direct DOM HTML parsing. Apply a restrictive CSP and Trusted Types where supported; neither replaces sanitization. Validate URL schemes and approved origins for links, images, forms, and navigation. React `preinit` and `preinitModule` receive only build-controlled URLs.

**SEC-WEB-02 — Props and state.** Never spread untrusted objects into DOM elements or privileged components. Map dynamic components through an exact registry. Treat Redux, RTK Query, URL parameters, refs, context, and all browser storage as untrusted. Clear private in-memory caches on logout, tenant/account switch, and session expiry. Tag asynchronous responses with identity/workspace generation so a late response cannot overwrite a newer session or show another workspace's data. No student content or tokens in local storage, IndexedDB, or service-worker cache.

**SEC-WEB-03 — Server rendering if introduced.** The reference architecture is a client-rendered Vite app. If SSR or React Server Functions are later added, each function reachable by the client is a remote endpoint and must authenticate, authorize, and validate arguments independently. Server render state is request scoped; protected cache keys include identity/workspace. Bootstrap scripts come from a trusted manifest, hydration payloads exclude secrets and other users' data, and any inline bootstrap JSON uses script-safe serialization. CSP nonces are per response. Do not hide hydration mismatches with `suppressHydrationWarning` without investigation. SSR adoption requires a security design review first.

**SEC-WEB-04 — Error handling and dependencies.** Client error boundaries display generic messages and correlation IDs, not stacks, tokens, raw API responses, or sensitive records. Pin matching patched `react`/`react-dom` versions and review renderers, build tools, component HTML escape hatches, and any `react-server-dom-*` package before use. Verify malicious markup, URL, prop, stale-state, and cross-user-cache cases in tests.

**SEC-WEB-05 — Cookie-backed mutation protection.** Protect every state-changing endpoint, including logout, import confirmation, proposal acceptance, and federation configuration, with server-validated CSRF defenses and origin checks. `SameSite` is defense in depth, not the sole control. Use `GET` only for safe retrieval. Reject cross-origin credentialed requests unless an explicit trusted origin and operation are configured. Test forged form and fetch requests against authenticated teacher sessions.

### 3.4 Node.js API and worker code

**SEC-NODE-01 — Runtime and startup.** Pin an exact patched Node.js release in Docker, CI, and `engines`; verify `process.version` at startup. Use Node.js 24 LTS until the reviewed Node.js 26 transition. At that transition, start with Node's permission model and per-workload grants where compatible, understanding it is not a sandbox; container and network isolation remain mandatory. Audit permissions in test environments before enforcement. Deployment-controlled startup flags, config, preloads, and env files must be immutable to the application user. Validate and freeze security configuration at startup; fail closed on missing values. Keep warnings visible. Test with deprecations treated as failures.

**SEC-NODE-02 — HTTP ingress.** Use strict HTTP parsing; never enable `insecureHTTPParser` or lenient HTTP/2 handling. Align proxy and Node limits for headers, body size, request/header/idle timeout, and concurrent connections or streams. Trust forwarding headers only from enumerated proxies. Destroy timed-out sockets/streams and bound requests per socket. Reject CR/LF in response headers, cookies, and redirects. Request IDs are generated or validated at ingress and cannot carry untrusted content into logs.

**SEC-NODE-03 — Validation and database access.** Validate every URL/path/query/body/CSV/job message with runtime schemas; TypeScript types are not validation. Apply limits to count, size, nesting, expansion ratio, and processing time. Use parameterized PostgreSQL queries and allowlisted sort/filter fields. Never derive an executable path, dynamic import, SQL identifier, object-storage path, or filesystem location directly from request data. Make failures generic and non-mutating.

**SEC-NODE-04 — Outbound and file operations.** Outbound HTTP uses fixed destinations or parsed allowlisted protocol, host, and port; reject redirects or revalidate each hop to prevent SSRF. Set connect, total, idle, and concurrency limits; preserve TLS verification. Bound input and decompressed bytes, stream with backpressure, and clean up on abort. For temporary files use unpredictable private directories, restrictive modes, exclusive creation, path confinement, and symlink defenses; client filenames are presentation data only. Avoid child processes; if unavoidable, use fixed executable and argument array with no shell, minimal environment, output/time limits, and reviewed privileges.

**SEC-NODE-05 — Crypto and runtime isolation.** Use a vetted identity/crypto library and managed key service where possible. Generate randomness from Node/Web Crypto CSPRNG; do not use weak hashes for password verifiers or tokens. Authenticate ciphertext before acting on plaintext. Never disable certificate validation. Do not use `node:vm` as an isolation boundary, dynamic evaluation, experimental FFI, or unnecessary native addons for untrusted content. Workers handling untrusted files run with separate OS identity, read-only root filesystem, CPU/memory limits, and egress policy. Diagnostic reports, heap snapshots, inspector, and core dumps are production secrets and are disabled or tightly restricted.

**SEC-NODE-06 — Failure handling.** Unhandled exceptions and rejections produce sanitized telemetry and terminate the process; orchestration restarts it. Graceful shutdown stops new requests, drains in-flight work to a deadline, and closes DB, queue, and file handles. Health endpoints disclose no runtime inventory or secrets. Long-running jobs have bounded retries, cancellation, and idempotent side effects.

### 3.5 Data, AI, and privacy

**SEC-DATA-01 — Classification and minimization.** Treat student identity, contacts, attendance, scores, notes, profiles, lesson support, AI prompts/outputs, exports, and history as restricted educational data. Keep contacts in dedicated APIs. Telemetry uses opaque IDs and excludes record content. Store original score scale and missing/exempt status exactly; integrity is a security requirement as well as an academic one.

**SEC-DATA-02 — Encryption and keys.** TLS protects all external and service connections. Managed encryption protects PostgreSQL, Redis persistence where enabled, object storage, backups, and secrets. Use region-approved KMS keys with role separation, rotation, and audit. Application-level field encryption may be added for especially sensitive contact/support fields after query and recovery impacts are reviewed. Secrets never enter repository, browser bundles, command-line arguments, logs, or images.

**SEC-DATA-03 — Imports and exports.** CSV imports stage, validate, preview, and require explicit confirmation. Validate identifiers, membership, dates, scales, row count, encoding, and duplicate candidates. Uploads have size/type limits and malware scanning where applicable. Exports require explicit field/scope selection, fresh authorization, audit, short-lived private object access, expiration/deletion, and spreadsheet-formula neutralization. Lesson exports omit student-specific notes unless deliberately selected (ROSTER-05, RECORD-06, SEC-08).

**SEC-DATA-04 — AI boundary.** Real student data remains blocked from AI until an approved provider contract, field list, processing region, no-training setting, retention/deletion process, and institution authorization are recorded. The model receives only same-workspace, task-relevant evidence and no contacts. Notes and standards are untrusted data, not instructions. Output uses a strict schema and allowlisted edits, with server validation of references and citations. The model cannot write authoritative records; teacher acceptance is a separate transaction (SEC-05, AI-01–08).

**SEC-DATA-05 — Retention and deletion.** Apply the approved policy to primary records, versions, proposals, exports, caches, projections, object storage, provider copies, and backups. A restored backup must replay the deletion ledger before serving users. Where law requires retained audit metadata, purge student content from that event and enforce the same access control. Verify deletion with synthetic fixtures (SEC-06, HISTORY-06).

**SEC-DATA-06 — Jurisdiction gate.** Before production use with children, establish school/institution authority, customer and vendor roles, target ages, jurisdiction, consent/notice basis, retention, incident response, subprocessors, and cross-border transfer rules. Assess FERPA/COPPA for relevant US deployments, GDPR/UK GDPR and child-specific rules where applicable, and local equivalents elsewhere. Chinese localization does not itself authorize processing data in China. Legal counsel and institutional policy determine applicability; do not claim blanket compliance from these technical controls.

### 3.6 Infrastructure, supply chain, and monitoring

**SEC-OPS-01 — Deployment.** Build immutable, minimal Docker images from pinned lockfiles, scan dependencies and images, generate an SBOM, and review install scripts/addons. Run as non-root with read-only root filesystem and only necessary writable mounts. Separate API and worker service identities. CI/CD uses short-lived credentials, protected branches, reviewed infrastructure code, signed or otherwise verifiable image provenance, staged rollout, and rollback plan. No real student data in CI or demos.

**SEC-OPS-02 — Network and storage.** Use regional private networks for PostgreSQL, Redis, and object storage; block direct internet access. Restrict worker egress to approved providers and update endpoints. Separate regions and environments, enforce private buckets, rotate credentials, and deny cross-region failover unless approved by the residency policy. Rate limit by account/workspace and costly operation; protect against noisy neighbors without putting student identifiers in metrics.

**SEC-OPS-03 — Audit and detection.** Record sign-in success/failure, assurance failure, passkey enrollment/recovery, session revocation, federation configuration, authorization denial, sensitive export, import confirmation, AI disclosure/acceptance, deletion, and privileged operational changes. Include timestamp, actor/service, workspace, action, outcome, and correlation ID. Never log credentials, cookies, tokens, raw prompts, student content, contacts, exports, or full request/response bodies. Protect audit integrity and access, set a policy retention period, and alert on repeated assurance failures, cross-workspace probes, bulk exports, unusual recovery, deletion failure, and backup/restore failure.

**SEC-OPS-04 — Incident and resilience.** Maintain runbooks for account compromise, IdP compromise, token/key exposure, data disclosure, malicious import, AI-provider incident, queue loss, and regional outage. Support immediate session revocation, connection disablement, key rotation, job cancellation, and scoped data access investigation. Backups and restores are encrypted and tested; restore must preserve deletion obligations. Notification and reporting timelines are set by applicable contracts and law, not guessed here.

### 3.7 OWASP verification baseline

Use [OWASP ASVS 5.0.0](https://github.com/OWASP/ASVS/tree/master/5.0/en) as the application-security verification catalog. The release review must assess all applicable Level 1 and Level 2 requirements, record evidence and explicit non-applicability, and track failures as release defects. This is a target and review method, not a claim that this specification or the future app is ASVS verified. IDs below were checked against the cited 5.0 source; they identify especially relevant controls and do not replace the full applicable-control review.

| LessonAtlas rule | Verified ASVS 5.0.0 mapping | Evidence to retain |
| --- | --- | --- |
| SEC-AUTH-03–04 | [V10.1.2, V10.2.1–2, V10.5.1–4](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x19-V10-OAuth-and-OIDC.md) | Transaction binding, mix-up, nonce, stable subject, issuer, and audience negative tests. |
| SEC-AUTH-06 | [V7.1.1–3, V7.2.1–4, V7.4.1–2, V7.6.1](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x16-V7-Session-Management.md) | Documented timeout/federation policy, session rotation, revocation, and disabled-account tests. |
| SEC-AZ-01–02 | [V8.2.1–3, V8.3.1, V8.4.1](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x17-V8-Authorization.md) | Function, object, field-level, and cross-tenant denial tests at the service boundary. |
| SEC-WEB-01, SEC-NODE-03 | [V1.2.1–4](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x10-V1-Encoding-and-Sanitization.md) | Context-safe output, URL validation, and parameterized-query tests. |
| SEC-DATA-01–05 | [V14.2.1–4](https://github.com/OWASP/ASVS/blob/master/5.0/en/0x23-V14-Data-Protection.md) | No sensitive URL data, reviewed cache behavior, no tracker disclosure, and data-class protection evidence. |

Use the [OWASP Cheat Sheet Series](https://github.com/OWASP/CheatSheetSeries/tree/master/cheatsheets) as implementation guidance, with the following pages reviewed when the corresponding component is built: [Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html), [Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html), [OAuth 2.0](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html), [Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html), [Cross-Site Scripting Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html), [Cross-Site Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html), [Server Side Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html), [Node.js Security](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html), and [Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html). A cheat sheet guides implementation; the explicit LessonAtlas rule and its tests determine release acceptance.

## 4. Authentication workflows

### 4.1 School/organization federation setup

1. An authorized organization onboarding process records the school identity, permitted workspace(s), IdP issuer, client registration, keys, callback, and assurance contract. This configuration is performed through a controlled operational workflow until school administration is in product scope.
2. Security review verifies the provider's passkey/user-verification enforcement, claim meanings, signing-key rotation, recovery policy, outage behavior, and account lifecycle. A test account proves both accepted strong login and rejected weak login.
3. Connection activation is audited. A teacher is bound to an existing approved workspace via `(issuer, sub)` and explicit invitation/provisioning rule. Display name or email alone does not grant access.

### 4.2 Teacher sign-in and session establishment

1. The browser begins login against a configured connection. The server creates short-lived state, nonce, and PKCE verifier tied to the browser session and exact redirect URI.
2. The IdP authenticates using a user-verified passkey or approved stronger authenticator and returns the authorization code.
3. The API exchanges the code server-side, validates the OIDC transaction and provider-specific assurance evidence, checks the teacher's workspace binding and account status, then rotates/creates the local session. Any missing or weak evidence denies the session with a generic explanation and a correlation ID.
4. Sensitive operations may require recent reauthentication at the same assurance level. A weak or expired upstream session never downgrades the local policy.

### 4.3 Sign-out, revocation, and provider outage

Local sign-out revokes the server session before optional provider logout. A revoked session is rejected across all replicas. On provider outage, active sessions continue only until their configured limits and policy allow; new sign-ins and required step-up fail closed. Session-store unavailability must not authorize requests. Record a safe user-visible failure, preserve only in-memory unsaved work until the page closes, and do not persist private drafts in browser storage.

### 4.4 Teacher account recovery

The recovery process verifies identity through a reviewed institution-assisted ceremony and binds a new passkey or stronger authenticator before access resumes. Recovery cannot be completed solely with email, a password, or SMS. Revoke prior sessions and compromised authenticators, notify the teacher through an approved channel, and audit the reason, verifier, and outcome without recording biometric or secret material. Define operational staffing and accessibility alternatives before launch.

### 4.5 Future student accounts: recorded policy, not initial scope

The product owner listed three possible student authentication choices: **password**, **password or passkey**, or **MFA**. These are alternatives for a future student-login design, not three simultaneous requirements and not authorization to add student access under A-03. Before implementation, revise REQUIREMENTS.md to choose an option by age, jurisdiction, school policy, and threat model; define the student's permitted resources, guardian/school provisioning, recovery, session limits, and accessibility. Do not reuse teacher sessions or teacher assurance claims for students.

| Future option | Minimum design decision before use |
| --- | --- |
| Password | Approved age-appropriate password policy, breached/common password blocking, slow salted verifier, rate limiting, recovery, and school authorization. It does not satisfy teacher policy. |
| Password or passkey | Both enrollment paths, equivalent account binding and recovery, and a clear assurance policy for sensitive student actions. The weaker option governs overall account takeover risk. |
| MFA | Specify actual factors and whether phishing resistance is required; “MFA” alone does not define assurance. Password plus OTP is not equivalent to passkey authentication. |

## 5. Verification gates and evidence

| Gate | Required evidence |
| --- | --- |
| Identity | Positive and negative OIDC tests for issuer, signature, audience, state, nonce, PKCE, expiry, `(issuer, sub)` binding, assurance mapping, weak IdP session, recovery, revocation, and provider outage. |
| Isolation | Automated cross-workspace tests for every resource family, including jobs, history, exports, object links, caches, and AI retrieval; no data or existence leak. |
| Client | Malicious JSX/rich text/URL/prop fixtures, CSP review, browser cache inspection, logout/account-switch race tests, and React dependency advisory review. |
| API/runtime | Fuzzed schema and CSV limits, request smuggling/timeout configuration review, SSRF and TLS negative cases, safe path/file handling, runtime flags, and deprecation tests. |
| Data integrity | Stale write and stale proposal rejection, atomic acceptance, idempotent import/reminder replay, traceable correction, deletion replay after backup restoration. |
| Privacy | Approved jurisdiction and institution record, provider data terms, telemetry redaction samples, export scope review, retention/deletion exercise. |
| Operations | SBOM and scan results, secrets and image review, restore exercise, incident drill, alert routing, and production configuration sign-off. |
| OWASP ASVS | Applicability register and test evidence for each applicable ASVS 5.0.0 Level 1 and Level 2 control, with reviewed reasons for exclusions and tracked defects. |

All tests use synthetic student data. Security-critical decisions and failure paths need positive and negative coverage. A failed gate blocks real-data production use unless the product owner, security owner, and relevant institution explicitly document an exception that is compatible with REQUIREMENTS.md and applicable law.

## 6. References and change control

Protocol and assurance references: [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html), [NIST SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b.html), [W3C WebAuthn](https://www.w3.org/TR/webauthn-3/), and [Node.js releases](https://nodejs.org/en/about/previous-releases). Privacy applicability references are linked in ARCHITECTURE.md. These sources informed the architecture; any legal or formal conformance claim requires separate assessment.

Review this file when identity provider, supported runtime, student access model, AI provider, region, or retention policy changes. Security rules are implementation requirements for the current product scope; product behavior changes require an update to REQUIREMENTS.md first.

### Change log

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 2026-10-05 | Initial security baseline, teacher passkey and school OIDC policy, conditional student authentication options, Node.js/React secure-coding rules, and OWASP ASVS/Cheat Sheet guidance. |
