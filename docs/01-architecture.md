# Architecture & Tech Stack

[← Back to index](../README.md)

---

## 1. High-level shape

ApexHR is a **decoupled two-tier application**: a Django REST API and a React single-page
application that talks to it over JSON. There is no server-rendered HTML in the product surface —
the API serves data, the SPA owns all presentation and routing.

```
┌─────────────────────────────────────────────────────────────────┐
│  React 18 SPA  (Material-UI, react-router, Axios)               │
│                                                                 │
│   ┌───────────────────────┐      ┌──────────────────────────┐  │
│   │  Administrator console│      │ Employee self-service    │  │
│   │  PrivateRoutes.js     │      │ Emproutes.js             │  │
│   └───────────┬───────────┘      └────────────┬─────────────┘  │
│               └───────── AuthContext ─────────┘                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │  JSON + Bearer JWT
┌───────────────────────────────┴─────────────────────────────────┐
│  Django 4.2 + Django REST Framework                             │
│                                                                 │
│   JWTAuthMiddleware       → authentication (who are you?)       │
│   ResignedEmployeeGate    → lifecycle kill-switch               │
│   permissions.py          → authorisation (what may you see?)   │
│                                                                 │
│   candidates/views.py     → 174 endpoints                       │
│   candidates/services/    → payroll, tax and Form 16 engines    │
│   candidates/models.py    → 66 models                           │
└──────┬───────────────┬───────────────┬──────────────┬───────────┘
       │               │               │              │
   PostgreSQL      Celery +        AWS S3         WhatsApp
   (relational     Celery Beat     (documents,    Cloud API
    system of      + Redis         payslips,      (OTP + inbox)
    record)        (async jobs)    Form 16 PDFs)
```

---

## 2. Technology choices

### Backend

| Layer | Technology | Why |
|---|---|---|
| Framework | Django 4.2 | Mature ORM and migrations for a 66-model relational domain |
| API | Django REST Framework | Serialisation, viewsets, consistent error contracts |
| Async jobs | Celery + Celery Beat + Redis | Payslip PDF generation, bulk email and scheduled jobs must not block requests |
| Realtime | Daphne / ASGI + Django Channels | WebSocket inbox for the WhatsApp Business integration |
| Object storage | AWS S3 via `django-storages` | Payslips, Form 16 PDFs, KYC and employee documents |
| Auth | PyJWT (HS256) | Stateless tokens shared by both frontends |
| Monitoring | New Relic APM | Production tracing |

### Frontend

| Layer | Technology |
|---|---|
| Framework | React 18 |
| Design system | Material-UI (MUI v5) with a customised theme |
| Routing | react-router v6 |
| HTTP | Axios with interceptors that attach the bearer token |
| Documents | Client-side PDF rendering for payslips and certificates |
| Spreadsheets | Excel import/export throughout the bulk workflows |

### Delivery

Docker image → AWS ECR → Jenkins pipeline (build → push → deploy → migrate → health-check).

---

## 3. The data model

66 models organised into eight clusters. The `Employee` model is the hub — nearly every other
table is keyed to it.

### Identity and organisation
`Employee` · `Admin` · `HRContact` · `Branch` · `Location` · `ReportingManager` · `TeamHead` ·
`EmployeeResignation` · `Asset`

### Recruitment and onboarding
`Candidate` · `InterviewApplication` · `ProgressStatus` · `OnboardingDocument` ·
`OnboardingKYCItem` · `OnboardingReminderLog` · `EmployeeDocument`

### Salary structure
`SalaryMaster` · `CTCItems` · `CTCRevision` · `GrossPay` · `NetPay` · `TotalDeduction` ·
`PerformanceIncentive` · `ReimbursementItems` · `OtherPaymentItems` · `MiscellaneousItems`

### Statutory components
`PFRelatedItems` · `ESIRelatedItems` · `ProfessionalTaxItems` · `TamilNaduPT` ·
`IncomeTaxRelatedItems` · `PerquisiteItems` · `ExemptionItems` · `StatutoryItems` ·
`SettlementRelatedItems` · `ProjectionItems`

