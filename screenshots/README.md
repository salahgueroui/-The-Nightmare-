# Screenshots

All 30 screenshots referenced in the README and `docs/` are in this folder, taken directly from Power Query and the Model view. Most are embedded inline in [`docs/build-process.md`](../docs/build-process.md) (the build narrative) and [`docs/data-model.md`](../docs/data-model.md) (the reference schema).

## Model & structure

| File | Used in |
|---|---|
| `stage_folder.jpg` | README, data-model.md, build-process.md — the 22 staging queries |
| `old_data_model.jpg` | data-model.md — the raw, unmodeled state before staging |
| `new_data_model.jpg` | README, data-model.md, build-process.md — the final star schema |

## Dimension tables

Each has a data-preview screenshot (embedded in `build-process.md`) and a `_columns.jpg` field-list screenshot (kept here as a clean column reference, not embedded inline):

`dim_customer` · `dim_product` · `dim_orders_flag` · `dim_geo` · `dim_campaign` · `dim_date`

## Fact tables

Same pattern — data preview embedded, `_columns.jpg` kept for reference:

`fact_sales` · `fact_inventory` · `fact_campaign_spend` · `fact_promotion_coverage` · `fact_order_process` · `fact_sales_targets`

## Security & measures

| File | Used in |
|---|---|
| `security.jpg` | build-process.md — the `security` table |
| `security_columns.jpg` | Field reference |
| `manage_security_roles.jpg` | README, dax-measures.md, build-process.md — the `regional access` RLS role and its DAX filter |
| `_measures.jpg` | dax-measures.md, build-process.md — the `_measures` table |

## Optional additions

Not required, but would round things out if you want to add them later:

- [ ] A screenshot of an actual report/dashboard page, if one exists — the docs currently cover the data model and ETL in depth but don't show the visual report canvas itself.
- [ ] A "View As" screenshot showing the report filtered down to a single region, to visually prove the RLS role works end-to-end (currently only the role *definition* is shown, not its effect).
