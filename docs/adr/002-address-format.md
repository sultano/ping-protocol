# ADR-002: Address Format

## Status

Accepted

## Context

We needed an addressing scheme for Ping that:

- Is distinct from email's `user@domain.com`
- Puts the human/agent name first
- Supports local shorthand within the same domain
- Works for both humans and AI agents
- Is URL-friendly with a URI scheme

## Decision

Use the format `@name/domain.ext` with the domain portion optional for local addressing.

## Format Specification

```
@alice/acme.ping      → full address
@alice                → local (domain inferred from context)
```

### Agent Addressing

Agents use the same format with a `.agent` suffix on the name:

```
@alice/acme.ping           → Alice (human)
@alice.agent/acme.ping     → Alice's AI agent
@support.agent/acme.ping   → Company's support agent
```

### URI Scheme

For links and protocol handlers:

```
ping:@alice/acme.ping
```

Analogous to `mailto:alice@example.com`.

## Rationale

### Comparison with Email

| Aspect | Email | Ping |
|--------|-------|------|
| Format | `alice@domain.com` | `@alice/domain.ping` |
| URI scheme | `mailto:` | `ping:` |
| Name prominence | Domain feels equal | Name comes first—human-centric |
| Agent-ready | Hackish (`alice+bot@`) | Native (`@alice.agent/`) |

### Why `/` Instead of `@`

- Visually distinct from email
- No `@user@domain` confusion
- `/` is a natural hierarchy separator
- URL concerns resolved via `ping:` URI scheme

### Local Shorthand

Within `acme.ping`, users can simply use `@alice` instead of the full `@alice/acme.ping`. The protocol always uses full addresses internally.

## Examples

| Scenario | Address |
|----------|---------|
| Ping a colleague locally | `@bob` |
| Ping someone external | `@bob/othercorp.ping` |
| Ping your own agent | `@me.agent` |
| Ping a company's AI | `@support.agent/acme.ping` |

## Consequences

- Addresses are always prefixed with `@`
- Domain uses `/` separator, not `@`
- `.ping` is the recommended domain suffix (convention, not TLD requirement)
- Clients display shorthand; protocol uses full addresses
