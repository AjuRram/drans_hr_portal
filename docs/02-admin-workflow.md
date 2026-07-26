# Administrator / HR Workflow

[← Back to index](../README.md)

The administrator console is the operations surface for HR and payroll staff. This document walks
through it in the order a real HR team uses it: daily monitoring, then the monthly payroll cycle,
then the annual statutory cycle.

---

## Navigation model

The sidebar groups the console into five sections:

| Group | Contains |
|---|---|
| **Management** | Dashboard, Employees, Calculator, Payroll, Form 16 |
| **Recruitment** | Interview pipeline |
| **People** | Projects, Contacts, Attendance |
| **Preferences** | Global Settings (master data, roles, org structure) |

---

## 1. Dashboard — the daily landing view

![Admin dashboard](../screenshots/dashboard.png)

The dashboard answers "what needs my attention today?" before any navigation:

- **Headcount tiles** — total employees, present today, resumes in the pipeline
- **Birthdays & work anniversaries** — with a period selector; these feed the automated
  greeting engine so HR can see what will be sent
- **Public holiday calendar** — the statutory calendar payroll and attendance both work against
- **Support inbox widget** — employee helpdesk tickets awaiting a reply

**Use case.** An HR executive opens the console at 9am. Without clicking anything they can see
headcount is 6, nobody has a birthday today, and the next public holiday is Independence Day.
Anything needing action is surfaced; anything routine stays quiet.

---

## 2. Employee master

![Employee directory](../screenshots/employees.png)

The searchable directory of every active employee — employee number, name, role, email,
department/location, contact and joining date, with inline edit and resignation actions.

**Capabilities**
- Server-side search and filtering across the directory
- Excel export of the filtered set
- Pagination for large headcounts
- Row-level edit; resignation initiates the exit workflow rather than deleting the record

### Creating an employee

![Create employee](../screenshots/employee-create.png)

Employee creation is a **structured multi-section form**, not a flat page — personal information,
employment details, organisational placement, salary structure and statutory identifiers each have
their own section. This matters because an employee record with a missing PAN or a missing PF number
is unusable at Form 16 time, months later. Capturing it correctly at creation is far cheaper than
reconciling it in March.

### Employees without salary structure

A dedicated view lists employees whose salary structure has not yet been defined. These are exactly
the records that would silently produce a zero or broken payslip, so they are surfaced as a
worklist to be cleared before the payroll run.

### Resigned employees

![Resigned employees](../screenshots/resigned-employees.png)

Resigned employees are **retained, not deleted**. Their record remains for statutory history —
they still need a Form 16 for the year in which they worked — but a middleware gate refuses to mint
or honour login tokens for them. A reactivation path exists for rehires.

---

## 3. Attendance administration

![Attendance](../screenshots/attendance-present.png)

Attendance runs on a **26→25 cycle**: the payroll month runs from the 26th of one calendar month to
the 25th of the next. This is a common Indian payroll convention that gives the payroll team a
processing window between cycle close and pay date.

**Workflow**
1. Download the attendance Excel template
2. Export raw punch data from the biometric device
3. Upload it — the system parses and maps to employee numbers
4. Review the daily status grid and correct exceptions
5. Upload LOP (loss of pay) adjustments where applicable
6. Attendance feeds the payroll run automatically

**Views available:** daily status, monthly grid, per-employee calendar, cycle summary, and a
year-summary per employee. Every view has an Excel export.

**Use case.** On the 26th, the payroll executive uploads last cycle's biometric export. The grid
flags three employees with unmatched punches; they correct those manually. The cycle summary now
shows correct present/absent/half-day counts, and payroll can proceed knowing LOP is accurate.

---

## 4. Leave administration

| Leave requests | Leave tracker |
|---|---|
| ![Leaves](../screenshots/leaves.png) | ![Leave tracker](../screenshots/leave-tracker.png) |

Administrators see all pending requests across the organisation, approve or reject with a reason,
and maintain balances.

**Team leaves** gives reporting managers a scoped view of only their own team:

![Team leaves](../screenshots/team-leaves.png)

**Bulk operations:** leave balances can be imported from Excel and exported as a summary report —
necessary at year-start when opening balances are set for the whole company at once.

---

## 5. The monthly payroll cycle

This is the core operational loop of the platform.

### 5.1 Salary management

![Payroll](../screenshots/payroll.png)

The salary master holds each employee's CTC structure — earnings components, statutory deductions
and employer contributions. Changes are versioned through `CTCRevision`, so a mid-year increment is
recorded with its effective date rather than overwriting history. This is what allows Form 16 to
correctly reflect a salary that changed in, say, October.

### 5.2 Bulk salary update

![Bulk salary update](../screenshots/payroll-bulk-salary.png)

Annual increments affect everyone at once. The bulk path is: download a pre-filled template →
edit in Excel → upload → validate → apply. Filter options let HR scope the template to a branch or
department.

### 5.3 Running payroll

![Payroll process](../screenshots/payroll-process.png)

The payroll run is a **six-stage pipeline**, each stage independently triggerable:

