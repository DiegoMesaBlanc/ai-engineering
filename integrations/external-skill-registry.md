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

## 3. Accessibility

- Name: accessibility
- Repository: https://github.com/addyosmani/web-quality-skills
- Source path: skills/accessibility/
- Local path: skills/accessibility/
- License: MIT
- License file: LICENSE.txt
- Required supporting files: references/WCAG.md, references/A11Y-PATTERNS.md
- Upstream commit: afa8da942115f2961fdbfa80807ea0b232ff6c00
- Revision verification: references/WCAG.md
  (SHA-256 16683e025e3833d931c9d880645d98a448827f97e10189b6c8f12c832e7a2361)
  and references/A11Y-PATTERNS.md
  (SHA-256 f10e238dd4dd3c4f51134c2056bc02d3dd4f8fb6ccc9145c2c1ba7aab4d58140)
  are byte-identical to the recorded upstream revision, confirmed 2026-10-17.
  LICENSE.txt matches the upstream MIT LICENSE at the repository root.
  SKILL.md was locally adapted, so it differs from the upstream revision.
- Local modifications: removed the reference to the upstream
  `web-quality-audit` orchestration Skill, which is intentionally not
  imported; no other content change

## 4. SEO

- Name: seo
- Repository: https://github.com/addyosmani/web-quality-skills
- Source path: skills/seo/
- Local path: skills/seo/
- License: MIT
- License file: LICENSE.txt
- Required supporting file: references/STRUCTURED-DATA.md
- Upstream commit: afa8da942115f2961fdbfa80807ea0b232ff6c00
- Revision verification: references/STRUCTURED-DATA.md
  (SHA-256 3e03876f41d223c89dfdbc0a0fe3668454e5d8819aad5379f4541fbf0bd67e17)
  is byte-identical to the recorded upstream revision, confirmed 2026-10-17.
  LICENSE.txt matches the upstream MIT LICENSE at the repository root.
  SKILL.md was locally adapted, so it differs from the upstream revision.
- Local modifications: removed the reference to the upstream
  `web-quality-audit` orchestration Skill (intentionally not imported) and
  added an "Applicability" section that gates SEO checks by application type

## 5. Performance

- Name: performance
- Repository: https://github.com/addyosmani/web-quality-skills
- Source path: skills/performance/
- Local path: skills/performance/
- License: MIT
- License file: LICENSE.txt
- Required supporting files: references/MEASUREMENT.md, references/RUM.md
- Upstream commit: afa8da942115f2961fdbfa80807ea0b232ff6c00
- Revision verification: references/MEASUREMENT.md
  (SHA-256 128f6f20c6c8fb55fcccd8b259cf680811022cf262dc55b03e9d623bcb555a99)
  and references/RUM.md
  (SHA-256 a2bdf660ecb23098afbadfb83254d39cef10c0da82938225f61271bad3f81b19)
  are byte-identical to the recorded upstream revision, confirmed 2026-10-17.
  LICENSE.txt matches the upstream MIT LICENSE at the repository root.
  SKILL.md was locally adapted, so it differs from the upstream revision.
- Local modifications: added a "Coverage and limits" section documenting the
  absence of a heap-profiling workflow and stating that memory leaks are not
  detected automatically

## 6. Core Web Vitals

- Name: core-web-vitals
- Repository: https://github.com/addyosmani/web-quality-skills
- Source path: skills/core-web-vitals/
- Local path: skills/core-web-vitals/
- License: MIT
- License file: LICENSE.txt
- Required supporting files: references/LCP.md, references/INP.md,
  references/CLS.md
- Upstream commit: afa8da942115f2961fdbfa80807ea0b232ff6c00
- Revision verification: references/LCP.md
  (SHA-256 522054d44b7a55409cbde511e8d365f7fa52399dd1e72b8d6b44a92ecddca81f),
  references/INP.md
  (SHA-256 7d40cf6053e9bb8a8a77e02da7c6b25aa8e6c1c092b48d8f9c8f30e9b25cf3a7),
  and references/CLS.md
  (SHA-256 c4fa0cd6311720b413018471a6ac9240e87db52af3b6e306ffb6684491f15898)
  are byte-identical to the recorded upstream revision, confirmed 2026-10-17.
  LICENSE.txt matches the upstream MIT LICENSE at the repository root.
  SKILL.md was locally adapted, so it differs from the upstream revision.
- Local modifications: added an "Applicability" section that gates Core Web
  Vitals checks by application type

## 7. Best Practices

- Name: best-practices
- Repository: https://github.com/addyosmani/web-quality-skills
- Source path: skills/best-practices/
- Local path: skills/best-practices/
- License: MIT
- License file: LICENSE.txt
- Required supporting file: references/SECURITY.md
- Upstream commit: afa8da942115f2961fdbfa80807ea0b232ff6c00
- Revision verification: references/SECURITY.md
  (SHA-256 af021e5db37e921c79b8807d63e1cfc02dcf63614d2806cfba04ba44a1366178)
  is byte-identical to the recorded upstream revision, confirmed 2026-10-17.
  LICENSE.txt matches the upstream MIT LICENSE at the repository root.
  SKILL.md was locally adapted, so it differs from the upstream revision.
- Local modifications: removed the reference to the upstream
  `web-quality-audit` orchestration Skill, which is intentionally not
  imported; no other content change

## Not Imported from This Repository

- `web-quality-audit` (Skill and `scripts/analyze.sh`): intentionally not
  imported. The Engineering System does not add a second orchestration Skill
  or an unaudited shell script. Its cross-Skill audit role is covered by the
  existing internal workflows and by referencing the five imported Skills.

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
