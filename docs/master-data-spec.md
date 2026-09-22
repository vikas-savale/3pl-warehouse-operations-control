# Master Data Specification

## 1. Purpose

This document defines the master-data rules for the 3PL Warehouse Operations & Dispatch Control System.

The master data supports operational transactions, traceability, warehouse analysis and commercial reporting.

---

## 2. Product Master

The product master contains **2,000 product variants**.

Product family mix reflects varied demand levels.

Each product variant is defined by:

- Product family
- Formulation or grade
- Package configuration

---

## 3. Product Families

The product catalog contains:

- Automotive Engine Oils
- Gear & Transmission Oils
- Hydraulic Fluids
- Automatic Transmission Fluids
- Coolants
- Compressor Oils
- Industrial Oils
- Turbine Oils
- Metalworking Fluids
- Greases

Each family uses controlled formulations, subcategories and package configurations.

---

## 4. Package and Quantity

| Package Configuration | each_per_pac | Base UOM |
| --------------------- | -----------: | -------- |
| `12 x 1 L`            |           12 | L        |
| `4 x 3 L`             |            4 | L        |
| `2 x 5 L`             |            2 | L        |
| `20 L`                |            1 | L        |
| `60 L`                |            1 | L        |
| `210 L`               |            1 | L        |
| `12 x 1 KG`           |           12 | KG       |
| `6 x 2 KG`            |            6 | KG       |
| `5 KG`                |            1 | KG       |
| `20 KG`               |            1 | KG       |
| `50 KG`               |            1 | KG       |
| `180 KG`              |            1 | KG       |

Package availability varies by product family and formulation.

`base_qty_per_pac` is derived from the package configuration.

For multi-unit packages:

```text
base_qty_per_pac =
each_per_pac x quantity_per_each
```

For single-unit packages, `base_qty_per_pac` equals the package quantity.

`base_uom` is `L` for liquid products and `KG` for greases.

---

## 5. Pack Type

| Package Configuration | `pack_type` |
| --------------------- | ----------- |
| `12 x 1 L`            | `BOTTLE_CARTON` |
| `4 x 3 L`             | `BOTTLE_CARTON` |
| `2 x 5 L`             | `CAN_CARTON` |
| `20 L`                | `PAIL` |
| `60 L`                | `DRUM` |
| `210 L`               | `DRUM` |
| `12 x 1 KG`            | `TUB_CARTON` |
| `6 x 2 KG`             | `TUB_CARTON` |
| `5 KG`                | `PAIL` |
| `20 KG`               | `PAIL` |
| `50 KG`               | `DRUM` |
| `180 KG`              | `DRUM` |

Pack type is determined by the package configuration and must follow the mapping above.

Pack types are:

| `pack_type` | Meaning |
| ----------- | ------- |
| `BOTTLE_CARTON` | Carton containing multiple bottles |
| `CAN_CARTON` | Carton containing multiple cans |
| `PAIL` | Individual pail |
| `DRUM` | Individual drum |
| `TUB_CARTON` | Carton containing multiple grease tubs |

---
## 6. Product Naming

Product names follow:

```text
[Brand] [Formulation] ([Pack Configuration])
```

Examples:

```text
FleetPro 15W-40 (12 x 1 L)
FleetPro 15W-40 (210 L)
HydroCore ISO VG 46 (210 L)
GreaseMax Lithium EP2 (180 KG)
```

---

## 7. Material Code

`material_code` is unique for every product variant.

Format:

```text
NIP-[FAMILY_CODE]-[FORMULATION_CODE]-[PACK_CODE]
```

Examples:

```text
NIP-AEO-FP1540-12X1L
NIP-AEO-FP1540-210L
NIP-HFL-HC46-210L
NIP-GRE-GR2-180KG
```

---

## 8. Manufacturing Plants

Each product family is assigned to a manufacturing plant.

Manufacturing plant codes are five-digit business identifiers.

The primary manufacturing plant is stored against the product.

The actual manufacturing plant is stored against the batch.

---

## 9. Batch Number

Batch numbers use the format:

```text
#####-#####
```

Structure:

```text
[Manufacturing Plant Code]-[Basic Price Code]
```

The first five digits identify the manufacturing plant.

The last five digits represent the basic price of one individual unit contained in the package.

Example:

```text
52000-12345
```

`52000` identifies the manufacturing plant and `12345` represents the basic price per individual unit.

Leading zeros in the five-digit price code must be preserved.

---

## 10. Batch Pricing

For multi-unit packages:

```text
PAC Basic Value =
each_per_pac x basic_price_per_each
```

Example:

```text
12 x 1 KG
Basic price per unit = 12,345

PAC basic value =
12 x 12,345
```

For single-unit packages, `basic_price_per_each` represents the basic price of the complete physical pack.

The basic price is before GST and other applicable tax components.

---

## 11. Batch Cost

Product cost is maintained separately from selling price.

`unit_cost_per_base_uom` represents product cost per base quantity unit.

Batch-level cost supports product contribution and margin analysis.

---

## 12. Physical Attributes

The product master contains:

- `density_kg_per_l`
- `packaging_tare_kg_per_pac`

Density applies to liquid products.

Packaging tare supports weight and warehouse-capacity analysis.

Values are synthetic project assumptions.

---

## 13. Commercial Values

The model supports:

- Selling price
- Sales value
- Discount
- Customer freight
- Product cost
- Transport cost
- Tax

Profit, margin and contribution are derived during analysis.

---

## 14. Transport Cost

Transport cost is maintained at trip level in `vehicle_schedule`.

The model supports:

- Planned transport cost
- Actual transport cost

Trip-level cost supports transporter and route cost analysis.

---

## 15. Active Status

Master records use `active_flag`.

Inactive records remain available for historical traceability and are not selected for new transactions.

---

## 16. Data Generation Rules

Master data must follow controlled business mappings.

Generated values must maintain valid relationships between:

- Product family
- Formulation
- Package configuration
- Manufacturing plant
- Batch
- Quantity
- Commercial attributes

Generated variation must not create invalid or meaningless records.

---

## 17. Validation

The generated master data must satisfy:

- 2,000 unique product variants.
- Unique `product_id`.
- Unique `material_code`.
- Valid family, formulation and package combinations.
- `pack_type` must follow the package-configuration mapping defined in Section 5.
- Correct `each_per_pac`.
- Correct `base_qty_per_pac`.
- Correct `base_uom`.
- Valid manufacturing plant reference.
- Batch format `#####-#####`.
- Batch plant code matches the manufacturing plant.
- Batch basic-price code matches `basic_price_per_each`.
- Batch is unique within its product context.
- Valid product cost.
- Valid physical attributes.
- No batch information in the `products` table.
