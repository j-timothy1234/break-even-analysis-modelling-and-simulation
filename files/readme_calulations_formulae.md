### How the Calculations Work (Formulas & Rationale)

Every number in this project flows from **`1_Assumptions.xlsx`** through a chain of Excel formulas. Below is exactly what each calculation does and why it was built that way.

#### 1. Total Initial Investment

```
Total Initial Investment = Float Capital + Airtime/Data Stock + Kiosk Setup
                            + Rent Deposit + Equipment
=SUM(B5:B9)   → UGX 5,000,000
```

**Why:** breakeven only means something once you know the exact amount the investor needs to recover. Every one-off cost of getting the kiosk running is added into a single capital figure.

#### 2. Converting monthly assumptions into daily ones

The assignment gives a **monthly** forecast (~500 transactions/month), but the ledger runs  **daily** . Two conversions were needed:

**a) Starting daily transaction count** (simple average):

```
Daily MM transactions (Day 1) = Monthly MM transactions / 30.44
```

30.44 is used instead of 30 because it's the true average number of days in a month across a year (365.25 ÷ 12) — it keeps the yearly total consistent no matter how the days fall.

**b) Daily-equivalent growth rate** (compounding, not simple division):

```
Daily growth rate = (1 + Monthly growth rate)^(1/30.44) − 1
```

**Why not just divide the monthly rate by 30?** Growth compounds. If transactions grow 2% every month, they don't grow exactly 2%/30 = 0.067% every day — they grow by a slightly smaller daily rate that, compounded over ~30.44 days, produces the same 2% monthly result. The formula above is the standard way to convert a compounding periodic rate into a compounding rate for a shorter period, so the daily model and the "monthly forecast" stay mathematically consistent.

#### 3. The daily transaction trend (before weekday effects)

```
Trend(Day 1)   = Assumptions!Daily starting transactions
Trend(Day n)   = Trend(Day n−1) × (1 + Daily growth rate for the current Sim Year)
```

This is a **recursive/compounding formula** — each day's trend is built on the previous day's, growing smoothly over time. It uses Year 1's daily growth rate for the first 365 days and the (slower) Year 2+ rate afterward, matching the idea that a new kiosk grows fast while it's building a customer base, then levels off.

**Why keep a separate "trend" column instead of applying growth directly to actual transactions?** If the weekend dip (see below) were baked into the growth chain, the model would compound a shrunken weekend number into next week's trend, artificially dragging growth down. Keeping trend and actual separate keeps the underlying growth curve smooth and the weekday effect purely cosmetic on top of it.

#### 4. The weekday/weekend effect

```
Weekday Multiplier = IF(WEEKDAY(Date,2) >= 6, Weekend Multiplier, 1)
Actual Transactions = ROUND(Trend × Weekday Multiplier, 0)
```

`WEEKDAY(date, 2)` numbers Monday=1 … Sunday=7, so 6 and 7 are Saturday/Sunday. The kiosk's customers are students and staff, so a 60% weekend multiplier was assumed — footfall drops but doesn't vanish (people still send/receive money and buy airtime on weekends, just less often).

#### 5. Revenue

```
Commission Revenue = MM Transactions (actual) × Average commission per transaction
Airtime Revenue    = Airtime Transactions (actual) × Average markup per sale
Total Daily Revenue = Commission Revenue + Airtime Revenue
```

This directly implements the assignment's two named revenue streams — "commission per transaction" and "markup on data bundles" — with no other logic layered on.

#### 6. Costs — inflation factor and lump-sum booking

```
Cost Inflation Factor = IF(Calendar Month >= 13, (1 + Monthly inflation)^(Month − 12), 1)
```

This compounds a small monthly cost inflation (0.3%) starting in Month 13, so costs drift upward realistically in Years 2–5 rather than staying frozen for 5 years.

```
Rent (that day)  = IF(DAY(Date) = 1, Monthly Rent × Inflation Factor, 0)
Wage (that day)  = IF(AND(DAY(Date) = 1, Month >= 13), Monthly Wage × Inflation Factor, 0)
```

(Transport, Agent Charges, Utilities, Misc follow the same `IF(DAY(Date)=1, …)` pattern.)

**Why book costs only on day 1 of each month instead of spreading them daily?** Rent, wages and float top-up transport are real-world costs that get **paid** on specific days (typically month-start), not accrued continuously. Booking them as a lump sum on day 1 mirrors an actual cash ledger — you'd see a big cost entry on the 1st and none for the rest of the month, exactly like a real kiosk owner's books would look.

#### 7. Net profit and the running balance

```
Total Daily Costs      = SUM(Rent : Misc)
Net Daily Profit       = Total Daily Revenue − Total Daily Costs
Cumulative Cash (Day 1)  = −Total Initial Investment + Net Daily Profit
Cumulative Cash (Day n)  = Cumulative Cash (Day n−1) + Net Daily Profit
```

This is a standard running-balance formula. It starts **negative** (−5,000,000) because the investor is "in the hole" by the full startup cost before the kiosk earns a single shilling, then climbs every day the kiosk turns a profit.

#### 8. Finding the break-even day

```
Break-even Day # = INDEX(Day#, MATCH(TRUE, Cumulative Cash >= 0, 0))
```

This is an **array-style INDEX/MATCH** lookup: it scans the Cumulative Cash column for the *first* day the value is ≥ 0, and returns that day's number. This is more accurate than a simple "capital ÷ average profit" estimate because it accounts for the actual, uneven day-to-day pattern (growth, weekends, monthly cost spikes) instead of assuming a flat average.

A simpler **sanity-check** version is also included:

```
Simple break-even (days) = Total Initial Investment / Average Net Daily Profit (Year 1)
```

This gives a rough estimate assuming Year 1's average profit held constant forever — useful for cross-checking that the dynamic (real) answer is in the right ballpark, but less accurate since it ignores growth and monthly cost spikes.

#### 9. Monthly and Annual roll-ups

```
Month's Total Revenue = SUMIF(Daily Ledger Month# column, this month, Daily Ledger Revenue column)
Month-end Cash Balance = LOOKUP(2, 1/(Month# = this month), Cumulative Cash column)
```

`SUMIF` totals every day belonging to a given month/year for revenue, costs, and profit. The `LOOKUP(2, 1/(condition), …)` formula is a common Excel trick to grab the **last matching value** in a range (here, the cumulative cash balance on the *last* day of that month/year) — a normal `SUMIF` can't do this because a running balance shouldn't be summed, only the final day's figure is meaningful.
