# Security & Data Policy

Treat this site as publicly readable.

## Red lines (never document here)

- passwords, tokens, API keys, backup codes
- account recovery procedures
- billing details, invoices, payment disputes
- client PII (phone/email/address, private incident details tied to identity)
- internal-only pricing experiments tied to specific clients

## Allowed (with constraints)

- checklists, runbooks, templates
- high-level integration patterns (sanitized)
- standards and definitions
- generic examples with fake data

## Sanitization requirements

- use fake names and fake domains
- remove IDs, account numbers, and screenshots of dashboards
- redact anything that would enable takeover

## Enforcement

If a page violates these rules, revert it immediately and treat as an incident.

