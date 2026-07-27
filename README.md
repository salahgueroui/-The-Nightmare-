# 🌙 The Nightmare — Retail Sales & Marketing Data Warehouse

A complete dimensional data warehouse built from the ground up in Power BI — from raw Excel extracts to a governed, security-enforced star schema — covering sales, inventory, marketing campaigns, and order fulfillment for a fictional retail business.

This isn't a single-table "connect and chart" report. It's an end-to-end Kimball-style build: staging raw data, resolving duplicate and inconsistent records, designing dimension and fact tables, wiring up a star schema, writing business measures in DAX, and enforcing row-level security by region. True to its name, getting there took over 50 documented transformation steps and more than one duplicate-key mystery — the full build log is in [`docs/build-process.md`](docs/build-process.md).

![Final star schema](screenshots/new_data_model.jpg)

## Table of Contents

- [Overview](#overview)
- [Highlights](#highlights)
- [Repository Structure](#repository-structure)
- [Data Model](#data-model)
- [Business Measures (DAX)](#business-measures-dax)
- [Row-Level Security](#row-level-security)
- [Build Process](#build-process)
- [Screenshots](#screenshots)
- [Data Source](#data-source)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Acknowledgments](#acknowledgments)
- [License](#license)

## Overview

The project simulates a retail company's data warehouse, bringing together four business processes into a single analytical model:

- **Sales** — customer orders, order lines, channels, and fulfillment status
- **Inventory** — product stock levels over time
- **Marketing** — campaigns, campaign spend, and which products each campaign promoted
- **Order Fulfillment** — the order → ship → invoice → payment lifecycle

All of it starts from a raw Excel workbook and is transformed entirely in Power Query into a clean star schema, ready for reporting, secured with row-level security so each regional user only sees their own data.

## Highlights

- **Full ETL pipeline in Power Query** — 22 raw staging queries are shaped into 6 clean, `snake_case` dimension tables and 6 fact tables, all built as query references (not copies) so the staging layer stays the single source of truth.
- **Kimball star schema** — 4 business processes (sales, inventory, marketing, order fulfillment) sharing a common set of conformed dimensions.
- **Role-playing dimension** — a single `dim_geo` table is reused for both *ship-to* and *bill-to* city lookups on `fact_sales`.
- **A dedicated bridge table** — `fact_promotion_coverage` cleanly answers "which products did this campaign promote?" without mixing that question into the campaign-spend metrics.
- **Real data-quality troubleshooting** — duplicate customer records, duplicate product names mapping to different keys, and mismatched text casing between tables were all identified and resolved during the build (see [Build Process](#build-process)).
- **Row-level security by region** — a dedicated `regional access` role restricts each user to their assigned region using `USERPRINCIPALNAME()`.
- **Documented DAX measure layer** — core KPIs (sales, orders, active customers, order-to-pay cycle time) are centralized in a dedicated `_measures` table.
- **Organized project structure** — Power Query queries are grouped into `01_stage`, `02_Dimensions`, `03_Facts`, and `04_Support` folders for maintainability.

## Repository Structure

```
The Nightmare/
├── The Nightmare.pbix          # Main Power BI report file
├── data/
│   ├── raw/
│   │   └── dataset.xlsx        # Source Excel workbook
│   └── README.md               # Description of source sheets
├── docs/
│   ├── build-process.md        # Full step-by-step ETL & modeling log
│   ├── data-model.md           # Star schema, table grains, and relationships
│   └── dax-measures.md         # DAX measure reference
├── screenshots/                # Power Query, model, and security screenshots
│   └── README.md
├── .gitignore
├── LICENSE
└── README.md
```

## Data Model

The model follows a star schema: fact tables at the center hold transactional/measurable data, surrounded by dimension tables that describe *who*, *what*, *where*, and *when*.

| Layer | Table | Description |
|---|---|---|
| Dimension | `dim_customer` | Customer master data, merged from customer, contact, user, and address/city sources |
| Dimension | `dim_product` | Product catalog with standardized category/subcategory names |
| Dimension | `dim_orders_flag` | Order channel, status, and priority attributes |
| Dimension | `dim_geo` | City lookup, reused for both ship-to and bill-to locations |
| Dimension | `dim_campaign` | Deduplicated marketing campaign list |
| Dimension | `dim_date` | Standard date table, related to every fact table |
| Fact | `fact_sales` | Order line-item grain — the core sales fact table |
| Fact | `fact_inventory` | Product stock levels by month, unpivoted from a wide source layout |
| Fact | `fact_campaign_spend` | Daily campaign performance: impressions, clicks, spend |
| Fact | `fact_promotion_coverage` | Bridge table mapping campaigns to the products they promoted |
| Fact | `fact_order_process` | Order lifecycle dates (order, ship, invoice, payment), used to calculate order-to-pay cycle time |
| Fact | `fact_sales_targets` | Monthly sales targets for performance tracking |

Full grain definitions, keys, and the entity-relationship diagram are in [`docs/data-model.md`](docs/data-model.md).

## Business Measures (DAX)

Centralized in a dedicated `_measures` table:

| Measure | DAX |
|---|---|
| Total Sales | `SUM(fact_sales[line_total])` |
| Total Orders | `DISTINCTCOUNT(fact_sales[order_id])` |
| Total Active Customers | `DISTINCTCOUNT(fact_sales[customer_id])` |
| Base Total Customers | `COUNT(dim_customer[customer_id])` |
| Average Order to Pay | `AVERAGE(fact_order_process[order_to_pay])` |

Details and business context for each measure are in [`docs/dax-measures.md`](docs/dax-measures.md).

## Row-Level Security

A **Row-Level Security (RLS)** role named `regional access` restricts each user to the data for their assigned region. The role filters `dim_customer` using:

```dax
[region_name] = LOOKUPVALUE(security[region], security[user_email], USERPRINCIPALNAME())
```

This looks up the signed-in user's email (via `USERPRINCIPALNAME()`) in a `security` table (`user_email` → `region`), then filters `dim_customer` — and everything related to it — down to just that region. The role was validated in Power BI Desktop by viewing the report as `nora.adel@arka.com`, who is assigned to North America.

![Manage security roles — regional access filter](screenshots/manage_security_roles.jpg)

## Build Process

The model was built in four phases:

1. **Data Import & Project Setup** — importing the source workbook and organizing queries into staging/dimension/fact/support folders.
2. **Building the Customer & Product Dimensions** — merging, deduplicating, and standardizing `dim_customer` and `dim_product`.
3. **Building the Fact Tables & Star Schema** — constructing all six fact tables, resolving duplicate-match issues, and wiring up the relationships.
4. **Polishing the Data Model** — adding the date dimension, centralizing DAX measures, and implementing row-level security.

The complete, detailed log of every transformation decision (including the data-quality issues found and how they were resolved), illustrated with a screenshot of every table, is documented in [`docs/build-process.md`](docs/build-process.md).

## Screenshots

![Power Query staging folder](screenshots/stage_folder.jpg)
*The Power Query editor showing the organized query structure: 22 staging queries feeding 5 dimensions and 6 fact tables (`dim_date` was added later as a calculated table).*

The full set of screenshots — every dimension and fact table, the before/after model views, and the RLS role manager — is embedded throughout [`docs/build-process.md`](docs/build-process.md) and [`docs/data-model.md`](docs/data-model.md). See [`screenshots/README.md`](screenshots/README.md) for an index.

## Data Source

The source data is a fictional retail dataset (`data/raw/dataset.xlsx`), used purely for practice/portfolio purposes — no real customer, employee, or business data is included. See [`data/README.md`](data/README.md) for the full list of sheets and what each one feeds into.

## Tech Stack

- **Power BI Desktop** — data modeling and reporting
- **Power Query (M)** — data extraction and transformation
- **DAX** — business measures and row-level security
- **Excel** — source data format

## Getting Started

1. Clone this repository.
2. Open `The Nightmare.pbix` in Power BI Desktop — the source workbook is already included at `data/raw/dataset.xlsx`.
3. If prompted, update the data source path (**Transform Data → Data Source Settings**) to point to your local copy of `data/raw/dataset.xlsx`.
4. Click **Refresh** to load the data.

## Acknowledgments

The ETL and data warehousing methodology used in this project was learned from this tutorial: [youtu.be/pQSMbRA3O6g](https://youtu.be/pQSMbRA3O6g). The data model, transformations, and measures documented here reflect my own implementation and decisions built on top of that foundation.

## License

This project is licensed under the [MIT License](LICENSE).
