# ADR-006: Encryption and Signatures

## Status

Accepted

## Context

Security is critical for a messaging protocol. We need to address:

- Transport security
- Message authenticity (preventing spoofing)
- Optional end-to-end encryption for sensitive messages

## Decision

- **Transport encryption**: TLS 1.3 mandatory
- **Message signatures**: Ed25519 mandatory
- **End-to-end encryption**: MLS optional

## Layer 1: Transport Encryption (Required)

TLS 1.3 mandatory for all connections:

- Server-to-server: HTTPS
- Client-to-server: HTTPS/WSS
- No plaintext ever

This protects pings in transit.

## Layer 2: Message Signatures (Required)

Every ping is signed by the sender:

```yaml
---
from: @alice/acme.ping
to: @bob/other.ping
signature: ed25519:a1b2c3d4...
---
```

### How It Works

1. Alice's client signs the ping with her private key
2. Bob's server fetches Alice's public key (from `acme.ping` DNS TXT)
3. Verifies signature before accepting

### What This Solves

| Problem | Solution |
|---------|----------|
| Spoofing | Can't fake a `from` address |
| Tampering | Message can't be modified |
| Non-repudiation | Alice provably sent it |

## Layer 3: End-to-End Encryption (Optional)

For sensitive pings, full E2E so even servers can't read the body.

### Recommended: MLS (Messaging Layer Security)

- IETF standard (RFC 9420)
- Designed for federated/group messaging
- Forward secrecy
- Scales to groups

```yaml
---
from: @alice/acme.ping
to: @bob/other.ping
encryption: mls
key-id: abc123
---
[encrypted blob]
```

Only Bob's client can decrypt. Servers just route the blob.

### Alternative

Simpler NaCl box encryption for 1:1 pings if MLS is too heavy.

## Signature Algorithm

**Ed25519** chosen for:

- Fast signing and verification
- Small signatures (64 bytes)
- Small keys (32 bytes public)
- Widely supported
- No configuration needed (unlike RSA key sizes)

## Rationale

### Why Mandatory Signatures?

Email's biggest security failure is spoofing. Anyone can send email claiming to be anyone. By requiring signatures at the protocol level:

- No unsigned pings accepted
- Spoofing eliminated by design
- Trust is cryptographic, not policy-based

### Why Optional E2E?

- Not all messages need E2E (newsletters, receipts)
- E2E adds complexity for clients
- Server-side features (search, AI agents) need readable messages
- Users should choose their security level

## Consequences

- All servers must implement Ed25519 verification
- All clients must implement Ed25519 signing
- Public keys distributed via DNS
- E2E encryption available but not required
- Spam prevention through cryptographic identity
