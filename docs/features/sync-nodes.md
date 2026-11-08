# Sync nodes

## Summary

High-level designs for common **Mir sync Imp nodes**. These are catalog-style sketches (ports and capabilities), not language bindings. Type ids live under the `mir.*` namespace.

## When to read this

- Implementing a Mir wire-up registry of sync node types
- Authoring sync Imp graphs
- Checking port and capability compatibility

## Requirements

### General

- Each sync node type **must** declare inputs/outputs in Imp port terms.
- Nodes that bind a store **must** take `storeId` (string) as an input (or equivalent) referencing the store registry—not inline store options.
- Wire-up **must** reject unknown types and incompatible observe edges.
- This catalog **must not** define an app-memory / working-set sync node. App memory is outside Mir; see [miro.md](./miro.md).

### `mir.sync.store`

Binds a registered store into the sync graph for observe participation.

| Port | Direction | Role |
| --- | --- | --- |
| `storeId` | input | Registry id of the store |
| `observeOut` | output | Emits sync signals when this store acts as an observe source |
| `observeIn` | input | Receives sync signals when this store acts as an observe sink |

**Capabilities:** the bound store must support source and/or sink roles for the ports actually wired. A store used only as a sink needs `observeIn`; only as a source needs `observeOut`.

**File-tree observe form:** when a text/file-oriented store exposes content-tree watching, that watch **may** be activated via this node’s `observeOut` (host maps the store kind to FS subscribe). See mutual exclusion with git tip watch below.

### `mir.sync.gitWatch`

Observe source driven by **VCS tip** change for a file-oriented store (for example resolved HEAD / branch tip), not by every content-tree file event.

| Port | Direction | Role |
| --- | --- | --- |
| `storeId` | input | Registry id of the file-oriented store whose VCS tip is watched |
| `observeOut` | output | Emits sync signals when the tip changes |

**Capabilities:** bound store (or host VCS integration for that store) must support tip observation and source-read sufficient to expand incremental / content scopes after a tip change.

**Baseline:** after cold/`full` sync, hosts **should** record the current tip as the sync baseline for that `storeId` so subsequent tip changes produce incremental or content scopes rather than unexplained full rebuilds. Storage of the baseline is a host concern (opaque key/value is enough).

### Mutual exclusion of observe forms

For a given source store, hosts **must** pick **one** steady-state observe form:

| Form | Typical node |
| --- | --- |
| VCS tip | `mir.sync.gitWatch` → sinks |
| Content / file tree | `mir.sync.store` `observeOut` → sinks |

When using git tip watch, the file store remains a **registry** entry (CRUD / source read) and **need not** also appear as a `mir.sync.store` graph node solely to watch the tree.

### Query sink usage

A database or other projection store used as a Mir sink is typically a `mir.sync.store` with `observeIn` wired from one or more sources. If the host designates a **query store**, that id usually names this sink (or another compiled projection). Mir does not require a distinct node type for “query sink.”

### Additional sinks

Hosts **may** register further sink stores (search indexes, secondary projections) as registry entries and wire them with the same `mir.sync.store` `observeIn` pattern. Fan-in rules in [imp-sync-wiring.md](./imp-sync-wiring.md) apply.

## Design rationale

- `storeId`-only inputs keep Imp graphs portable across machines and option shapes.
- Separating `gitWatch` from file-tree `observeOut` avoids double-apply when durable writes already update files that a content watcher would see again.
- Omitting an app-memory node keeps Mir’s graph about durable/query adapters, not ephemeral working sets.

## Behavior / pipeline

At boot, the host resolves each sync node’s `storeId`, checks capabilities against wired ports, subscribes observe sources, and connects them to sink `apply` handlers. At runtime, sources emit signals; sinks apply—no further Imp graph execution for sync.

## Configuration

See the illustrative sync graph under [imp-sync-wiring.md](./imp-sync-wiring.md) § Configuration.

## See also

- [imp-sync-wiring.md](./imp-sync-wiring.md)
- [miro.md](./miro.md)
- Imp node libraries (imp-spec `imp-core-types` / node-libraries)
