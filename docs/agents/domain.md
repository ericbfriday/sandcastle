# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

- `CONTEXT.md` at the repo root
- `docs/adr/` for architectural decisions relevant to the area being changed

If either is missing, proceed silently.

## Layout

This is a single-context repo:

- one `CONTEXT.md` at the repo root
- one `docs/adr/` directory at the repo root

## Terminology

Use terms from `CONTEXT.md` when naming concepts in issues, plans, tests, and proposals.

## ADR conflicts

If a proposed change conflicts with an ADR, surface the conflict explicitly instead of silently overriding it.
