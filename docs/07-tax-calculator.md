# Income Tax Calculator

[← Back to index](../README.md)

A full Indian income-tax engine comparing the old and new regimes. It is not a standalone widget —
it is the same computation the platform uses to project monthly TDS and to build Form 16 Part B.

---

## 1. Why one shared engine

Three places in an HR system must answer "how much tax?":

1. the employee asking which regime to elect
2. payroll deciding how much TDS to deduct this month
3. Form 16 reporting the year's computation to the tax department

If these are implemented separately they will diverge, and the divergence surfaces as an employee
whose Form 16 disagrees with what was deducted. Here all three call `calculate_tax()`. They agree
because they are the same code, not because someone kept them in sync.

`tax_calculator.py` is written as **pure functions over an input dictionary** with no database
writes, which is what makes it safely reusable in all three contexts.

---

## 2. Interface

![Employee tax calculator](../screenshots/emp-tax-calculator.png)

A four-step wizard with a live slab reference alongside.

| Step | Captures |
|---|---|
| **Basic Info** | Age, metro / non-metro, assessment year |
| **Income** | Salary components, house property, other income |
| **Deduction** | Chapter VI-A investments and expenses |
| **Summary** | Side-by-side regime comparison and recommendation |

Employee salary is **pre-filled from the salary master** (`get_employee_autofill()`), so the first
result shown is about the employee's real position. *Clear* empties the form for what-if modelling.

Supported assessment years: **AY 2027-28** and **AY 2026-27**.

![Admin tax calculator](../screenshots/tax-calculator.png)

HR has the same tool for advisory conversations and increment modelling.

---

## 3. Slab rates

### New regime

| Income | Rate |
|---|---|
| ₹0 – ₹4,00,000 | 0% |
| ₹4,00,001 – ₹8,00,000 | 5% |
| ₹8,00,001 – ₹12,00,000 | 10% |
| ₹12,00,001 – ₹16,00,000 | 15% |
| ₹16,00,001 – ₹20,00,000 | 20% |
| ₹20,00,001 – ₹24,00,000 | 25% |
| Above ₹24,00,000 | 30% |

Standard deduction **₹75,000** · rebate threshold **₹7,00,000**

### Old regime

| Income | Rate |
|---|---|
| ₹0 – ₹2,50,000 | 0% |
| ₹2,50,001 – ₹5,00,000 | 5% |
| ₹5,00,001 – ₹10,00,000 | 20% |
| Above ₹10,00,000 | 30% |

Standard deduction **₹50,000** · rebate threshold **₹5,00,000**

Health & education cess of **4%** applies to both.

Slabs are declared as module-level constants (`NEW_REGIME_SLABS`, `OLD_REGIME_SLABS`,
`*_REBATE_LIMIT`, `*_STANDARD_DEDUCTION`). A Budget change is an edit to these constants — it does
not require touching computation logic, and it propagates to the calculator, payroll TDS and
Form 16 simultaneously.

---

## 4. Deduction caps

| Section | Cap |
|---|---|
| 80C | ₹1,50,000 |
| 80CCD(1B) | ₹50,000 |
| 80D self | ₹25,000 |
| 80D parents | ₹25,000 |
| 80TTA | ₹10,000 |
| 24(b) home loan interest | ₹2,00,000 |
| House property loss set-off | ₹2,00,000 |

Chapter VI-A deductions apply under the **old regime**. The new regime trades them for lower slab
rates and a higher standard deduction — which is exactly the trade-off the comparison exists to
quantify.

---

## 5. Computation

```
Gross salary
  − Exemptions (HRA, LTA)              [old regime]
  − Standard deduction                 (₹50,000 old · ₹75,000 new)
─────────────────────────────────────
= Income from salary
  ± House property
      rental income
      − 30% standard deduction on rent
      − home loan interest (capped ₹2,00,000)
      → net, loss capped at ₹2,00,000
  + Other income
─────────────────────────────────────
= Gross total income
  − Chapter VI-A                       [old regime, each capped]
─────────────────────────────────────
= Total taxable income
  → slab tax
  − rebate 87A                         (if within threshold)
  + surcharge                          (if applicable)
  + cess @ 4%
─────────────────────────────────────
= Total tax payable
```

Run for both regimes, then compared.

### HRA exemption

Least of:
1. Actual HRA received
2. Rent paid − 10% of basic + DA
3. 50% of basic + DA (metro) or 40% (non-metro)

Metro status is captured in Basic Info because it changes the third limb.

### House property

Net income from house property is computed as rental income, less a 30% statutory standard
deduction on rent, less home loan interest — with the resulting loss capped at ₹2,00,000 for
set-off against salary.

### Precision

All arithmetic uses Python's `Decimal` with `ROUND_HALF_UP`, not floating point. Binary floats
cannot represent decimal currency exactly, and accumulated error in a tax computation produces
figures that fail statutory reconciliation.

---

## 6. Output

The summary returns, for each regime:

- taxable income
- tax before cess
- surcharge
- cess
- total tax payable
- **recommended regime** — whichever produces lower liability

**Use case.** An employee earning ₹14,00,000 with ₹1,50,000 of 80C investment and ₹25,000 of health
insurance runs the comparison. The old regime allows ₹1,75,000 of deductions plus HRA exemption; the
new regime offers lower rates and a ₹75,000 standard deduction. The calculator computes both and
names the cheaper one — rather than the employee guessing, or HR modelling it by hand in a
spreadsheet.

---

## Advantages

| Capability | Value |
|---|---|
| One engine, three consumers | Calculator, payroll TDS and Form 16 cannot disagree |
| Pure functions | Safe to reuse; no side effects on a what-if run |
| Constants in one place | A Budget change is a single, low-risk edit |
| Salary pre-fill | Answers the employee's real question immediately |
| Both regimes, side by side | Makes an irreversible annual election an informed one |
| Multi-AY support | Current and prior assessment year both available |
| Decimal arithmetic | Statutory-grade precision |
| Self-service | Removes a recurring advisory load from HR |
