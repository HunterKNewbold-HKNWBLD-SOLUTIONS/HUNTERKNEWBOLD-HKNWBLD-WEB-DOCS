# GBP Review Responder — Zapier Architecture (Paths + Logging + Escalation)

## Objective

Implement a Zapier workflow that:
- replies automatically to **3–5 star** GBP reviews,
- generates **draft-only** responses for **1–2 star** reviews,
- blocks and escalates **safety/legal** content,
- logs every output for auditability.

This is a public-facing automation. Reliability and containment matter more than speed.

---

## Preconditions

- Readiness gate passed
- GBP ownership verified and client-controlled
- Zapier workspace ownership verified
- Locked system prompt available (copied verbatim)
- An audit log destination exists (Google Sheet or Airtable)
- An escalation channel exists (Slack or email)

---

## Architecture Summary

**Trigger → Normalize → Route (Paths) → Generate Response → Decide (Auto-post vs Draft) → Log → Notify (if needed)**

Key design principle:
- **Routing decisions occur outside the AI step.**
- The AI step only produces text.

---

## Data Contract (Fields You Must Capture)

From GBP trigger:
- Review ID / Review Name (GBP identifier)
- Reviewer name
- Star rating (number)
- Review text
- Create time / publish time (if available)
- Location (business location ID/name)

Internal workflow fields:
- Route taken (AUTO_POST / DRAFT / BLOCKED_SAFETY)
- Generated response text
- Post status (POSTED / DRAFTED / BLOCKED / FAILED)
- Timestamp of each action

---

## Zap Build (Step-by-Step)

### Step 1 — Trigger: Google Business Profile → New Review

**App:** Google Business Profile  
**Event:** New Review  
**Account:** Client-owned Google account  
**Location:** Select the specific GBP location

**Output requirements:**
- Confirm star rating is a numeric field
- Confirm review ID/name is available
- Confirm review text is available (may be empty for star-only reviews)

---

### Step 2 — Formatter: Normalize Inputs (Recommended)

**App:** Formatter by Zapier  
**Event:** Utilities → Line Item to Text / Text → Replace (choose minimal transforms)

Purpose:
- Coerce star rating to a number (if Zapier returns as text)
- Set an explicit fallback for missing review text

Suggested fields:
- `rating_num` = numeric star rating
- `review_text_safe` = review text or `"(no written comment provided)"`

Fail condition:
- rating cannot be parsed into a number

---

### Step 3 — Safety Keyword Screen (Blocker)

**App:** Filter by Zapier (or Paths if you prefer a single router)  
**Rule:** If `review_text_safe` contains safety/legal keywords → route to **BLOCKED_SAFETY** path

Keyword set (starter):
- safety, unsafe, hurt, injured, injury
- illegal, fraud, stolen, theft
- discrimination, racist, harassment
- police, lawyer, lawsuit, sue
- threat, threatened

Notes:
- Keep this list short and high-signal.
- You are minimizing false negatives, not maximizing automation rate.

If matched:
- do **not** auto-post
- generate draft only or skip generation and escalate directly (your choice)

---

### Step 4 — Router: Paths by Zapier (Required)

Create three paths:

#### Path A — AUTO_POST (3–5 stars, no safety flags)

Entry conditions:
- `rating_num` >= 3
- safety keyword screen did not match

#### Path B — DRAFT (1–2 stars, no safety flags)

Entry conditions:
- `rating_num` <= 2
- safety keyword screen did not match

#### Path C — BLOCKED_SAFETY (safety keyword match)

Entry conditions:
- safety keyword matched (regardless of rating)

---

## Path A — AUTO_POST (3–5 stars)

### A1 — AI: Generate Response Text

**App:** ChatGPT (or AI by Zapier)  
**Model:** use your approved model selection  
**System/Instructions:** paste the **locked system prompt** verbatim  
**User Message:** map:

Reviewer Name: {{Reviewer Name}}
Star Rating: {{Star Rating}}
Review Text: {{review_text_safe}}


