# Ping Protocol Specification

Version: 0.1.0 (Draft)

## Overview

Ping is a structured messaging protocol. Messages are data—simple enough for humans to read, structured enough for machines to parse.

## Design Principles

| Principle | Description |
|-----------|-------------|
| Structured | Messages are data with typed fields, not text blobs |
| Simple | Minimal spec, easy to implement |
| Extensible | Custom fields and content types for any use case |
| Secure | Cryptographic signatures—no spoofing, no forgery |
| Federated | Decentralized, no single owner |

## Addressing

### Format

```
@<name>/<domain>
```

- `@alice/acme.ping` — full address
- `@alice` — local shorthand (domain inferred)

### URI Scheme

```
ping:@alice/acme.ping
```

## Message Format

Messages use a front matter format with YAML headers and flexible body content.

### Structure

```
---
<yaml front matter>
---
<body>
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique message identifier (UUID v7 recommended) |
| `from` | string | Sender address |
| `to` | string or array | Recipient address(es) |
| `timestamp` | string | ISO 8601 datetime |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `subject` | string | Human-readable subject line |
| `intent` | string | Semantic type: `fyi`, `action`, `reply`, `receipt` |
| `content-type` | string | Body MIME type (default: `text/plain`) |
| `reply-to` | string | ID of parent ping |
| `thread` | string | Thread identifier |
| `expires` | string | ISO 8601 expiration datetime |
| `signature` | string | Cryptographic signature |
| `attachments` | array | List of attachment references |

### Example Message

```yaml
---
id: 01HX2Z3A4B5C6D7E8F9G
from: @alice/acme.ping
to: @bob/acme.ping
timestamp: 2025-01-03T10:30:00Z
subject: Q4 Report Review
intent: action
signature: ed25519:a1b2c3d4...
---
Hey Bob,

Can you review the attached report by Friday?

Thanks,
Alice
```

### File Extension

`.ping`

## Discovery

Server discovery uses DNS SRV and TXT records.

### SRV Record

```dns
_ping._tcp.<domain>. <ttl> IN SRV <priority> <weight> <port> <host>
```

Example:
```dns
_ping._tcp.acme.ping. 300 IN SRV 10 1 443 ping.acme.ping.
```

### TXT Record (Public Key)

```dns
_ping._tcp.<domain>. <ttl> IN TXT "pk=<algorithm>:<base64-key>"
```

Example:
```dns
_ping._tcp.acme.ping. 300 IN TXT "pk=ed25519:base64encodedkey..."
```

### Discovery Flow

1. Resolve `_ping._tcp.<domain>` SRV record
2. Extract host and port
3. Resolve TXT record for public key
4. Connect to server

## Transport

### Server-to-Server

HTTPS POST to the recipient server's inbox:

```http
POST /inbox HTTP/1.1
Host: ping.other.ping
Content-Type: application/ping
Signature-Input: sig=("@method" "@path" "content-digest");keyid="acme.ping"
Signature: sig=:base64signature:

---
from: @alice/acme.ping
to: @bob/other.ping
...
---
<body>
```

### Client-to-Server

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Send ping | POST | `/inbox` |
| Fetch pings | GET | `/inbox` |
| Real-time stream | WebSocket | `/stream` |

### WebSocket Connection

```
wss://<host>/stream?token=<auth-token>
```

## Security

### Transport Encryption

TLS 1.3 required for all connections.

### Message Signatures

All pings must be signed using Ed25519.

```yaml
signature: ed25519:<base64-signature>
```

Signature covers:
- All front matter fields (except signature itself)
- Body content

### Signature Verification

1. Receiving server extracts sender domain from `from` field
2. Queries sender's DNS TXT record for public key
3. Verifies signature
4. Rejects if invalid

### End-to-End Encryption (Optional)

For sensitive messages, use MLS (RFC 9420):

```yaml
---
from: @alice/acme.ping
to: @bob/other.ping
encryption: mls
key-id: abc123
---
<encrypted-blob>
```

## Server API

### Endpoints

| Endpoint | Method | Auth Required | Description |
|----------|--------|---------------|-------------|
| `/inbox` | POST | Server signature | Receive federated pings |
| `/inbox` | GET | User token | Fetch user's pings |
| `/stream` | WebSocket | User token | Real-time ping stream |
| `/keys` | GET | None | Public keys (optional) |

### Response Codes

| Code | Meaning |
|------|---------|
| 202 | Ping accepted |
| 400 | Malformed ping |
| 401 | Authentication failed |
| 403 | Signature verification failed |
| 404 | Recipient not found |
| 429 | Rate limited |

## Delivery Flow

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│ Alice's     │       │ acme.ping   │       │ other.ping  │
│ Client      │       │ Server      │       │ Server      │
└──────┬──────┘       └──────┬──────┘       └──────┬──────┘
       │                     │                     │
       │ 1. Send ping        │                     │
       │ ──────────────────► │                     │
       │   (signed)          │                     │
       │                     │                     │
       │                     │ 2. DNS lookup       │
       │                     │    other.ping       │
       │                     │                     │
       │                     │ 3. POST /inbox      │
       │                     │ ──────────────────► │
       │                     │  (HTTP signature)   │
       │                     │                     │
       │                     │ 4. 202 Accepted     │
       │                     │ ◄────────────────── │
       │                     │                     │
       │                     │                     │ 5. Notify Bob
       │                     │                     │ ──► (WebSocket)
       │                     │                     │
       │                     │ 6. Delivery receipt │
       │                     │ ◄────────────────── │
       │                     │                     │
       │ 7. Receipt          │                     │
       │ ◄────────────────── │                     │
```

## Intent Types

Standard intent values for machine-readable message classification:

| Intent | Description |
|--------|-------------|
| `fyi` | Informational, no action needed |
| `action` | Action required from recipient |
| `reply` | Response to a previous ping |
| `receipt` | Delivery or read receipt |
| `meeting_request` | Calendar invite |
| `approval_request` | Requires approval decision |

## Content Types

Common body content types:

| Content-Type | Use Case |
|--------------|----------|
| `text/plain` | Simple text (default) |
| `text/markdown` | Rich formatted text |
| `application/json` | Structured data |
| `text/html` | Rendered HTML content |

## Spam Prevention

### First-Contact Tokens

Unknown senders must provide a first-contact token:

```yaml
---
from: @unknown/external.ping
to: @alice/acme.ping
first-contact-token: <proof-of-work-or-payment>
---
```

### Reputation

Federated reputation scoring based on:
- Signature validity history
- Recipient feedback
- Server trust relationships

## Versioning

Protocol version in message:

```yaml
---
ping-version: "1.0"
from: @alice/acme.ping
...
---
```

Servers should support backwards compatibility within major versions.

## MIME Type

```
application/ping
```

## References

- [RFC 9421](https://www.rfc-editor.org/rfc/rfc9421.html) - HTTP Message Signatures
- [RFC 9420](https://www.rfc-editor.org/rfc/rfc9420.html) - Messaging Layer Security (MLS)
- [Ed25519](https://ed25519.cr.yp.to/) - High-speed signatures
- [RFC 2782](https://www.rfc-editor.org/rfc/rfc2782.html) - DNS SRV Records
