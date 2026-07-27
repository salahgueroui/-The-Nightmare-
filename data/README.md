# Data

This project is built from a single Excel workbook, `dataset.xlsx`, containing one sheet per source table. Each sheet is imported as a separate query in the **01_stage** layer before being transformed into the final dimension and fact tables — see [`docs/build-process.md`](../docs/build-process.md) for the full transformation log.

## Source Sheets

| Sheet | Used to build |
|---|---|
| `CUST_MASTER` | `dim_customer` |
| `customer_contacts` | `dim_customer` (deduplicated on `IsPrimary`) |
| `user_details` | `dim_customer` |
| `Address` | `dim_customer` |
| `cities` | `dim_customer`, `dim_geo` |
| `regions` | `dim_customer` |
| `products` | `dim_product` |
| `subcategories` | `dim_product` |
| `ORDERS_2025`, `ORDERS_2026` | Appended into `orders`, which feeds `dim_orders_flag` and `fact_sales` |
| `order_line_items` | `fact_sales` |
| `inventory` | `fact_inventory` |
| `CAMPAIGN_LOG` | `dim_campaign`, `fact_campaign_spend` |
| `campaign_skus` | `fact_promotion_coverage` |
| `shipments` | `fact_order_process` |
| `INVOICES`, `invoice_lines` | `fact_order_process` |
| `payments` | `fact_order_process` |
| `sales_targets` | `fact_sales_targets` |
| `security` | Row-level security (user email → region) |
| `exchange_rates` | Loaded to staging but not used in the final model |
| `dim_order`, `Sheet1` | Early/intermediate imports, removed during the build |

The `channels` table used to resolve channel names in `dim_orders_flag` is **not** a sheet in this workbook — it's a small reference list (channel code → channel name) entered directly in Power Query.

## Notes on the Data

- This is a **fictional, practice dataset** — company names, customer names, and email addresses (e.g. `nora.adel@arka.com`) are not real and are used only to demonstrate the model and row-level security.
- If you're recreating this project with your own data, the exact sheet names don't need to match — what matters is following the same staging → dimension/fact transformation pattern described in [`docs/build-process.md`](../docs/build-process.md).