| Stage | Action |
|---|---|
| 1 | **Calculate** — compute gross, deductions and net for the period |
| 2 | **Generate payslips** — create payslip records |
| 3 | **Generate PDFs** — render documents (queued asynchronously) |
| 4 | **Review** — inspect before anything reaches an employee |
| 5 | **Publish** — make payslips visible in the employee portal |
| 6 | **Email** — bulk-deliver payslips to registered addresses |

Separating *generate* from *publish* is deliberate. Payroll errors are discovered by looking at
output, and output has to exist before it can be reviewed. Because publication is a distinct step,
a mistake found at stage 4 costs a regeneration — not an apology to the whole company.

### 5.4 Payslips and logs

| Payslip view | Payslip logs |
|---|---|
| ![Payslip](../screenshots/payroll-payslip.png) | ![Payslip logs](../screenshots/payroll-logs.png) |

The log records every generation and delivery event, which answers the recurring month-end question
"was this person's payslip actually sent, and when?"

### 5.5 Detailed salary report

![Detailed salary report](../screenshots/payroll-salary-report.png)

A full analytical breakdown across the workforce — component-level, exportable to Excel, used for
finance reconciliation and audit.

---

## 6. The annual statutory cycle — Form 16

Full detail in [the Form 16 module doc](05-form16.md); this is the administrator's path through it.

### 6.1 Employer setup

![Employer setup](../screenshots/form16-employer.png)

TAN, PAN, employer name and address — the deductor identity printed on every certificate.
Configured once per financial year.

### 6.2 Reviewing employee declarations

![Declarations review](../screenshots/form16-declarations.png)

Employees submit investment declarations with uploaded proofs. HR reviews each, downloads the proof
document, and approves or rejects with a reason. **Only approved declarations reduce taxable income**
— an unverified claim never silently lowers someone's tax and creates a liability at assessment.

### 6.3 Generating certificates

![Form 16 list](../screenshots/form16-list.png)

Filter by financial year, status, branch or employee number, then generate — individually or in bulk
across the whole company.

Each certificate has:
- **Part A** — employer/employee identity, employment period, quarterly TDS deducted and deposited
- **Part B** — the salary and tax computation annexure

Supporting the annexure are **TDS deposits** and **challans**, recorded so that what the certificate
claims was deposited can be traced to an actual government challan.

### 6.4 Reconciliation

Before publishing, a reconciliation view diffs computed tax against actual TDS deducted. A mismatch
is exactly the condition that produces an income-tax notice, so it is surfaced as a blocking review
step rather than discovered by the employee later.

### 6.5 Publish, revoke, rectify

![Rectifications](../screenshots/form16-rectifications.png)

Certificates are published to the employee portal, and can be **revoked** if an error is found after
release. Employees can raise a **rectification request** against a published certificate, which
lands in this admin queue for correction and reissue.

---

## 7. Tax calculator (admin view)

![Tax calculator](../screenshots/tax-calculator.png)

The same engine employees use, available to HR for advisory conversations — modelling an employee's
liability under both regimes when they ask which to elect, or projecting the effect of a proposed
increment.

---

## 8. Recruitment

| Scheduled candidates | Candidate pipeline |
|---|---|
| ![Scheduled](../screenshots/recruitment-scheduled.png) | ![Total candidates](../screenshots/recruitment-total.png) |

The pipeline tracks candidates through stages — total → scheduled → in progress → selected /
rejected — with interview scheduling, automated invitation and round emails, and Microsoft Teams
meeting creation. A selected candidate converts directly into the onboarding flow, carrying their
data forward instead of being re-typed.

---

## 9. Onboarding

![Onboarding](../screenshots/onboarding.png)

See [Recruitment & Onboarding](09-recruitment-onboarding.md). HR sends a secure link; the new hire
completes their joining form, e-signs, and uploads KYC documents before their first day — without
needing an account, because they do not have one yet.

---

## 10. Master data & organisation

![Master data](../screenshots/master-data.png)

Central maintenance of branches, locations, reporting managers, team heads, HR contacts, assets and
financial years. These are the reference entities every other module points at, so they are
maintained in one place rather than typed as free text per employee.

---

## 11. Role management

![Role management](../screenshots/role-management.png)

HR staff accounts are created here with **module-level permissions** and **branch scoping**. A
restricted HR user assigned to one branch sees only that branch's employees — enforced in the query
layer on the server, not by hiding menu items. See [Security & RBAC](10-security-rbac.md).

---

## 12. WhatsApp Business inbox

![WhatsApp dashboard](../screenshots/whatsapp.png)

A two-way WhatsApp Business Cloud API integration with a realtime inbox over WebSockets, message
templates and campaigns. It carries the OTP delivery for employee login and is used for HR
broadcasts — reaching a workforce that reads WhatsApp far more reliably than email.

---

## Annual calendar at a glance

| When | What HR does |
|---|---|
| Daily | Dashboard check, approve leave, answer support tickets |
| 26th | Close attendance cycle, upload biometric data, apply LOP |
| Month-end | Calculate → generate → review → publish → email payslips |
| Quarterly | Record TDS deposits and challans |
| Jan–Mar | Collect and review employee investment declarations |
| Apr–Jun | Generate, reconcile and publish Form 16 for the closed year |
| Ongoing | Onboarding, recruitment, master data, increments |
