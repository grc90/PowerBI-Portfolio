# Data Model

The raw dataset arrives as a single flat table (2,240 rows × 28 columns). Rather than modeling directly against this wide structure, it is restructured into a **star/constellation schema** with one customer dimension, three fact tables (spend, channel, campaign), a campaign dimension, and a minimal date dimension.

## Why not keep it flat?

Three groups of columns follow the same repeating pattern: 6 product spend columns, 3 purchase channel columns, and 6 campaign response columns. Every business question on the Campaign/Channel and Product pages involves comparing *across* these repeated columns.

In wide format, this requires one measure per column (six `[Total Wines]`, `[Total Fruits]`… measures for a single bar chart). In long format, one measure per fact — sliced by category, channel, or campaign — produces the same visual and composes naturally with every other filter.

The restructuring is driven by the business questions, not by "star schema for its own sake."

## Model diagram

```
Dim_Customer (2,240 rows)
├── CustomerID (PK)
├── Year_Birth, Age, Age_Bucket
├── Education, Marital_Status
├── Income, Income_Bucket
├── Kidhome, Teenhome, Has_Children
├── Country
├── Enrollment_Date
├── Recency, Complain, NumWebVisitsMonth, NumDealsPurchases
      │
      ├─── 1:* ─── Fact_Spend        (13,440 rows: 2,240 × 6 categories)
      ├─── 1:* ─── Fact_Channel      (6,720 rows:  2,240 × 3 channels)
      └─── 1:* ─── Fact_Campaign     (13,440 rows: 2,240 × 6 campaigns)
                        │
                        └─── *:1 ─── Dim_Campaign (6 rows)

Dim_Date (linked to Enrollment_Date only)
```

All relationships are one-to-many, single-directional (Dim → Fact). No bidirectional filtering.

## Tables

### `Dim_Customer`

One row per customer. Holds all demographic and behavioral attributes used for segmentation.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (PK) | From raw `ID` |
| `Year_Birth` | Integer | Nulled for 3 rows with implausible values (see Data Quality) |
| `Age` | Integer (calc) | `2014 - Year_Birth`. Fixed to snapshot year, not `TODAY()` |
| `Age_Bucket` | Text | Young Adult (≤34), Adult (35–54), Senior (55+) |
| `Education` | Text | 5 categories retained as-is |
| `Marital_Status` | Text | Cleaned: `Alone`→`Single`; `YOLO`/`Absurd`→`Other` |
| `Income` | Currency | 25 rows nulled (24 zero-fills + 1 outlier) |
| `Income_Bucket` | Text | Low (<$35k), Mid ($35k–$68k), High (>$68k), Unknown |
| `Kidhome`, `Teenhome` | Integer | Retained for granular analysis |
| `Has_Children` | Boolean | `Kidhome + Teenhome > 0` |
| `Country` | Text | 8 countries |
| `Enrollment_Date` | Date | From raw `Dt_Customer` |
| `Recency` | Integer | Days since last purchase (snapshot value) |
| `Complain` | Boolean | Complaint in last 2 years |
| `NumWebVisitsMonth` | Integer | Web visits in the last month |
| `NumDealsPurchases` | Integer | Kept in `Dim_Customer` as a pricing-mode attribute, not treated as a fourth channel |

### `Fact_Spend`

Unpivoted from the 6 `Mnt*` columns. One row per customer × product category.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (FK → Dim_Customer) | |
| `Category` | Text | Wines, Fruits, Meat, Fish, Sweets, Gold |
| `Amount` | Currency | Spend in the last 2 years |

### `Fact_Channel`

Unpivoted from the 3 real channel columns (`NumWebPurchases`, `NumCatalogPurchases`, `NumStorePurchases`). One row per customer × channel.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (FK → Dim_Customer) | |
| `Channel` | Text | Web, Catalogue, Store |
| `NumPurchases` | Integer | Purchase count |

`NumDealsPurchases` is deliberately excluded — a "deal" is a pricing mode, not a channel, and mixing them would double-count purchases.

### `Fact_Campaign`

Unpivoted from the 5 `AcceptedCmpN` columns plus `Response`. One row per customer × campaign.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (FK → Dim_Customer) | |
| `CampaignID` | Integer (FK → Dim_Campaign) | 1–6 |
| `Accepted` | Boolean | 1 if the customer accepted the campaign offer |

### `Dim_Campaign`

| CampaignID | Campaign_Name | Is_Latest |
|---|---|---|
| 1 | Campaign 1 | 0 |
| 2 | Campaign 2 | 0 |
| 3 | Campaign 3 | 0 |
| 4 | Campaign 4 | 0 |
| 5 | Campaign 5 | 0 |
| 6 | Last Campaign | 1 |

`Is_Latest` tags the most recent campaign so DAX measures on the Executive Summary page can filter on the dimension rather than hardcoding a column name.

### `Dim_Date`

Standard date table linked to `Enrollment_Date` only. Minimal scope — this project is not a time-series analysis (see Limitations). Used for enrollment cohort views on Page 2 if needed.

## Design decisions and their rationale

- **Buckets pre-built in Power Query, not calculated in DAX.** Bucket definitions are modeling decisions, not runtime logic. Centralizing them in the ETL layer means one place to change definitions.
- **Income and age bucket thresholds are data-driven** (based on dataset quartiles), not arbitrary fixed dollar amounts. This makes each bucket meaningful relative to *this* customer base.
- **`Age` calculated as `2014 - Year_Birth`**, not `TODAY() - Year_Birth`. The dataset is a fixed historical snapshot; using `TODAY()` would silently age every customer by 10+ years and corrupt every age-based visual.
- **`NumDealsPurchases` stays in `Dim_Customer`** rather than joining the channel fact. Deals are a pricing mode; treating them as a fourth channel would double-count purchases already captured in Web/Catalogue/Store.
- **`Is_Latest` flag in `Dim_Campaign`** rather than special-casing the `Response` column in DAX. Keeps campaign logic uniform.