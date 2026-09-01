# Design Reference

This directory holds optional visual design material produced outside the SDD
workflow. It is reference material for implementation, not a specification.

Behavior, contracts, acceptance criteria, and technical architecture remain in
the feature packet under `specs/`. Visual material does not replace
`create-design` or ADRs.

## Suggested Layout

```text
design/epic-NNN/
  mockup.html
  design-tokens.md
  screenshot.png
```

For the React/TypeScript frontend, `design-tokens.md` should be the source for
the theme. If visual material changes materially, update the owning epic brief
and re-verify affected feature slices.
