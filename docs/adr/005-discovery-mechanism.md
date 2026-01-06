# ADR-005: Discovery Mechanism

## Status

Accepted

## Context

When sending a ping to `@bob/other.ping`, we need to discover:

1. Which server handles `other.ping`
2. The server's public key for signature verification

We explicitly decided against using `.well-known` endpoints in favor of pure DNS-based discovery.

## Decision

Use DNS SRV and TXT records for all discovery.

## DNS Records

For domain `acme.ping`:

```dns
; Server discovery
_ping._tcp.acme.ping. 300 IN SRV 10 1 443 ping.acme.ping.

; Server public key (for signature verification)
_ping._tcp.acme.ping. 300 IN TXT "pk=ed25519:base64..."

; Optional: backup server
_ping._tcp.acme.ping. 300 IN SRV 20 1 443 backup.acme.ping.
```

### SRV Record Format

```
_ping._tcp.<domain>. SRV <priority> <weight> <port> <host>
```

### TXT Record Format

```
_ping._tcp.<domain>. TXT "pk=<algorithm>:<base64-public-key>"
```

## Discovery Flow

To ping `@bob/other.ping`:

1. Query `_ping._tcp.other.ping` SRV record
2. Get host and port (e.g., `ping.other.ping:443`)
3. Query TXT record for public key
4. POST the ping to `https://ping.other.ping/inbox`

## Rationale

### Why DNS Over .well-known?

| Approach | Pros | Cons |
|----------|------|------|
| DNS SRV/TXT | Single lookup, standard infra, cached | Key rotation requires DNS update |
| .well-known | Easy to update, richer metadata | Extra HTTP roundtrip, another endpoint |

DNS is:
- Already required for hostname resolution
- Universally cached
- Doesn't require server implementation changes
- Simpler server footprint

### Why Not .well-known?

- Adds HTTP dependency to discovery
- Another endpoint to maintain
- Another potential point of failure
- DNS already solves this problem

## Key Management

Primary key stored in DNS TXT record. For key rotation:

1. Add new key to TXT record
2. Sign with both keys during transition
3. Remove old key after transition period

Optional `/keys` endpoint can provide richer key metadata if needed.

## Consequences

- One DNS query provides host + port + public key
- Servers don't need `.well-known` endpoints
- Key rotation requires DNS updates
- Relies on DNS infrastructure (DNSSEC recommended)
