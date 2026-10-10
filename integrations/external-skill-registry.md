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

Each upstream commit identifies a specific baseline revision.

Record the full commit SHA used for verification.

Do not claim that it was the original import revision unless evidence
establishes that fact.

For unchanged upstream files, verify content against the recorded revision.

For adapted files:

- preserve the upstream source reference
- document all intentional local changes
- retain upstream license notices
- verify required supporting files
- ensure the adapted instructions do not conflict with AGENTS.md

Never invent a source revision.

Do not automatically overwrite local adaptations during updates.

External Skills must follow the Engineering System's workspace safety,
minimal-effective-architecture, security, and testing policies.

Do not consider End-to-End validation ready while an external Skill is
missing, its provenance is unverified, or active instructions conflict
with system policies.
