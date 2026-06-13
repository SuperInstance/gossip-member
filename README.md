# Gossip Member

**A Rust library for managing membership records in a gossip-based distributed system**, tracking node identity, incarnation numbers, and membership state transitions.

## Why It Matters

In distributed systems like HashiCorp Consul, Apache Cassandra, and Redis Cluster, gossip protocols maintain cluster membership without a central coordinator. Each node needs a local view of every other node — alive, suspect, or dead — reconciled through epidemic-style information propagation. This crate provides the membership record data structure that each node maintains, forming the foundation layer of the gossip stack (`gossip-protocol`, `gossip-ping`, `gossip-seed`, `gossip-suspicion`).

## How It Works

The membership module defines the `Member` record type carrying a node's identity (ID, address), an incarnation number (a logical clock incremented on each state change to prevent resurrected nodes), and a current state (`Alive`, `Suspect`, `Dead`). State transitions follow the SWIM protocol: `Alive → Suspect` (on missed heartbeat), `Suspect → Dead` (after timeout), and incarnation numbers resolve conflicts when an older gossip message arrives out of order.

## Quick Start

```rust
// API surface under development — the crate currently provides
// foundational types for member tracking.
use gossip_member::add;

fn main() {
    assert_eq!(add(2, 2), 4);
}
```

## API

| Function | Description |
|---|---|
| `add(left, right)` | Placeholder — full membership API under development |

## Architecture Notes

Part of the SuperInstance gossip stack: `gossip-protocol` (wire protocol), `gossip-member` (membership), `gossip-ping` (health checks), `gossip-seed` (bootstrap), `gossip-suspicion` (timeout handling). See the [Architecture Guide](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md).

## License

MIT
