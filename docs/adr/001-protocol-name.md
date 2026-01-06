# ADR-001: Protocol Name Selection

## Status

Accepted

## Context

We needed a name for a new messaging protocol designed to replace email in the AI era. The name needed to:

- Work as both a noun and verb (like "email me" or "send me an email")
- Be short, memorable, and globally pronounceable
- Feel modern without being tied to AI terminology
- Be distinct from existing protocols

## Decision

We chose **Ping** as the protocol name.

## Rationale

### Linguistic Flexibility

| Usage | Example |
|-------|---------|
| Verb | "Ping me when you're free" |
| Noun | "Did you get my ping?" |
| Forward | "I'll forward you that ping" |

### Strengths

- **Already a verb**: "Ping me" is universal—zero learning curve
- **Short & punchy**: 4 letters, one syllable, easy in any language
- **Tech-native**: Developers and tech-savvy users already understand it
- **Implies speed**: A ping is instant, lightweight, efficient
- **Friendly tone**: Casual, approachable, human

### Considered Alternatives

| Name | Rationale | Why Not Chosen |
|------|-----------|----------------|
| Nex | Short for Nexus, modern, ownable | Less familiar, requires learning |
| Vox | Latin for "voice", elegant | More formal, less action-oriented |
| Relay | Implies intelligent routing | More formal, longer |
| Pulse | Fast, alive, rhythmic | Doesn't work well as "pulse me" |

### Addressing the Network Ping Concern

While "ping" exists as a network diagnostic tool (ICMP), context separates them easily. Google meant search before it became a company name. The protocol can redefine an existing term.

## Consequences

- File extension: `.ping`
- URI scheme: `ping:`
- Natural adoption path due to existing familiarity with "ping me"
