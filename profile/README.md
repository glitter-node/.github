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

---

## Verification Commands

The following commands can be used to validate DNSSEC, DANE, MTA-STS, and TLS negotiation.

```bash
dig -4 +dnssec +multi glitter.kr SOA
dig -4 +dnssec +multi glitter.kr DNSKEY
dig -4 +dnssec +multi glitter.kr DS
dig -4 +dnssec +multi glitter.kr TXT
dig -4 +dnssec +multi glitter.kr CAA
dig -4 +trace glitter.kr @a.root-servers.net
dig -4 +dnssec +multi _smtp._tls.glitter.kr TXT
dig -4 +dnssec +multi _587._tcp.mail.glitter.kr TLSA
dig -4 +dnssec +multi _443._tcp.captcha.glitter.kr TLSA
dig -4 +dnssec +multi glitter.kr SOA @1.1.1.1
dig -4 +dnssec +multi glitter.kr DNSKEY @1.1.1.1
dig -4 +dnssec +multi glitter.kr DS @1.1.1.1
dig -4 +dnssec +multi _smtp._tls.glitter.kr TXT @1.1.1.1
dig -4 +dnssec +multi _587._tcp.mail.glitter.kr TLSA @1.1.1.1
dig -4 +dnssec +multi _443._tcp.captcha.glitter.kr TLSA @1.1.1.1
dig -4 +dnssec +multi +nocmd +noall +answer +comments glitter.kr A
dig -4 +dnssec +multi glitter.kr DS @203.248.252.2
dig -4 +dnssec +multi _mta-sts.glitter.kr TXT
delv -4 +vtrace A glitter.kr
delv -4 nonexistent123.glitter.kr A
delv -4 @1.1.1.1 +vtrace DS glitter.kr
delv -4 @1.1.1.1 _587._tcp.mail.glitter.kr TLSA
delv -4 @1.1.1.1 _443._tcp.captcha.glitter.kr TLSA
curl -s https://mta-sts.glitter.kr/.well-known/mta-sts.txt
openssl s_client -starttls smtp -connect smtp.glitter.kr:587 -servername smtp.glitter.kr -brief < /dev/null
```
