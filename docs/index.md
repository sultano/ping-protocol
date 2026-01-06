# Ping Protocol Documentation

A transport protocol for federated messaging.

## What is Ping?

Ping is a **transport protocol**. It defines how messages flow between servers. What messages mean is defined by **schemas**.

```
Ping      = Protocol (transport, routing, identity, keys)
Schema    = Collection of related message types
Type      = Individual message definition
```

## Core Principles

| Principle | Description |
|-----------|-------------|
| Transport only | Ping moves envelopes; schemas give them meaning |
| Pull-based | Receivers fetch messages from senders |
| Key-based identity | Public keys are identity; addresses are aliases |
| LLM-friendly | Format designed for AI readability |
| Federated | Open, decentralized, no single owner |

## Quick Example

```
<meta>
id: msg123
from: @alice/acme.ping
to: @bob/other.ping
timestamp: 2024-01-03T10:00:00Z
signature: ed25519:a1b2c3...
type: messaging.message.v1
subject: Hello
</meta>

Hey Bob!

This is a message sent over **Ping**.
```

## Documentation

### Protocol Specification

- [Ping Protocol Specification](./ping-protocol-specification.md) - Complete protocol reference

### Schemas

- [Messaging Schema](./schemas/messaging.md) - Human-to-human communication

### Architecture Decision Records

| ADR | Title | Status |
|-----|-------|--------|
| [ADR-001](./adr/001-protocol-name.md) | Protocol Name Selection | Accepted |
| [ADR-002](./adr/002-address-format.md) | Address Format | Accepted |
| [ADR-003](./adr/003-message-format.md) | Message Format | Accepted (Revised) |
| [ADR-004](./adr/004-transport-layer.md) | Transport Layer | Accepted (Revised) |
| [ADR-005](./adr/005-discovery-mechanism.md) | Discovery Mechanism | Accepted (Revised) |
| [ADR-006](./adr/006-encryption-and-signatures.md) | Encryption and Signatures | Accepted |
| [ADR-007](./adr/007-identity-and-keys.md) | Identity and Keys | Accepted |
| [ADR-008](./adr/008-protocol-schema-separation.md) | Protocol vs Schema Separation | Accepted |

## Key Concepts

### Addressing

```
@alice/acme.ping      - Full address
@alice                - Local shorthand
```

Addresses are human-friendly aliases that resolve to public keys.

### Message Format

```
<meta>
...yaml metadata...
</meta>

...markdown body...
```

- `<meta>` block contains YAML metadata
- Body is markdown by default
- Use `<body type="...">` for other formats

### Identity

- Public key IS identity
- One identity can have multiple keys (multi-device)
- Keys fetched via RPC, not DNS

### Delivery (Pull-Based)

```
1. Alice stores message in her outbox
2. Alice's server notifies Bob's server
3. Bob's server fetches from Alice
4. HTTP 200 = delivered
```

### RPC Transport

Single endpoint for all operations:

```
POST /
Content-Type: application/ping
```

Methods: `keys`, `notify`, `fetch`, `ping`

## Technology Stack

| Component | Technology |
|-----------|------------|
| Envelope | XML-style `<meta>` + body |
| Meta format | YAML (default) |
| Body format | Markdown (default) |
| Identity | Ed25519 keys (JWK format) |
| Discovery | DNS SRV/TXT + RPC |
| Transport | HTTPS (POST /) |
| Encryption | TLS 1.3 (mandatory) + MLS (optional E2E) |

## Protocol vs Schema

| Layer | Defines |
|-------|---------|
| **Protocol** | Envelope, routing, keys, RPC, delivery |
| **Schema** | Message types, semantics, fields |

Example schemas:
- `messaging` - Human messaging
- `agents` - AI agent communication
- `webhooks` - Event notifications
- `iot` - Device events
