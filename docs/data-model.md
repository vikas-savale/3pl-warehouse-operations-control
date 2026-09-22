# Data Model

## 1. Purpose

This document defines the data model for the 3PL Warehouse Operations & Dispatch Control System.

The model covers warehouse operations, inventory, customer orders, fulfillment, dispatch, billing, commercial analysis and stock reconciliation.

---

## 2. Quantity Convention

- `PAC` = number of physical packs.
- `base_qty` = normalized material quantity.
- `base_uom` = `L` or `KG`.
- `base_qty_per_pac` = base quantity represented by one PAC.

---

# 3. Master Data

## 3.1 manufacturing_plants

**Grain:** One row = one manufacturing plant.

| Column        | Type    | Description                         |
| ------------- | ------- | ----------------------------------- |
| `plant_id`    | Text    | Internal plant identifier           |
| `plant_code`  | Text    | Five-digit manufacturing plant code |
| `plant_name`  | Text    | Manufacturing plant name            |
| `city`        | Text    | Plant city                          |
| `state`       | Text    | Plant state                         |
| `region`      | Text    | Operating region                    |
| `active_flag` | Boolean | Current active status               |

---

## 3.2 warehouses

**Grain:** One row = one warehouse.

| Column                 | Type    | Description                   |
| ---------------------- | ------- | ----------------------------- |
| `warehouse_id`         | Text    | Internal warehouse identifier |
| `warehouse_code`       | Text    | Business warehouse code       |
| `warehouse_name`       | Text    | Warehouse name                |
| `city`                 | Text    | Warehouse city                |
| `state`                | Text    | Warehouse state               |
| `region`               | Text    | Operating region              |
| `warehouse_type`       | Text    | Warehouse type                |
| `storage_capacity_kg`  | Decimal | Planned storage capacity      |
| `dock_count`           | Integer | Number of docks               |
| `operating_start_time` | Time    | Normal operating start time   |
| `operating_end_time`   | Time    | Normal operating end time     |
| `active_flag`          | Boolean | Current active status         |

---

## 3.3 locations

**Grain:** One row = one physical warehouse location.

| Column          | Type    | Description                   |
| --------------- | ------- | ----------------------------- |
| `location_id`   | Text    | Internal location identifier  |
| `warehouse_id`  | Text    | Parent warehouse              |
| `sloc_code`     | Text    | Stock-location classification |
| `location_code` | Text    | Physical location code        |
| `zone_code`     | Text    | Warehouse zone                |
| `aisle_code`    | Text    | Aisle identifier              |
| `rack_code`     | Text    | Rack identifier               |
| `level_no`      | Integer | Rack level                    |
| `position_no`   | Integer | Bin or position number        |
| `location_type` | Text    | Location purpose              |
| `pick_sequence` | Integer | Preferred picking sequence    |
| `max_weight_kg` | Decimal | Planned location capacity     |
| `active_flag`   | Boolean | Current active status         |

`sloc_code` values:

- `FGST` = Good Stock
- `RJCT` = Rejected Stock

`location_type` values:

- `STORAGE`
- `PICK_FACE`
- `STAGING`
- `QUARANTINE`
- `DOCK`

---

## 3.4 products

**Grain:** One row = one material/product variant.

| Column                           | Type    | Description                            |
| -------------------------------- | ------- | -------------------------------------- |
| `product_id`                     | Text    | Internal product identifier            |
| `material_code`                  | Text    | Business material code                 |
| `product_name`                   | Text    | Product name                           |
| `brand_name`                     | Text    | Brand                                  |
| `product_category`               | Text    | Product category                       |
| `product_subcategory`            | Text    | Product subcategory                    |
| `primary_manufacturing_plant_id` | Text    | Primary manufacturing plant            |
| `pack_type`                      | Text    | Main package type                      |
| `pack_size_label`                | Text    | Human-readable pack size               |
| `each_per_pac`                   | Integer | Individual units per PAC               |
| `base_qty_per_pac`               | Decimal | Base quantity per PAC                  |
| `base_uom`                       | Text    | Base quantity unit                     |
| `density_kg_per_l`               | Decimal | Density assumption for liquid products |
| `packaging_tare_kg_per_pac`      | Decimal | Packaging weight per PAC               |
| `active_flag`                    | Boolean | Current active status                  |

`base_uom` values:

- `L`
- `KG`

`batch_no` is not stored in the product master because one product variant can exist in multiple batches.

