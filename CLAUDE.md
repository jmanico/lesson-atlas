# Agent operating rules

## AG-01 — Read the owner before changing a rule

| File | Ownership |
| --- | --- |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Functional/non-functional behavior and acceptance. |
| [DESIGN.md](DESIGN.md) | Brand, visual tokens, frontend conventions, accessibility targets. |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Components, interfaces, data flow, technology decisions. |
| [SECURITY.md](SECURITY.md) | All security requirements, threats, and boundary enforcement. |

Author each rule once in its owning file; elsewhere cite its ID. Security takes precedence for overlapping domains. Preserve stable IDs or provide a canonical-reference alias when consolidating them. Update the owner before implementing changed behavior. Issues and meaningful acceptance tests cite their owning IDs.

## AG-02 — Issue authoring

Every new GitHub issue must follow [REQUIREMENT_TEMPLATE.md](REQUIREMENT_TEMPLATE.md). Issue bodies reference canonical requirements; do not create a competing specification.

Use one template per atomic issue. Replace placeholders; use `N/A` only after evaluating applicability, and `TO BE DECIDED` for unresolved values. Normative terms follow RFC 2119/8174 (BCP 14). Split compound requirements in the owning specification. Issue acceptance criteria reference its observable pass/fail outcomes using Given/When/Then or equivalent, with preconditions, action, and result; do not add behavior in rationale or test criteria. Include only verified, versioned standards mappings; never guess IDs.

For a decomposition issue, list every child exactly once, in build order, under **Child Requirements**, as `- [ ] <child issue URL> — <child Metadata Title verbatim>`. A child belongs to only one parent and is not repeated under **Downstream Requirements**. Omit the child field on leaf issues. A parent cannot reach `Verified` until all children do.

## AG-03 — Implementation and verification

Build shared tokens/components before screens. Run formatting, lint, type checks, unit/domain tests, OpenAPI compatibility, localization checks, and accessibility automation when implemented. Integration tests use PostgreSQL/Redis containers; end-to-end tests cover applicable acceptance criteria. Follow SECURITY.md SEC-VERIFY-01 for security gates and fixture policy. A feature is complete only when its applicable criteria, persistence, failure states, and history/proposal behavior pass.

Build, test, and run commands: TO BE DECIDED; this repository currently contains specifications and artwork, with no executable application.

## AG-04 — Enforcement configuration

If `.claude/` enforcement is added, agents, commands, and hooks must read owning files at runtime and cite enforced rule IDs, without restating them. `.claude/settings.json` contains only permission, sandbox, and hook wiring. None is present yet.

## AG-05 — Secure-coding reference maintenance

For relevant implementation work, read the local Node.js, React, and shared JavaScript guidance below, applying only rules for features actually in use. SECURITY.md owns the application controls; resolve differences there rather than creating a second policy in agent files.

- `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge/content/js/nodejs/26/prompt.md`
- `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge/content/js/react/19/prompt.md`
- `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge/content/js/javascript/ES2026/prompt.md`