**Output:** `response_text`

**Validation check (recommended):**
- Ensure output contains the exact signature line.

### A2 — Filter: Signature Enforcement (Required)

**App:** Filter by Zapier  
Continue only if:
- `response_text` contains exactly:
  `@hknwbld | Hunter KNewbold`

Fail action:
- mark as FAILED
- do not post
- escalate to Slack/email

### A3 — Action: Google Business Profile → Create Reply

**App:** Google Business Profile  
**Event:** Create Reply  
**Review Name/ID:** map from Trigger Step 1  
**Reply Text:** map `response_text`

### A4 — Audit Log Append (Required)

Append a row to your log store (Sheet/Airtable) with:
- Review ID
- Rating
- Review text
- Response text
- Route: AUTO_POST
- Post status: POSTED
- Timestamp

### A5 — Optional: Notification (Recommended)

Post to Slack channel:
- “Posted AUTO reply”
- Include review ID, rating, and a truncated preview (no secrets)

---

## Path B — DRAFT (1–2 stars)

### B1 — AI: Generate Draft Response Text

Same as A1.

### B2 — Signature Enforcement (Required)

Same as A2.
If it fails:
- escalate and stop

### B3 — Audit Log Append (Required)

Append:
- Route: DRAFT
- Post status: DRAFTED
- Include generated response

### B4 — Notify Human (Required)

Send to Slack/email:
- rating + review text
- generated draft response
- instruction: “Human review required. Do not auto-post.”

Important:
- GBP does not always support “draft reply” posting via API. Treat “draft” as “generated text awaiting manual paste/approval.”

Optional:
- create a task in your task system (if you have one). If not, Slack message is sufficient.

---

## Path C — BLOCKED_SAFETY

### C1 — Audit Log Append (Required)

Append:
- Route: BLOCKED_SAFETY
- Post status: BLOCKED
- Include the review text
- Leave response blank or set to “(blocked)”

### C2 — Notify Human (Required)

Slack/email must include:
- “SAFETY BLOCK — DO NOT AUTO-POST”
- rating, review text
- instruction: “Handle manually. Consider offline resolution language. Escalate internally.”

### C3 — Optional: Generate Draft (Only If Desired)

If you choose to generate a draft anyway:
- run AI step with the locked prompt
- do not post
- include draft in Slack message

---

## Reliability Controls

### Duplicate Prevention

Goal: prevent replying twice to the same review.

Implement one of:
- Storage by Zapier “Get Value/Set Value” with Review ID
- Google Sheet lookup “Find Row” by Review ID before acting

Rule:
- If Review ID already exists in the audit log → stop the Zap.

### Failure Notifications

If any of these fail:
- AI step errors
- posting errors
- logging errors
Send an alert to Slack/email with the Review ID and the failed step name.

### Rate Limits / Outages

If GBP or Zapier errors persist:
- disable the Zap (kill switch)
- switch to manual response process until stable

---

## Test Plan (Before Turning On)

Use three controlled tests:

1. **5-star positive**  
Expected: AUTO_POST + logged + signature present

2. **2-star negative**  
Expected: DRAFT + logged + Slack notification + no posting

3. **Safety keyword case** (“injured”, “police”, “lawsuit”)  
Expected: BLOCKED_SAFETY + logged + Slack notification + no posting

Pass criteria:
- No path posts a 1–2 star reply
- Safety keyword case never posts
- All cases write to the audit log
- Signature rule is enforced

---

## Operational Rules

- If audit logging fails, treat as unsafe: stop posting.
- If signature enforcement fails, treat as unsafe: stop posting.
- If routing fails, treat as unsafe: stop posting.
- “Draft” is not a GBP object unless your API supports it; drafts must be handled as human workflow.

---

## Change Control

Any change to:
- keyword set
- routing thresholds
- signature enforcement
- prompt
- logging schema

Requires:
- version note
- test plan rerun
- approval before enabling