---

## 3.5 product_batches

**Grain:** One row = one batch of one product variant.

| Column                   | Type    | Description                                            |
| ------------------------ | ------- | ------------------------------------------------------ |
| `batch_id`               | Text    | Internal batch identifier                              |
| `product_id`             | Text    | Product represented by the batch                       |
| `manufacturing_plant_id` | Text    | Actual manufacturing plant                             |
| `batch_no`               | Text    | 10-digit batch number in `#####-#####` format          |
| `batch_date`             | Date    | Batch production date                                  |
| `basic_price_per_each`   | Decimal | Basic price represented by the final five batch digits |
| `unit_cost_per_base_uom` | Decimal | Product cost per base quantity unit                    |
| `active_flag`            | Boolean | Current batch status                                   |

Batch format:

```text
#####-#####
```

The first five digits identify the manufacturing plant.

The last five digits represent the basic price of one individual unit contained in the package.

The plant code embedded in `batch_no` must match `manufacturing_plant_id`.

---

## 3.6 customers

**Grain:** One row = one customer account.

| Column                | Type    | Description                        |
| --------------------- | ------- | ---------------------------------- |
| `customer_id`         | Text    | Internal customer identifier       |
| `customer_code`       | Text    | Business customer code             |
| `customer_name`       | Text    | Customer name                      |
| `customer_group_code` | Text    | Customer or distribution group     |
| `customer_type`       | Text    | Customer business type             |
| `service_level`       | Text    | Operational service classification |
| `home_city`           | Text    | Main customer city                 |
| `home_state`          | Text    | Main customer state                |
| `active_flag`         | Boolean | Current active status              |

`customer_group_code` values:

- `AM`
- `EN`
- `IN`
- `OE`

`customer_type` values:

- `Distributor`
- `Fleet Operator`
- `Industrial Buyer`
- `Equipment Dealer`
- `Service Network`

`service_level` values:

- `Standard`
- `Priority`
- `Critical`

---

## 3.7 customer_locations

**Grain:** One row = one customer location.

| Column                 | Type    | Description                           |
| ---------------------- | ------- | ------------------------------------- |
| `customer_location_id` | Text    | Internal customer-location identifier |
| `customer_id`          | Text    | Parent customer                       |
| `location_role`        | Text    | Billing or shipping role              |
| `location_code`        | Text    | Business location code                |
| `location_name`        | Text    | Location name                         |
| `address_line_1`       | Text    | Primary address                       |
| `address_line_2`       | Text    | Additional address detail             |
| `city`                 | Text    | City                                  |
| `state`                | Text    | State                                 |
| `pincode`              | Text    | Postal code                           |
| `region`               | Text    | Region                                |
| `active_flag`          | Boolean | Current active status                 |

`location_role` values:

- `BILL_TO`
- `SHIP_TO`
- `BOTH`

---

## 3.8 suppliers

**Grain:** One row = one supplier or supplying source.

| Column           | Type    | Description                  |
| ---------------- | ------- | ---------------------------- |
| `supplier_id`    | Text    | Internal supplier identifier |
| `supplier_code`  | Text    | Business supplier code       |
| `supplier_name`  | Text    | Supplier name                |
| `supplier_type`  | Text    | Supplier type                |
| `city`           | Text    | Supplier city                |
| `state`          | Text    | Supplier state               |
| `lead_time_days` | Integer | Expected supply lead time    |
| `active_flag`    | Boolean | Current active status        |

---

## 3.9 transporters

**Grain:** One row = one transport service provider.

| Column             | Type    | Description                     |
| ------------------ | ------- | ------------------------------- |
| `transporter_id`   | Text    | Internal transporter identifier |
| `transporter_code` | Text    | Business transporter code       |
| `transporter_name` | Text    | Transporter name                |
| `service_region`   | Text    | Primary service region          |
| `transport_mode`   | Text    | Transport mode                  |
| `active_flag`      | Boolean | Current active status           |

---

## 3.10 vehicles

**Grain:** One row = one physical vehicle.

| Column           | Type    | Description                            |
| ---------------- | ------- | -------------------------------------- |
| `vehicle_id`     | Text    | Internal vehicle identifier            |
| `vehicle_number` | Text    | Vehicle registration or display number |
| `transporter_id` | Text    | Operating transporter                  |
| `vehicle_type`   | Text    | Vehicle type                           |
| `capacity_kg`    | Decimal | Planned payload capacity               |
| `home_city`      | Text    | Vehicle base city                      |
| `active_flag`    | Boolean | Current active status                  |

