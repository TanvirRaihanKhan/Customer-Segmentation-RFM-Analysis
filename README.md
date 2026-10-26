# Customer Analytics & RFM Segmentation

> Segmenting about 18,500 customers by how recently, how often and how much they buy, so marketing team can target each group with the right action.
>
> 

![Dashboard overview](dashboard_demo.gif)

**Tools:** Excel · Power BI · DAX 
**Dataset:** AdventureWorksDW (Internet sales, July 2005 to July 2008)
<!-- TODO: optional link to the published report or a short GIF walkthrough -->

---

## Table of contents
1. [Business problem](#business-problem)
2. [What is RFM?](#what-is-rfm)
3. [Dataset](#dataset)
4. [Method](#method)
5. [Customer segments](#customer-segments)
6. [Dashboard walkthrough](#dashboard-walkthrough)
7. [Key insights](#key-insights)
8. [Recommendations](#recommendations)
9. [Repository structure](#repository-structure)
10. [How to run it](#how-to-run-it)
11. [Limitations and next steps](#limitations-and-next-steps)
12. [Contact](#contact)

---

## Business problem
Not every customer is worth the same effort. A shop that sends the same email to everyone wastes money on people who have left and under-serves its best buyers.

This project answers three questions:
- Who are our most valuable customers?
- Which customers are about to leave?
- What should marketing do for each group?

## What is RFM?
RFM scores every customer on three simple behaviours:

| Letter | Meaning | Question it answers | Measured as |
|---|---|---|---|
| **R** | Recency | How recently did they buy? | Days since last order |
| **F** | Frequency | How often do they buy? | Number of distinct orders |
| **M** | Monetary | How much do they spend? | Total sales amount |

Each letter gets a score from 1 (worst) to 5 (best). The three digits together form a code, such as **545**. The code is then mapped to a named segment, such as **Champions**.

## Dataset
AdventureWorksDW is a sample data warehouse from Microsoft for a fictional bike company.

| Table | Rows | Used for |
|---|---|---|
| `FactInternetSales` | 60,398 | Orders, dates and sales amounts |
| `DimCustomer` | 18,484 | Customer names and details |
| `DimGeography` | 655 | Country and city |
| `DimProduct` | 606 | Product details |
| `DimDate` | 2,191 | Calendar |
| `RFM_Table.xlsx` | 125 codes | Maps each RFM code to a segment |

## Method

**1. Calculate R, F and M per customer**
- Recency uses a fixed reference date of **2008-08-01**. That is the day after the last order in the data.
- Frequency counts distinct `SalesOrderNumber` values, not order lines.
- Monetary sums `SalesAmount`.

**2. Score each value from 1 to 5**
Customers are ranked and split into five equal groups (quintiles). For recency, fewer days scores higher.

**3. Build the RFM code and map it to a segment**
`RFM Score = R × 100 + F × 10 + M`, then joined to the segment lookup table.

<details>
<summary>DAX used</summary>

```dax
Recency Value =
VAR RefDate = CALCULATE ( MAX ( FactInternetSales[OrderDate] ), ALL ( FactInternetSales ) ) + 1
RETURN DATEDIFF ( MAX ( FactInternetSales[OrderDate] ), RefDate, DAY )

Frequency Value = DISTINCTCOUNT ( FactInternetSales[SalesOrderNumber] )

Monetary Value = SUM ( FactInternetSales[SalesAmount] )

RFM Table =
ADDCOLUMNS (
    SUMMARIZE ( FactInternetSales, FactInternetSales[CustomerKey] ),
    "Recency Value", [Recency Value],
    "Frequency Value", [Frequency Value],
    "Monetary Value", [Monetary Value]
)

R Score =
VAR Rnk = RANKX ( 'RFM Table', 'RFM Table'[Recency Value], , DESC )
RETURN ROUNDUP ( 5 * Rnk / COUNTROWS ( 'RFM Table' ), 0 )
-- F Score and M Score follow the same pattern with ASC
```
</details>

The SQL version of the same logic is in [`sql/rfm_scores.sql`](sql/rfm_scores.sql).

<!-- TODO: add a screenshot of your data model (Model view) -->
![Data model](images/data_model.png)

## Customer segments

| Segment | Who they are | Marketing action | Customers |
|---|---|---|---|
| Champions | Buy often, recently, and spend the most | Reward with loyalty perks and early access | TODO |
| Loyal | Regular, high-value buyers | Upsell and ask for reviews | TODO |
| Potential Loyalist | Recent buyers with growing frequency | Offer a membership or loyalty programme | TODO |
| Promising | Recent buyers with low spend | Build the relationship with onboarding offers | TODO |
| New Customers | Bought recently for the first time | Welcome series and second-purchase offer | TODO |
| Need Attention | Average on all three | Limited-time offers based on past purchases | TODO |
| About To Sleep | Engagement is dropping | Re-engagement email with a reminder | TODO |
| At Risk | Used to buy often, now quiet | Personal win-back offer | TODO |
| Cannot Lose Them | Big spenders who stopped buying | VIP outreach and strong incentive | TODO |
| Hibernating | Long inactive, low value | Low-cost reactivation campaign | TODO |
| Lost | Inactive, low value on every score | Exclude from paid campaigns | TODO |

<!-- TODO: fill the Customers column from your dashboard -->

## Dashboard walkthrough
<!-- One screenshot and two or three lines per page -->

**Page 1: Overview**
![Overview](images/page1.png)
KPI cards for total customers, revenue and average order value. Customers and revenue by segment.

**Page 2: Segment detail**
![Segment detail](images/page2.png)
Customer table with R, F and M scores. Segments are color-coded from green (best) to red (worst) with conditional formatting.

**Page 3: TODO**
<!-- e.g. geography, product categories, or trends over time -->

## Key insights
<!-- Write 3 to 5 findings with real numbers from your dashboard. Examples of the shape: -->
- TODO: "Champions are X% of customers but bring in Y% of revenue."
- TODO: "Z% of customers placed only one order, so repeat purchase is the biggest growth lever."
- TODO: "Cannot Lose Them holds N customers worth $M in past revenue."
- TODO: a geography or product finding

## Recommendations
<!-- Turn each insight into one action -->
1. TODO
2. TODO
3. TODO

## Repository structure
```
rfm-segmentation/
├── data/
│   ├── AdventureWorksDW.xlsx
│   └── RFM_Table.xlsx
├── sql/
│   └── rfm_scores.sql
├── powerbi/
│   └── RFM_Dashboard.pbix
├── images/
│   └── (screenshots)
└── README.md
```

## How to run it
1. Clone this repository.
2. Open `powerbi/RFM_Dashboard.pbix` in Power BI Desktop.
3. If prompted, point the data source to the files in `data/`.
4. Click **Refresh**.

To run the SQL version, restore AdventureWorksDW in SQL Server and run `sql/rfm_scores.sql`.

## Limitations and next steps
- **Ties in frequency.** About 63% of customers placed only one order, so frequency separates customers less well than recency and spend.
- **Static scores.** The RFM table is a calculated table, so scores don't change with report slicers.
- **Sample data.** Adventure Works is synthetic, so the findings show the method rather than real business results.

Next steps:
- Try K-means clustering on R, F and M and compare it with the rule-based segments.
- Track how customers move between segments over time.
- Add customer lifetime value (CLV).

## Contact
**Tanvir**, MSc Data Science, IT University of Copenhagen
<!-- TODO: LinkedIn · portfolio · email -->
