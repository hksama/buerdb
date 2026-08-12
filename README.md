# BuerDB

A Redis-inspired, in-memory key-value store written in Rust. BuerDB is a learning and prototyping project focused on building a production-shaped database from the wire protocol up, with a path toward distributed, eventually consistent replication.

> **Status:** This repository is under active development. APIs, internals, and behavior are subject to change.

## Overview

BuerDB implements a TCP server and interactive CLI client using Tokio. The design follows Redis conventions: commands are expressed over the [RESP](https://redis.io/docs/reference/protocol-spec/) protocol, and the storage layer is intended to use compact in-memory structures similar to those in Redis (listpack, ziplist, quicklist).

The long-term goal is a single-node store that can evolve into a multi-node system where replicas converge without strict coordination, using simple CRDTs for selected data types.

## Current Progress

| Area | Status |
|------|--------|
| Async TCP server (Tokio) | Accepts connections; per-connection task model in place |
| CLI client | Interactive command loop with `GET`, `SET`, `DEL`, `PING` parsing |
| RESP types | `Command` and `RespFrames` enums defined; encode/decode in progress |
| Error handling | Typed protocol errors via `thiserror` |
| Networking I/O | Basic read/write loop; full request/response pipeline not yet wired |
| Storage engine | Placeholder structures (`Listpack`, `Ziplist`, `Quicklist`); no active store |
| Allocator | `jemalloc` on non-MSVC targets |

**Supported commands (planned):** `GET`, `SET`, `DEL`, `PING`

## Architecture (target)

```
Client (CLI)  --RESP/TCP-->  Server  -->  Command handler  -->  In-memory store
                                                                  (listpack / quicklist)
```

## Roadmap

### Near term

- Complete RESP encoding and decoding (arrays, bulk strings, errors, null)
- End-to-end command execution: client input → wire format → server → response
- In-memory key-value table backed by compact list structures
- Connection lifecycle, graceful shutdown, and basic observability

### Distributed & CRDT layer

Planned extensions for multi-node, eventually consistent operation:

- **LWW-Register** — last-writer-wins register for scalar values (`SET`/`GET` under concurrent writes)
- **G-Counter / PN-Counter** — grow-only and positive–negative counters for increment/decrement without a central coordinator
- **OR-Set** — add/remove set with observed-remove semantics for collection types
- **Merge & sync** — anti-entropy or gossip-based state exchange so replicas converge after partition

These are intentionally scoped to simple, well-understood CRDTs suitable for a prototype—not a full geo-distributed database.

### Later

- Persistence (snapshot / append-only log)
- Replication topology and failure handling
- Benchmarking and correctness tests under partition scenarios

## Building & Running

Requires Rust 2021 edition and Tokio.

```bash
# Server (listens on 127.0.0.1:6379)
cargo run --bin mini-redis-server

# Client (interactive CLI)
cargo run --bin mini-redis-client
```

## Tech Stack

- **Rust** (2021 edition)
- **Tokio** — async runtime and TCP I/O
- **bytes** — zero-copy buffer handling
- **tikv-jemallocator** — allocator on Unix-like platforms

## License

MIT
