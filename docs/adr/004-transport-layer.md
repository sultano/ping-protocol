# ADR-004: Transport Layer

## Status

Accepted

## Context

We needed to decide how ping messages are transported between:

- Server to server (federation)
- Client to server (user interaction)

The transport must be secure, firewall-friendly, and use existing infrastructure where possible.

## Decision

- **Server-to-Server**: HTTPS POST
- **Client-to-Server**: HTTPS + WebSocket for real-time

## Server-to-Server Federation

Simple HTTPS POST to the destination server's inbox:

```http
POST https://ping.other.ping/inbox
Content-Type: application/ping
Authorization: Ping-Signature <signed-request>

---
from: @alice/acme.ping
to: @bob/other.ping
id: 01HX2Z...
timestamp: 2025-01-03T10:30:00Z
---
Hey Bob, are you free Friday?
```

### Why HTTPS?

- Works everywhere (firewalls, proxies, CDNs)
- TLS built in
- Tooling is mature
- No reinventing the wheel

## Client-to-Server

| Method | Use Case |
|--------|----------|
| HTTPS POST | Send a ping |
| HTTPS GET | Fetch inbox, history |
| WebSocket | Real-time push (new pings, receipts) |

```
wss://ping.acme.ping/stream?token=xxx
```

Client connects once, receives pings in real-time.

## Server Endpoints

Minimal API surface:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/inbox` | POST | Receive incoming pings |
| `/inbox` | GET | Fetch your pings (authenticated) |
| `/keys` | GET | Public keys (optional) |

## Request Authentication

HTTP Message Signatures (RFC 9421) for server-to-server authentication:

```http
POST /inbox HTTP/1.1
Host: ping.other.ping
Content-Type: application/ping
Signature-Input: sig=("@method" "@path" "content-digest");keyid="acme.ping"
Signature: sig=:base64encodedEd25519signature:
```

Receiving server:
1. Fetches `acme.ping`'s public key via DNS
2. Verifies the HTTP signature
3. Accepts or rejects

## Rationale

### Considered Alternatives

| Alternative | Why Not |
|-------------|---------|
| SMTP | Ancient, complex, no encryption by default |
| Matrix | Good but heavy—brings features we don't need |
| libp2p / P2P | NAT traversal is painful; servers are fine for v1 |
| Custom TCP | Firewall hell, unnecessary complexity |

HTTPS is boring. Boring is good for infrastructure.

## Consequences

- All transport is over TLS (mandatory)
- Servers are simple HTTP services
- Real-time features use WebSocket
- No custom protocol to debug
