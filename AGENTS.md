# AGENTS Guide — Mir

## Repository purpose

**Mir** (Mirror) is a language-neutral **design-spec** repository for Imp-configured multi-store sync. Hosts reimplement from these docs; this repo does not ship a runtime.

| Layer | Meaning |
| --- | --- |
| **Mir** | Sync fabric: who observes whom, signals, store capabilities, example `mir.*` sync nodes |
| **Miro** | Working title — profile sub-spec: text file → database → pull **app memory** |

## Scope boundary

**In Mir**
- Imp as normative encoding for sync wiring graphs
- Store registry vs wiring graph separation
- Sync signals, observe capabilities, fan-in
- Example high-level sync node designs (`mir.*`)
- Problem-domain background for multi-store sync

**In Miro** (same repo, sub-spec)
- Text-file authoring SoT → database query projection
- App memory as on-demand **pull** working set (not a Mir sync-graph node)

**Out of this repo**
- Language bindings / packages
- Property-graph resource models beyond opaque sync identities
- Concrete path layouts, SQL schemas, codecs, archive trees, or host domain inventories as Mir-normative
- App memory as a Mir push sink or sync-graph participant

## Locked rules

1. **One-way Imp dependency.** Mir may reference Imp graph-model / node-library concepts. Imp specs must not mention Mir or Miro.
2. **No Tome-accident leakage in Mir.** Keep record identity **opaque**. Do not encode host path layouts, archive trees, SQL schemas, markdown/codecs, or fixed domain inventory shapes (e.g. node/relationship id-set schemas) as Mir requirements. Prefer general scopes and opaque ids.
3. **App memory is outside Mir.** Specify it only in Miro; never as a `mir.*` sync node in this revision.
4. **Tome citations.** Mir docs: at most a brief historical note (derivation / expected conformance). Miro may cite Tome or Rust hosts **illustratively**, not as normative dependencies.

## Feature documentation

| If your task involves… | Read |
| --- | --- |
| Problem domain, Mir vs Miro | [problem-domain.md](./docs/overview/problem-domain.md) |
| Authoring Mir/Miro specs | [authoring.md](./docs/overview/authoring.md) |
| Imp sync wiring, signals, capabilities | [imp-sync-wiring.md](./docs/features/imp-sync-wiring.md) |
| Example sync nodes | [sync-nodes.md](./docs/features/sync-nodes.md) |
| Text → database → app memory profile | [miro.md](./docs/features/miro.md) |

Doc map: [docs/overview/README.md](./docs/overview/README.md).

## History

Mir was originally derived from sync solutions later productized in Tome. Tome is expected to conform to and implement Mir. Do not deep-link Tome feature docs from Mir requirements.

## Working conventions

- Specs only; no implementation packages in this repo yet.
- Prototypal stage — no backwards-compatibility obligation for the docs themselves.
- Namespace sync node type ids under `mir.*`.
- “Miro” is a **working title** and may rename later; treat the name as provisional in prose.
