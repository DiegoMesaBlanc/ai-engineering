---
description: Apply an adjustment to the ai-engineering system and prove in the same run that the result is correct, with a structural audit of the repository and a behavioral audit that builds a real artifact, proves the system accepts it, and proves it rejects the violation the adjustment was meant to prevent.
agent: orchestrator
---

Run the System Audit Workflow:

`workflows/system-audit.md`

## Input

The adjustment to apply, in plain language:

```
/audit-e2e <the adjustment I want in ai-engineering>
```

The text is the only input. It defines the scope: what changes, which area
of the system it touches, and therefore which checks the audit runs and
which artifacts it builds. There are no other modes and no flags.

## Mandatory Behavior

1. Resolve the canonical system root.
2. **Understand the adjustment** from the text alone. Determine what
   changes, which area it belongs to, which rules are in scope, and which
   artifacts the behavioral audit must build. If the adjustment is ambiguous
   in a way that changes the outcome, ask the user before implementing.
3. **Implement** the adjustment following `workflows/task-to-code.md`: align,
   prepare the branch, implement, validate, review. A new rule needs its
   canonical wording in `AGENTS.md`, its required surfaces, its registry row,
   its regression row, and its check. Do not create a parallel flow, command,
   or file that duplicates what already exists.
4. **Audit structurally**, always: reference integrity, index and inventory,
   numbering, rule reachability, cross-client reach in `AGENTS.md`,
   duplication and contradiction, and coverage per ecosystem and project
   type. Every check reports `PASS`, `FAIL`, or `NOT VERIFIED` with the
   reason and the evidence.
5. **Audit behaviorally**: build a real artifact per affected ecosystem or
   project type in a temporary directory outside the repository, install the
   canonical artifacts, verify the compliant artifact passes, inject the
   violation the adjustment was meant to prevent, verify it is detected with
   `file:line` evidence, fix it, re-verify, and clean the temporary
   directory.
6. When a check finds a defect in the system's own artifact, report it as a
   defect with the evidence, correct it, and repeat the cycle.
7. Never report an unexecuted check as passed.
8. Report the limits: state explicitly what could not be verified and why,
   and distinguish what the mechanism guarantees from what depends on an
   agent following the flow.
9. Present the commit text. **Do not create the commit**; it requires the
   explicit approval of the commit approval policy.

## Safety

- Do not create commits, push, or change remote state.
- Do not install tools or change configuration in the audited repository.
- Keep the behavioral artifacts in a temporary directory outside the
  repository, and delete them when finished.
- Do not print credentials or secret values.
- Use shell commands only when necessary and request permission when
  required by OpenCode.

## Output

Return the report defined by the workflow: the adjustment understood and
implemented, the structural audit, the behavioral audit with its `file:line`
evidence, the defects found, the limits, and the commit text.
