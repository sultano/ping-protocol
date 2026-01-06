# ADR-004: Transport Layer

## Status

Accepted (Revised)

## Context

We needed to decide how ping messages are transported between:

- Server to server (federation)
- Client to server (user interaction)

### Previous Decision

Originally chose REST-style HTTPS endpoints (`POST /inbox`, `GET /keys`). This worked but:

- Multiple endpoints to maintain
- REST semantics don't fit RPC patterns well
- Harder to extend with new operations

## Decision

- **Single RPC endpoint**: `POST /`
- **Pull-based delivery**: Receivers fetch from senders
- **Methods in payload**: Operation specified via `method` field

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
| `ping` | Health check | none |

### Request Format

```
<meta>
id: req123
from: @bob/other.ping
method: keys
params:
  address: @alice
</meta>
```

### Response Format

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

### Why Pull Instead of Push?

| Aspect | Push (old email model) | Pull (new) |
|--------|------------------------|------------|
| Data ownership | Receiver stores | Sender stores |
| Spam control | Reactive (block after) | Proactive (choose to fetch) |
| Offline handling | Sender queues | Sender stores, receiver fetches when ready |
| Confirmation | Fire and forget | HTTP 200 = delivered |

### Flow

```
1. Alice stores message in outbox (per-recipient)
2. Alice's server → notify → Bob's server
3. Bob's server → fetch → Alice's server
4. Alice's server returns message
5. HTTP 200 = Bob has it
```

### Per-Recipient Outboxes

Sender maintains separate storage per recipient:

```
Alice's server:
  outbox/@bob/msg123
  outbox/@carol/msg123
```

Benefits:
- Easy tracking per recipient
- Clean deletion when fetched
- No cross-recipient leakage

## Rationale

### Why Single RPC Endpoint?

| Advantage | Details |
|-----------|---------|
| Simpler routing | One endpoint handles all operations |
| Easy to extend | Add methods without new URLs |
| Consistent format | All requests use same envelope |
| Batch-friendly | Could batch multiple operations |

### Why Not REST?

| REST Pattern | Issue |
|--------------|-------|
| `GET /keys/@alice` | URL encoding of `@` is messy |
| Multiple endpoints | More surface area to maintain |
| HTTP verbs | Don't map cleanly to our operations |

### Considered Alternatives

| Alternative | Why Not |
|-------------|---------|
| GraphQL | Overkill for simple operations |
| gRPC | Requires protobuf, less human-readable |
| REST | Multiple endpoints, doesn't fit RPC pattern |
| JSON-RPC | Standard, but why use JSON when we have Ping format? |
| Amazon Ion | Richer types, but adds dependency and complexity |

We use Ping's own format for RPC calls. This means the same envelope format (`<meta>` + body) works for both messages and protocol operations. One format to parse, one format to generate.

### Why Pull Solves Moderation

A key insight: the pull model handles spam/abuse by design.

**Push model (email):**
```
Spammer → pushes into your inbox → you deal with it after
```

**Pull model (Ping):**
```
Spammer → sends notification → you see unknown sender → ignore it
```

The receiver decides at the notification stage whether to fetch. Unwanted content never lands in your inbox because you never pulled it. No need for complex spam filtering rules—just don't fetch from senders you don't recognize or trust.

### Shared Inboxes and Mailing Lists

We considered protocol-level support for:
- **Shared inboxes** (`@support` where multiple people handle messages)
- **Mailing lists** (one address that broadcasts to subscribers)

**Decision:** These are authorization patterns, not protocol concerns.

- Shared inbox = multiple people authorized to act as one address
- Mailing list = multi-recipient where recipient list = subscriber list

The protocol just sees addresses and keys. How an organization manages who can access an address is an implementation detail.

## Consequences

- All transport is `POST /` with `application/ping`
- Operations specified via `method` field
- Pull-based delivery gives receiver control
- Sender maintains per-recipient outboxes
- Simpler server implementation (one endpoint)
