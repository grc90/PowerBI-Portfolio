# Power Query Pipeline

The ETL for this project is built entirely in Power Query using a **two-layer architecture**: a staging layer that centralizes all data quality remediation, and a model layer that references staging and shapes it into the final star schema.

## Architecture

```
STAGING LAYER (load disabled — not visible in the model)
  Src_MarketingData         ← raw import, unmodified
  Stg_MarketingData_Clean   ← all data quality fixes applied

MODEL LAYER (load enabled — visible in the model)
  Dim_Customer     ← references Stg_MarketingData_Clean
  Fact_Spend       ← references Stg_MarketingData_Clean, unpivots 6 Mnt* columns
  Fact_Channel     ← references Stg_MarketingData_Clean, unpivots 3 channel columns
  Fact_Campaign    ← references Stg_MarketingData_Clean, unpivots 6 campaign columns
  Dim_Campaign     ← manually created (6 rows)
  Dim_Date         ← generated from M code, bounded to 2012–2014
```

### Why two layers?

Every downstream query needs the same data quality fixes: nulled income zeros, nulled implausible birth years, recoded marital statuses. Applying these fixes once in a staging query and referencing it from downstream queries means:

- Fixes are written and audited once, not five times.
- New data quality issues can be added to one place and propagate everywhere.
- Downstream queries stay focused on shaping, not cleaning — separation of concerns.

The alternative (five queries each loading the raw file independently) is a maintenance liability that drifts out of sync.

## Query 1 — `Src_MarketingData`

**Load disabled.** Raw import from `Marketing_Campaign.xlsx`, sheet `marketing_data`. First row promoted as headers. No transformations applied — this is the untouched source of truth, kept accessible in case an issue needs to be traced back to the original data.

## Query 2 — `Stg_MarketingData_Clean`

**Load disabled.** References `Src_MarketingData`. Applies all data quality remediations documented in `02_data_quality.md`.

Applied steps (in order):

| Step | Purpose |
|---|---|
| `Trim Income Header` | Renames `"Income "` → `"Income"` (removes trailing space) |
| `Set Column Types` | Explicit type for every column — no reliance on auto-detection |
| `Null Invalid Incomes` | Conditional Column: sets `Income` to null when 0 or 666666 |
| `Null Implausible Birth Years` | Conditional Column: sets `Year_Birth` to null when ≤ 1900 |
| `Recode Alone to Single` | `Marital_Status` normalization |
| `Recode YOLO to Other` | `Marital_Status` normalization |
| `Recode Absurd to Other` | `Marital_Status` normalization |

### Note on Conditional Column vs Replace Values for numeric nulls

For numeric columns, the Replace Values dialog does not accept an empty value — it requires a numeric input. Empty fields in Conditional Column, however, produce a true `null` if the resulting column is typed numeric immediately. The nulling of `Income` and `Year_Birth` was implemented via Conditional Column with explicit numeric typing afterwards, avoiding the common pitfall where a "blank" value silently becomes an empty string (`""`) instead of null.

## Query 3 — `Dim_Customer`

References `Stg_MarketingData_Clean`. Steps:

1. Select customer-attribute columns (drops `Mnt*`, `AcceptedCmp*`, `Response`, and the three channel `Num*Purchases` columns — those go to facts)
2. Rename `ID` → `CustomerID`, `Dt_Customer` → `Enrollment_Date`
3. Add `Age` — custom column: `if [Year_Birth] = null then null else 2014 - [Year_Birth]`
4. Add `Age_Bucket` — conditional column (null → Unknown, ≤34 → Young Adult, ≤54 → Adult, else Senior). The null-check clause must come first, since numeric comparisons against null return null (neither true nor false) and would fall through to the else branch.
5. Add `Income_Bucket` — same pattern (null → Unknown, <35k → Low, ≤68k → Mid, else High)
6. Add `Has_Children` — custom column: `if [Kidhome] + [Teenhome] > 0 then "Yes" else "No"`
7. Reorder columns and set final types

## Query 4 — `Fact_Spend`

References `Stg_MarketingData_Clean`. Steps:

1. Select `ID` + the 6 `Mnt*` columns
2. Rename `ID` → `CustomerID`
3. **Unpivot** the 6 `Mnt*` columns → produces `CustomerID`, `Attribute`, `Value`
4. Rename `Attribute` → `Category`, `Value` → `Amount`
5. Six Replace Values steps to clean category names (`MntWines` → `Wines`, `MntMeatProducts` → `Meat`, etc.)
6. Set types

