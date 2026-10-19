---
name: engineering-review
description: Review implemented changes for correctness, architecture, maintainability, typing, error handling, testing, security, performance, and unnecessary complexity.
---

# Engineering Review

## Review scope

Review the actual changes, not the entire repository unless required.

## Check

### Correctness

- Does the implementation satisfy the requirement?
- Are edge cases handled?
- Are errors handled correctly?

### Architecture

- Does the change respect the existing architecture and conventions?
- Were unnecessary abstractions introduced?
- Were existing abstractions reused?
- Perform the post-implementation architectural validation defined in
  `skills/architecture-advisor/SKILL.md`: contrast the result with the
  proposed design and the project's conventions, including placement,
  cohesion, reuse, module conventions, and unnecessary fragmentation.
- **Unnecessary abstraction or fragmentation is a finding that must be
  resolved: revert the change or justify it with evidence against the
  thresholds in `skills/complexity-advisor/SKILL.md` (Refactoring and
  Abstraction Justification). A conditional PASS without resolution is not
  acceptable.**

### Code quality

- readability
- naming
- cohesion
- coupling
- duplication
- typing
- maintainability

### Testing

- appropriate unit tests
- integration tests where required
- E2E tests where required
- regression coverage

### Security

Look for relevant vulnerabilities based on the technology stack.

### Performance

Consider performance only when materially relevant.

For web-facing changes, use `skills/performance/SKILL.md` and
`skills/core-web-vitals/SKILL.md`, and `skills/accessibility/SKILL.md`,
`skills/seo/SKILL.md`, and `skills/best-practices/SKILL.md` for
accessibility, search, and web security. Keep field, lab, and hypothesis
evidence distinct, and never present a lab or Lighthouse score as WCAG
conformance or as a ranking prediction.

### Scope

Identify unrelated changes.

## Output

Classify findings:

P0 Critical
P1 Important
P2 Recommended
P3 Nice-to-have

Do not modify code during review unless explicitly requested.
