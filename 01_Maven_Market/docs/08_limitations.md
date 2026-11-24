# Known Limitations

Every real dataset has a gap between what stakeholders want to know and what the data can actually tell them. I'd rather write those gaps down than quietly work around them.

## Seasonal demand analysis isn't possible

The brief asks for "seasonal demand trends" under the product performance objective. This dataset can't deliver that.

**Why:** the six product spend columns (`MntWines`, `MntFruits`, `MntMeatProducts`, `MntFishProducts`, `MntSweetProducts`, `MntGoldProds`) are **cumulative totals over a 2-year window**, not individual transactions. Nothing records when a purchase happened. The only date in the dataset is `Dt_Customer`, which is when the customer signed up, not when they bought anything.

**Workarounds I considered and rejected:**

- **Using `Dt_Customer` as a proxy.** That would present signup cohorts as demand seasonality. It isn't a rough approximation, it's measuring a different thing.
- **Using `Recency` as a proxy.** `Recency` is a single snapshot of days since the last purchase. You can't turn one snapshot into a time series.

**What I did instead:** the Product Performance page reports category totals and segment preferences, and the dashboard states this limitation openly instead of covering it with a misleading chart.

## Country comparisons are skewed by sample size

Customers are very unevenly spread across countries:

- Spain: ~1,095 customers (~49%)
- Saudi Arabia, Canada, Australia, India, Germany, US: smaller samples
- Mexico: 3 customers

Filtering by country works technically, but comparisons involving the small countries (Mexico especially) aren't reliable. Wherever the dashboard breaks results down by country, small samples are either excluded or clearly labeled. Three customers aren't enough to support any conclusion.

## No product detail within categories

Products are reported at the **category** level (Wines, Fruits, Meat, Fish, Sweets, Gold). There's no SKU or brand breakdown inside a category, so anything more granular would need a different dataset.

## No link between campaigns and purchases

The data shows **whether** a customer accepted a campaign and **how much** they spent overall, but it doesn't connect a specific campaign to a specific purchase. Claiming that a campaign generated a certain amount of revenue would mean assuming an incrementality the data can't back up. That's why I measure campaign performance by acceptance rate, not by attributed revenue.

## It's a historical snapshot, not a live feed

All the analysis is tied to the 2012–2014 period the data covers. `Age` is calculated as `2014 - Year_Birth` rather than `TODAY() - Year_Birth`, so every visual stays consistent with that period. The project doesn't forecast or predict anything beyond the observed window.
