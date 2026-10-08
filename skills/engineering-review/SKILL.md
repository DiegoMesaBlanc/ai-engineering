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

- Does the change respect the existing architecture?
- Were unnecessary abstractions introduced?
- Were existing abstractions reused?

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

### Scope

Identify unrelated changes.

## Output

Classify findings:

P0 Critical
P1 Important
P2 Recommended
P3 Nice-to-have

Do not modify code during review unless explicitly requested.
