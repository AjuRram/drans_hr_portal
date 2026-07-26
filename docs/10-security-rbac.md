# Security & Access Control

[← Back to index](../README.md)

An HR system holds the most sensitive data a company has about its people: salary, bank details,
PAN, Aadhaar, medical declarations, performance and exit records. Access control is a primary
design concern, not a feature.

---

## 1. Two separated concerns

The platform separates **authentication** from **authorisation** into different layers:

| | Question | Where |
|---|---|---|
| **Authentication** | Who is this caller? | `utils/middleware.py` |
| **Authorisation** | What is this caller allowed to see or do? | `utils/permissions.py` |

Collapsing these — the common shortcut — produces views that check identity and permission in the
same conditional, and are therefore easy to get subtly wrong. Keeping them separate means every
request is *identified* by infrastructure and *scoped* by explicit policy.

---

## 2. Authentication

`JWTAuthMiddleware` runs on **every request**:

1. Is the path on the public whitelist? (login, OTP, onboarding token links) → continue
2. Otherwise require an `Authorization: Bearer` token
3. Verify the HS256 signature against the server secret
4. Verify expiry
5. Attach the decoded identity payload to the request
6. On any failure → **401, before the view is reached**

### Secure by default

The significant property is the default. A newly added endpoint is authenticated **without its
author doing anything**. Forgetting a decorator cannot expose data anonymously — the failure mode
of the common alternative, where protection is opt-in per view.

Public access is an explicit, reviewable whitelist entry rather than an omission.

### Token properties

| Property | Value |
|---|---|
| Algorithm | HS256 |
| Lifetime | 24 hours |
| Claims | identity, employee/admin id, issued-at, expiry, token type |
| Secret | Environment variable, required at startup |

The signing secret is mandatory configuration. If it were absent and the application fell back to a
random per-process value, tokens would silently stop working across restarts and behind multiple
workers — so its absence is a startup failure rather than an intermittent production bug.

---

## 3. Lifecycle gate

`ResignedEmployeeGateMiddleware` runs after authentication and refuses requests from employees who
have resigned or been deactivated — **even with a structurally valid, unexpired token**.

Without this, an employee who left the day after receiving a 24-hour token would retain access
until it expired. The gate closes that window at the moment of deactivation.

The same check is repeated at OTP verification, so a valid in-flight OTP cannot mint a token for
someone who was deactivated between request and verification.

---

## 4. Authorisation — three tiers

```
┌──────────────────────────────────────────────────────────┐
│ Global administrator                                     │
│   Full access across all branches and all modules        │
├──────────────────────────────────────────────────────────┤
│ HR user (branch-scoped, module-permissioned)             │
│   Only assigned branches                                 │
│   Only permitted modules, at view or edit level          │
├──────────────────────────────────────────────────────────┤
│ Employee                                                 │
│   Own records only                                       │
│   Team records if a reporting manager                    │
└──────────────────────────────────────────────────────────┘
```

Implemented by named predicates:

| Function | Purpose |
|---|---|
| `identity_is_global_admin()` | Unrestricted administrator |
| `identity_is_admin_or_hr()` | Either administrator tier |
| `hr_contact_for_payload()` | Resolve the HR user behind a token |
| `hr_contact_branches()` | Branches that HR user may access |
| `scope_employees_for_identity()` | **Filter a queryset to the permitted set** |
| `identity_is_employee()` | Is this token the employee in question? |
| `can_access_employee_record()` | Combined check for a specific record |
| `require_admin()` | Decorator enforcing administrator access |

### Query-level scoping

`scope_employees_for_identity()` is the important one. Restriction is applied by **filtering the
queryset**, not by hiding UI elements.

A branch-restricted HR user who calls the employee list API directly, or edits the request in
developer tools, still receives only their branch's employees — because the database query never
selected the others. UI-level hiding would be cosmetic; query-level scoping is enforcement.

---

## 5. Frontend access control

![Role management](../screenshots/role-management.png)

HR accounts are configured with module permissions and branch assignment. The React router wraps
protected routes in a `ProtectedRoute` component taking a `module` and an `action` (`view` /
`edit`), so navigation reflects entitlement.

This is a **usability layer, not the security boundary**. The server enforces the same rules
independently; the frontend simply avoids showing a user actions that would be rejected.

---

## 6. Employee authentication

Employees never have a password.

| Risk with passwords | Status here |
|---|---|
| Reused from a breached site | No password exists |
| Phishable and storable | Nothing to store |
| Shared between colleagues | OTP goes to a personal device |
| Reset requests burden HR | No reset flow |
| Leak in a database breach | No credential to leak |

OTPs are 6-digit, single-use, short-TTL, held in Redis, and deleted on verification.

---

## 7. Data protection

| Concern | Handling |
|---|---|
| Passwords (admin/HR) | Django PBKDF2 hashing, with transparent upgrade of any legacy plaintext on first successful login |
| Documents | S3 with access-controlled URLs |
| Salary in the UI | CTC masked behind a reveal toggle by default |
| Sessions | 30-minute inactivity auto-logout, coordinated across tabs |
| CORS | Explicit origin allow-list |
| CSRF | Trusted-origin list for cookie-bearing requests |
| Payslip visibility | Employees see only *published* payslips |
| Form 16 visibility | Employees see only *published* certificates |

---

## 8. Audit trails

| Recorded | Model |
|---|---|
| Payslip generation and delivery | `EmployeePaySlip`, payslip logs |
| Form 16 rectification requests | `Rectification` |
| TDS deposits and challans | `TDSDeposit`, `TDSChallan` |
| Salary revision history | `CTCRevision` |
| LOP uploads | `LopUpload` |
| Onboarding reminders | `OnboardingReminderLog` |
| WhatsApp webhooks | `WhatsAppWebhookLog` |
| Automated greetings | `BirthdayWishLog`, `WorkAnniversaryLog` |

Salary and statutory changes are **appended, not overwritten**, so the state of a record at any
past date can be reconstructed — which is the requirement during a payroll audit or a tax
assessment.

---

## Advantages

| Property | Value |
|---|---|
| Deny-by-default authentication | New endpoints are protected on day one |
| Auth and authz separated | Each layer stays simple and reviewable |
| Query-level scoping | Restrictions cannot be bypassed via the API |
| Lifecycle gate | Departed employees lose access immediately |
| Passwordless employees | An entire credential-attack class removed |
| Publication gates | Draft financial documents are never visible |
| Append-only history | Auditable reconstruction of any past state |
| Mandatory signing secret | Misconfiguration fails loudly at startup |
