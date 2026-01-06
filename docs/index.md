# Ping Protocol Documentation

Structured messaging.

## Why Ping?

Existing messaging protocols treat messages as opaque blobs of text. Ping treats messages as structured data—simple enough for humans, parseable by machines.

This means one protocol for:

- Human-to-human messaging
- AI agents and automation
- Webhooks and notifications
- IoT and device events
- Workflow triggers

## Core Principles

| Principle | Description |
|-----------|-------------|
| Structured | Messages are data with YAML front matter, not text blobs |
| Simple | Easy to read, write, and implement |
| Extensible | One format adapts to any messaging use case |
| Secure | Cryptographic identity—no spoofing, no forgery |
| Federated | Open, decentralized, no single owner |

## Quick Links

### Protocol Specification

- [Ping Protocol Specification](./ping-protocol-specification.md) — Complete protocol reference

### Architecture Decision Records

| ADR | Title | Status |
|-----|-------|--------|
| [ADR-001](./adr/001-protocol-name.md) | Protocol Name Selection | Accepted |
| [ADR-002](./adr/002-address-format.md) | Address Format | Accepted |
| [ADR-003](./adr/003-message-format.md) | Message Format | Accepted |
| [ADR-004](./adr/004-transport-layer.md) | Transport Layer | Accepted |
| [ADR-005](./adr/005-discovery-mechanism.md) | Discovery Mechanism | Accepted |
| [ADR-006](./adr/006-encryption-and-signatures.md) | Encryption and Signatures | Accepted |

## Key Concepts

### Addressing

```
@alice/acme.ping      — Full address
@alice                — Local shorthand
```

### Message Format

```yaml
---
from: @alice/acme.ping
to: @bob/acme.ping
subject: Hello
---
Message body here.
```

### URI Scheme

```
ping:@alice/acme.ping
```

## Technology Stack

| Component | Technology |
|-----------|------------|
| Message format | YAML front matter + flexible body |
| Identity | Ed25519 signatures |
| Discovery | DNS SRV/TXT records |
| Transport | HTTPS + WebSocket |
| Encryption | TLS 1.3 (mandatory) + MLS (optional E2E) |

## Goals

1. Simple enough to implement in an afternoon
2. Extensible enough for any messaging use case
3. Secure by default—cryptographic identity baked in
4. Federated and open
