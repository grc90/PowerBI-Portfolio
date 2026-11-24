# DAX Measures

Every measure lives in a dedicated `_Measures` table (an empty table with one hidden placeholder column) so the Fields pane stays tidy. I built them in four layers and checked each one against source totals before building the next on top of it.

## Layer 1 — Base measures

Plain aggregations over the fact tables. Everything else is built from these.

```dax
Total Spend = SUM ( Fact_Spend[Amount] )

Total Purchases = SUM ( Fact_Channel[NumPurchases] )

Total Accepted = SUM ( Fact_Campaign[Accepted] )

Customer Count = COUNTROWS ( Dim_Customer )
```

**Verified values (no filter):** Total Spend = 1,356,988 · Total Purchases = 28,083 · Total Accepted = 1,001 · Customer Count = 2,240

## Layer 2 — Ratios and averages

Every division uses `DIVIDE()`, never the `/` operator. When a filter leaves zero rows, the visual shows a blank instead of an error.

```dax
Total Campaign Offers = COUNTROWS ( Fact_Campaign )

Response Rate = DIVIDE ( [Total Accepted], [Total Campaign Offers] )

Avg Spend per Customer = DIVIDE ( [Total Spend], [Customer Count] )

Purchases per Customer = DIVIDE ( [Total Purchases], [Customer Count] )

Acceptance per Customer = DIVIDE ( [Total Accepted], [Customer Count] )
```

**Verified values (no filter):** Total Campaign Offers = 13,440 · Response Rate = 7.45% · Avg Spend per Customer = 605.80 · Purchases per Customer = 12.54 · Acceptance per Customer = 0.45

**Context test passed:** with a Spain filter, every ratio recalculates correctly. Avg Spend per Customer, for example, stays around 605, which tells me the numerator and denominator move together and neither one is stuck at the global value.

## Layer 3 — Contextual measures

These use `CALCULATE` to apply specific filters. They answer the brief's question about which campaign performed best, this time as a rate instead of a raw count, which fixes the method used in the previous analysis.

```dax
Latest Campaign Response Rate = 
CALCULATE ( [Response Rate], Dim_Campaign[Is_Latest] = 1 )

Historical Response Rate = 
CALCULATE ( [Response Rate], Dim_Campaign[Is_Latest] = 0 )

Customers Who Accepted Any = 
CALCULATE ( DISTINCTCOUNT ( Fact_Campaign[CustomerID] ), Fact_Campaign[Accepted] = 1 )

Campaign Reach Rate = DIVIDE ( [Customers Who Accepted Any], [Customer Count] )
```

**Verified values (no filter):** Latest Campaign Response Rate = 14.91% · Historical Response Rate = 5.96% · Customers Who Accepted Any = 609 · Campaign Reach Rate = 27.19%

**What stood out:** the latest campaign's response rate is about 2.5× the historical average. Overall reach, though, is only 27.19%: roughly 3 in 4 customers didn't accept a single one of the six campaigns over two years.

**Logic check passed:** `Customers Who Accepted Any` (609) ≤ `Total Accepted` (1,001). The number of distinct customers can't be higher than the number of acceptances, so this is a cheap sanity check worth keeping.

## Layer 4 — Percentage of total

```dax
% of Total Spend = 
DIVIDE (
    [Total Spend],
    CALCULATE ( [Total Spend], ALLSELECTED ( Fact_Spend[Category] ) )
)

% of Total Purchases = 
DIVIDE (
    [Total Purchases],
    CALCULATE ( [Total Purchases], ALLSELECTED ( Fact_Channel[Channel] ) )
)

% of Customers = 
DIVIDE (
    [Customer Count],
    CALCULATE ( [Customer Count], ALLSELECTED ( Dim_Customer ) )
)
```

### The bug that taught me ALL() vs ALLSELECTED()

My first version of these measures used `ALL()`. Testing with `Country = Spain` turned up an inconsistency: `% of Total Spend` and `% of Total Purchases` added up to 100% across categories and channels, as they should, but `% of Customers` only reached 48.88%. That's Spain's share of the *global* customer base, not 100% of the filtered group.

**Root cause:** `ALL(Dim_Customer)` removes every filter on that table, including the `Country` filter from the slicer, because `Country` lives on that same table. `ALL(Fact_Spend[Category])` only clears the `Category` column, so a `Country` filter coming from another table survives.

**Fix:** I switched all three measures to `ALLSELECTED()`. It clears the breakdown field the visual introduces (say, `Age_Bucket` in a table) but keeps the filters set by slicers elsewhere on the page. It's the standard DAX pattern for a "% of total" that has to stay consistent whatever dimension you break it down by and whatever slicers are active.

**Verified:** with `Country = Spain`, tables broken down by `Fact_Spend[Category]`, `Fact_Channel[Channel]` and `Dim_Customer[Age_Bucket]` each add up to exactly 100%. The absolute values underneath also rebuild the Layer 1 totals for Spain (662,220 spend, 13,583 purchases, 1,095 customers), so the two layers confirm each other.

## Added for the report

```dax
Avg Income = AVERAGE ( Dim_Customer[Income] )

Avg Age = AVERAGE ( Dim_Customer[Age] )
```

**Verified values (no filter):** Avg Income = $51,970 · Avg Age = 45

`AVERAGE` skips blanks, so the 25 nulled incomes and 3 nulled birth years drop out on their own instead of pulling the average down.

## Format strings

Formats are set on the measures, not on each visual: currency `$#,0` for spend and income, `0.00 %` for every rate and % of total, and `0.0` for Purchases per Customer. Before I did this, every new visual showed raw decimals like 0.0745.

## Naming convention

- Measures are referenced without a table name: `[Total Spend]`
- Columns are always referenced with their table: `Fact_Spend[Amount]`
- I never reference a column without its table, even where DAX allows it. The point is that you can tell measures from columns at a glance.

## Why I verify every layer

Each layer was checked against ground-truth totals before I built anything on top of it. Where a figure in my own earlier notes was only approximate, I went back to the source Excel to confirm it. The `% of Customers` bug is a good example of why this matters: I caught it by testing breakdown tables with a slicer active. Card visuals would never have shown it. A card has no breakdown dimension, so `ALL` and `ALLSELECTED` return the same thing and the bug stays hidden.
