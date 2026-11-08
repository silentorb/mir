# Imp sync wiring

## Summary

Mir configures multi-store sync with two layers: a heterogeneous **store registry** (instance settings) and a floating **Imp sync graph** (who observes whom). The graph installs observers **once at boot**. It is not a sync executor and is not re-run on each change. Change units are **sync signals** with scoped payloads; sinks apply them according to capability.

## When to read this

- Defining or implementing Mir sync boot / wire-up
- Designing store adapters that emit or apply sync signals
- Separating store options from observe topology

## Requirements

### Two configuration layers

| Layer | Role |
| --- | --- |
| **Store registry** | Map of store instance ids → module/kind + opaque per-kind options; opened into a runtime registry |
| **Sync Imp graph** | Imp `Graph` that wires observe topology; nodes reference **`storeId` only** (no store settings in the graph) |

Hosts **must** keep adapter settings out of the Imp graph so the same wiring pattern can reuse across environments.

### Graph role

- The sync graph **must** install observers at boot only.
- The sync graph **must not** perform ongoing sync via Imp collection / query execution.
- The graph **must** be a DAG; cycles **must** fail boot.
- Unknown node types and incompatible port/capability pairs **must** fail wire-up.

### Stores and capabilities

A **store** is a capability-bearing adapter identified by `storeId`. Mir does not prescribe the physical format (files, SQL, remote graph, etc.).

Relevant capability families (hosts may refine names):

| Capability | Role |
| --- | --- |
| **Observe source** | Can emit sync signals (watch or publish) |
| **Observe sink** | Can apply sync signals |
| **Source read** | Can answer reads needed to expand a signal scope when preview is absent |
| **Query surface** | Optional: host-designated store for compiled queries over synced data |

Wire-up **must** reject edges that connect ports whose required capabilities the bound stores do not provide.

### Sync signals

A **sync signal** carries:

| Field | Role |
| --- | --- |
| `source` | Handle for the originating store / source-read capability |
| `scope` | What changed (see scopes below) |
| `preview` | Optional opaque payload so sinks can apply without waiting on durable source materialization |

**Scopes** (conceptual; hosts map to concrete encodings):

| Scope | Meaning |
| --- | --- |
| **`full`** | Rebuild / reapply from the source(s) as a whole (cold sync, unknown drift) |
| **Incremental** | Opaque record identities and change kinds (created / modified / deleted, or host-equivalent). Identities **must** remain opaque at the Mir layer—no Mir-normative domain inventory schema |
| **Content / config** | Non-record configuration or content-model material changed; sinks refresh accordingly |

Watch-driven paths typically omit `preview` and use `source` read to expand scope. Mutation publish paths **may** include `preview` so sinks apply in parallel with durable persist.

### Designated query store

When a host needs a single compiled query surface over synced data, configuration **may** name one registry store id as the **query store**. Mir does not require Imp (or any particular query language) for that surface—only that the designation exist when the host uses it.

### Fan-in

When multiple sources observe into one sink:

- A **`full`** apply **must** aggregate all inbound sources.
- A sink **must not** clear and replace from only the first source when other inbound sources still contribute to the compiled view.

### Non-participants

**App memory** / process working sets are **not** Mir sync-graph participants in this revision. See [miro.md](./miro.md).

## Design rationale

- Separating registry options from Imp wiring keeps graphs small and reusable.
- Boot-only install avoids treating Imp as a per-change sync engine; Imp describes topology, the host runtime owns apply.
- Opaque incremental identities keep Mir from absorbing one product’s record taxonomy.
- Optional preview enables low-latency sink updates without coupling Mir to a specific mutation API.

## Behavior / pipeline

```text
store registry → open adapters
sync Imp graph → wire observers once (DAG check, capability check)
startup: cold sync (typically scope full) from sources to sinks
steady state: watch or publish → SyncSignal → sink.apply
```

Mutation-style publish (host-defined): durable write may proceed in parallel with preview-bearing sink apply.

## Configuration

Conceptual shape (illustrative, not a required JSON schema):

```json
{
  "dataStores": {
    "authoring": { "module": "…", "options": { } },
    "projection": { "module": "…", "options": { } }
  },
  "sync": {
    "queryStoreId": "projection",
    "graph": {
      "nodes": {
        "src": { "type": "mir.sync.gitWatch", "inputs": { "storeId": "authoring" } },
        "sink": { "type": "mir.sync.store", "inputs": { "storeId": "projection" } }
      },
      "edges": {
        "obs": {
          "from": { "node": "src", "port": "observeOut" },
          "to": { "node": "sink", "port": "observeIn" }
        }
      }
    }
  }
}
```

Node types are specified in [sync-nodes.md](./sync-nodes.md). Hosts choose defaults when `graph` is omitted.

## See also

- [sync-nodes.md](./sync-nodes.md)
- [miro.md](./miro.md)
- [problem-domain.md](../overview/problem-domain.md)
- Imp graph model (imp-spec `imp-core-types` / graph-model)
