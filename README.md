# Gossip Member

A **membership management library** for gossip-based distributed systems, implementing the member list data structure that tracks each node's identity, state (alive/suspect/dead), and incarnation number for use by SWIM-style failure detection protocols.

## Why It Matters

Every distributed system needs to know which nodes are participating. In gossip protocols (SWIM, Consul, Serf), membership isn't maintained by a central registry — it's disseminated peer-to-peer through periodic gossip exchanges. This library provides the core data structure: a concurrent member table with incarnation numbers that prevent resurrected nodes from being confused with fresh ones. The incarnation number is the key insight — it's a monotonically increasing counter per node that breaks ties when conflicting state updates arrive. Without incarnation numbers, a delayed "alive" message could override a correct "dead" message, causing phantom nodes.

## How It Works

**Member state machine**: Each node cycles through three states:

```
       ┌───────┐
       │ ALIVE │ ←──── join (inc=0)
       └───┬───┘
           │ no response to ping
       ┌───▼────┐
       │ SUSPECT │ ←──── incarnation bumps to refute
       └───┬────┘
           │ timeout (T_suspect)
       ┌───▼──┐
       │ DEAD │ ←──── permanent (until reaped)
       └──────┘
```

**Incarnation numbers** enforce monotonic ordering. When node A receives a state update for node B with incarnation `i_new`:
- If `i_new > i_current`: accept the update (newer information)
- If `i_new < i_current`: discard (stale information)
- If `i_new == i_current`: apply state precedence (ALIVE < SUSPECT < DEAD)

This is Lamport's logical clock principle applied to membership.

**Complexity**:
- `add(node)`: O(1) — HashMap insert
- `remove(node)`: O(1) — HashMap delete
- `get(node)`: O(1) — HashMap lookup
- `members()`: O(N) — iterate all entries
- `apply_state(node, state, inc)`: O(1) — compare-and-swap on incarnation

**Gossip fanout**: Each gossip round, a node picks `fanout` random peers (typically 3) and exchanges membership deltas. Full convergence in O(log N) rounds with high probability, by the same analysis as epidemic spreading (the "rumor spreading" problem).

## Quick Start

```rust
use gossip_member::{MemberList, NodeState};

let mut members = MemberList::new();
members.add("node-1");
members.add("node-2");
members.add("node-3");

// Mark a node as suspect
members.set_state("node-2", NodeState::Suspect, 1);

// Apply a remote update (with incarnation arbitration)
members.apply_state("node-2", NodeState::Alive, 2); // higher incarnation wins

assert_eq!(members.alive_count(), 3);
```

## API

| Type | Description |
|------|-------------|
| `MemberList::new()` | Create an empty member table |
| `.add(node_id)` | Add a new member with state ALIVE, incarnation 0 |
| `.remove(node_id)` | Remove a member |
| `.set_state(node_id, state, inc)` | Set state with incarnation arbitration |
| `.apply_state(node_id, state, inc)` | Merge a remote state update |
| `.alive_count()` | Count of currently ALIVE members |
| `.members()` | Iterator over all members |

## Architecture Notes

Gossip Member is the membership substrate for the SuperInstance gossip protocol stack (gossip-protocol, gossip-ping, gossip-suspicion, gossip-seed). It provides the shared data structure that all gossip sub-protocols read and write. In **γ + η = C**, decentralized membership reduces γ — no central coordinator means no single point of failure. See [Architecture](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md).

## References

- Das, A. Gupta, I. & Motivala, A. "SWIM: Scalable Weakly-consistent Infection-style Process Group Membership Protocol," DSN (2002).
- Lamport, L. "Time, Clocks, and the Ordering of Events in a Distributed System," CACM (1978).
- Hashicorp Consul: Gossip Protocol. https://developer.hashicorp.com/consul/docs/architecture/gossip

## License

MIT
