# Glitter Node

Self-operated infrastructure running on physical hardware.

Glitter Node documents and operates a layered infrastructure model built around explicit exposure control and direct trust verification. No external CDN, reverse proxy, or third-party TLS gateway is used.

---

## Topology

All public traffic terminates at a single edge.  
Internal services remain loopback-bound.

```
Internet
  ↓
Nginx (TLS)
  ↓
FastAPI instances (loopback)
  ↓
DNS / Mail / Captcha / Data
```

---

## Architecture

### Edge
- Direct TLS negotiation
- Automated certificate lifecycle
- Strict security headers
- No external TLS termination

Public: 80 / 443

---

### DNS
- Self-hosted authoritative BIND
- Multi-node nameservers
- Full DNSSEC (KSK/ZSK)
- DANE / TLSA for SMTP

Trust chain:

```
Root → TLD → Domain → TLSA → Service
```

Public: 53 TCP/UDP

---

### Mail
- Postfix + Dovecot
- SPF / DKIM / DMARC
- MTA-STS + TLSRPT
- SMTP DANE enforcement

Public: 25 / 587 / 993

---

### Application
- Independent FastAPI instances per vHost
- systemd sandbox hardening
- Loopback-only upstream binding

Internal:
- 8000 (app)
- 3306 / 5432 / 6379 (data)

---

### Internal Verification
Self-operated captcha service:
- HMAC-SHA256 tokens
- 120s TTL
- Rate limiting
- No third-party dependency

---

## Exposure Model

Public interfaces are minimal and explicit.  
Internal services are never externally routable.

Trust is established through DNSSEC + DANE rather than proxy delegation.

---

## Operational Discipline

- Automated DNSSEC resigning
- Automated certificate renewal
- Centralized logging
- Service isolation
- Minimal open-port policy

---

## Constraints

- No CDN-based DDoS absorption
- No global Anycast edge
- Capacity bound to physical hardware

---

## Principle

Security is defined as direct control over every exposed boundary.
