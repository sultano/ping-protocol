# ADR-003: Message Format

## Status

Accepted

## Context

We needed a message format that:

- Is human-readable and human-writable
- Supports structured metadata for machines
- Allows flexible body content (text, markdown, JSON, etc.)
- Is easy to debug and version control
- Avoids the complexity of email's MIME format

## Decision

Use a front matter format with YAML headers followed by a flexible body, similar to Jekyll/Hugo markdown files.

## Format Specification

```
---
[YAML front matter]
---
[Body: any content type]
```

### Rules

1. File starts with `---`
2. Front matter ends with `---`
3. Everything after is the body
4. Front matter declares `content-type` if body isn't plain text

### File Extension

`.ping`

## Examples

### Simple Text Ping

```
---
from: @alice/acme.ping
to: @bob/acme.ping
subject: Lunch?
---
Free at noon?
```

### Structured Action (JSON body)

```
---
from: @alice.agent/acme.ping
to: @bob.agent/othercorp.ping
intent: meeting_request
content-type: application/json
---
{
  "action": "schedule_meeting",
  "title": "Project Sync",
  "proposed_times": ["2025-01-05T10:00Z", "2025-01-05T14:00Z"],
  "duration_minutes": 30
}
```

### Rich Content (Markdown body)

```
---
from: @alice/acme.ping
to: @team/acme.ping
subject: Weekly Update
content-type: text/markdown
---
# Weekly Update

## Done
- Finished onboarding docs
- Fixed login bug

## Blocked
- Waiting on API access
```

## Front Matter Fields

### Required

| Field | Description |
|-------|-------------|
| `from` | Sender address |
| `to` | Recipient(s) — array or single |
| `id` | Unique message ID |
| `timestamp` | ISO 8601 datetime |

### Optional

| Field | Description |
|-------|-------------|
| `subject` | Human-readable subject |
| `intent` | Semantic type: `fyi`, `action`, `reply`, `receipt` |
| `content-type` | Body MIME type (default: `text/plain`) |
| `reply-to` | ID of parent ping |
| `thread` | Thread ID |
| `expires` | TTL for ephemeral pings |
| `signature` | Cryptographic signature |
| `attachments` | List of attachment references |

## Rationale

### Why Not Pure JSON?

| Advantage | Details |
|-----------|---------|
| Human-readable | Open a ping in any text editor |
| Human-writable | Compose in Notepad if needed |
| Flexible body | Plain text, Markdown, HTML, JSON |
| Familiar | Developers know this pattern |
| Diffable | Easy to version control |
| Graceful degradation | Even if parsing fails, humans can read it |

### YAML vs TOML

We chose YAML for familiarity, but strictly specced (no anchors, no complex types). TOML is acceptable as an alternative.

## Consequences

- Messages are stored as `.ping` files
- Any text editor can create valid pings
- Structured data goes in the body with appropriate `content-type`
- AI agents can parse front matter without NLP
