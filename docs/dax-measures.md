# DAX Measures

All business measures are centralized in a dedicated `_measures` table, kept separate from data tables for maintainability (see Phase 4 of [`build-process.md`](build-process.md)).

![The _measures table in the Fields pane](../screenshots/_measures.jpg)

## Core Business Measures

### Total Sales
```dax
total_sales = SUM(fact_sales[line_total])
```
The sum of all order line totals — the headline revenue figure for the sales business process.

### Total Orders
```dax
total_orders = DISTINCTCOUNT(fact_sales[order_id])
```
Counts unique orders rather than order lines, so an order with multiple line items is only counted once.

### Total Active Customers
```dax
total_active_customers = DISTINCTCOUNT(fact_sales[customer_id])
```
Counts customers who have at least one recorded sale — i.e. customers active within the current filter context (date range, region, etc.).

### Base Total Customers
```dax
base_total_customers = COUNT(dim_customer[customer_id])
```
Counts every customer in the customer dimension, regardless of purchase activity. Comparing this against **Total Active Customers** highlights the gap between the full customer base and customers who are actually transacting.

### Average Order to Pay
```dax
avg_order_to _pay = AVERAGE(fact_order_process[order_to_pay])
```
Averages the order-to-pay duration tracked in `fact_order_process` (the elapsed time between an order being placed and payment being received), giving a view into fulfillment and collections efficiency. *(The name carries a stray space before "pay" in the model — reproduced here exactly as defined.)*

## Row-Level Security

Configured via **Manage security roles**, not as a measure, but documented here alongside the other DAX logic in the model.

**Role:** `regional access`
**Table filter on `dim_customer`:**
```dax
[region_name] = LOOKUPVALUE(security[region], security[user_email], USERPRINCIPALNAME())
```

![Manage security roles — regional access filter](../screenshots/manage_security_roles.jpg)

`USERPRINCIPALNAME()` returns the email of the currently signed-in user. `LOOKUPVALUE` uses that email to find the user's assigned region in the `security` table, and the role filters `dim_customer` — and everything related to it — down to just that region. Validated by viewing the report as `nora.adel@arka.com` (assigned to North America in the `security` table).
