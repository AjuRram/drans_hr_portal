# ApexHR — HR & Payroll Management Platform

> A production HRIS + Indian payroll system covering the full employee lifecycle:
> recruitment → onboarding → attendance → leave → payroll → statutory tax → Form 16.

This repository is a **documentation and specification archive**. It contains the functional
specification, module workflows, architecture notes and annotated screenshots of the platform.
It does not contain application source code, configuration, or credentials.

---

## Contents

| Document | What it covers |
|---|---|
| [Architecture & Tech Stack](docs/01-architecture.md) | System design, data model, request lifecycle, deployment |
| [Admin / HR Workflow](docs/02-admin-workflow.md) | Every screen an HR administrator uses, end to end |
| [Employee Workflow](docs/03-employee-workflow.md) | The self-service portal from the employee's side |
| [Payroll Module](docs/04-payroll.md) | Salary structure, the 6-stage payroll run, payslips |
| [Form 16 Module](docs/05-form16.md) | Part A/B generation, TDS reconciliation, publish flow |
| [Attendance Module](docs/06-attendance.md) | The 26→25 cycle, biometric import, LOP |
| [Tax Calculator](docs/07-tax-calculator.md) | Old vs new regime engine, slabs, deduction caps |
| [Leave Management](docs/08-leave-management.md) | Request → approve → balance → payroll impact |
| [Recruitment & Onboarding](docs/09-recruitment-onboarding.md) | ATS pipeline and passwordless onboarding |
| [Security & Access Control](docs/10-security-rbac.md) | JWT auth, three-tier authorisation, data scoping |
| [API Reference](docs/11-api-reference.md) | All 174 endpoints grouped by module |
| [Screenshot Gallery](docs/12-screenshot-gallery.md) | Every screen in one place |

---

## What the platform does

ApexHR replaces the spreadsheet-and-email workflow most small-to-mid Indian companies use to run HR.
The design premise is **one employee record, read by every module** — a salary revision entered once
is immediately correct in the payslip, the Form 16, the tax projection and the CTC report, with no
re-keying and no reconciliation step.

### Modules at a glance

| Module | Purpose |
|---|---|
| **Core HR** | Employee master, documents, assets, org structure (branch / location / manager / team head) |
| **Recruitment** | Candidate pipeline, interview scheduling, Teams meetings, offer → joined |
| **Onboarding** | Passwordless joining forms, e-signature, KYC document upload |
| **Attendance** | Biometric import, 26→25 pay cycle, LOP, calendars, exports |
| **Leave** | Request/approve chain, CC recipients, balances, team views |
| **Payroll** | CTC structure, salary calculation, payslip PDFs, bulk email delivery |
| **Form 16** | Part A + Part B generation, quarterly TDS, reconciliation, publish/revoke |
| **Tax** | Employee declarations with proof upload, admin review, old vs new regime calculator |
| **Engagement** | Birthday/anniversary automation, WhatsApp Business inbox, support helpdesk |

---

## Scale

Measured from the codebase:

| Metric | Value |
|---|---|
| Backend (Python, excl. dependencies) | **27,626 LOC** |
| Frontend (JavaScript / JSX) | **53,618 LOC** |
| Total application code | **~81,000 LOC** |
| Database models | **66** |
| REST endpoints | **174** |
| React components | **178** |

---

## The two experiences

The platform presents two entirely separate interfaces from one codebase, chosen at the router
based on the authenticated identity.

### Administrator console

A dense operations console for HR and payroll staff — employee master, payroll runs, Form 16
generation, attendance and leave administration, and reporting.

![Admin dashboard](screenshots/dashboard.png)

### Employee self-service portal

A focused personal portal: attendance, leave balance and requests, payslips, tax declarations,
Form 16 downloads and a tax calculator. CTC is masked behind a reveal toggle by default.

![Employee dashboard](screenshots/emp-home.png)

---

## Entry points

The public landing page routes visitors to whichever of the two logins applies.

![Landing page](screenshots/landing.png)

Administrators authenticate with a username and password. Employees authenticate **passwordlessly
via a 6-digit OTP** delivered to their registered mobile or email — no employee password exists to
be leaked, reset or reused.

| Employee login (OTP) | Admin login (password) |
|---|---|
| ![Employee login](screenshots/login-employee.png) | ![Admin login](screenshots/login-admin.png) |

---

## Why it is built this way

**One record, many readers.** Payroll, attendance, tax and Form 16 all read the same `Employee` and
`SalaryMaster` rows. There is no export/import step between modules, so the classic failure mode —
a mid-year salary revision reflected in payroll but not in Form 16 — cannot occur.

**Statutory logic lives in one place.** Slab rates, deduction caps, surcharge and cess are declared
as named constants in a single service module. A Union Budget change is a constant edit, not a hunt
through view code. The same engine drives the employee-facing calculator, the payroll TDS projection
and the Form 16 Part B — so all three agree by construction.

**Secure by default.** Authentication middleware rejects every route unless explicitly whitelisted,
so a newly added endpoint is protected before its author thinks about it. Authorisation is a separate
layer that scopes each query to what the caller is allowed to see.

**Bulk-first.** Every high-volume operation — salary updates, attendance import, payslip generation,
payslip email, Form 16 generation — has a bulk path with an Excel template, because HR teams work in
batches at month-end, not one employee at a time.

**Built for the Indian statutory context.** The 26→25 attendance cycle, LOP handling, PF/ESI/PT,
old-vs-new regime election, Chapter VI-A caps, quarterly TDS and Form 16 Part A/B are first-class
concepts, not adaptations of a foreign payroll model.

---

## Documentation scope

This archive is intended to communicate **what the system does and how it works**. It is not a
deployment guide and deliberately omits:

- application source code
- environment configuration, connection strings and API keys
- login credentials of any kind

All data visible in screenshots is **synthetic demo data** generated for documentation purposes.
No real employee, salary or tax record appears anywhere in this repository.
