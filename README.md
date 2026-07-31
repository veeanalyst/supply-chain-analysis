# Supply Chain Analysis: Delivery & Discount Performance

## Overview
SQL analysis of 180,519 order line items from the DataCo Smart Supply Chain
dataset, identifying late-delivery patterns, discounting inefficiencies, and
where operational fixes would have the greatest impact.

## Dataset
[DataCo Smart Supply Chain Dataset](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)
— 180,519 rows, covering orders from January 2015 to January 2018.

Raw CSV is not included in this repo (see `.gitignore`) — download it directly
from Kaggle if you want to reproduce this analysis.

## Tools & Environment
- **Database:** PostgreSQL 18
- **Analysis:** SQL (conditional aggregation, `HAVING`, window functions)
- **Development environment:** VS Code with SQLTools extension
- **OS/Shell:** Windows, PowerShell (used for source file encoding conversion
  — see Data Quality Notes below)

## Questions Answered
1. What's the on-time delivery rate overall, and which regions are worst?
2. Which shipping mode has the highest late-delivery risk?
3. Where is discounting hurting profit margin the most?
4. Which region/shipping-mode combination needs fixing most urgently
   (high volume *and* high late-delivery risk)?
5. Are certain customer segments or markets disproportionately affected
   by late deliveries?

## Key Findings
*(To be completed once all queries are run.)*

- **Q1 — Delivery rate:** [X]% of orders overall were late; worst region was
  [Region] at [Y]%, [Z] points above the company average.
- **Q2 — Shipping mode:** [Mode] had the highest late-delivery rate at [X]%.
- **Q3 — Discounting vs. profit:** [Category] carried the highest average
  discount rate ([X]%) while returning [above/below]-average profit per order.
- **Q4 — Priority fix:** [Region] + [Shipping Mode] combined high order
  volume ([X] orders) with a [Y]% late-delivery rate, representing
  $[Z] in at-risk order value.
- **Q5 — Segment impact:** [Segment/Market] experienced a [X]% late-delivery
  rate, compared to [Y]% company-wide.

## Data Quality Notes
- Several categorical fields (e.g., `order_region`) contained inconsistent
  whitespace (leading/trailing spaces, double-internal-spaces — e.g.
  `"South of  USA "`). Rather than modifying the raw table, a view
  (`orders_clean`) was created to standardize these values for analysis,
  preserving the original source data untouched.
- Source CSV required an encoding conversion (Latin-1 → UTF-8) before
  loading into PostgreSQL. Full technical breakdown, including the exact
  errors encountered and how they were diagnosed, is documented in
  [notes/setup-lessons.md](notes/setup-lessons.md).

## Repo Structure
```
├── README.md
├── .gitignore
├── notes/
│   └── setup-lessons.md      # technical setup + encoding case study
├── views/
│   └── orders_clean.sql      # cleaned view definition
├── queries/
│   ├── 00_create_table.sql
│   ├── 01_delivery_rate_by_region.sql
│   ├── 02_late_risk_by_shipping_mode.sql
│   ├── 03_discount_vs_profit.sql
│   ├── 04_priority_region_shipping_combo.sql
│   └── 05_segment_market_impact.sql
└── findings.md                # detailed write-up per question
```

## How to Reproduce
1. Download the dataset from Kaggle (link above).
2. Convert encoding if needed (see `notes/setup-lessons.md`).
3. Run `queries/00_create_table.sql` to create the schema.
4. Load the CSV via `\copy` (see setup notes for exact command).
5. Run `views/orders_clean.sql` to create the cleaned view.
6. Run each numbered query in `queries/` in order.
