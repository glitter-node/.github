# Glitter Node

Self-operated infrastructure nodes within the Glitter environment.

Glitter Node operates a physically hosted, layered infrastructure model designed around explicit exposure control and verifiable trust boundaries. All critical components are directly managed without reliance on external CDN or proxy gateways.

---

## System Topology

All public traffic terminates at a single edge layer. Internal services remain loopback-bound and are never directly exposed.

```
Internet
   ↓
Nginx (TLS termination)
   ↓
Application layer (FastAPI instances)
   ↓
Core services (DNS / Mail / Captcha / Data)
```

No third-party TLS termination or reverse proxy layer exists.

---

## Layered Architecture

### Edge Layer
- Direct TLS negotiation
- Automated certificate lifecycle
- HSTS preload, strict CSP, COOP/CORP
- Reverse proxy only within the host

Public exposure:
- 80 / 443

---

### DNS Layer
- Self-hosted authoritative BIND instances
- Multi-node nameserver deployment
- Full DNSSEC (KSK/ZSK separation)
- Automated zone resigning
- DANE / TLSA for SMTP verification

Trust chain:

```
Root → TLD → glitter domain → TLSA → Service endpoint
```

Public exposure:
- 53 TCP/UDP

---

### Mail Layer
- Postfix + Dovecot
- SPF / DKIM / DMARC
- MTA-STS + TLSRPT
- SMTP DANE enforcement

Relay topology:

```
Primary MX → Regional hub → Relay nodes
```

Public exposure:
- 25 / 587 / 993

---

### Application Layer
- Independent FastAPI instances per virtual host
- systemd sandbox restrictions
- Loopback-only binding
- No direct public upstream access

Loopback-only:
- 8000 (application upstream)
- 3306 / 5432 / 6379 (data services)

---

### Internal Verification Module

Self-operated captcha service:
- HMAC-SHA256 token signing
- 120-second TTL expiration
- IP + App-key rate limiting
- No third-party dependency

All verification paths remain internal.

---

## Exposure Model

The infrastructure follows an explicit exposure policy.

Public:
- Edge (HTTP/HTTPS)
- Authoritative DNS
- Mail transport ports

Internal-only:
- Application upstream
- Data services
- Inter-service communication

Internal traffic is bound to loopback and not routable externally.

---

## Operational Automation

- Automated DNSSEC resigning
- Automated certificate renewal
- Centralized logging (systemd + nginx)
- Service-level sandbox hardening
- Minimal open-port discipline

---

## Known Constraints

- No CDN-based DDoS absorption layer
- No global Anycast edge
- Capacity limited to physical host resources
- Operational reliability depends on operator discipline

---

## Design Principle

Security is defined as direct control over every exposed boundary.

Glitter Node represents a minimal-dependency, self-verifiable infrastructure model operated on physical hardware.