### Payroll output
`Payslip` · `EmployeePaySlip` · `Tds`

### Time and attendance
`EmployeeAttendance` · `EmployeeWorkdays` · `LeaveRequest` · `LeaveRequestCC` · `LopUpload`

### Tax and Form 16
`FinancialYear` · `Employer` · `Form16` · `Form16PartA` · `Form16PartB` · `Form16SalaryEntry` ·
`TaxDeclaration` · `TaxDeclarationProof` · `TDSDeposit` · `TDSChallan` · `Rectification`

### Engagement and support
`WhatsAppContact` · `WhatsAppMessage` · `WhatsAppTemplate` · `WhatsAppCampaign` ·
`WhatsAppWebhookLog` · `WhatsAppBusinessProfile` · `SupportTicket` · `SupportMessage` ·
`BirthdayWishLog` · `WorkAnniversaryLog`

### Why the salary structure is split across many tables

A single wide `salary` table cannot express Indian payroll. PF has its own employer/employee split
and wage ceiling; ESI applies only below a threshold; Professional Tax is state-specific (hence a
dedicated `TamilNaduPT`); perquisites and exemptions are taxed differently from earnings. Splitting
these into typed component tables means each carries its own rules and history, and the Form 16
engine can aggregate exactly the categories the statutory form requires.

---

## 4. Request lifecycle

Every API request passes through the same chain:

```
Request
  │
  ├─ 1. CORS            → origin on the allow-list?
  │
  ├─ 2. JWTAuthMiddleware
  │       • whitelisted public path? → continue unauthenticated
  │       • else require Bearer token, verify HS256 signature + expiry
  │       • attach decoded identity payload to the request
  │       • on failure → 401, request never reaches a view
  │
  ├─ 3. ResignedEmployeeGateMiddleware
  │       • token valid but employee resigned/inactive → 403
  │
  ├─ 4. View + permission decorator
  │       • require_admin / can_access_employee_record
  │       • querysets scoped by branch for restricted HR users
  │
  └─ Response
```

The critical property is that **step 2 denies by default**. A newly added endpoint is authenticated
without its author doing anything; forgetting a decorator cannot silently expose data to anonymous
callers. Public routes (login, OTP, onboarding token links) are explicit exceptions on a whitelist.

---

## 5. The service layer

Business logic that is too heavy or too consequential for a view lives in `candidates/services/`.

| Service | Responsibility |
|---|---|
| `form16.py` | Part A/B snapshots, quarterly TDS aggregation, regime computation, reconciliation |
| `tax_calculator.py` | Pure old-vs-new regime engine — no database writes |
| `salary_breakdown.py` | CTC structure rows and month-paid rows for reporting |

`tax_calculator.py` is deliberately **pure functions over a dictionary of inputs**. It has no
database dependency in its computation path, which means the same code answers "what would my tax
be?" for an employee's what-if scenario and "what should we deduct?" for the payroll run. Two
answers that must agree are produced by one function.

---

## 6. Asynchronous work

Operations that are slow or bulk-natured are queued rather than run inline:

- payslip PDF generation for an entire branch
- bulk payslip email delivery
- Form 16 bulk generation across a financial year
- scheduled birthday and work-anniversary messages (Celery Beat)
- WhatsApp webhook ingestion

This keeps the request cycle responsive during month-end, when an HR user may trigger work
affecting hundreds of employees in a single click.

---

## 7. Storage strategy

| Data | Where | Why |
|---|---|---|
| Relational records | PostgreSQL | Transactional integrity across payroll writes |
| Generated PDFs | S3 | Immutable artefacts, served via signed URLs |
| Uploaded documents | S3 | KYC, proofs, employee files |
| OTPs, transient state | Redis cache | Short TTL, single-use, deleted on verification |

Generated payslips and Form 16 PDFs are treated as **immutable artefacts**. Regenerating produces a
new object rather than mutating the old one, so a published certificate cannot silently change
underneath an employee who has already downloaded it.
