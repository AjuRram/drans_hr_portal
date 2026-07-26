# Attendance Module

[← Back to index](../README.md)

Attendance is the input that makes payroll correct. Loss of pay, food allowance eligibility and
leave reconciliation all derive from it.

---

## 1. The 26→25 cycle

The attendance period runs from the **26th of one month to the 25th of the next**, not the calendar
month.

```
      Jul 26 ─────────────────────────────► Aug 25      cycle closes
                                              │
                                              ▼
                                     Aug 26–31: payroll processing window
                                              │
                                              ▼
                                     Aug 31: salary credited
```

This is a widespread Indian payroll convention. A calendar-month cycle closing on the 31st with
salary due the same day leaves no time to import biometric data, correct exceptions, calculate,
review and disburse. The 26→25 cycle creates a working window between close and pay date.

The cycle is a first-class concept: exports, summaries, calendars and the employee portal all
default to cycle boundaries rather than calendar months, with a calendar-month toggle where useful.

![Employee attendance calendar](../screenshots/emp-home.png)

---

## 2. Status model

| Status | Meaning | Payroll effect |
|---|---|---|
| `PRESENT` | Full day worked | Full pay; qualifies for food allowance |
| `HALF_DAY` | Half day worked | Half-day pro-rating; qualifies for food allowance |
| `ON_DUTY` | Official work off-site | Full pay; qualifies for food allowance |
| `LEAVE` | Approved leave | Paid or unpaid per leave type |
| `ABSENT` | Unapproved absence | Loss of pay |
| `WEEKEND` / `HOLIDAY` | Non-working day | No effect |

Food allowance eligibility is evaluated by a dedicated rule
(`qualifies_for_food_allowance(in_time, status)`) taking both status *and* punch-in time — a
present-but-very-late day may not qualify. Encoding it as a rule rather than a manual adjustment
keeps the allowance consistent across the workforce.

---

## 3. Administrator workflow

![Attendance administration](../screenshots/attendance-present.png)

### Import from biometric devices

1. **Download the template** — the exact column layout the parser expects
2. **Export from the biometric device** — raw punch data
3. **Upload** — the system parses, maps to employee numbers, derives daily status
4. **Review** — the daily status grid surfaces unmatched or anomalous records
5. **Correct** — individual records are editable
6. **Close** — the cycle summary reflects final counts

Templated import rather than free-form upload is what keeps this reliable: the parser knows the
column contract, and mismatches fail loudly at upload instead of silently producing wrong LOP.

### LOP management

Loss of pay can be uploaded separately via its own template, tracked as `LopUpload` records with a
dedicated report. LOP sometimes arises from circumstances not visible in punch data — a suspension,
an unpaid sabbatical — so it needs a path independent of the biometric import.

### Views and exports

| View | Purpose |
|---|---|
| Daily status | Who is present today, organisation-wide |
| Monthly grid | Employees × days matrix for a period |
| Per-employee calendar | One employee's full cycle |
| Cycle summary | Aggregated counts per employee for the closed cycle |
| Year summary | Twelve-month view for one employee |

Every view exports to Excel.

---

## 4. Employee view

![Employee attendance](../screenshots/emp-attendance.png)

Employees see their own record — cycle, monthly and year summary — with a colour-coded legend.

The transparency has an operational payoff. Because employees see the same data payroll will
consume, a missing punch is typically reported by the employee within days, rather than surfacing
as an unexplained salary deduction weeks later. Reconciliation moves from after-the-fact dispute to
before-the-fact correction.

### Team attendance

![Team attendance](../screenshots/emp-team-attendance.png)

Reporting managers see their own team's attendance without access to the admin console.

---

## 5. Integration

| Consumer | What it uses attendance for |
|---|---|
| **Payroll** | LOP days pro-rate gross pay |
| **Leave** | Approved leave writes attendance status |
| **Food allowance** | Per-day eligibility by status and punch time |
| **Employee portal** | Cycle summary and calendar |
| **Reports** | Cycle and year summaries, Excel exports |

---

## Advantages

| Capability | Value |
|---|---|
| Native 26→25 cycle | Matches how Indian payroll actually runs; creates a processing window |
| Templated biometric import | Device-agnostic; errors fail at upload, not in payroll |
| Employee visibility | Discrepancies reported before they become pay errors |
| Separate LOP path | Handles adjustments punch data cannot express |
| Rule-based food allowance | Consistent treatment, no manual adjustment |
| Excel export everywhere | Fits existing HR and audit workflows |
| Manager team views | Delegated oversight without admin access |
