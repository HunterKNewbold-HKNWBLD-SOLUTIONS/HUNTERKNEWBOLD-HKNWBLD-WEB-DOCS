# Phase 1 — Access Consolidation

## Objective

Eliminate ambiguous access, regain verified admin control, and assign durable ownership for every critical system.

“Consolidated” means:
- every system has a named owner,
- only necessary users have access,
- recovery methods are controlled,
- access changes are documented and reversible.

This is control recovery. It precedes automation.

---

## Scope

Included:
- user/access inventory per system
- removal of unknown/unapproved access
- assignment of primary owner + backup owner (when feasible)
- recovery method validation (email/phone/2FA presence, not the values)
- documentation of roles and access boundaries

Excluded:
- credential storage in documentation
- account recovery walkthroughs
- security bypass methods
- tool migrations not required for stability

---

## Preconditions

- Phase 0 inventory exists (list of systems, URLs, and claimed owners)
- Client designates:
  - **System Owner** (business responsibility)
  - **Operator** (day-to-day)
- Client confirms the “internal docs are public-readable” constraint:
  - do not write secrets, tokens, recovery codes, or PII here

---

## Operating Rules

1. **Least privilege by default**  
   If access is not required for the next 30 days of operations, remove it.

2. **No orphaned ownership**  
   If no one can prove ownership, the system is not stable.

3. **Recovery is part of access**  
   Admin access without controlled recovery is not true ownership.

4. **One primary owner**  
   One accountable person per system. Everyone else is role-based.

---

## Standard Roles (Use These Words)

- **Owner (Business)**: accountable, approves changes, controls recovery
- **Admin (Technical)**: config authority, can add/remove users
- **Operator**: performs routine actions
- **Analyst/Viewer**: read-only
- **Vendor/Agency**: time-bounded access with explicit exit date

Document roles; do not document credentials.

---

## Procedure

### Step 1 — Create the Access Register (Single Source)

Create/update a page or sheet titled: **Access Register** with columns:

- System
- URL / console location
- Primary Owner (person)
- Backup Owner (person, if any)
- Admins (list names only)
- Operators (list names only)
- Viewers (list names only)
- External/Vendor access (name + purpose + end date)
- Recovery controlled by client? (Yes/No/Unknown)
- Notes / risk

**Stop condition**: if you cannot enumerate users for a system, mark it **Unknown** and escalate.

---

### Step 2 — Triage Each User (Keep / Downgrade / Remove)

For each user:
- **Keep** only if required for operations in the next 30 days
- **Downgrade** if they do not need admin-level access
- **Remove** if unknown, inactive, or no longer engaged

Rules:
- Vendor access must be time-bounded.
- “Just in case” access is noncompliant.

Validation:
- Every user has a purpose.
- Every purpose maps to a role.
- No unknown admins remain.

---

### Step 3 — Recovery Control Check (Per System)

For each system, verify (without recording values):
- recovery email exists and is controlled by client
- recovery phone exists and is controlled by client
- 2FA is enabled where supported
- at least one backup method exists (secondary admin or recovery method)

If recovery control is **Unknown**:
- mark system “Not stabilized”
- do not build automation on top of it

---

### Step 4 — System-by-System Access Consolidation Checklists

#### A) Google Business Profile (GBP) + Google Account Access

Goal: the client controls the Google account that controls the GBP.

Checklist:
- Identify the controlling Google account(s)
- Confirm **primary owner** and remove unknown managers
- Confirm manager roles are minimal (avoid granting Owner unless necessary)
- Remove prior agencies unless contractually required
- Document:
  - business profile URL or location identifier
  - owner name(s) (people)
  - operator name(s)
  - vendor end date(s)

Stop conditions:
- The GBP is controlled by a Google account the client does not own.
- Unknown “Owner” role managers exist.

Rollback:
- Restore access from the pre-change user list if removal breaks operations.

---

#### B) Domain Registrar + DNS Provider

