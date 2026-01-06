# Messaging Schema

Version: 0.1.0 (Draft)

## Overview

The messaging schema defines types for human-to-human communication over Ping. It provides familiar email-like semantics while leveraging Ping's modern transport.

## Relationship to Protocol

```
Ping Protocol    →  Transport (envelope, routing, keys, delivery)
Messaging Schema →  Semantics (message types, threads, receipts)
```

This schema builds on top of Ping. Protocol fields (`id`, `from`, `to`, `timestamp`, `signature`) are required. This schema adds messaging-specific fields.

## Types

### messaging.message.v1

A standard message for human communication.

**Schema Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | `messaging.message.v1` |
| `subject` | string | No | Human-readable subject line |
| `thread` | string | No | Thread identifier for grouping |
| `reply-to` | string | No | ID of message being replied to |
| `priority` | string | No | `high`, `normal`, `low` |

**Example:**

```
<meta>
id: msg123
from: @alice/acme.ping
to: @bob/other.ping
timestamp: 2024-01-03T10:00:00Z
signature: ed25519:a1b2c3...

type: messaging.message.v1
subject: Quick question
thread: thread-456
</meta>

Hey Bob,

Do you have time for a call this afternoon?

Thanks,
Alice
```

### messaging.receipt.v1

Delivery or read receipt.

**Schema Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | `messaging.receipt.v1` |
| `receipt-for` | string | Yes | ID of original message |
| `receipt-type` | string | Yes | `delivered`, `read`, `failed` |
| `reason` | string | No | Reason for failure (if failed) |

**Example:**

```
<meta>
id: receipt789
from: @bob/other.ping
to: @alice/acme.ping
timestamp: 2024-01-03T10:05:00Z
signature: ed25519:d4e5f6...

type: messaging.receipt.v1
receipt-for: msg123
receipt-type: read
</meta>
```

### messaging.reaction.v1

Lightweight reaction to a message.

**Schema Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | `messaging.reaction.v1` |
| `reaction-to` | string | Yes | ID of message being reacted to |
| `emoji` | string | Yes | Emoji code (e.g., `thumbsup`, `heart`, `laughing`) |

**Example:**

```
<meta>
id: reaction012
from: @bob/other.ping
to: @alice/acme.ping
timestamp: 2024-01-03T10:06:00Z
signature: ed25519:g7h8i9...

type: messaging.reaction.v1
reaction-to: msg123
emoji: thumbsup
</meta>
```

Emoji codes are short string identifiers. Clients map these to actual emoji characters for display.

### messaging.typing.v1

Typing indicator (ephemeral).

**Schema Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | `messaging.typing.v1` |
| `thread` | string | No | Thread where typing |
| `status` | string | Yes | `started`, `stopped` |

**Example:**

```
<meta>
id: typing345
from: @bob/other.ping
to: @alice/acme.ping
timestamp: 2024-01-03T10:07:00Z
signature: ed25519:j0k1l2...

type: messaging.typing.v1
thread: thread-456
status: started
</meta>
```

## Threading

### Thread Model

Messages can be grouped into threads using the `thread` field:

```
<meta>
...
type: messaging.message.v1
thread: thread-456
</meta>
```

### Replies

Direct replies reference the parent message:

```
<meta>
...
type: messaging.message.v1
thread: thread-456
reply-to: msg123
</meta>

That works for me!
```

### Thread Creation

First message in a thread typically uses its own ID as the thread ID:

```
<meta>
id: msg100
...
type: messaging.message.v1
thread: msg100
subject: Project discussion
</meta>
```

## Attachments

### Attachment Model

Attachments are referenced in schema, fetched separately:

```
<meta>
...
type: messaging.message.v1
subject: Q4 Report
attachments:
  - id: att001
    name: report.pdf
    size: 245000
    content-type: application/pdf
    hash: sha256:abc123...
</meta>

Please review the attached report.
```

### Fetching Attachments

Attachments are fetched via the `fetch` RPC method (same as messages):

```
<meta>
id: req789
from: @bob/other.ping
signature: ed25519:x1y2z3...
method: fetch
params:
  id: att001
  recipient: @bob
</meta>
```

The server returns the attachment content in the response body.

## Priority

Messages can indicate priority:

| Priority | Meaning |
|----------|---------|
| `high` | Urgent, needs attention |
| `normal` | Default |
| `low` | FYI, no rush |

```
<meta>
...
type: messaging.message.v1
subject: Server down!
priority: high
</meta>
```

## Compatibility

### With Email

Messaging schema maps naturally to email concepts:

| Email | Messaging Schema |
|-------|------------------|
| Subject | `subject` |
| Reply-To | `reply-to` |
| Thread-Id | `thread` |
| Attachments | `attachments` |
| Read receipt | `messaging.receipt.v1` |

### With Chat

Also works for chat-style communication:

| Chat | Messaging Schema |
|------|------------------|
| Message | `messaging.message.v1` |
| Reaction | `messaging.reaction.v1` |
| Typing | `messaging.typing.v1` |
| Thread/reply | `thread`, `reply-to` |

## Future Types

Potential additions:

- `messaging.edit.v1` - Edit notification
- `messaging.delete.v1` - Delete notification
- `messaging.presence.v1` - Online/offline status
- `messaging.group.v1` - Group membership changes
