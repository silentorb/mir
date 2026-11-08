# Mir

**Mir** is a language-neutral **design spec** for Imp-configured multi-store sync.

This repository holds reusable designs. Language hosts reimplement from the specs (agentic coding across stacks is expected).

| Layer | Role |
| --- | --- |
| **Mir** | Sync **fabric**: store registry, boot-time Imp wiring graph, change signals, example sync nodes |
| **Miro** | Working title for a **profile** on Mir: text file → database, then pull-based **app memory** |

App memory is **not** part of Mir’s sync graph. It is specified in [Miro](./docs/features/miro.md) as an on-demand subset of Mir-synced data (pull, not push, in this revision).

## Layout

| Path | Role |
| --- | --- |
| [`AGENTS.md`](./AGENTS.md) | Agent router — start here |
| [`docs/overview/`](./docs/overview/) | Problem domain, authoring rules, doc map |
| [`docs/features/`](./docs/features/) | Mir fabric + Miro profile specs |

## History

Mir was originally derived from sync solutions later productized in Tome. Tome is expected to conform to and implement Mir (and Miro where that profile applies). Mir feature requirements do not depend on Tome.

## Imp

Mir uses [Imp](https://github.com/silentorb/imp-spec) graphs as the normative **configuration encoding** for sync observe topology. Mir does not redefine Imp’s graph IR; see Imp’s language-neutral specs for `Graph`, ports, and `NodeDefinition`.