---

## 3.11 employees

**Grain:** One row = one warehouse employee.

| Column          | Type    | Description                  |
| --------------- | ------- | ---------------------------- |
| `employee_id`   | Text    | Internal employee identifier |
| `employee_code` | Text    | Business employee code       |
| `employee_name` | Text    | Employee name                |
| `warehouse_id`  | Text    | Primary warehouse            |
| `role`          | Text    | Operational role             |
| `shift_code`    | Text    | Assigned shift               |
| `active_flag`   | Boolean | Current active status        |

`role` values:

- `Supervisor`
- `Picker`
- `Loader`
- `Receiver`
- `Warehouse Executive`

---

# 4. Order and Fulfillment

## 4.1 orders

**Grain:** One row = one customer sales order.

| Column                   | Type | Description                    |
| ------------------------ | ---- | ------------------------------ |
| `order_id`               | Text | Internal order identifier      |
| `sales_order_no`         | Text | Sales order number             |
| `order_date`             | Date | Order date                     |
| `customer_id`            | Text | Ordering customer              |
| `bill_to_location_id`    | Text | Bill-to location               |
| `bill_to_code`           | Text | Bill-to code                   |
| `ship_to_location_id`    | Text | Ship-to location               |
| `ship_to_code`           | Text | Ship-to code                   |
| `customer_po_no`         | Text | Customer purchase order number |
| `customer_po_date`       | Date | Customer PO date               |
| `order_acceptance_no`    | Text | Order acceptance reference     |
| `order_acceptance_date`  | Date | Order acceptance date          |
| `order_channel`          | Text | Order intake channel           |
| `priority`               | Text | Operational priority           |
| `required_dispatch_date` | Date | Required dispatch date         |
| `order_status`           | Text | Current order status           |

`order_channel` values:

- `EMAIL`
- `PORTAL`
- `PHONE`

`priority` values:

- `Normal`
- `High`
- `Critical`

---

## 4.2 order_lines

**Grain:** One row = one product line within one sales order.

| Column                   | Type    | Description                         |
| ------------------------ | ------- | ----------------------------------- |
| `order_line_id`          | Text    | Internal order-line identifier      |
| `order_id`               | Text    | Parent order                        |
| `line_no`                | Integer | Line number                         |
| `product_id`             | Text    | Ordered product                     |
| `ordered_pac`            | Integer | Ordered pack quantity               |
| `ordered_base_qty`       | Decimal | Ordered base quantity               |
| `base_uom`               | Text    | Base quantity unit                  |
| `unit_rate_per_base_uom` | Decimal | Selling rate per base quantity unit |
| `line_value`             | Decimal | Order line value                    |

---

## 4.3 order_allocations

**Grain:** One row = one allocation segment for one order line.

| Column                | Type    | Description                    |
| --------------------- | ------- | ------------------------------ |
| `allocation_id`       | Text    | Internal allocation identifier |
| `order_line_id`       | Text    | Parent order line              |
| `product_id`          | Text    | Allocated product              |
| `warehouse_id`        | Text    | Supplying warehouse            |
| `location_id`         | Text    | Stock location                 |
| `batch_no`            | Text    | Allocated batch                |
| `allocation_sequence` | Integer | Allocation order               |
| `allocation_date`     | Date    | Allocation date                |
| `allocated_pac`       | Integer | Allocated pack quantity        |
| `allocated_base_qty`  | Decimal | Allocated base quantity        |
| `base_uom`            | Text    | Base quantity unit             |
| `allocation_status`   | Text    | Allocation status              |

`allocation_status` values:

- `ALLOCATED`
- `PARTIAL`
- `RELEASED`
- `CANCELLED`

---

# 5. Inbound and Inventory

## 5.1 inbound_receipts

**Grain:** One row = one product/batch line under one receipt.

