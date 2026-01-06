# Ping Protocol Specification

Version: 0.2.0 (Draft)

## Overview

Ping is a transport protocol for federated messaging. It defines how messages flow between servers. What messages mean is defined by schemas.

**Federated** means there's no central authority. Anyone can run a Ping server on their own domain. Servers discover each other via DNS and communicate directly, similar to how email works but with modern security.

```
Ping      = Protocol (transport, routing, identity, keys)
Schema    = Collection of related message types
Type      = Individual message definition
```

## Design Principles

| Principle | Description |
|-----------|-------------|
| Transport only | Ping moves envelopes; schemas give them meaning |
| Pull-based | Receivers fetch messages from senders |
| Key-based identity | Public keys are identity; addresses are aliases |
| LLM-friendly | XML-style tags are easier for AI to parse than JSON brackets or YAML indentation |
| Federated | No central server; anyone can run a Ping server and communicate with others |
| Simple | Minimal spec, easy to implement |

## Message Format

Messages use an XML-style envelope with YAML metadata and flexible body.

### Structure

```
<meta type="yaml">
...yaml key-value pairs...
</meta>

...body content (markdown by default)...
```

### Rules

1. `<meta>` block is required
2. `type` attribute on meta is optional (defaults to `yaml`)
3. Everything after `</meta>` is the body
4. Body is markdown by default
5. Use `<body type="...">` for non-markdown body content

### Examples

**Simple message:**

```
<meta>
id: msg123
from: @alice/acme.ping
to: @bob/other.ping
timestamp: 2024-01-03T10:00:00Z
signature: ed25519:a1b2c3...
type: messaging.message.v1
</meta>

Hey Bob,

This is **markdown** by default.
```

**RPC request:**

```
<meta>
id: req123
from: @bob/other.ping
method: keys
params:
  address: @alice
</meta>
```

**RPC response with structured body:**

```
<meta>
from: @acme.ping
in-reply-to: req123
</meta>

<body type="yaml">
keys:
  - kty: OKP
    crv: Ed25519
    kid: laptop
    x: a1b2c3...
</body>
```

## Required Fields (Protocol Level)

| Field | When Required | Purpose |
|-------|---------------|---------|
| `id` | Always | Unique identifier |
| `from` | Always | Sender address (for signature verification) |
| `signature` | Always | Ed25519 signature |
| `to` | Messages | Routing (can be array for multi-recipient) |
| `timestamp` | Messages | Ordering |
| `method` | RPC calls | Which operation |
| `in-reply-to` | Responses | Links response to request |

### Schema-Level Fields

These are NOT part of the protocol. Schemas define their own fields:

- `type` - Message schema type (e.g., `messaging.message.v1`)
- `subject`, `thread`, `attachments`, etc. - Defined by schemas

## Addressing

### Format

```
@<name>/<domain>
```

- `@alice/acme.ping` - full address
- `@alice` - local shorthand (domain inferred)

### Address as Alias

Addresses are human-friendly aliases that resolve to public keys:

```
@alice/acme.ping  →  [key1, key2, key3]
```

One identity can have multiple keys (like GitHub SSH keys).

### URI Scheme

```
ping:@alice/acme.ping
```

## Identity and Keys

### Key-Based Identity

- Public key IS identity
- Address is human-friendly alias
- One identity can have multiple keys (multi-device)
- Server hosts keys for its users

### Key Format

Keys use JWK (RFC 7517):

```yaml
keys:
  - kty: OKP
    crv: Ed25519
    kid: laptop
    x: base64url-encoded-public-key
  - kty: OKP
    crv: Ed25519
    kid: phone
    x: base64url-encoded-public-key
```

## Transport

### Single Endpoint

All operations go through root:

```
POST /
Content-Type: application/ping
```

### RPC Methods

| Method | Purpose | Params |
|--------|---------|--------|
| `keys` | Fetch user's public keys | `address` |
| `notify` | Alert recipient of new message | `id`, `from`, `to` |
| `fetch` | Retrieve message from outbox | `id`, `recipient` |
| `update` | Notify recipient of edit/delete | `id`, `action` |
| `ping` | Health check | none |

### Request Example

```
POST /
Content-Type: application/ping

<meta>
id: req123
from: @bob/other.ping
method: keys
params:
  address: @alice
</meta>
```

