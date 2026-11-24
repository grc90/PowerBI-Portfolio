# Data Dictionary

Field-by-field reference for the model. The reasoning behind the design is in [`05_data_model.md`](05_data_model.md), and the cleaning decisions are in [`03_data_quality.md`](03_data_quality.md).

## Dim_Customer (2,240 rows, 1 per customer)

| Column | Type | Source | Description |
|---|---|---|---|
| CustomerID | Integer | `ID` | Customer key |
| Year_Birth | Integer | `Year_Birth` | Birth year; null for 3 implausible values (≤1900) |
| Age | Integer | calc | `2014 - Year_Birth` (snapshot year) |
| Age_Bucket | Text | calc | Young Adult (≤34), Adult (35–54), Senior (55+), Unknown |
| Education | Text | `Education` | Basic, 2n Cycle, Graduation, Master, PhD |
| Marital_Status | Text | `Marital_Status` | Married, Together, Single, Divorced, Widow, Other |
| Income | Decimal | `Income` | Yearly household income; null for 0 and 666,666 |
| Income_Bucket | Text | calc | Low (<$35k), Mid ($35k–$68k), High (>$68k), Unknown |
| Kidhome | Integer | `Kidhome` | Number of small children at home |
| Teenhome | Integer | `Teenhome` | Number of teenagers at home |
| Has_Children | Text | calc | Yes if Kidhome + Teenhome > 0 |
| Country | Text | `Country` | 8 countries; Spain ~49%, Mexico only 3 customers |
| Enrollment_Date | Date | `Dt_Customer` | Date the customer joined |
| Recency | Integer | `Recency` | Days since last purchase |
| Complain | Integer | `Complain` | 1 if the customer complained in the last 2 years |
| NumDealsPurchases | Integer | `NumDealsPurchases` | Purchases made with a discount |
| Deal_Shopper | Text | calc | Deal-driven (deals ≥ 1/3 of channel purchases), Full price, No purchases |
| Spend_Tier | Text | calc | Top 25% (total spend > $1,045) or Rest |
| NumWebVisitsMonth | Integer | `NumWebVisitsMonth` | Web visits last month |
| Age_Bucket_Sort, Income_Bucket_Sort | Integer | calc | Hidden; sort order for the buckets |

## Fact_Spend (13,440 rows: customer × category)

| Column | Type | Description |
|---|---|---|
| CustomerID | Integer | FK to Dim_Customer |
| Category | Text | Wines, Fruits, Meat, Fish, Sweets, Gold (from the `Mnt*` columns) |
| Amount | Decimal | Spend in the last 2 years |

## Fact_Channel (6,720 rows: customer × channel)

| Column | Type | Description |
|---|---|---|
| CustomerID | Integer | FK to Dim_Customer |
| Channel | Text | Web, Catalogue, Store |
| NumPurchases | Integer | Number of purchases through that channel |

## Fact_Campaign (13,440 rows: customer × campaign)

| Column | Type | Description |
|---|---|---|
| CustomerID | Integer | FK to Dim_Customer |
| CampaignID | Integer | FK to Dim_Campaign (1–5 = `AcceptedCmp1–5`, 6 = `Response`) |
| Accepted | Integer | 1 if the customer accepted the offer |

## Dim_Campaign (6 rows)

| Column | Type | Description |
|---|---|---|
| CampaignID | Integer | Campaign key |
| Campaign_Name | Text | Campaign 1–5, Last Campaign |
| Is_Latest | Integer | 1 for the last campaign |

## Dim_Date (1,096 rows, 2012–2014)

| Column | Type | Description |
|---|---|---|
| Date | Date | Key, linked to Dim_Customer[Enrollment_Date] |
| Year, MonthNum, MonthName, YearMonth, Quarter | — | Standard calendar attributes |

This table only filters customers by the date they enrolled (the "Enrollment year" slicer). It is **not** a sales date, and it shouldn't be read as one.

## Helper tables

- **_Measures**: holds every measure (see [`07_dax_measures.md`](07_dax_measures.md)), plus one hidden placeholder column.
- **_Chart_ResponseComparison**: two rows (Latest Campaign / Historical Average) that serve as the axis for the comparison chart on the Executive Summary.
