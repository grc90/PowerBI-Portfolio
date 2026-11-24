# Business Questions

The brief set three objectives: customer segmentation, campaign and channel performance, and product performance. Objectives are hard to build visuals against, so I broke each one into specific questions and tied each question to a dashboard page.

I dropped anything too vague to chart ("explore patterns" doesn't tell you what to put on the screen). One requested topic, seasonal demand, turned out to be unanswerable with this data. I documented it as a limitation instead of faking it with a proxy that doesn't fit.

## Page 1 — Executive Summary

The headline KPIs, for stakeholders who have 30 seconds:

- Total customers, total revenue (sum of all product categories), overall campaign response rate, average customer income and age
- The latest campaign's response rate next to the historical average

## Page 2 — Customer Segmentation

- Who are our customers? (age, education, marital status, income, country)
- How does spending change across income brackets and life stage (kids or teens at home)?
- Which segments respond best to campaigns?

## Page 3 — Campaign & Channel Performance

- Which campaign has the highest **acceptance rate**? The previous analysis compared raw response counts, so I wanted to redo it with rates.
- Which channel (Web / Catalogue / Store) drives the most purchases per customer? This also re-checks the earlier claim that Catalogue is underperforming.
- Are discount-driven shoppers (`NumDealsPurchases`) really a different kind of customer from full-price buyers?

## Page 4 — Product Performance

- Which product categories drive the most total and average spend?
- Do high spenders focus on a few categories, or spread their spend around?
- Do product preferences change by segment (income tier, children at home)?

**Documented limitation:** the brief also asked for seasonal demand analysis. This dataset can't support it, because the spend columns are 2-year cumulative totals with no transaction dates. More on that in [`08_limitations.md`](08_limitations.md).
