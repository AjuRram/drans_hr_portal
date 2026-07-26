# Payroll Module

[← Back to index](../README.md)

Payroll is the highest-stakes module in the platform. A bug here is not a cosmetic defect — it is
someone's rent paid late, or a statutory filing that is wrong.

---

## 1. The salary structure

An employee's compensation is not one number. It is a structure decomposed across typed component
tables, because each category behaves differently for tax, for statutory contribution, and for the
Form 16 annexure.

```
CTC (annual cost to company)
│
├── Earnings ─────────────────────────────────────────
│     Basic · HRA · Special Allowance · Conveyance
│     Medical · Food Allowance · Other Allowances
│
├── Employer contributions ────────────────────────────
│     Employer PF · Employer ESI · Gratuity
│
├── Employee deductions ───────────────────────────────
│     Employee PF · Employee ESI · Professional Tax
│     Income Tax (TDS) · Loans / Advances
│
├── Variable ──────────────────────────────────────────
│     Performance incentives · Reimbursements
│     Other payments · Miscellaneous
│
└── Statutory / tax classification ────────────────────
      Perquisites · Exemptions · Projections · Settlement
```

### Why the split

| Component | Why it needs its own handling |
|---|---|
| **PF** | Employer and employee halves are treated differently; a wage ceiling applies; employer PF is CTC but not taxable salary |
| **ESI** | Only applies below an earnings threshold; eligibility can change mid-year with an increment |
| **Professional Tax** | State-specific slabs — the platform carries a dedicated Tamil Nadu PT table |
| **Perquisites** | Taxed under distinct rules and reported separately on Form 16 |
| **Exemptions** | HRA, LTA and similar reduce taxable income rather than gross |
| **Reimbursements** | Paid but generally not taxable — must not inflate taxable salary |

A single flat salary table would force all of this into view-level conditionals. Typed component
tables let each carry its own rules, and let the Form 16 engine aggregate precisely the categories
the statutory form asks for.

---

## 2. Salary revisions

![Salary management](../screenshots/payroll.png)

Increments are recorded as **`CTCRevision` rows with effective dates**, not overwrites.

This is what makes mid-year changes correct everywhere. If someone is revised in October:

- payslips before October use the old structure
- payslips from October use the new one
- Form 16 for the year aggregates *both periods accurately*
- the tax projection reflects the blended annual figure

Had the structure simply been overwritten, the Form 16 would report twelve months at the new salary
— overstating income and misreporting TDS to the tax department.

---

## 3. Bulk salary update

![Bulk salary update](../screenshots/payroll-bulk-salary.png)

Appraisal cycles change many employees at once. The bulk path:

1. Choose filters (branch, department) to scope the set
2. Download a **pre-filled Excel template** containing current values
3. Edit new figures in Excel — the tool HR already uses for compensation planning
4. Upload
5. Server-side validation reports errors per row
6. Apply — each change written as a revision with its effective date

Pre-filling the template matters: HR edits known-good current data rather than authoring a
spreadsheet from scratch, which eliminates an entire class of column-mismatch and typo errors.

---

## 4. The payroll run

![Payroll process](../screenshots/payroll-process.png)

Six discrete stages. Each is triggered independently and can be re-run.

### Stage 1 — Calculate

For each employee in scope:

```
Gross pay        = Σ earnings components (pro-rated for LOP / partial month)
Statutory        = PF + ESI + Professional Tax  (per applicable rules)
Income tax       = monthly TDS from the annual projection,
                     net of approved declarations
Other deductions = loans, advances, recoveries
─────────────────────────────────────────────────
Net pay          = Gross − (Statutory + Income tax + Other)
```

Attendance is an input here: LOP days from the closed 26→25 cycle pro-rate the gross.

Available individually (`api/calculate-salary/`) or in bulk (`api/bulk-calculate-salary/`).

### Stage 2 — Generate payslips

Creates `Payslip` records with a full component breakdown. Records exist but are **not yet visible
to employees**.

### Stage 3 — Generate PDFs

Renders documents and stores them in S3. Queued asynchronously — generating hundreds of PDFs must
not hold an HTTP request open.

### Stage 4 — Review

![Payslip](../screenshots/payroll-payslip.png)

HR inspects generated payslips before release. Corrections at this point cost a regeneration.

### Stage 5 — Publish

Flips visibility. Only now do payslips appear in the employee portal.

### Stage 6 — Email

Bulk-delivers payslips to registered addresses, with delivery logged per employee.

### Why the stages are separate

The alternative — a single "Run Payroll" button that calculates, publishes and emails — has no
point at which a human can catch an error. Payroll mistakes are found by *looking at output*, and
output must exist before it can be examined. Splitting generation from publication creates exactly
that window. An error found at stage 4 is a non-event; the same error found after stage 6 is a
company-wide correction.

---

## 5. Payslip logs

![Payslip logs](../screenshots/payroll-logs.png)

Every generation and delivery event is recorded. This answers the standard month-end questions:
was it generated, was it published, was it emailed, when, and to which address.

---

## 6. Detailed salary report

![Detailed salary report](../screenshots/payroll-salary-report.png)

A component-level analytical view across the workforce, exportable to Excel for finance
reconciliation and audit. Built on `salary_breakdown.py`, which produces two row types:

- **structure rows** — what an employee's CTC is composed of
- **paid rows** — what was actually paid in a given month

Comparing the two is how a payroll team reconciles "what we agreed to pay" against "what we paid".

---

## 7. Employee-facing output

![Employee payslip](../screenshots/emp-payslip.png)

Employees see published payslips in their portal with full breakdown, downloadable as PDF, retained
permanently.

---

## 8. Integration with other modules

| Module | Relationship to payroll |
|---|---|
| **Attendance** | LOP days pro-rate gross pay |
| **Leave** | Unpaid leave becomes LOP |
| **Tax declarations** | Approved declarations reduce monthly TDS |
| **Tax calculator** | Same engine computes the TDS projection |
| **Form 16** | Aggregates the year's payslips into the statutory annexure |

Because these are foreign-key relationships inside one database rather than integrations between
systems, there is no synchronisation step and no window in which two modules disagree.

---

## Advantages

| Capability | Operational value |
|---|---|
| Staged pipeline with review gate | Errors are caught before employees see them |
| Effective-dated revisions | Mid-year increments stay correct in payslips *and* Form 16 |
| Bulk Excel workflows | An appraisal cycle is one upload, not hundreds of edits |
| Async PDF and email | Month-end work does not block the UI |
| Component-level modelling | Statutory rules are expressible without special-casing |
| Delivery logging | Every payslip is auditable end to end |
| Shared tax engine | Projected TDS and final Form 16 cannot drift apart |
