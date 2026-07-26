# Employee Self-Service Workflow

[← Back to index](../README.md)

The employee portal is a deliberately small surface. An employee is not an HR operator — they have
perhaps six recurring needs, and the portal is built around exactly those.

---

## Design principle

Every question an employee would otherwise ask HR by email should be answerable in the portal in
under three clicks. "How many leaves do I have left?" "Where's my payslip?" "Has my declaration been
approved?" "Which tax regime should I pick?" Each of these is a self-service answer, and each one
answered is an email HR does not have to write.

---

## Logging in

![Employee login](../screenshots/login-employee.png)

Employees log in with their **Employee ID or registered mobile number** — the backend auto-detects
which was typed — and a **6-digit OTP** delivered over WhatsApp or email.

**Why passwordless:**

| Password login | OTP login |
|---|---|
| Employee must remember a credential | Nothing to remember |
| Password reset requests reach HR constantly | No reset flow exists |
| Reused across sites; breach elsewhere is a breach here | Nothing reusable |
| Stored (hashed) and can leak | Nothing stored to leak |
| Shared between colleagues | Bound to a personal device |

The OTP is single-use, short-lived, held in Redis, and deleted on verification. Even a valid
in-flight OTP will not mint a token for a resigned employee — the lifecycle gate is checked again at
verification time.

---

## 1. Home

![Employee home](../screenshots/emp-home.png)

The landing view is a personal status summary:

- **Attendance tiles** — present days, leaves, half days and weekends in the current cycle
- **Attendance calendar** — the live 26 Jul → 25 Aug cycle with a colour-coded legend
  (present / leave / half-day / weekend), switchable between cycle and calendar month
- **Leave balance** — sick and casual leave remaining, with a direct *Apply for Leave* action
- **Recent requests** — status of what they have already submitted
- **Food allowance** — accrued amount for the cycle
- **Market holidays** — the company holiday calendar
- **Header** — shift, current time, weather, and **monthly CTC masked behind a reveal toggle**

The CTC masking is intentional. Salary is visible on demand but not exposed to anyone glancing at a
screen in an open-plan office.

**Use case.** An employee wants to take Friday off. They open the portal, see 10 casual leaves
remaining, click *Apply for Leave*, and submit — without messaging HR to ask their balance first.

---

## 2. Attendance

![Employee attendance](../screenshots/emp-attendance.png)

The employee's own attendance record: cycle view, monthly view and a year summary, showing exactly
what payroll will use. Because employees can see the same data payroll consumes, a missing punch is
usually reported by the employee before it becomes an incorrect salary — the discrepancy surfaces
days earlier than it otherwise would.

### Team attendance

![Team attendance](../screenshots/emp-team-attendance.png)

Reporting managers get a team view — who is present today — without being given the full admin
console.

---

## 3. Leave

![Employee leaves](../screenshots/emp-leaves.png)

**Flow:** apply (dates, type, reason, optional CC recipients) → routed to the reporting manager →
approved or rejected with a reason → balance updated → attendance and payroll reflect it.

Employees can **withdraw** a pending request themselves. CC recipients are notified without being
approvers, which mirrors how leave is actually communicated in a team.

---

## 4. Payslips

![Employee payslip](../screenshots/emp-payslip.png)

All published payslips, viewable and downloadable as PDF, with full earnings and deductions
breakdown. Employees see a payslip only after HR **publishes** it — drafts under review are never
visible.

Historical payslips remain permanently available, which removes the recurring "can you resend my
March payslip for my loan application?" request.

---

## 5. Tax declarations

![Employee declarations](../screenshots/emp-declarations.png)

Each financial year, employees declare planned investments and expenses so TDS is deducted at an
accurate rate rather than the maximum.

**Flow**
1. Enter declarations by section — 80C, 80D, 80CCD(1B), 80TTA, 24(b) home loan interest, 80E, HRA
2. Upload proof documents against each claim
3. Submit for HR review
4. HR approves or rejects each item with a reason
5. Approved amounts flow into the TDS computation and later into Form 16 Part B

Rejected items are returned with the reason visible, so the employee can correct and resubmit
instead of discovering the problem in their Form 16 a year later.

---

## 6. Form 16

![Employee Form 16](../screenshots/emp-form16.png)

Published Form 16 certificates by financial year, downloadable as **Part A, Part B, or a combined
PDF**.

If an employee finds an error — wrong PAN, missing exemption, incorrect TDS — they raise a
**rectification request** from this screen. It lands in the HR queue, is corrected, and the
certificate is reissued. The whole loop is tracked in the system rather than conducted over email.

---

## 7. Tax calculator

![Employee tax calculator](../screenshots/emp-tax-calculator.png)

A four-step wizard — **Basic Info → Income → Deduction → Summary** — that computes liability under
both regimes side by side.

Key behaviours:
- **Pre-filled from the salary master.** The employee's actual salary is loaded automatically, so
  the first answer they see is about their real situation, not a blank form.
- **Live slab reference** alongside the form, switchable between new and old regime and between
  AY 2027-28 and AY 2026-27.
- **Clear** resets to a blank state for what-if modelling.
- **Recommendation.** The summary states which regime produces lower tax for their numbers.

This is the same engine that drives payroll TDS and Form 16 Part B — so the number an employee sees
here is the number the company will actually deduct. See [Tax Calculator](07-tax-calculator.md).

**Use case.** In April an employee must elect a regime. Rather than guessing or asking HR, they open
the calculator — already populated with their salary — enter their planned 80C investment, and see
that the old regime saves them ₹18,000. They elect accordingly and declare that investment.

---

## 8. Profile

![Employee profile](../screenshots/emp-profile.png)

Personal details, employment information, documents and assets assigned to them. Employees can
update their photo and password-protected fields; identity and salary fields are read-only, since
those are HR-controlled records.

---

## 9. Support

An in-portal helpdesk. Employees raise a ticket, exchange threaded messages with HR, and track
status — with an AI assistant handling common questions and handing off to a human when it cannot.

---

## Session handling

Sessions **auto-expire after 30 minutes of inactivity**, coordinated across tabs via shared
activity timestamps so one active tab keeps the session alive and a genuinely abandoned session
closes itself. Tokens expire after 24 hours regardless.

---

## Employee's year at a glance

| When | What the employee does |
|---|---|
| Daily | Check attendance; apply for leave as needed |
| Monthly | Download payslip |
| April | Elect tax regime using the calculator |
| Apr–Jan | Submit investment declarations with proofs |
| Jun onward | Download Form 16; raise a rectification if something is wrong |
| Anytime | Update profile, view assets, raise a support ticket |
