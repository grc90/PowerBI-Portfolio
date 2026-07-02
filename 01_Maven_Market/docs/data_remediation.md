## Data Quality Remediation

Before modeling, the raw dataset (2,240 rows, `marketing_data` sheet) was audited for nulls, outliers, and inconsistent categorical values. Each issue was resolved with a documented, minimal-intervention decision — no rows were dropped, and customers were retained wherever a single corrupted field could be isolated and nulled instead.

| Issue | Rows Affected | Decision | Rationale |
|---|---|---|---|
| `Income` = 0 (originally missing, silently zero-filled in source data) | 24 | Recoded to `null`; excluded from income-based measures | A $0 income is not a real value and would distort every income aggregation. Nulling allows it to be automatically excluded from averages while preserving the customer's other data (spend, campaigns, channel activity). |
| `Income` = $666,666 | 1 | Nulled income only; customer retained | Repeating-digit value is a near-certain data entry artifact rather than a genuine household income. |
| `Year_Birth` < 1900 (implied age 126+) | 3 | Nulled `Year_Birth` only; rows retained | Biologically implausible values, most likely entry errors. Retaining the rows preserves valid spend/campaign/channel data. |
| `Marital_Status` = "Alone" | 3 | Recoded → `Single` | Functionally synonymous category. |
| `Marital_Status` = "YOLO" / "Absurd" | 4 | Recoded → `Other` | Non-standard values (likely test/joke entries), bucketed rather than dropped. |
| `Income` column header (trailing whitespace) | — | Trimmed | Prevents broken references in Power Query / DAX. |
| Customers with zero purchases across all channels | 6 | Retained, no changes | Represents a legitimate "enrolled but inactive" segment rather than a data error. |
| Country sample imbalance (e.g., Mexico n=3) | — | Retained; flagged as a visualization constraint | Not a data quality issue — noted for dashboard design to avoid misleading conclusions from very small samples. |

**Result:** 2,240 rows retained (0 dropped); targeted null/recode applied to a small number of cells across `Income`, `Year_Birth`, and `Marital_Status`. Because this introduces nulls rather than substitute values, all downstream DAX measures use `AVERAGE`/`DIVIDE`-based logic that correctly ignores blanks rather than treating them as zero.