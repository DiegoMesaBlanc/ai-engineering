# External Skill Registry

## Purpose

Record the source, license, revision, local path, and adaptation status of
each external Skill used by the AI Engineering System.

## 1. Frontend Design

- Name: frontend-design
- Repository: https://github.com/anthropics/skills
- Source path: skills/frontend-design/
- Local path: skills/frontend-design/
- License: Apache-2.0
- License file: LICENSE.txt
- Upstream commit: PENDING
- Local modifications: none planned

## 2. Test-Driven Development

- Name: test-driven-development
- Repository: https://github.com/obra/superpowers
- Source path: skills/test-driven-development/
- Local path: skills/test-driven-development/
- License: MIT
- License file: LICENSE-MIT.txt
- Required supporting file: writing-good-tests.md
- Upstream commit: PENDING
- Local modifications: language-neutral applicability and compatibility
  with Engineering System policies

## Verification Rules

Replace PENDING with the full upstream Git commit SHA used during import.

Do not invent or guess a source revision.

Verify that license files and required supporting files are present.

Document the changes made to imported Skills.

Keep external Skills under the canonical skills/ directory.

Do not install entire external agent frameworks when only an individual
Skill is required.

External Skills must follow AGENTS.md, the project conventions, the
minimal-effective-architecture principle, and applicable security rules.

Do not consider the End-to-End setup ready until the required Skills have
been installed and verified.
