# Glitter Node

Self-operated infrastructure on physical hardware with explicit exposure control and DNSSEC-based trust verification, without external CDN or proxy delegation.

---

## Topology

```
Internet
  ↓
Nginx (TLS)
  ↓
FastAPI (loopback)
  ↓
DNS / Mail / Captcha / Data
```

All internal services are loopback-bound and never externally routable.

---

## Architecture Overview

| Layer | Model | Exposure |
|-------|-------|----------|
| Edge | Direct TLS, strict headers | 80 / 443 |
| DNS | Authoritative BIND, DNSSEC, DANE | 53 TCP/UDP |
| Mail | Postfix, DKIM, DMARC, MTA-STS, DANE | 25 / 587 / 993 |
| App | FastAPI per vHost, systemd sandbox | loopback (8000) |
| Data | DB / Cache | loopback (3306 / 5432 / 6379) |

---

## Internal Verification

- HMAC-SHA256 tokens  
- 120s TTL  
- Rate limiting  
- No third-party dependency  

---

## Exposure Model

Public interfaces are minimal and explicit.  
Internal services are never externally routable.  
Trust is enforced through DNSSEC + DANE rather than proxy delegation.

---

## Infrastructure Discipline

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
Delegation is minimized; verification remains local.

---

## Verification Commands

### DNSSEC
```bash
dig -4 +dnssec +multi glitter.kr SOA
dig -4 +dnssec +multi glitter.kr DNSKEY
dig -4 +dnssec +multi glitter.kr DS
delv -4 +vtrace A glitter.kr
```

### DANE / TLSA
```bash
dig -4 +dnssec +multi _587._tcp.mail.glitter.kr TLSA
dig -4 +dnssec +multi _443._tcp.captcha.glitter.kr TLSA
```

### SMTP / MTA-STS
```bash
dig -4 +dnssec +multi _mta-sts.glitter.kr TXT
curl -s https://mta-sts.glitter.kr/.well-known/mta-sts.txt
openssl s_client -starttls smtp -connect smtp.glitter.kr:587 -servername smtp.glitter.kr -brief < /dev/null
```
