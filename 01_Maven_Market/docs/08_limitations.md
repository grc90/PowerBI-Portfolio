# Known Limitations

Every real dataset has gaps between what stakeholders want and what the data can answer. Documenting those gaps transparently is more useful than working around them silently.

## Seasonal demand analysis is not supported

The business brief requests "seasonal demand trends" as part of the product performance objective. This dataset cannot support that analysis.

**Reason:** the six product spend columns (`MntWines`, `MntFruits`, `MntMeatProducts`, `MntFishProducts`, `MntSweetProducts`, `MntGoldProds`) are **cumulative totals over a 2-year window**, not transaction-level records. There is no field describing when any individual purchase took place. The only date field in the dataset is `Dt_Customer` (customer enrollment date), which records when the customer joined — not when they bought anything.

**Options considered and rejected:**

- **Using `Dt_Customer` as a proxy** — would mislabel signup cohort trends as demand seasonality. This is a category error, not an approximation.
- **Using `Recency` as a proxy** — `Recency` is a snapshot count of days since last purchase, taken at a single point in time. It cannot be plotted as a time series.

**Correct handling:** the Product Performance page reports category-level totals and segment-level preferences, and this limitation is stated explicitly rather than approximated with a misleading proxy.

## Country-level analysis is affected by sample imbalance

Country distribution in the dataset is skewed:

- Spain: ~1,095 customers (~49%)
- Saudi Arabia, Canada, Australia, India, Germany, US: smaller samples
- Mexico: 3 customers

Country-level slicing works technically, but comparisons involving small-sample countries (especially Mexico) are statistically unreliable. Where country breakdowns appear on the dashboard, the visual design will either exclude small-sample countries or annotate them explicitly. No conclusion should be drawn from three customers.

## No product-level data within categories

Product performance is reported at the **category** level (Wines, Fruits, Meat, Fish, Sweets, Gold). There is no SKU-level or brand-level breakdown within a category. Product recommendations at a finer grain would require a different dataset.

## No causal attribution between campaigns and purchases

The dataset records **whether** a customer accepted a campaign and **how much** they spent overall, but there is no linkage between a specific campaign and a specific purchase. Statements about campaign-driven revenue would require assumptions about incrementality that this dataset does not support. Campaign performance is therefore measured by acceptance rate, not by attributed revenue.

## The dataset is a historical snapshot, not a live feed

All analysis is anchored to the dataset's 2012–2014 timeframe. `Age` is calculated as `2014 - Year_Birth`, not `TODAY() - Year_Birth`, to keep every visual internally consistent with the period the data represents. This project does not produce forecasts or predictions extending beyond the observed window.