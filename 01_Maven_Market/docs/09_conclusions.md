# Conclusions

Every number here comes from the Power BI model with no filters applied: 2,240 customers and two years of cumulative spend. For what this data can't tell us, see [`08_limitations.md`](08_limitations.md).

## Campaigns

| Campaign | Accepted | Response rate |
|---|---|---|
| Campaign 1 | 144 | 6.43% |
| Campaign 2 | 30 | 1.34% |
| Campaign 3 | 163 | 7.28% |
| Campaign 4 | 167 | 7.46% |
| Campaign 5 | 163 | 7.28% |
| Last Campaign | 334 | 14.91% |

- **The last campaign wins, and it isn't close.** 14.91% against a historical average of 5.96%, roughly 2.5×.
- **Campaign 2 is the one I'd want to dig into.** At 1.34% it sits well below everything else. I can't tell from this data what went wrong, but I wouldn't reuse its format until someone figures that out.
- **Rates vs counts.** Part of my goal was to redo the earlier analysis with response rates instead of raw counts. It turns out every campaign went to all 2,240 customers, so both methods produce the same ranking. I still think rates are the right default. They just don't change the answer this time.
- **Reach is the bigger problem.** Only 27.19% of customers accepted any of the six campaigns. Roughly 3 out of 4 never responded to anything.

## Who responds

| Segment | Response rate |
|---|---|
| High income (>$68k) | 16.55% |
| Mid income | 5.05% |
| Low income (<$35k) | 2.94% |
| No children at home | 14.42% |
| With children at home | 4.67% |

- **Income matters more than anything else I looked at.** High-income customers respond about 5.6× more often than low-income ones.
- **Children at home cut response by about 3×.** My first guess was that this was just income in disguise, but the gap survives inside the high (19.77% vs 9.38%) and mid (8.10% vs 4.61%) brackets. In the low bracket it vanishes: everyone sits around 2–3%.
- The best cell in the age × income matrix is young, high-income adults at 22.36%. It's a small group, so I'd treat it as a lead, not a finding.

## Channels

- **Store comes first** with 5.79 purchases per customer, followed by Web (4.08) and Catalogue (2.66).
- **"Catalogue is underperforming" is true on volume, and that's where I'd stop.** Catalogue accounts for 23.4% of purchases among full-price shoppers but only 12.7% among deal-driven ones. It's where the higher-value customers buy, which isn't what a dying channel looks like. Whether it underperforms in revenue is a separate question, and without revenue per channel this dataset can't answer it.

## Deal-driven shoppers

Customers who buy at least 1 in 3 items on a deal (764 of them) behave like a separate segment:

| | Deal-driven | Full price |
|---|---|---|
| Customers | 764 | 1,470 |
| Avg income | $36,531 | $59,929 |
| Avg spend per customer | $183 | $828 |
| Purchases per customer | 7.4 | 15.3 |
| Response rate | 3.95% | 9.30% |
| With children | 96% | 59% |

Nearly all of them have children at home, and they buy mostly in Store and on the Web. Sending them the same offer as everyone else looks like a mistake to me. What gets their attention is price.

## Products

- **Wines (50.17%) and Meat (27.56%) account for about 78% of total spend.** The other four categories share what's left.
- **The top 25% of spenders bring in 61.5% of total spend.** Their basket leans more toward Meat (30.65% vs 22.62% for everyone else) and less toward Gold (5.30% vs 10.42%).
- **Low-income customers buy something quite different.** Wines falls to 26.6% of their spend and Gold rises to 20.2%.
- **Customers without children spend about 2.7× more** ($1,106 vs $407 on average). The widest gap is in Meat: $373 vs $85.

## What I'd do with this

1. **Start the next campaign from whatever the last one did.** Target high-income customers without children first. They respond the most and they spend the most.
2. **Give deal-driven shoppers their own campaign** built around price, instead of folding them into the general one.
3. **Don't cut Catalogue on volume alone.** Get revenue per channel first, then decide.
4. **Put most of the promotional effort on Wines and Meat**, since that's where the value is. The exception is Gold, which matters a lot more to lower-income customers than its overall share suggests.
