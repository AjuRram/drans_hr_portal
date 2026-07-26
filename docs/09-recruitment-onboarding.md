# Recruitment & Onboarding

[← Back to index](../README.md)

---

## Part 1 — Recruitment (ATS)

### Pipeline

```
Candidate added / applies
      │
      ▼
  Total candidates ──► Interview scheduled ──► In progress (rounds)
                                                    │
                                        ┌───────────┴───────────┐
                                        ▼                       ▼
                                    Selected                Rejected
                                        │                  (with reason)
                                        ▼
                              Onboarding initiated
```

| Scheduled candidates | Full pipeline |
|---|---|
| ![Scheduled](../screenshots/recruitment-scheduled.png) | ![Total](../screenshots/recruitment-total.png) |

### Capabilities

| Feature | Detail |
|---|---|
| Candidate records | Details, résumé storage, application tracking |
| Interview scheduling | Date, time, panel, round |
| Teams integration | Microsoft Teams meetings created from the platform |
| Automated email | Invitations and per-round communication |
| Round tracking | Multi-round progress with per-round status |
| Rejection reasons | Recorded against the candidate |
| Status views | Separate views for total, scheduled, in-progress, selected, rejected |

### Selected → employee

A selected candidate is **converted**, not re-entered. Their captured data carries into onboarding
and then into the employee master. The name, contact details and résumé collected at application
are the same records the employee record is built from — eliminating the transcription step where
a phone number or PAN typically gets mistyped.

---

## Part 2 — Onboarding

![Onboarding](../screenshots/onboarding.png)

### The problem it solves

A new hire needs to submit personal details, bank details, PAN, Aadhaar, education certificates and
a signed joining form — **before their first day**, and therefore before they have a company
account. Creating credentials for someone who has not yet joined is both an administrative burden
and a security question.

### Token-based passwordless flow

```
HR initiates onboarding for a selected candidate
      │
      ▼
System generates a signed, single-purpose token
      │
      ▼
Secure link emailed to the candidate
      │
      ▼
Candidate opens the link — no account, no password
      │
      ├─► Completes the joining form
      ├─► E-signs the document
      └─► Uploads KYC documents
      │
      ▼
HR reviews submission ──► marks Joined ──► employee record created
                     └──► marks Dropped ──► pipeline closed
```

The token in the URL *is* the credential. It is scoped to a single application and a single purpose,
so it grants exactly the ability to complete one onboarding — nothing else. These routes are among
the few explicitly whitelisted from JWT authentication, precisely because the person using them has
no identity in the system yet.

### Capabilities

| Feature | Detail |
|---|---|
| Multi-page joining form | Structured sections, resumable |
| E-signature | Digital signing of the joining document |
| KYC upload | PAN, Aadhaar, education, bank proof — tracked per item |
| Preview | HR previews the submission before accepting |
| Reminders | Automated nudges for incomplete onboarding |
| Joined / Dropped | Explicit resolution either way |
| Document store | Files retained against the employee record |

### Reminders

Onboarding stalls are common — a candidate starts the form and does not finish. `OnboardingReminderLog`
tracks automated follow-ups, so incomplete onboarding is chased by the system rather than depending
on an HR executive remembering to check.

---

## Advantages

| Capability | Value |
|---|---|
| Single pipeline, application → employee | No re-keying; data entered once |
| Teams + email automation | Scheduling overhead removed from HR |
| Passwordless onboarding | New hires need no account before joining |
| Purpose-scoped tokens | Access limited to one onboarding, nothing more |
| Pre-first-day KYC | Day one is productive, not paperwork |
| Automated reminders | Stalled onboarding is chased automatically |
| Explicit Joined / Dropped | No candidates left in limbo |
| Documents retained | KYC available later for statutory needs |
