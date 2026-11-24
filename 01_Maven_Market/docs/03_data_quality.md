# Data Quality Remediation

Before building anything, I audited the raw dataset (2,240 rows, `marketing_data` sheet) for nulls, outliers and inconsistent categories. My rule was to change as little as possible. I didn't drop a single row. When only one field in a record was broken, I nulled that field and kept the customer, because the rest of their data (spend, campaigns, channels) was still good.

| Issue | Rows affected | Decision | Why |
|---|---|---|---|
| `Income` = 0 (originally missing, quietly filled with zeros in the source) | 24 | Recoded to `null`; excluded from income measures | Nobody has a household income of $0. Left as zeros, these rows would drag down every income average. As nulls, they drop out of averages on their own, and the customer's other data stays usable. |
| `Income` = $666,666 | 1 | Nulled the income only; customer kept | A repeating-digit number like this is almost certainly a typo, not a real income. |
| `Year_Birth` ≤ 1900 (implied age 126+) | 3 | Nulled `Year_Birth` only; rows kept | Biologically impossible, so most likely entry errors. The rest of each row is still valid. |
| `Marital_Status` = "Alone" | 3 | Recoded → `Single` | Means the same thing. |
| `Marital_Status` = "YOLO" / "Absurd" | 4 | Recoded → `Other` | Look like test or joke entries. Grouped instead of deleted. |
| `Income` column header (trailing space) | — | Trimmed | A stray space in a header breaks references in Power Query and DAX. |
| Customers with zero purchases in every channel | 6 | Kept as is | I read this as a real "enrolled but inactive" group, not a data error. |
| Unbalanced country samples (e.g., Mexico n=3) | — | Kept; flagged as a visualization constraint | Not a quality problem, but it affects how the dashboard should show countries. Nobody should draw conclusions from three customers. |

**Result:** all 2,240 rows kept (0 dropped), with targeted nulls and recodes in a handful of cells across `Income`, `Year_Birth` and `Marital_Status`. Because I used nulls instead of substitute values, every downstream DAX measure relies on `AVERAGE`/`DIVIDE` logic that skips blanks instead of counting them as zero.
