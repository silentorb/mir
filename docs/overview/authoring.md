# Authoring Mir specs

## Spec vs code

| Layer | Location | Regenerated? |
| --- | --- | --- |
| Language-neutral specs | `mir/docs/` | **No** — authored source of truth |
| Host implementations | Separate language repos | **Yes** — implement from these specs |

When the model changes, update the Mir (or Miro) spec first, then update host bindings to match.

## Dependency rules

1. **Imp.** Mir may require Imp graph IR concepts (`Graph`, ports, `NodeDefinition`, etc.) by reference to Imp’s language-neutral specs. Do not fork or redefine Imp inside Mir. Imp must not mention Mir or Miro.
2. **Mir vs Miro.** Mir fabric docs must not require Miro. Miro may require Mir. App memory belongs only in Miro.
3. **Hosts.** Mir feature **requirements** must not depend on a named product host. Brief historical notes (derivation / expected conformance) belong in README or `AGENTS.md` only. Miro may mention named hosts **illustratively** (roles only—no concrete API or crate contracts as Mir/Miro normative requirements).
4. **No host accidents.** Do not promote path layouts, archive trees, SQL schemas, codecs, or domain inventory shapes into Mir-normative requirements. Keep sync record identity **opaque**.

## Adding a feature spec

1. Create `docs/features/<name>.md` using the template below.
2. Add a routing row to root [`AGENTS.md`](../../AGENTS.md) and this overview [`README.md`](./README.md).

## Feature doc template

```markdown
# <Feature name>

## Summary

## When to read this

## Requirements

## Design rationale

## Behavior / pipeline

## Configuration

## See also
```

Implementation-specific sections (`Quick start`, `Verification`, `Implementation pointers` tied to a language repo) belong in host repos, not in Mir.

## Naming

- Sync node type ids use the `mir.*` namespace.
- **Miro** is a working title; note provisional naming when introducing the profile in new docs.
