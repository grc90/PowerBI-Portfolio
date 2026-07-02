# Business Questions

The client brief defined three strategic objectives: customer segmentation, campaign and channel performance, and product performance. These were translated into a concrete, measurable set of questions, each mapped to a specific dashboard page.

Questions that were too vague to build a visual against ("explore patterns") were discarded. One requested topic — seasonal demand trends — was found to be unanswerable with the available data and is documented as a limitation rather than approximated with an unsuitable proxy.

## Page 1 — Executive Summary

High-level KPIs for stakeholders who need a 30-second read of the business:

- Total customers, total revenue (sum across all product categories), overall campaign response rate, average customer income and age
- Comparison of the most recent campaign's response rate against the historical average

## Page 2 — Customer Segmentation

- Who are our customers, demographically? (age, education, marital status, income, country)
- How does spend behavior differ across income brackets and life-stage (presence of kids/teens at home)?
- Which segments are most responsive to marketing campaigns?

## Page 3 — Campaign & Channel Performance

- Which campaign has the highest **acceptance rate** — a re-examination of the prior analysis, which compared raw response counts rather than rates
- Which channel (Web / Catalogue / Store) drives the most purchases per customer — re-verifying the claim that Catalogue is underperforming
- Are discount-driven shoppers (`NumDealsPurchases`) a distinct behavioral segment from full-price channel buyers?

## Page 4 — Product Performance

- Which product categories drive the most total and average spend?
- Do high spenders concentrate their spend in specific categories, or spread it broadly?
- Does product preference vary by customer segment (income tier, presence of children)?

**Documented limitation:** the brief also requested seasonal demand analysis, which cannot be supported by this dataset — spend columns are 2-year cumulative totals with no per-transaction date field. See [`06_limitations.md`](06_limitations.md).