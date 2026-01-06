# ADR-002: Address Format

## Status

Accepted

## Context

We needed an addressing scheme for Ping that:

- Is distinct from email's `user@domain.com`
- Puts the name first (human-readable)
- Supports local shorthand within the same domain
- Is URL-friendly with a URI scheme

## Decision

Use the format `@name/domain.ext` with the domain portion optional for local addressing.

## Format Specification

```
@alice/acme.ping      → full address
@alice                → local (domain inferred from context)
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
| Name prominence | Domain feels equal | Name comes first |

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

## Consequences

- Addresses are always prefixed with `@`
- Domain uses `/` separator, not `@`
- `.ping` is the recommended domain suffix (convention, not TLD requirement)
- Clients display shorthand; protocol uses full addresses
