# Canonical tags (IDOR)

Normalized 2026-11-16. Use these spellings in `Tags:` lines.

- Reference type: numeric-id, uuid, graphql, encoded-id
- Mechanism: mass-assignment, session-misbinding, cross-tenant, new-feature, price-manipulation, business-logic
- Action/impact: pii-disclosure, account-takeover, delete, change-email, forgot-password, otp
- Access: api, unauth, enumeration, wayback-recon
- Meta: methodology, bypass, escalation

Notes:
- business-logic / price-manipulation (note 07 Acronis): trusted-client value tampering, NOT object-ownership IDOR. Use as a false-positive example in SKILL.md section 5.
