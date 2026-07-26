# Leave Management

[← Back to index](../README.md)

---

## 1. The approval chain

```
Employee applies
      │
      ▼
Reporting manager  ──reject (with reason)──► employee notified, balance untouched
      │
   approve
      │
      ▼
Balance debited ──► attendance status written ──► payroll reflects paid / unpaid
      │
      ▼
CC recipients notified (informational, not approvers)
```

The **CC mechanism** (`LeaveRequestCC`) matters more than it first appears. In practice a leave
request needs to be *known* by more people than need to *approve* it — a project lead, a shift
coordinator, a client-facing colleague. Without CC, teams either add unnecessary approvers (slowing
everything down) or communicate out-of-band over chat (losing the record). Separating notification
from authority keeps one approver and still informs everyone.

---

## 2. Employee experience

![Employee leaves](../screenshots/emp-leaves.png)

**Apply:** leave type, date range, reason, optional CC recipients.

**Track:** current status of every request, with rejection reasons visible.

**Withdraw:** a pending request can be withdrawn by the employee — no need to ask a manager to
reject it.

**Balance:** sick and casual leave balances are shown on the home screen next to the apply action,
so the decision and the information sit together.

![Leave balance on home](../screenshots/emp-home.png)

---

## 3. Manager experience

![Team leaves](../screenshots/team-leaves.png)

Reporting managers see requests from their own team only — scoped server-side. They approve or
reject with a reason, and see team leave patterns without any access to the admin console.

---

## 4. Administrator experience

| All requests | Leave tracker |
|---|---|
| ![Leaves](../screenshots/leaves.png) | ![Leave tracker](../screenshots/leave-tracker.png) |

HR sees organisation-wide requests, can act on any of them, and maintains balances.

### Bulk balance management

Opening balances are set for the whole company at the start of a leave year. Doing that
one-employee-at-a-time is impractical, so the module provides:

- an Excel import template for leave balances
- bulk update across employees
- per-employee update where needed
- a leave summary report, exportable

---

## 5. Entitlement

Leave entitlement is derived from **employment status** — the model assigns default balances when
an employee is created, based on whether they are permanent, probationary, contract or intern,
unless balances are explicitly set. New employees therefore start with correct entitlement without
HR configuring each record by hand.

---

## 6. Integration

| Module | Relationship |
|---|---|
| **Attendance** | Approved leave writes the corresponding attendance status |
| **Payroll** | Unpaid leave becomes LOP and pro-rates gross pay |
| **Employee portal** | Balance and history surfaced on the home screen |
| **Notifications** | Approver and CC recipients notified on submission and decision |

---

## Advantages

| Capability | Value |
|---|---|
| Single approver + CC | Fast decisions, full visibility, complete record |
| Self-service withdrawal | Removes a trivial task from managers |
| Visible rejection reasons | No follow-up email to ask why |
| Automatic attendance write-through | Leave and attendance cannot disagree |
| Direct payroll effect | Unpaid leave becomes LOP without manual intervention |
| Bulk balance import | A leave-year rollover is one upload |
| Status-derived entitlement | Correct balances from day one |
| Scoped manager views | Delegation without privilege escalation |
