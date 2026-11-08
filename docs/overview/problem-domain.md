# Problem domain

## Summary

Systems that keep durable data often need **more than one representation**: text files for human and VCS-friendly authoring, databases for compiling multiple sources and querying at scale, and process-local memory for a fast working set. Mir addresses the **sync fabric** between stores. Miro (working title) is a common **profile** that uses that fabric for text → database, then treats app memory as a **pull** over synced data.

## When to read this

- Deciding whether a concern belongs in Mir or Miro
- Understanding why app memory is outside Mir’s sync graph
- Orienting before reading wiring or profile feature specs

## Requirements

### Multi-representation data

Hosts **may** keep the same logical records in more than one store. Mir does **not** require any particular pair of formats. It requires a way to declare **observe topology**, **change signals**, and **adapter capabilities** so stores stay coherent.

### Configurable observe topology

Fixed pipelines (`always files then always SQLite then always RAM`) are too rigid. Mir’s fabric is a **boot-time wiring** of who observes whom among registered stores. The normative configuration encoding for that wiring is an **Imp graph**.

### What Mir is not

- Not a property-graph domain model or document product
- Not a prescription of file layout, SQL schema, or codecs
- Not a definition of app memory / UI working sets

### What Miro adds

**Miro** names one widespread profile:

1. Text files as authoring / VCS source of truth
2. A database as a derived query projection fed by Mir sync
3. **App memory** as a host-local, on-demand **subset** of Mir-synced data

App memory is **outside Mir**: it is not a first-class Mir data store and is **not** a node in the sync Imp graph. In this revision the relationship Mir-synced data → app memory is **pull** (the host materializes what it needs), not Mir **push** into memory.

### Complementary specs

Mir (and Miro) are intended as **one slice** of a larger family of reusable designs. Other LCDs (for example general resource graphs) may appear in other specs. Do not fold those concerns into Mir.

## Design rationale

- Text remains a strong compromise for configuration and authored content; databases remain stronger for multi-source compile and query; memory remains strongest for the live working set. The mistake is forcing one layer to do all three jobs.
- Separating **fabric** (Mir) from **profile** (Miro) avoids conflating “how to wire sync” with “the usual text→DB→memory story.”
- Keeping app memory out of the sync graph matches how hosts actually behave: they load neighborhoods on demand rather than mirroring the entire projection into process memory via observe edges.

## Behavior / pipeline

```text
Mir:   store registry + Imp wiring → observers → SyncSignal among stores
Miro:  text SoT ──Mir sync──► database projection
       database (or other Mir-synced surface) ──host pull──► app memory
```

## Configuration

None at this overview level. See [imp-sync-wiring.md](../features/imp-sync-wiring.md) and [miro.md](../features/miro.md).

## See also

- [imp-sync-wiring.md](../features/imp-sync-wiring.md)
- [sync-nodes.md](../features/sync-nodes.md)
- [miro.md](../features/miro.md)
- Root [AGENTS.md](../../AGENTS.md)
