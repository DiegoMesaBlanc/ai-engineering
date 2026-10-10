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
- Upstream commit: dbd4588f9e1033efb41dad4bef2f7947c8993d44
- Revision verification: byte-identical match of SKILL.md and LICENSE.txt
  against the recorded upstream revision, confirmed 2026-10-09.
- Local modifications: none planned

## 2. Test-Driven Development

- Name: test-driven-development
- Repository: https://github.com/obra/superpowers
- Source path: skills/test-driven-development/
- Local path: skills/test-driven-development/
- License: MIT
- License file: LICENSE-MIT.txt
- Required supporting file: writing-good-tests.md
- Upstream commit: 8ca22dba9a94f28898bbce59f2537ff4d87c747d
- Revision verification: writing-good-tests.md and LICENSE-MIT.txt are
  byte-identical to the recorded upstream revision, confirmed 2026-10-09.
  License-MIT.txt matches the upstream MIT LICENSE at the repository root.
  SKILL.md was locally adapted for language-neutral applicability and
  Engineering System policy alignment, so it differs from the upstream
  revision.
- Local modifications: language-neutral applicability and compatibility
  with Engineering System policies

## Verification Rules

Replace PENDING with the full upstream Git commit SHA used during import.

Confirm the recorded revision by checking that the imported files are
byte-identical to that revision's content when no local modifications
affected them.

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
