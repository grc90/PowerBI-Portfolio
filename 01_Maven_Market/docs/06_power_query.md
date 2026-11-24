# Power Query Pipeline

I built the whole ETL in Power Query with **two layers**: a staging layer where all the data quality fixes live, and a model layer that reads from staging and shapes the data into the final star schema.

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

Every downstream query needs the same fixes: income zeros nulled, implausible birth years nulled, marital statuses recoded. If I apply them once in a staging query and have everything else reference it:

- I write and check each fix once, not five times.
- A new data quality issue gets fixed in one place and the fix reaches every table.
- The downstream queries only deal with shaping the data, not cleaning it.

The alternative, five queries each loading the raw file on their own, works on day one and then slowly drifts out of sync. I'd rather not maintain that.

## Query 1 — `Src_MarketingData`

**Load disabled.** Raw import from `Marketing_Campaign.xlsx`, sheet `marketing_data`, with the first row promoted to headers. Nothing else. I keep it untouched so I can always trace a problem back to the original data.

## Query 2 — `Stg_MarketingData_Clean`

**Load disabled.** References `Src_MarketingData` and applies every fix documented in [`03_data_quality.md`](03_data_quality.md).

Applied steps, in order:

| Step | Purpose |
|---|---|
| `Trim Income Header` | Renames `"Income "` → `"Income"` (removes trailing space) |
| `Set Column Types` | Explicit type for every column, no relying on auto-detection |
| `Null Invalid Incomes` | Conditional Column: sets `Income` to null when 0 or 666666 |
| `Null Implausible Birth Years` | Conditional Column: sets `Year_Birth` to null when ≤ 1900 |
| `Recode Alone to Single` | `Marital_Status` normalization |
| `Recode YOLO to Other` | `Marital_Status` normalization |
| `Recode Absurd to Other` | `Marital_Status` normalization |

### Conditional Column vs Replace Values for numeric nulls

This one cost me some time. On a numeric column, the Replace Values dialog won't accept an empty value; it insists on a number. A Conditional Column with an empty output, on the other hand, produces a real `null`, as long as you type the column as numeric right away. So I nulled `Income` and `Year_Birth` with Conditional Columns and typed them immediately after. Otherwise you fall into a classic trap: the "blank" ends up stored as an empty string (`""`) instead of a null.

## Query 3 — `Dim_Customer`

References `Stg_MarketingData_Clean`. Steps:

1. Keep only the customer attributes (drops `Mnt*`, `AcceptedCmp*`, `Response` and the three channel `Num*Purchases` columns, which go to the facts)
2. Rename `ID` → `CustomerID`, `Dt_Customer` → `Enrollment_Date`
3. Add `Age`: custom column `if [Year_Birth] = null then null else 2014 - [Year_Birth]`
4. Add `Age_Bucket`: conditional column (null → Unknown, ≤34 → Young Adult, ≤54 → Adult, else Senior). The null check has to come first. Comparing a number against null returns null, which is neither true nor false, so those rows would slip through to the else branch and get labeled Senior.
5. Add `Income_Bucket` with the same pattern (null → Unknown, <35k → Low, ≤68k → Mid, else High)
6. Add `Has_Children`: custom column `if [Kidhome] + [Teenhome] > 0 then "Yes" else "No"`
7. Add `Age_Bucket_Sort` / `Income_Bucket_Sort` (1–4) as sort-by columns
8. Reorder columns and set final types

`Spend_Tier` and `Deal_Shopper` are added *before* the column selection (steps `Add Spend Tier` and `Add Deal Shopper`), because they need the `Mnt*` and channel columns that don't make it into `Dim_Customer`. The thresholds are hardcoded from the source distribution: p75 of total spend = 1,045, and p75 of deal share = 1/3.

## Query 4 — `Fact_Spend`

References `Stg_MarketingData_Clean`. Steps:

1. Select `ID` + the 6 `Mnt*` columns
2. Rename `ID` → `CustomerID`
3. **Unpivot** the 6 `Mnt*` columns → `CustomerID`, `Attribute`, `Value`
4. Rename `Attribute` → `Category`, `Value` → `Amount`
5. Six Replace Values steps to clean the category names (`MntWines` → `Wines`, `MntMeatProducts` → `Meat`, etc.)
6. Set types

