# Data Model

The raw data comes as one flat table (2,240 rows × 28 columns). I didn't want to model on top of that wide structure, so I reshaped it into a **star/constellation schema**: one customer dimension, three fact tables (spend, channel, campaign), a campaign dimension and a small date dimension.

## Why not keep it flat?

Three groups of columns repeat the same pattern: 6 product spend columns, 3 purchase channel columns and 6 campaign response columns. Almost every question on the Campaign/Channel and Product pages means comparing *across* one of those groups.

In wide format, that means one measure per column: six measures (`[Total Wines]`, `[Total Fruits]`…) just to build one bar chart. In long format, a single measure per fact, sliced by category, channel or campaign, gives you the same visual and works with every other filter without extra effort.

So the business questions drove the restructuring. I wasn't building a star schema just because it's the textbook answer.

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

Every relationship is one-to-many and single-direction (Dim → Fact). No bidirectional filtering.

## Tables

### `Dim_Customer`

One row per customer, with every demographic and behavioral attribute used for segmentation.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (PK) | From raw `ID` |
| `Year_Birth` | Integer | Nulled for 3 rows with implausible values (see Data Quality) |
| `Age` | Integer (calc) | `2014 - Year_Birth`. Fixed to the snapshot year, not `TODAY()` |
| `Age_Bucket` | Text | Young Adult (≤34), Adult (35–54), Senior (55+), Unknown |
| `Education` | Text | 5 categories, kept as they are |
| `Marital_Status` | Text | Cleaned: `Alone`→`Single`; `YOLO`/`Absurd`→`Other` |
| `Income` | Currency | 25 rows nulled (24 zero-fills + 1 outlier) |
| `Income_Bucket` | Text | Low (<$35k), Mid ($35k–$68k), High (>$68k), Unknown |
| `Kidhome`, `Teenhome` | Integer | Kept for more granular analysis |
| `Has_Children` | Text | "Yes" if `Kidhome + Teenhome > 0`, otherwise "No" |
| `Country` | Text | 8 countries |
| `Enrollment_Date` | Date | From raw `Dt_Customer` |
| `Recency` | Integer | Days since last purchase (snapshot value) |
| `Complain` | Integer | 1 if the customer complained in the last 2 years |
| `NumWebVisitsMonth` | Integer | Web visits in the last month |
| `NumDealsPurchases` | Integer | Kept here as a pricing attribute, not treated as a fourth channel |
| `Deal_Shopper` | Text | Deal-driven (deals ≥ 1/3 of channel purchases, ~p75), Full price, No purchases (6 customers) |
| `Spend_Tier` | Text | Top 25% (total spend > $1,045, the p75) vs Rest |
| `Age_Bucket_Sort`, `Income_Bucket_Sort` | Integer (hidden) | Sort-by columns so buckets appear in logical order, not alphabetical |

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

I left `NumDealsPurchases` out on purpose. A deal is a way of paying, not a place to buy, and mixing the two would count the same purchases twice.

### `Fact_Campaign`

Unpivoted from the 5 `AcceptedCmpN` columns plus `Response`. One row per customer × campaign.

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | Integer (FK → Dim_Customer) | |
| `CampaignID` | Integer (FK → Dim_Campaign) | 1–6 |
| `Accepted` | Integer | 1 if the customer accepted the campaign offer |

### `Dim_Campaign`

| CampaignID | Campaign_Name | Is_Latest |
|---|---|---|
| 1 | Campaign 1 | 0 |
| 2 | Campaign 2 | 0 |
| 3 | Campaign 3 | 0 |
| 4 | Campaign 4 | 0 |
| 5 | Campaign 5 | 0 |
| 6 | Last Campaign | 1 |

`Is_Latest` marks the most recent campaign, so the Executive Summary measures filter on the dimension instead of hardcoding a column name.

### `Dim_Date`

A standard date table, linked to `Enrollment_Date` only. I kept it small on purpose: this isn't a time-series analysis (see Limitations). It's there for enrollment cohort views on Page 2, if they're needed.

## Design decisions

- **Buckets are built in Power Query, not in DAX.** How you define a bucket is a modeling decision, not something to calculate at runtime. Keeping those definitions in the ETL layer means there's only one place to change them.
- **Income and age thresholds come from the data** (dataset quartiles), not from round dollar amounts picked in advance. That way each bucket means something relative to *this* customer base.
- **`Age` is `2014 - Year_Birth`, not `TODAY() - Year_Birth`.** The data is a fixed historical snapshot. Using `TODAY()` would quietly add 10+ years to every customer and throw off every age-based visual.
- **`NumDealsPurchases` stays in `Dim_Customer`** instead of going into the channel fact. Deals describe how someone pays. Treating them as a fourth channel would double-count purchases already recorded under Web, Catalogue or Store.
- **`Is_Latest` lives in `Dim_Campaign`** instead of special-casing the `Response` column in DAX. That keeps the campaign logic the same for all six campaigns.