| Column                   | Type     | Description                      |
| ------------------------ | -------- | -------------------------------- |
| `receipt_line_id`        | Text     | Internal receipt-line identifier |
| `receipt_id`             | Text     | Receipt identifier               |
| `grn_number`             | Text     | Goods receipt number             |
| `receipt_date`           | Date     | Receipt date                     |
| `warehouse_id`           | Text     | Receiving warehouse              |
| `supplier_id`            | Text     | Supplying plant or vendor        |
| `vehicle_id`             | Text     | Inbound vehicle                  |
| `product_id`             | Text     | Received product                 |
| `batch_no`               | Text     | Received batch                   |
| `supplier_document_no`   | Text     | Supplier delivery document       |
| `supplier_document_date` | Date     | Supplier document date           |
| `arrival_time`           | Datetime | Vehicle arrival time             |
| `dock_start`             | Datetime | Receiving start time             |
| `dock_end`               | Datetime | Receiving end time               |
| `received_pac`           | Integer  | Received pack quantity           |
| `received_base_qty`      | Decimal  | Received base quantity           |
| `accepted_pac`           | Integer  | Accepted pack quantity           |
| `accepted_base_qty`      | Decimal  | Accepted base quantity           |
| `rejected_pac`           | Integer  | Rejected pack quantity           |
| `rejected_base_qty`      | Decimal  | Rejected base quantity           |
| `base_uom`               | Text     | Base quantity unit               |
| `receipt_status`         | Text     | Receipt status                   |
| `exception_reason`       | Text     | Receiving exception              |

---

## 5.2 inventory_events

**Grain:** One row = one inventory movement event.

| Column               | Type     | Description                   |
| -------------------- | -------- | ----------------------------- |
| `inventory_event_id` | Text     | Internal event identifier     |
| `event_datetime`     | Datetime | Movement date and time        |
| `warehouse_id`       | Text     | Warehouse                     |
| `location_id`        | Text     | Stock location                |
| `sloc_code`          | Text     | Stock-location classification |
| `product_id`         | Text     | Product                       |
| `batch_no`           | Text     | Batch                         |
| `event_type`         | Text     | Inventory movement type       |
| `quantity_pac`       | Integer  | Pack quantity when applicable |
| `quantity_base_qty`  | Decimal  | Base quantity moved           |
| `base_uom`           | Text     | Base quantity unit            |
| `stock_status`       | Text     | Stock status                  |
| `reference_type`     | Text     | Source transaction type       |
| `reference_id`       | Text     | Source transaction identifier |
| `event_reason`       | Text     | Movement reason               |

`event_type` values:

- `INBOUND`
- `PICK`
- `ADJUSTMENT`
- `TRANSFER_IN`
- `TRANSFER_OUT`

`stock_status` values:

- `GOOD`
- `REJECTED`

---

# 6. Picking and Vehicle Planning

## 6.1 picking_events

**Grain:** One row = one picking activity for one allocation.

| Column               | Type     | Description                 |
| -------------------- | -------- | --------------------------- |
| `pick_id`            | Text     | Internal pick identifier    |
| `pick_sequence`      | Integer  | Pick sequence               |
| `order_id`           | Text     | Parent order                |
| `order_line_id`      | Text     | Parent order line           |
| `allocation_id`      | Text     | Allocation being picked     |
| `warehouse_id`       | Text     | Picking warehouse           |
| `location_id`        | Text     | Pick location               |
| `batch_no`           | Text     | Picked batch                |
| `picker_id`          | Text     | Picker                      |
| `pick_start`         | Datetime | Pick start time             |
| `pick_end`           | Datetime | Pick end time               |
| `requested_pac`      | Integer  | Requested pack quantity     |
| `requested_base_qty` | Decimal  | Requested base quantity     |
| `picked_pac`         | Integer  | Actual picked pack quantity |
| `picked_base_qty`    | Decimal  | Actual picked base quantity |
| `base_uom`           | Text     | Base quantity unit          |
| `pick_status`        | Text     | Picking status              |
| `exception_code`     | Text     | Picking exception           |

`pick_status` values:

- `COMPLETE`
- `PARTIAL`
- `PENDING`
- `CANCELLED`

---

## 6.2 vehicle_schedule

**Grain:** One row = one planned vehicle/trip schedule.

| Column                   | Type     | Description                           |
| ------------------------ | -------- | ------------------------------------- |
| `schedule_id`            | Text     | Internal schedule identifier          |
| `warehouse_id`           | Text     | Dispatch warehouse                    |
| `vehicle_id`             | Text     | Planned vehicle                       |
| `scheduled_date`         | Date     | Schedule date                         |
| `scheduled_arrival`      | Datetime | Planned vehicle arrival               |
| `actual_arrival`         | Datetime | Actual vehicle arrival                |
| `dock_slot`              | Text     | Planned dock slot                     |
| `dock_no`                | Text     | Dock number                           |
| `route_type`             | Text     | Route type                            |
| `vehicle_status`         | Text     | Schedule status                       |
| `planned_departure`      | Datetime | Planned departure                     |
| `actual_departure`       | Datetime | Actual departure                      |
| `planned_transport_cost` | Decimal  | Planned transporter cost for the trip |
| `actual_transport_cost`  | Decimal  | Transporter cost for the trip         |

