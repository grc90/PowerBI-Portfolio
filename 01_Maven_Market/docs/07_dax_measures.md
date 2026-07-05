# DAX Measures

All measures live in a dedicated `_Measures` table (empty table, single hidden placeholder column) to keep the Fields pane organized. Built in four layers, each verified against source totals before the next layer was built.

## Layer 1 — Base measures

Direct aggregations over fact tables. Everything else composes from these.

```dax
Total Spend = SUM ( Fact_Spend[Amount] )

Total Purchases = SUM ( Fact_Channel[NumPurchases] )

Total Accepted = SUM ( Fact_Campaign[Accepted] )

Customer Count = COUNTROWS ( Dim_Customer )
```

**Verified values (no filter):** Total Spend = 1,356,988 · Total Purchases = 28,083 · Total Accepted = 1,001 · Customer Count = 2,240

## Layer 2 — Ratios and averages

All divisions use `DIVIDE()`, never the `/` operator, so a zero-row filter context returns blank instead of breaking the visual.

```dax
Total Campaign Offers = COUNTROWS ( Fact_Campaign )

Response Rate = DIVIDE ( [Total Accepted], [Total Campaign Offers] )

Avg Spend per Customer = DIVIDE ( [Total Spend], [Customer Count] )

Purchases per Customer = DIVIDE ( [Total Purchases], [Customer Count] )

Acceptance per Customer = DIVIDE ( [Total Accepted], [Customer Count] )
```

**Verified values (no filter):** Total Campaign Offers = 13,440 · Response Rate = 7.45% · Avg Spend per Customer = 605.80 · Purchases per Customer = 12.54 · Acceptance per Customer = 0.45

**Context test passed:** filtering to Spain recalculates all ratios correctly (e.g., Avg Spend per Customer stays ~605, confirming numerator and denominator move together rather than one being stuck).

## Layer 3 — Contextual measures

Use `CALCULATE` to apply specific filters, answering the brief's question about which campaign performed best — as a rate, not a raw count, correcting the prior analysis's methodology.

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

**Key insight:** the most recent campaign achieved a response rate ~2.5× the historical average, while overall campaign reach was only 27.19% — roughly 3 in 4 customers never accepted any of the six campaigns offered over two years.

**Logic check passed:** `Customers Who Accepted Any` (609) ≤ `Total Accepted` (1,001), confirming the distinct-customer count cannot exceed the total acceptance-event count.

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

### Design note: ALL() vs ALLSELECTED()

These measures were initially built with `ALL()` instead of `ALLSELECTED()`. Testing surfaced an inconsistency: with a `Country = Spain` filter applied, `% of Total Spend` and `% of Total Purchases` correctly summed to 100% within category/channel breakdowns, but `% of Customers` collapsed to 48.88% — the country's share of the *global* customer base, not 100% of the filtered subset.

**Root cause:** `ALL(Dim_Customer)` clears every filter on the entire table, including the `Country` filter coming from the slicer (since `Country` lives on that same table). `ALL(Fact_Spend[Category])`, by contrast, only clears the `Category` column — the `Country` filter (applied via a different table) survives untouched.

**Fix:** replaced `ALL()` with `ALLSELECTED()` across all three measures. `ALLSELECTED` clears the breakdown field introduced by the visual (e.g., `Age_Bucket` in a table) while preserving filters explicitly set by slicers elsewhere on the page. This is the standard DAX pattern for "% of total" measures that must remain consistent regardless of which dimension is used to break them down, or which slicers are active.

**Verified:** with `Country = Spain` applied, a table broken down by `Fact_Spend[Category]`, `Fact_Channel[Channel]`, and `Dim_Customer[Age_Bucket]` each sums to exactly 100% — and the underlying absolute values reconstruct the Layer 1 totals for Spain (662,220 spend, 13,583 purchases, 1,095 customers), cross-validating both layers against each other.

## Naming convention

- Measures referenced without a table name: `[Total Spend]`
- Columns referenced with their table: `Fact_Spend[Amount]`
- Never reference a column without its table, even where DAX permits it — the convention exists so a formula's structure is legible at a glance.

## Verification discipline

Every layer was validated against ground-truth totals (verified directly from the source Excel where the original claim in this project's write-ups was approximate) before the next layer was built on top of it. The `% of Customers` bug above is a direct product of that discipline — it was caught by testing table breakdowns under an active slicer, not by trusting card visuals alone (cards don't expose row/column context bugs, since there's no breakdown dimension for `ALL` vs `ALLSELECTED` to disagree on).