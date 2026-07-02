# Data Model

The raw dataset arrives as a single flat table (2,240 rows × 28 columns). Rather than modeling directly against this wide structure, it is restructured into a **star/constellation schema** with one customer dimension, three fact tables (spend, channel, campaign), a campaign dimension, and a minimal date dimension.

## Why not keep it flat?

Three groups of columns follow the same repeating pattern: 6 product spend columns, 3 purchase channel columns, and 6 campaign response columns. Every business question on the Campaign/Channel and Product pages involves comparing *across* these repeated columns.

In wide format, this requires one measure per column (six `[Total Wines]`, `[Total Fruits]`… measures for a single bar chart). In long format, one measure per fact — sliced by category, channel, or campaign — produces the same visual and composes naturally with every other filter.

The restructuring is driven by the business questions, not by "star schema for its own sake."

## Model diagram

```
                      Dim_Date
                         │ 1
                         ▼ *
                    Dim_Customer (2,240)
                    /     │      \
                 1 ▼    1 ▼     1 ▼
                  *      *       *
        Fact_Spend  Fact_Channel  Fact_Campaign
         (13,440)      (6,720)      (13,440)
                                        ▲ *
                                        │ 1
                                   Dim_Campaign (6)
```

All 5 relationships: **1:\*, single-direction filter**, active.

| From | Column | To | Column |
|---|---|---|---|
| `Dim_Customer` | `CustomerID` | `Fact_Spend` | `CustomerID` |
| `Dim_Customer` | `CustomerID` | `Fact_Channel` | `CustomerID` |
| `Dim_Customer` | `CustomerID` | `Fact_Campaign` | `CustomerID` |
| `Dim_Campaign` | `CampaignID` | `Fact_Campaign` | `CampaignID` |
| `Dim_Date` | `Date` | `Dim_Customer` | `Enrollment_Date` |

`Dim_Date` is marked as the model's date table, disabling Power BI's auto-generated date hierarchies.

## Tables

### `Dim_Customer` (2,240 rows)

One row per customer. Holds all demographic and behavioral attributes used for segmentation.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (PK) | From raw `ID` |
| `Year_Birth` | Integer | Nulled for 3 rows with implausible values (≤1900) |
| `Age` | Integer (calc) | `2014 - Year_Birth`. Fixed to snapshot year, not `TODAY()` |
| `Age_Bucket` | Text | Young Adult (≤34), Adult (35–54), Senior (55+), Unknown |
| `Education` | Text | 5 categories retained as-is |
| `Marital_Status` | Text | Cleaned: `Alone`→`Single`; `YOLO`/`Absurd`→`Other` |
| `Income` | Decimal | 25 rows nulled (24 zero-fills + 1 outlier at $666,666) |
| `Income_Bucket` | Text | Low (<$35k), Mid ($35k–$68k), High (>$68k), Unknown |
| `Kidhome`, `Teenhome` | Integer | Retained for granular analysis |
| `Has_Children` | Text | `"Yes"` / `"No"` based on `Kidhome + Teenhome > 0` |
| `Country` | Text | 8 countries |
| `Enrollment_Date` | Date | From raw `Dt_Customer` |
| `Recency` | Integer | Days since last purchase (snapshot value) |
| `Complain` | Integer | 1 if complained in last 2 years |
| `NumWebVisitsMonth` | Integer | Web visits in the last month |
| `NumDealsPurchases` | Integer | Kept in `Dim_Customer` as a pricing-mode attribute, not a channel |

### `Fact_Spend` (13,440 rows = 2,240 × 6)

Unpivoted from the 6 `Mnt*` columns. One row per customer × product category.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (FK → Dim_Customer) | |
| `Category` | Text | Wines, Fruits, Meat, Fish, Sweets, Gold |
| `Amount` | Integer | Spend in the last 2 years |

### `Fact_Channel` (6,720 rows = 2,240 × 3)

Unpivoted from the 3 real channel columns. One row per customer × channel.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (FK → Dim_Customer) | |
| `Channel` | Text | Web, Catalogue, Store |
| `NumPurchases` | Integer | Purchase count |

`NumDealsPurchases` deliberately excluded — a deal is a pricing mode, not a channel. Including it would double-count purchases already captured in Web/Catalogue/Store.

### `Fact_Campaign` (13,440 rows = 2,240 × 6)

Unpivoted from the 5 `AcceptedCmpN` columns plus `Response`. One row per customer × campaign.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (FK → Dim_Customer) | |
| `CampaignID` | Integer (FK → Dim_Campaign) | 1–6, mapped from source column name |
| `Accepted` | Integer | 0 or 1 |

### `Dim_Campaign` (6 rows)

| CampaignID | Campaign_Name | Is_Latest |
|---|---|---|
| 1 | Campaign 1 | 0 |
| 2 | Campaign 2 | 0 |
| 3 | Campaign 3 | 0 |
| 4 | Campaign 4 | 0 |
| 5 | Campaign 5 | 0 |
| 6 | Last Campaign | 1 |

### `Dim_Date` (1,096 rows, 2012-01-01 to 2014-12-31)

Standard date table linked to `Enrollment_Date`. Bounded to 2012–2014 because that's when enrollments occurred. Columns: `Date`, `Year`, `MonthNum`, `MonthName`, `YearMonth`, `Quarter`.

## Design decisions and rationale

- **Buckets pre-built in Power Query, not calculated in DAX.** Bucket definitions are modeling decisions, not runtime logic. Centralizing them in the ETL layer means one place to change definitions.
- **Income and age bucket thresholds are data-driven** (based on dataset quartiles), not arbitrary fixed dollar amounts. This makes each bucket meaningful relative to *this* customer base.
- **`Age` calculated as `2014 - Year_Birth`**, not `TODAY() - Year_Birth`. The dataset is a fixed historical snapshot; using `TODAY()` would silently age every customer by 10+ years and corrupt every age-based visual.
- **`NumDealsPurchases` stays in `Dim_Customer`** rather than joining the channel fact. Deals are a pricing mode; treating them as a fourth channel would double-count purchases already captured in Web/Catalogue/Store.
- **`Is_Latest` flag in `Dim_Campaign`** rather than special-casing the `Response` column in DAX. Keeps campaign logic uniform — measures filter on the dimension, not on hardcoded column names.
- **Single-direction filter on all relationships.** Bidirectional filtering introduces ambiguity in multi-fact models. Default to single; only change with a concrete tested reason.
- **`Accepted`, `Is_Latest`, `Complain` stored as integers, not booleans.** Enables `SUM()` in DAX without conversion.

## Model validation

Post-load verification passed:

- Row counts: `Dim_Customer` = 2,240, `Fact_Spend` = 13,440, `Fact_Channel` = 6,720, `Fact_Campaign` = 13,440, `Dim_Campaign` = 6, `Dim_Date` = 1,096
- `SUM(Fact_Channel[NumPurchases])` = 28,083, matching the sum of the 3 original channel columns
- `SUM(Fact_Campaign[Accepted])` = 1,001, matching the sum of the 6 original campaign flag columns (144 + 30 + 163 + 167 + 163 + 334)
- Per-campaign breakdown validates that `AcceptedCmp1` → `CampaignID = 1` and `Response` → `CampaignID = 6`, ruling out inverted mapping
- Country-level slicing of all three facts returns distinct per-country totals, confirming relationships filter correctly