`route_type` values:

- `LOCAL`
- `REGIONAL`
- `LONG_ROUTE`

---

# 7. Dispatch

## 7.1 dispatch_events

**Grain:** One row = one dispatched product/batch line.

| Column                | Type     | Description                  |
| --------------------- | -------- | ---------------------------- |
| `dispatch_id`         | Text     | Internal dispatch identifier |
| `dispatch_line_no`    | Integer  | Dispatch line number         |
| `order_id`            | Text     | Parent order                 |
| `order_line_id`       | Text     | Dispatched order line        |
| `product_id`          | Text     | Dispatched product           |
| `batch_no`            | Text     | Dispatched batch             |
| `delivery_no`         | Text     | Delivery document number     |
| `delivery_date`       | Date     | Delivery/dispatch date       |
| `warehouse_id`        | Text     | Dispatch warehouse           |
| `schedule_id`         | Text     | Vehicle schedule             |
| `vehicle_id`          | Text     | Dispatch vehicle             |
| `transporter_id`      | Text     | Transporter                  |
| `loading_start`       | Datetime | Loading start time           |
| `loading_end`         | Datetime | Loading completion time      |
| `dispatch_time`       | Datetime | Actual dispatch time         |
| `dispatched_pac`      | Integer  | Dispatched pack quantity     |
| `dispatched_base_qty` | Decimal  | Dispatched base quantity     |
| `base_uom`            | Text     | Base quantity unit           |
| `dispatch_status`     | Text     | Dispatch status              |
| `pgi_document_no`     | Text     | PGI reference                |
| `pgi_posted_at`       | Datetime | PGI posting time             |
| `invoice_number`      | Text     | Related invoice number       |
| `e_way_bill_no`       | Text     | E-way bill reference         |
| `lr_number`           | Text     | Lorry receipt reference      |

`dispatch_status` values:

- `PLANNED`
- `LOADED`
- `DISPATCHED`
- `CANCELLED`

---

# 8. Billing

## 8.1 invoices

**Grain:** One row = one invoice document.

| Column                       | Type    | Description                    |
| ---------------------------- | ------- | ------------------------------ |
| `invoice_id`                 | Text    | Internal invoice identifier    |
| `invoice_number`             | Text    | Invoice number                 |
| `reference_number`           | Text    | Reference number               |
| `invoice_date`               | Date    | Invoice date                   |
| `order_id`                   | Text    | Related order                  |
| `sales_order_no`             | Text    | Sales order number             |
| `order_acceptance_no`        | Text    | Order acceptance reference     |
| `delivery_no`                | Text    | Delivery reference             |
| `customer_id`                | Text    | Customer                       |
| `bill_to_location_id`        | Text    | Bill-to location               |
| `bill_to_code`               | Text    | Bill-to code                   |
| `ship_to_location_id`        | Text    | Ship-to location               |
| `ship_to_code`               | Text    | Ship-to code                   |
| `transporter_id`             | Text    | Transporter                    |
| `vehicle_id`                 | Text    | Vehicle                        |
| `payment_terms_days`         | Integer | Payment term in days           |
| `payment_due_date`           | Date    | Payment due date               |
| `e_invoice_irn`              | Text    | E-invoice reference            |
| `e_invoice_ack_no`           | Text    | E-invoice acknowledgement      |
| `e_invoice_ack_date`         | Date    | E-invoice acknowledgement date |
| `e_way_bill_no`              | Text    | E-way bill number              |
| `lr_number`                  | Text    | Lorry receipt number           |
| `place_of_supply_state`      | Text    | Place of supply state          |
| `place_of_supply_state_code` | Text    | State code                     |
| `customer_freight_amount`    | Decimal | Freight charged to customer    |
| `taxable_value`              | Decimal | Taxable invoice value          |
| `cgst_amount`                | Decimal | CGST amount                    |
| `sgst_amount`                | Decimal | SGST amount                    |
| `igst_amount`                | Decimal | IGST amount                    |
| `total_tax_amount`           | Decimal | Total tax                      |
| `rounding_amount`            | Decimal | Rounding adjustment            |
| `net_payable_amount`         | Decimal | Final invoice amount           |
| `invoice_status`             | Text    | Invoice status                 |
| `payment_status`             | Text    | Payment status                 |

