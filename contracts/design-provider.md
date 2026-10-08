# Design Provider Contract

## Purpose

Define the provider-agnostic interface for obtaining design context.

The design source may be:

- Figma
- Penpot
- Sketch
- Adobe XD
- Storybook
- PDF
- images
- screenshots
- HTML/CSS
- diagrams
- another design platform

---

# Provider Capabilities

A Design Provider may expose:

- get design artifact
- get document
- get page
- get frame
- get component
- get variants
- get variables
- get styles
- get assets
- get text
- get layout information
- get interaction information
- get design links

---

# Generic Design Artifact

Normalize retrieved information into concepts such as:

```text
DesignArtifact
├── id
├── name
├── source
├── type
├── url
├── pages
├── frames
├── components
├── variants
├── variables
├── styles
├── assets
├── layout
├── interactions
└── metadata