Row count: 13,440 (= 2,240 × 6). Total `Amount` matches the sum of the six original `Mnt*` columns.

## Query 5 — `Fact_Channel`

References `Stg_MarketingData_Clean`. Steps:

1. Select `ID` + the 3 real channel columns (`NumWebPurchases`, `NumCatalogPurchases`, `NumStorePurchases`). `NumDealsPurchases` is deliberately excluded.
2. Rename `ID` → `CustomerID`
3. Unpivot the 3 channel columns
4. Rename `Attribute` → `Channel`, `Value` → `NumPurchases`
5. Three Replace Values steps: `NumWebPurchases` → `Web`, etc.
6. Set types

Row count: 6,720 (= 2,240 × 3). Total `NumPurchases` = 28,083, matching the sum of the three original channel columns.

## Query 6 — `Fact_Campaign`

References `Stg_MarketingData_Clean`. Steps:

1. Select `ID` + `AcceptedCmp1`–`AcceptedCmp5` + `Response`
2. Rename `ID` → `CustomerID`
3. Unpivot the 6 campaign columns
4. Rename `Value` → `Accepted` (keep `Attribute` temporarily)
5. **Map source column name to `CampaignID`** — conditional column: `AcceptedCmp1` → 1, `AcceptedCmp2` → 2, …, `Response` → 6. Output type explicitly set to Integer to avoid string keys.
6. Remove the temporary `Attribute` column
7. Reorder columns and set types

Row count: 13,440 (= 2,240 × 6). `SUM(Accepted)` = 1,001, matching the sum of the six original campaign columns (144 + 30 + 163 + 167 + 163 + 334). Per-campaign breakdown confirms mapping is correct: `CampaignID = 1` returns 144, `CampaignID = 6` returns 334.

### Why map to an integer key instead of joining on the source column name

A join on text (`"AcceptedCmp1"` = `"AcceptedCmp1"`) would work but has three drawbacks: relationships on integers are faster than on text, renaming `"Last Campaign"` in `Dim_Campaign` would require no change in `Fact_Campaign`, and dimensional modeling convention uses surrogate keys (integers) rather than natural keys (text).

## Query 7 — `Dim_Campaign`

Manually created via **Especificar datos**. 6 rows, 3 columns (`CampaignID`, `Campaign_Name`, `Is_Latest`). The `Is_Latest` flag identifies the most recent campaign (`Response`, mapped to `CampaignID = 6`) so DAX measures can filter on the dimension rather than hardcoding a column name.

## Query 8 — `Dim_Date`

Generated from a blank query via M code in the Advanced Editor. Bounded to `2012-01-01` through `2014-12-31` (1,096 rows). Columns: `Date`, `Year`, `MonthNum`, `MonthName`, `YearMonth`, `Quarter`.

Marked as the model's date table in Model view. Combined with disabling "Auto date/time" in the file's data load options, this prevents Power BI from generating implicit date hierarchies that would compete with `Dim_Date`.

## Relationships (built in Model view)

Configured after Close & Apply. All five relationships: 1:*, single-direction filter, active.

| From | Column | To | Column |
|---|---|---|---|
| `Dim_Customer` | `CustomerID` | `Fact_Spend` | `CustomerID` |
| `Dim_Customer` | `CustomerID` | `Fact_Channel` | `CustomerID` |
| `Dim_Customer` | `CustomerID` | `Fact_Campaign` | `CustomerID` |
| `Dim_Campaign` | `CampaignID` | `Fact_Campaign` | `CampaignID` |
| `Dim_Date` | `Date` | `Dim_Customer` | `Enrollment_Date` |

## Key learnings from this pipeline

- **Type your columns immediately after creation.** A column typed as `ABC 123` ("Any") will silently store empty strings where you expect nulls. Every calculated or conditional column should be explicitly typed as its next step.
- **Prefer Conditional Column over Replace Values for numeric nulls.** Replace Values requires a valid numeric input; Conditional Column allows an empty output that becomes null when the column is typed.
- **Verify totals after every unpivot.** The sum of the unpivoted `Value` column should exactly equal the sum of the original columns. Any deviation signals a filter or type problem.
- **Rename applied steps as you go.** `Replaced Value1`, `Replaced Value2` is not documentation. `Recode Alone to Single` is.