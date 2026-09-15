# Mobile Money & Airtime Kiosk — Modelling & Simulation Workbook

## Overview

This project simulates the day-to-day financial performance of a **mobile
money agent line and airtime/data kiosk** serving students and staff, over
a  **5-year horizon (1 Jan 2026 – 31 Dec 2030, 1,826 days)** .

The business idea, revenue streams, cost structure, and influence diagram
were taken directly from the assignment scenario:

* **Revenue streams:** commission per mobile money transaction + markup on
  airtime/data bundle sales
* **Costs:** float capital, agent line commission fees, kiosk rent,
  transport for topping up float
* **Influence diagram:** Float (Inputs) → Transactions Processed →
  Commission Earned → Revenue
* **Forecast:** ~500 mobile money transactions/month as the starting point

The model turns that scenario into a full  **day-by-day simulation dataset** ,
rolls it up into monthly and annual summaries, and calculates the
**break-even point** — the exact day the accumulated profit fully repays
the starting capital, after which the investor is earning clean profit.

Everything is driven by a single set of assumptions (growth rates,
commission rates, costs, etc.), so the whole simulation can be re-run under
different scenarios simply by changing those inputs.

---

## Files in this project

### 1_Assumptions.xlsx

The control panel for the entire model. Every number the simulation depends
on lives here, grouped into sections:

1. **Simulation period** — start date, end date, number of days
2. **Initial (startup) capital** — float capital, airtime/data stock,
   kiosk setup, rent deposit, equipment → summed into a
   **Total Initial Investment of UGX 5,000,000**
3. **Mobile money revenue drivers** — starting transactions/month (500, per
   the assignment forecast), monthly growth rates for Year 1 and Year 2+,
   and the equivalent *daily* compounding growth rates used by the Daily
   Ledger, plus average commission per transaction
4. **Airtime/data revenue drivers** — starting transactions/month, growth
   rate, and average markup profit per sale
5. **Day-of-week effect** — a weekend multiplier (60%) reflecting lighter
   footfall on Saturdays/Sundays, since the customer base is students/staff
6. **Monthly operating costs** — rent, float top-up transport, agent line
   charges, utilities, attendant wage (starts Month 13), miscellaneous
7. **Cost inflation** — a small monthly inflation rate applied to costs
   from Month 13 onward

Blue-on-yellow cells are the editable inputs; grey cells are calculated
from them. This file still contains live formulas, since every formula in
it only refers to other cells within the same sheet.

### 2_Daily_Ledger.xlsx

The heart of the simulation:  **one row per day for all 1,826 days** .
Each row shows:

* Date, simulation year, calendar month number, day of week
* Mobile money transactions (trend + actual, after the weekend effect) and
  the commission revenue they generate
* Airtime/data transactions (trend + actual) and the markup revenue they
  generate
* Total daily revenue
* Each cost line (rent, transport, agent charges, utilities, wage, misc) —
  booked as a lump sum on the  **1st of each calendar month** , the way these
  costs are actually paid in real life
* Total daily costs and net daily profit
* **Cumulative net cash position** — running total starting at
  –UGX 5,000,000 (the initial investment) and climbing as profits accrue
* A **"Breakeven Reached?"** flag (YES/No) for every day

This sheet was originally formula-driven from the Assumptions file; because
it's now a standalone file, the numbers are saved as fixed values.

### 3_Monthly_Summary.xlsx

The 1,826 daily rows rolled up into  **60 months** : transactions, revenue,
costs, net profit, and month-end cumulative cash position. Useful for
spotting the month-to-month trend without scrolling through daily detail.

### 4_Annual_Summary.xlsx

The same data rolled up into  **5 annual rows plus a 5-year total row** :
transactions, revenue, costs, net profit, and year-end cumulative cash
position for each of the 5 years.

---

## Key result: Break-even

| Metric                                                  | Value                       |
| ------------------------------------------------------- | --------------------------- |
| Total Initial Investment                                | UGX 5,000,000               |
| Average net daily profit (Year 1)                       | ≈ UGX 5,015                |
| Break-even day (dynamic, from cumulative cash position) | **Day 1,550**         |
| Break-even date                                         | **30 March 2030**     |
| Break-even falls in                                     | **Simulation Year 5** |
| 5-year total net profit                                 | ≈ UGX 6,254,025            |
| Cumulative cash position at end of Year 5               | ≈ UGX 1,254,025            |

Break-even is the day the running cumulative cash position (Daily Ledger,
"Cumulative Net Cash Position" column) crosses from negative to zero or
positive — the point at which accumulated profit has fully repaid the
UGX 5,000,000 starting capital. Every shilling of profit earned after that
day is clean profit for the investor.

---

## How the pieces fit together

```
1_Assumptions.xlsx
        │  (growth rates, commission, costs, capital)
        ▼
2_Daily_Ledger.xlsx  ──── one row per day, 1,826 rows
        │  (SUMIF rollups by month / year)
        ▼
3_Monthly_Summary.xlsx        4_Annual_Summary.xlsx
```

In the original combined workbook, all four sheets were formula-linked, so
changing an assumption automatically recalculated every day, month, and
year. In this split-file version, each file stands alone with its values
locked in as of the last calculation — changing a number in
`1_Assumptions.xlsx` will **not** automatically update the other three
files. If you need to change an assumption and regenerate consistent
numbers everywhere, the Daily Ledger, Monthly Summary, and Annual Summary
need to be rebuilt from the Assumptions again.

---

## Modelling notes / assumptions worth knowing

* Transaction volumes follow a smooth compounding growth trend (2%/month
  in Year 1, 0.5%/month from Year 2), then are scaled down by 40% on
  weekends to reflect the student/staff customer base.
* Monthly costs are booked in full on the 1st of each calendar month
  rather than spread evenly across days — this mirrors how rent and wages
  are actually paid out.
* An attendant wage is only introduced from Month 13 onward, on the
  assumption the owner runs the kiosk alone in Year 1.
* A small monthly cost inflation (0.3%) is applied from Month 13 onward.
* All figures are in Ugandan Shillings (UGX).

These are reasonable planning assumptions for a simulation exercise, not
guaranteed real-world figures — they can and should be adjusted to match
whatever data or scenario your assignment specifies.
