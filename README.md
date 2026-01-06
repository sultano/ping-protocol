# Ping Protocol

A transport protocol for federated messaging.

## What is Ping?

Ping is a **transport protocol**—it defines how messages flow between servers. What messages mean is defined by **schemas**.

```
Ping      = Protocol (transport, routing, identity, keys)
Schema    = Collection of related message types
Type      = Individual message definition
```

**Federated** means there's no central authority. Anyone can run a Ping server on their own domain. Servers discover each other via DNS and communicate directly, similar to how email works but with modern security.

## Example

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

## Key Features

| Feature | Description |
|---------|-------------|
| **Pull-based delivery** | Receivers fetch from senders. You control what enters your inbox. |
| **Key-based identity** | Public keys are identity. Addresses are human-friendly aliases. |
| **Multi-device** | One identity, multiple keys (like GitHub SSH keys). |
| **LLM-friendly** | XML-style tags are easier for AI to parse than JSON or YAML. |
| **Schema separation** | Protocol handles transport. Schemas define message types. |

## How It Works

```
1. Alice stores message in her outbox
2. Alice's server notifies Bob's server
3. Bob's server fetches from Alice
4. HTTP 200 = delivered
```

All operations go through a single RPC endpoint:

```
POST /
Content-Type: application/ping
```

## Documentation

- [Protocol Specification](docs/ping-protocol-specification.md) — Complete protocol reference
- [Messaging Schema](docs/schemas/messaging.md) — Human-to-human communication types
- [Architecture Decision Records](docs/index.md#architecture-decision-records) — Design decisions and rationale

## Design Principles

- **Transport only** — Ping moves envelopes; schemas give them meaning
- **Pull-based** — Receivers fetch messages from senders
- **Key-based identity** — Public keys are identity; addresses are aliases
- **LLM-friendly** — Format designed for AI readability
- **Federated** — No central server; anyone can run a Ping server
- **Simple** — Minimal spec, easy to implement

## Technology

| Component | Technology |
|-----------|------------|
| Envelope | XML-style `<meta>` + body |
| Meta format | YAML (default) |
| Body format | Markdown (default) |
| Identity | Ed25519 keys (JWK format) |
| Discovery | DNS SRV/TXT |
| Transport | HTTPS |
| Encryption | TLS 1.3 (mandatory) + MLS (optional E2E) |

## Status

This protocol is in **draft** stage. The specification is being developed and may change.

## License

[To be determined]
