# ADR-008: Protocol vs Schema Separation

## Status

Accepted

## Context

Early designs mixed transport concerns with message semantics. Fields like `subject`, `intent`, and `attachments` were defined at the protocol level.

This caused problems:

- Protocol became bloated with messaging-specific fields
- Harder to use Ping for non-messaging use cases (webhooks, IoT, agents)
- Unclear what was required vs optional

## Decision

Separate Ping into two layers:

| Layer | Name | Defines |
|-------|------|---------|
| Transport | **Protocol** | Envelope, routing, identity, keys, delivery |
| Semantics | **Schema** | Message types, fields, meaning |

## Terminology

```
Ping      = Protocol (transport layer)
Schema    = Collection of related message types
Type      = Individual message definition
```

## What Protocol Defines

Protocol-level fields (transport concerns):

| Field | Purpose |
|-------|---------|
| `id` | Unique identifier |
| `from` | Sender (for routing, signatures) |
| `to` | Recipient (for routing) |
| `timestamp` | Ordering |
| `signature` | Authentication |
| `method` | RPC operation |
| `in-reply-to` | Response linking |

Protocol also defines:
- Envelope format (`<meta>` + body)
- RPC methods (`keys`, `notify`, `fetch`, `ping`)
- Delivery model (pull-based)
- Discovery (DNS)
- Key management

## What Schemas Define

Schema-level fields (semantic concerns):

| Field | Schema |
|-------|--------|
| `type` | Which schema/version |
| `subject` | messaging |
| `thread` | messaging |
| `attachments` | messaging |
| `task` | agents |
| `result` | agents |
| `event` | webhooks |

## Example Schemas

### messaging

Human-to-human communication:

```
messaging.message.v1
messaging.receipt.v1
messaging.thread.v1
```

### agents

AI agent communication:

```
agents.task.v1
agents.result.v1
agents.status.v1
```

### webhooks

Event notifications:

```
webhooks.event.v1
webhooks.subscription.v1
```

## Message Example

Protocol fields + schema fields together:

```
<meta>
# Protocol fields
id: msg123
from: @alice/acme.ping
to: @bob/other.ping
timestamp: 2024-01-03T10:00:00Z
signature: ed25519:a1b2c3...

# Schema fields
type: messaging.message.v1
subject: Quick question
thread: thread-456
</meta>

Hey Bob, got a minute?
```

## Rationale

### Why Separate?

| Benefit | Details |
|---------|---------|
| Clean protocol | Ping does one thing well: transport |
| Extensible | New schemas without protocol changes |
| Use-case specific | Different schemas for different needs |
| Clear requirements | Protocol fields are always required |

### Analogy

| Layer | HTTP | Ping |
|-------|------|------|
| Transport | HTTP protocol | Ping protocol |
| Semantics | REST/GraphQL/JSON-RPC | messaging/agents/webhooks schemas |

HTTP doesn't define what JSON fields mean. Ping doesn't define what `subject` means.

## Type Namespacing

Types use namespaced strings:

```
messaging.message.v1      # messaging schema
agents.task.v1            # agents schema
com.acme.invoice.v1       # custom schema
```

Protocol doesn't mandate naming convention, just warns that generic names risk collision.

### Influenced by CloudEvents

We studied CloudEvents' approach:
- `specversion` - version of CloudEvents spec itself
- `type` - identifies the event schema (recommends reverse-DNS)

**Why Ping differs:**

CloudEvents has separate `specversion` and `type`. We use only `type` with version embedded (e.g., `messaging.message.v1`).

Reasoning:
- The envelope format (`<meta>` + body) won't change
- Only message types evolve, not the protocol structure
- Adding `specversion` is unnecessary complexity
- Version in the type (`.v1`, `.v2`) handles schema evolution

If we ever need to change the envelope format fundamentally, that would be Ping 2.0—a new protocol, not a version field.

### Why Not Mandate Reverse-DNS?

CloudEvents recommends reverse-DNS naming (e.g., `com.example.event.v1`). We considered mandating this but decided against it:

- Protocol shouldn't be a gatekeeper
- Different organizations have different conventions
- Simple names work fine for internal use
- Collision is the user's problem to solve

We document the risk, not the solution.

## Consequences

- Protocol stays minimal and stable
- Schemas evolve independently
- Anyone can create new schemas
- Clear separation of concerns
- Easier to implement (protocol vs full messaging)
