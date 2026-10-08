---
name: design-analysis
description: Analyze design artifacts and translate visual and interaction requirements into implementation-ready engineering requirements without coupling the system to a specific design platform.
---

# Design Analysis

## Purpose

Analyze design artifacts associated with a development task and translate
them into clear, implementation-oriented requirements.

A design artifact may come from:

- Figma
- Penpot
- Sketch
- Adobe XD
- FigJam
- Storybook
- HTML/CSS
- PDF
- PNG/JPG
- screenshots
- wireframes
- diagrams
- documentation
- URLs
- an existing application
- another design or prototyping system

Do not assume a specific design platform.

The design source is a provider concern.
Design interpretation is an engineering concern.

---

## Core Principle

Do not reproduce a design blindly.

Determine:

1. What the design is communicating.
2. Which requirements are explicit.
3. Which behaviors are implied.
4. Which existing project components should be reused.
5. Which parts require new implementation.
6. Which requirements are ambiguous.
7. Which design decisions conflict with the existing architecture or design system.

Prefer reuse over duplication.

Do not create a new component if an existing component already provides
the required behavior and appearance with reasonable configuration.

---

# Analysis Process

## 1. Identify the Design Source

Determine:

- design platform
- artifact type
- available pages/screens
- relevant frames/artboards
- components
- variants
- assets
- links
- documentation
- design tokens
- interaction specifications

If the source cannot be accessed, do not invent information.

Clearly identify unavailable information.

---

## 2. Identify Scope

Determine which parts of the design are relevant to the current task.

Ignore unrelated screens and components.

Map the design to:

- task requirements
- acceptance criteria
- affected application areas

---

## 3. Layout Analysis

Analyze:

- page structure
- containers
- sections
- grids
- columns
- alignment
- spacing
- padding
- margins
- positioning
- responsive behavior
- breakpoints
- viewport assumptions

Do not blindly convert visual measurements into hardcoded values.

Prefer the project's existing layout and spacing system.

---

## 4. Component Analysis

Identify:

- buttons
- inputs
- selects
- dropdowns
- tables
- cards
- modals
- dialogs
- navigation
- tabs
- menus
- forms
- alerts
- notifications
- loaders
- pagination
- lists
- custom components

For each component determine:

- existing component candidate
- required behavior
- required variants
- required states
- whether a new component is actually necessary

---

## 5. Design System Analysis

Identify when available:

- colors
- typography
- font sizes
- font weights
- spacing
- border radius
- shadows
- borders
- icons
- breakpoints
- tokens
- component variants

Prefer existing project tokens/design-system components.

Do not introduce a new design system because a design contains values
different from the current implementation.

If there is a conflict, report it.

---

## 6. Component States

Identify relevant states such as:

- default
- hover
- focus
- active
- disabled
- loading
- error
- empty
- success
- selected
- validation error
- read-only

Do not assume states that are not supported by evidence.

If an important state is missing from the design, identify it as an
implementation consideration rather than inventing a visual design.

---

## 7. Interaction Analysis

Identify:

- navigation
- clicks
- form submission
- validation
- modal behavior
- dropdown behavior
- tabs
- pagination
- filtering
- sorting
- search
- loading behavior
- asynchronous operations
- error handling
- success feedback
- confirmation flows

Distinguish:

- explicitly specified behavior
- behavior strongly implied by the design
- behavior that requires clarification

---

## 8. Responsive Analysis

When responsive designs exist, compare:

- desktop
- tablet
- mobile

Identify:

- layout changes
- hidden elements
- reordered content
- component transformations
- navigation changes
- typography changes
- spacing changes
- table transformations
- modal behavior

Do not assume that desktop behavior automatically defines mobile behavior.

---

## 9. Accessibility Analysis

Check for:

- semantic structure
- keyboard navigation
- focus behavior
- accessible names
- labels
- form errors
- contrast
- interactive states
- screen-reader considerations
- touch target considerations

Accessibility requirements should be preserved even when the visual design
does not explicitly document them.

---

## 10. Existing Code Mapping

Compare the design with the current repository.

Identify:

- existing components
- existing layouts
- existing tokens
- existing hooks/services
- existing form abstractions
- existing state management
- existing design-system components

Prefer:

Existing implementation >
Configuration/extension >
Small reusable component >
New abstraction >
Architectural change

Do not introduce architectural changes merely to reproduce a design.

---

## 11. Design / Code Gaps

Identify discrepancies such as:

- design component does not exist
- existing component differs visually
- existing component lacks required variant
- design requires behavior not supported by current component
- design conflicts with accessibility requirements
- design conflicts with responsive conventions
- design introduces duplicated UI patterns
- design conflicts with existing design system

Classify findings:

- Critical
- Important
- Recommended
- Informational

Do not automatically modify unrelated components.

---

# Output

Produce a concise:

## DESIGN ANALYSIS

### Source

- Type:
- Platform:
- Artifact:
- Availability:

### Relevant Screens

- ...

### Functional Behavior

- ...

### Components

| Design Element | Existing Component | Action |
| -------------- | ------------------ | ------ |
| ...            | ...                | Reuse  |
| ...            | ...                | Extend |
| ...            | None               | Create |

### Responsive Behavior

- ...

### States

- ...

### Accessibility

- ...

### Design Tokens

- ...

### Existing Code Reuse

- ...

### Gaps / Risks

- ...

### Unknowns

- ...

### Recommendations

- ...

---

# Engineering Rules

1. Never invent unavailable design information.
2. Never assume Figma is the design source.
3. Never create duplicate components unnecessarily.
4. Prefer existing project components and design systems.
5. Do not introduce architectural complexity to reproduce visual designs.
6. Do not change global design-system components without evaluating their
   impact on existing consumers.
7. Separate visual requirements from functional requirements.
8. Separate explicit requirements from inferred behavior.
9. Identify ambiguity before implementation.
10. Preserve accessibility.
11. Preserve responsive behavior.
12. Respect existing project conventions.
13. Do not expand task scope without justification.
14. Report conflicts between design and implementation before changing them.
