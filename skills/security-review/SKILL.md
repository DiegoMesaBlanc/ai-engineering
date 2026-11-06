---
name: security-review
description: Review application and backend code for exploitable security weaknesses using an evidence bar of trust boundary and observed result, independent verdicts (confirmed, needs_validation, rejected), and severity anchors. Use for security review of a change, a module, or an API, or when code-audit dimension 6 applies. Route browser headers and Lighthouse checks to best-practices and CI/CD pipeline controls to secure-delivery.
---

# Security Review

Find weaknesses that cross a real trust boundary, then report the source
evidence, the demonstrated result, the severity that result justifies, and
the smallest effective fix. This is a defensive, source-first review.

The categories to check are in `skills/vulnerability-catalog/SKILL.md`.
This skill defines how a candidate becomes a finding.

## Routing

| Question | Skill |
|---|---|
| Authorization, injection, authentication, crypto, business logic, APIs, LLM, native code | this skill plus `vulnerability-catalog` |
| Browser headers, CSP, Trusted Types, cookies, Lighthouse | `best-practices` |
| CI/CD pipeline gates, secrets in delivery, artifact integrity | `secure-delivery` |
| Dependency and lockfile hygiene | `secure-delivery` (Phase 3), with the catalog's supply-chain category |

## Evidence bar

A candidate is a finding only when all of these are named:

1. **Principal.** Who can send the input or trigger the action, and their trust level.
2. **Input or action.** What they send or do.
3. **Intended control.** The check that should stop it, and where it lives. A
   control is a control only when every external input it guards reaches it: a
   validator, parser, sanitizer, or guard that exists in the code, or is
   documented, but is not called on the path from the entry point to the sink is
   **unwired** (M-48). An unwired control is
   the vulnerability, not a missing layer: the boundary is unprotected (M-48).
4. **Boundary.** The trust boundary crossed: a user acting on another user's data,
   an unauthenticated caller reaching a privileged function, untrusted data
   reaching an interpreter, and so on.
5. **Result.** What the attacker obtains or changes, observed in code or in a
   local test with dummy data.

Do not report as vulnerabilities:

- a missing best practice with no reachable boundary violation;
- **defense in depth:** if an earlier layer blocks the attack, the absence of a
  later layer is a hardening note, not a finding;
- self-impact, where the principal only affects their own data and holds that
  authority anyway;
- a generic crash or parser error with no security effect;
- deployment, proxy, or provider behavior that is not in the repository.

## Verdicts

- **`confirmed`**: the full evidence bar is met from source, or from a local test
  that exercised the path. It carries a severity.
- **`needs_validation`**: a specific, source-grounded boundary hypothesis that is
  blocked by one exact missing fact: deployment configuration, an external
  service's behavior, an environment that cannot be reproduced locally, or
  execution that is not safe here. Name the fact and the safe way to check it.
  It has **no severity**.
- **`rejected`**: the candidate was disproved. Record why.

Severity is assigned only to `confirmed` findings, and it never exceeds the
impact that was demonstrated.

## Severity anchors

- **critical**: an unauthenticated actor obtains code execution, full data-store
  access, or takeover of arbitrary accounts.
- **high**: an actor fully defeats an explicit control with real consequences:
  authentication bypass, cross-tenant read or write, stored script that runs for
  other users, authenticated code execution, or an unauthenticated stop of a
  shared service.
- **medium**: a real boundary violation with limited scope, uncommon
  preconditions, or consequences confined to a narrow resource set.
- **low**: disclosure of non-secret internals, or an attack that needs sustained
  effort for little gain.
- **informational**: a minimal-impact observation, useful as a step in a larger
  finding.

The high/medium test: does the result **fully defeat** an explicit control for an
action with real consequences, or only weaken it? If you cannot state the
concrete damage, the severity is lower than it feels.

## Execution scope

- Review source by reading it. Do not probe deployed endpoints, shared services,
  production identities, or other people's data.
- Run target code only with dummy principals, dummy data, and no external network,
  and only when the environment can enforce that. Otherwise keep the candidate as
  `needs_validation` and describe the safe check.
- Stop at the minimum effect that establishes the result. Do not build persistence,
  evasion, or post-exploitation material.
- Never print, copy, or store secret values found in the code. Report the file,
  line, and type of secret only.

## Procedure

1. **Map the boundaries.** Identify entry points, authentication and authorization
   checks, data stores, interpreters, outbound calls, and third-party inputs. Use
   the project's architecture notes when they exist.
   - **Entry-point sweep.** When the project has an identity mechanism (a login,
     a session, a token, or a credential), enumerate every entry point of the
     project, not only those in the change, and record for each the identity it
     accepts and the check that verifies it. The identity must come from a
     credential the client cannot forge; one read from a header, query string,
     or body the client controls is not authentication (M-52). A public endpoint
     is recorded as public with its reason. A project with no identity mechanism
     has no sweep requirement; the review states that the mechanism is absent.
2. **Select categories.** Use the catalog to choose the categories that apply to
   the changed surface. Do not run every category against every change.
3. **Build candidates.** For each category, look for the signal and record each
   candidate with the five evidence elements above.
   - **Control wiring.** For every exported or created validator, parser,
     sanitizer, or guard, and for every statement in the repository's
     documentation or README that a boundary is validated, filtered, or
     protected: search the call sites, follow the path from each entry point
     (deep link, notification payload, request, message handler) to the sink, and
     record whether the control is on it. A control with no call on the path is a
     `confirmed` candidate in the boundary's category, with the documentation
     line as evidence when a claim exists. A documentation claim the code does
     not support is reported even when the impact is low, because the claim
     misleads the next reviewer.
   - **Scan coverage.** A secret or content scan that excludes inputs by extension,
     path, or size reports every input it did not read as `NOT VERIFIED`, never as
     clean. List those inputs. An excluded input that could hold a secret blocks
     the delivery gate until it is scanned (M-51). Report the scan's exclusion
     rules next to the result, so a clean scan states what it covered.
4. **Verify independently.** Attempt to disprove each candidate by reading the
   code paths that would stop it, including middleware, framework defaults, and
   tests. A candidate that survives becomes `confirmed`, a blocked one becomes
   `needs_validation`, and a disproved one becomes `rejected`.
5. **Recommend the fix.** Name the invariant to enforce and the narrowest change
   that enforces it at the last trusted decision point. Add a regression test that
   fails before the fix and passes after it. The review describes the fix; it does
   not apply it unless the task asks for it.

## Open threat-model items

When the repository has a threat model, list its open High or P1 items whose
surface the change or the review touches, with their location and status, in
the report as known open items. Report them as they are; do not re-score them,
and do not treat them as new findings. They are reported, not blocked by this
review.

## Finding format

For each `confirmed` or `needs_validation` finding:

- **Title** and **verdict**.
- **Location:** repository-relative path and line range.
- **Principal, input, control, boundary, result** (one line each).
- **Severity** (`confirmed` only) with the anchor that justifies it.
- **Missing fact and safe check** (`needs_validation` only).
- **Fix:** the invariant, the change, and the regression test.
- **Sources:** the catalog category and its official reference with version.

Group findings by file. A finding with no location is not reportable.

## Anti-patterns

- Listing checklist deviations as vulnerabilities.
- Reporting defense in depth as a finding.
- Assigning severity to an unverified candidate.
- Stating a deployment or proxy behavior that the repository does not show.
- Reviewing only the diff when the vulnerable path lives outside it. Follow the
  call path into the unchanged code that the change exposes.
