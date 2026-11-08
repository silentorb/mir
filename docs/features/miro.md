# Miro (working title)

## Summary

**Miro** is a **profile** on Mir: text files as authoring source of truth, a database as derived query projection fed by Mir sync, and **app memory** as a host-local working set pulled on demand from Mir-synced data. “Miro” is a provisional name.

App memory is **not** a Mir sync-graph node and is **not** a Mir push sink in this revision.

## When to read this

- Applying Mir to the common text → database → memory story
- Clarifying pull semantics for app memory
- Comparing host realizations (editor clients vs in-process tools)

## Requirements

### Layer roles

| Layer | Role in Miro |
| --- | --- |
| **Text files** | Authoring / VCS-oriented source of truth for durable records |
| **Database** | Derived projection optimized to compile sources and answer queries; fed by Mir observe/apply from text (and any other Mir sources the host wires) |
| **App memory** | Process-local working set: an **on-demand subset** of Mir-synced data |

Mir owns cold sync and steady-state observe between participating **stores** (typically text → database). Miro adds how applications treat memory relative to that synced surface.

### Text → database

- The text-file store **should** be the authoring SoT when Miro is chosen.
- The database **should** be treated as a **derived** query projection under Miro (not a second competing SoT), unless the host documents a deliberate variation (for example database-only sessions). Such variations are outside Miro’s canonical profile but remain valid Mir topologies.
- Sync between text and database **must** use Mir wiring and signals ([imp-sync-wiring.md](./imp-sync-wiring.md)); Miro does not invent a parallel sync protocol.

### App memory (pull)

- App memory **must not** appear as a Mir sync Imp node in this revision.
- Materialization into app memory **must** be **pull**: the host loads the subset it needs from a Mir-synced surface (commonly the database query store, or an equivalent API over it).
- Mir **does not** push working-set updates into app memory as an observe sink in this revision.
- Invalidation / refresh strategies (reload on signal, TTL, explicit user action) are **host policy**. Miro only requires that memory remain a subset view, not a third durable SoT.

### Opaque data

Miro **must not** prescribe file layouts, SQL schemas, codecs, or domain record taxonomies. Those remain host concerns. Miro speaks only in layer **roles** and Mir sync participation.

### Illustrative hosts (non-normative)

These examples clarify the app-memory role; they are **not** Miro requirements.

| Host sketch | App memory realization | Path length |
| --- | --- | --- |
| Editor / web client over a Mir-synced server projection | Client-held node (or equivalent) working set loaded through the product API | Often longer: network between DB projection and client memory |
| In-process tool (for example a Rust host beside a local DB) | Structures pulled in-process from the projection | Often shorter: little or no networking between DB and memory |

One familiar product sketch: a Tome-style client keeps page/node data in memory as the Miro working set while Mir keeps text and database stores coherent on the server side. A Rust Imp-oriented tool may pull the same conceptual subset with a shorter DB→memory gap.

## Design rationale

- Splitting Miro from Mir keeps the sync fabric free of UI/session memory concerns.
- Pull matches real clients: they rarely want the entire projection mirrored into RAM via sync edges.
- Text→database remains the durable Miro spine; memory is ephemeral and host-shaped.

## Behavior / pipeline

```text
Boot: Mir wires text (and watches) → database sink; cold full sync
Steady: Mir signals keep database projection current
Use:   host pulls neighborhoods / records into app memory as needed
```

Writes (when present) typically update the text SoT (and/or publish Mir signals with optional preview) per host rules; app memory refreshes by pull or host invalidation—not by Mir observeIntoMemory.

## Configuration

Miro does not add a separate config language. Hosts that claim Miro conformance:

1. Register text and database (or equivalent) stores in the Mir registry
2. Wire Mir observe edges text → database (tip watch or file-tree watch per [sync-nodes.md](./sync-nodes.md))
3. Implement app memory as pull over the synced projection

## See also

- [imp-sync-wiring.md](./imp-sync-wiring.md)
- [sync-nodes.md](./sync-nodes.md)
- [problem-domain.md](../overview/problem-domain.md)
- Root [AGENTS.md](../../AGENTS.md)