---

## 8.2 invoice_lines

**Grain:** One row = one billed product/batch line.

| Column                   | Type    | Description                      |
| ------------------------ | ------- | -------------------------------- |
| `invoice_line_id`        | Text    | Internal invoice-line identifier |
| `invoice_id`             | Text    | Parent invoice                   |
| `order_line_id`          | Text    | Related order line               |
| `dispatch_id`            | Text    | Related dispatch                 |
| `product_id`             | Text    | Billed product                   |
| `material_code`          | Text    | Material code                    |
| `batch_no`               | Text    | Billed batch                     |
| `billed_pac`             | Integer | Billed pack quantity             |
| `billed_base_qty`        | Decimal | Billed base quantity             |
| `base_uom`               | Text    | Base quantity unit               |
| `hsn_code`               | Text    | HSN classification               |
| `unit_rate_per_base_uom` | Decimal | Billing rate                     |
| `gross_value`            | Decimal | Gross line value                 |
| `discount_amount`        | Decimal | Discount amount                  |
| `taxable_value`          | Decimal | Taxable line value               |
| `cgst_rate`              | Decimal | CGST rate                        |
| `cgst_amount`            | Decimal | CGST amount                      |
| `sgst_rate`              | Decimal | SGST rate                        |
| `sgst_amount`            | Decimal | SGST amount                      |
| `igst_rate`              | Decimal | IGST rate                        |
| `igst_amount`            | Decimal | IGST amount                      |
| `line_total`             | Decimal | Final line amount                |

---

# 9. Physical Stock Reconciliation

## 9.1 stock_audits

**Grain:** One row = one physical count result for one product/batch/location.

| Column                   | Type    | Description                    |
| ------------------------ | ------- | ------------------------------ |
| `audit_id`               | Text    | Internal audit identifier      |
| `audit_session_id`       | Text    | Count session identifier       |
| `audit_date`             | Date    | Audit date                     |
| `warehouse_id`           | Text    | Warehouse                      |
| `location_id`            | Text    | Counted location               |
| `sloc_code`              | Text    | Stock-location classification  |
| `product_id`             | Text    | Counted product                |
| `batch_no`               | Text    | Counted batch                  |
| `system_base_qty`        | Decimal | System quantity before count   |
| `physical_good_qty`      | Decimal | Physical good quantity         |
| `physical_damage_qty`    | Decimal | Physical damage quantity       |
| `physical_leakage_qty`   | Decimal | Physical leakage quantity      |
| `physical_total_qty`     | Decimal | Total physical quantity        |
| `base_uom`               | Text    | Physical quantity unit         |
| `variance_qty`           | Decimal | Physical minus system quantity |
| `count_status`           | Text    | Reconciliation result          |
| `variance_reason`        | Text    | Main variance reason           |
| `remarks`                | Text    | Supporting remark              |
| `counted_by_employee_id` | Text    | Employee performing the count  |

`count_status` values:

- `TALLY`
- `SHORT`
- `EXCESS`

---

# 10. Key Relationships

- `manufacturing_plants.plant_id` -> `products.primary_manufacturing_plant_id`
- `manufacturing_plants.plant_id` -> `product_batches.manufacturing_plant_id`

- `warehouses.warehouse_id` -> `locations.warehouse_id`
- `warehouses.warehouse_id` -> `order_allocations.warehouse_id`
- `warehouses.warehouse_id` -> `inbound_receipts.warehouse_id`
- `warehouses.warehouse_id` -> `inventory_events.warehouse_id`
- `warehouses.warehouse_id` -> `picking_events.warehouse_id`
- `warehouses.warehouse_id` -> `vehicle_schedule.warehouse_id`
- `warehouses.warehouse_id` -> `dispatch_events.warehouse_id`
- `warehouses.warehouse_id` -> `stock_audits.warehouse_id`

- `products.product_id` -> `product_batches.product_id`
- `products.product_id` -> `order_lines.product_id`
- `products.product_id` -> `order_allocations.product_id`
- `products.product_id` -> `inbound_receipts.product_id`
- `products.product_id` -> `inventory_events.product_id`
- `products.product_id` -> `picking_events.product_id`
- `products.product_id` -> `dispatch_events.product_id`
- `products.product_id` -> `invoice_lines.product_id`
- `products.product_id` -> `stock_audits.product_id`