Row count: 13,440 (= 2,240 × 6). Total `Amount` matches the sum of the six original `Mnt*` columns.

## Query 5 — `Fact_Channel`

References `Stg_MarketingData_Clean`. Steps:

1. Select `ID` + the 3 real channel columns (`NumWebPurchases`, `NumCatalogPurchases`, `NumStorePurchases`). `NumDealsPurchases` is left out on purpose.
2. Rename `ID` → `CustomerID`
3. Unpivot the 3 channel columns
4. Rename `Attribute` → `Channel`, `Value` → `NumPurchases`
5. Three Replace Values steps: `NumWebPurchases` → `Web`, etc.
6. Set types

Row count: 6,720 (= 2,240 × 3). Total `NumPurchases` = 28,083, which matches the sum of the three original channel columns.

## Query 6 — `Fact_Campaign`

References `Stg_MarketingData_Clean`. Steps:

1. Select `ID` + `AcceptedCmp1`–`AcceptedCmp5` + `Response`
2. Rename `ID` → `CustomerID`
3. Unpivot the 6 campaign columns
4. Rename `Value` → `Accepted` (keep `Attribute` for now)
5. **Map the source column name to `CampaignID`** with a conditional column: `AcceptedCmp1` → 1, `AcceptedCmp2` → 2, …, `Response` → 6. The output type is set to Integer explicitly so the keys don't end up as text.
6. Remove the temporary `Attribute` column
7. Reorder columns and set types

Row count: 13,440 (= 2,240 × 6). `SUM(Accepted)` = 1,001, matching the six original campaign columns (144 + 30 + 163 + 167 + 163 + 334). I also checked the mapping per campaign: `CampaignID = 1` returns 144 and `CampaignID = 6` returns 334.

### Why an integer key and not the column name

Joining on text (`"AcceptedCmp1"` = `"AcceptedCmp1"`) would work, but I see three reasons not to. Relationships on integers are faster than on text. Renaming `"Last Campaign"` in `Dim_Campaign` doesn't require touching `Fact_Campaign`. And dimensional modeling convention favors surrogate keys (integers) over natural keys (text).

## Query 7 — `Dim_Campaign`

Created by hand with **Especificar datos** (Enter Data): 6 rows, 3 columns (`CampaignID`, `Campaign_Name`, `Is_Latest`). `Is_Latest` flags the most recent campaign (`Response`, mapped to `CampaignID = 6`), so DAX measures filter on the dimension instead of hardcoding a column name.

## Query 8 — `Dim_Date`

Generated from a blank query with M code in the Advanced Editor. Runs from `2012-01-01` to `2014-12-31` (1,096 rows). Columns: `Date`, `Year`, `MonthNum`, `MonthName`, `YearMonth`, `Quarter`.

Marked as the model's date table in Model view. I also turned off "Auto date/time" in the file's data load options, so Power BI doesn't create hidden date hierarchies that compete with `Dim_Date`.

## Relationships (built in Model view)

Set up after Close & Apply. All five are 1:*, single-direction and active.

| From | Column | To | Column |
|---|---|---|---|
| `Dim_Customer` | `CustomerID` | `Fact_Spend` | `CustomerID` |
| `Dim_Customer` | `CustomerID` | `Fact_Channel` | `CustomerID` |
| `Dim_Customer` | `CustomerID` | `Fact_Campaign` | `CustomerID` |
| `Dim_Campaign` | `CampaignID` | `Fact_Campaign` | `CampaignID` |
| `Dim_Date` | `Date` | `Dim_Customer` | `Enrollment_Date` |

## What I'd tell myself before starting

- **Type every column right after you create it.** A column left as `ABC 123` ("Any") will quietly store empty strings where you expected nulls. The next step after any calculated or conditional column should be setting its type.
- **Use Conditional Column, not Replace Values, for numeric nulls.** Replace Values wants a valid number. Conditional Column lets you leave the output empty, and it becomes null once the column is typed.
- **Check totals after every unpivot.** The unpivoted `Value` column should add up to exactly the same total as the original columns. If it doesn't, something is wrong with a filter or a type.
- **Rename applied steps as you go.** `Replaced Value1` and `Replaced Value2` don't document anything. `Recode Alone to Single` does.