### Response Example

```
<meta>
from: @acme.ping
in-reply-to: req123
</meta>

<body type="yaml">
keys:
  - kty: OKP
    crv: Ed25519
    kid: laptop
    x: a1b2c3...
</body>
```

## Delivery Model (Pull-Based)

### Flow

```
1. Alice stores message in her outbox (per-recipient)
2. Alice's server notifies Bob's server (method: notify)
3. Bob's server fetches from Alice's server (method: fetch)
4. HTTP 200 = delivered
```

### Per-Recipient Outboxes

Sender maintains separate outbox per recipient:

```
Alice's server:
  outbox/@bob/msg123      ← Bob fetches here
  outbox/@carol/msg123    ← Carol fetches here
```

### Why Pull?

- Sender controls their data
- Receiver decides what to fetch (spam control)
- Clear ownership and audit trail
- No queue pressure on sender

## Discovery

### DNS Records

For domain `acme.ping`:

```dns
; Server discovery
_ping._tcp.acme.ping. 300 IN SRV 10 1 443 ping.acme.ping.

; Server public key
_ping._tcp.acme.ping. 300 IN TXT "pk=ed25519:base64..."
```

### Discovery Flow

1. Query `_ping._tcp.<domain>` SRV record → host, port
2. Query TXT record → server public key
3. Connect to server
4. User keys fetched via `keys` RPC method

DNS provides server discovery. User keys come from the server via RPC.

## Security

### Transport Encryption

TLS 1.3 required for all connections.

### Message Signatures

All messages must be signed with Ed25519:

```
<meta>
id: msg123
from: @alice/acme.ping
signature: ed25519:a1b2c3...
</meta>
```

**What's signed:**
- Meta content (excluding signature field)
- Body content

### Signature Verification

1. Receiver extracts `from` address
2. Fetches sender's public keys (via `keys` RPC)
3. Verifies signature
4. Rejects if invalid

### End-to-End Encryption (Optional)

For sensitive messages, MLS (RFC 9420) is recommended.

## Multi-Recipient

`to` field can be an array:

```
<meta>
id: msg123
from: @alice/acme.ping
to:
  - @bob/other.ping
  - @carol/third.ping
</meta>

Hey team!
```

Sender creates per-recipient outboxes and notifies each server.

## Edit and Delete

### Not Yet Fetched

Edit message in place in outbox. Recipient will fetch latest version.

### Already Fetched

Send update notification:

```
<meta>
id: update123
from: @alice/acme.ping
to: @bob/other.ping
method: update
params:
  id: msg123
  action: edit  # or: delete
</meta>
```

Recipient can re-fetch or ignore. They have their copy.

## Errors

Errors returned as responses:

```
<meta>
from: @acme.ping
in-reply-to: req123
error:
  code: not_found
  message: Address not found
</meta>
```

Standard error codes:
- `not_found` - Address or message doesn't exist
- `invalid_signature` - Signature verification failed
- `unauthorized` - Not allowed
- `rate_limited` - Too many requests
- `server_error` - Internal error

## Content Type

```
Content-Type: application/ping
```

## Type Namespacing

Types are namespaced strings. Protocol doesn't prescribe naming convention, but warns:

- Generic names (`message.v1`) risk collision
- Use namespacing that works for your use case

Examples:
```
messaging.message.v1      # messaging schema
agents.task.v1            # agents schema
com.acme.invoice.v1       # custom
```

## Protocol vs Schema

| Layer | Defines |
|-------|---------|
| **Protocol (Ping)** | Envelope, routing, keys, RPC, delivery |
| **Schema** | Message types, semantics, fields |

Ping transports envelopes. Schemas define what's inside.

### Example Schemas

- `messaging` - Human messaging (messages, threads, receipts)
- `agents` - AI agent communication (tasks, results)
- `webhooks` - Event notifications
- `iot` - Device events

## References

- [RFC 7517](https://www.rfc-editor.org/rfc/rfc7517.html) - JSON Web Key (JWK)
- [RFC 9420](https://www.rfc-editor.org/rfc/rfc9420.html) - Messaging Layer Security (MLS)
- [Ed25519](https://ed25519.cr.yp.to/) - High-speed signatures
- [RFC 2782](https://www.rfc-editor.org/rfc/rfc2782.html) - DNS SRV Records