- `customers.customer_id` -> `customer_locations.customer_id`
- `customers.customer_id` -> `orders.customer_id`
- `customers.customer_id` -> `invoices.customer_id`

- `customer_locations.customer_location_id` -> `orders.bill_to_location_id`
- `customer_locations.customer_location_id` -> `orders.ship_to_location_id`
- `customer_locations.customer_location_id` -> `invoices.bill_to_location_id`
- `customer_locations.customer_location_id` -> `invoices.ship_to_location_id`

- `suppliers.supplier_id` -> `inbound_receipts.supplier_id`

- `transporters.transporter_id` -> `vehicles.transporter_id`
- `transporters.transporter_id` -> `dispatch_events.transporter_id`
- `transporters.transporter_id` -> `invoices.transporter_id`

- `vehicles.vehicle_id` -> `inbound_receipts.vehicle_id`
- `vehicles.vehicle_id` -> `vehicle_schedule.vehicle_id`
- `vehicles.vehicle_id` -> `dispatch_events.vehicle_id`
- `vehicles.vehicle_id` -> `invoices.vehicle_id`

- `employees.employee_id` -> `picking_events.picker_id`
- `employees.employee_id` -> `stock_audits.counted_by_employee_id`

- `orders.order_id` -> `order_lines.order_id`
- `orders.order_id` -> `dispatch_events.order_id`
- `orders.order_id` -> `invoices.order_id`

- `order_lines.order_line_id` -> `order_allocations.order_line_id`
- `order_lines.order_line_id` -> `picking_events.order_line_id`
- `order_lines.order_line_id` -> `dispatch_events.order_line_id`
- `order_lines.order_line_id` -> `invoice_lines.order_line_id`

- `order_allocations.allocation_id` -> `picking_events.allocation_id`

- `vehicle_schedule.schedule_id` -> `dispatch_events.schedule_id`

- `invoices.invoice_id` -> `invoice_lines.invoice_id`

- `dispatch_events.dispatch_id` -> `invoice_lines.dispatch_id`

- `locations.location_id` -> `order_allocations.location_id`
- `locations.location_id` -> `inventory_events.location_id`
- `locations.location_id` -> `picking_events.location_id`
- `locations.location_id` -> `stock_audits.location_id`

- `product_batches.(product_id, batch_no)` -> `order_allocations.(product_id, batch_no)`
- `product_batches.(product_id, batch_no)` -> `inbound_receipts.(product_id, batch_no)`
- `product_batches.(product_id, batch_no)` -> `inventory_events.(product_id, batch_no)`
- `product_batches.(product_id, batch_no)` -> `picking_events.(product_id, batch_no)`
- `product_batches.(product_id, batch_no)` -> `dispatch_events.(product_id, batch_no)`
- `product_batches.(product_id, batch_no)` -> `invoice_lines.(product_id, batch_no)`
- `product_batches.(product_id, batch_no)` -> `stock_audits.(product_id, batch_no)`

---

# 11. Core Data Validation

- `plant_code` must contain exactly five numeric digits.
- The first five digits of `batch_no` must match `manufacturing_plants.plant_code`.
- The last five digits of `batch_no` must represent the basic price per individual unit.
- `batch_no` must match the related product and manufacturing plant.
- `ordered_base_qty` must match the product quantity represented by `ordered_pac`.
- Cumulative `allocated_pac` must not exceed the ordered pack quantity for an order line.
- Cumulative `allocated_base_qty` must not exceed the ordered base quantity for an order line.
- Cumulative `picked_pac` must not exceed the allocated pack quantity.
- Cumulative `picked_base_qty` must not exceed the allocated base quantity.
- Cumulative `dispatched_pac` must not exceed the picked pack quantity.
- Cumulative `dispatched_base_qty` must not exceed the picked base quantity.
- `received_base_qty` must equal accepted plus rejected base quantity.
- `physical_total_qty` must equal good plus damage plus leakage quantity.
- `variance_qty` must equal physical total quantity minus system quantity.
- Transaction quantities must use the same `base_uom` as the product.
- `customer_freight_amount` must remain separate from `actual_transport_cost`.
- Profit, margin and contribution values must be derived from source values rather than stored as raw transaction fields.
