# /task

Start or continue implementation from an engineering task.

## Behavior

1. Determine project state.
2. Determine task provider.
3. Retrieve the task when possible.
4. Load Task Context.
5. Detect design references.
6. Load Design Analysis when relevant.
7. Analyze repository.
8. Detect stack.
9. Determine complexity.
10. Determine architecture impact.
11. Produce implementation plan.
12. Ask only required questions.
13. Implement after approval.
14. Validate.
15. Review.
16. Prepare Git changes.
17. Optionally prepare a PR.

## Important

Do not require the user to manually specify:

- framework
- language
- architecture
- task provider
- design provider
- Git provider

when they can be detected.