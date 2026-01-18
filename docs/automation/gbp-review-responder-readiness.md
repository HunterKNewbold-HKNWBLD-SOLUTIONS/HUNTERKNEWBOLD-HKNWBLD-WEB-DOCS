# Automation Readiness Gate — Google Business Profile Review Responder

## Purpose

Prevent premature or unsafe automation of public-facing Google Business Profile (GBP) review responses.

This gate exists to ensure:
- ownership and access are stable,
- failure modes are understood,
- escalation paths are enforced,
- automation does not create reputational or legal risk.

Passing this gate is mandatory before any Zapier- or AI-driven response is activated.

---

## Automation in Scope

- Automated generation of GBP review replies
- Conditional auto-posting for positive/neutral reviews
- Draft-only generation for negative or sensitive reviews
- Audit logging of all generated responses

Out of scope:
- dispute handling
- legal responses
- suspension appeals
- review solicitation

---

## Hard Prerequisites (All Required)

### 1. Stabilization Complete

- Phase 1 — Stabilization is complete and signed off
- Phase 1 — Access Consolidation is complete
- No unresolved stop conditions remain

If stabilization is incomplete, automation is blocked.

---

### 2. GBP Ownership & Control

Verify and document (names only, no credentials):

- [ ] Client-owned Google account is **Primary Owner** of the GBP
- [ ] No unknown Owners or Managers exist
- [ ] Prior agencies removed or time-bounded
- [ ] Client controls Google account recovery methods

Fail condition:
- GBP ownership resides with a vendor or unknown account

---

### 3. Automation Ownership & Kill Switch

- [ ] Zapier workspace is client-owned or contractually transferable
- [ ] Named human owner for the automation exists
- [ ] Automation can be disabled within 60 seconds
- [ ] Client knows how to turn it off without vendor assistance

Fail condition:
- Automation cannot be disabled quickly
- Automation runs under vendor-only ownership

---

## Behavioral Constraints (Non-Negotiable)

### 4. Response Policy Alignment

Automation must comply with the **Auto Responder Compendium**:

- Professional, calm, human tone
- No arguments, blame, or legal admissions
- Plain text only
- No links
- No emojis
- No mention of AI, automation, or internal policies
- Responses between defined word limits
- Mandatory signature enforced exactly

Fail condition:
- Any response can be generated without the mandatory signature
- Any response can exceed or ignore length limits

---

### 5. Sentiment Routing Rules

Automation must implement sentiment-aware routing:

- **4–5 stars** → auto-generate and auto-post
- **3 stars** → auto-generate and auto-post
- **1–2 stars** → generate draft only; human approval required

Fail condition:
- 1–2 star reviews can be posted automatically

---

### 6. Safety & Legal Escalation

Automation must detect and block posting if review text includes references to:

- safety incidents
- illegal activity
- discrimination or harassment
- threats or coercion

Blocked reviews must:
- generate a draft response only
- trigger human notification (Slack/email)
- never auto-post

Fail condition:
- safety-related reviews can be posted without human review

---

## Technical Readiness Checks

### 7. Prompt Governance

- [ ] System prompt is locked and versioned
- [ ] Prompt includes explicit behavioral constraints
- [ ] Prompt includes mandatory signature rule
- [ ] Prompt version is documented

Fail condition:
- Prompt can be modified without review
- Multiple conflicting prompt versions exist

---

### 8. Audit Logging (Required)

Every generated response must be logged with:

- Review ID
- Review date/time
- Star rating
- Original review text
- Generated response text
- Automation path taken (auto-post / draft / blocked)
- Timestamp of action

Logs must be:
- immutable or append-only
- client-accessible
- retained for review

Fail condition:
- Responses are generated or posted without an audit trail

---

### 9. Error Handling & Observability

- [ ] Zap failures are detectable
- [ ] Failed generations are surfaced to a human
- [ ] Duplicate posting is prevented
- [ ] Rate limits and API errors are handled safely

Fail condition:
- Silent failure
- Duplicate or partial responses posted publicly

---

## Platform-Specific Guardrails (Zapier)

### Required Zap Architecture

- Trigger: New GBP Review
- Filter/Paths:
  - rating-based routing
  - keyword-based safety routing
- AI step:
  - locked system prompt
  - constrained output
- Action:
  - Create Reply (only on approved paths)
- Logging:
  - Google Sheet / Airtable append step
- Notification:
  - Slack/email for drafts and blocked cases

Fail condition:
- Single linear Zap with no routing or logging

---

## Validation Checklist (All Must Pass)

- [ ] GBP ownership verified and client-controlled
- [ ] Automation owner identified
- [ ] Kill switch confirmed
- [ ] Sentiment routing enforced
- [ ] Safety escalation enforced
- [ ] Prompt locked and versioned
- [ ] Audit logging active
- [ ] Human review path tested
- [ ] Client trained on “when automation stops”

If any item fails: **automation is not approved**.

---

## Go / No-Go Decision

Document one outcome:

- **GO** — automation may be enabled
- **NO-GO** — list blocking issues and remediation steps

No automation is assumed. Explicit approval is required.

---

## Rollback

If automation causes harm or uncertainty:

1. Disable the Zap immediately
2. Revert to manual responses
3. Review audit log to identify scope of impact
4. Document the failure mode
5. Do not re-enable until a control change is documented

---

## Enforcement Rule

Automation is a privilege, not a default.

If readiness cannot be proven, the correct action is **not to automate**.