Goal: client-owned registrar access and known DNS control.

Checklist:
- Confirm registrar account owner (person)
- Confirm DNS provider (may be registrar or third-party)
- Export DNS records before any removals/changes
- Enumerate registrar users (if supported)
- Remove unknown/unused users
- Confirm renewal responsibility is assigned (role)

Stop conditions:
- registrar ownership cannot be proven
- DNS provider is unknown
- no one can export current DNS records

Rollback:
- revert user changes if operations break
- restore DNS from snapshot if changes were made

---

#### C) Website Hosting / CMS

Goal: at least one client-owned admin, plus documented operators.

Checklist:
- Identify hosting provider and CMS admin surface
- Enumerate users:
  - hosting account users
  - CMS admin users
- Remove unknown CMS admins
- Ensure backups exist (do not document secret keys)
- Assign:
  - technical admin
  - routine operator

Stop condition:
- backups are absent or unverified

Rollback:
- restore from last known backup (reference location, not credentials)

---

#### D) Analytics (GA / Search Console)

Goal: client-owned property/admin with controlled access.

Checklist:
- Confirm property ownership and admin
- Enumerate users, remove unknown
- Downgrade to viewer where appropriate
- Verify tracking works on the live site (baseline observability)

Stop condition:
- no admin access exists
- tracking cannot be verified

Rollback:
- re-add removed viewers/admins from pre-change list

---

#### E) HubSpot (if used)

Goal: no orphaned super-admin and documented role-based access.

Checklist:
- identify **super admins**
- reduce super admins to minimum required
- enforce role-based access for sales/support/operators
- remove unknown users and ex-employees
- verify email domain control for primary admins

Stop conditions:
- unknown super admin exists
- primary admin is a vendor account
- client cannot control account recovery for admin users

Rollback:
- restore users from pre-change list

---

#### F) Zapier (if used)

Goal: automation control is client-owned, auditable, and reversible.

Checklist:
- identify who owns the Zapier workspace
- enumerate users and remove unknown access
- ensure automation changes require approval (process, not product)
- ensure logs exist (where responses are stored)

Stop conditions:
- Zapier is owned by an agency account
- no one can disable the automation quickly
- no audit logging exists for public-facing automation

Rollback:
- disable Zaps (document which Zaps and why; do not store tokens)

---

#### G) Slack (if used)

Goal: operational alerts and escalation are owned and not noisy.

Checklist:
- confirm workspace owner
- confirm dedicated channel for escalations
- restrict who can change workflow automations
- remove unknown users and old vendors where feasible

Stop condition:
- escalation channel not defined
- too many admins (cannot enforce governance)

Rollback:
- restore admin list from pre-change record

---

## Deliverables

- Updated **Access Register**
- Pre-change user lists (stored privately; not embedded here)
- Post-change user lists (stored privately; not embedded here)
- “Stabilization Access Summary” (one-page):
  - what changed
  - what risks remain
  - what is blocked until resolved

---

## Validation Checklist (Pass/Fail)

- [ ] Every system has a primary owner (named person)
- [ ] Unknown admins removed
- [ ] Vendor access time-bounded
- [ ] Recovery control verified (Yes/No, not values)
- [ ] Rollback path documented
- [ ] Client can explain “who owns what” without you

If any checkbox fails: access consolidation is incomplete.

---

## Failure Modes

- “We removed access and broke operations”  
  Cause: no pre-change inventory; fix by restoring access and re-triaging.

- “Client thinks they own it but agency controls recovery”  
  Cause: ownership confusion; fix by ownership transfer and recovery control.

- “Automation runs under vendor ownership”  
  Cause: tool-first setup; fix by moving ownership and adding kill-switch process.

---

## Rollback

Rollback requires:
- pre-change access inventory (who had access, roles)
- system-specific restore mechanism (DNS snapshot, user list restore, Zap disable)

If rollback inventory does not exist, do not make access changes. Stop and escalate.
