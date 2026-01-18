# Contributing

This repo is operated through branches and pull requests. The goal is change control, not bureaucracy.

## Branch rules

- Branch from `main`.
- Use short, explicit names:
  - `docs/<topic>`
  - `playbook/<phase>-<topic>`
  - `standard/<area>`

## Pull request requirements

Every PR must:
1. increase clarity
2. reduce ambiguity
3. improve repeatability
4. avoid secrets and client PII

### Required PR checklist

- [ ] No credentials, tokens, recovery codes, billing details, or account takeover procedures
- [ ] No client PII (names tied to private issues, phone/email, invoices, addresses)
- [ ] SOP/playbook structure present (Objective → Scope → Preconditions → Steps → Validation → Failure modes → Rollback → Handoff)
- [ ] Links resolve (no dead internal references)
- [ ] `mkdocs build --strict` passes
- [ ] Any automation prompt change includes:
  - version header
  - failure modes
  - rollback note

## Writing standards

- Active voice.
- One idea per sentence.
- Concrete nouns and verbs.
- Minimal adjectives.
- No urgency language.
- No “tool-first” framing.

